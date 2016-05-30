---
title: 自定义View的疑问
date: 2016-05-30 20:02:50
tags: View
categories: View
toc: true 
---
- invalidate()方法 ：

`   说明：请求重绘View树，即draw()过程，假如视图发生大小没有变化就不会调用layout()过程，并且只绘制那些“需要重绘的”
视图，即谁(View的话，只绘制该View ；ViewGroup，则绘制整个ViewGroup)请求invalidate()方法，就绘制该视图。
 
   一般引起invalidate()操作的函数如下：
   1、直接调用invalidate()方法，请求重新draw()，但只会绘制调用者本身。
   
   2、setSelection()方法 ：请求重新draw()，但只会绘制调用者本身。
   
   3、setVisibility()方法 ： 当View可视状态在INVISIBLE转换VISIBLE时，会间接调用invalidate()方法 继而绘制该View。
  
   4 、setEnabled()方法 ： 请求重新draw()，但不会重新绘制任何视图包括该调用者本身。`

<!-- more-->

- requestLayout()方法 


 
`：会导致调用measure()过程 和 layout()过程 。
 
说明：只是对View树重新布局layout过程包括measure()和layout()过程，不会调用draw()过程，但不会重新绘制

任何视图包括该调用者本身。
 
 一般引起invalidate()操作的函数如下：
 1、setVisibility()方法：
 当View的可视状态在INVISIBLE/ VISIBLE 转换为GONE状态时，会间接调用requestLayout() 和invalidate方法。
 同时，由于整个个View树大小发生了变化，会请求measure()过程以及draw()过程，同样地，只绘制需要“重新绘制”的视图。`
    
 
-  requestFocus()函数说明：
```
          说明：请求View树的draw()过程，但只绘制“需要重绘”的视图。
```
- View的坐标参数 主要有哪些？分别有什么注意的要点？

```
 Left，Right，top,Bottom 注意这4个值其实就是 view 和 他的父控件的 相对坐标值。 并非是距离屏幕左上角的绝对值，这点要注意。
　
　此外，X和Y 其实也是相对于父控件的坐标值。 TranslationX,TranslationY 这2个值 默认都为0，是相对于父控件的左上角的偏移量。
　
　换算关系：
　
　x=left+tranX,y=top+tranY.
　 
  很多人不理解，为什么事这样，其实就是View 如果有移动的话，比如平移这种，你们就要注意了，top和left 这种值 是不会变化的。
   
   无论你把view怎么拖动，但是 x,y,tranX,tranY 的值是随着拖动平移 而变化的。想明白这点 就行了。
   
```
- onTouchEvent和GestureDetector 在什么时候用哪个比较好

```
只有滑动需求的时候 就用前者，如果有双击等这种行为的时候 就用后者。

```
- .Scroller 用来解决什么问题？

```
view的scrollTo和scrollBy 滑动效果太差了，是瞬间完成。而scroller可以配合view的computeScroll 来完成 渐变的滑动效果。体验更好。

```
- ScrollTo和ScrollBy 有什么需要注意的？

```
前者是绝对滑动，后者是相对滑动。滑动的是view的内容 而不是view本身。这很重要。比如textview 调用这2个方法  滑动的就是显示出来的字的内容。

一般而言 我们用scrollBy会比较多一些。传值的话 其实 记住几个法则就可以了。 右-左 x为正 否则x为负  上-下 y为负，否则y为正。

可以稍微看一下 这2个的源码：
public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
    }

 public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }

看到里面有2个变量 mScrollX 和mScrollY 这2个东西没，这2个单位的 值是像素，前者代表 view的左边缘和view内容左边缘的距离。 后者代表 view上边缘和view内容上边缘的距离。

```
- 让view滑动总共有几种方式，分别要注意什么？都适用于那些场景？

```
总共有三种：

a：scrollto，scrollby。这种是最简单的，但是只能滑动view的内容 不可以滑动view本身。

b：动画。动画可以滑动view内容，但是注意非属性动画 就如我们问题5说的内容 会影响到交互，使用的时候要多注意。不过多数复杂的滑动效果都是属性动画来完成的，属于大杀器级别、

c：改变布局参数。这种最好理解了，无非是动态的通过java代码来修改 margin等view的参数罢了。不过用的比较少。我本人不怎么用这种方法。

```
- .Scroller是干嘛的？原理是什么？

```
```

- getWidth()与getMeasuredWidth()`有什么区别呢？ 

```
一般情况下这两个的值是相同的，`getMeasureWidth()`方法在`measure()`过程结束后就可以获取到了，而`getWidth()`方法要在`layout()`过程结束后才能获取到。
而且`getMeasureWidth()`的值是通过`setMeasuredDimension()`设置的，但是`getWidth()`的值是通过视图右边的坐标减去左边的坐标计算出来的。如果我们在`layout`的时候将宽高
不传`getMeasureWidth`的值，那么这时候`getWidth()`与`getMeasuredWidth`的值就不会再相同了，当然一般也不会这么干...

```

