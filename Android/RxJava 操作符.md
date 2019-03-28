## RxJava 操作符

### 创建操作

- create: 使用OnSubscribe 从头创建一个Observable，这种方法比较简单。需要注意的是，在使用该方法创建时，建议在OnSubscribe #call 方法中检查订阅状态，以便及时停止发射数据和运算

  ```java
  Observable.create(new Observable.OnSubscribe<String>() {

          @Override
          public void call(Subscriber<? super String> subscriber) {

              subscriber.onNext("item1");
              subscriber.onNext("item2");
              subscriber.onCompleted();
          }
      });
  ```

- from : 将一个Iterable,一个Future，或者一个数组，内部通过代理的方式转换成一个Observable。

  ```java
  //Iterable
      List<String> list=new ArrayList<>();
      ...
      Observable.from(list)
              .subscribe(new Action1<String>() {
          @Override
          public void call(String s) {

          }
      });

      //Future
       Future<String> futrue= Executors.newSingleThreadExecutor().submit(new Callable<String>() {

          @Override
          public String call() throws Exception {
              Thread.sleep(1000);
              return "maplejaw";
          }
      });

      Observable.from(futrue)
                .subscribe(new Action1<String>() {
          @Override
          public void call(String s) {

          }
      });
  ;
  ```

- just : 将一个或多个对象转换成发射这个或这些对象的一个Observable。如果是单个对象，内部创建的是ScalarSynchronousObservable 对象，如果是多个对象，则是调用了from方法创建。

- empty : 创建一个什么都不做直接通知完成的Observable。

- error : 创建一个什么都不做直接通知错误的Observable。

- never : 创建一个什么都不做的Observable。

  ```java
   Observable observable1=Observable.empty();//直接调用onCompleted。
      Observable observable2=Observable.error(new RuntimeException());//直接调用onError。这里可以自定义异常
      Observable observable3=Observable.never();//啥都不做
  ```

- timer : 创建一个在给定延时之后发射数据项为0 的Observable<Long>,内部通过OnSubscribeTimerOnce 工作。

  ```java
   Observable.timer(1000,TimeUnit.MILLISECONDS)
              .subscribe(new Action1<Long>() {
                  @Override
                  public void call(Long aLong) {
                      Log.d("JG",aLong.toString()); // 0
                  }
              });
  ```

- interval: 创建一个按照给定时间间隔发射行0开始的整数序列的Observable<Long> ,内部通过OnsubscribeTimerPeriodically 工作。

  ```java
  Observable.interval(1, TimeUnit.SECONDS)
              .subscribe(new Action1<Long>() {
                  @Override
                  public void call(Long aLong) {
                       //每隔1秒发送数据项，从0开始计数
                       //0,1,2,3....
                  }
              });
  ```

- range : 创建一个发射指定范围的整数序列的Observable<Integer>

  ```java
   Observable.range(2,5).subscribe(new Action1<Integer>() {
          @Override
          public void call(Integer integer) {
              Log.d("JG",integer.toString());// 2,3,4,5,6 从2开始发射5个数据
          }
      });
  ```

- defer : 只有当订阅者订阅才创建Observable,为每一个订阅创建一个新的Observable。内部通过OnsubscribeDefer 在订阅时顶用Func0创建Observable。

  ```java
  Observable.defer(new Func0<Observable<String>>() {
          @Override
          public Observable<String> call() {
              return Observable.just("hello");
          }
      }).subscribe(new Action1<String>() {
          @Override
          public void call(String s) {
              Log.d("JG",s);
          }
      });
  ```

### 合并操作

以下操作符用于组合多个Observable。

- concat : 按顺序连接多个Observable。 需要注意的是 Observable.cincat(a,b)等价于a.concatWith(b)。

  ```java
  Observable<Integer> observable1=Observable.just(1,2,3,4);
      Observable<Integer>  observable2=Observable.just(4,5,6);

      Observable.concat(observable1,observable2)
              .subscribe(item->Log.d("JG",item.toString()));//1,2,3,4,4,5,6
  ```

- startWith : 在数据序列的开头增加一项数据。startWith的内也是调用了concat

  ```java
   Observable.just(1,2,3,4,5)
              .startWith(6,7,8)
      .subscribe(item->Log.d("JG",item.toString()));//6,7,8,1,2,3,4,5 
  ```

- merge : 将多个Observable 合并为一个。不同于concat，merge 是按照时间线来连接的。

### 过滤操作

