---
title: 博大精深的this
date: 2018-02-11 04:00:23
tags:
  - JS
---

# about this

前端开发避免不了this

- this是JS语言中的关键字

- 总的原则是：**this永远指向的是调用这个函数的域**

本篇以例子为主，阐述this的指向、如何改变this的指向、运用到实际开发中。基本目录如下：
<div style="width: 300px">
  {% asset_img p1.png %}
</div>

<!--more-->

## 一、全局的this（浏览器）

- 当没有明确的执行时的当前对象时，指向全局对象Window

  指向全局对象Window <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
  ```js
    console.log(this); // window
    console.log(this.document); // #document

    this.a = 37;
    console.log(window.a); // 37
  ```

  指向全局对象Window <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
  ```js
    var a = 'hello'
    function test() {
      console.log('this.a:', this.a)
    }
    test(); // 'hello'

  ```

- 进阶版 <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
  ```js
    var name = 'window';
    var person1 = {
      name: 'Bob',
      show: function() {
        console.log(this.name)
      }
    }
    var test1 = person1.show
    test1();  // 'window'

    var test2 = person1.show();  // 'Bob'

  ```
  这个很容易和对象的方法相混淆
  可以理解为test是Window对象下的方法，所以执行时当前的对象是Window

## 二、一般函数的this（浏览器）

非严格模式下，指向全局对象 <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
```js
  function f1() {
    return this;
  }
  f1();   // window

  function f2() {
    "use strict"
    return this;
  }
  f2();   // undefined
```

## 三、作为对象方法的函数this

指向上级的对象 <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
```js
  var obj1 = {
    prop: 37,
    f: function(){
      return this.prop
    }
  };

  console.log(obj1.f());  //37

  var obj2 = {
    prop: 37
  };

  function independent(){
    return this.prop
  };

  obj2.func = independent;

  console.log(this);  // window
  console.log(obj2.func());  //37
```

## 四、对象原型链上的this

```js
  var obj = {
    f:  function() {
      return this.a + this.b
    }
  };

  var p = Object.create(obj);  // 对象P的原型指向obj

  p.a = 1;
  p.b = 4;

  console.log(p.f());  // 5 调用了原型的方法
```

## 五、构造器中的this

所谓构造函数，就是通过这个函数生成一个新的对象(object)
这时，this就指向这个新对象 <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
```js
  function MyClass() {
    this.a = 37;
  }

  var obj = new MyClass();

  console.log(obj.a);  //37

  function C2() {
    this.a = 37;
    return {a : 38};
  }

  var obj2 = new C2();

  console.log(obj2.a);  //38
```

注意，不要与this指向全局对象混淆 <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
```js
  var a = 'aa'
  var b = 'bb'

  // 这是一个混淆视听的函数
  function MyClass1() {
    this.a = 'AA'
    console.log('this.a:', this.a)
  }
  // 这是一个构造函数
  function MyClass2() {
    this.b = 'BB'
    console.log('this.b:', this.b)
  }

  MyClass1()  // this.a: AA，全局对象
  var obj = new MyClass2()  // this.b: BB，构造函数
  console.log('this.b:', this.b)  // this.b: bb，全局对象。
```
运行结果为bb，证明全局变量b的值根本没变

- **类的方法内部含有this，this默认指向类的实例。但如果把类的方法提取出来使用，this指向该方法运行时的环境** <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
  ```js
  class Point {
    toString() {
      this.toNumber()
    }

    toNumber(){
      console.log('xxx')
    }
  }

  var point = new Point()

  const {toString} = point

  point.toString();  // 'xxx'

  toString();  // TypeError: this.toNumber is not a function
  ```

  那如何改变this的指向呢？运用到了下面的方法
    - 方法一，在构造方法中绑定this  <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
      ```js
      class Point {
        constructor() {
          this.toString = this.toString.bind(this)
        }
        // ...
      }
      ```
    - 方法二，使用箭头哦函数  <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>
      ```js
      class Point {
        toString = () => {
          this.toNumber()
        }
        // ....
      }
      ```

## 六、注意点

- this的指向是**执行时的指向**，而不是定义时的指向

  ```js
    var obj1 = {
      prop: 37,
      f: function(){
        return this.prop
      }
    };

    var obj2 = {
      prop: 38,
      f: obj1.f
    }

    console.log(obj2.f());  // 38
  ```
  虽然this是在obj1声明的，但运行的时候是在obj2，所以this指向obj2

# 修改this指向的域

## 一、call/apply

- 传递上下文环境，具体的来说就是传递this的指向
- 是函数对象的方法，改变函数的调用对象
- 第一个参数：调用这个函数的对象。为空时，默认调用全局对象

call和apply的区别：
- call: 直接传参数
- apply: 把数组作为参数传进去

- 例子1（改变this的指向）

  ```js
    var obj = {
      a: 1,
      b: 2
    };
    var a = 3;
    var b = 4;

    function add (c, d) {
      if (!c) {
        c = 0
      }

      if (!d) {
        d = 0
      }

      return this.a + this.b + c + d
    };

    var test = add.call(obj, 5, 7);

    var test1 = add.apply(obj, [10, 7]);

    var test2 = add.call()

    var test3 = add.apply()

    console.log(test);  // 15
    console.log(test1);  // 20
    console.log(test2);  // 7
    console.log(test3);  // 7
  ```
  `add.call(obj, 5, 7)` add方法执行时，通过call改变了this的指向，this指向了`obj`

