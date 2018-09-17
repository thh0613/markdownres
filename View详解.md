## View详解

## 1. 基础知识

### 1. view的位置参数

`top`,`left`,`right`,`bottom` 是相对于该`view`的父容器而言的相应位置的距离。

```java
  /**
     * The distance in pixels from the left edge of this view's parent
     * to the left edge of this view.
     * {@hide}
     */
    @ViewDebug.ExportedProperty(category = "layout")
    protected int mLeft;
    /**
     * The distance in pixels from the left edge of this view's parent
     * to the right edge of this view.
     * {@hide}
     */
    @ViewDebug.ExportedProperty(category = "layout")
    protected int mRight;
    /**
     * The distance in pixels from the top edge of this view's parent
     * to the top edge of this view.
     * {@hide}
     */
    @ViewDebug.ExportedProperty(category = "layout")
    protected int mTop;
    /**
     * The distance in pixels from the top edge of this view's parent
     * to the bottom edge of this view.
     * {@hide}
     */
    @ViewDebug.ExportedProperty(category = "layout")
    protected int mBottom;
```

![tmp080ca58e](https://raw.githubusercontent.com/southmoney/markdownres/master/tmp080ca58e.png)

`translationX`表示`view`相对于`left position`移动的距离。 `translationY` 表示`view`相对于`top position`移动的距离。

```java
 /**
     * The horizontal location of this view relative to its {@link #getLeft() left} position.
     * This position is post-layout, in addition to wherever the object's
     * layout placed it.
     *
     * @return The horizontal position of this view relative to its left position, in pixels.
     */
    @ViewDebug.ExportedProperty(category = "drawing")
    public float getTranslationX() {
        return mRenderNode.getTranslationX();
    }

 /**
     * The vertical location of this view relative to its {@link #getTop() top} position.
     * This position is post-layout, in addition to wherever the object's
     * layout placed it.
     *
     * @return The vertical position of this view relative to its top position,
     * in pixels.
     */
    @ViewDebug.ExportedProperty(category = "drawing")
    public float getTranslationY() {
        return mRenderNode.getTranslationY();
    }
```



`

```java
/**
 * The visual x position of this view, in pixels. This is equivalent to the
 * {@link #setTranslationX(float) translationX} property plus the current
 * {@link #getLeft() left} property.
 *
 * @return The visual x position of this view, in pixels.
 */
@ViewDebug.ExportedProperty(category = "drawing")
public float getX() {
    return mLeft + getTranslationX();
}

 /**
     * The visual y position of this view, in pixels. This is equivalent to the
     * {@link #setTranslationY(float) translationY} property plus the current
     * {@link #getTop() top} property.
     *
     * @return The visual y position of this view, in pixels.
     */
    @ViewDebug.ExportedProperty(category = "drawing")
    public float getY() {
        return mTop + getTranslationY();
    }
```

所以一般情况下，`translationX`,`translationY` ＝ 0， 即`getLeft() == getX()`, getTop() == getY()

 ### 2. MotionEvent 和 TouchSlop

`MotionEvent`在手指接触屏幕后所产生的一系列事件中，事件类型：`ACTION_DOWN`,`ACTION_MOVE`,`ACTION_UP`, 通过`MotionEvent`可以得到点击事件发生的`x`和`y`坐标。`getX/getY`返回的是相对于当前`View`左上角的`x`和`y`坐标，而`getRawX`和`getRawY`返回的是相对于手机屏幕左上角的`x`和`y`坐标。

![tmp3518df75](https://raw.githubusercontent.com/southmoney/markdownres/master/tmp3518df75.png)

`TouchSlop` 是系统所能识别的被认为是滑动的最小距离。`ViewConfiguration.get(getContext()).getScaledTouchSlop()`， 在`frameworkds/base/core/res/res/values/config.xml`文件中，这是一个常量，和设备有关，在不同设备上这个值可能是不同的。

```java
    <!-- Base "touch slop" value used by ViewConfiguration as a
         movement threshold where scrolling should begin. -->
    <dimen name="config_viewConfigurationTouchSlop">8dp</dimen>
```



```java
mTestView.post(new Runnable() {
            @Override
            public void run() {
               Log.i("thh ", ViewConfiguration.get(MainActivity.this).getScaledTouchSlop() + " ");
            }
        });
```



### 3. VelocityTracker, GestureDetector  和 Scroller

**VelocityTracker**: 速度追踪，用于追踪手指在滑动过程中的速度。

```java
 @Override
    public boolean onTouchEvent(MotionEvent event) {

        boolean consume = mDetector.onTouchEvent(event);
        VelocityTracker tracker = VelocityTracker.obtain();
        tracker.addMovement(event);
        tracker.computeCurrentVelocity(1000);
        Log.i("thh tracker", tracker.getXVelocity() + " " + tracker.getYVelocity());
        return consume;
    }
```



- 在`ConstraintLayout`中使用没有速度。且朝`view`的坐标轴的方向速度为正。

**GestureDetector ：**手势检测，用于辅助检测用户的单机、滑动、长按、双击等行为。

```java
 @Override
    public boolean onTouchEvent(MotionEvent event) {

        boolean consume = mDetector.onTouchEvent(event);
        return consume;
    }

```



只适合用于辅助，对于监听滑动，还是在`onTouchEvent`自行监听。如果用于双击，可以使用`GestureDetector`

**scroller:** 弹性滑动对象，用于实现`View`的弹性滑动。

```
  mTestView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mTestView.smoothScrollTo(0, -100);
            }
        });
