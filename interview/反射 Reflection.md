### 反射 Reflection

1.反射

在运行时获取类的函数、属性、父类、接口等 Class 内部信息的机制，

情景：

* 通过反射的形式来使用在运行时才存在的类
* 对于类的内部信息不可知，必须得到运行时才能获取类的具体信息

2.反射Class以及构造对象

2.1 获取Class对象

三种方式：

（1）类的名字

```
Class<?> myObjectClass = MyObject.class;
```

(2) 获得对象—>Class对象

```
Student me = new Student("mr.simple");
Class<?> clazz = me.getClass();
```

(3) 完整类路径

```
Class<?> myObjectClass = Class.forName("com.simple.User");
```

接口说明：

```
// 加载指定的 Class 对象，参数 1 为要加载的类的完整路径，例如"com.simple.Student". ( 常用方式 )
public static Class<?> forName (String className)

// 加载指定的 Class 对象，参数 1 为要加载的类的完整路径，例如"com.simple.Student";
// 参数 2 为是否要初始化该 Class 对象，参数 3 为指定加载该类的 ClassLoader.
public static Class<?> forName (String className, boolean shouldInitialize, ClassLoader classLoader)
```

2.2通过Class对象构造目标类型的对象

通过反射构造对象，我们首先要获取类的 Constructor(构造器)对象，然后通过 Constructor 来创建目标类的对象。

```
private static void classForName() {
        try {
            // 获取 Class 对象
            Class<?> clz = Class.forName("org.java.advance.reflect.Student");
            // 通过 Class 对象获取 Constructor，Student 的构造函数有一个字符串参数
            // 因此这里需要传递参数的类型 ( Student 类见后面的代码 )
            Constructor<?> constructor = clz.getConstructor(String.class);
            // 通过 Constructor 来创建 Student 对象
            Object obj = constructor.newInstance("mr.simple");
            System.out.println(" obj :  " + obj.toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

获取构造函数接口

```
// 获取一个公有的构造函数，参数为可变参数，如果构造函数有参数，那么需要将参数的类型传递给 getConstructor 方法
public Constructor<T> getConstructor (Class...<?> parameterTypes)
// 获取目标类所有的公有构造函数
public Constructor[]<?> getConstructors ()
```

在反射调用之前将此对象的 accessible 标志设置为 true，以此来提升反射速度。值为 true 则指示反射的对象在使用时应该取消 Java 语言访问检查。值为 false 则指示反射的对象应该实施 Java 语言访问检查。

```
Constructor<?> constructor = clz.getConstructor(String.class);
   // 设置 Constructor 的 Accessible
   constructor.setAccessible(true);

   // 设置 Methohd 的 Accessible
   Method learnMethod = Student.class.getMethod("learn"， String.class);
   learnMethod.setAccessible(true);
