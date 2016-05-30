---
title: Handler消息传递机制
date: 2016-05-30 22:02:13
tags: Handler
categories: Andoroid通信

---
- Handler消息传递机制

Handler类的主要机制有两个：在子线程中发送消息；在主线程中获取处理消息。
Handler包含如下方法用于发送处理消息：
<!-- more-->

```
void handleMessage(Message msg)：处理消息
final boolean hasMessages(int what, Object object)：检查消息队列中是否包含what属性为指定值且object属性为指定对象的消息
boolean sendEmptyMessage(int what)：发送空消息
boolean sendEmptyMessageDelayed(int what, long delayMillis)：延迟发送空消息
boolean sendMessage(Message msg)：发送消息
boolean sendMessageDelayed(Message msg, long delayMillis)：延迟发送消息

```
- Handler、Looper、MessageQueue
    
Handler、Looper、MessageQueue各自作用如下：
Handler：能发送消息给MessageQueue，并能处理Looper分发给它的消息；
Looper：每个线程只有一个Looper，负责管理MessageQueue，从MessageQueue中取出消息分发给对应的Handler；
MessageQueue：采用先进先出管理Message；

注意：避免在主线程中执行耗时操作，否则会引发ANR异常。