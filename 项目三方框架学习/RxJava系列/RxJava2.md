### 学习博客
[RxJava2 浅析](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0907/6604.html)

---
### RxJava2 三部曲

#### 1：初始化一个 Observable
```
       Observable<Integer> observable=Observable.create(new ObservableOnSubscribe<Integer>() {

            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onComplete();
            }
        });
```
#### 2：初始化一个 Observer
```
        Observer<Integer> observer= new Observer<Integer>() {
          private Disposable disposable; // 用于接收异常数据时解除订阅

            @Override
            public void onSubscribe(Disposable d) {
              // 相比RxJava多的回调方法
              // 参数 Disposable 相当于 RxJava1.x中的 Subscription，用于解除订阅
              // 可方便的在接收到异常数据项时进行解除订阅操作, 优于RxJava1的设计
              // RxJava1 只能在建立订阅关系的时候返回 Disposable 参数

              disposable = d;
            }

            @Override
            public void onNext(Integer value) {
              if (value > 3) { // 假设当 > 3 时为异常数据
                disposable.dispose();
              }
            }

            @Override
            public void onError(Throwable e) {
            }

            @Override
            public void onComplete() {
            }
        }
```
#### 3：建立订阅关系
```
    observable.subscribe(observer); //建立订阅关系
```

#### 简化订阅
> RxJava2.x中仍然保留了其他简化订阅方法，我们可以根据需求，选择相应的简化订阅。只不过传入的对象改为了Consumer。
```
Disposable disposable = observable.subscribe(new Consumer<Integer>() {
         @Override
         public void accept(Integer integer) throws Exception {
               //这里接收数据项
         }
     },
     new Consumer<Throwable>() {
         @Override
         public void accept(Throwable throwable) throws Exception {
           //这里接收onError
         }
     },
    new Action() {
         @Override
         public void run() throws Exception {
           //这里接收onComplete。
         }
     });
```
----
### Flowable
>Flowable是RxJava2.x中新增的类，专门用于应对背压（Backpressure）问题，但这并不是RxJava2.x中新引入的概念。所谓背压，即生产者的速度大于消费者的速度带来的问题，比如在Android中常见的点击事件，点击过快则会造成点击两次的效果。

我们知道，在RxJava1.x中背压控制是由Observable完成的，使用如下：
```
Observable.range(1,10000)
          .onBackpressureDrop()
          .subscribe(integer -> Log.d("JG",integer.toString()));
```

而在RxJava2.x中将其独立了出来，取名为Flowable。因此，原先的Observable已经不具备背压处理能力。***通过Flowable我们可以自定义背压处理策略。***
