>[掘金]:（ https://juejin.im/post/5c4fb8e9e51d4502723b1f68）
>
>

# 1.Handler 

[TOC]



## 1.1 对Handler的描述

```java
|--Handler允许您发送和处理与线程的MessageQueue关联的消息和可运行对象。
|--每个Handler实例都与单个线程和该线程的消息队列相关联。
|--当您创建一个新的Handler时，它会被绑定到正在创建它的线程的线程/消息队列上，
|--从那时起，它将向该消息队列传递消息和可运行项，并在它们从消息队列发出时执行它们。

|--Handler有两大主要的用处：
|--(1) 安排将消息和可运行项在将来的某个点执行
|--(2) 将在不同线程上执行的操作加入队列。

|--调度消息是通过方法：
|--post(Runnable), postAtTime(Runnable, long), postDelayed(Runnable, long),
|--sendEmptyMessage(int), sendMessage(Message),sendMessageAtTime(Message, long),
|--sendMessageDelayed(Message, long). 

```

## 1.2 Handler的成员变量

```java
final Looper mLooper;
final MessageQueue mQueue;
final Callback mCallback;
final boolean mAsynchronous;
IMessenger mMessenger;
```

## 1.3 Handler的构造方法

> 核心就只有两个构造方法(三个hide的，外面不能用)，其他四个可以用

```java
|--对具有指定回调接口的[当前线程]使用Looper，并设置handler是否应该是异步的。
|--默认情况下，handler是同步的，除非使用此构造函数创建一个严格异步的handler。
|--(插句话，此方法是隐藏的，说明外部调用者是无法创建异步的handler)

|--异步消息表示：不需要对同步消息进行全局排序的中断或事件。
|--异步消息不受MessageQueue#enqueueSyncBarrier(long)引入的同步屏障的限制。

* @param callback 处理消息的回调接口，或 null。
* @param async 如果为true，handler将为[发送到它那的每个Message或Runnable]调用
* Message#setAsynchronous(boolean)
---->[Handler#Handler(Callback,boolean)]-------------------------------
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {//final常量---false，所以不管他  
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }
    mLooper = Looper.myLooper();//通过Looper的myLooper方法返回值，为mLooper赋值
    if (mLooper == null) {//如果mLooper拿不到，报异常
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;//mQueue通过mLooper获取
    mCallback = callback;//入参
    mAsynchronous = async;//入参
}
|--貌似也没有做什么东西，只是将mLooper、mQueue、mCallback、mAsynchronous赋值
|--焦点在Looper.myLooper()上

---->[Handler#Handler(Callback,boolean)]-------------------------------
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
|--三参的大佬挺任性，直接赋值，所以Handler的构造函数没有什么太特别的
|--下面看一下Looper类
```
