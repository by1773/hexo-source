---
title: Rxjs中的Subjects
date: 2020-03-01 11:15:08
tags: ['Rxjs','Subjects','Angular','响应式编程']
categories: Rxjs
keywords: [Rxjs,Subjects,Angular]
top_img : https://goss.cfp.cn/creative/vcg/nowarter800/version2/gic11154705.jpg?x-oss-process=image/format,webp
cover : https://rxjs-dev.firebaseapp.com/generated/images/marketing/home/Rx_Logo-512-512.png
---

#彻底搞懂RxJS中的Subjects

&emsp;&emsp;每周大约有1700万次npm下载，RxJS在JavaScript世界中非常受欢迎。如果您是Angular开发人员，则不会错过RxJS Observables，但您可能对Subjects不太熟悉。虽然它们不像简单的Observable被频繁使用，但还是非常有用的。了解它们将帮助我们编写更好，更简洁的响应式代码。
## Observables

>直观地，我们可以将Observables视为发出值流的对象，或者按照RxJS文档所述：Observables是多个值的惰性Push集合。

例如，我们可以使用Observables每秒发出0到59之间的数字：

```
import { Observable } from 'rxjs';
const observable = new Observable((subscriber) => {
  for (let i = 0; i < 60; i += 1) {
    setTimeout(() => {
      subscriber.next(i);
    }, i * 1000);
  }
});
observable.subscribe((value) => {
  console.log(`Observer receives: ${value}`);
});
```

&emsp;&emsp;需要订阅Observable才能开始计数，这与调用函数的方式相同。同样类似于函数，第二个"调用"将触发新的独立执行。如果两秒钟后再次订阅此Observable，我们将在控制台中看到两个"计数器"，第二个计数器有两秒钟的延迟。
```
import { Observable } from 'rxjs';
    
const observable = new Observable((subscriber) => {
  for (let i = 0; i < 60; i += 1) {
    setTimeout(() => {
      subscriber.next(i);
    }, i * 1000);
  }
});

console.log('First observer subscribes');
observable.subscribe((value) => {
  console.log(`First observer receives: ${value}`);
});

setTimeout(() => {
  console.log('Second observer subscribes');
  observable.subscribe((value) => {
    console.log(`Second observer receives: ${value}`);
  });
}, 2000);
```

&emsp;&emsp;这意味着我们不能同时向两个观察者发出相同的值，至少不能使用简单的Observable。因此，需要Subject。
## Subject

>Subject就像一个可观察对象，但是可以多播到许多观察者。

&emsp;&emsp;Subject也是可观察的。我们可以使用Subject创建每秒发射0到59的相同计数器：

```
import { Subject } from 'rxjs';

const subject = new Subject();

console.log('Observer subscribes');
subject.subscribe((value) => {
  console.log(`Observer receives: ${value}`);
});

for (let i = 0; i < 60; i += 1) {
  setTimeout(() => {
    subject.next(i);
  }, i * 1000);
}
```

&emsp;&emsp;您可能会发现我们之前的示例的主要区别。在声明一个Observable时，我们提供了一个函数作为参数，告诉Observable向用户发出什么。可以，因为每个新订户都将开始新的执行。另一方面，在这种情况下，我们只有一个执行，而新订户只是开始“监听”它。我们只需使用new Subject（）创建一个新对象。
&emsp;&emsp;我们也可以订阅主题，因为主题是可观察的。然后，我们直接调用主题，因为主题是观察者。
&emsp;&emsp;任何新订户将被添加到主题在内部保留的订户列表中，并且同时将获得与其他订户相同的值。如果我们在第一次订阅后两秒钟订阅主题，则新订阅者将错过前两个值：
```
import { Subject } from 'rxjs';

const subject = new Subject();

console.log('First observer subscribes');
subject.subscribe((value) => {
  console.log(`First observer receives: ${value}`);
});

setTimeout(() => {
  console.log('Second observer subscribes');
  subject.subscribe((value) => {
    console.log(`Second observer receives: ${value}`);
  });
}, 2000);

for (let i = 0; i < 60; i += 1) {
  setTimeout(() => {
    subject.next(i);
  }, i * 1000);
}
```
我们可以使用Subject一次向多个观察者发出值。
## BehaviorSubject

&emsp;&emsp;Subject可能存在的问题是，观察者将仅收到订阅主题后发出的值。
在上一个示例中，第二个发射器未接收到值0、1和2。有时，我们需要在订阅该对象之前，知道该对象最后一次发射了哪个值。例如，如果我们发出日期，情况就是这样。任何在3月1日订阅的观察者，无论何时订阅，都将获得3月1日的订阅。在午夜，每个订阅者都会收到日期已更改的通知。
&emsp;&emsp;对于这种情况，可以使用BehaviorSubject。BehaviorSubject保留其发出的最后一个值的内存。订阅后，观察者立即接收到最后发出的值。如果我们改编前面的示例，这意味着第二个观察者在订阅时收到值2，然后像第一个观察者一样接收之后的所有其他值。

