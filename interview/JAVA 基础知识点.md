### JAVA 基础知识点

1. private 

   public 

   default

   protected

2.final 

​	声明的方法不能被覆盖，

3.static

4.执行顺序

​	初始化代码块在构造器执行之前执行，类初始化阶段先执行最顶层父类的静态初始化块，依次向下执行，最后执行当前类的静态初始化块；创建对象时，先调用顶层父类的构造方法，依次向下执行，最后调用本类 的构造方法。

5.

```
String s = "hello";
String t = "hello";
char c[] = ['h','e','l','l','o'];
下面语句将返回true
1.s.equals(t);
2.s == t;
3.t.equals(new String("hello"));
```

字符数组会在最后加上`'\n'`

6.抽象类和接口

* 抽象类
  * ​
* 接口
  * 允许定义成员，但必须是常量
* 都无法实例化，

7.hashmap 和hashtable 的区别

* hashtable 不允许null值(key 和value都不行)，hashmap允许null值（key和value都可以）。
* hashtable 的方法是同步的，hashmap不是
* 迭代hashmap采用快速失败机制，而hashtable不是

8.反射

