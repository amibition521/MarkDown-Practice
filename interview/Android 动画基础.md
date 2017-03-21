### Android 动画基础

1.Tween动画  

* alpha --- 渐入渐出
  * fromAlpha
  * toAlpha
* scale --- 缩放
  * fromXScale toXScale
  * fromYScale toYScale
  * 中心点
    * pivotX
    * PivotY
* translate --- 平移
  * fromXDelta toXDelta     ---Float & num%
  * fromYDelta toYDelta
* rotate  --- 旋转
  * fromDegrees  toDegrees
  * pivotX pivotY
* set --- 持有其他动画元素的容器。
* interpolator ---插值器，指定了插值器资源的引用 
![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/ic/interpolators.png)
* 自定义Interpolator
  * 实现Interpolator接口
  * float getInterpolation(float input)这个方法被调用

2.Property Animation

两个步骤：

​	1.计算属性值

​	2.为目标对象的属性设置属性值，即应用和刷新动画

2.1.1 计算属性值

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/ic/valuecaculate.png)

* 计算已完成动画分数 elapsed fraction ---动画完成的进度

* 计算插值（动画变化率）interpolated fraction

     当 ValueAnimator 计算完已完成动画分数后，它会调用当前设置的 TimeInterpolator，去计算得到一个 interpolated（插值）分数，在计算过程中，已完成动画百分比会被加入到新的插值计算中。  

* 计算属性值   当插值分数计算完成后，ValueAnimator 会根据插值分数调用合适的 TypeEvaluator 去计算运动中的属性值。

2.2.1  Interpolators

插值器：时间的函数，定义了动画的变化律。
插值器只需实现一个方法：`getInterpolation(float input)`,其作用就是把 0 到 1 的 elapsed fraction 变化映射到另一个 interpolated fraction。 Interpolator 接口的直接继承自`TimeInterpolator`，内部没有任何方法，而`TimeInterpolator`只有一个`getInterpolation`方法，所以所有的插值器只需实现`getInterpolation`方法即可。

插值器的原理就是通过改变实际执行动画的时间点，提前或延迟默认动画的时间点来达到加速/减速的效果。动画插值器目前都只是对动画执行过程的时间进行修饰，并没有对轨迹进行修饰。

2.2.2 Evaluators

计算属性值。

1.IntEvaluator

2.FloatEvaluator

3.ArgbEvaluator

4.TypeEvaluator   用于用户自定义计算器的接口，

`TypeEvaluator`接口只有一个方法，就是`evaluate()`方法，它允许你使用的 animator 返回一个当前动画点的属性值。

TimeInterpolator 和TypeEvaluator的区别：

`TypeEvaluator`所做的是根据数据结构计算最终的属性值，允许你定义自己的数据结构，这是官方对它的真正定义，如果你所定义的属性值的数据类型不是 float、int、color 类型，那么你需要实现 TypeEvaluator 接口的`evaluate()`方法，自己进行属性值的计算

`Interpolator`更倾向于你定义一种运动的变化率，比如匀速、加速、减速等

2.2.3 ValueAnimator

属性动画中的主要的时序引擎，如动画时间，开始、结束属性值，相应时间属性值计算方法等。包含了所有计算动画值的核心函数。也包含了每一个动画时间上的细节，信息，一个动画是否重复，是否监听更新事件等，并且还可以设置自定义的计算类型。

2.2.4 ObjectAnimator

允许你指定要进行动画的对象以及该对象的一个属性。该类会根据计算得到的新值自动更新属性。

缺点：需要目标对象的属性提供指定的处理方法   

`ObjectAnimator`的自动更新功能，依赖于属性身上的`setter`和`getter`方法，所以为了让`ObjectAnimator`能够正确的更新属性值，你必须遵从以下规范:

1. 该对象的属性必须有`get`和`set`方法（方法的格式必须是驼峰式），方法格式为 set()，因为 ObjectAnimator 会自动更新属性，它必须能够访问到属性的`setter`方法，比如属性名为`foo`,你就需要一个`setFoo()`方法，如果 setter 方法不存在，你有三种选择：
   a.添加 setter 方法
   b.使用包装类。通过该包装类通过一个有效的 setter 方法获取或者改变属性值的方法，然后应用于原始对象，比如 NOA 的`AnimatorProxy`。
   c.使用 ValueAnimator 代替

（这 3 点的意思总结起来就是一定要有一个`setter`方法，让`ObjectAnimator`能够访问到）

2.2.5 AnimatorSet    提供组合动画能力的类

2.2.6 ViewPropertyAnimator

可以方便的为某个 View 的多个属性添加并行的动画，只使用一个`ViewPropertyAnimator`对象就可以完成。它的行为更像一个`ObjectAnimator`，因为它修改的是对象的实际属性值。但它为一次性给多个属性添加动画提供了方便，而且使用`ViewPropertyAnimator`的代码更连贯更易读。

2.2.7 PropertyValuesHolder  该类持有属性，相关属性值的操作以及属性的 setter，getter 方法的创建，属性值以 Keyframe 来承载，最终由 KeyframeSet 统一处理。

2.2.8 KeyFrame 

一个`keyframe`对象由一对 time / value 的键值对组成，可以为动画定义某一特定时间的特定状态。
每个`keyframe`可以拥有自己的插值器，用于控制前一帧和当前帧的时间间隔间内的动画。

`Keyframe.ofFloat(0f,0f);` 第一个参数为：要执行该帧动画的时间节点（elapsed time / duration）第二个参数为属性值。

**Keyframe 的定制性更高，你如果想精确控制某一个时间点的动画值及其运动规律，你可以自己创建特定的 Keyframe**

**Keyframe 使用**
为了实例化一个`keyframe`对象，你必须使用某一个工厂方法：ofInt(), ofFloat(), or ofObject() 去获取合适的`keyframe`类型，然后你调用`ofKeyframe`工厂方法去获取一个`PropertyValuesHolder`对象，一旦你拥有了该对象，你可以将 PropertyValuesHolder 作为参数获取一个`Animator`

2.2.9 KeyFrameSet  

根据 Animator 传入的值，为当前动画创建一个特定类型的 KeyFrame 集合。
通常通过 ObjectAnimator.ofFloat(...)进行赋值时，这些值其实是通过一个 KeyFrameSet 来维护的