- 例子2（改变this的指向）

  ```js
    var pet = {
      words: '...',
      speak: function(say){
        console.log(say + ' ' + this.words)
      }
    }
    var dog = {
      words: 'wang'
    }
    pet.speak.call(dog, 'hello')  // hello wang
  ```
  `pet.speak.call(dog, 'hello')`：`call`方法改变了`this`的指向，this指向了`dog`

- 例子3（继承构造函数）

  ```js
    function Pet(words) {
      this.words = words
      this.speak = function(){
        console.log(this.words)
      }
      console.log('Pet this:', this)  // Pet this: Dog {words: "wang", speak: ƒ}
    }

    function Dog(words){
      // 使用call方法继承构造函数Pet，此时执行环境是在 Dog 下，所以指针指向 Dog
      Pet.call(this, words)
      console.log('Dog this:', this)  // Dog this: Dog {words: "wang", speak: ƒ}
    }

    var dog = new Dog('wang')

    dog.speak()  // 'wang'
  ```

## 二、bind

- 改变函数的调用对象

```js
  function func() {
    return this.a;
  }

  var g = func.bind({
    a: "test"
  });

  console.log(g());  // test

  var obj = {
    a: 37,
    func: func,
    g: g
  }

  console.log(obj.func());  // 37
  console.log(obj.g());  // "test"
```

## 三、使用箭头函数(arrow function)

箭头函数是 ES6 新增的一种函数

- 箭头函数this的指向，与本篇的相反
  > 箭头函数的this，总是指向定义时所在的对象，而不是运行时所在的对象
  > 箭头函数没有自己的this，导致箭头函数内部的this就是外层代码块的this
  > 箭头函数没有自己的this，所以不用使用call()、apply()、bind()等改变this的指向

  ```js
    function foo() {
      setTimeout(() => {
        console.log('id:', this.id);
      }, 100);
    }

    var id = 21;

    foo.call({ id: 42 });  // 42
  ```
  setTimeout的第一个参数是箭头函数，它是在100秒之后才执行。如果是普通的函数，this指向全局对象Window，此时的值是21；但箭头函数的this总是指向定义时的对象，即{id: 42}，所以结果是 42

- 作为经历了`func.bind(this)`和箭头函数的人，箭头函数真的是一大利器
  - 从此不用bind、bind、bind了
  - `var that = this`这种hack写法也不需要了

# 应用

## 定时器、匿名函数、箭头函数、异步

- **在浏览器中，setTimeout/setInterval执行时的当前对象是Window**
  在现实开发数据模拟中，经常使用setTimeout/setInterval模拟异步，所以可以得出，**异步执行时的当前对象也是Window** <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>

  ```js
    var a = 'a'

    var obj = {
      a: 'A',
      say: function () {
        setTimeout(function() {
          console.log(this.a);
        }, 50)
      }
    }

    obj.say()  // 'a'
  ```

  如果想要obj.say()执行的结果是 A 呢？。。。有很多方法，开发中也经常使用 <span style="display: inline-block; color: #6BD9EE; margin: 0 0 0 10px">▼</span>

  ```js
  // 方法一，使用箭头函数
  var a = 'a'

  var obj = {
    a: 'A',
    say: function () {
      setTimeout(() => {
        console.log(this.a);
      }, 50)
    }
  }

  obj.say()  // 'A'

  // 方法二，把this赋值给变量
  var a = 'a'

  var obj = {
    a: 'A',
    say: function () {
      var that = this;

      setTimeout(function() {
        console.log(that.a);
      }, 50)
    }
  }

  obj.say();  // 'A'
  ```

- **匿名函数执行时的当前对象是Window**
  ```js
    var a = 'a'

    var obj = {
      a: 'A',
      say: function() {
        console.log(this.a)
      },
      showSay: function () {
        // 匿名函数
        (function(callback) {
          callback()
        })(this.say)
      }
    }

    obj.say();  // 'A'
    obj.showSay();  // 'a'
  ```
  - 当obj.showSay执行时，触发了匿名函数的执行，匿名函数把obj.say作为参数传递给回调函数，然后回调函数执行
  - 由于匿名函数的当前对象是Window，所以当回调函数执行时，this指向全局对象。打印出来的是'a'

# 参考文档
- [Javascript的this用法](http://www.ruanyifeng.com/blog/2010/04/using_this_keyword_in_javascript.html)，阮一峰，2010年4月30日
- [Javascript中this关键字详解](http://www.cnblogs.com/justany/archive/2012/11/01/the_keyword_this_in_javascript.html)，杨文坚，2012年11月1日
- [箭头函数](http://es6.ruanyifeng.com/#docs/function)，阮一峰
- [箭头函数](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001438565969057627e5435793645b7acaee3b6869d1374000)，廖雪峰
