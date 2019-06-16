## Javascript继承的多种方式

实现js继承的方式有以下几种：

* 原型链继承
* 构造函数继承
* 组合继承
* 原型式继承
* 寄生式继承
* 寄生式 + 组合（完美继承）

![](./img/extend.png)

### 原型链继承

原型链是指每个构造函数都有一个prototype属性，该属性指向一个对象，而通过该构造函数创建的实例的原型，就是该对象。那么基于原型链的继承，可以用下列方式来实现：

```js
const Parent = function() {
  this.say = function() {
    console.log("I'm parent.");
  };
};
const Child = function() {}
const extend = function(ChildClass, SuperClass) {
  ChildClass.prototype = new SuperClass();
};
extend(Child, Parent);
const childInstance = new Child();
```

优点：

* 子类的实例沿着原型链可以访问父类的方法或属性，同时能通过`instanceof`的检测

缺点：

* 子类无法传递参数给父类
* 父类引用类型的属性会被所有子类的实例共享

## 构造函数继承

主要是借助`call`或`apply`实现，然后把子类的`this`指针传入。

```js
const Parent = function(name) {
  this.say = function() {
    console.log("I'm " + name + ".");
  };
};
const Child = function() {
  Parent.apply(this, arguments);
};
const childInstance = new Child("child");
```

优点：

* 支持传递参数给父类
* 父类引用类型的属性不会被所有子类的实例共享

缺点：

* 无法通过`instanceof`的检测

### 组合继承

既然原型链继承与构造函数继承各有优缺点，那么我们整合它们的优势，于是就有`组合继承`这种方式：

```js
const Parent = function(name) {
  this.skills = ["tennis", "football"]
  this.say = function() {
    console.log("I'm " + name + ".");
  };
  this.addSkills = function(skill) {
    this.skills.push(skill);
  }
};
const Child = function() {
  Parent.apply(this, arguments);
}
const extend = function(ChildClass, SuperClass) {
  ChildClass.prototype = new SuperClass();
};
extend(Child, Parent);
const childInstance1 = new Child("child1");
const childInstance2 = new Child("child2");
childInstance2.addSkills("basketball");
console.log(childInstance1.skills); // ["tennis", "football"]
console.log(childInstance2.skills); // ["tennis", "football", "basketball"]
console.log(childInstance1 instanceof Parent); // true
```
但是这种方式仍然有不足的地方：

* 实现每个子类的继承，父类都会被调用两次，如果父类的实现很复杂的话，会造成消耗问题。

另外，这里或许有人会疑惑，好像`ChildClass.prototype = SuperClass.prototype`也可以实现。但是会有个问题，在`ChildClass.prototype`上增强方法时，也会添加到父类的prototype，而实例化父类则不用有这种问题（这里需要理解在js中，new做了什么事情）。

### 原型式继承

原型式继承利用传入的对象作为创建的对象的原型，主要用到了`Object.create`这个方法，其内部的实现也可以用es5去模拟，但是还是存在一些区别（可以去另外一篇文章了解）：

```js
const createObject = function(o) {
  const F = function() {}
  F.prototype = o;
  return new F();
};
const parent = {
  say: function() {
    console.log("I'm parent.");
  }
}
const child = createObject(parent));
```

缺点：

* 子类无法传递参数给父类
* 本质上是实现浅复制，父类对象的引用类型的属性会被子类共享

### 寄生式继承

简单来说其实是原型式继承实现的封装，并不是什么继承。并可以增强子类的属性或方法。

```js
const createObject = function(o) {
  const F = function() {}
  F.prototype = o;
  return new F();
};

const extend = function(o) {
  const obj = createObject(o);
  obj.play = function() {};
  return obj;
};
```

缺点跟原型式一样

### 寄生组合式继承（完美继承）：

我们知道，组合式继承已经近乎完美了，只是在实现继承时，会调用两次父类，因此可以组合这两种方式。React就是继承就是使用这种方式。

```js
const Parent = function(name) {
  this.skills = ["tennis", "football"]
  this.say = function() {
    console.log("I'm " + name + ".");
  };
  this.addSkills = function(skill) {
    this.skills.push(skill);
  };
};

const Child = function() {
  Parent.apply(this, arguments);
};

const createObject = function(o) {
  if (typeof Object.create === 'function') {
    return Object.create(o);
  } else {
    const F = function() {}
    F.prototype = o;
    return new F();
  }
};

const extend = function(child, SuperClass) {
  const prototype = createObject(SuperClass.prototype);
  child.prototype = prototype;
  child.constructor = SuperClass;
};

extend(Child, Parent);
const childInstance1 = new Child("child1");
const childInstance2 = new Child("child2");
childInstance2.addSkills("basketball");
console.log(childInstance1.skills); // ["tennis", "football"]
console.log(childInstance2.skills); // ["tennis", "football", "basketball"]
console.log(childInstance1 instanceof Parent); // true
```