```

注意，`scroller`只能改变`View`内容的位置，而不改变`View`在布局中的位置，且在Y轴：正为上，负为下。X轴： 正为左，负为右。因为`mScrollX = View.getLeft() - View.Content.getLeft()`

## 2. 滑动

### 1. 使用`scrollTo/scrollBy`

这两个方法只能改变`View`内容的位置，而不改变`View`在布局中的位置。

`mScrollX:` 等于`View`左边缘和`View`内容左边缘在水平方向的距离

`mScrollY:` 等于`View`上边缘和`view`内容上边缘在竖直方向的距离。

```
  private void init() {
        mScroller = new Scroller(getContext());
    }

    public void smoothScrollTo(int desX, int desY) {
        int scrollX = getScrollX();
        int delta = desX - scrollX;
        int scrollY = getScrollY();
        int deltaY = desY - scrollY;
        mScroller.startScroll(scrollX, scrollY, delta, deltaY, 1000);
        invalidate();
    }

    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }
```

(1). `mScroller.startScroll(int startX, int startY, int deltaX, int deltaY, int duration)` 这个方法并没有真正执行滑动，只是将`mStartX,mStartY, mFinalX, mFinalY ,mDuration`赋值了。

(2). `invalidate` 方法会导致调用`computeScroll()`方法，通过调用`computeScrollOffset`来判断动画有没有结束，如果没有结束，则继续`invalidate`。 真正滑动的是`ScrollTo`。 `postInvalidate`是非UI线程，`invalidate`是UI线程。

### 2. 使用动画

使用动画来移动`view`， 主要是操作`View`的`translationX`和`translationY`属性。

经过测试发现：

- 值动画是对`View`的影像做操作，并不能改变`View`的位置参数，包括宽高。而属性动画则不会。
- 设置`View`的`translationX,tranlationY`, 点击会跟着`view`走。`getX,getY`会变化。

### 3. 改变布局参数

改变布局参数，即改变`layoutParams`



## 3. 弹性滑动

将一次大的滑动分成若干次小的滑动并在一个时间段内完成。所以可以分成3种方法：

### 3.1 使用`scroller`（见上述demo）

### 3.2 通过动画（ValueAnimator）

```java
public void smoothScrollAnim(final int desX, int desY) {
        ValueAnimator animator = ValueAnimator.ofInt(0,1).setDuration(1000);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float fraction = animation.getAnimatedFraction();
                scrollTo((int) (mStartX + (desX - mStartX) * fraction), 0);
            }
        });
        animator.start();
    }
