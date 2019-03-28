# Android View  动画框架

## 视图动画

- 透明度动画

  ```java
  AlphaAnimation aa = new AlphaAnimation(0,1);
  aa.setDuration(1000);
  view.starttAnimation(aa);
  ```

- 旋转动画

  ```java
  /**
  * @param  ①旋转的初始角度  ②旋转的终止角度  ③ ④  旋转中心的横纵坐标,也可以用自身当作参考系
  */
  RotateAnimation ra = new RotateAnimation(0,360,100,100)
  ra.setDuration(1000);
  view.startAnimation(ra);
  ```

- 位移动画

  ```java
  /**
  *@param   始末位置的横纵坐标
  */
  TranslateAnimation ta = new TranslateAnimation(0,200,0,300);
  ta.setDuration(1000);
  view.startAnimation(ta);
  ```

- 缩放动画

  ```java
  /*
  * @param   缩放的中心点
  */
  ScaleAnimation sa = new ScaleAnimation(0,2,0,2);
  sa.setDuratuon(1000);
  view.startAnimation(sa);
  ```

- 动画集合

  ```java
  AnimationSet as = new AnimationSet(true);
  as.addAnimation(animation1);
  as.addAnimation(animation2);
  .....
  view.startAnimation(as)
  ```

## 属性动画

### 1. ObjectAnimator

```java
/*
* @param 	①操作的View   ②要操纵的属性	③设置属性的变化值 
*/
ObjectAnimator animator = ObjectAnimator.ofFloat(view,"translateX",300);
animator.setDuration(1000);
animator.start();
```

- **要操纵的属性必须具有get、set方法，不然ObjectAnimator就无法起效**	
  - translationX		translationY
   - rotationrotationX		rotationY
  - scaleX     scaleY
  - x和y
  - alpha

### 2. PropertyValuesHolder

类似于视图动画中的AniamtionSet

```java
 PropertyValuesHolder translationY = PropertyValuesHolder.ofFloat("translationY", 100f);
        PropertyValuesHolder scaleX = PropertyValuesHolder.ofFloat("ScaleX", 1f, 0, 1f);
        PropertyValuesHolder sacleY = PropertyValuesHolder.ofFloat("ScaleY", 1f, 0, 1f);
        ObjectAnimator.ofPropertyValuesHolder(mImage,translationY,scaleX,sacleY).setDuration(1000).start();
```

### 3.ValueAnimator

本身不提供任何动画效果，可以在AnimatorUpdateListener中监听数值的变换

```java
ValueAnimator valueAnimator = ValueAnimator.ofInt(start, end);

        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                int animatedValue = (int) animation.getAnimatedValue();

            }
        });
```

### 4.AnimatorSet

```java
ObjectAnimator translationX = ObjectAnimator.ofFloat(mImage, "translationX", 300f);
        ObjectAnimator scaleX = ObjectAnimator.ofFloat(mImage, "ScaleX", 1f, 0, 1f);
        ObjectAnimator scaleY = ObjectAnimator.ofFloat(mImage, "ScaleY", 1f, 0, 1f);
        AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.setDuration(1000);
        animatorSet.playTogether(translationX,scaleX,scaleY);
        animatorSet.start();
```

### 5.在XML中使用动画

```xml
<objectAnimator
        android:duration="1000"
        android:propertyName="scaleX"
        android:valueFrom="1.0"
        android:valueTo="2.0"
        android:valueType="floatType"/>
 
```

```java
Animator animator = AnimatorInflater.loadAnimator(this, R.animator.scale_animator);
        animator.setTarget(mImage);
        animator.start();
```

### 6.View的animate方法

```java
mImage.animate()
                .alpha(1)    //设置透明度
                .y(300)      //设置 坐标
                .setDuration(1000)   //设置动画时长
                .withStartAction(new Runnable() {
                    @Override
                    public void run() {

                    }
                })
                .withEndAction(new Runnable() {
                    @Override
                    public void run() {

                    }
                })
                .start();
```

## 布局动画

所谓的布局动画是指在ViewGroup上，给ViewGroup增加View时添加一个动画过渡效果

`Android：animateLayoutChanges = "true"`

```java
mViewGroup = (LinearLayout) findViewById(R.id.imagegroup);
        //设置过渡动画
        ScaleAnimation scaleAnimation = new ScaleAnimation(0, 1, 0, 1);
        scaleAnimation.setDuration(2000);
        //设置布局动画的显示属性
        LayoutAnimationController lac = new LayoutAnimationController(scaleAnimation, 0.5f);
        lac.setOrder(LayoutAnimationController.ORDER_NORMAL);
        //给ViewGroup设置布局动画
        mViewGroup.setLayoutAnimation(lac);
```

