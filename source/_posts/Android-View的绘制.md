---
title: Android View的绘制
date: 2016-05-30 20:20:47
tags: View
categories: View
toc: true 
---
- .View的绘制流程分几步，从哪开始？哪个过程结束以后能看到view？

 ```
 从ViewRoot的performTraversals开始，经过measure，layout,draw 三个流程。draw流程结束以后就可以在屏幕上看到view了。
 
 ```
 
 - view的测量宽高和实际宽高有区别吗？
 
 ```
 基本上百分之99的情况下都是可以认为没有区别的。有两种情况，有区别。第一种 就是有的时候会因为某些原因 view会多次测量，那第一次测量的宽高 肯定和最后实际的宽高 是不一定相等的，但是在这种情况下最后一次测量的宽高和实际宽高是一致的。此外，实际宽高是在layout流程里确定的，我们可以在layout流程里 将实际宽高写死 写成硬编码，这样测量的宽高和实际宽高就肯定不一样了，虽然这么做没有意义 而且也不好。
 
 ```
<!-- more-->
 - view的measureSpec 由谁决定?顶级view呢？
 
```
由view自己的layoutparams和父容器  一起决定自己的measureSpec。一旦确定了spec，onMeasure中就可以确定view的宽高了。

顶级view就稍微特殊一点，对于decorView的测量在ViewRootImpl的源码里。

/desire的这2个参数就代表屏幕的宽高，
  childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
  childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
  performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

  //decorView的measureSpec就是在这里确定的，其实比普通view的measurespec要简单的多
  //代码就不分析了 一目了然的东西
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

- 对于普通view来说，他的measure过程中，与父view有关吗？如果有关，这个父view也就是viewgroup扮演了什么角色？

```
//对于普通view的measure来说 是由这个view的 父view ，也就是viewgroup来触发的。
//也就是下面这个measureChildWithMargins方法

protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
         //第一步 先取得子view的 layoutParams 参数值   
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        //然后开始计算子view的spec的值，注意这里看到 计算的时候除了要用子view的 layoutparams参数以外
        //还用到了父view 也就是viewgroup自己的spec的值
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

//这个算view的spec的方法 看上去一大串 但是真的逻辑非常简单 就是根据父亲viewgroup
//的meaurespec 同时还有view自己的params来确定 view自己的measureSpec。
//注意这里的参数是padding,这个值的含义是 父容器已占用的控件的大小 所以view的Specsize
//的值 你们可以看到 是要减去这个padding的值的。总大小-已经用的 =可用的。 很好理解。

//然后就是下面的switch逻辑 要自己梳理清楚。其实也不难，主要是下面几条原则
//如果view采用固定宽高，也就是写死的数值那种。那就不管父亲的spec的值了，view的spec 就肯定是exactly 并且大小遵循layout参数里设置的大小。

//如果view的宽高是match_parent ，那么就要看父容器viewgroup的 spec的值了，如果父view的spec是exactly模式，
//那view也肯定是exactly,并且大小就是父容器剩下的空间。如果父容器是at_most模式，那view也是at_most 并且不会超过剩余空间大小

//如果view的宽高是wrap_content, 那就不管父容器的spec了，view的spec一定是at_most 并且不会超过父view 剩余空间的大小。

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
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
    
