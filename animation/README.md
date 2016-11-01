# 属性动画

## 类介绍

### Animators

|Class|Description|
|-----|-----------|
|ValueAnimator|属性动画的主要定时引擎, 也是计算动画的属性值. 它包括计算动画值得所有核心功能, 包含每个动画的时间细节, 动画是否重复的信息, 监听事件更新, 计算自定义属性的值. 改变动画属性有两步: 计算动画的值和设置这些值到已经改变的对象中. ValueAnimator不实现第二步, 那么你需要使用ValueAnimator去监听值得更新, 然后修改对象的值. |
|ObjectAnimator|这是一个ValueAnimator的子类, 允许你设置一个需要运行的目标对象. 当为动画计算好一个新值时, 这个类会相应的更新对象的属性值. 你会经常用到这个类, 因为它能很简单的处理动画过程中的值. 不过有时你还得要直接用ValueAnimator, 因为ObjectAnimator有很多限制, 比如需要在目标中指定特别的acessor方法. |
|AnimatorSet|提供动画组的机制, 以便运行一组相关的动画. 你可以多个动画一起进行, 有序的进行, 或者指定一个延时播放. |


求值程序告诉属性动画系统怎么计算给定属性的值。他们需要Animator类提供时间数据，动画开始和结束的值，然后基于这些值计算动画后的值。属性动画系统提供了下面这些求值程序：

### Evaluators

|Class|Description|
|-----|-----------|
|IntEvaluator|计算int属性值的默认求值程序。|
|FloatEvaluator|计算float属性值的默认求值程序。|
|ArgbEvaluator|计算十六进制颜色值的默认求值程序。|
|TypeEvaluator|一个允许创建自己求值程序的接口。如果你的动画属性不是int，float，或者颜色，你必须实现TypeEvaluator接口，你也可以自定义一个计算int，float和颜色值的TypeEvaluator，如果你想实现与众不同的动画行为。使用一个TypeEvaluator 章节会教你怎么写一个自定义的求值程序。|

一个时间插入器指定一个时间函数中特定的值。例如，你可以指定动画是线性的，意味着动画是匀速的，或者你可以指定动画使用非线性的时间，例如，开始的时候加速，结束阶段减速。表格三描述了包含在android.view.animation中的 插入器 。如果没有提供你需要的程序，你可以实现TimeInterpolator接口创建你自己的程序。使用interpolator 部分有关于自定义 插入器 的实现。

### Interpolator

|Class/Interface|Description|
|-----|-----------|
|AccelerateDecelerateInterpolator|开始和结束时比较慢，中间加速通过的插入器。|
|AccelerateInterpolator|开始比较慢，然后逐渐加速的插入器。|
|AnticipateInterpolator|先向后运动，然后迅速向前的插入器。|
|AnticipateOvershootInterpolator|先向后运动，然后迅速向前的超过目标值，最后返回一个指定的值。|
|BounceInterpolator|在运动的后面产生弹跳的效果。|
|CycleInterpolator|循环运行特定次数。
|DecelerateInterpolator|开始很快，然后逐渐减速。|
|LinearInterpolator|匀速运动。|
|OvershootInterpolator|抛出，超过设定的值，然后返回。|
|TimeInterpolator|可以实现自己的插入器的接口。|

## 使用方法

### ValueAnimator

ValueAnimator类让你可以改变运动对象的int, float, 或者color值。你可以使用它的工厂方法：ofInt(), ofFloat(), ofObject()来获得ValueAnimator对象。例如：

```java
ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
animation.setDuration(1000);
animation.start();
```

在上面的代码中，当start()方法执行时，ValueAnimator开始计数动画的值，这个值在0到1之间，持续时间是1000ms。

你也可以定义自己的类型：

```java
ValueAnimator animation = ValueAnimator.ofObject(new MyTypeEvaluator(), startPropertyValue, endPropertyValue);
animation.setDuration(1000);
animation.start();
```

在上面的代码中，区间值是startPropertyValue到endPropertyValue之间，在MyTypeEvaluator中定义计算支持的逻辑。

上面的代码片段是没有真实效果的。因为ValueAnimator不直接操作一个对象或者属性。而你想做的是使用计算好的值去改变一个对象。你需要在ValueAnimator中定义一个监听，然后处理动画中重要的事件，就如更新帧。当你实现了监听，你就可以取得计算好的值，使用getAnimatedValue()去更新帧。

### ObjectAnimator

