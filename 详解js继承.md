## **第一部分：预备知识**

### **1、构造函数的属性**

```
funcion A(name) {
    this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
    this.arr = [1]; // 实例引用属性 (该属性，强调私用，不共享)
    this.say = function() { // 实例引用属性 (该属性，强调复用，需要共享)
     console.log('hello')
    }
}

注意：数组和方法都属于‘实例引用属性’，但是数组强调私有、不共享的。方法需要复用、共享。
     在构造函数中，一般很少有数组形式的引用属性，大部分情况都是：基本属性 + 方法。
```

### 2、什么是原型对象

简单来说，每个函数都有prototype属性，它就是原型对象，通过函数实例化出来的对象有个__proto__属性，指向原型对象。

```
let a = new A()
a.__proto__ == A.prototype

// prototype的结构如下
A.prototype = {
	constructor: A,
  ...其他的原型属性和方法
}
```

### 3**、原型对象的作用**

原型对象的用途是为每个实例对象存储共享的方法和属性，它仅仅是一个普通对象而已。**并且所有的实例是共享同一个原型对象**，**因此有别于实例方法或属性，原型对象仅有一份**。而实例有很多份，且实例属性和方法是独立的。在构造函数中：为了属性(实例基本属性)的私有性、以及方法(实例引用属性)的复用、共享。我们提倡：

- 将属性封装在构造函数中
- 将方法定义在原型对象上

```
funcion A(name) {
    this.name = name; // (该属性，强调私有，不共享)
}
A.prototype.say = function() { // 定义在原型对象上的方法 (强调复用，需要共享)
   console.log('hello')
}

// 不推荐的写法：[原因](https://blog.csdn.net/kkkkkxiaofei/article/details/46474303)
A.prototype = {
    say: function() { 
        console.log('hello')
    }
}
```

## **第二部分：五种js 继承方式**

### **方式1、原型链继承**

- 核心：将父类实例作为子类原型
- 优点：方法复用
    - 由于方法定义在父类的原型上，复用了父类构造函数的方法。比如say方法。
- 缺点：
    - 创建子类实例的时候，不能传父类的参数（比如name）。
    - 子类实例共享了父类构造函数的引用属性，比如arr属性。
    - 无法实现多继承。

```
function Parent(name) {
    this.name = name || '父亲';  // 实例基本属性 (该属性，强调私有，不共享)
		this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function() { // -- 将需要复用、共享的方法定义在父类原型上 
    console.log('hello')
}
function Child(like) {
    this.like = like;
}

Child.prototype = new Parent() // 核心，但此时Child.prototype.constructor==Parent
Child.prototype.constructor = Child // 修正constructor指向

let boy1 = new Child()
let boy2 = new Child()

// 优点：共享了父类构造函数的say方法
console.log(boy1.say(), boy2.say(), boy1.say === boy2.say); // hello , hello , true

// 缺点1：不能向父类构造函数传参
console.log(boy1.name, boy2.name, boy1.name===boy2.name); // 父亲，父亲，true

// 缺点2: 子类实例共享了父类构造函数的引用属性，比如arr属性
boy1.arr.push(2); 
// 修改了boy1的arr属性，boy2的arr属性，也会变化，因为两个实例的原型上(Child.prototype)有了父类构造函数的实例属性arr；
所以只要修改了boy1.arr，boy2.arr的属性也会变化。
console.log(boy2.arr); // [1,2]

注意1：修改boy1的name属性，是不会影响到boy2.name。因为设置boy1.name相当于在子类实例新增了name属性。

注意2：
console.log(boy1.constructor); // Parent 你会发现实例的构造函数居然是Parent。
而实际上，我们希望子类实例的构造函数是Child,所以要记得修复构造函数指向。
修复如下：Child.prototype.constructor = Child;
```

### **方式2、借用构造函数**

- 核心：借用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类。
- 优点：实例之间独立。
    - 创建子类实例，可以向父类构造函数传参数。
    - 子类实例不共享父类构造函数的引用属性。如arr属性
    - 可实现多继承（通过多个call或者apply继承多个父类）
- 缺点：
    - 父类的方法不能复用

    由于方法在父构造函数中定义，导致方法不能复用(因为每次创建子类实例都要创建一遍方法)。比如say方法。(方法应该要复用、共享)

    - 子类实例，继承不了父类原型上的属性。(因为没有用到原型)

