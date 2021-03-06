---
title: View事件处理
categories: View
tags: View
toc: true  
---
- 基础知识

```
(1) 所有Touch事件都被封装成了MotionEvent对象，包括Touch的位置、时间、历史记录以及第几个手指(多指触摸)等。
(2) 事件类型分为ACTION_DOWN, ACTION_UP, ACTION_MOVE, ACTION_POINTER_DOWN, ACTION_POINTER_UP, ACTION_CANCEL，每个事件都是以ACTION_DOWN开始ACTION_UP结束。
(3) 对事件的处理包括三类，分别为传递——dispatchTouchEvent()函数、拦截——onInterceptTouchEvent()函数、消费——onTouchEvent()函数和OnTouchListener

```


- 传递流程

```
 (1) 事件从Activity.dispatchTouchEvent()开始传递，只要没有被停止或拦截，从最上层的View(ViewGroup)开始一直往下(子View)传递。子View可以通过onTouchEvent()对事件进行处理。
(2) 事件由父View(ViewGroup)传递给子View，ViewGroup可以通过onInterceptTouchEvent()对事件做拦截，停止其往下传递。
(3) 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent()函数。
(4) 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来。
(5) OnTouchListener优先于onTouchEvent()对事件进行消费。
上面的消费即表示相应函数返回值为true。


(01) View中的dispatchTouchEvent()会将事件传递给"自己的onTouch()", "自己的onTouchEvent()"进行处理。而且onTouch()的优先级比onTouchEvent()的优先级要高。 
(02) onTouch()与onTouchEvent()都是View中用户处理触摸事件的API。onTouch是OnTouchListener接口中的函数，OnTouchListener接口需要用户自己实现。onTouchEvent()是View自带的接口，Android系统提供了默认的实现；当然，用户可以重载该API。
(03) onTouch()与onTouchEvent()有两个不同之处：(01), onTouch()是View提供给用户，让用户自己处理触摸事件的接口。而onTouchEvent()是Android系统自己实现的接口。(02)，onTouch()的优先级比onTouchEvent()的优先级更高。dispatchTouchEvent()中分发事件的时候，会先将事件分配给onTouch()进行处理，然后才分配给onTouchEvent()进行处理。 如果onTouch()对触摸事件进行了处理，并且返回true；那么，该触摸事件就不会分配在分配给onTouchEvent()进行处理了。只有当onTouch()没有处理，或者处理了但返回false时，才会分配给onTouchEvent()进行处理。


```
<!-- more-->

- view的事件传递机制 如何用伪代码来表示？

```
/**
     * 对于一个root viewgroup来说，如果接受了一个点击事件，那么首先会调用他的dispatchTouchEvent方法。
     * 如果这个viewgroup的onInterceptTouchEvent 返回true，那就代表要拦截这个事件。接下来这个事件就
     * 给viewgroup自己处理了，从而viewgroup的onTouchEvent方法就会被调用。如果如果这个viewgroup的onInterceptTouchEvent
     * 返回false就代表我不拦截这个事件，然后就把这个事件传递给自己的子元素，然后子元素的dispatchTouchEvent
     * 就会被调用，就是这样一个循环直到 事件被处理。
     *
     */
public　boolean dispatchTouchEvent(MotionEvent ev)
{
    boolean consume=false;
    if (onInterceptTouchEvent(ev)) {
        consume=onTouchEvent(ev);
    }else
    {
        consume=child.dispatchTouchEvent(ev);
    }
    return consume;
}

```
- .view的onTouchEvent，OnClickListerner和OnTouchListener的onTouch方法 三者优先级如何？


```
onTouchListener优先级最高，如果onTouch方法返回 false ，那onTouchEvent就被调用了，返回true 就不会被调用。至于onClick 优先级最低。

```
- 点击事件的传递顺序如何？