ObjectAnimator是ValueAnimator的子类，结合了时间引擎和ValueAnimator的值计算，可以改变目标对象自己命名的属性。它使得运动任何对象变得简单，而且你不需要实现ValueAnimator.AnimatorUpdateListener，因为属性是自动更新的。

实例化一个ObjectAnimator和ValueAnimator差不多，不过你需要指定对象和对象的属性名，以及属性变化的区间：

```java
ObjectAnimator anim = ObjectAnimator.ofFloat(foo, "alpha", 0f, 1f);
anim.setDuration(1000);
anim.start();
```

要正确更新ObjectAnimator的属性值，你必须做下面这些事：

* 属性值必须有一个设置函数（驼峰式写法）：`set<propertyName>`。因为ObjectAnimator要自动更新属性值需要使用这个方法。例如，属性名是foo，那么需要有 `setFoo()` 方法。如果这个设置函数不存在，你有三个选择：
  1. 如果你有权利的话，添加设置方法到类中。
  2. 使用外部类的有效设置函数接收值，然后传递值给初始对象。
  3. 使用ValueAnimator。
* 如果ObjectAnimator工厂方法中你只指定了一个值，那么这个值会被认为是动画的结束值。因此，你必须有一个取值函数来取得开始的值。函数样式是 `get<propertyName>` 。例如，属性名是foo的话，取值函数就是 `getFoo()` 。
* 取值函数和设置函数操作的属性值必须和动画的开始和结束值是同类型的。例如，如果使用下面的代码构造ObjectAnimator的话，两个函数就要是 `targetObject.setPropName(float)` 和 `targetObject.getPropName(float)` 。

  ```java
  ObjectAnimator.ofFloat(targetObject, "propName", 1f)
  ```

* 根据你动画的属性或者对象，你可能需要在View中调用invalidate()来重画更新后的值。你可以在onAnimationUpdate()回调函数中实现。例如，要改变一个绘图对象的颜色，在重画的时候才会更新到屏幕上。view中的所有属性设置，比如setAlpha()和setTranslationX()都会正确的作废View，所以当你调用这个方法设置新值的时候，你不需要再作废View。

### AnimatorSet

很多情况下，你希望一个动画开始或者结束的同时播放另外一个动画。Android系统让你绑定多个动画到AnimatorSet中，以便你定义同时启动动画，还是按照顺序启动动画，或者延时播放动画。

下面的实例代码来自Bouncing Balls实例，分别以下面的方式播放Animator对象：

1. 播放bounceAnim。
2. 同时播放squashAnim1, squashAnim2, stretchAnim1, stretchAnim2。
3. 播放bounceBackAnim。
4. 播放fadeAnim。

```java
AnimatorSet bouncer = new AnimatorSet();
bouncer.play(bounceAnim).before(squashAnim1);
bouncer.play(squashAnim1).with(squashAnim2);
bouncer.play(squashAnim1).with(stretchAnim1);
bouncer.play(squashAnim1).with(stretchAnim2);
bouncer.play(bounceBackAnim).after(stretchAnim2);
ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
fadeAnim.setDuration(250);
AnimatorSet animatorSet = new AnimatorSet();
animatorSet.play(bouncer).before(fadeAnim);
animatorSet.start();
```

## 动画监听

你可以在动画期间监听重要的事件。下面有一些监听器的描述：

* Animator.AnimatorListener
  `onAnimationStart()` - 动画开始时被调用
  `onAnimationEnd()` - 动画停止时被调用
  `onAnimationRepeat()` - 动画重复播放时被调用
  `onAnimationCancel()` - 动画被取消时调用，取消动画也会调用onAnimationEnd()，不管它是怎么被停止。
* ValueAnimator.AnimatorUpdateListener
  `onAnimationUpdate()` - 每帧的运动都会调用。监听这个事件是为了使用动画期间ValueAnimator产生的值。使用ValueAnimator对象的 `getAnimatedValue()` 取得值，传递到这个事件中。如果你使用ValueAnimator，那么实现这个监听是必须的。

根据你动画的属性或者对象，你可能需要在View中调用 `invalidate()` 来重画更新后的值。你可以在 `onAnimationUpdate()` 回调函数中实现。例如，要改变一个绘图对象的颜色，在重画的时候才会更新到屏幕上。view中的所有属性设置，比如 `setAlpha()` 和 `setTranslationX()` 都会正确的作废View，所以当你调用这个方法设置新值的时候，你不需要再作废View。