```
function Parent(name) {
    this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
    this.arr = [1]; // (该属性，强调私有)
    this.say = function() { // 实例引用属性 (该属性，强调复用，需要共享)
      console.log('hello')
    }
}
function Child(name,like) {
    Parent.call(this,name);  // 核心 拷贝了父类的实例属性和方法
    this.like = like;
}
let boy1 = new Child('小红','apple');
let boy2 = new Child('小明', 'orange ');

// 优点1：可向父类构造函数传参
console.log(boy1.name, boy2.name); // 小红， 小明
// 优点2：不共享父类构造函数的引用属性
boy1.arr.push(2);
console.log(boy1.arr,boy2.arr);// [1,2] [1]

// 缺点1：方法不能复用
console.log(boy1.say === boy2.say) // false (说明，boy1和boy2的say方法是独立，不是共享的)

// 缺点2：不能继承父类原型上的方法
Parent.prototype.walk = function () {   // 在父类的原型对象上定义一个walk方法。
  console.log('我会走路')
}
boy1.walk;  // undefined (说明实例，不能获得父类原型上的方法)
```

### **方式3、组合继承**

- 核心：通过调用父类构造函数，继承父类的属性并保留传参的优点；然后通过将父类实例作为子类原型，实现函数复用。
- 优点：
    - 保留构造函数的优点：创建子类实例，可以向父类构造函数传参数。
    - 保留原型链的优点：父类的方法定义在父类的原型对象上，可以实现方法复用。
    - 不共享父类的引用属性。比如arr属性
- 缺点：
    - 由于调用了2次父类的构造方法，会存在一份多余的父类实例属性，具体原因见文末。
- 注意：'组合继承'这种方式，要记得修复Child.prototype.constructor指向

第一次Parent.call(this);从父类拷贝一份父类实例属性，作为子类的实例属性，第二次Child.prototype = new Parent();创建父类实例作为子类原型，Child.protype中的父类属性和方法会被第一次拷贝来的实例属性屏蔽掉，所以多余。

```
function Parent(name) {
    this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
    this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function() { // --- 将需要复用、共享的方法定义在父类原型上 
    console.log('hello')
}
function Child(name,like) {
    Parent.call(this,name,like) // 核心   第二次
    this.like = like;
}
Child.prototype = new Parent() // 核心   第一次

Child.prototype.constructor = Child // 修正constructor指向

let boy1 = new Child('小红','apple')
let boy2 = new Child('小明','orange')

// 优点1：可以向父类构造函数传参数
console.log(boy1.name,boy1.like); // 小红，apple
// 优点2：可复用父类原型上的方法
console.log(boy1.say === boy2.say) // true
// 优点3：不共享父类的引用属性，如arr属性
boy1.arr.push(2)
console.log(boy1.arr,boy2.arr); // [1,2] [1] 可以看出没有共享arr属性。

// 缺点1：由于调用了2次父类的构造方法，会存在一份多余的父类实例属性
```

其实Child.prototype = new Parent()
console.log(Child.prototype.__proto__ === Parent.prototype); // true 
因为Child.prototype等于Parent的实例，所以__proto__指向Parent.prototype

### **方式4、组合继承优化1**

- 核心：

通过这种方式，砍掉父类的实例属性，这样在调用父类的构造函数的时候，就不会初始化两次实例，避免组合继承的缺点。

- 优点：
    - 只调用一次父类构造函数。
    - 保留构造函数的优点：创建子类实例，可以向父类构造函数传参数。
    - 保留原型链的优点：父类的实例方法定义在父类的原型对象上，可以实现方法复用。
- 缺点：
    - 修正构造函数的指向之后，父类实例的构造函数指向，同时也发生变化(这是我们不希望的)
- 注意：'组合继承优化1'这种方式，要记得修复Child.prototype.constructor指向

原因是：不能判断子类实例的直接构造函数，到底是子类构造函数还是父类构造函数。

```
function Parent(name) {
    this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
    this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function() { // --- 将需要复用、共享的方法定义在父类原型上 
  console.log('hello')
}
function Child(name,like) {
    Parent.call(this,name,like) // 核心  
    this.like = like;
}
Child.prototype = Parent.prototype // 核心  子类原型和父类原型，实质上是同一个

<!--这里是修复构造函数指向的代码-->
Child.prototype.constructor = Child

let boy1 = new Child('小红','apple')
let boy2 = new Child('小明','orange')
let p1 = new Parent('小爸爸')

// 优点1：可以向父类构造函数传参数
console.log(boy1.name,boy1.like); // 小红，apple
// 优点2：可复用父类原型上的方法
console.log(boy1.say === boy2.say) // true

// 缺点1：当修复子类构造函数的指向后，父类实例的构造函数指向也会跟着变了。

没修复之前：console.log(boy1.constructor); // Parent
修复代码：Child.prototype.constructor = Child
修复之后：console.log(boy1.constructor); // Child
        console.log(p1.constructor);// Child 这里就是存在的问题(我们希望是Parent)

具体原因：因为是通过原型来实现继承的，Child.prototype的上面是没有constructor属性的，
就会往上找，这样就找到了Parent.prototype上面的constructor属性；当你修改了子类实例的
construtor属性，所有的constructor的指向都会发生变化。
```