```

项目可经常使用`ValueAnimator`来实现动画。

### 3.3 使用延时策略（handler）

```
 private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:{
                    mCount++;
                    if (mCount < FRAME_COUNT) {
                        float fraction = (float) (1.0 * mCount / FRAME_COUNT);
                        int scrollX = (int) (fraction * 100);
                        scrollTo(-scrollX,0);
                        mHandler.sendEmptyMessageDelayed(1,100);
                    }
                    break;
                }
            }
        }
    };
```



使用`handler`的一个缺点就是无法做到流畅，不知道每隔多久发送一个消息。

## 4. View 的事件分发机制

### 4.1 点击事件的传递规则

> 点击事件的事件分发，其实就是对`MotionEvent`事件的分发过程，即当一个`MotionEvent`产生了以后，系统需要把这个事件传递给具体的一个`View`， 由3个很重要的方法共同完成：`dispatchTouchEvent`,`onInterceptTouchEvent`,`onTouchEvent`

**`public boolean dispatchTouchEvent(MotionEvent ev)`**

用于事件的分发，如果事件能够传递给当前`View`, 那么此方法一定会被调用，返回结果受当前`view.onTouchEvent`和下级`view.dispatchTouchEvent`影响，表示是否消耗当前事件。



**`public boolean onInterceptTouchEvent(MotionEvent ev)`**

判断是否拦截某个事件，如果当前`view`拦截了某个事件，那么在同一个事件序列当中，此方法不会再次调用。表示是否拦截当前事件。



**`public boolean onTouchEvent(MotionEvent ev)`**

在`dispatchTouchEvent`方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一事件序列中，当前`view`无法再次接受到事件。



**分发规则：**

1. 当一个点击事件发生后，传递顺序为`Activity` -> `Window` -> `View`.  即事件总是先传递给`Activity`， `Activity` 再传递给`Window`, 最后`Window`再传递给 `View`, 顶级`View`接到事件后，就会按照事件分发机制去分发事件。如果一个`View`的`onTouchEvent` 返回`false`，那么它的父容器的`onTouchEvent`将会被调用。直到这个事件传递给`Activity`.

2. 对于根`ViewGroup`， 它的`dispatchTouchEvent`就会被调用，如果这个`view`的`onInterCeptTouchEvent`方法返回`true`表示拦截当前事件，接着交给这个`ViewGroup.onTouchEvent`处理。如果返回`false`, 则表示不拦截当前事件，就会继续传递给它的子元素。

3. 当一个`view`需要处理事件时，如果设置了`onTouchListener`,那么`onTouchListener`的`onTouch`方法会被调用，如果返回false, 那么当前`view.onTouchEvent`方法会被调用，如果返回true，那么`onTouchEvent`方法不会调用了。在`onTouchEvent`方法中，如果设置了`onClickListener`，那么`onClick`方法会被调用。所以优先级为`onTouch` ->  `onTouchEvent` -> `onClick`

    

   `Activity#dispatchTouchEvent`

   ```java
    /**
        * Called to process touch screen events.  You can override this to
        * intercept all touch screen events before they are dispatched to the
        * window.  Be sure to call this implementation for touch screen events
        * that should be handled normally.
        *
        * @param ev The touch screen event.
        *
        * @return boolean Return true if this event was consumed.
        */
       public boolean dispatchTouchEvent(MotionEvent ev) {
           if (ev.getAction() == MotionEvent.ACTION_DOWN) {
               onUserInteraction();
           }
           if (getWindow().superDispatchTouchEvent(ev)) {
               return true;
           }
           return onTouchEvent(ev);
       }
   ```

   `PhoneWindow#dispatchTouchEvent`

   ```
    @Override
       public boolean superDispatchTouchEvent(MotionEvent event) {
           return mDecor.superDispatchTouchEvent(event);
       }
   
   ```

   

4. 如果子`view`在`onTouchEvent`中没有返回`true`, 那么父的`viewGroup`的`onTouchEvent`仍然会执行。

