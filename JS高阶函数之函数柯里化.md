
### 什么是函数柯里化？

维基百科上说道：

柯里化(Currying)，是把接受多个参数的函数，变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术

一个简单的例子
```
  // 假设现在有一个 add 函数
  function add(x, y) {
    return x + y;
  }
  
  // 变成函数柯里化形式
  function add_2(x) {
    return function (y) {
      return x + y
    }
  }
  
  // 效果就是
  let func_1 = add_2(1);
  let result = func_1(2);
```
柯里化其实是一种分步处理问题的方式

### 柯里化有什么用？

现在有一个疑问，为什么一个函数写的好好的，要写成这种这么绕的方式呢？将多个参数全部丢进一个函数中处理完得到一个结果，不香吗？

#### 提前确认，复用结果

有时候有这样的场景，一个函数在传入多个参数后，内部有一个步骤的执行都是一样的，并且跟别的参数没有关系；

这样的话每次调用这个函数，这部分可能就会无意义的重复执行，如果能把这部分先执行了，之后只使用它的执行结果来执行其他的步骤岂不是更好。

所以这种场景就如下：

```
  // 非柯里化函数 on，每次调用都要重复判断兼容环境
  var on = function(element, event, handler) {
      if (document.addEventListener) {
          if (element && event && handler) {
              element.addEventListener(event, handler, false);
          }
      } else {
          if (element && event && handler) {
              element.attachEvent('on' + event, handler);
          }
      }
  }

  // 柯里化函数 on, 环境判断已经执行过一次，之后使用会直接调用内部 return 的函数
  var on = (function() {
      if (document.addEventListener) {
          return function(element, event, handler) {
              if (element && event && handler) {
                  element.addEventListener(event, handler, false);
              }
          };
      } else {
          return function(element, event, handler) {
              if (element && event && handler) {
                  element.attachEvent('on' + event, handler);
              }
          };
      }
  })();
  
```
#### 参数复用

比如正则匹配的例子

```
  // 正则匹配函数
  function check(reg, txt) {
      return reg.test(txt)
  }

  check(/\d+/g, 'test')       //false
  check(/[a-z]+/g, 'test')    //true

  // Currying后，第一步传入不同的正则，获得了专用于匹配不同类型参数的函数
  function curryingCheck(reg) {
      return function(txt) {
          return reg.test(txt)
      }
  }

  // 语义化后功能更加明显
  var hasNumber = curryingCheck(/\d+/g)
  var hasLetter = curryingCheck(/[a-z]+/g)

  hasNumber('test1')      // true
  hasNumber('testtest')   // false
  hasLetter('21212')      // false
  
```
#### bind() 方法的实现

首先明白 bind 方法的作用：修改函数内部 this 指向，接收原函数的部分参数，返回一个接收剩余参数并且返回计算结果的新函数

实现自己的 bind 方法：
```
  FUNCTION.prototype.new_bind() { // 这里不锁死输入的参数，主要是为了实现不限定参数个数，第一个参数是新的 this 指向，其他的参数是先输入的原函数的部分参数
    
    let origin_fun = this; // bind 函数里面的 this 是调用 bind 函数的原函数
    let new_this = Array.prototype.slice.call(arguments, 0, 1); // 获取新的 this 指向
    let some_params = Array.prototype.slice.call(arguments, 1); // 获取其他输入参数
    
    return function() {
      let left_params = Array.prototype.slice.call(arguments); // 获取剩余输入参数
      origin_fun.apply(new_this, some_params.concat(left_params))
    }
  }
  
```

既然有函数柯里化的应用场景，有没有什么方法可以任意函数转化成柯里化后的函数的方法呢？蔽日一个通用处理函数。

### 柯里化通用函数

第一种，将原函数的参数分成两部分,第一部分在第一步输入，剩余参数在第二步输入

```
  function Curry(fn){ // fn 表示即将被柯里化的函数
    let some_params = Array.prototype.slice.call(arguments, 1); // 传入柯里化的部分参数
    return function () {
      let left_params = Array.prototype.slice.call(arguments); // 接收剩余参数
      return fn.apply(undefined,left_params.concat(some_params))
    }
  }
```
首先是初步封装,通过闭包把初步参数给保存下来，然后通过获取剩下的arguments进行拼接，最后执行需要currying的函数。

第二种，原函数有多少个参数，可以柯里化成多个步骤，直到所有的参数都传入，后自动计算出结果

```
  function Curry(fn){
        let params_length = fn.length; // 原函数参数的总个数
        let params = Array.prototype.slice.call(arguments, 1) || []; // 获取柯里化时传入的部分参数
        
        let return_func = function() {
            let some_params = Array.prototype.slice.call(arguments) || []; // 接收新传入的参数
            params = params.concat(some_params);
            if (params.length >= params_length){
                return fn.apply(undefined, params);
            }
            return return_func;
        }
        return return_func;
    }

```

### 函数柯里化涉及到的性能点

1、柯里化函数中都是用 arguments 来获取参数，相比于形参速度较慢

2、使用 call、apply 等方法相对于直接调用函数慢一些

3、柯里化函数存在闭包环境，需要保存闭包状态，需要额外内存空间

### 一道与函数柯里化类似的面试题

设计一个函数实现以下 add 函数效果，可以实现任意次传参，生成的结果即可以直接但会结果，也可以继续传参：
```
  add(1)(2)(3) = 6;
  add(1, 2, 3)(4) = 10;
  add(1)(2)(3)(4)(5) = 15;
```
设计如下：
```
  function add() {
    let params = Array.prototype.slice.call(arguments); // 获取第一次传入的所有参数，并报错
    let inner_add = function(){
      params = params.concat(Array.prototype.slice.call(arguments));
      return inner_add;
    }
    
    inner_add.toString = function() {
      // 这里只是计算和，可以不用 reduce
      return params.reduce(function(a, b) {
        return a + b;
      });
    }
    
    return inner_add;
  }
```
可以顺便复习一下函数这个数据类型以及函数的 toString 方法

