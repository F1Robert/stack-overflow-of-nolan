# 解决 Retrofit io.reactivex.exceptions.OnErrorNotImplementedException: HTTP 404 Not Found

问题发生在这样一个场景，当前的后端服务是购买的第三方服务，现在将后端服务迁移到自研项目，

Android端代码，在搭配之前服务器后端使用时，未出现问题

```java
`try {`

​    `RetrofitClient.getInstance().setServiceApi().postHealthDataZJ(uploadBean)`

​      `.subscribeOn(Schedulers.io())`

​      `.subscribe(object : Consumer<Any?> {`

​        `@Throws(java.lang.Exception::class)`

​        `override fun accept(o: Any?) {`

​          `when (type) {`

​            `0 -> {`

​              `Log.e(UploadWorker.TAG, "ZJ血氧数据已上传")`

​            `}`

​            `1 -> {`

​              `Log.e(UploadWorker.TAG, "ZJ心率数据已上传")`

​            `}`

​            `2 -> {`

​              `Log.e(UploadWorker.TAG, "ZJ运动数据已上传")`

​            `}`

​            `3 -> {`

​              `Log.e(UploadWorker.TAG, "ZJ睡眠数据已上传")`

​            `}`

​          `}`

​          `//LogUtil.LongLog("上传血氧数据之后返回的数据：" + o);`

​        `}`

​      `})`

  `} catch (e: java.lang.Exception) {`

​    `LogUtil.error("网络错误", "网络问题")`

  `}`
```

现在后端项目迁移为自研之后，同样的代码报错

```java
io.reactivex.exceptions.OnErrorNotImplementedException: HTTP 404                                                                                                     	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:704)                                                                                                    	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:701)                                                                                                    	at io.reactivex.internal.observers.LambdaObserver.onError(LambdaObserver.java:77)                                                                                                    	at io.reactivex.internal.operators.observable.ObservableSubscribeOn$SubscribeOnObserver.onError(ObservableSubscribeOn.java:63)                                                                                                    	at retrofit2.adapter.rxjava2.BodyObservable$BodyObserver.onNext(BodyObservable.java:56)                                                                                                    	at retrofit2.adapter.rxjava2.BodyObservable$BodyObserver.onNext(BodyObservable.java:37)                                                                                                    	at retrofit2.adapter.rxjava2.CallExecuteObservable.subscribeActual(CallExecuteObservable.java:47)                                                                                                    	at io.reactivex.Observable.subscribe(Observable.java:11194)                                                                                                    	at retrofit2.adapter.rxjava2.BodyObservable.subscribeActual(BodyObservable.java:34)                                                                                                    	at io.reactivex.Observable.subscribe(Observable.java:11194)                                                                                                    	at io.reactivex.internal.operators.observable.ObservableSubscribeOn$SubscribeTask.run(ObservableSubscribeOn.java:96)                                                                                                    	at io.reactivex.Scheduler$DisposeTask.run(Scheduler.java:463)                                                                                                    	at io.reactivex.internal.schedulers.ScheduledRunnable.run(ScheduledRunnable.java:66)                                                                                                    	at io.reactivex.internal.schedulers.ScheduledRunnable.call(ScheduledRunnable.java:57)                                                                                                    	at java.util.concurrent.FutureTask.run(FutureTask.java:266)                                                                                                    	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:301)                                                                                                    	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1167)                                                                                                    	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:641)                                                                                                    	at java.lang.Thread.run(Thread.java:929)                                                                                                    Caused by: retrofit2.adapter.rxjava2.HttpException: HTTP 404      
```

于是查找io.reactivex.exceptions.OnErrorNotImplementedException这个error在何处抛出

最后找到ON_ERROR_MISSING这个静态量

![image-20231225104319296](C:\Users\86156\AppData\Roaming\Typora\typora-user-images\image-20231225104319296.png)

于是查看ON_ERROR_MISSING这个静态量在何处调用

![image-20231225104457378](C:\Users\86156\AppData\Roaming\Typora\typora-user-images\image-20231225104457378.png)

alt+F7 发现ON_ERROR_MISSING在多处调用

通过查找我们发现

![image-20231225110207509](C:\Users\86156\AppData\Roaming\Typora\typora-user-images\image-20231225110207509.png)

在这个subscribe(Consumeer )的构造方法中默认将ON_ERROR_MISSING作为异常抛出，如果遇到error将会用ON_ERROR_MISSING进行异常抛出，通过查询可知，该error提示在retrofit请求过程中发生异常的异常处理方法未实现。

于是修改原本代码为

```java
.subscribe(
    //    object : Consumer<Void> {
    //    override fun accept(t: Void?) {
    //        TODO("Not yet implemented")
    //    }
    //}, object :Consumer<Throwable> {
    //    override fun accept(t: Throwable?) {
    //        TODO("Not yet implemented")
    //        LogUtil.error("404","")
    //    }
    //}
    { response ->
        LogUtil.error("SUCCESS", "")
        // This is the onNext lambda
        // Handle the successful response here
    },
    { error ->
        LogUtil.error("404", "")
        // This is the onError lambda
        // Handle the error here
    }
```

response 为第一个参数，指定执行成功执行的逻辑

error为第二个参数，实现网络请求过程的异常处理

至此问题得解

结论：

io.reactivex.exceptions.OnErrorNotImplementedException是在使用retrofit进行网络请求时，未进行异常情况处理时的报错，

推荐无论如何场景使用retrofit，默认加上异常处理函数，即subscribe时至少传递两个参数

```
.subscribe(
    //    object : Consumer<Void> {
    //    override fun accept(t: Void?) {
    //        TODO("Not yet implemented")
    //    }
    //}, object :Consumer<Throwable> {
    //    override fun accept(t: Throwable?) {
    //        TODO("Not yet implemented")
    //        LogUtil.error("404","")
    //    }
    //}
    { response ->
        LogUtil.error("SUCCESS", "")
        // This is the onNext lambda
        // Handle the successful response here
    },
    { error ->
        LogUtil.error("404", "")
        // This is the onError lambda
        // Handle the error here
    }，至少传递两个参数
```

另外.subscribe可以传递4个参数

```
onNext:你希望获取到的服务器返回值

onError:进行yichangchuli

onComplete:在Http请求完成后的回调方法

onSubscribe:接收从上游返回的数据
```

