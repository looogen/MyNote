 [掘金]
 (https://juejin.im/post/5dafc49b6fb9a04e17209922)

# JetPack MVVM

## 1. Livecycle
> 解决生命周期管理一致性的问题，手工根据生命周期来激活、解绑和叫停容易滋生一致性隐患。

带来的好处

- 规避为监听状态而注入视图控制器的做法
- 规避为追溯事故来源而注入视图控制器的做法

~~~java
getLifecycle().addObserver(DefaultLifecycleObserver observer);
~~~

## 2. LiveData

> 遵循唯一可信源分发状态标准化开发理念

- 在LiveData之前分发状态多是通过EventBus 或者是Java Interface 来完成。

  引发的问题：1、EventBus容易因去中心化地滥用。造成诸如收到预期外的不明来源的推送、拿到过时的数据及事件源追溯复杂度为n^2的局面。2、EventBus本身缺乏Lifecycle的加持，存在生命周期管理一致性的问题。
  
- 在单例的配合下，承上启下地完成 状态 从 唯一可信源到视图控制器 的输送。

  这使得任何一次状态推送，都可预期、都能方便地追溯来源，而不至于在 事件追溯复杂度为 n² 的迷宫中白费时间。

- LiveData 被设计成粘性事件 MutableLiveData 可以专用于非粘性事件

## 3. ViewModel
> 解决状态管理和状态共享的问题

- 视图控制器重建时，用于重量级状态的托管
   轻量级的状态可以通过控制器的saveInstanceState机制，以序列化的方式完成保存和恢复。

- 状态托管和 **状态管理分治**
   在 Jetpack ViewModel 面市之前，MVP 的 Presenter 和 MVVM - Clean 的 ViewModel 都不具备状态管理分治的能力。
Presenter 和 Clean ViewModel 的生命周期都与视图控制器同生共死，因而它们顶多是为 DataBinding 提供状态的托管，而无法实现状态的分治。
   
   

## 4.  Databinding



## 综上

- Lifecycle 的存在，主要是为了解决 **生命周期管理 的一致性问题**。

- LiveData 的存在，主要是为了帮助 新手老手 都能不假思索地 **遵循 通过唯一可信源分发状态 的标准化开发理念**，从而在快速开发过程中 规避一系列 **难以追溯、难以排查、不可预期** 的问题。

- ViewModel 的存在，主要是为了解决 **状态管理 和 状态共享 的问题**。

- DataBinding 的存在，主要是为了解决 **视图 的一致性问题**。

它们的存在 大都是为了解决一致性的问题、将容易出错的操作在后台封装好，**方便使用者快速、稳定、不产生预期外错误地编码**。