- filter : 过滤数据。内部通过OnSubscribeFilter 过滤数据。

  ```java
   Observable.just(3,4,5,6)
              .filter(new Func1<Integer, Boolean>() {
                  @Override
                  public Boolean call(Integer integer) {
                      return integer>4;
                  }
              })
      .subscribe(item->Log.d("JG",item.toString())); //5,6 
  ```

- ofType : 过滤指定类型的数据，于filter类似

  ```java
  Observable.just(1,2,"3")
              .ofType(Integer.class)
              .subscribe(item -> Log.d("JG",item.toString()));
  ```

- take : 只发射开始的N项数据或者一定时间内的数据。内部通过Operator和OperaTakeTimed过滤数据。

  ```java
    Observable.just(3,4,5,6)
              .take(3)//发射前三个数据项
              .take(100, TimeUnit.MILLISECONDS)//发射100ms内的数据
  ```

- takeLast : 只发射最后的N项数据或者一定时间内的数据。内部通过OperatorTakeLast和OperatorTakeLastTimed过滤数据。takeLastBuffer和takeLast类似，不同点在于takeLastBuffer会收集成List后发射。

  ```java
  Observable.just(3,4,5,6)
              .takeLast(3)
              .subscribe(integer -> Log.d("JG",integer.toString()));//4,5,6
  ```

- takeFirst : 提取满足条件的第一项。内部实现代码

  ```java
  public final Observable<T> takeFirst(Func1<? super T, Boolean> predicate) {
        return filter(predicate).take(1); //先过滤，后提取
  }
  ```

- first/firstOrDefault :　只发射第一项数据，可以指定默认值

  ```java
  Observable.just(3,4,5,6)
              .first()
              .subscribe(integer -> Log.d("JG",integer.toString()));//3

      Observable.just(3,4,5,6)
                 .first(new Func1<Integer, Boolean>() {
                     @Override
                     public Boolean call(Integer integer) {
                         return integer>3;
                     }
                 }) .subscribe(integer -> Log.d("JG",integer.toString()));//4
  ```

- last/lastOrDefault :只发射最后一项，可以指定默认值

- skip : 跳过开始的N项数据或者是一定时间内的数据。内部通过OPeratorSkip和OperatorSkipTimed 实现过滤。

  ```java
    Observable.just(3,4,5,6)
                 .skip(1)
              .subscribe(integer -> Log.d("JG",integer.toString()));//4,5,6
  ```

- skipLast :　跳过最后的N项数据或者是一定时间的数据。内部通过OperatorSkipLast和OpearatorLastSkipTimed实现过滤。

- elementAt/elementAtOrDefault : 发射某一项数据，如果超出了范围可以指定默认值。内部通过OperatorElementAt 过滤。

  ```java
    Observable.just(3,4,5,6)
                   .elementAt(2)
          .subscribe(item->Log.d("JG",item.toString())); //5
  ```

### 条件/布尔操作

- all : 判断所有数据是否满足某个条件，内部通过OperatorAll 实现。

  ```java
   Observable.just(2,3,4,5)
              .all(new Func1<Integer, Boolean>() {
                  @Override
                  public Boolean call(Integer integer) {
                      return integer>3;
                  }
              })
      .subscribe(new Action1<Boolean>() {
          @Override
          public void call(Boolean aBoolean) {
              Log.d("JG",aBoolean.toString()); //false
          }
      })
      ;
  ```

- exist : 判断是否存在数据满足某个条件。内部通过OperatorAll 实现。

  ```java
     Observable.just(2,3,4,5)
              .exists(integer -> integer>3)
              .subscribe(aBoolean -> Log.d("JG",aBoolean.toString())); //true
  ```

- contains : 判断发射的所有数据中是否包含指定的数据，内部调用的其实是exists

  ```java
    Observable.just(2,3,4,5)
              .contains(3)
              .subscribe(aBoolean -> Log.d("JG",aBoolean.toString())); //true
  ```

- sequenceEqual : 判断两个Observable 发射的数据是否相同（数据、发射顺序、终止状态）。

  ```java
  Observable.sequenceEqual(Observable.just(2,3,4,5),Observable.just(2,3,4,5))
              .subscribe(aBoolean -> Log.d("JG",aBoolean.toString()));//true
  ```

- isEmpty : 用于判断Observable 发射完毕后，有没有发射数据。有数据false,如果只收到了onComplete通知则为true。

  ```java
    Observable.just(3,4,5,6)
                 .isEmpty()
                .subscribe(item -> Log.d("JG",item.toString()));//false
  ```

### 聚合操作