```

3.反射获取 类中函数

3.1 获取当前类中定义的方法

要获取当前类中定义的所有方法可以通过 Class 中的 getDeclaredMethods 函数，它会获取到当前类中的 public、default、protected、private 的所有方法。而 getDeclaredMethod(String name, Class...<?> parameterTypes)则是获取某个指定的方法。

 private static void showDeclaredMethods() {

      Student student = new Student("mr.simple");
        Method[] methods = student.getClass().getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("declared method name : " + method.getName());
        }
    
        try {
            Method learnMethod = student.getClass().getDeclaredMethod("learn", String.class);
            // 获取方法的参数类型列表
            Class<?>[] paramClasses = learnMethod.getParameterTypes() ;
            for (Class<?> class1 : paramClasses) {
                System.out.println("learn 方法的参数类型 : " + class1.getName());
            }
            // 是否是 private 函数，属性是否是 private 也可以使用这种方式判断
            System.out.println(learnMethod.getName() + " is private "
                    + Modifier.isPrivate(learnMethod.getModifiers()));
            learnMethod.invoke(student, "java ---> ");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
3.2 获取当前类、父类中定义的公有方法

要获取当前类以及父类中的所有 public 方法可以通过 Class 中的 getMethods 函数，而 getMethod 则是获取某个指定的方法。代码示例如下 :

```
    private static void showMethods() {
        Student student = new Student("mr.simple");
        // 获取所有方法
        Method[] methods = student.getClass().getMethods();
        for (Method method : methods) {
            System.out.println("method name : " + method.getName());
        }

        try {
            // 通过 getMethod 只能获取公有方法，如果获取私有方法则会抛出异常，比如这里就会抛异常
            Method learnMethod = student.getClass().getMethod("learn", String.class);
            // 是否是 private 函数，属性是否是 private 也可以使用这种方式判断
            System.out.println(learnMethod.getName() + " is private " + Modifier.isPrivate(learnMethod.getModifiers()));
            // 调用 learn 函数
            learnMethod.invoke(student, "java");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

接口说明

```
// 获取 Class 对象中指定函数名和参数的函数，参数一为函数名，参数 2 为参数类型列表
public Method getDeclaredMethod (String name, Class...<?> parameterTypes)

// 获取该 Class 对象中的所有函数( 不包含从父类继承的函数 )
public Method[] getDeclaredMethods ()

// 获取指定的 Class 对象中的**公有**函数，参数一为函数名，参数 2 为参数类型列表
public Method getMethod (String name, Class...<?> parameterTypes)

// 获取该 Class 对象中的所有**公有**函数 ( 包含从父类和接口类集成下来的函数 )
public Method[] getMethods ()

```

这里需要注意的是 getDeclaredMethod 和 getDeclaredMethods 包含 private、protected、default、public 的函数，并且通过这两个函数获取到的只是在自身中定义的函数，从父类中集成的函数不能够获取到。而 getMethod 和 getMethods 只包含 public 函数，父类中的公有函数也能够获取到。

4. 反射获取类中的属性

4.1 获取当前类中定义的属性

要获取当前类中定义的所有属性可以通过 Class 中的 getDeclaredFields 函数，它会获取到当前类中的 public、default、protected、private 的所有属性。而 getDeclaredField 则是获取某个指定的属性。代码示例如下 :

```
    private static void showDeclaredFields() {
        Student student = new Student("mr.simple");
        // 获取当前类和父类的所有公有属性
        Field[] publicFields = student.getClass().getDeclaredFields();
        for (Field field : publicFields) {
            System.out.println("declared field name : " + field.getName());
        }

        try {
            // 获取当前类和父类的某个公有属性
            Field gradeField = student.getClass().getDeclaredField("mGrade");
            // 获取属性值
            System.out.println(" my grade is : " + gradeField.getInt(student));
            // 设置属性值
            gradeField.set(student, 10);
            System.out.println(" my grade is : " + gradeField.getInt(student));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

```

#### 4.2 获取当前类、父类中定义的公有属性

要获取当前类以及父类中的所有 public 属性可以通过 Class 中的 getFields 函数，而 getField 则是获取某个指定的属性。代码示例如下 :

```
    private static void showFields() {
        Student student = new Student("mr.simple");
        // 获取当前类和父类的所有公有属性
        Field[] publicFields = student.getClass().getFields();
        for (Field field : publicFields) {
            System.out.println("field name : " + field.getName());
        }

        try {
            // 获取当前类和父类的某个公有属性
            Field ageField = student.getClass().getField("mAge");
            System.out.println(" age is : " + ageField.getInt(student));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

```

**接口说明**

```
// 获取 Class 对象中指定属性名的属性，参数一为属性名
public Method getDeclaredField (String name)

// 获取该 Class 对象中的所有属性( 不包含从父类继承的属性 )
public Method[] getDeclaredFields ()

// 获取指定的 Class 对象中的**公有**属性，参数一为属性名
public Method getField (String name)

// 获取该 Class 对象中的所有**公有**属性 ( 包含从父类和接口类集成下来的公有属性 )
public Method[] getFields ()

```

这里需要注意的是 getDeclaredField 和 getDeclaredFields 包含 private、protected、default、public 的属性，并且通过这两个函数获取到的只是在自身中定义的属性，从父类中集成的属性不能够获取到。而 getField 和 getFields 只包含 public 属性，父类中的公有属性也能够获取到。

5. 反射获取父类与接口

5.1 父类

```
Student student = new Student("mr.simple");
    Class<?> superClass = student.getClass().getSuperclass();
    while (superClass != null) {
        System.out.println("Student's super class is : " + superClass.getName());
        // 再获取父类的上一层父类，直到最后的 Object 类，Object 的父类为 null
        superClass = superClass.getSuperclass();
    }
```

5.2 接口

```
private static void showInterfaces() {
        Student student = new Student("mr.simple");
        Class<?>[] interfaceses = student.getClass().getInterfaces();
        for (Class<?> class1 : interfaceses) {
            System.out.println("Student's interface is : " + class1.getName());
        }
    }
```

6.注解Annotation

```
// 获取指定类型的注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) ;
// 获取 Class 对象中的所有注解
public Annotation[] getAnnotations() ;
```