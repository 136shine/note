# 创建对象

## 工厂模式
 ```javaScript
function Person(name, age) {
    var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.sayName = function(){
        console.log(this.name);
    };
    return obj;
};

// 实例化
var p1 = Person('xx', 18);
console.log(p1.name)
console.log(p1.sayName())

```

## 构造函数

```javaScript
function Person(name, age) {
	this.name = name;
	this.age = age;
	this.sayName = function () {
		console.log(this.name)
	}
}

// 实例化
var p2 = new Person('xx', 18);
console.log(p2.name)
console.log(p2.sayName())
```

# 继承

## 借用构造函数 (对象冒充)
核心：将父类构造函数的内容复制给了子类的构造函数。这是所有继承中唯一一个不涉及到prototype的继承。
在子类函数中，通过call／apply 方法调用父类函数后，子类实例 child1, 可以访问到 Person 构造函数和它所有属性和方法。这样就实现了子类向父类的继承。
```
Parent.call(Child);
```
优点：和原型链继承完全相反
1. 父类的引用属性不会被共享
2. 子类构建实例时可以向父类传递参数
缺点：父类的方法不能复用，子类实例的方法每次都是单独创建的，浪费内存。

```javaScript
function Parent(name, age) {
	this.name = name;
	this.age = age;
	this.habby = ['1', '2']
}
Parent.prototype.sayName = function () {
	console.log(this.name)
}

function Child(name, age, job) {
	Parent.call(this, name, age)
	this.job = job
}
Child.prototype.sayName = function () {
	console.log(this.name)
}

// 实例化
var child1 = new Child('xx1', 18, 'coding18')
var child2 = new Child('xx2', 19, 'coding19')

console.log(child1.name) // xx1
console.log(child1.job)  // coding18
child1.sayName()		 // xx1
console.log(child2.name) // xx2
console.log(child2.job)  // coding19
child2.sayName()		 // xx2

console.log(child1.habby) // ['1', '2']
console.log(child2.habby) // ['1', '2']
console.log(child1.habby === child2.habby) // false

console.log(child2 === child1) // false
console.log(child2.sayName === child1.sayName) // true
```

## 原型链 
核心：将父类的实例作为子类的原型
``` javaScript
Child.prototype = new Parent() 
// 所有涉及到原型链继承的继承方式都要修改子类构造函数的指向，否则子类实例的构造函数会指向 Child。
Child.prototype.constructor = Child;
```

优点：父类方法可以复用

缺点：
1. 父类的引用属性会被所有子类实例共享
2. 子类构建实例时不能向父类传递参数

```javaScript
function Parent() {
	this.name = 'xx1';
	this.age = 18;
	this.job = ['job1', 'job2']
}
Parent.prototype.sayName = function () {
	console.log(this.name)
}

function Child(){}
Child.prototype = new Parent()
Child.prototype.constructor = Child

// 实例化
var child1 = new Child()
var child2 = new Child()

console.log(child1.name) // xx1
console.log(child1.job)  // ['job1', 'job2']
child1.sayName() 		 // xx1

console.log(child2.name) // xx1
console.log(child2.job)  // ['job1', 'job2']
child2.sayName() 		 // xx1

console.log(child1 === child2) // false
console.log(child1.name === child2.name) // true
console.log(child1.job === child2.job)   // true

child1.job.push('job3')
console.log(child1.job) // ['job1', 'job2', 'job3']
console.log(child2.job) // ['job1', 'job2', 'job3']
console.log(child1.job === child2.job)   // true

child1.job = ['job1', 'job2', 'job3'] // 重新赋值，不是继承自父级
console.log(child1.job) // ['job1', 'job2', 'job3']
console.log(child2.job) // ['job1', 'job2']
console.log(child1.job === child2.job)   // false
```

## 组合式继承
优点：
1. 父类的方法可以被复用
2. 父类的引用属性不会被共享
3. 子类构建实例时可以向父类传递参数