5. 如果在`viewGroup`拦截了`ACTION_DOWN`, 即`ViewGroup.onInterceptTouchEvent` 返回true，那么`ViewGroup.onTouchEvent`就会执行。那么剩下所有的事件，都由`viewGroup`接收。 如果只是拦截了`ACTION_MOVE`或者`ACTION_UP`, 则其他事件仍继续往下分发。

   ```java
   
               // Check for interception.
               final boolean intercepted;
               if (actionMasked == MotionEvent.ACTION_DOWN
                       || mFirstTouchTarget != null) {
                   final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                   if (!disallowIntercept) {
                       intercepted = onInterceptTouchEvent(ev);
                       ev.setAction(action); // restore action in case it was changed
                   } else {
                       intercepted = false;
                   }
               } else {
                   // There are no touch targets and this action is not an initial down
                   // so this view group continues to intercept touches.
                   intercepted = true;
               }
   ```

6. 点击事件时`MotionDown` 和`MotionUP`组合而成，如果在任意一个事件中没有`super.onTouchEvent`则点击事件消失。

7. `ACTION_UP`在父容器中必须返回`false`,因为如果父容器没有拦截任何事件，那么返回`true`,将导致子元素收不到`onClick`事件，如果父容器拦截了事件，那么后续的事件都会交给他，即便父容器的`ACTION_UP`返回`false`

8. `View`的`onTouchEvent`默认都会消耗事件（返回true），除非是不可点击的（`clickable,longclickable`同时为false), ｀Button｀的`clickable`属性默认为`true`,而`TextView`的`clickable`属性默认为`false`, 所以如果是子元素为`Button`， 默认是消耗事件，即返回`true`, 父容器的`onTouchEvent`不会被执行。而如果是子元素为`TextView`,那么不会消耗事件，父容器的`onTouchEvent`就会被执行。

## 5. View的滑动冲突

### 5.1 滑动冲突场景

（1）外部滑动方向和内部滑动方向不一致

​	如`ScollView` 和 `ListView`。

（2）外部滑动方向和内部滑动方向一致

  如 内外同时能上下滑动或左右滑动。

（3）上述的嵌套。

### 5.2 滑动冲突的处理规则

（1）根据滑动方向来处理： 当用户左右滑动时，需要让外部的`view`拦截点击事件，当用户上下滑动时，需要让内部`View`拦截点击事件。

（2）根据业务逻辑来处理：当处于某种状态时需要外部`View`响应用户的滑动，而处于另外一种状态时则需要内部`View`来响应`View`的滑动。

（3）同（2），还是从业务逻辑来判断。

### 5.3 滑动冲突的解决方式

（1）外部拦截法：点击事件先经过父容器的拦截处理，如果父容器需要此事件就拦截，不需要就不拦截。即父容器重写`onInterceptTouchEvent`方法。

```java
 @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        int x = (int) ev.getX();
        int y = (int) ev.getY();

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastXIntercept;
                int deltaY = y - mLastYIntercept;
                if (Math.abs(deltaX) > Math.abs(deltaY)) {
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            default:
                break;
        }
        
        return intercepted;

    }
```

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    mVelocityTracker.addMovement(event);
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch(event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            break;
        case MotionEvent.ACTION_MOVE:
            int deltaX = x - mLastX;
            int deltaY = y - mLastY;
            scrollBy(-deltaX, 0);
            break;
        case MotionEvent.ACTION_UP:
            //TODO 逻辑
            ...
            break;
    }
    mLastX = x;
    mLastX = y;
    return true;
}
```

（2）内部拦截法：指父容器不拦截任何事件，所以的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交由父容器进行处理。

```java
 /**
     * Called when a child does not want this parent and its ancestors to
     * intercept touch events with
     * {@link ViewGroup#onInterceptTouchEvent(MotionEvent)}.
     *
     * <p>This parent should pass this call onto its parents. This parent must obey
     * this request for the duration of the touch (that is, only clear the flag
     * after this parent has received an up or a cancel.</p>
     * 
     * @param disallowIntercept True if the child does not want the parent to
     *            intercept touch events.
     */
    public void requestDisallowInterceptTouchEvent(boolean disallowIntercept);
