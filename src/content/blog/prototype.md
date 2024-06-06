---
title: '原型、原型链与继承'
description: '原型、原型链、继承、类'
pubDate: 'Sept 12 2023'
heroImage: '../../assets/images/te.jpg'
category: 'Basic'
tags: ['JavaScript']
---

## 从对象创建讲起

### 工厂模式

- 创建对象
- 给对象赋值
- 返回赋值完的对象

`无法说明创建对象的类型`

### 构造函数

- 没有显式创建对象
- 赋值
- this指向调用实例
- 没有显式返回对象

`通过Constructor标明对象类型，定义的方法存在同名不可复用问题`

### 原型

- 构造函数prototype指向原型对象
- 原型对象Constructor指回构造函数
- 实例对象`__proto__`属性指向原型对象
- 实例对象与原型对象有直接联系，同构造函数没有
- 定义在原型上的方法及属性在实例对象中共享，且由于Constructor的存在能够标明对象类型
- 若原型对象是另一个父对象的实例，原型对象的`__proto__`指向父对象的原型对象,否则默认为Object的实例
- 检索属性顺序：实例对象➡原型对象➡原型对象的原型对象直到Object的原型对象null
- instanceof 及 isProperty 是基于原型链的判断

## 再到继承

> JavaScript的继承只有实现继承

### 原型链

> 原理是将父构造函数的实例作为子构造函数的原型对象
>
> SubType.prototype = new SuperType();
>
> 将使得Subtype.prototype的`__proto__`属性指向SuperType.prototype，共享SuperType原型上的属性及方法
>
> 注意这个时候SubType实例的类型还是SubType，不会发生类型丢失

缺点在于只能继承父构造函数**原型**上的属性和方法，定义在父构造函数的属性和方法无法继承

### 盗用构造函数

> 本质是在子构造函数中调用父构造函数并改变this指向
>
> function SubType(){
>
> SuperType.call(this);
>
> }
>
> 使得SubType实例能够继承SuperType构造函数上的属性和方法

缺点在于Subtype继承的属性和方法都是一致的，无法根据实际情况初始构建

### 组合继承

> 本质是：原型链 + 盗用构造函数 = 组合继承

问题在于父构造函数的属性同时出现在子类实例及子类原型对象上，只是由于调用顺序原型对象上的属性检索不需要使用

### 原型式继承

Object.create()做的事情：

- 改了`__proto__`
- 改了Conctructor，即定义了对象类型，如果用在继承上实际是导致Constructor指向错误

```javascript
function object(o) {
	function F() {}
	F.prototype = o
	return new F()
}
function Person() {
	this.name = '人'
}
const person1 = new Person()
console.log(person1) // Person {name:'人'}
const person2 = object(Person.prototype)
console.log(person2) // F {}
```

发生了类型丢失，object等价于Object.create()实际用于对象拷贝

实现继承需要手动补充prototype的constructor指向

### 寄生式继承

如果不修复子类的构造函数指向，会导致以下后果：

1. 构造函数指向错误：子类的 `constructor` 属性会指向父类而不是子类本身。这意味着当你通过子类创建新对象时，`constructor` 属性会指向父类的构造函数，而不是子类的构造函数。这可能会导致一些意外的行为或错误。
2. 类型判断错误：由于构造函数的指向错误，使用 `instanceof` 运算符进行类型判断时可能会出现错误结果。如果不修复构造函数指向，`instanceof` 运算符可能会认为子类的实例是父类的实例，而不是子类的实例。

通过修复子类的构造函数指向，即 `ChildClass.prototype.constructor = ChildClass`，可以确保构造函数指向子类本身，从而避免上述问题。这样做可以确保正确的类型判断和行为，以及更好的代码可读性和维护性。

### 寄生式组合继承

> 实际是原型式继承+修复了Constructor丢失+给子构造函数加属性强化然后返回罢了

```javascript
// 父类
function Animal(name) {
	this.name = name
}

Animal.prototype.sayName = function () {
	console.log('My name is ' + this.name)
}

// 子类
function Dog(name, breed) {
	Animal.call(this, name)
	this.breed = breed
}

// 创建一个继承自 Animal 的对象，并在原型上添加额外的方法
function createDog(name, breed) {
	var dog = Object.create(Animal.prototype)
	dog.constructor = Dog
	dog.name = name
	dog.breed = breed

	dog.bark = function () {
		console.log('Woof!')
	}

	return dog
}

// 创建一个 Dog 对象
var myDog = createDog('Max', 'Labrador')
myDog.sayName() // 输出: My name is Max
myDog.bark() // 输出: Woof!
```

## 最后到ES6的类

> 本质还是原型+构造函数的封装

## 对象拷贝

> 区分继承/复用和深浅拷贝(不保留原始构造关系,通常拷贝出来的对象只是Object的实例)

深拷贝操作并不会保留原始对象的构造函数。如果需要在深拷贝后的对象中保留原始的构造函数，可以手动设置 `constructor` 属性，例如 `copyPerson.constructor = Person`。

看看AI的回答：

深拷贝和原型继承是两种不同的概念和用途，它们的区别如下：

1. 定义和目的：深拷贝是创建一个对象的完全独立副本，副本中的所有属性值都是原对象属性的副本。深拷贝的目的是为了在不影响原对象的情况下，创建一个与原对象完全独立的对象。原型继承是通过原型链建立对象之间的关联关系，一个对象可以继承另一个对象的属性和方法。
2. 复制方式：深拷贝是通过递归复制对象的所有属性值，包括嵌套对象和数组。深拷贝后的对象和原对象是完全独立的，修改其中一个对象的属性不会影响另一个对象。原型继承是通过将一个对象设置为另一个对象的原型来建立对象之间的关联。继承后的对象会共享原型对象的属性和方法，修改其中一个对象的属性可能会影响其他对象。
3. 对象关系：深拷贝后的对象是完全独立的，它与原对象没有任何关联，它们是两个独立的对象。原型继承建立了对象之间的关联，继承后的对象可以访问原型对象的属性和方法，原型对象也可以通过原型链访问继承对象的属性。
4. 使用场景：深拷贝适用于需要创建一个与原对象完全独立的对象，例如在修改对象时需要保留原始对象的副本，或者在传递对象时需要避免引用传递。原型继承适用于需要创建一个对象，并使其继承另一个对象的属性和方法，从而实现代码重用和继承关系的场景。

需要注意的是，深拷贝和原型继承都有各自的优缺点和适用场景。在选择使用哪种方式时，需要根据具体的需求和情况进行考虑。深拷贝可能会导致性能上的损耗，特别是对于嵌套层级很深的对象或包含大量数据的对象。原型继承可以方便地实现属性和方法的继承，但也可能导致对象之间的耦合性增加。

深拷贝

```javascript
function deepClone(obj) {
	if (obj === null || typeof obj !== 'object') {
		return obj
	}

	let clone = Array.isArray(obj) ? [] : {}

	for (let key in obj) {
		if (obj.hasOwnProperty(key)) {
			clone[key] = deepClone(obj[key])
		}
	}

	return clone
}
```
