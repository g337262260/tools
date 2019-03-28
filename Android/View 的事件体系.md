# View 的事件体系

## View 的基础知识

### 1. View  what?

view 是Android 中所有空间的基类，就是界面层控件的一种抽象。在Android 设计中，ViewGroup 也继承了View ，所以View 既可以是单个控件，也可以是一组控件。

### 2. View 的位置参数

- left              getLeft()
- right            getRight()
- top              getTop()
- bottom        getBottom()

在Android 3.0 后增加了额外的几个参数，x、y、translationX、translationY,View 在平移的过程中，top和left表示的是原始左上角的位置 信息，其值并不会发生改变，此时发生改变的是x、y、translationX、translationY。

```java
x = left + translationX
y = right + translationY
```

### 3. MotionEvent 和TouchSlop

#### MotionEvent

手指触摸屏幕后的一系列事件

- ACTION_DOWN
- ACTION_UP
- ACTION_MOVE

#### TouchSlop

系统所能识别的被认为是滑动的最小距离。

```JAVA
int touchSlop = ViewConfiguration.get(this).getScaledTouchSlop();
```

### 4. VelocityTracker、GestureDetector 和Scoller

####  VelocityTracker

速度追踪，用于追中手指在滑动过程中的速度

```java
 VelocityTracker velocityTracker = VelocityTracker.obtain();
 velocityTracker.addMovement(event);
 //1000  为单位时间
 velocityTracker.computeCurrentVelocity(1000);
 int xVelocity = (int) velocityTracker.getXVelocity();
 int yVelocity = (int) velocityTracker.getYVelocity();
 //不需要使用的时候 调用clear 方法来重置并回收内存
 velocityTracker.clear();
 velocityTracker.recycle();
```

#### GestureDetector

收拾检测，用于辅助检测用户的单击、滑动、长按、双击等行为。

```java
GestureDetector gestureDetector = new GestureDetector(this);
//解决长按屏幕后无法拖动的现象
gestureDetector.setIsLongpressEnabled(false);
boolean consume = gestureDetector.onTouchEvent(event);
return consume;
```

#### Scoller

弹性滑动对象，用于实现View的弹性滑动。

## View 的滑动

### 1. 使用scrollTo/scrollBy

### 2.使用动画

### 3.改变布局参数

- 改变LayoutParams 里面的参数
- 放置一个空View

### 4. 各种滑动方式的对比

- scrollTo/scrollBy ：操作简单，适合View中内容的滑动
- 动画：操作简单，主要适用于没有交互的View 和实现比较复杂的 动画效果
- 改变布局参数：操作稍微复杂，适用于有交互的View。


## 弹性滑动

### 1. 使用Scroller

关键的几个方法

- computeScroll()

  ```java
  @override
  public void computeScroll(){
      if(mScroll.computeScrollOffset){
          scrollTo(...);
          postInvalidate();
      }
  }
  ```

- startScroll()

  保存了我们传递的几个参数

- computeScrollOffset()

  ```java
  /**
       * Call this when you want to know the new location.  If it returns true,
       * the animation is not yet finished.
       */ 
      public boolean computeScrollOffset() {
          if (mFinished) {
              return false;
          }

          int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
      
          if (timePassed < mDuration) {
              switch (mMode) {
              case SCROLL_MODE:
                  final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                  mCurrX = mStartX + Math.round(x * mDeltaX);
                  mCurrY = mStartY + Math.round(x * mDeltaY);
                  break;
             ...
              }
          }
        return true;
      }
  ```

  根据时间的流逝来计算出当前的scrollX，scrollY。这个方法的返回值也很重要，true表示滑动还未结束，false表示滑动已经停止了。

### 2. 通过动画

- 属性动画