你可以通过扩展AnimatorListenerAdapter类来代替Animator.AnimatorListener接口，如果你不想实现Animator.AnimatorListener接口的所有方法，AnimatorListenerAdapter类提供一个空实现，你可以有选择的重写。

例如，Bouncing Balls例子创建了一个AnimatorListenerAdapter仅仅实现了 `onAnimationEnd()` 回调。

```java
ValueAnimatorAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
fadeAnim.setDuration(250);
fadeAnim.addListener(new AnimatorListenerAdapter() {
  public void onAnimationEnd(Animator animation) {
    balls.remove(((ObjectAnimator)animation).getTarget());
  }
}
```

## ViewGroup中实现布局动画

属性动画系统提供改变ViewGroup对象的能力，和改变view一样简单。

你可以使用LayoutTransition类改变ViewGroup中的布局。当你调用View的 `setVisibility()` 方法或者添加View或者删除View时，ViewGroup中的View可以有出现和消失动画。ViewGroup中剩下的View也可以移动位置。你可以在LayoutTransition中调用 `setAnimator()` 然后传递一个下面的常量到Animator对象中实现动画效果：

* APPEARING - 指定运行时view在容器中出现的动画
* CHANCE_APPEARING - 一个新的view出现时，当前view的动画。
* DISAPPEARING - View消失时的动画。
* CHANGE_DISAPPEARING - 一个view消失时，当前view移动的动画。

你可以自定义动画实现上面的四种类型事件，或者是调用系统默认的动画。

APIDemo中的LayoutAnimations例子展示了怎么为样式变化定义一个动画，然后设置这些动画到你想作用的View对象上。

LayoutAnimationsByDefault和对应的layout_animations_by_default.xml文件展示了怎么实现一个默认样式动画在XML中。你只需要做一件事，就是设置android.animateLayoutchanges属性为true。例如：

```java
<LinearLayout
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:id="@+id/verticalContainer"
    android:animateLayoutChanges="true" />
```

设置这个属性为true以后，在ViewGroup中添加和删除View时就会自动带有动画效果了。

## 使用TypeEvaluator

如果你想改变一个android系统不知道的类型，你可以通过实现TypeEvaluator接口创建自己的计算程序，android知道的类型是int, float, 或者一个颜色，通过IntEvaluator, FloatEvaluator, ArgbEvaluator实现。

下面是TypeEvaluator接口中 `evaluate()` 函数的实现。为动画当前点反应一个对应的属性值。FloatEvaluator类展示了实现方法：

```java
public class FloatEvaluator implements TypeEvaluator {
    public Object evaluate(float fraction, Object startValue, Object endValue) {
        float startFloat = ((Number) startValue).floatValue();
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
    }
}
```

提示：当ValueAnimator运行时，它会计算一个正确的运行分数（0-1之间），然后根据你使用的插入器计算出一个插值分数，这个插值分数就是TypeEvaluator中的fraction参数，所以当你计算值的时候不需要考虑插入器。


## 使用Interpolators

插入器定义了动画是怎么计算值的。例如，你可以指定一个动画是线性的，就是说动画是匀速的，或者你可以指定动画是非线性的，例如使用加速度或者减速度。

插入器从Animator接收一个运行分数，根据动画类型改变这个分数。在android.view.animation包中提供了一些普通的插入器，如果没有适合你用的，你需要自己实现TimeInterpolator接口。

下面是AccelerateDecelerateInterpolator和LinearInterpolator的比较，LinearInterpolator直接返回运行分数，而AccelerateDecelerateInterpolator开始是加速，结束时减速。下面是代码：

AccelerateDecelerateInterpolator

```java
public float getInterpolation(float input) {
    return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
}
```

### LinearInterpolator

```java
public float getInterpolation(float input) {
    return input;
}
```

下面的表格展示了1000ms中这两个插入器计算后的近似值：

|ms nelapsed|Elapsed fraction/Interpolated fraction (Linear)|Interpolated fraction (Accelerate/Decelerate)|
|---|---|
|0|0|0|
|200|.2|.1|
|400|.4|.345|
|600|.6|.8|
|800|.8|.9|
|1000|1|1|

根据表格显示，LInearInterpolator的值改变速度是相同的，没200ms为0.2，而AccelerateDecelerateInterpolator在200ms-600ms时改变比较快，在600ms-1000ms改变比较慢。

## 指定关键帧

一个 Keyframe 对象包含一个 时间/值 的数据，让你可以指定动画中特定时间下的特定状态。每个关键帧也可以有自己的插入器去控制上一个关键帧到这个关键帧时间段内的行为。