```
import { BehaviorSubject } from 'rxjs';

const behaviorSubject = new BehaviorSubject(0);

for (let i = 1; i < 60; i += 1) {
  setTimeout(() => {
    behaviorSubject.next(i);
  }, i * 1000);
}

console.log('First observer subscribes');
behaviorSubject.subscribe((value) => {
  console.log(`First observer receives: ${value}`);
});

setTimeout(() => {
  console.log('Second observer subscribes');
  behaviorSubject.subscribe((value) => {
    console.log(`Second observer receives: ${value}`);
  });
}, 2000);
```

&emsp;&emsp;您可能已经在示例中注意到，我们需要为BehaviorSubject提供一个初始值，而Subject则不需要。这是因为BehaviorSubject始终需要当前值。
## ReplaySubject

&emsp;&emsp;ReplaySubjects与BehaviorSubjects非常相似。所不同的是，他们不仅记住了最后一个值，还记住了之前发出的多个值。订阅后，它们会将所有记住的值发送给新观察者。
在创建时不给它们任何初始值，而是定义它们应在内存中保留多少个值。在示例中，我们保留两个值：
```
import { ReplaySubject } from 'rxjs';

const replaySubject = new ReplaySubject(2);

for (let i = 0; i < 60; i += 1) {
  setTimeout(() => {
    replaySubject.next(i);
  }, i * 1000);
}

console.log('First observer subscribes');
replaySubject.subscribe((value) => {
  console.log(`First observer receives: ${value}`);
});

setTimeout(() => {
  console.log('Second observer subscribes');
    replaySubject.subscribe((value) => {
      console.log(`Second observer receives: ${value}`);
    });
}, 2000);
```

&emsp;&emsp;当第二个观察者订阅ReplaySubject时，已经发出0、1和2。由于ReplaySubject保留了最后两个值，第二个观察者立即收到1和2。
## AsyncSubject

&emsp;&emsp;使用AsyncSubjects，在主题完成之前，观察者实际上什么也没收到。
```
import { AsyncSubject } from 'rxjs';

const asyncSubject = new AsyncSubject();

console.log('First observer subscribes');
asyncSubject.subscribe((value) => {
  console.log(`First observer receives: ${value}`);
});

setTimeout(() => {
  console.log('Second observer subscribes');
  asyncSubject.subscribe((value) => {
    console.log(`Second observer receives: ${value}`);
  });
}, 2000);

for (let i = 0; i < 60; i += 1) {
  setTimeout(() => {
    asyncSubject.next(i);
    if (i === 59 ) {
      asyncSubject.complete();
    }
  }, i * 1000);
}
```

&emsp;&emsp;在我们的示例中使用AsyncSubject，我们必须等待一分钟，然后观察者才能收到东西。
&emsp;&emsp;我们必须完成主题。如果不这样做，我们的观察者将一无所获。
&emsp;&emsp;在AsyncSubject完成后订阅的任何观察者将收到相同的值。
```
import { AsyncSubject } from 'rxjs';

const asyncSubject = new AsyncSubject();

console.log('First observer subscribes');
asyncSubject.subscribe((value) => {
  console.log(`First observer receives: ${value}`);
});

setTimeout(() => {
  console.log('Second observer subscribes');
  asyncSubject.subscribe((value) => {
    console.log(`Second observer receives: ${value}`);
  });
}, 2000);

for (let i = 0; i < 60; i += 1) {
  setTimeout(() => {
    asyncSubject.next(i);
    if (i === 59 ) {
      asyncSubject.complete();
    }
  }, i * 1000);
}
  
setTimeout(() => {
  console.log('Third observer subscribes');
  asyncSubject.subscribe((value) => {
    console.log(`Third observer receives: ${value}`);
  });
}, 65000);
```
&emsp;&emsp;在此示例中，第三个观察者在AsyncSubject完成五秒钟后对其进行订阅。订阅时，它将收到最后一个值：59。
&emsp;&emsp;这使得AsyncSubjects对于获取和缓存值很有用，例如HTTP响应，我们只希望获取一次，但是以后可以从其他位置进行访问。
## 最后
&emsp;&emsp;自己尝试这些示例并对其进行修改，以了解其如何影响结果。对RxJS主题的深入了解将有助于我们在响应式编程方面编写更具可读性和更高效的代码。