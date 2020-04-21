
### 什么是高阶函数？

一个函数可以接收另一个函数作为参数，这种函数就称之为高阶函数。

一个最简单的高阶函数：

```
  'use strict';

  function add(x, y, f) {
      return f(x) + f(y);
  }
  var x = add(-5, 6, Math.abs); // 11
  console.log(x);
```
### 常见的高阶函数

下面以 array 的几个内置函数为例说明高阶函数

#### map 函数

map 函数接受一个函数作为参数，该函数将依次接收数组的每一项 value, index，以及数组本身作为参数，对数组中的每一个元素进行处理，最后得到一个新的数组

例如，将数组中的每一个元素进行平方，输出一个新数组
```
  'use strict';

  function pow(x) {
      return x * x;
  }
  var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
  var results = arr.map(pow); // [1, 4, 9, 16, 25, 36, 49, 64, 81]
  console.log(results);
  
```
> 注意，因为 map 函数会在传入的函数中，依次传入 value ，index，以及原数组作为参数，所以 map 在传入函数参数时，一定要注意该函数定义了哪些形参以及含义

#### reduce 函数

Array的reduce()把一个函数作用在这个Array的[x1, x2, x3...]上，这个函数必须接收两个参数，reduce()把结果继续和序列的下一个元素做累积计算，其效果就是：

```
  [x1, x2, x3, x4].reduce(f) = f(f(f(x1, x2), x3), x4)
```
要把 [1, 3, 5, 7, 9] 变换成整数 13579

```
  var arr = [1, 3, 5, 7, 9];
  arr.reduce(function (x, y) {
      return x * 10 + y;
  }); // 13579
```
#### filter 函数

Array 的 filter() 也接收一个函数。filter() 把传入的函数依次作用于每个元素，然后根据返回值是 true 还是 false 决定保留还是丢弃该元素。

利用filter，可以巧妙地去除Array的重复元素：

```
  'use strict';
  var r;
  var arr = ['apple', 'strawberry', 'banana', 'pear', 'apple', 'orange', 'orange', 'strawberry'];
  r = arr.filter(function (element, index, self) {
    return self.indexOf(element) === index;
  });

```
当然高阶函数只是一个概念，其用法还有更过的空间可以给我们探索。