```

子元素：

```java
@Override
public boolean dispatchTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            getParent().requestDisallowInterceptTouchEvent(true);
            break;
        case MotionEvent.ACTION_MOVE:
            if (Math.abs(deltaX) > Math.abs(deltaY)) {
            	getParent().requestDisallowInterceptTouchEvent(false);
                ｝
            break;
        case MotionEvent.ACTION_UP:
            break;
    }
    return super.dispatchTouchEvent(event);
}
```

父容器：

```java
 @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
       int action = ev.getAction();
       if (action == MotionEvent.ACTION_DOWN) {
           return false;
       } else {
           return true;
       }
    }
```



## 6. View 的工作原理

### 6.1 初识`ViewRoot`和`DecorView`

`View`的绘制流程是从`ViewRoot`的｀performTraversals｀方法开始的。经过`measure`,`layout`和`draw`三个过程才能最终将一个`View`绘制出来。

`measure`用来测量`view`的宽和高，`layout`用来确定`view`在父容器中的放置位置，`draw`负责将`view`绘制在屏幕上。

`measure`过程决定了`view`的宽／高，`Measure`完成之后，可以通过`getMeasuredWidth`和`getMeasuredHeight` 方法来获取到`View`的宽／高，几乎所有的情况下，都等于`View`最终的宽／高。

`layout` 过程决定了`view`的四个定点的坐标和实际的`view`的宽／高，完成以后，可以通过`getTop,getBottom,getLeft,getRight`来拿到`View`四个顶点的位置。并可以通过`getWidth`和`getHeight`来拿刀`View`的最终宽高。

`draw` 过程决定了`view`的显示，只有`draw`方法完成以后`View`的内容才能显示在屏幕上。

###6.2 理解MeasureSpec

在测量过程中，系统会将`View`的`LayoutParams`根据父容器所施加的规则转换成对应的`MeasureSpec`，然后再根据这个`measureSpec`来测量出`view`的宽／高。

#### 6.2.1 MeasureSpec

`MeasureSpec`代表一个32位`int`值，高2位代表`specMode`,指测量模式 低30位代表`SpecSize`

```java
    public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        public static final int EXACTLY     = 1 << MODE_SHIFT;
        public static final int AT_MOST     = 2 << MODE_SHIFT;
		public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size, @MeasureSpecMode int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }
        
        @MeasureSpecMode
        public static int getMode(int measureSpec) {
            //noinspection ResourceType
            return (measureSpec & MODE_MASK);
        }

        /**
         * Extracts the size from the supplied measure specification.
         *
         * @param measureSpec the measure specification to extract the size from
         * @return the size in pixels defined in the supplied measure specification
         */
        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

```

`MeasureSpec`装包和解包过程。

三类`SpecMode`:

`UNSPECIFIED:`

> The parent has not imposed any constraint on the child. It can be whatever size it wants.

父容器不对`View`任何限制，要多大有多大。用于系统内部，表示一种测量的状态。

`EXACTLY`:

> The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.

已经检测出`View`所需要的精确大小，这个时候`view`的最终大小就是`specSize`所指定的值，对应于具体的数值和`match_parent`。

`AT_MOST:`

> The child can be as large as it wants up to the specified size.

父容器指定了一个可用大小`specSize`，`View`可以任意大，但是不能大过这个`specSize`

#### 6.2.2 MeasureSpec 和 LayoutParams的对应关系

`MeasureSpec`一旦确定，那么`onMeasure`方法就可以确定`view`的宽高了，通常我们通过设置`view`自身的`layoutparams`， 系统会将`layoutParams`在父容器的约束下转换成对应的`MeasureSpec`. 也就是说，`LayoutParams`需要和父容器一起才能决定`View`的`measureSpec`. 

对于顶级的`view`(即`DecorView`)：其`MeasureSpec`由窗口的尺寸和其自身的`LayoutParams`来共同确定。

```java
 childWidthMeasureSpec = getRootMeasureSpec(baseSize, lp.width);
 childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
 performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
 

    /**
     * Figures out the measure spec for the root view in a window based on it's
     * layout params.
     *
     * @param windowSize
     *            The available width or height of the window
     *
     * @param rootDimension
     *            The layout params for one dimension (width or height) of the
     *            window.
     *
     * @return The measure spec to use to measure the root view.
     */
  private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }

```



对于普通的`view`: 由父容器的`MeasureSpec`和自身的`LayoutParams`来决定。

```java
 /**
     * Does the hard part of measureChildren: figuring out the MeasureSpec to
     * pass to a particular child. This method figures out the right MeasureSpec
     * for one dimension (height or width) of one child view.
     *
     * The goal is to combine information from our MeasureSpec with the
     * LayoutParams of the child to get the best possible results. For example,
     * if the this view knows its size (because its MeasureSpec has a mode of
     * EXACTLY), and the child has indicated in its LayoutParams that it wants
     * to be the same size as the parent, the parent should ask the child to
     * layout given an exact size.
     *
     * @param spec The requirements for this view
     * @param padding The padding of this view for the current dimension and
     *        margins, if applicable
     * @param childDimension How big the child wants to be in the current
     *        dimension
     * @return a MeasureSpec integer for the child
     */
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

通过上述的方法，很显然看出子元素的`measureSpec`的创建和父容器的`MeasureSpec`和子元素的`LayoutParams`有关。此处的`padding`指父容器已占用的空间的大小。

总结：

（1）当`View`采用固定宽／高的时候，不管父容器的`MeasureSpec`是什么，`View`的`MeasureSpec`都是精确模式并且其大小遵循`LayoutParams`中的大小。

（2）当`view`的宽／高为`match_parent`时，如果父容器是精准模式，那么`View`也是精准模式，且大小是父容器的剩余大小。如果父容器是最大模式，那么`view`也是最大模式，且大小不会超过父容器的剩余空间。

（3）当`View`的宽／高是`wrap_content`时，不管父容器的模式是精准还是最大化，`view`的模式总是最大化，且大小不超过父容器的剩余空间。

### 6.3 `View`的工作流程

#### 6.3.1 Measure

`view`的`measure`方法会调用`onMeasure()`方法

```java
 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec), getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
        }

```

```java
  public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```



从`getDefaultSize`方法可知，`view`的宽高由`specSize`决定，直接继承`view`的自定义需要重写`onMeasure`方法并设置`wrap_content`时的自身大小，否则就相当于`match_parent`了。所以像`TextView`都有重写。

```java
  @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int defaultWidth = 100;
        int defaultHeight = 100;
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);

        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(defaultWidth, defaultHeight);
        } else if (widthMeasureSpec == MeasureSpec.AT_MOST) {
            setMeasuredDimension(defaultWidth, widthSpecSize);
        } else if (heightMeasureSpec == MeasureSpec.AT_MOST) {
            setMeasuredDimension(heightSpecSize, defaultHeight);
        }
    }
```



`ViewGroup`的`measure`过程

`ViewGroup`除了完成自己的`measure`过程以外，还会遍历调用所有子元素的`measure`方法，各个子元素再递归地去执行这个过程。没有重写`view`的`onMeasure`方法，而是提供了`measureChildren`方法。

```java
/**
     * Ask all of the children of this view to measure themselves, taking into
     * account both the MeasureSpec requirements for this view and its padding.
     * We skip children that are in the GONE state The heavy lifting is done in
     * getChildMeasureSpec.
     *
     * @param widthMeasureSpec The width requirements for this view
     * @param heightMeasureSpec The height requirements for this view
     */
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
```

```java
/**
     * Ask one of the children of this view to measure itself, taking into
     * account both the MeasureSpec requirements for this view and its padding.
     * The heavy lifting is done in getChildMeasureSpec.
     *
     * @param child The child to measure
     * @param parentWidthMeasureSpec The width requirements for this view
     * @param parentHeightMeasureSpec The height requirements for this view
     */
    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

```