- reduce  : 对序列使用reduce() 函数并发射最终的结果，内部使用OnSubscribeReduce 实现

  ```java
    Observable.just(3,4,5,6)
                 .isEmpty()
                .subscribe(item -> Log.d("JG",item.toString()));//false
  ```

### 转换操作

- toList : 收集原始Observable 发射的所有数据到一个列表，然后返回这个列表

  ```java
      Observable.just(2,3,4,5)
              .toList()
              .subscribe(new Action1<List<Integer>>() {
                  @Override
                  public void call(List<Integer> integers) {

                  }
              });
  ```

- toSortedList : 收集原始Observable发射的所有数据到一个有序的列表，然后返回这个列表

  ```java
    Observable.just(6,2,3,4,5)
              .toSortedList(new Func2<Integer, Integer, Integer>() {//自定义排序
                  @Override
                  public Integer call(Integer integer, Integer integer2) {
                      return integer-integer2; //>0 升序 ，<0 降序
                  }
              })
              .subscribe(new Action1<List<Integer>>() {
                  @Override
                  public void call(List<Integer> integers) {
                      Log.d("JG",integers.toString()); // [2, 3, 4, 5, 6]
                  }
              });
  ```

- toMap : 将序列数据转换成一个Map。可以根据数据项生成key和生成value。

### 变换操作

- map : 对Observable 发射的每一项数据都应用一个函数来变换。

  ```java
   Observable.just(6,2,3,4,5)
              .map(integer -> "item:"+integer)
              .subscribe(s -> Log.d("JG",s));//item:6,item:2....
  ```

- cast : 在发射之前强制将Observable 发射的所有数据转换为指定类型。

- flatMap : 将Observable 发射的数据变换为Observable 集合，然后将这些Observable发射的数据平坦话的放进一个单独的Observable,内部采用merge合并。

  ```java
        Observable.just(2,3,5)
              .flatMap(new Func1<Integer, Observable<String>>() {
                  @Override
                  public Observable<String> call(Integer integer) {
                      return Observable.create(subscriber -> {
                          subscriber.onNext(integer*10+"");
                          subscriber.onNext(integer*100+"");
                          subscriber.onCompleted();
                      });
                  }
              })
      .subscribe(o -> Log.d("JG",o)) //20,200,30,300,50,500
  ```

### 错误处理/重试机制

- onErrorResumeNext : 当原始Observable在遇到错误时，使用备用的Observable。

  ```java
   Observable.just(1,"2",3)
      .cast(Integer.class)
      .onErrorResumeNext(Observable.just(1,2,3))
      .subscribe(integer -> Log.d("JG",integer.toString())) //1,2,3
      ;
  ```

- onExceptionResumeNext : 当原始Observable 在遇到异常时，使用备用的Observable。

- retry : 当原始Observable 在遇到错误时进行重试

  ```java
   Observable.just(1,"2",3)
      .cast(Integer.class)
      .retry(3)
      .subscribe(integer -> Log.d("JG",integer.toString()),throwable -> Log.d("JG","onError"))
      ;//1,1,1,1,onError
  ```

- retryWhen : 当原始Observable 在遇到错误的时候，将错误传递给另一个Observable 来决定是否要重新订阅这个Observable ，内部调用的是retry。

  ```java
  Observable.just(1,"2",3)
      .cast(Integer.class)
      .retryWhen(new Func1<Observable<? extends Throwable>, Observable<Long>>() {
          @Override
          public Observable<Long> call(Observable<? extends Throwable> observable) {
              return Observable.timer(1, TimeUnit.SECONDS);
          }
      })
      .subscribe(integer -> Log.d("JG",integer.toString()),throwable -> Log.d("JG","onError"));
      //1,1
  ```

### 连接操作

- `ConnectableObservable`与普通的Observable差不多，但是可连接的Observable在被订阅时并不开始发射数据，只有在它的connect()被调用时才开始。用这种方法，你可以等所有的潜在订阅者都订阅了这个Observable之后才开始发射数据。 
  `ConnectableObservable.connect()`指示一个可连接的Observable开始发射数据. 
  `Observable.publish()`将一个Observable转换为一个可连接的Observable 
  `Observable.replay()`确保所有的订阅者看到相同的数据序列的`ConnectableObservable`，即使它们在Observable开始发射数据之后才订阅。 
  `ConnectableObservable.refCount()`让一个可连接的Observable表现得像一个普通的Observable。

  ```java
  ConnectableObservable<Integer> co= Observable.just(1,2,3)
                  .publish();

          co .subscribe(integer -> Log.d("JG",integer.toString()) );
          co.connect();//此时开始发射数据
  ```

  ​