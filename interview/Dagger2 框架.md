### Dagger2 框架

1.带着问题去阅读：

- dagger2中的Inject,Component,Module,Provides等等都是什么东东，有什么作用?
- dagger2到底能带来哪些好处？
- 怎样把dagger2应用到具体项目中？

2.介绍

使用注解来实现依赖注入，但它利用 APT(Annotation Process Tool) 在编译时生成辅助类，这些类继承特定父类或实现特定接口，程序在运行时 Dagger 加载这些辅助类，调用相应接口完成依赖生成和注入。

3.关键字

3.1 `@Inject`  

3.1.1 对象的生成   

用`@Inject` 修饰构造器，生成调用对象

```
public class Boss {
    ...
    
    @Inject
    public Boss() {
        ...
    }
    
    ...
}
```

*Dagger 可调用的对象生成方式有两种：一种是用 @Inject 修饰的构造函数，上面就是这种方式。另外一种是用 @Provides 修饰的函数，*

3.1.2  设置到`Activity` 中

```
public class MainActivity extends Activity {
    @Inject Boss boss;
    ...
}
```

最后，我们在合适的位置(例如 onCreate() 函数中)调用 ObjectGraph.inject() 函数，Dagger 就会自动调用上面 (1) 中的生成方法生成依赖的实例，并注入到当前对象(MainActivity)。

```
public class MainActivity extends Activity {
    @Inject Boss boss;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ObjectGraph.create(AppModule.class).inject(this);
    }
    ...
}
```

ObjectGraph 是由 Dagger 提供的类，可以简单理解为一个依赖管理类，它的 create() 函数的参数是一个数组，为所有需要用到的 Module(例如本例中的 AppModule)。AppModule 是一个自定义类，在 Dagger 中称为`Module`，通过 @Module 注解进行标记，代码如下：

```
@Module(injects = MainActivity.class)
public class AppModule {
}
```

3.2 自定义依赖生成方式

3.2.1 `Provides`    

对构造函数进行注解是很好用的依赖对象生成方式，然而它并不适用于所有情况。例如：

- 接口(Interface)是没有构造函数的，当然就不能对构造函数进行注解
- 第三方库提供的类，我们无法修改源码，因此也不能注解它们的构造函数
- 有些类需要提供统一的生成函数(一般会同时私有化构造函数)或需要动态选择初始化的配置，而不是使用一个单一的构造函数.

以上三种方式可以通过`@Provides` 注解来标记自定义的生成函数.

 ```
@Provides
Coder provideCoder(Boss boss) {
    return new Coder(boss);
}
 ```

*@Provides 注解修饰的函数如果含有参数，它的所有参数也需要提供可被 Dagger 调用到的生成函数。*

所有 @Provides 注解的生成函数都需要在`Module`中定义实现，这就是上面提到的 Module 的作用之一——让 ObjectGraph 知道怎么生成某些依赖。

```
@Module
public class AppModule {
    @Provides
    Coder provideCoder(Boss boss) {
        return new Coder(boss);
    }
}
```

3.2.2

##### @Inject 和 @Provide 两种依赖生成方式区别

a. @Inject 用于注入可实例化的类，@Provides 可用于注入所有类
b. @Inject 可用于修饰属性和构造函数，可用于任何非 Module 类，@Provides 只可用于用于修饰非构造函数，并且该函数必须在某个`Module`内部
c. @Inject 修饰的函数只能是构造函数，@Provides 修饰的函数必须以 provide 开头

3.3 单例 `Singleton`  

```
// @Inject 注解构造函数的单例模式
@Singleton
public class Boss {
    ...

    @Inject
    public Boss() {
        ...
    }

    ...
}
```

```
// @Provides 注解函数的单例模式
@Provides
@Singleton
Coder provideCoder(Boss boss) {
    return new Coder(boss);
}
```

3.4 `Qualifier`  限定符   

3.4.1 

如果有两类程序员，他们的能力值 power 分别是 5 和 1000，应该怎样让 Dagger 对他们做出区分呢？使用 @Qualifier 注解即可。

(1). 创建一个 @Qualifier 注解，用于区分两类程序员：

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Level {
  String value() default "";
}

```

(2). 为这两类程序员分别设置 @Provides 函数，并使用 @Qualifier 注解对他们做出不同的标记：

```
@Provides @Level("low") Coder provideLowLevelCoder() {
    Coder coder = new Coder();
    coder.setName("战五渣");
    coder.setPower(5);
    return coder;
}

@Provides @Level("high") Coder provideHighLevelCoder() {
    Coder coder = new Coder();
    coder.setName("大神");
    coder.setPower(1000);
    return coder;
}

```

(3). 在声明 @Inject 对象的时候，加上对应的 @Qualifier 注解。

```
@Inject @Level("low") Coder lowLevelCoder;
@Inject @Level("high") Coder highLevelCoder;
```

3.5 编译时检查

**(1)**. 所有需要依赖注入的类，需要被显式声明在相应的`Module`中。
**(2)**. 一个`Module`中所有 @Provides 函数的参数都必须在这个 Module 中提供相应的被 @Provides 修饰的函数，或者在 @Module 注解后添加 "complete = false" 注明这是一个不完整 Module，表示它依赖不属于这个 Module 的其他 Denpendency。
**(3)**. 一个`Module`中所有的 @Provides 函数都要被它声明的注入对象所使用，或者在 @Module 注解后添加 "library = ture" 注明它含有对外的 Denpendency，可能被其他`Module`依赖。

3.6 Dagger相关概念

**Module：**也叫 ModuleClass，指被 @Module 注解修饰的类，为 Dagger 提供需要依赖注入的 Host 信息及一些 Dependency 的生成方式。

**ModuleAdapter：**指由 APT 根据 @Module 注解自动生成的类，父类是 Dagger 的 ModuleAdapter.java，与 ModuleClass 对应，以 ModuleClass 的 ClassName 加上 $$ModuleAdapter 命名，在 ModuleClass 的同一个 package 下。

**Binding：**指由 APT 根据 @Inject 注解和 @Provides 注解自动生成，最终继承自 Binding.java 的类。为下面介绍的 DAG 图中的一个节点，每个 Host 及依赖都是一个 Binding。

**InjectAdapter：**每个属性或构造函数被 @Inject 修饰的类都会生成一个 继承自 Binding.java 的子类，生成类以修饰类的 ClassName 加上 $$InjectAdapter 命名，在该类的同一个 package 下。

**ProvidesAdapter：**每个被 @Provides 修饰的生成函数都会生成一个继承自 ProvidesBinding.java 的子类，ProvidesBinding.java 继承自 Binding.java，生成类以 Provide 函数名首字母大写加上 ProvidesAdapter 命名，是 Provide 函数所在 Module 对应生成的`ModuleAdapter`中的静态内部类。
Binding 更具体信息在下面会介绍。

**Binding 安装：**指将 Binding 添加到 Binding 库中。对 Dagger Linker.java 代码来说是将 Binding 添加到 Linker.bindings 属性中，Linker.bindings 属性表示某个 ObjectGraph 已安装的所有 Binding。对于下面的 DAG 图来说是将节点放到图中，但尚未跟其他任何节点连接起来。

**Binding 连接：**把当前 Binding 和它内部依赖的 Binding 进行连接，即初始化这个 Binding 内部的所有 Binding，使它们可用。对 DAG 的角度说，就是把某个节点与其所依赖的各个节点连接起来。