缺点：  
调用了两次父类的构造函数，第一次给子类的原型添加了父类的属性，第二次又给子类的构造函数添加了父类的属性，从而覆盖了子类原型中的同名参数。这种被覆盖的情况造成了性能上的浪费。
```javaScript
function Person(name, age) {
	this.name = name
	this.age = age
	this.job = ['job1', 'job2']
}
Person.prototype.sayName = function () {
	console.log(this.name)
}

function Child(name, age, habby) {
	// 属性继承
	Person.call(this, name, age) // 第二次调用 Person
	// 自有属性
	this.habby = habby
}

// 方法继承
Child.prototype = new Person() // 第一次调用 Person
Child.prototype.constructor = Child
Child.prototype.sayName2 = function () {
	console.log(this.name)
}

// 实例化
var child1 = new Child('xx1', 18, ['habby1', 'habby2'])
var child2 = new Child('xx2', 19, ['habby1', 'habby2'])

console.log(child1.name) // xx1
console.log(child1.job)  // ['job1', 'job2']
child1.sayName()		 // xx1

console.log(child2.name) // xx2
console.log(child2.job)  // ['job1', 'job2']
child2.sayName()		 // xx2

console.log(child1.name === child2.name) // false
console.log(child1.job === child2.job) // false 子类实例的属性每次都是单独创建的
console.log(child1.sayName === child2.sayName) // true 原型上的方法共有

console.log(child1.job) // ['job1', 'job2']
console.log(child2.job) // ['job1', 'job2']
console.log(child1.job === child2.job)   // false
child1.job.push('job3')
console.log(child1.job) // ['job1', 'job2', 'job3']
console.log(child2.job) // ['job1', 'job2']
console.log(child1.job === child2.job)   // false
```

## 寄生组合式继承
寄生组合式继承 = 寄生式继承 + 借用构造函数继承。通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。  
基本思想：在借用构造函数实现实例属性继承的基础上，使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。  
目的：为了解决组合式继承产生的两次调用父类（超类型）构造函数  
借用构造函数实现属性继承，通过原型链实现方法共用，同时消除了组合继承两次调用父级构造函数的缺点
```javaScript
function Person(name, age) {
	this.name = name
	this.age = age
	this.job = ['job1', 'job2']
}
Person.prototype.sayName = function () {
	console.log(this.name)
}

function Child(name, age, habby) {
	Person.call(this, name, age)
	this.habby = habby
}

// Child.prototype = new Person()
function inheritObj(parent) {  
	function F(){} 
	F.prototype = parent
	return new F() 
}

function inheritPrototype(childObj, parentObj) {
	var f = inheritObj(parentObj.prototype) //创建对象 创建超类型原型的一个副本
	f.constructor = childObj // 增强对象 为创建的副本添加constructor属性，从而弥补因重写原型而失去的默认的constructor 属性
	childObj.prototype = f //指定对象 将新创建的对象(即副本)赋值给子类型的原型
}

inheritPrototype(Child, Person) // 相当于 childObj.prototype = parentObj.prototype

// 实例化
var child1 = new Child('xx1', 18, ['habby1', 'habby2'])
var child2 = new Child('xx1', 18, ['habby1', 'habby2'])

console.log(child1.name) // xx1
console.log(child1.job)	 // ['job1', 'job2']
child1.sayName()		 // xx1

console.log(child1 === child2)  // false
console.log(child1.job === child2.job)	 // false 引用类型 引用指针存储的栈区 不同
console.log(child1.name === child2.name)  // true 基本类型 比较值 值相同

child1.job.push('job3')
console.log(child1.job) // ['job1', 'job2', 'job3']
console.log(child2.job) // ['job1', 'job2']
console.log(child1.job === child2.job)   // false
```


**ES5 VS ES6 继承**
* ES5中继承的实质是：（那种经典寄生组合式继承法）
    * 先由子类（SubClass）构造出实例对象this
    * 然后在子类的构造函数中，将父类（SuperClass）的属性添加到this上，SuperClass.apply(this, arguments)
    * 子类原型（SubClass.prototype）指向父类原型（SuperClass.prototype）
    * 所以instance是子类（SubClass）构造出的（所以没有父类的[[Class]]关键标志）
    * 所以，instance有SubClass和SuperClass的所有实例属性，以及可以通过原型链回溯，获取SubClass和SuperClass原型上的方法
* ES6中继承的实质是：
    * 先由父类（SuperClass）构造出实例对象this，这也是为什么必须先调用父类的super()方法（子类没有自己的this对象，需先由父类构造）
    * 然后在子类的构造函数中，修改this（进行加工），譬如让它指向子类原型（SubClass.prototype），这一步很关键，否则无法找到子类原型（注，子类构造中加工这一步的实际做法是推测出的，从最终效果来推测）
    * 然后同样，子类原型（SubClass.prototype）指向父类原型（SuperClass.prototype）
    * 所以instance是父类（SuperClass）构造出的（所以有着父类的[[Class]]关键标志）
    * 所以，instance有SubClass和SuperClass的所有实例属性，以及可以通过原型链回溯，获取SubClass和SuperClass原型上的方法


ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。（ES5通过原型模式的继承：创建子类，然后将父类的实例赋值给子类原型，也就是重写子类原型，代之以一个新类型的实例。）
ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。


