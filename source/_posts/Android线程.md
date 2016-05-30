---
title: Android线程
toc: true
date: 2016-05-30 22:27:26
tags: 线程
categories: 性能优化
---
- 进程优先级（Process Priority）
  
    - Foreground Process
    ```
    Foreground一般意味着用户双眼可见，可见却不一定是active。    在Android的世界里，一个Activity处于前台之时，如果能采集用    户的input事件，就可以判定为active，如果中途弹出一个Dialog    ，Dialog变成新的active实体，直接面对用户的操作。被部分遮    挡的activity尽管依然可见，但状态却变为inactive。
    
    ```
    - Background Process
    
    ```
    所谓的Background可以理解为不可见（invisible）。对于不可见的任务，Android也有重要性的区分。重要的后台任务定义为Service，如果一个进程包含Service（称为Service Process），那么在“重要性”上就会被系统区别对待，其优先级自然会高于不包含Service的进程（称为Background Process），最后还剩一类空进程（Empty Process）。Empty Process初看有些费解，一个Process如果什么都不做，还有什么存在的必要。其实Empty Process并不Empty，还存在不少的内存占用。
    
    ```
<!-- more-->
- 线程调度（Thread Scheduling）
  
```
Android将线程分为多个group，其中两类group尤其重要。一类是default group，UI线程属于这一类。另一类是background group，工作线程应该归属到这一类。background group当中所有的线程加起来总共也只能分配到5～10%的time slice，剩下的全部分配给default group，这样设计显然能保证UI线程绘制UI的流畅性。

Android开发者需要显式的将工作线程归于background group。

new Thread(new Runnable() {
  @Override
  public void run() {
    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
  }
}).start();

虽说Android系统在任务调度上是以线程为基础单位，设置单个thread的优先级也可以改变其所属的control groups，从而影响CPU time slice的分配。但进程的属性变化也会影响到线程的调度，当一个App进入后台的时候，该App所属的整个进程都将进入background group，以确保处于foreground，用户可见的新进程能获取到尽可能多的CPU资源。用adb可以查看不同进程的当前调度策略。

$ adb shell ps -P

当你的App重新被用户切换到前台的时候，进程当中所属的线程又会回归的原来的group。在这些用户频繁切换的过程当中，thread的优先级并不会发生变化，但系统在time slice的分配上却在不停的调整。

移动端App新启线程一般都是为了保证UI的流畅性，增加App用户操作的响应度。但是否需要将任务放入工作线程需要先了解任务的瓶颈在哪，是i/o，gpu还是cpu？UI出现卡顿并不一定是UI线程出现了费时的计算，有可能是其它原因，比如layout层级太深。

```
- 用什么姿势开线程？

    - new Thread()
    - AsyncTask
    - HandlerThread
    - ThreadPoolExecutor
    - IntentService
```
这是Android系统里开线程最简单的方式，也只能应用于最简单的场景，简单的好处却伴随不少的隐患。

new Thread(new Runnable() {
            @Override
            public void run() {
 
            }
        }).start();

这种方式仅仅是起动了一个新的线程，没有任务的概念，不能做状态的管理。start之后，run当中的代码就一定会执行到底，无法中途取消。

Runnable作为匿名内部类还持有了外部类的引用，在线程退出之前，该引用会一直存在，阻碍外部类对象被GC回收，在一段时间内造成内存泄漏。

没有线程切换的接口，要传递处理结果到UI线程的话，需要写额外的线程切换代码。

如果从UI线程启动，则该线程优先级默认为Default，归于default cgroup，会平等的和UI线程争夺CPU资源。这一点尤其需要注意，在对UI性能要求高的场景下要记得

Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

虽说处于background group的线程总共只能争取到5～10%的CPU资源，但这对绝大部分的后台任务处理都绰绰有余了，1ms和10ms对用户来说，都是快到无法感知，所以我们一般都偏向于在background group当中执行工作线程任务。

```
  
  
```
一个典型的AsyncTask实现如下：

public class MyAsyncTask extends AsyncTask {
 
        @Override
        protected Object doInBackground(Object[] params) {
            return null;
        }
 
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }
 
        @Override
        protected void onPostExecute(Object o) {
            super.onPostExecute(o);
        }
    }

和使用Thread()不同的是，多了几处API回调来严格规范工作线程与UI线程之间的交互。我们大部分的业务场景几乎都符合这种规范，比如去磁盘读取图片，缩放处理需要在工作线程执行，最后绘制到ImageView控件需要切换到UI线程。

AsyncTask的几处回调都给了我们机会去中断任务，在任务状态的管理上较之Thread()方式更为灵活。值得注意的是AsyncTask的cancel()方法并不会终止任务的执行，开发者需要自己去检查cancel的状态值来决定是否中止任务。

AsyncTask也有隐式的持有外部类对象引用的问题，需要特别注意防止出现意外的内存泄漏。

AsyncTask由于在不同的系统版本上串行与并行的执行行为不一致，被不少开发者所诟病，这确实是硬伤，绝大部分的多线程场景都需要明确任务是串行还是并行。

线程优先级为background，对UI线程的执行影响极小。

```

```

在需要对多任务做更精细控制，线程切换更频繁的场景之下，Thread()和AsyncTask都会显得力不从心。HandlerThread却能胜任这些需求甚至更多。

HandlerThread将Handler，Thread，Looper，MessageQueue几个概念相结合。Handler是线程对外的接口，所有新的message或者runnable都通过handler post到工作线程。Looper在MessageQueue取到新的任务就切换到工作线程去执行。不同的post方法可以让我们对任务做精细的控制，什么时候执行，执行的顺序都可以控制。HandlerThread最大的优势在于引入MessageQueue概念，可以进行多任务队列管理。

HandlerThread背后只有一个线程，所以任务是串行执行的。串行相对于并行来说更安全，各任务之间不会存在多线程安全问题。

HandlerThread所产生的线程会一直存活，Looper会在该线程中持续的检查MessageQueue。这一点和Thread()，AsyncTask都不同，thread实例的重用可以避免线程相关的对象的频繁重建和销毁。

HandlerThread较之Thread()，AsyncTask需要写更多的代码，但在实用性，灵活度，安全性上都有更好的表现。

```

```
Thread(),AsyncTask适合处理单个任务的场景，HandlerThread适合串行处理多任务的场景。当需要并行的处理多任务之时，ThreadPoolExecutor是更好的选择。

public static Executor THREAD_POOL_EXECUTOR
= new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);

线程池可以避免线程的频繁创建和销毁，显然性能更好，但线程池并发的特性往往也是疑难杂症的源头，是代码降级和失控的开始。多线程并行导致的bug往往是偶现的，不方便调试，一旦出现就会耗掉大量的开发精力。

ThreadPool较之HandlerThread在处理多任务上有更高的灵活性，但也带来了更大的复杂度和不确定性。

```

```
IntentService又是另一种开工作线程的方式，从名字就可以看出这个工作线程会带有service的属性。和AsyncTask不同，没有和UI线程的交互，也不像HandlerThread的工作线程会一直存活。IntentService背后其实也有一个HandlerThread来串行的处理Message Queue，从IntentService的onCreate方法可以看出：

@Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.
 
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();
 
        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

只不过在所有的Message处理完毕之后，工作线程会自动结束。所以可以把IntentService看做是Service和HandlerThread的结合体，适合需要在工作线程处理UI无关任务的场景。

```

