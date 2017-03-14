### Support Annotation Library 详解

1.Nullness

* @Nullable作用于函数参数或返回值，标记参数或返回值为可以空
* @NonNull作用于函数参数或返回值，标记参数或返回值为不能为空

测试工具 Android Lint 

2.资源类型注解

- AnimatorRes：标记整型值是Android.R.animation类型。
- AnimRes：标记整型是android.R.anim类型。
- AnyRes：标记整型是任何一种资源类型，如果确切知道表示的是哪一个具体资源的话，建议显式指定。
- ArrayRes：标记整型是android.R.array类型。
- AttrRes：标记整型是android.R.attr类型。
- BoolRes：标记整型是布尔类型。
- ColorRes：标记整型是android.R.color类型。
- DrawableRes：标记整型是android.R.drawable类型。
- FranctionRes：标记整型值是fraction类型，这个比较少见，这种类型资源常见于Animation Xml中，比如50%，表示占parent的50%
- IdRes：标记整型是android.R.id类型。
- IntegerRes：标记整型是android.R.integer类型。
- InterpolatorRes：标记整型是android.R.interpolator类型，插值器，在Animation Xml中使用较多。
- LayoutRes：标记整型是android.R.layout类型。
- MenuRes：标记整型是android.R.menu类型。
- RawRes：标记整型是android.R.raw类型。
- StringRes：标记整型是android.R.string类型。
- StyleableRes：标记整型是android.R.styleable类型。
- StyleRes：标记整型是android.R.style类型。
- XmlRes：标记整型是android.R.xml类型。

3.类型定义注解

@IntDef注解用来创建一个整型类型定义的新注解，我们可以使用这个新注解来标记自己编译的API

```
public abstract class AnnotationTest {
    public static final  int TEST_1 = 0;
    public static final  int TEST_2 = 1;
    public static final  int TEST_3 = 2;

    @Retention(RetentionPolicy.SOURCE)
    @IntDef({TEST_1, TEST_2, TEST_3})
    public @interface TestAnnotation{}

    @TestAnnotation
    public abstract int getTestAnnotation();

    public abstract void setTestAnnotation(@TestAnnotation int testAnnotation);

}
```

4.线程注解

- 1.@UiThread：标记运行在UI线程，一个UI线程是Activity运行所在的主窗口，对于一个应用而言，可能存在多个UI线程。每个UI线程对应不同的主窗口。
- 2.@MainThread：标记运行在主线程，一个应用只有一个主线程，主线程也是@UiThread线程。通常情况下，我们使用@MainThread来注解生命周期相关函数，使用@UiThread来注解视图相关函数，一般情况下@MianThread和@UiThraed是可以互换的。
- 3.@WorkerThread：标记运行在后台运行线程。
- 4.@BinderThread：标记运行在Binder线程。

5.RGB颜色值注解

@ColorRes来标记参数类型需要传入颜色类型的id，而使用@ColorInt注解是标记参数类型需要传入RGB或者ARGB颜色值的整型值。

6.值范围注解

- 1.@Size：对于类似数组、集合和字符串之类的参数，我们可以使用@Size注解来表示这些参数的大小。

  用法：

  @Size(min=1)//可以表示集合不可以为空

  @Size(max=23)//可以表示字符串最大字符个数为23

  @Size(2)//表示数组元素个数为2个

  @Size(multiple=2)//可以表示数组大小是2的倍数

- 2.@IntRange：参数类型是int或者long，用法如下

  public void setInt(@intRange(from=0,to=255)){...}

  3.@FloatRange：参数类型是float或者double，用法如下。

- public void setFloat(@FloatRange(from=0.0,to=1.0)){...}

7.权限注解

@RequiresPermission注解。

1.如果需要一个权限则加注解。

> @RequiresPermission(Manifest.permission.SET_WALLPAPER)

2.如果需要一个集合至少一个权限，那么就加注解。

> @RequiresPermission(anyOf = {Manifest.permission.SET_WALLPAPER,Manifest.permission.CAMERA})

3.如果同时需要多个权限，那么就加注解。

> @RequiresPermission(allOf = 
>
> {Manifest.permission.SET_WALLPAPER,Manifest.permission.CAMERA})

4.对于Intent调用所需权限的ACTION字符串定义处添加注解。

> @RequiresPermission(android.Manifest.permission.BLUETOOTH)
>
> String ACTION_REQUEST_DISCOVERRAVLE = "android.bluetooth.adapter.REQUEST_DISCOVERRAVLE";

5.对于ContentProvider所需权限，可能有读和写两个操作。对应不同的权限。

> @RequiresPermission.Read(@RequestPermission(READ_HISTORY_BOOLMARKS))
>
> @RequiresPermission.Write(@RequestPermission(WRITE_HISTORY_BOOLMARKS))
>
> public static final Uri BOOKMARKS_URI = Uri.parse("content://browser/bookmarks);

8.重写函数注解

如果API允许重写某个函数，但是要求在重写该函数时需要调用super父类的函数。可以加注解@CallSuper来提示开发者。

9.@Keep注解

@keep是用来标记在Proguard混淆过程中不需要混淆的类或者方法。在混淆时一些不需要混淆的会使用

> -keep class com.foo.bar{public static <method>}

9.@SuppressWarning 注解----过滤警告信息

@SuppressWarnings("unchecked")

告诉编译器忽略 unchecked 警告信息，如使用List，ArrayList等未进行参数化产生的警告信息。

·   @SuppressWarnings("serial")

如果编译器出现这样的警告信息：The serializable class WmailCalendar does not declare a static final serialVersionUID field of type long使用这个注释将警告信息去掉。

·   @SuppressWarnings("deprecation")

如果使用了使用@Deprecated注释的方法，编译器将出现警告信息。使用这个注释将警告信息去掉。

·   @SuppressWarnings("unchecked", "deprecation")

告诉编译器同时忽略unchecked和deprecation的警告信息。

·   @SuppressWarnings(value={"unchecked", "deprecation"})

等同于@SuppressWarnings("unchecked", "deprecation")