要实例化Keyframe对象，你需要一个工厂方法，`ofInt()` , `ofFloat()` 或者 `ofObject()` 去获得合适的关键帧类型。然后你可以调用 `ofKeyframe()` 工厂方法获得PropertyValuesHolder对象，传递这些关键帧到这个对象中就可以实现动画。下面的代码片段展示了它们是怎么实现的：

```java
Keyframe kf0 = Keyframe.ofFloat(0f, 0f);
Keyframe kf1 = Keyframe.ofFloat(.5f, 360f);
Keyframe kf2 = Keyframe.ofFloat(1f, 0f);
PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe("rotation", kf0, kf1, kf2);
ObjectAnimator rotationAnim = ObjectAnimator.ofPropertyValuesHolder(target, pvhRotation)
rotationAnim.setDuration(5000ms);
```

完整的使用关键帧的例子，可以看APIDemo中的MultiPropertyAnimation例子。

## 视图动画

属性动画系统允许View对象的流线型动画，比视图动画系统提供了很多改进。视图动画系统通过重绘改变View对象，由于View本身没有属性来操作这些，所以需要在每个View的容器中处理。这就导致了一个对象已经被绘制到了其他地方，而对象的行为还是在原来的位置。在 `android3.0` 中，新的属性和对应的get和set函数已经被添加，就是为了消除这个弊端。

属性动画系统能够通过改变View对象真实的属性来改变屏幕中的View对象。而且View会自动调用 `invalidate()` 函数更新屏幕。View类中的新属性是：

* translationX和translationY：设置View相对于布局容器的左边和上部的坐标值。
* rotation, rotationX, rotationY：控制相对于轴心点2D（rotation属性）和3D的旋转。
* scaleX和scaleY：控制视图根据轴心进行2D缩放。
* pivotX和pivotY：指定轴心的位置，给缩放和旋转使用，默认是对象的中心点。
* x和y：视图在容器中的最终坐标位置，是容器的左边和上部值与translationX和translationY的和。
* alpha：透明度，默认为1，为0时不可见。

改变对象的属性很简单，你只需要创建一个属性动画，然后指定想改变的属性值：

```java
ObjectAnimator.ofFloat(myView, "rotation", 0f, 360f);
```

## 使用ViewPropertyAnimator实现动画

ViewpropertyAnimator提供一个简单的方法去平行的改变View中的多个属性，它的行为像ObjectAnimator，因为他改变了View的真实属性值，但是在改变多个属性时有很多不同。使用ViewPropertyAnimator的代码更加简洁和易读。下面的代码是使用多个ObjectAnimator对象，使用单个ObjectAnimator对象，和使用ViewPropertyAnimator时的比较：

多个ObjectAnimator对象

```java
ObjectAnimator animX = ObjectAnimator.ofFloat(myView, "x", 50f);
ObjectAnimator animY = ObjectAnimator.ofFloat(myView, "y", 100f);
AnimatorSet animSetXY = new AnimatorSet();
animSetXY.playTogether(animX, animY);
animSetXY.start();
```

一个ObjectAnimator

```java
PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvyY).start();
```

ViewPropertyAnimator

```java
myView.animate().x(50f).y(100f);
```

## 在XML中定义动画

属性动画系统让你可以在XML中声明一个动画。这样你就可以在多个Activity中复用这个动画，而且编辑动画队列也很容易。

为了区别属性动画文件和以前使用的视图动画文件，从 `Android3.1` 开始，属性动画保存在res/animator/目录中（代替了res/anim/）。使用animator目录时可选，但是如果你使用ADT的话，你必须使用这个名称，英文ADT只会在这个目录下查找属性动画资源。

下面是属性动画类对应的XML标识：
* ValueAnimator - <animator>
* ObjectAnimator - <objectAnimator>
* AnimatorSet - <set>
下面的例子是顺序播放两个动画集，第一动画集中包含两个动画：

```
<set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
</set>
```

为了能运行动画，你需要包含你的XML资源在一个AnimatorSet对象中，然后在开始动画前设置动画的目标对象。使用 `setTarget()` 函数为所有的AnimatorSet子内容设置一个目标对象：

```java
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext, R.anim.property_animator);
set.setTarget(myObject);
set.start();
```

更多XML定义属性动画的内容，查看： [Animation Resources](http://developer.android.com/guide/topics/resources/animation-resource.html#Property)。
