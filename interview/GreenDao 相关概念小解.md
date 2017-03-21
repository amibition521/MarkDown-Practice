### GreenDao 相关概念小解

1.介绍

greenDAO是一个对象关系映射（ORM）的框架，能够提供一个接口通过操作对象的方式去操作关系型数据库，它能够让你操作数据库时更简单、更方便。

优点：

* 性能高，号称Android最快的关系型数据库
* 内存占用小
* 库文件比较小，小于100K，编译时间低，而且可以避免65K方法限制
* 支持数据库加密 greendao支持SQLCipher进行数据库加密
* 简洁易用的API

`采用注解的方式通过编译方式生成Java数据对象和DAO对象。`

#### 1.）实体@Entity注解

**schema**：告知GreenDao当前实体属于哪个schema
**active：**标记一个实体处于活动状态，活动实体有更新、删除和刷新方法
**nameInDb：**在数据中使用的别名，默认使用的是实体的类名
**indexes：**定义索引，可以跨越多个列
**createInDb：**标记创建数据库表**

#### 2.）基础属性注解

**@Id :**主键 Long型，可以通过@Id(autoincrement = true)设置自增长
**@Property：**设置一个非默认关系映射所对应的列名，默认是的使用字段名举例：@Property (nameInDb="name")
**@NotNul：**设置数据库表当前列不能为空
**@Transient**：添加次标记之后不会生成数据库表的列

#### 3.)索引注解

**@Index：**使用@Index作为一个属性来创建一个索引，通过name设置索引别名，也可以通过unique给索引添加约束
**@Unique：**向数据库列添加了一个唯一的约束

#### 4.）关系注解

**@ToOne：**定义与另一个实体（一个实体对象）的关系
**@ToMany：**定义与多个实体对象的关系

(一) @Entity 定义实体
@nameInDb 在数据库中的名字，如不写则为实体中类名
@indexes 索引
@createInDb 是否创建表，默认为true,false时不创建
@schema 指定架构名称为实体
@active 无论是更新生成都刷新
(二) @Id
(三) @NotNull 不为null
(四) @Unique 唯一约束
(五) @ToMany 一对多
(六) @OrderBy 排序
(七) @ToOne 一对一
(八) @Transient 不存储在数据库中
(九) @generated 由greendao产生的构造函数或方法



