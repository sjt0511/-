# Generator 函数

## Generator 函数的语法

### 简介

#### 基本概念

Generator 函数是 ES6 提供的一种**异步编程解决方案**，语法行为与传统函数完全不同（状态机{`yield`定义内部状态}、遍历器对象生成函数、调用`next`分段执行）

Generator 函数有多种理解角度：

- 语法上，首先可以把它理解成，Generator 函数是一个**状态机**，封装了**多个内部状态**
  - 执行 Generator 函数会返回一个**遍历器对象**，也就是说，Generator 函数除了状态机，还是一个**遍历器对象生成函数**
  - 返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态
- 形式上，Generator 函数是一个普通函数，但是有两个特征
  - 一是，`function`关键字与函数名之间有一个星号 => `function*`
  - 二是，函数体内部**使用`yield`表达式，定义不同的内部状态**（yield在英语里的意思就是“产出”）

Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 Generator 函数后，**该函数并不执行**，返回的也不是函数运行结果，而是**一个指向内部状态的指针对象**，也就是遍历器对象（Iterator Object）

必须调用遍历器对象的`next`方法，**使得指针移向下一个状态**。也就是说，每次调用`next`方法，内部指针就**从函数头部或上一次停下来的地方开始执行**，直到遇到下一个`yield`表达式（或`return`语句）为止。换言之，Generator 函数是**分段执行**的，`yield`表达式是**暂停执行**的标记，而`next`方法可以**恢复执行**

``` JS
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();

hw.next()
// { value: 'hello', done: false }
hw.next()
// { value: 'world', done: false }
hw.next()
// { value: 'ending', done: true }
hw.next()
// { value: undefined, done: true }
```

上面代码定义了一个 Generator 函数`helloWorldGenerator`，它内部有两个`yield`表达式（`hello`和`world`），即该函数有**三个状态**：`hello`，`world` 和 `return` 语句（结束执行）

上面代码一共调用了四次`next`方法：

- 第一次调用，Generator 函数开始执行，直到遇到第一个`yield`表达式为止 => `next`方法返回一个对象，它的`value`属性就是当前`yield`表达式的值`hello`，`done`属性的值`false`，表示遍历还没有结束
- 第二次调用，Generator 函数从上次`yield`表达式停下的地方，一直执行到下一个`yield`表达式 => `next`方法返回的对象的`value`属性就是当前`yield`表达式的值`world`，`done`属性的值`false`，表示遍历还没有结束
- 第三次调用，Generator 函数从上次`yield`表达式停下的地方，一直执行到`return`语句（**如果没有`return`语句，就执行到函数结束**）=> `next`方法返回的对象的`value`属性，就是紧跟在`return`语句后面的表达式的值（如果没有`return`语句，则`value`属性的值为`undefined`），`done`属性的值`true`，表示遍历已经结束
- 第四次调用，此时 Generator 函数已经运行完毕 => `next`方法返回对象的`value`属性为`undefined`，`done`属性为`true`，以后再调用`next`方法，返回的都是这个值

总结一下，调用 Generator 函数 => 返回一个`遍历器对象`，代表 Generator 函数的`内部指针` => 以后，每次调用遍历器对象的`next`方法，就会返回一个有着`value`和`done`两个属性的对象 => `value`属性表示当前的内部状态的值，是`yield`表达式后面那个表达式的值；`done`属性是一个布尔值，表示是否遍历结束

ES6 没有规定，`function`关键字与函数名之间的星号，写在哪个位置，这导致下面的写法都能通过。由于 Generator 函数仍然是普通函数，所以一般的写法是上面的第三种，即`function* foo(){}`

``` JS
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· }
function*foo(x, y) { ··· }
```

#### yield表达式

由于 Generator 函数返回的遍历器对象，只有调用`next`方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。`yield`表达式就是**暂停标志**

遍历器对象的`next`方法的运行逻辑如下：

1. 遇到`yield`表达式，就**暂停执行**后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值
2. 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式
3. 如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值
4. 如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`

需要注意的是，`yield`表达式后面的表达式(`yield xxx`的`xxx`)，只有当调用`next`方法、**内部指针指向该语句时才会执行**，因此等于为 JavaScript 提供了手动的“**惰性求值**”（`Lazy Evaluation`）的语法功能

另外需要注意，`yield`表达式只能用在 Generator 函数里面，用在其他地方都会报错

``` JS
// yield后面的表达式123 + 456，不会立即求值，只会在next方法将指针移到这一句时，才会求值
function* gen() {
  yield  123 + 456;
}
```

`yield`表达式与`return`语句:

- 相似之处在于，都能返回紧跟在语句后面的那个表达式的值
- 区别在于每次遇到`yield`，函数**暂停执行**，下一次再从该位置继续向后执行
- 而`return`语句不具备位置记忆的功能
- 一个函数里面，只能执行一次（或者说一个）`return`语句，但是可以执行多次（或者说多个）`yield`表达式
- 正常函数只能返回一个值，因为只能执行一次`return`；Generator 函数可以返回一系列的值，因为可以有任意多个`yield`
- 从另一个角度看，也可以说 **Generator 生成了一系列的值**，这也就是它的名称的来历（英语中，`generator` 这个词是“生成器”的意思）

Generator 函数可以不用`yield`表达式，这时就变成了一个**单纯的暂缓执行函数**

另外需要注意，`yield`表达式只能用在 Generator 函数里面，用在其他地方都会报错

`yield`表达式如果用在另一个表达式之中，必须放在圆括号里面

`yield`表达式用作函数参数或放在赋值表达式的右边，可以不加括号

``` JS
// 暂缓执行函数
function* f() {
  console.log('执行了！')
}
var generator = f();
setTimeout(function () {
  generator.next()
}, 2000);


