- 定义Retrofit 网络请求接口  
- 初始化Retrofit 实例

1. 配置 OkHttpClient 添加查看 Request / Response 数据拦截器

2. 构建Retrofit  对象
3. 拿到Retrofit 请求对象

-  **Retrofit + OKHttp**  的使用方式

```java
Call<JsonObject> call = api.getWXarticles();
	//OKHttp 异步访问网络
  call.enqueue(new Callback<JsonObject>() {
  @Override
  public void onResponse(Call<JsonObject> call, retrofit2.Response<JsonObject> response) {
  	Log.d(TAG, "Retrofit + OKHttp "+response.body().toString());
  }

  @Override
  public void onFailure(Call<JsonObject> call, Throwable t) {
  	Log.d(TAG, "Retrofit + OKHttp "+t.getMessage());
  }
});
```



-  **RxJava + Retrofit + OKHttp** 
```java
  Observable<JsonObject> observable = api.getWXarticle();
  observable.subscribeOn(Schedulers.io())
                .subscribeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<JsonObject>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "RxJava + Retrofit + OKHttp 订阅成功");
                    }

                    @Override
                    public void onNext(JsonObject s) {
                        Log.d(TAG, "RxJava + Retrofit + OKHttp "+s.toString());
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "RxJava + Retrofit + OKHttp "+e.getMessage());
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "RxJava + Retrofit + OKHttp onComplete");
                    }
                });

```



- Okhttp 源码分析

1. 创建OkHttpClient

2. 构建一个Request 请求

3. 创建RealCall

4. 同步执行

   - 是否已经执行过了

   - 调用dispather() 调度器分发给同步执行

      	> 将当前请求的Call添加进行同步队列中

     

   - 调用拦截器

      > getResponseWithInterceptorChain 

      

   - 不管是否执行成功，关闭当前请求任务

5. 异步处理
	与同步执行类似，最后调用分发器进行处理
	
6. 	dispatcher.enqueue  
```java
  void enqueue(AsyncCall call) {
    synchronized (this) {
      //1. 将要执行的异步任务加入准备执行的异步队列中
      readyAsyncCalls.add(call);
      //2. 判断是否是WebSocket
      if (!call.get().forWebSocket) {
        AsyncCall existingCall = findExistingCallWithHost(call.host());
        if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
      }
    }
    //3. 准备执行
    promoteAndExecute();
  }



 //分析 promoteAndExecute 
  private int maxRequests = 64;
  private int maxRequestsPerHost = 5;

private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
      //1. 准备执行的异步任务
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();
          //2. 判断正在运行的异步任务数量是否已经超过最大限制64
        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        //3. 判断请求同一个主机的Host不能大于5 （可以自定义设置）
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.
				//4. 先删除当前需要执行的异步任务
        i.remove();
        asyncCall.callsPerHost().incrementAndGet();
        //5. 将当前需要准备执行的异步任务加入到正在运行异步任务的队列
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
      isRunning = runningCallsCount() > 0;
    }
		//6. 遍历当前需要执行的异步任务，拿到异步任务执行
    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
        //executorService 返回一个线程池对象
      asyncCall.executeOn(executorService());
    }

    return isRunning;
  }

```

7. 子线程run ->  execute 
   - 通过 getResponseWithInterceptorChain 拿到服务器响应数据
   - 通过回调把响应数据返回给调用层
   - finished，判断是否有需要执行的任务，然后继续删除准备执行的异步任务，开始执行删除的异步任务


8. getResponseWithInterceptorChain

	- 添加拦截器  分类为：应用拦截器  网络拦截器
	- 
	