```

- .view的meaure和onMeasure有什么关系？

```
//view的measure是final 方法 我们子类无法修改的。
 public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        boolean optical = isLayoutModeOptical(this);
        if (optical != isLayoutModeOptical(mParent)) {
            Insets insets = getOpticalInsets();
            int oWidth  = insets.left + insets.right;
            int oHeight = insets.top  + insets.bottom;
            widthMeasureSpec  = MeasureSpec.adjust(widthMeasureSpec,  optical ? -oWidth  : oWidth);
            heightMeasureSpec = MeasureSpec.adjust(heightMeasureSpec, optical ? -oHeight : oHeight);
        }

        // Suppress sign extension for the low bytes
        long key = (long) widthMeasureSpec << 32 | (long) heightMeasureSpec & 0xffffffffL;
        if (mMeasureCache == null) mMeasureCache = new LongSparseLongArray(2);

        if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
                widthMeasureSpec != mOldWidthMeasureSpec ||
                heightMeasureSpec != mOldHeightMeasureSpec) {

            // first clears the measured dimension flag
            mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

            resolveRtlPropertiesIfNeeded();

            int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
                    mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // measure ourselves, this should set the measured dimension flag back
                onMeasure(widthMeasureSpec, heightMeasureSpec);
                mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                long value = mMeasureCache.valueAt(cacheIndex);
                // Casting a long to int drops the high 32 bits, no mask needed
                setMeasuredDimensionRaw((int) (value >> 32), (int) value);
                mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            }

            // flag not set, setMeasuredDimension() was not invoked, we raise
            // an exception to warn the developer
            if ((mPrivateFlags & PFLAG_MEASURED_DIMENSION_SET) != PFLAG_MEASURED_DIMENSION_SET) {
                throw new IllegalStateException("View with id " + getId() + ": "
                        + getClass().getName() + "#onMeasure() did not set the"
                        + " measured dimension by calling"
                        + " setMeasuredDimension()");
            }

            mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
        }

        mOldWidthMeasureSpec = widthMeasureSpec;
        mOldHeightMeasureSpec = heightMeasureSpec;

        mMeasureCache.put(key, ((long) mMeasuredWidth) << 32 |
                (long) mMeasuredHeight & 0xffffffffL); // suppress sign extension
    }

//不过可以看到的是在measure方法里调用了onMeasure方法
//所以就能知道 我们在自定义view的时候一定是重写这个方法！
 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
 
 ```
 - 简要分析view的measure流程？
 
```
viewgroup 算出子view的spec以后 会调用子view的measure方法，而子view的measure方法 我们问题5也看过了实际上是调用的onMeasure方法。

所以我们只要分析好onMeasure方法即可，注意onMeasure方法的参数 正是他的父view算出来的那2个spec的值(这里view的measure方法会把这个spec里的specSize值做略微的修改 这个部分 不做分析 因为measure方法修改specSize的部分很简单)。

//可以看出来这个就是setMeasuredDimension方法的调用 这个方法看名字就知道就是确定view的测量宽高的
//所以我们分析的重点就是看这个getDefaultSize 方法 是怎么确定view的测量宽高的
 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }


//这个方法特别简单 基本可以认为就是近似的返回spec中的specSize，除非你的specMode是UNSPECIFIED
//UNSPECIFIED 这个一般都是系统内部测量才用的到，这种时候返回size 也就是getSuggestedMinimumWidth的返回值
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

//跟view的背景相关 这里不多做分析了
protected int getSuggestedMinimumWidth() {
        return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
    }
 
 ```
 - .自定义view中 如果onMeasure方法 没有对wrap_content 做处理 会发生什么？为什么？怎么解决？
 
```
 如果没有对wrap_content做处理 ，那即使你在xml里设置为wrap_content.其效果也和match_parent相同。看问题4的分析。我们可以知道view自己的layout为wrap，那mode就是at_most（不管父亲view是什么specmode）.

这种模式下宽高就是等于specSize(getDefaultSize函数分析可知)，而这里的specSize显然就是parentSize的大小。也就是父容器剩余的大小。那不就和我们直接设置成match_parent是一样的效果了么？

解决方式就是在onMeasure里 针对wrap 来做特殊处理 比如指定一个默认的宽高，当发现是wrap_content 就设置这个默认宽高即可。

```
- 为什么在activity的生命周期里无法获得测量宽高？有什么方法可以解决这个问题吗？

```
因为measure的过程和activity的生命周期  没有任何关系。你无法确定在哪个生命周期执行完毕以后 view的measure过程一定走完。可以尝试如下几种方法 获取view的测量宽高。

//重写activity的这个方法
public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if (hasFocus) {
            int width = tv.getMeasuredWidth();
            int height = tv.getMeasuredHeight();
            Log.v("burning", "width==" + width);
            Log.v("burning", "height==" + height);

        }
    }

或者重写这个方法