-----------------------------------

#【更新】笔记整理&迁移
根据自己的理解加以解释

# 面向对象
1. 封装：对一些属性的隐藏域暴露 （js 通过函数实现）
2. 多态：以不同的方式调用方法（通过参数区分）
3. 继承：一个引用类型从另一个引用类型获取相应的属性和方法

## 创建对象
### 1.对象字面量
```
var obj1 = {
  a: 1,
  b: {
    b1: 'bbb'
  }
}
```

### 2. 通过 new 实例化构造函数
```
var obj2 = new Object()
obj2.name = 'xx'
```

## JS 继承
### 原型链继承
缺点：
* 父类中的引用类型属性被子类的所有实例共享
* 子类实例化时，不能向父类传递参数

优点：
* 父类方法可以复用
```
function Parent() {
  this.name = 'Parent name'
  this.age = '30'
  this.work = ['1', '2', '3']
}

Parent.prototype.getName = function() {
  console.log(this.name)
}

function Child(hobby) {
  this.hobby = hobby
  this.name = 'Child name'
}

Child.prototype = new Parent()
Child.prototype.constructor = Child
Child.prototype.getHobby = function () {
  console.log(this.hobby)
}


var child1 = new Child(['play1-1', 'play1-2'])
var child2 = new Child(['play2-1', 'play2-2'])
child1.getName()
child2.getHobby()
child2.hobby.push('ooo')
child1.getHobby()
child2.getHobby()
```
	
### 构造函数
缺点：
* 父类的原型方法，子类不可见【父类方法不能复用/子类无法继承父类原型上的属性和方法】

优点：
* 父类的引用属性不会被所有子类实例共享【子类共享父类引用属性】
* 子类实例化的时候，可以向父类传参
```
function Parent (name, age, works) {
  this.name = name
  this.age = age
  this.works = works
}

Parent.prototype.getName = function() {
  console.log(this.name)
}

function Child (name, age, works, hobby) {
  Parent.call(this, name, age, works)
  this.hobby = hobby
}
Child.prototype.getName = function() {
  console.log(this.name)
}

var child3 = new Child('child1 name', 20, ['play1-1', 'play1-2'], 'game')
var child4 = new Child('child2 name', 21, ['play2-1', 'play2-2'], 'game')

console.log(child3.getName === child4.getName)
```
		
### 组合式继承
优点：
* 父类原型方法被继承（复用）
* 子类可以向父类传参
* 父类引用属性不会被子类共享 

缺点：
* 父类构造函数被调用两次，子类原型上的属性被构造函数上的同名属性覆盖

```
function Parent(name, age, works) {
  this.name = name
  this.age = age
  this.works = works
}
Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child(name, age, works, hobby) {
  Parent.call(this, name, age, works) 【2】
  this.hobby = hobby
}
Child.prototype = new Parent() 【1】
Child.prototype.constructor = Child

var child1 = new Child('child1 name', 20, ['play1-1', 'play1-2'], 'game')
```

### 寄生组合式继承
```
function Parent(name, age, works) {
  this.name = name
  this.age = age
  this.works = works
}

Parent.prototype.getName = function() {
  console.log(this.name)
}

function Child(name, age, works, hobby) {
  Parent.call(this, name, age, works)
  this.hobby = hobby
}

Child.prototype = Object.create() ? Object.create(Parent.prototype) : inhertObj(Parent.prototype)
Child.prototype.constructor = Child


// 兼容 Object.create()
function inhertObj(obj) {
  function F(){}
  F.prototype = obj
  return new F()
}

var child1 = new Child()
```


参考：
[两张图看懂ES5和ES6中的继承，值得收藏](https://baijiahao.baidu.com/s?id=1593627663270143849&wfr=spider&for=pc)
[一篇文章理解JS继承——原型链/构造函数/组合/原型式/寄生式/寄生组合/Class extends](https://segmentfault.com/a/1190000015727237)
[ES6类以及继承的实现原理](https://segmentfault.com/a/1190000014798678)
[ES6中的继承剖析](https://www.cnblogs.com/kable/p/5705552.html)
[如何继承Date对象？由一道题彻底弄懂JS继承](https://segmentfault.com/a/1190000012841509#articleHeader14)