```
Activity-Window-View。从上到下依次传递，当然了如果你最低的那个view onTouchEvent返回false 那就说明他不想处理 那就再往上抛，都不处理的话
最终就还是让Activity自己处理了。举个例子，pm下发一个任务给leader，leader自己不做 给架构师a，小a也不做 给程序员b，b如果做了那就结束了这个任务。

b如果发现自己搞不定，那就找a做，a要是也搞不定 就会不断向上发起请求，最终可能还是pm做。

//activity的dispatchTouchEvent 方法 一开始就是交给window去处理的
//win的superDispatchTouchEvent 返回true 那就直接结束了 这个函数了。返回false就意味
//这事件没人处理，最终还是给activity的onTouchEvent 自己处理 这里的getwindow 其实就是phonewindow
 public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }


//来看phonewindow的这个函数 直接把事件传递给了mDecor

 @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }

//devorview就是 我们的rootview了 就是那个framelayout 我们的setContentView里面传递的那个layout
//就是这个decorview的 子view了
     @Override
    public final View getDecorView() {
        if (mDecor == null) {
            installDecor();
        }
        return mDecor;
    }

```

- 事件分为几个步骤？

```

down事件开头，up事件结尾，中间可能会有数目不定的move事件。

```

- ViewGroup如何对点击事件分发？

```

viewgroup就是在actionMasked == MotionEvent.ACTION_DOWN 和 mFirstTouchTarget != null 这两种情况来判断是否会进入拦截事件的流程

看代码可以知道 如果是ACTION_DOWN事件  那就肯定进入 是否要拦截事件的流程

如果不是ACTION_DOWN事件 那就要看mFirstTouchTarget != null 这个条件是否成立

这个地方有点绕但是也好理解，其实就是 对于一个事件序列来说 down是事件的开头 所以肯定进入了这个事件是否拦截的流程 也就是if 括号内。


mFirstTouchTarget其实是一个单链表结构他指向的是 成功处理事件的子元素。

也就是说 如果有子元素成功处理了 事件，那这个值就不为NULL。反过来说

只要viewgroup拦截了事件，mFirstTouchTarget就不为NULL，所以括号内就不会执行，也就侧面说明了一个结论：

某个view 一旦决定拦截事件，那么这个事件所属的事件序列 都只能由他来执行。并且onInterceptTouchEvent 这个方法不会被调用了

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

- .如果某个view 处理事件的时候 没有消耗down事件 会有什么结果？

```
 
 假如一个view，在down事件来的时候 他的onTouchEvent返回false， 那么这个down事件 所属的事件序列 就是他后续的move 和up 都不会给他处理了，全部都给他的父view处理。

```
- 如果view 不消耗move或者up事件 会有什么结果？

```
那这个事件所属的事件序列就消失了，父view也不会处理的，最终都给activity 去处理了。

```

- .ViewGroup 默认拦截事件吗？

```
默认不拦截任何事件，onInterceptTouchEvent返回的是false。

```

- .requestDisallowInterceptTouchEvent 可以在子元素中干扰父元素的事件分发吗？如果可以，是全部都可以干扰吗？

```
肯定可以，但是down事件干扰不了。

```

- dispatchTouchEvent每次都会被调用吗？

```
是的，onInterceptTouchEvent则不会。
```

- 滑动冲突问题如何解决 思路是什么？

```
要解决滑动冲突 其实最主要的就是有一个核心思想。你到底想在一个事件序列中，让哪个view 来响应你的滑动？比如 从上到下滑，是哪个view来处理这个事件，从左到右呢？

用业务需求 来想明白以后 剩下的 其实就很好做了。核心的方法 就是2个 外部拦截也就是父亲拦截，另外就是内部拦截，也就是子view拦截法。 学会这2种 基本上所有的滑动冲突

都是这2种的变种，而且核心代码思想都一样。

外部拦截法：思路就是重写父容器的onInterceptTouchEvent即可。子元素一般不需要管。可以很容易理解，因为这和android自身的事件处理机制 逻辑是一模一样的