`ViewGroup`是一个抽象类，并没有提供`onMeasure`方法。需要其他各个类自己去实现。以`LinearLayout`为例，

```java
 @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }

```

#### 6.3.2`layout`过程

`layout`的作用是`viewGroup`用来确定子元素的位置，当`Viewgroup`的位置被确定后，它在`onLayout`中会遍历所有的子元素并调用其`layout`方法，在`layout`方法中`onLayout`又会被调用。`layout`方法确定`view`本身位置，而`onLayout`则会确定所有子元素的位置。

流程：

(1) 首先通过`setFrame`来设定自己四个顶点的位置，那么`view`在父容器中的位置就确定了

(2) 接着调用`onLayout`，让父容器确定子元素的位置，`onLayout`的具体实现和具体布局有关，所以这个没有实现。

(3) 看`LinearLayout`的`onLayout`方法，分成了两类。

```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l, t, r, b);
    }
}
```

(4) 看`layoutVertical`发现，遍历所有的`child`， 对`child`设置`setChildFrame`， 从而确定每个`child`的位置。

(5) 通过`onLayout`和`setChildFrame`,可以发现一般情况下，`getMeasuredWidth`和`getWidth`值是一样的，`getMeasuredHeight`和`getHeight`值是一样的。

```java
final int childWidth = child.getMeasuredWidth();
final int childHeight = child.getMeasuredHeight();
setChildFrame(child, childLeft, childTop + getLocationOffset(child),childWidth, childHeight);

private void setChildFrame(View child, int left, int top, int width, int height) {
    child.layout(left, top, left + width, top + height);
}

@ViewDebug.ExportedProperty(category = "layout")
public final int getWidth() {
     return mRight - mLeft;
}

@ViewDebug.ExportedProperty(category = "layout")
public final int getHeight() {
    return mBottom - mTop;
}
```

(6) 但是可以通过`layout`方法来改变宽高。

```java
public void layout(int l, int t, int r, int b) {
    super.layout(l, t, r + 100, b+100);
}
```



####6.3.3 draw 过程



> ```
> /*
>  * Draw traversal performs several drawing steps which must be executed
>  * in the appropriate order:
>  *
>  *      1. Draw the background
>  *      2. If necessary, save the canvas' layers to prepare for fading
>  *      3. Draw view's content
>  *      4. Draw children
>  *      5. If necessary, draw the fading edges and restore layers
>  *      6. Draw decorations (scrollbars for instance)
>  */
> ```

## 7. View 官方文档

**`onFinishInflate()`**

> called after a view and all of its children has been inflated from XML

该方法当`view`及其子`view`从XML文件中加载完成后被调用。但此时`onMeasure`还没有调用，此时获取的宽高还是0.

**`onMeasure(int, int)`**

> Called to determine the size requirements for this view and all of its children.

该方法在计算当前`View`及其所有子`View`尺寸大小需求时会被调用

**`onLayout(boolean, int, int, int, int)`**

> Called when this view should assign a size and position to all of its children.

该方法在当前`View`需要为其子View分配尺寸和确定位置时会被调用

**`onSizeChanged(int, int, int, int)`**

> Called when the size of this view has changed.

发生在`onLayout`方法里。当大小发生变化时会调用。

**`onDraw(canvas)`**

> Called when the size of this view has changed.

需要绘制内容时被调用。

**`onKeyDown(int, KeyEvent)`**

>Called when a new hardware key event occurs.

用来捕捉手机键盘被按下的事件

**`onKeyUp(int, keyEvent)`**

>Called when a hardware key up event occurs.

用来捕捉手机键盘按下抬起时的事件。

**`onTrackballEvent(MotionEvent)`**

> Called when a trackball motion event occurs.

手机中的轨迹球事件

**`onTouchEvent`**

> called when a touch screen motion event occurs

触摸事件发生时触发。

**`onFocusChanged(boolean, int, Rect)`**