- ```java
  final int startX = 0;
  final int deltaX = 100;
  ValueAnimator animator = ValueAnimator.OfInt(0,1).setDuration(1000);
  animator.addUpdateListener(new AnimatorUpdateListener(){
    @override
    public void onAnimationUpdate(ValueAnimator animator){
      float fraction = animator.getAnimatedFraction();
      mButton.scrollTo(startX+(int) (deltaX *fraction,0))
    }
  });
  animator.start();
  ```

  上述代码中本质上并没有作用于任何对象，它只是在1000ms 内完成了整个动画过程

### 3. 使用延时策略

​	通过发送一系列延时消息，从而达到一种渐进式的效果，具体可以使用Handler 或View的postDelayed 方法，也可以使用线程的sleep 方法，采用这种方法不能精确的定时，因为系统的消息调度也是需要时间的，并且所需时间不定。

## View 的事件分发机制

### 1. 点击事件的传递规则

-  **public boolean dispatchTouchEvent(MotionEvent ev) **

  传递事件，用来进行事件的分发。如果事件能够传递给当前的View，那么此方法一定会被调用，返回结果受当前View 的onTouchEvent 和下级的dispatchTouchEvent 方法影响，表示是否消耗当前的事件。

-  **public boolean onInterceptTouchEvent(MotionEvent ev) **

  拦截事件，出现在ViewGroup中，返回结果表示是否拦截当前的事件

- **public boolean onTouchEvent(MotionEvent event) **

  消费事件，用来处理点击的结果，返回结果表示是否消耗当前事件，如果不消耗，在同一个事件序列中，当前View无法接受到事件。

1. 同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，即从down事件开始到up事件结束。
2. 正常情况下，一个事件序列只能被一个View拦截且消耗。例外就是一个View 将本该自己处理的事件通过onTouchEvent 强行传递给其他View处理。
3. 某个View 一旦决定拦截，那么这一个事件序列都只能又它来进行处理，并且它的的onInterceptTouchEvent 方法不会再被调用。
4. 事件一旦交给某一个View处理，那么它必须要消耗掉，否则同一事件序列中剩下的事件就不再交给它处理了。
5. 如果View 不消耗除了ACTION_DOWN 以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent 不会被调用，并且当前的View 可以持续收到后续的事件，最终这些事件会传递给Activity 处理。
6. ViewGroup 默认不拦截任何事件。
7. View 没有onInterceptTouchEvent 方法。
8. View 的onTouchEvent 默认都会消耗事件（返回true），除非它是不可点击的（clickable 和longClickable 同时为false）。
9. View 的enable 属性不影响onTouchEvent 的默认返回值，哪怕一个View 是disable 状态的。只要它的clickable 和longClickable有一个为true，那么它的onTouchEvent 就返回true。
10. onClick 会发生的前提是当前的View 是可以点击的，并且它收到了down和up事件。
11. 事件的传递过程是由外向内的，事件总是先传递给父元素，然后由父元素分发给子View  通过requestDisallowInterceptTouchEvent  方法可以在子元素中干预父元素的分发过程，但是ACTION_DOWN 事件除外。


 ## View 的滑动冲突

### 1. 常见的滑动冲突场景

- 外部滑动方向和内部滑动方向不一致
- 外部滑动方向和内部滑动方向一致
- 上面两种情况的嵌套

### 2.滑动冲突的解决方式

	#### 1.外部拦截法

​	点击事件都先经过父容器的拦截处理，如果父容器需要此点击事件就进行拦截，如果不需要就不进行拦截，这种方法比较符合点击事件的分发机制。外部拦截法主要是重写父容器的onInterceptTouchEvent 方法，在内部做相应的拦截即可。

- ACTION_DOWN：父容器必须返回false，若是拦截了该事件，后续的事件都会交给父容器进行处理，没法再传递给子元素了。
- ACTION_MOVE：根据需要来决定是否要拦截。
- ACTION_UP：必须要返回false，因为该事件本身没有太大的意义。

#### 2. 内部拦截法

​	父容器不拦截任何事件，所有时间都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交由父容器进行处理，这种方法需要配合requestDisallowIntetceptTouchEvent 方法才能正常工作，使用起来相对外部拦截法略微复杂。