@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {

        boolean intercepted = false;
        int x = (int) ev.getX();
        int y = (int) ev.getY();

        switch (ev.getAction()) {
            //down事件肯定不能拦截 拦截了后面的就收不到了
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                break;
            case MotionEvent.ACTION_MOVE:
                if (你的业务需求) {
                    //如果确定拦截了 就去自己的onTouchEvent里 处理拦截之后的操作和效果 即可了
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            case MotionEvent.ACTION_UP:
                //up事件 我们一般都是返回false的 一般父容器都不会拦截他。 因为up是事件的最后一步。这里返回true也没啥意义
                //唯一的意义就是因为 父元素 up被拦截。导致子元素 收不到up事件，那子元素 就肯定没有onClick事件触发了，这里的
                //小细节 要想明白
                intercepted = false;
                break;
            default:
                break;
        }
        return intercepted;
    }

内部拦截法：内部拦截法稍微复杂一点，就是事件到来的时候，父容器不管，让子元素自己来决定是否处理。如果消耗了 就最好，没消耗 自然就转给父容器处理了。

子元素代码：

@Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                if (如果父容器需要这个点击事件) {
                    getParent().requestDisallowInterceptTouchEvent(false);
                }//否则的话 就交给自己本身view的onTouchEvent自动处理了
                break;
            case MotionEvent.ACTION_UP:
                break;
            default:
                break;
        }
        return super.dispatchTouchEvent(event);
    }

父亲容器代码也要修改一下，其实就是保证父亲别拦截down：

@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {

        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            return false;

        }
        return true;
    }
    
```

- 事件来源

```
InputEvent有2个子类：KeyEvent和MotionEvent，其中KeyEvent表示键盘事件，而MotionEvent表示点击事件

```

- 基于监听的事件监听器

```
View.OnClickListener    // 单击事件
View.OnCreateContextMenuListener    // 创建上下文菜单事件
View.OnFocusChangeListener    // 焦点改变事件
View.OnKeyListener    // 按钮事件
View.OnLongClickListener    // 长点击事件
View.OnTouchListener    // 触摸屏事件

```
- 基于回调的事件处理

```
   boolean onKeyDown(int keyCode, KeyEvent event)    // 按下
boolean onKeyLongPress(int keyCode, KeyEvent event)    // 长按
boolean onKeyUp(int keyCode, KeyEvent event)    // 松开
boolean onKeyShortcut(int keyCode, KeyEvent event)        // 键盘快捷键触发时
boolean onTouchEvent(MotionEvent event)        // 触摸屏事件

基于监听的事件处理更有优势：可维护性高、保证监听的事件监听器会被优先触发。
基于回调的事件处理更适合于那些比较固定的View。

    *事件传递*

所有基于回调的事件处理的回调方法返回true，表明已处理完成，不会继续传递；返回false，表明未处理完成，该事件继续传递下去。

当某个键被按下时候，Android最先触发的是该按键上绑定的事件监听器，然后触发该组件提供的事件回调方法，最后传递到该组件所在的Activity。

---

一种是委托式一种是回调式。第一种就是将事件的处理委托给监听器处理，你可以定义一个View.OnTouchListener接口的子类作为监听器，其中有onTouch()方法。而第二种是重写View类自己本身的onTouchEvent方法，也就是控件自己处理事件。onTouch方法接收一个MotionEvent参数和一个View参数，而onTouchEvent方法仅接收MotionEvent参数。这是因为监听器可以监听多个View控件的事件。无论是通过onTouchEvent还是onTouch方法 它们的返回值都是boolean类型。true的含义是如果当前处理程序在处理完毕该事件后不希望传播给其他控件，则返回true。如果View对象不但对此事件不感兴趣，而且对与此触摸序列相关的任何未来事件都不感兴趣，那么返回false。比如如果Button的onTouchEvent方法想要处理用户的一次点击 则会有2个事件产生ACTION_DOWN和ACTION_UP，按道理这两个事件都会调用onTouchEvent方法，如果方法返回false则在按下时你可以做一些操作，但是手指抬起时你将不能再接收到MotionEvent对象了，所以你也就无从处理抬起事件了。

---
```