// 在一个普通函数中使用yield表达式，结果产生一个句法错误
(function (){
  yield 1;
})()
// SyntaxError: Unexpected number


// 会产生句法错误，因为forEach方法的参数是一个普通函数，但是在里面使用了yield表达式（这个函数里面还使用了yield*表达式）
var arr = [1, [[2, 3], 4], [5, 6]];
var flat = function* (a) {
  a.forEach(function (item) {
    if (typeof item !== 'number') {
      yield* flat(item);
    } else {
      yield item;
    }
  });
};
for (var f of flat(arr)){
  console.log(f);
}
// 一种修改方法是改用for循环
var arr = [1, [[2, 3], 4], [5, 6]];
var flat = function* (a) {
  var length = a.length;
  for (var i = 0; i < length; i++) {
    var item = a[i];
    if (typeof item !== 'number') {
      yield* flat(item);
    } else {
      yield item;
    }
  }
};
for (var f of flat(arr)) {
  console.log(f);
} // 1, 2, 3, 4, 5, 6


// yield表达式如果用在另一个表达式之中，必须放在圆括号里面
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}


// yield表达式用作函数参数或放在赋值表达式的右边，可以不加括号
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```

上面代码中，函数`f`如果是普通函数，在为变量`generator`赋值时就会执行。但是，函数`f`是一个 Generator 函数，就变成只有调用`next`方法时，函数`f`才会执行。

#### 与 Iterator 接口的关系

任意一个对象的`Symbol.iterator`方法，等于该对象的**遍历器生成函数**，调用该函数会返回该对象的一个遍历器对象

- 由于 Generator 函数就是遍历器生成函数，因此**可以把 Generator 赋值给对象的`Symbol.iterator`属性**，从而使得该对象具有 Iterator 接口
- Generator 函数执行后，返回一个遍历器对象。该对象本身也具有`Symbol.iterator`属性，**执行后返回自身**

``` JS
// Generator函数作为对象的Symbol.iterator属性
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};
[...myIterable] // [1, 2, 3]



// Generator 函数执行后，返回一个遍历器对象。该对象本身也具有Symbol.iterator属性，执行后返回自身。
function* gen(){
  // some code
}
var g = gen();
g[Symbol.iterator]() === g
// true
```

上面代码中，Generator 函数赋值给`Symbol.iterator`属性，从而使得`myIterable`对象具有了 Iterator 接口，可以被`...`运算符遍历了

`gen`是一个 Generator 函数，调用它会生成一个遍历器对象`g`。它的`Symbol.iterator`属性，也是一个遍历器对象生成函数，执行后返回它自己

### next方法

`yield`表达式本身没有返回值，或者说总是返回`undefined`

`next`方法可以带一个参数，该参数就会被当作**上一个`yield`表达式的返回值**

``` JS
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
// 当第三次执行next函数后，传入的true被当成了上一次yield的返回值，reset变为true，继续执行，重置i为-1，i++变为0，遇到yield i暂停执行
```

上面代码先定义了一个可以**无限运行**的 Generator 函数`f`，如果`next`方法没有参数，每次运行到`yield`表达式，变量`reset`的值总是`undefined`。当`next`方法带一个参数`true`时，变量`reset`就被重置为这个参数（即`true`），因此`i`会等于`-1`，下一轮循环就会从`-1`开始递增

这个功能有很重要的语法意义。Generator 函数从暂停状态到恢复运行，它的**上下文状态（`context`）是不变**的。通过`next`方法的参数，就有办法在 Generator 函数**开始运行之后，继续向函数体内部注入值**。也就是说，可以在 Generator 函数运行的不同阶段，**从外部向内部注入不同的值，从而调整函数行为**

注意，由于`next`方法的参数表示上一个`yield`表达式的返回值，所以在第一次使用`next`方法时，传递参数是无效的。V8 引擎直接忽略第一次使用`next`方法时的参数，只有从第二次使用`next`方法开始，参数才是有效的。从语义上讲，第一个`next`方法用来启动遍历器对象，所以不用带有参数

如果想要第一次调用`next`方法时，就能够输入值，可以在 Generator 函数外面再包一层

``` JS
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

上面代码中，第二次运行`next`方法的时候不带参数，导致 `y` 的值等于`2 * undefined`（即`NaN`），除以 `3` 以后还是`NaN`，因此返回对象的`value`属性也等于`NaN`。第三次运行`Next`方法的时候不带参数，所以`z`等于`undefined`，返回对象的`value`属性等于`5 + NaN + undefined`，即`NaN`。

如果向`next`方法提供参数，返回结果就完全不一样了。上面代码第一次调用`b`的`next`方法时，返回`x+1`的值`6`；第二次调用`next`方法，将上一次`yield`表达式的值设为`12`，因此`y`等于`24`，返回`y / 3`的值`8`；第三次调用`next`方法，将上一次`yield`表达式的值设为`13`，因此`z`等于`13`，这时`x`等于`5`，`y`等于`24`，所以`return`语句的值等于`42`

``` JS
// 通过next方法的参数，向 Generator 函数内部输入值
// 每次通过next方法向 Generator 函数输入值，然后打印出来
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}
let genObj = dataConsumer();
genObj.next();
// Started
genObj.next('a')
// 1. a
genObj.next('b')
// 2. b


// Generator 函数如果不用wrapper先包一层，是无法第一次调用next方法，就输入参数的
function wrapper(generatorFunction) {
  return function (...args) {
    let generatorObject = generatorFunction(...args);
    generatorObject.next();
    return generatorObject;
  };
}
const wrapped = wrapper(function* () {
  console.log(`First input: ${yield}`);
  return 'DONE';
});
wrapped().next('hello!')
// First input: hello!
```

## Generator 函数的异步应用