- `LayoutAnimationController.ORDER_NORMAL`——顺序
- `LayoutAnimationController.ORDER_RANDOM` ——随机
- `LayoutAnimationController.ORDER_REVERSE`——反序

## Interpolators(插值器)

定义动画的变换速率，类似于物理中的加速度

```java
android:interpolator="@android:anim/accelerate_interpolator" 设置动画为加速动画(动画播放中越来越快)  
  
android:interpolator="@android:anim/decelerate_interpolator" 设置动画为减速动画(动画播放中越来越慢)  
  
android:interpolator="@android:anim/accelerate_decelerate_interpolator" 设置动画为先加速在减速(开始速度最快 逐渐减慢)  
  
android:interpolator="@android:anim/anticipate_interpolator" 先反向执行一段，然后再加速反向回来（相当于我们弹簧，先反向压缩一小段，然后在加速弹出）  
  
android:interpolator="@android:anim/anticipate_overshoot_interpolator" 同上先反向一段，然后加速反向回来，执行完毕自带回弹效果（更形象的弹簧效果）  
  
android:interpolator="@android:anim/bounce_interpolator" 执行完毕之后会回弹跳跃几段（相当于我们高空掉下一颗皮球，到地面是会跳动几下）  
  
android:interpolator="@android:anim/cycle_interpolator" 循环，动画循环一定次数，值的改变为一正弦函数：Math.sin(2* mCycles* Math.PI* input)  
  
android:interpolator="@android:anim/linear_interpolator" 线性均匀改变  
  
android:interpolator="@android:anim/overshoot_interpolator" 加速执行，结束之后回弹 
```

## 自定义动画

继承Animation  重写initialize ()  以及  applyTransformation()方法

```java
 @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
        //设置默认时长
        setDuration(5000);
        //动画结束后保留状态
        setFillAfter(true);
        //设置默认的插值器
        setInterpolator(new BounceInterpolator());

        this.mCenterWidth = width/2;
        this.mCenterHeight = height/2;
    }
```

```java
 @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        super.applyTransformation(interpolatedTime, t);
        this.mCamera = new Camera();

        Matrix matrix = t.getMatrix();
        mCamera.save();
        //使用Camera设置旋转的角度
        mCamera.rotateX(20*interpolatedTime);
        //将旋转变换作用到Metrix上
        mCamera.getMatrix(matrix);
        mCamera.restore();
        //通过pre 方法设置矩阵作用前的的偏移量来改变旋转中心
        matrix.preTranslate(mCenterWidth,mCenterHeight);
        matrix.postTranslate(-mCenterWidth,-mCenterHeight);

    }
```
## SVG      (Scalable Vector Graphics )       5.0

### path

使用<path>标签创建SVG  常用的指令

- L   绘制直线  
- M  移动到某一坐标
- A  绘制弧线

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height = "200dp"
    android:width = "200dp"
    android:viewportHeight = "100"
    android:viewportWidth = "100">

    <group
        android:name="test"
        android:pivotX="36"
        android:pivotY="36"
        android:rotation="0">
        <path
            android:name="v"
            android:fillColor="@color/colorAccent"
            android:pathData="M 25 50 a 25,25 0 1,0 50,0 L 25 80"
            android:strokeColor="@color/colorPrimary"
            android:strokeWidth="2"
            />
    </group>
</vector>
```

### AnimatedVectorDrawable

1. 在res的drawable文件夹下通过<animated-vector>标签来声明对AnimatedVectorDrawable的使用，并指定其作用的path或group.

   ```xml
   <animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:tools="http://schemas.android.com/tools"
       android:drawable="@drawable/yuanhu"
       tools:targetApi="lollipop">

       <target
           android:animation="@anim/rotate_animator"
           android:name="test"/>
   </animated-vector>
   ```

2. 对应的vector即为静态的VectorDrawable.

   注意：target 中name的属性和vector中需要作用的name属性保持一致

3. 通过animation属性，将一个动画作用到了对应name的元素上，objectAnimator的代码如下：

   ```xml
   <set xmlns:android="http://schemas.android.com/apk/res/android">
       <objectAnimator
           android:duration="4000"
           android:propertyName="rotation"
           android:valueFrom="0"
           android:valueTo="360" />
   </set>
   ```

4. 将AnimatrdVectorDrawable XML 文件设置给一个ImageView 作为背景显示

   ```xml
   <ImageView
               android:id="@+id/vectorimage"
               android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:layout_gravity="center_horizontal"
               android:src="@drawable/animator_yuanhu"/>
   ```

5. 在代码中控制SVG动画

   ```java
   mVector = (ImageView) findViewById(R.id.vectorimage);
   ((Animatable) mVector.getDrawable()).start();
   ```