> called when the views gains or loses focus

当获取或失去焦点时触发。

一些关于焦点的方法：

>setFocusable方法：设置View是否可以拥有焦点。
>
>isFocusable方法：监测此View是否可以拥有焦点。
>
>setNextFocusDownId方法：设置View的焦点向下移动后获得焦点View的ID。
>
>hasFocus方法：返回了View的父控件是否获得了焦点。
>
>requestFocus方法：尝试让此View获得焦点。
>
>isFocusableTouchMode方法：设置View是否可以在触摸模式下获得焦点，在默认情况下是不可用获得的。

**`onWindowFocusChanged(boolean)`**

> Called when the window containing the view gains or loses focus.

该方法在包含当前View的window获得或失去焦点时被调用。真正的visible时间点是onWindowFocusChanged()函数被执行时. 另外`view`已经初始化完毕，宽／高已经准备好了，这个时候可以去获取宽／高，另外，`onWindowFocusChanged` 会被调用多次，当`Activity`的窗口得到焦点和失去焦点时，均会被调用一次。

**`onAttachedToWindow()`**

> called when the view is attached to the window.
>
> This is called when the view is attached to a window. At this point it has a Surface and will start drawing. Note that this function is guaranteed to be called before`onDraw(android.graphics.Canvas)`, however it may be called any time before the first onDraw -- including before or after `onMeasure(int, int)`

该方法在当前View被附到一个window上时被调用。

**`onDetachedToWindow()`**

> Called when the view is detached from its window.

该方法在当前`view`从一个`window`上分离时调用

**`onWindowVisibilityChanged()`**

> Called when the visibility of the window containing the view has changed.

该方法在包含当前View的window可见性改变时被调用。

>1. To initiate a layout, call `requestLayout()`. This method is typically called by a view on itself when it believes that is can no longer fit within its current bounds.
>2. To force a view to draw, call `invalidate()`.If the view is visible, `onDraw(android.graphics.Canvas)` will be called at some point in the future. This must be called from a UI thread. To call from a non-UI thread, call `postInvalidate()`.
>3. To get a particular view to take focus, call `requestFocus()`.

![1734948-b4493f7b0234dd69](https://raw.githubusercontent.com/southmoney/markdownres/master/1734948-b4493f7b0234dd69.jpg)

## 8. View的生命周期

### 8.1 创建过程

(![tmp46033ac7](https://raw.githubusercontent.com/southmoney/markdownres/master/tmp42c35154.png))

从图中可以发现：`onFinishInflate` => `onAttachedToWindow` => `onWindowVisibilityChanged` => `onMeasure` => `onSizeChanged` => `onLayout` => `onDraw` => `onWindowFocusChanged`

1. `onFinishInflate` 完成后，并不能获取该`view`的宽／高，因为`onMeasure`还没走完。
2. `onSizedChanged` 发生在`onLayout`前面
3. `onWindowFocusChanged` 可以获取宽／高。
4. 说明在`Activity`中的`onResume`方法中不一定能获取到宽高。

### 8.2 熄灭屏幕

![https://raw.githubusercontent.com/southmoney/markdownres/master/tmp46033ac7.png](https://raw.githubusercontent.com/southmoney/markdownres/master/tmp46033ac7.png)



`onWindowVisibilityChanged` 从可见变成不可见。窗口失去焦点，可以发现，当获取焦点或失去焦点，`onWindowFocusChanged`都会被调用。

### 8.3 从后台回来

![https://raw.githubusercontent.com/southmoney/markdownres/master/tmp3bc4a6ad.png](https://raw.githubusercontent.com/southmoney/markdownres/master/tmp3bc4a6ad.png)

竟然发现调用了`onDraw`方法。

### 8.4 退出应用

![https://raw.githubusercontent.com/southmoney/markdownres/master/tmp7e5eb7fe.png](https://raw.githubusercontent.com/southmoney/markdownres/master/tmp7e5eb7fe.png)

最后调用了`Activity`的`onDestroy` 和`view`的`onDetachedFromWindow`