### **方式5、组合继承优化2 又称 寄生组合继承 --- 完美方式**

- 核心：

- 优点：完美
- 缺点：---

```
function Parent(name) {
    this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
    this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function() { // --- 将需要复用、共享的方法定义在父类原型上 
    console.log('hello')
}
function Child(name,like) {
    Parent.call(this,name,like) // 核心  
    this.like = like;
}
// 核心  通过创建中间对象，子类原型和父类原型，就会隔离开。不是同一个啦，有效避免了方式4的缺点。
Child.prototype = Object.create(Parent.prototype) 

// 这里是修复构造函数指向的代码
Child.prototype.constructor = Child

let boy1 = new Child('小红','apple')
let boy2 = new Child('小明','orange')
let p1 = new Parent('小爸爸')

注意：这种方法也要修复构造函数的
修复代码：Child.prototype.constructor = Child
修复之后：console.log(boy1.constructor); // Child
				console.log(p1.constructor); // Parent  完美😊
```

## **第三部分：其他相关问题**

### **1、Object.create(object, propertiesObject)**

Object.create()方法创建一个新对象，使用第一个参数来提供新创建对象的__proto__（以第一个参数作为新对象的构造函数的原型对象）；
方法还有第二个可选参数，是添加到新创建对象的属性，写法如下。

```
const a = Object.create(Person.prototype, {
    age: {
      value: 12,
      writable:true,
      configurable:true,
    }
})
```

- new 与 Object.create() 的区别？

new 产生的实例，优先获取构造函数上的属性；构造函数上没有对应的属性，才会去原型上查找；如果构造函数中以及原型中都没有对应的属性，就会报错。Object.create() 产生的对象，只会在原型上进行查找属性，原型上没有对应的属性，就会报错。

```
let Base1 = function() {
  this.a = 1
}
let o1 = new Base1()
let o2 = Object.create(Base1.prototype)
console.log(o1.a); // 1
console.log(o2.a); // undefined

let Base2 = function() {}
Base2.prototype.a = 'aa'
let o3 = new Base2()
let o4 = Object.create(Base2.prototype)
console.log(o3.a); // aa
console.log(o4.a); // aa

let Base3 = function() {
  this.a = 1
}
Base3.prototype.a = 'aa'
let o5 = new Base3()
let o6 = Object.create(Base3.prototype)
console.log(o5.a); // 1
console.log(o6.a); // aa
```

### **2、new 的过程**

- 创建新对象（如obj）。
- 将新对象的_proto_指向构造函数的prototype对象。
- 执行构造函数，为这个新对象添加属性，并将this指向创建的新对象obj。
- 当构造函数本身返回值为对象时，返回该对象，否则返回新对象。

```
//创建Person构造函数，参数为name,age
function Person(name,age){
 　　this.name = name;
 　　this.age = age;
 }

 function _new(){
 　　//1.拿到传入的参数中的第一个参数，即构造函数名Func
　　var Func = [].shift.call(arguments);

 　　//2.创建一个空对象obj,并让其继承Func.prototype
　　var obj = Object.create(Func.prototype);

 　　//3.执行构造函数，并将this指向创建的空对象obj
　　const result = Func.apply(obj,arguments)

 　　//4.当函数也有返回值且为对象时返回该对象，否则返回创建的新对象obj
　　return (result instanceOf Object ? result : obj)
 }

 let ming = _new(Person,'xiaoming',18);
 console.log(ming);
```

[].shift.call表示删除并返回auguments[0]。也可以通过以下方式取得函数名和函数的参数：

```
 function _new(Func, ...params){
 　　...
 }
```

Object.create创建obj，使得obj.__proto__ = Func.prototype

### **3、为什么‘组合继承’这种方式，会执行两次父类构造函数？？**

- 第一次：Child.prototype = new Parent()

‘new 的过程’的第三步，其实就是执行了父类构造函数。

- 第二次：Parent.call(this,name,like)

call的作用是改变函数执行时的上下文。比如：A.call(B)。其实，最终执行的还是A函数，只不过是用B来调用而已。所以，你就懂了Parent.call(this,name,like) ,也就是执行了父类构造函数Person。