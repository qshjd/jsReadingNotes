## 6.2创建对象

* 工厂模式
* 构造函数模式
* 原型模式

### 工厂模式
在创建多个属性相同的对象时，使用字面量来直接创建每个对象，会产生大量的重复代码，降低代码的可维护性和阅读性，工厂模式是一种常见的设计模式，它抽象出了创建具体对象的过程，用函数来封装创建对象的细节。

*示例*
``` JavaScript
function createPerson(name, age) {
    var obj = new Object()
    obj.name = name
    obj.age = age
    obj.sayHi = function() {
        console.log(`hi Im ${this.name}`)
    }
    return obj
}
let s1 = createPerson('raymond',21)
```

**小结：**在使用工厂模式来创建一批具体的对象时，在createPerson内部使用了Object的构造函数来创建一个新对象，然后返回。但工厂模式也存在一点问题：没有解决对象识别的问题，无法知道生成对象的具体类型。后面又出现了构造函数模式。

### 构造函数模式

*示例*
``` JavaScript
function Person(name, age) {
    this.name = name
    this.age = age
    this.sayHi = function() {
        console.log(`hi I'm ${this.name}`)
    }
}
let s2 = new Person('kiki', 18)
let s3 = new Person('koko', 66)
```

在上面代码中，Person有一个prototype（原型）属性，这个属性是一个指针，指向了原型对象：Person.prototype，在s2中有一个__proto__属性也指向了该原型对象，这就解决了对象识别的问题，使用**s2 instanceof Person**可以得到s2是否是Person构造函数的实例。

**小结：**在创建构造函数时，一般使用大写字母开头来表示它是一个构造函数，通过new来实例化它的对象，但构造函数也可以作为普通函数被调用，当在全局上被直接调用时this对象会指向window对象。构造函数模式虽然解决了对象识别的问题，但是还是存在一些问题，主要问题是每个方法都要在构造函数中重新创建一遍
``` JavaScript
s2.sayHi() === s3.sayHi() //false
```
两个对象中相同的方法是不相等的，相同方法没必要每个实例都创建一次。解决方案：原型模式

### 原型模式
创建的每一个函数都有一个prototype属性，也就是原型对象，原型对象可以让所有实例共享它所包含的属性和方法。

*示例*
``` JavaScript
function Animal() {}
Animal.prototype.name = 'dog'
Animal.prototype.action = function() {
    console.log(`${this.name} can run `)
}
let a1 = new Animal()
a1.action()
let a2 = new Animal()
a2.name = 'cat'
a2.action()
```

**构造函数、原型对象、实例之间的关系**
在构造函数中，Animal.prototype指向了原型对象
在原型对象中，Animal.prototype.constructor又指回了构造函数
实例中，包含了一个属性[[prototype]]指向了Animal.prototype(原型对象)
可以看出，构造函数与其生成的实例是由原型对象串联起来的

**对象识别的方法**
使用原型对象的**isPrototypeof()**
``` JavaScript
console.log(Animal.prototype.isPrototypeOf(a1))   //true
```
返回指定对象的原型**Object.getPrototypeOf()**
``` JavaScript
console.log(Object.getPrototypeOf(a1) === Animal.prototype)  //true
```

**实例属性与原型属性的关系**
通过实例访问属性时，首先搜索实例本身有没有这个属性，有就返回，没有就向原型对象上查找。可以通过对象实例来访问原型对象的属性，但是不能够改变它，只能覆盖。

判断属性是位于实例对象还是原型中
1、通过**hasOwnProperty()**可以区分访问的是实例属性还是原型属性
2、通过in操作符 可以判断某个属性是否位于实例或原型中
综合两个方法可以判断一个属性是位于实例对象中还是原型中
``` JavaScript
function hasPrototypeProperty(object, name) {
    return !object.hasOwnProperty(name) && (name in object)
}
```

**小结：**原型对象也存在一些不足，问题恰恰是共享导致的，如果共享了包含引用类型的值，对于实例上同名属性的修改会影响到原型对象上的属性，从而造成其它实例访问的值也变了。

### 组合使用构造函数模式和原型模式
构造函数模式的缺点是没有共享能力，而原型模式的缺点是一些不能被共享的属性也被共享出去了，所以可以两者组合使用来达到效果

``` JavaScript
function Person(name, age) {
    this.name = name
    this.age = age
}
Person.prototype = {
    constructor: Person,
    sayHi() {
        console.log(`hello I'm ${this.name}`)
    }
}
```