@Override
    protected void onStart() {
        super.onStart();
        tv.post(new Runnable() {
            @Override
            public void run() {
                int width = tv.getMeasuredWidth();
                int height = tv.getMeasuredHeight();
            }
        });
    }

再或者：

@Override
    protected void onStart() {
        super.onStart();
        ViewTreeObserver observer = tv.getViewTreeObserver();
        observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                int width = tv.getMeasuredWidth();
                int height = tv.getMeasuredHeight();
                tv.getViewTreeObserver().removeOnGlobalLayoutListener(this);
            }
        });
    }

```
- layout和onLayout方法有什么区别？

```
layout是确定本身view的位置 而onLayout是确定所有子元素的位置。layout里面 就是通过serFrame方法设设定本身view的 四个顶点的位置。这4个位置以确定 自己view的位置就固定了。

然后就调用onLayout来确定子元素的位置。view和viewgroup的onlayout方法都没有写。都留给我们自己给子元素布局

```

- draw方法 大概有几个步骤？

``` 
一共是4个步骤， 绘制背景---------绘制自己--------绘制chrildren----绘制装饰。

```
- setWillNotDraw方法有什么用？

```
这个方法在view里。

/**
     * If this view doesn't do any drawing on its own, set this flag to
     * allow further optimizations. By default, this flag is not set on
     * View, but could be set on some View subclasses such as ViewGroup.
     *
     * Typically, if you override {@link #onDraw(android.graphics.Canvas)}
     * you should clear this flag.
     *
     * @param willNotDraw whether or not this View draw on its own
     */
    public void setWillNotDraw(boolean willNotDraw) {
        setFlags(willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);
    }

用于设置标志位的 也就是说 如果你的自定义view 不需要draw的话，就可以设置这个方法为true。这样系统知道你这个view 不需要draw 可以优化执行速度。viewgroup 一般都默认设置这个为true，因为viewgroup多数都是只负责布局，不负责draw的。而view 这个标志位 默认一般都是关闭的。

```

- 自定义view 有哪些需要注意的点？

```
 主要是要处理wrap_content 和padding。否则xml 那边设置这2个属性就根本没用了。还有不要在view中使用handler 因为人家已经提供了post方法。如果是继承自viewGroup,那在onMeasure和onLayout里面 也要考虑padding和layout的影响。也就是说specSize 要算一下 。最后就是如果view的动画或者线程需要停止，可以考虑在onDetachedFromWindow里面来做。

针对上述的几点，给出几个简单的自定义view 供大家理解。

给出一个圆形的view 范例：

package com.example.administrator.motioneventtest;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.view.View;

/**
 * Created by Administrator on 2016/2/4.
 */
public class CircleView extends View {

    private int mColor = Color.RED;
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    private void init() {
        mPaint.setColor(mColor);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

        //处理为wrap_content时的情况
        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, 200);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, heightSpecSize);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, 200);
        }

    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //处理padding的情况
        final int paddingLeft = getPaddingLeft();
        final int paddingRight = getPaddingRight();
        final int paddingTop = getPaddingTop();
        final int paddingBottom = getPaddingBottom();


        int width = getWidth() - paddingLeft - paddingRight;
        int height = getHeight() - paddingTop - paddingBottom;
        int radius = Math.min(width, height) / 2;
        canvas.drawCircle(paddingLeft + width / 2, paddingTop + height / 2, radius, mPaint);
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    public CircleView(Context context) {
        super(context);
        init();

    }

    public CircleView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }
}

```
```

	1. Measure 过程决定了 View 的测量宽/高，measure 过程结束后可以通过 getMeasuredWidth 和 getMeasuredHeight 方法获取 View 测量后的宽/高（正常情况下它都等于 View 最后的宽高，但是也有特殊情况）
	2. Layout 决定了四个顶点的坐标和实际 View 的宽和高，完成以后可以通过 getTop,getBottom,getLeft,getRight 拿到四个顶点的坐标，通过 getWidth,getHeight得到最终实际的宽高。
	3. Draw 决定 View 的显示，只有 draw 执行完毕以后，View 的内容才最终显示到屏幕上。

```


