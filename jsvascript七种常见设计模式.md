# Javascript常见设计模式

> 设计模式总的来说是一个抽象的概念，是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

## 1、工厂模式

假设有一份很复杂的代码需要用户去调用，但是用户并不关心这些复杂的代码，只需要你提供给我一个接口去调用，用户只负责传递需要的参数，至于这些参数怎么使用，内部有什么逻辑是不关心的，只需要你最后返回我一个实例。这个构造过程就是工厂。

工厂模式是我们最常用的实例化对象模式，是用工厂方法代替new操作的一种模式。

```jsx
function Animal(opts){
    var obj = new Object();
    obj.color = opts.color;
    obj.name= opts.name;
    obj.getInfo = function(){
        return '名称：'+ onj.name+'， 颜色：'+ obj.color;
    }
    return obj;
}
var cat = Animal({name: '波斯猫', color: '白色'});
cat.getInfo();
```

你可能有个疑问，为什么不使用 new 关键字配合构造函数直接创建对象，而是要借助工厂方法呢？

原因在于构造函数对于对象整体的创建过程能力有限，有时候需要交给一个具有更高视野的工厂来处理。这些场景包括创建过程涉及到对象缓存，对象共享或复用，复杂逻辑或者对象需要和不同的资源和设备打交道。如果你的应用程序需要控制对象的创建过程，可以考虑用工厂模式。

---

## 2、单例模式

在它的核心结构中只包含一个被称为单例的特殊类。通过单例模式可以保证系统中，应用该模式的一个类只有一个实例。

```jsx
class Singleton {
  constructor() {}
}

Singleton.getInstance = (function() {
  let instance
  return function() {
    if (!instance) {
      instance = new Singleton()
    }
    return instance
  }
})()

let s1 = Singleton.getInstance()
let s2 = Singleton.getInstance()
console.log(s1 === s2) // true
```

在vuex源码中，通过一个外部变量来控制全局只有一个Vue实例：

```jsx
let Vue // bind on install
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    // 如果发现 Vue 有值，就不重新创建实例了
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```

---

## 3、适配器模式

适配器用来解决两个接口不兼容的情况，不需要改变已有的接口，通过包装一层的方式实现两个接口的正常协作。

```jsx
// 已有的接口
var googleMap = {
    show: function(){
        console.log( '开始渲染谷歌地图' );
    }
};
var baiduMap = {
    display: function(){
        console.log( '开始渲染百度地图' );
    }
};
var renderMap = function( map ){
    if ( map.show instanceof Function ){
        map.show();
    }
};

// 目的
renderMap( googleMap ); // 开始渲染谷歌地图
renderMap( baiduMap ); // 无效

// 适配器
var baiduMapAdapter = {
    show: function(){
        return baiduMap.display();

    }
};
renderMap( googleMap ); // 开始渲染谷歌地图
renderMap( baiduMapAdapter ); // 开始渲染百度地图
```

---

## 4、装饰模式

装饰模式不需要改变已有的接口，作用是给对象添加功能。比如使用ES7 中的装饰器语法：

```jsx
function readonly(target, key, descriptor) {
  descriptor.writable = false
  return descriptor
}

class Test {
  @readonly
  name = 'yck'
}

let t = new Test()
t.yck = '111' // 不可修改
```

---

## 5、代理模式

在某些情况下，一个对象不适合或者不能直接引用另一个对象，而代理对象可以在两个对象之间起到中介的作用。

```jsx
class Car {
    drive() {
        return "driving";
    };
}

class Driver {
    constructor(age) {
        this.age = age;
    }
}

class CarProxy {
    constructor(driver) {
        this.driver = driver;
    }
    drive() {
        return  ( this.driver.age < 18) ? "too young to drive" : new Car().drive();
    };
}
```

此外，还有es6 proxy，也就是在目标对象之前架设一层拦截，或者叫代理：

```jsx
var obj = {}
var proxy = new Proxy(obj, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
proxy.count = 1
//  setting count!
++proxy.count
//  getting count!
//  setting count!
//  2
console.log(obj.count) // 2
```

---

## 6、观察者模式

定义了对象间一对多的依赖关系，当目标对象 Subject 的状态发生改变时，所有依赖它的对象 Observer 都会得到通知。

```jsx
// 目标者类
class Subject {
  constructor() {
    this.observers = [];  // 观察者列表
  }
  // 添加
  add(observer) {
    this.observers.push(observer);
  }
  // 删除
  remove(observer) {
    let idx = this.observers.findIndex(item => item === observer);
    idx > -1 && this.observers.splice(idx, 1);
  }
  // 通知
  notify() {
    for (let observer of this.observers) {
      observer.update();
    }
  }
}

// 观察者类
class Observer {
  constructor(name) {
    this.name = name;
  }
  // 目标对象更新时触发的回调
  update() {
    console.log(`目标者通知我更新了，我是：${this.name}`);
  }
}

// 实例化目标者
let subject = new Subject();

// 实例化两个观察者
let obs1 = new Observer('前端开发者');
let obs2 = new Observer('后端开发者');

// 向目标者添加观察者
subject.add(obs1);
subject.add(obs2);

// 目标者通知更新
subject.notify();  
// 输出：
// 目标者通知我更新了，我是前端开发者
// 目标者通知我更新了，我是后端开发者
```

vue源码中的依赖管理和通知更新就是基于观察者模式。

---

## 7、发布订阅模式

实现了对象间多对多的依赖关系，通过事件中心管理多个事件。目标对象并不直接通知观察者，而是通过事件中心来派发通知。

```jsx
// 事件中心
let pubSub = {
  list: {}, // {onwork:[fn1,fn2],offwork:[fn1,fn2],launch:[fn1,fn2]}
  subscribe: function (key, fn) {   // 订阅
    if (!this.list[key]) {
      this.list[key] = [];
    }
    this.list[key].push(fn);
  },
  publish: function(key, ...arg) {  // 发布
    for(let fn of this.list[key]) {
      fn.call(this, ...arg);
    }
  },
  unSubscribe: function (key, fn) {     // 取消订阅
    let fnList = this.list[key];
    if (!fnList) return false;

    if (!fn) {
      // 不传入指定取消的订阅方法，则清空所有key下的订阅
      fnList && (fnList.length = 0);
    } else {
      fnList.forEach((item, index) => {
        if (item === fn) {
          fnList.splice(index, 1);
        }
      })
    }
  }
}

// 订阅
pubSub.subscribe('onwork', time => {
  console.log(`上班了：${time}`);
})
pubSub.subscribe('offwork', time => {
  console.log(`下班了：${time}`);
})
pubSub.subscribe('launch', time => {
  console.log(`吃饭了：${time}`);
})

// 发布
pubSub.publish('offwork', '18:00:00'); 
pubSub.publish('launch', '12:00:00');

// 取消订阅
pubSub.unSubscribe('onwork');
```