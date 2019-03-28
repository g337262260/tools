# Material Design 动画效果

## Ripple效果

在Android 5.X中，Material Design 大量使用了Ripple效果，就是点击后的波纹效果。

`android:background = @android:attr/selectableItemBackground       //波纹有边界`

`android:background = @android:attr/selectableItemBackgroundBordless    //波纹超出边界`

可以在XML文件中直接创建一个具有Ripple效果的XML文件

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android" android:color="@color/colorPrimaryDark">
    <item >
        <shape android:shape="oval">
            <solid android:color="@color/colorAccent"/>
        </shape>
    </item>
</ripple>
```

```xml
<Button
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:text="Hello World!"
        android:background="@drawable/button_ripple" />
```

## Circular Reveal

具体表现为一个View以圆形的 形式展开、揭示出来，主要是通过ViewAnimationUtils.createCircularReveal()方法可以创建一个RevealAnimator动画

```java

   	/* @param view The View will be clipped to the animating circle.
     * @param centerX The x coordinate of the center of the animating circle, relative to
     *                <code>view</code>.
     * @param centerY The y coordinate of the center of the animating circle, relative to
     *                <code>view</code>.
     * @param startRadius The starting radius of the animating circle.
     * @param endRadius The ending radius of the animating circle.
     */
    public static Animator createCircularReveal(View view,
            int centerX,  int centerY, float startRadius, float endRadius) {
        return new RevealAnimator(view, centerX, centerY, startRadius, endRadius);
    }
```

```java
 /**
 * CircularReveal(中心)
 */
 private void  ovalAnimation(){
        Animator animator = ViewAnimationUtils.createCircularReveal(mOval, mOval.getWidth() / 2, mOval.getHeight() / 2, mOval.getWidth(), mOval.getWidth()/3);
        animator.setInterpolator(new AccelerateDecelerateInterpolator());
        animator.setDuration(2000);
        animator.start();

    }
```

```java
    /**
     * CircularReveal(边界)
     */
    private void rectAnimation(){
        Animator animator = ViewAnimationUtils.createCircularReveal(mRect, 0, 0, 0, ((float) Math.hypot(mRect.getWidth(), mRect.getHeight())));
        animator.setInterpolator(new AccelerateDecelerateInterpolator());
        animator.setDuration(2000);
        animator.start();
    }
```

