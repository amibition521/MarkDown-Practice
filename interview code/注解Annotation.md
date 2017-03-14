### 注解Annotation

1.概念

> An annotation is a form of metadata, that can be added to Java source code. Classes, methods, variables, parameters and packages may be annotated. Annotations have no direct effect on the operation of the code they annotate.

能够添加到 Java 源代码的语法元数据。类、方法、变量、参数、包都可以被注解，可用来将信息元数据与程序元素进行关联。Annotation 中文常译为“注解”。

2.作用

a. 标记，用于告诉编译器一些信息
b. 编译时动态处理，如动态生成代码
c. 运行时动态处理，如得到注解信息
这里的三个作用实际对应着后面自定义 Annotation 时说的 @Retention 三种值分别表示的 Annotation

3.Annotaion分类

3.1 标准Annotation 

​	Override、Deprecated、SuppressWarnings

3.2元Annotation 

元 Annotation 是指用来定义 Annotation 的 Annotation，

 	@Retention,@Target,@Inherited,@Documented

3.3 自定义Annotation

4.Annotation自定义

4.1 调用

调用自定义 Annotation——MethodInfo 的示例。
MethodInfo Annotation 作用为给方法添加相关信息，包括 author、date、version。

```
public class App {

    @MethodInfo(
        author = “trinea.cn+android@gmail.com”,
        date = "2014/02/14",
        version = 2)
    public String getAppName() {
        return "trinea";
    }
}
```

4.2 定义

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface MethodInfo {

    String author() default "trinea@gmail.com";

    String date();

    int version() default 1;
}
```

这里是 MethodInfo 的实现部分
(1). 通过 @interface 定义，注解名即为自定义注解名
(2). 注解配置参数名为注解类的方法名，且：
a. 所有方法没有方法体，没有参数没有修饰符，实际只允许 public & abstract 修饰符，默认为 public，不允许抛异常
b. 方法返回值只能是基本类型，String, Class, annotation, enumeration 或者是他们的一维数组
c. 若只有一个默认属性，可直接用 value() 函数。一个属性都没有表示该 Annotation 为 Mark Annotation

勘误：不是只有一个默认属性，应该是唯一的没有默认属性的元素，当然它也可以有默认属性。　　　　　　　

(3). 可以加 default 表示默认值

4.3 元Annotation

@Documented 是否会保存到 Javadoc 文档中
@Retention 保留时间，可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS，SOURCE 大都为 Mark Annotation，这类 Annotation 大都用来校验，比如 Override, SuppressWarnings
@Target 可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER 等，未标注则表示可修饰所有
@Inherited 是否可以被继承，默认为 false

5.Annotation解析

5.1 运行时Annotation解析

(1) 运行时 Annotation 指 @Retention 为 RUNTIME 的 Annotation，可手动调用下面常用 API 解析

```
method.getAnnotation(AnnotationName.class);
method.getAnnotations();
method.isAnnotationPresent(AnnotationName.class);

```

其他 @Target 如 Field，Class 方法类似
getAnnotation(AnnotationName.class) 表示得到该 Target 某个 Annotation 的信息，因为一个 Target 可以被多个 Annotation 修饰
getAnnotations() 则表示得到该 Target 所有 Annotation
isAnnotationPresent(AnnotationName.class) 表示该 Target 是否被某个 Annotation 修饰
(2) 解析示例如下：

```
public static void main(String[] args) {
    try {
        Class cls = Class.forName("cn.trinea.java.test.annotation.App");
        for (Method method : cls.getMethods()) {
            MethodInfo methodInfo = method.getAnnotation(
MethodInfo.class);
            if (methodInfo != null) {
                System.out.println("method name:" + method.getName());
                System.out.println("method author:" + methodInfo.author());
                System.out.println("method version:" + methodInfo.version());
                System.out.println("method date:" + methodInfo.date());
            }
        }
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
}

```

以之前自定义的 MethodInfo 为例，利用 Target（这里是 Method）getAnnotation 函数得到 Annotation 信息，然后就可以调用 Annotation 的方法得到响应属性值

5.2 编译时Annotation 解析

1) 编译时 Annotation 指 @Retention 为 CLASS 的 Annotation，甴编译器自动解析。需要做的
a. 自定义类继承自 AbstractProcessor
b. 重写其中的 process 函数

```
@SupportedAnnotationTypes({ "cn.trinea.java.test.annotation.MethodInfo" })
public class MethodInfoProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
        HashMap<String, String> map = new HashMap<String, String>();
        for (TypeElement te : annotations) {
            for (Element element : env.getElementsAnnotatedWith(te)) {
                MethodInfo methodInfo = element.getAnnotation(MethodInfo.class);
                map.put(element.getEnclosingElement().toString(), methodInfo.author());
            }
        }
        return false;
    }
}
```

SupportedAnnotationTypes 表示这个 Processor 要处理的 Annotation 名字。
process 函数中参数 annotations 表示待处理的 Annotations，参数 env 表示当前或是之前的运行环境
process 函数返回值表示这组 annotations 是否被这个 Processor 接受，如果接受后续子的 rocessor 不会再对这个 Annotations 进行处理