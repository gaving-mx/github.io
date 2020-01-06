RxJS 是 [Reactive Extensions](http://reactivex.io/) 在 JavaScript 上的实现，其全称是Reactive Extensions for JavaScript，Reactive Extensions 在其他语言也有相应的实现，如 RxJava、RxAndroid、RxSwift 等. 这篇文章只是简单阐述RxJS中的一些基本概念, 不会涉及到太多的用法. 如果你是没有接触过RxJS的同学, 希望可以帮助你理解.

## Reactive Programming 是针对数据流(同步或异步)的编程.
事件总线、点击事件都是数据流。开发者可以观测这些数据流(Observable)，并调用特定的逻辑对它们进行处理(Observer)。使用Reactive如同开挂：你可以创建点击、悬停之类的任意流. 通常流很廉价（点击一下就出来一个），种类丰富多样：变量，用户输入，属性，缓存，数据结构等等都可以产生流.

基于流的概念，Reactive赋予了你一系列神奇的函数工具集(Operator)，使用他们可以合并、创建、过滤这些流。一个流或者一系列流可以作为另一个流的输入. 如果流是Reactive programming的核心，我们不妨从点击事件体验一下数据流。

## 通过点击事件创建数据流
引入[Rx.js](https://unpkg.com/@reactivex/rxjs@5.0.0-beta.12/dist/global/Rx.js)库
```
Rx.Observable.fromEvent(document, 'click')
             .subscribe(e => console.log(e));
```
打开控制台, 点击页面任意位置, 观察控制台的输出.

![fromEvent.gif](http://upload-images.jianshu.io/upload_images/2261833-b465ad2c57dbf193.gif?imageMogr2/auto-orient/strip)

## 基本概念

### Push vs. Pull
如果你熟悉Iterator, 在这种情况下, 你是主宰, 无论何时只要你想要数据, 你只需要调用next()来pull数据.
```
var it = makeIterator(['yo', 'ya']);
console.log(it.next().value); // 'yo'
```
但是在RxJs中, Observable是主宰, 无论何时它得到新数据, 它会push数据给你, 而你需要做的事情就是"listen".

### Observable

可以简单理解为一个不断发射数据的函数,  为了更好理解, 把这个函数假设成网球发球机, 每发出的一个网球都是函数的值, 所有的网球源源不断的发射出去形成了我们所谓的数据流. 创建一个Observable有多种方式, 可以通过现有的promise, 事件, 数据去创建, 详情可以参考 [creation-operators](http://reactivex.io/rxjs/manual/overview.html#creation-operators), 这里仅介绍`create`方法. 

```
var observable = Rx.Observable.create(function (observer) {
  observer.next(1);//发网球
  observer.next(2);
  observer.next(3);
  ...
});
```
上面代码中function的参数`observer`有三个方法可以使用分别是
* next 
Observer 提供一个 next 方法来接收数据.
* complete 
当不再有新的数据发出时，将触发 Observer的complete 方法.
* error
当在处理事件中出现异常报错时，Observer提供error 方法来接收错误进行统一处理.

它们的关系可以用以下正则表达式来描述
```
next*(error|complete)?
```
next函数在complete或error之前可以执行任意次, complete和error最多只能执行一次, 而且它们两只能有一个被执行. 
```
var observable = Rx.Observable.create(function (observer) {
  observer.next(1);
  observer.error('error!');
  //or
  observer.complete('complete!');
});
```

### Observer
观察者，对 Observables 发出的数据进行消费的角色. 以网球的例子来说, Observer就是网球运动员, 时刻观察网球机发出的网球, 准备随时接球, 它是由几个回调函数组成的对象, 这几个回调函数与我们之前提到的next, complete, error一一对应. 我们可以通过以下两种方式来创建并使用一个Observer.
```
var observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
observable.subscribe(observer);
//or
observable.subscribe(
  x => console.log('Observer got a next value: ' + x),
  err => console.error('Observer got an error: ' + err),
  () => console.log('Observer got a complete notification')
);
```
第二种方式其实是在subscribe方法内部创建了一个observer对象. 另外, RxJs并没有强制要求必须提供对三个事件的回调函数. 换句话说, 你可以忽略其中的某些事件.
```
var observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
};
``` 
[Demo Of Observable and Observer](https://jsbin.com/qusilu/1/edit?html,js,console)

### Subscription
之前的代码, `Observer.subscribe`方法的返回值就是一个Subscription对象. 它有一个很重要的方法unsubscribe, 用来释放资源或者结束Observable的执行. 类似于removeEventListener的感觉.

```
//每0.2s发出一个新值 0 => 1 => 2 => 3 => ...
var observable = Rx.Observable.interval(200);
var subscription = observable.subscribe(x => console.log(x));

//1s后终止observable的执行
setTimeout(() => {
  subscription.unsubscribe();
}, 1000);
```
[Demo Of Unsubscribe](https://jsbin.com/dahetes/edit?js,console)

只不过subscription还有一个`add`函数, 可以把两个subscription组合到一起, 组合后的subscription可以同时被unsubscribe.
```
var observable1 = Rx.Observable.interval(300);
var observable2 = Rx.Observable.interval(400);

var subscription = observable1.subscribe(x => console.log('first: ' + x));
var childSubscription = observable2.subscribe(x => console.log('second: ' + x));

subscription.add(childSubscription);

setTimeout(() => {
  // Unsubscribes BOTH subscription and childSubscription
  subscription.unsubscribe();
}, 1000);
```
[Demo of Multiple Unsubscribe](https://jsbin.com/jeyigiz/edit?js,console)

和add功能相反的还有一个remove函数. 可以撤销add.

### Subject
Subject 很特殊, 简单来说, 它既是一个Observable也是一个Observer(*但是Subject和Observable有着非常重要的区别, 后文解释*), 开发者可以在Subject对象上调用subscribe()来消费收到的数据, 也可以调用next(), complete(), error()主动发送数据. 就像你对着墙壁练习兵乓球, 你既可以随时发球, 也可以接球, 当然这个例子不太恰当, 但大概就是那么个意思.
```
var subject = new Rx.Subject();

//消费数据
subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});
subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

//发送数据
subject.next(1);
subject.next(2);

//输出
"observerA: 1"
"observerB: 1"
"observerA: 2"
"observerB: 2"
```
除了上面这种Subject类型, 还有三种其他的类型. 它们分别是BehaviorSubject, ReplaySubject, and AsyncSubject. 但我不想把这篇简介搞得太复杂, 所以如果你感兴趣, 你可以在这里了解.

### Operators
用来处理数据流的函数我们称之为Operator, 这也是RxJs的强大所在, 开发者有非常多的Operators可以使用, 处理不同的场景需求, 例如 .map(...), .filter(...), .merge(...), 可以链式地调用他们： clickStream.map(f).filter(g). 

通过[构建流式应用—RxJs详解](https://github.com/joeyguo/blog/issues/11)中RxJs实现前端搜索的例子可以感受到它的牛逼之处了. 但这仅仅是冰山一角. RxJs还提供了许多其他实用的[Operators](http://reactivex.io/documentation/operators).

改进一下第一步的点击事件, 每次点击输出点击的横坐标乘以十倍

```
Rx.Observable.fromEvent(document, 'click')
	.pluck('clientX')//提取clientX属性
	.map(x => x * 10)//乘以十
    .subscribe(x => console.log(x));//输出
```

![fromevent.gif](http://upload-images.jianshu.io/upload_images/2261833-14f0561a151ab0a0.gif?imageMogr2/auto-orient/strip)

## Schedulers

Schedulers是一个强有力的机制来处理并发性. 通过改变它们的并发模型，你可以很细腻的掌握一个Observable是如何发射数据的. 
```
var timeStart = Date.now();

Rx.Observable.from([10, 20, 30]).subscribe(
  function onNext() {},
  function onError() {},
  function onCompleted() {
    console.log('Total time: ' + (Date.now() - timeStart) + 'ms');
  }
);

console.log('After subscribe');
//输出
"Total time: 1ms"
"After subscribe"
```
```
var timeStart = Date.now();

Rx.Observable.from([10, 20, 30], Rx.Scheduler.async).subscribe(
  function onNext() {},
  function onError() {},
  function onCompleted() {
    console.log('Total time: ' + (Date.now() - timeStart) + 'ms');
  }
);

console.log('After subscribe');
//输出
"After subscribe"
"Total time: 7ms"
```
上面两端代码唯一的区别在于第二段代码在创建Observable的时候加入了并发模型 `Rx.Observable.from([10, 20, 30], Rx.Scheduler.async)`, 模型内部使用了setInterval, 改变了JS线程执行代码的顺序. 

可用的Scheduler类型
* null
* Rx.Scheduler.queue
* Rx.Scheduler.asap
* Rx.Scheduler.async


## 参考列表
[http://reactivex.io/rxjs/manual/overview.html#introduction](http://reactivex.io/rxjs/manual/overview.html#introduction)
[http://hao.jser.com/archive/9081/?utm_source=tuicool&utm_medium=referral](http://hao.jser.com/archive/9081/?utm_source=tuicool&utm_medium=referral)
[http://reactivex.io/documentation/operators](http://reactivex.io/documentation/operators)
[Observables Under The Hood](https://netbasal.com/javascript-observables-under-the-hood-2423f760584)
