# 组织化

编写JS代码是一回事，但正确组织JS代码是另一回事。 利用常见的模式进行组织和重构将大大提高代码的可读性和可理解性。 记住：代码至少像与其他开发人员进行沟通一样，那就是关于计算机指令的回馈。

ES6有几个重要的特性，有助于显着改善这些模式，包括：iterators，generators，modules和classes。

## Iterators 迭代器
迭代器是用于，以一次一个的方式，从源拉取信息的结构化模式。这种模式已经在编程中应用了很长时间了。可以肯定的是，因为在任何人都能够记住之前，JS开发人员已经在JS程序中专门设计和实现迭代器，，所以它不是一个新的话题。

ES6已经做的事情是为迭代器引入一个隐式标准化接口。 JavaScript中的许多内置数据结构，现在将暴露实现此标准的迭代器。您还可以构建自己的迭代器遵守相同的标准，以实现最大的互操作性。

迭代器是一种组织有序，顺序化，基于拉取的数据消耗方式。

例如，您可以实现每次请求时生成新的唯一标识符的实用程序。或者，你可以产生一个无限的系列值，以循环方式旋转通过固定列表。或者，您可以将迭代器附加到数据库查询结果，以一次一个地提取新行。

虽然它们通常不以这种方式在JS中使用，但是迭代器也可以被认为是一步一次地控制行为。这可以在考虑generators时非常清楚地说明（参见本章后面的“Generators”），虽然你没有Generators也可以做到。

## Interfaces 接口
在撰写本文时，ES6第25.1.1.2节（https://people.mozilla.org/~jorendorff/es6-draft.html#sec-iterator-interface）详细描述了`Iterator`接口具有以下要求：

```
Iterator [required]
    next() {method}: retrieves next IteratorResult
```

这里有两个，一些迭代器可以拓展的可选成员：

```
Iterator [optional]
    return() {method}: stops iterator and returns IteratorResult
    throw() {method}: signals error and returns IteratorResult
```

`IteratorResult`接口是指:

```
IteratorResult
    value {property}: current iteration value or final return value
        (optional if `undefined`)
    done {property}: boolean, indicates completion status
```

注意：我调用这些隐式接口不是因为它们没有在规范中明确调用 - 尽管他们确实是！ - 但是因为它们没有暴露成为，代码可访问的直接对象。` JavaScript`在`ES6`中不支持任何`“接口”`的概念，所以对自己的代码是纯粹的遵守常规。 然而，JS期望一个迭代器  ，一个`for..of`循环，例如 - 你提供的，必须遵循的接口或将失效的代码。

还有一个`Iterable`接口，它描述为一定能够产生迭代器的对象：

```
Iterable
    @@iterator() {method}: produces an Iterator
```

如果你记得第2章中的“内置符号”，`@@iterator`是表示可以为对象产生迭代器的方法的特殊内置符号。

##  `IteratorResult`  
`IteratorResult`接口指定来自任何迭代器操作的返回值将是以下形式的对象：

```
{ value: .. , done: true / false }
```

内置迭代器将始终返回此形式的值，但是更多属性当然可以根据需要显示在返回值上。

例如，自定义迭代器可以向结果对象添加附加元数据（例如，数据来自何处，取回所花费的时间，缓存期满长度，适当的下一请求的频率等）。

注意：在技术上，例如在值未定义的情况下，`value`是可选的，否则将被认为`缺少`或`undefined` ，。 因为访问`res.value`将产生`undefined `不论存在该值或完全不存在，属性的存在/不存在更多是实现细节或优化（或两者都有），而不是功能上的问题。

## `next()` Iteration
让我们看一个数组，它是可迭代的，它可以产生iterator 来迭代它的值：

```
var arr = [1,2,3];

var it = arr[Symbol.iterator]();

it.next();        // { value: 1, done: false }
it.next();        // { value: 2, done: false }
it.next();        // { value: 3, done: false }

it.next();        // { value: undefined, done: true }
```

每当对当前`arr`值调用位于`Symbol.iterator`（见第`2`和`7`章）的方法时，它将产生一个新的迭代器。 大多数结构都会这样做，包括`JS`中的所有内置数据结构。

然而，像事件队列消费者的结构,可能只产生单个迭代器（单例模式）。 或者一个结构可能一次仅允许一个唯一的迭代器，需要在创建新的迭代器之前完成当前的迭代器。

当您收到值`3`,前面片段中的`it`迭代器不会报告`done：true`。 你不得不再次调用`next()` ，实质上超出数组的阈值，以获得完整的信号`done: true`。 可能直到本节后面，你也不清楚为什么，但是决策设计通常被认为是最佳实践。

原始字符串值在默认情况下也是`iterable`：

```
var greeting = "hello world";

var it = greeting[Symbol.iterator]();

it.next();        // { value: "h", done: false }
it.next();        // { value: "e", done: false }
..
```

注意：在技术上，原始值本身是不可迭代的，但是由于“boxing”，“hello world”被强制/转换为它的String对象包装形式，这是一个可迭代的。 有关详细信息，请参阅本系列的`Types & Grammar `标题。

ES6还包括几个新的数据结构，称为集合（见第`5`章）。 这些集合不仅是`iterables`本身，而且还提供了API方法来生成迭代器，例如：

```
var m = new Map();
m.set( "foo", 42 );
m.set( { cool: true }, "hello world" );

var it1 = m[Symbol.iterator]();
var it2 = m.entries();

it1.next();        // { value: [ "foo", 42 ], done: false }
it2.next();        // { value: [ "foo", 42 ], done: false }
..
```
迭代器的`next(..)`方法可以选择引用一个或多个参数。 内置的迭代器大多不使用这个功能，虽然生成器的迭代器肯定会这样做（参见本章后面的“Generators”）。

按照一般约定，包括所有内置的迭代器，在已经耗尽的迭代器上调用`next(..)`不是一个错误，而是将继续返回结果`{value：undefined，done：true}`。

## Optional: `return(..)` and `throw(..)`

迭代器接口上的可选方法 - `return(..)`和`throw(..)` - 不在大多数内置迭代器上实现。然而，它们确实意味着有些是在生成器的上下文中，因此请参阅“Generators”以获取更具体的信息。

`return(..)`被定义为向迭代器发送`迭代代码`完成的信号，并且不会从中拉出任何更多的值。这个信号可以用于通知生产者（响应`next(..)`调用的迭代器）执行它可能需要做的任何清理工作，例如释放/关闭网络，数据库或文件句柄资源。

如果迭代器目前有`return(..)`方法，并且任意可以自动解释为异常或提前终止迭代器的情况发生了，则`return(..)`将被自动调用。你可以手动调用`return(..)`。

`return(..)`将返回一个`IteratorResult`对象，就像`next(..)`一样。一般来说，发送给`return(..)`的可选值将作为`IteratorResult`中的值返回，尽管有些`细微`的情况可能不是真的。

`throw(..)`用于向迭代器发出`异常/错误`信号，迭代器可能使用与`return(..)`隐含的完成信号不同的异常/错误。它不一定意味着迭代器的完全停止，因为`return(..)`通常会。

例如，使用生成器,迭代器，`throw(..)`实际上将一个抛出的异常注入到`generator`的暂停执行上下文中，可以使用`try..catch`来捕获它。未捕获的`throw(..)`异常,将最终,异常终止`generator`的`iterator`。

注意：按照一般惯例，在调用`return(..)`或`throw(..)`.之后，迭代器不应该产生任何更多的结果。


## Iterator Loop 迭代器循环
正如我们在第2章中的"`for..of`"部分所述，ES6 `for..of`循环直接使用一个一致的迭代。

如果迭代器也是可迭代的，它可以直接用于`for..of`循环。 你可以通过给一个迭代器一个`Symbol.iterator`方法来返回迭代器本身：
```
var it = {
    // make the `it` iterator an iterable
    [Symbol.iterator]() { return this; },

    next() { .. },
    ..
};

it[Symbol.iterator]() === it;        // true
```

我们现在可以使用一个`for..of`循环来迭代`it`迭代器了:

```
for (var v of it) {
    console.log( v );
}
```
为了完全理解一个循环是如何起作用的，让我们重新回顾第二章中的`for`等式中的`for..of `循环:

```
for (var v, res; (res = it.next()) && !res.done; ) {
    v = res.value;
    console.log( v );
}
```

你如果仔细看，你会看到`it.next()`在每次迭代之前被调用，然后查询`res.done`。如果`res.done`为`true`，则表达式的计算结果为`false`，不会发生迭代。

回想一下，我们建议迭代器一般不应返回`done：true`以及迭代器的最终预期值。现在你可以看到为什么。

如果迭代器返回`{done：true，value：42}`，`for..of`循环将完全丢弃值`42`，它将丢失。出于这个原因，假设你的迭代器可能被`for..of`循环或它的手动等价的模式`for`所迭代，你应该等待返回`done: true`通信完成，直到你已经返回所有相关的迭代值。

警告：当然，你可以有意地设计你的迭代器返回一些相关的`值`，同时返回`done：true`。但是不要这样做，除非你记录了这种情况，并因此隐式强制迭代器的消费者使用不同的模式进行迭代，而不是由`for..of`或我们所描述的其手动等价模式。

## Custom Iterators
除了标准的内置迭代器，你可以自己做！ 使它们与ES6的迭代港式（例如，`for..of`循环和`...`操作符）进行互操作所需要的是遵守适当的接口。

让我们尝试构造一个迭代器，在斐波纳契序列中产生无穷数字序列：
```
var Fib = {
    [Symbol.iterator]() {
        var n1 = 1, n2 = 1;

        return {
            // make the iterator an iterable
            [Symbol.iterator]() { return this; },

            next() {
                var current = n2;
                n2 = n1;
                n1 = n1 + current;
                return { value: current, done: false };
            },

            return(v) {
                console.log(
                    "Fibonacci sequence abandoned."
                );
                return { value: v, done: true };
            }
        };
    }
};

for (var v of Fib) {
    console.log( v );

    if (v > 50) break;
}
// 1 1 2 3 5 8 13 21 34 55
// Fibonacci sequence abandoned.
```

警告：如果我们没有插入`断点`条件，这个`for..of`循环将永远运行，可能不希望打破您的程序的期望结果！

调用时的`Fib[Symbol.iterator]()`方法返回具有`next()`和`return(..)`方法的迭代器对象。 状态通过`n1`和`n2`变量维护，这些变量由闭包保存。

让我们来考虑一个迭代器，它被设计为通过一系列（也就是一个队列）的动作，一次一个项目：

```
var tasks = {
    [Symbol.iterator]() {
        var steps = this.actions.slice();

        return {
            // make the iterator an iterable
            [Symbol.iterator]() { return this; },

            next(...args) {
                if (steps.length > 0) {
                    let res = steps.shift()( ...args );
                    return { value: res, done: false };
                }
                else {
                    return { done: true }
                }
            },

            return(v) {
                steps.length = 0;
                return { value: v, done: true };
            }
        };
    },
    actions: []
};
```

`tasks`的迭代器通过在`actions`数组属性中找到的函数（如果有的话）执行它们，并一次执行一个，传递给下一个传递的任何参数`next(..)`，并在标准`IteratorResult`对象中返回任何返回值 。

以下是我们如何使用此`tasks`队列 ：

```
tasks.actions.push(
    function step1(x){
        console.log( "step 1:", x );
        return x * 2;
    },
    function step2(x,y){
        console.log( "step 2:", x, y );
        return x + (y * 2);
    },
    function step3(x,y,z){
        console.log( "step 3:", x, y, z );
        return (x * y) + z;
    }
);

var it = tasks[Symbol.iterator]();

it.next( 10 );            // step 1: 10
                        // { value:   20, done: false }

it.next( 20, 50 );        // step 2: 20 50
                        // { value:  120, done: false }

it.next( 20, 50, 120 );    // step 3: 20 50 120
                        // { value: 1120, done: false }

it.next();                // { done: true }
```
这种特殊用法加强了迭代器可以作为组织功能的模式，而不仅仅是数据。 这也让人联想到我们将在下一节看到的generators 。

你甚至可以创造性地定义一个迭代器，它代表单个数据块上的元操作。 例如，我们可以为数字定义一个迭代器，默认值 范围为0到无穷大，或者（或无穷小）都取决于问题的数字。

试想一下：

```
if (!Number.prototype[Symbol.iterator]) {
    Object.defineProperty(
        Number.prototype,
        Symbol.iterator,
        {
            writable: true,
            configurable: true,
            enumerable: false,
            value: function iterator(){
                var i, inc, done = false, top = +this;

                // iterate positively or negatively?
                inc = 1 * (top < 0 ? -1 : 1);

                return {
                    // make the iterator itself an iterable!
                    [Symbol.iterator](){ return this; },

                    next() {
                        if (!done) {
                            // initial iteration always 0
                            if (i == null) {
                                i = 0;
                            }
                            // iterating positively
                            else if (top >= 0) {
                                i = Math.min(top,i + inc);
                            }
                            // iterating negatively
                            else {
                                i = Math.max(top,i + inc);
                            }

                            // done after this iteration?
                            if (i == top) done = true;

                            return { value: i, done: false };
                        }
                        else {
                            return { done: true };
                        }
                    }
                };
            }
        }
    );
}
```

现在，这个创造力能为我们提供什么技巧？

```
for (var i of 3) {
    console.log( i );
}
// 0 1 2 3

[...-3];                // [0,-1,-2,-3]
```
有一些有趣的技巧，虽然实用程序是有点争论。 然而，有可能会想知道为什么ES6不只是搭载这样一个小玩意，往往是令人愉快的功能彩蛋！

我可能忽略了，是否我至少没有提醒你，扩展原型，因为我在前面的代码段做的事情，你应该只是谨慎地意识到潜在的危害。

在这种情况下，你将与其他代码或甚至未来的JS功能碰撞的机会可能是非常低的。 但只是意识到潜在的可能性。 为了后人的缘故记录你所做的事情。

注意：如果你想了解更多的细节，我已经在这篇博文（http://blog.getify.com/iterating-es6-numbers/）中阐述了这个特殊的技术。 这篇文章（http://blog.getify.com/iterating-es6-numbers/comment-page-1/#comment-535294）甚至阐述可一个类似的伎俩，但是使字符串字符范围。

## Iterator Consumption

我们已经显示了使用`for..of`循环使用迭代器迭代每个项。 但是还有其他ES6结构可以使用迭代器。

让我们考虑连接到这个数组的迭代器（虽然我们选择的迭代器会有以下表现）：

```
var a = [1,2,3,4,5];
```
`...`操作符完全迭代了迭代器。试想一下：


```
function foo(x,y,z,w,p) {
    console.log( x + y + z + w + p );
}

foo( ...a );            // 15
```
`...`操作符同样的可以完全迭代一个数组迭代器：
```
var b = [ 0, ...a, 6 ];
b;                        // [0,1,2,3,4,5,6]
```
数组解构（参见第2章中的"解构"）可以部分或完全（如果与`...`rest / gather操作符配对使用迭代器：

```
var it = a[Symbol.iterator]();

var [x,y] = it;            // take just the first two elements from `it`
var [z, ...w] = it;        // take the third, then the rest all at once

// is `it` fully exhausted? Yep.
it.next();                // { value: undefined, done: true }

x;                        // 1
y;                        // 2
z;                        // 3
w;                        // [4,5]
```

## Generators

所有函数运行到结束，对吧？换句话说，一旦函数开始运行，它在任何其他可能中断之前，结束。

至少这就是到目前为止的，JavaScript的，整个历史。从ES6开始，一种新的有些奇怪的功能被引入，称为`generator`。generator 可以在中期运行，执行暂停，并且可以立即恢复，或在稍后的时间恢复。所以它，显然不包含，正常函数所能做的，运行到结束的保证。

此外，中间执行中的每个`暂停/恢复`循环是用于双向消息传递的机会，其中`generator `可以返回值，并且恢复它的控制代码可以重新发送回调值。

与上一节中的迭代器一样，有多种方式来思考generator 是什么，或者它最有用的。没有正确的答案，但我们将尝试考虑几个角度。

注意：有关生成器的更多信息，请参阅本系列的`Async＆Performance`标题，另请参阅本当前标题的第4章。


### Syntax 语法：


generator 函数使用以下新语法声明：

```
function *foo() {
    // ..
}
```
`*`的位置与功能无关。相同的声明可以写为以下任何一个:

```
function *foo()  { .. }
function* foo()  { .. }
function * foo() { .. }
function*foo()   { .. }
..
```
这里唯一的区别是风格偏好。 大多数其他文献似乎偏向于`function* foo(..) { .. }`。 我更加偏向于`function *foo(..) { .. }`，所以这就是我将介绍这个标题的其余部分。

我的理由纯粹是教学性质。 在本文中，当引用一个generator 函数时，我将使用`*foo(..)`，而不是`foo(..)`用于正常函数。 我观察到`*foo(..)`更接近匹配的函数`function *foo(..) { .. }`的`*`定位。

此外，正如我们在第2章中用简洁的方法所看到的，在对象字面量中有一个简洁的生成器形式：
```
var a = {
    *foo() { .. }
};
```
我想说，使用简洁的生成器，`*foo() { .. }`比`* foo() { .. }`更自然。 所以进一步主张匹配`*foo()`的一致性。

一致性简化了理解和学习。

### Executing a Generator 执行生成器

虽然一个生成器用`*`声明，然而你仍然可以像普通函数一样执行它：


```
foo();
```
你仍然可以给他传递参数，例如：

```
function *foo(x,y) {
    // ..
}

foo( 5, 10 );
```

主要的区别是,执行一个generator，如`foo(5,10)`实际上并不运行`generator`中的代码。 相反，它产生一个iterator(迭代器) ，来控制`generator`执行其代码。

我们稍后将在“Iterator Control（迭代器控制）”中回过头来，但简要地说：

```
function *foo() {
    // ..
}

var it = foo();

// to start/advanced `*foo()`, call
// `it.next(..)`
```
### `yield`

Generators( 生成器)也有一个新的关键字，你可以在里面使用，以示意暂停点：`yield`。试想一下：

```
function *foo() {
    var x = 10;
    var y = 20;

    yield;

    var z = x + y;
}
```

在这个`*foo()`生成器中，前两行的操作将在开始时运行，然后`yield`将暂停生成器。 如果并且当恢复时，`*foo()`的最后一行将运行。`yield`可以出现任何次数（或根本不会，技术上！）在generator中。

你甚至可以把`yield`放在一个循环中，它可以表示一个重复的暂停点。 事实上，一个永远不会完成的循环意味着一个永不完成的生成器，这是完全有效的，有时完全是你需要的。

`yield `不仅仅是一个暂停点。 它还是一个表达式，当暂停generator时,发出一个值。 这里是在一个生成器中的一个`while..true`循环，每次迭代`yield`产生一个新的随机数：

```
function *foo() {
    while (true) {
        yield Math.random();
    }
}
```

`yield ..`表达式不仅仅是发送值 - 无值的`yield`与`yield undefined`相同 - 而且还接收（例如，被替换）最终恢复值。 试想一下：

```
function *foo() {
    var x = yield 10;
    console.log( x );
}
```
当发生暂停时，该generator 将首先产生(`yield `out)值`10`。 当你恢复generator  - 使用`it.next(..)`时，我们前面提到 - 无论什么值（如果有）你恢复将`替换/完成`整个`yield 10`表达式，意味着该值将被分配给`x `变量。

`yield ..`表达式可以出现在正常表达式的任何位置。例如：

```
function *foo() {
    var arr = [ yield 1, yield 2, yield 3 ];
    console.log( arr, yield 4 );
}
```

`*foo()`这里有四个`yield ..`表达式。 每个`yield`将导致生成器暂停等待恢复值，然后在各种表达式上下文中使用恢复值。

`yield `在技术上不是操作符，虽然当使用像`yield 1`，它肯定看起来像。 因为`yield`可以在`var x = yield`中单独使用，所以它被认为是一个操作符有时可能会引起混淆。

从技术上讲，`yield ..`是相同的“表达式优先级” - 在概念上类似于运算符优先级 - 作为一个赋值表达式像`a = 3`。这意味着`yield ..`基本上可以出现在任何地方`a = 3`可以有效地出现。

让我们来说明对称性：

```
var a, b;

a = 3;                    // valid
b = 2 + a = 3;            // invalid
b = 2 + (a = 3);        // valid

yield 3;                // valid
a = 2 + yield 3;        // invalid
a = 2 + (yield 3);        // valid
```
注意：如果你考虑它，它是一种概念意义上，`yield ..`表达式将类似于一个赋值表达式。 当恢复暂停的`yield `表达式时，它以完全不同于“分配”该值的方式`完成/替换`为恢复值。

拓展：如果你需要`yield ..`出现在一个位置，像`a = 3`本身不允许，它需要包含在一个`( )`内。

由于`yield`关键字的低优先级，几乎在`yield ..`之后的任何表达式,都将在,与`yield`一起发送之前,进行计算。 只有`... spread`运算符和`,`运算符具有较低的优先级，这意味着它们将在`yield`被计算后绑定。

所以就像普通语句中的多运算符一样，其他案例的位置可能需要使用`( )`来覆盖（提升）`yield`的低优先级，例如这些表达式之间的区别：
```
yield 2 + 3;            // same as `yield (2 + 3)`

(yield 2) + 3;            // `yield 2` first, then `+ 3`
```
就像`= `分配符一样，`yield`也是“右关联”，这意味着连续的多个`yield`表达式被视为已经使用`( .. )`从右到左分组。因此，将`yield yield yield 3`视为`yield (yield (yield 3))`。如`((yield) yield) yield 3`的“左相关”解释将没有意义。

就像运算符一样，使用`( .. )`分组是个好主意，即使不是严格要求，如果`yield`与其他运算符或`yield`相结合，也可以消除歧义。

注意：有关运算符优先级和关联性的详细信息，请参阅本系列的`Types & Gramma`标题。

### `yield *`

以同样的方式，`*`使一个函数声明为`function *`生成器声明，`*`使得`yield`为`yield *`，这是一个非常不同的机制，称为`yield`委托。语法上，`yield * ..`的行为将与`yield ..`相同，如前一节所讨论的。

`yield * ..`需要一个迭代;它将调用该可迭代的迭代器，并将其自己的主生成器的控制委托给该迭代器，直到它完全迭代。试想一下：

```
function *foo() {
    yield *[1,2,3];
}
```
注意：与生成器声明中的`*`位置一样（前面已经讨论过），`yield *`表达式中的`*`定位在风格上取决于你。 大多数其他文献喜欢`yield* ..`，但我更喜欢`yield *..`，因为已经讨论过的非常对称的原因。

值`[1,2,3]`产生一个遍历其值的迭代器，因此`*foo()`生成器将在迭代时产生这些值。 另一种说明行为的方法，是在委托给另一个生成器的`yield`中：

```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

function *bar() {
    yield *foo();
}
```
当`*bar()`调用`*foo() `通过`yield *`被委托，意味着任何`*foo()`产生的迭代器值将由`**bar()`产生。

而`yield`。表达式的完成值来自于使用`it.next(..)`恢复生成器，`yield * ..`表达式的完成值来自于委托到迭代器的返回值（如果有的话） 。

内置迭代器通常没有返回值，正如我们在本章前面的“迭代器循环”部分的末尾所述。 但是如果你定义自己的自定义迭代器（或生成器），你可以设计它返回一个值，这将产生`yield *.. `捕获：

```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    return 4;
}

function *bar() {
    var x = yield *foo();
    console.log( "x:", x );
}

for (var v of bar()) {
    console.log( v );
}
// 1 2 3
// x: 4
```
当从`*foo()`产生值`1`,`2`和`3`，然后从`*bar()`产生值`1`，`2`和`3`时，从`*foo()`返回的值`4`是`yield *foo()`表达式的完成值，然后被分配给`x`。

因为`yield *`可以调用另一个生成器（通过委托给它的迭代器），它还可以通过调用自身来执行一种生成器递归：

```
function *foo(x) {
    if (x < 3) {
        x = yield *foo( x + 1 );
    }
    return x * 2;
}

foo( 1 );
```

`foo(1)`的结果然后调用迭代器的`next()`通过它的,递归步骤,运行它。第一个`*foo(..)`运行其`x`的值为`1`，这是`x <3. x + 1` 被递归地传递给`*foo(..)`，所以`x`是`2`.另一个递归调用导致`x`为`3`。

现在，因为`x <3`失败，递归停止，并且返回`3 * 2`将`6`返回到上一个调用的`yield * ..`表达式，然后将其分配给`x`。 另一个返回`6 * 2`返回`12`回到上一个调用的`x`。 最后，从`*foo(..) `生成器的完成运行返回`12 * 2`或`24`。

### Iterator Control 迭代器控制

之前，我们简要介绍了生成器由迭代器控制的概念。让我们现在完全将其挖掘。 回忆上一节中的递归`*foo(..)`。下面是我们如何运行它：

```
function *foo(x) {
    if (x < 3) {
        x = yield *foo( x + 1 );
    }
    return x * 2;
}

var it = foo( 1 );
it.next();                // { value: 24, done: true }
```

在这种情况下，生成器不会真的暂停，因为没有`yield ..`表达式。 相反，`yield *`只通过递归调用保留当前迭代步骤。 因此，只需调用迭代器的`next()`函数就可以完全运行生成器。

现在让我们考虑一个生成器，它将有多个步骤，从而产生多个值：

```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}
```

我们已经知道我们可以使用一个迭代器，甚至一个像`*foo()`这样的生成器，它带有一个`for..of`循环：

```
for (var v of foo()) {
    console.log( v );
}
// 1 2 3
```
注意：`for..of`循环需要一个可迭代。 生成器函数引用（如`foo`）本身不是可迭代的; 您必须使用`foo()`执行它以获取迭代器（它也是可迭代的，如我们在本章前面所解释的）。 你可以理论上扩展`GeneratorPrototype`（所有生成器函数的原型）与一个`Symbol.iterator`函数，本质上只是返回`this()`。 这将使`foo`引用本身是一个迭代，这意味着`for (var v of foo) { .. }`（通知没有`()`在`foo`之后）将起作用。

让我们手动替换生成器的迭代器：
```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it = foo();

it.next();                // { value: 1, done: false }
it.next();                // { value: 2, done: false }
it.next();                // { value: 3, done: false }

it.next();                // { value: undefined, done: true }
```

如果你仔细看，有三个`yield`语句和四个`next()`调用。 这可能看起来像一个奇怪的不匹配。 事实上，总是会有一个`next()`调用比`yield`表达式多一个，假设所有都被求值，生成器完全运行到结束。

但是如果你从相反的角度看（从内而外，而不是从外到内），`yield`和`next()`之间的匹配更有意义。

回想一下，`yield ..`表达式将由您恢复生成器的值完成。 这意味着你传递给`next(..) `的参数完成了，不管`yield ..`表达式当前处于暂停状态等待着完成 。

让我们以这种方式说明这个观点：

```
function *foo() {
    var x = yield 1;
    var y = yield 2;
    var z = yield 3;
    console.log( x, y, z );
}
```
在这个代码片段中，每个`yield ..`都发送一个值（`1`，`2`，`3`），但是更直接的，它会暂停发生器并且等待着一个值。 换句话说，它几乎像问一个问题：“我应该在这里使用什么值？我会等待。

现在，我们如何控制`*foo()`启动它：

```
var it = foo();

it.next();                // { value: 1, done: false }
```
第一次`next()`调用从其初始暂停状态启动`generator`，并将其运行到第一个`yield`。 现在你调用第一个`next()`，没有`yield ..`表达式等待完成。 如果你传递一个值给第一个`next()`调用，它将被丢弃，因为没有`yield`等待接收这样的值。

注意：对于“beyond ES6”时间框架的早期提议，将允许您，访问通过生成器中，单独的`meta`属性（参见第7章），传递给初始`next(..)`调用的值。

现在，让我们回答目前悬而未决的问题：“应该为`x`分配什么值？ 我们将通过发送一个值到下一个`next(..)`调用来应答它：

```
it.next( "foo" );        // { value: 2, done: false }
```
现在，x将具有值"`foo`"，但我们还问了一个新问题，“应该为`y`分配什么值？我们回答：

```
it.next( "bar" );        // { value: 3, done: false }
```

回答给出，另一个问题。最终答案：

```
it.next( "baz" );        // "foo" "bar" "baz"
                        // { value: undefined, done: true }
```

现在应该更清楚如何每个`yield ..`“问题”由下一个`next(..)`调用，所以,我们观察到的“额外”`next(..)`调用,总是只是初始的一个,开始一切进行。

让我们把所有这些步骤放在一起：

```
var it = foo();

//启动generator
it.next();                // { value: 1, done: false }

// 回答第一个问题
it.next( "foo" );        // { value: 2, done: false }

//回答第二个问题
it.next( "bar" );        // { value: 3, done: false }

// 回答第三个问题
it.next( "baz" );        // "foo" "bar" "baz"
                        // { value: undefined, done: true }
```

您可以将generator 视为值的生产者，在这种情况下，每次迭代只是生成要使用的值。

但是在更一般的意义上，或许将生成器视为受控的渐进式代码执行是很合适的，就像之前的“Custom Iterators(自定义迭代器)”部分中的任务队列示例一样。

注意：这个角度正是我们将在第4章讨论generators的动机。具体来说，没有理由在`next()`完成后立即调用`next()`。 当生成器的内部执行上下文被暂停时，程序的其余部分继续解除阻塞，包括异步动作控制generator 何时恢复的能力。

## Early Completion
正如我们在本章前面所述，连接到生成器的迭代器支持可选的`return(..)`和`throw(..)`方法。两者都具有立即中止暂停的生成器的效果。 试想一下：

```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it = foo();

it.next();                // { value: 1, done: false }

it.return( 42 );        // { value: 42, done: true }

it.next();                // { value: undefined, done: true }
```

`return(x)`就像强制一个返回`x`在指定时刻被处理，这样你就得到了指定的值。 一旦generator 正常或如上所示提早完成，它将不再处理任何代码或返回任何值。

除了可以手动调用`return(..)`外，它还可以在迭代结束时由任何使用迭代器的`ES6`结构体自动调用，例如`for..of`循环和`... spread`操作符。

这个能力的目的在于，如果控制代码不再在它上面进行迭代，则可以通知generator，使得它可以执行任何清除任务（释放资源，重置状态等）。 与正常的函数清理模式相同，完成此操作的主要方法是使用`finally`子句：

```
function *foo() {
    try {
        yield 1;
        yield 2;
        yield 3;
    }
    finally {
        console.log( "cleanup!" );
    }
}

for (var v of foo()) {
    console.log( v );
}
// 1 2 3
// cleanup!

var it = foo();

it.next();                // { value: 1, done: false }
it.return( 42 );        // cleanup!
                        // { value: 42, done: true }
```

警告：不要在`finally`子句中放置`yield`语句！ 它是有效和合法的，但它也是一个非常糟糕的想法。 它在某种意义上作为延迟完成`return(..)`调用，作为任何`yield ..`在`finally`子句中的表达式尊重暂停和发送消息; 你不会如预期立即得到一个完整的generator 。 基本上没有好的理由选择这个疯狂的，不好的，部分，所以应该避免这样做！

除了上面的代码片段显示`return(..)`如何中止生成器，同时仍然触发`finally`子句，它还演示了一个generator每次调用时生成一个全新的iterator。 实际上，您可以同时使用附加同一个generator,多个iterator：

```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it1 = foo();
it1.next();                // { value: 1, done: false }
it1.next();                // { value: 2, done: false }

var it2 = foo();
it2.next();                // { value: 1, done: false }

it1.next();                // { value: 3, done: false }

it2.next();                // { value: 2, done: false }
it2.next();                // { value: 3, done: false }

it2.next();                // { value: undefined, done: true }
it1.next();                // { value: undefined, done: true }
```

### Early Abort 提前中止

除了调用`return(..)`，你可以调用`throw(..)`。 就像`return(x)`本质上是在当前暂停点处,将一个`return x`注入到生成器中，调用`throw(x)`本质上就像在暂停点处注入一个`throw x`。

除了异常行为（我们涵盖了下一节中`try`子句的含义），`throw(..)`产生同样的提前完成 ，中止generator在当前暂停点的运行。 例如：


```
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it = foo();

it.next();                // { value: 1, done: false }

try {
    it.throw( "Oops!" );
}
catch (err) {
    console.log( err );    // Exception: Oops!
}

it.next();                // { value: undefined, done: true }
```

因为`throw(..)`基本上注入了一个`throw ..`替换generator的`yield 1`行(hang)，没有处理这个异常，它将会立即传播给回调代码，它将使用`try..catch`处理它。

与`return(..)`不同，迭代器的`return(..)`方法从来不会被自动调用。

当然，虽然在前面的代码段中没有展示，但是当你调用`throw(..)`时，如果`try..finally`子句在`generator` 内部等待，则在异常传播给回调代码之前,`finally`子句将被给予一个完成的机会,。

## Error Handling 错误处理

正如我们已经提示的，生成器的错误处理可以用`try..catch`来表示，它在进入和返回方向都可以起到作用：

```
function *foo() {
    try {
        yield 1;
    }
    catch (err) {
        console.log( err );
    }

    yield 2;

    throw "Hello!";
}

var it = foo();

it.next();                // { value: 1, done: false }

try {
    it.throw( "Hi!" );    // Hi!
                        // { value: 2, done: false }
    it.next();

    console.log( "never gets here" );
}
catch (err) {
    console.log( err );    // Hello!
}
```
错误也可以通过`yield *`委托在两个方向传播：
```
function *foo() {
    try {
        yield 1;
    }
    catch (err) {
        console.log( err );
    }

    yield 2;

    throw "foo: e2";
}

function *bar() {
    try {
        yield *foo();

        console.log( "never gets here" );
    }
    catch (err) {
        console.log( err );
    }
}

var it = bar();

try {
    it.next();            // { value: 1, done: false }

    it.throw( "e1" );    // e1
                        // { value: 2, done: false }

    it.next();            // foo: e2
                        // { value: undefined, done: true }
}
catch (err) {
    console.log( "never gets here" );
}

it.next();                // { value: undefined, done: true }
```
当`*foo()`调用`yield 1`时，值`1`通过`*bar()`不变，正如我们已经看到的。

但是这段代码最有趣的是当`*foo()`调用`throw "foo: e2"`时，这个错误传播到`*bar()`，并立即被`*bar()`的`try..catc`h块捕获。 错误不会像值`1`通过`*bar()`。

`*bar()`的`catch`接着正常的输出`err ("foo: e2") `，然后`*bar()`正常结束，这就是为什么，从`it.next()`返回的迭代器结果是`{ value: undefined, done: true }`。

如果`*bar()`没有一个`try..catch`在`yield * ..`表达式周围，错误会自然传播出来，在途中它仍然会完成（终止）`*bar()`.


## Transpiling a Generator 转换 Generator

在ES6之前，是否可以表示Generator的功能？ 事实证明，是可行的，因为有几个伟大的工具，这样做，包括最突出的`Facebook`的`Regenerator`工具（https://facebook.github.io/regenerator/）。

但只是为了,更好地了解`Generator`，让我们试着,手动转换。 基本上，我们将创建一个简单的,基于闭包的,状态机。

我们将保持generator源代码很简单：

```
function *foo() {
    var x = yield 42;
    console.log( x );
}
```

首先，我们需要一个可以执行的函数`foo()`，它需要返回一个迭代器：

```
function foo() {
    // ..

    return {
        next: function(v) {
            // ..
        }

        // we'll skip `return(..)` and `throw(..)`
    };
}
```

现在，我们需要一些`内部变量`来跟踪我们在"generator"'逻辑的步骤中的位置。 我们称它为`状态`(`state`)。 这将有三个状态：`0`,最初，`1`,等待满足`yield`表达式，`2`,生成器立即完成。

每次调用`next(..)`，我们需要处理下一步，然后增加`state`。 为了方便起见，我们将每个步骤放在`switch`语句的`case`子句中，我们将在`next(..)`可调用的内部函数`nextState(..)`中保存。 另外，因为`x`是` "generator," `的整个范围上的变量，所以它需要放在``nextState(..)``函数之外。

这里是完整的（显然有点简化，为了保持概念图更清楚）：

```
function foo() {
    function nextState(v) {
        switch (state) {
            case 0:
                state++;

                // the `yield` expression
                return 42;
            case 1:
                state++;

                // `yield` expression fulfilled
                x = v;
                console.log( x );

                // the implicit `return`
                return undefined;

            // no need to handle state `2`
        }
    }

    var state = 0, x;

    return {
        next: function(v) {
            var ret = nextState( v );

            return { value: ret, done: (state == 2) };
        }

        // we'll skip `return(..)` and `throw(..)`
    };
}
```

最终，我们来测试ES6之前版本的`"generator"`.

```
var it = foo();

it.next();                // { value: 42, done: false }

it.next( 10 );            // 10
                        // { value: undefined, done: true }
```

不错吧，嗯哼？ 希望这个练习在你的脑海中巩固了`generators`实际上只是`状态机`逻辑的简单语法。这使它们广泛适用.



## Generator Uses
所以，现在我们更深刻地了解generators是如何工作的，它们有什么用处？

我们已经看到了两种主要`模式`：

*生成一系列值*：此用法可以是简单的（例如，随机字符串或递增的数字），或者它可以表示更多结构化数据访问（例如，对从数据库查询返回的数据进行迭代）。

无论哪种方式，我们使用iterator来控制generator，以便可以为每次调用`next(..)`调用一些逻辑。数据结构上的正常迭代器,仅仅拉取值,而没有任何控制逻辑。

*任务的队列串行执行*：此用法通常表示算法中的步骤的流控制，其中每个步骤都需要从某个外部源检索数据。每条数据的满足可以是立即的，或者可以,被异步地延迟。

从生成器内部的代码的角度来看，在屈服点处的,同步或异步的细节,是完全不透明的。此外，这些细节,被有意地抽象掉，例如不会隐藏具有这样的自然顺序表达步骤的实现并发症。抽象也意味着实现可以经常`交换/重构`，而不触及generator 中的代码。

当根据这些用途来观察generators时，它们对于手动状态机，变得不仅仅是不同，而是更好的语法。它们是用于组织和控制有序生产和迭代数据的强大抽象工具。

## Modules
我不认为这是一个夸张的建议，JavaScript所有的模式中唯一最重要的代码组织模式，是，并始终是，模块。 对于我自己，我认为对于社区的大部分人，`模块模式`驱动了绝大多数代码。

## The Old Way

```
function Hello(name) {
    function greeting() {
        console.log( "Hello " + name + "!" );
    }

    // public API
    return {
        greeting: greeting
    };
}

var me = Hello( "Kyle" );
me.greeting();            // Hello Kyle!
```

这个`Hello(..)`模块，可以通过调用后续次数，产生多个实例。 有时，模块只被称为单例（即，它只需要一个实例），在这种情况下，使用`IIFE`的以前的代码段的一个小的变化是常见的：

```
var me = (function Hello(name){
    function greeting() {
        console.log( "Hello " + name + "!" );
    }

    // public API
    return {
        greeting: greeting
    };
})( "Kyle" );

me.greeting();            // Hello Kyle!
```

这种模式被尝试过和测试过了。 它还具有足够的灵活性，可以针对多种不同的场景拥有各种各样的变化。

其中最常见的是异步模块定义（AMD），另一种是通用模块定义（UMD）。 我们不会在这里介绍这些模式和技术的细节，但它们在很多在线方面具有广泛解释。

## Moving Forward
从ES6开始，我们不再需要`依赖封装函数`和`闭包`为我们提供`模块支持`。 ES6模块具有一流的`语法`和`功能支持`。

在我们了解特定的语法之前，重要的是要了解ES6模块与过去处理模块的一些相当重要的概念差异：

ES6使用`基于文件`的模块，意味着`每个文件一个模块`。此时，没有将`多个模块组合`成`单文件`的标准化方法。

这意味着如果要将ES6模块直接加载到浏览器Web应用程序中，您只能单独加载它们，而不是作为，性能优化工作，常见的，单个文件中的，大包。

预期`HTTP/2`的同时出现将显着地减轻任何这样的性能问题，因为它在持久`套接字`连接上操作，并且因此可以,非常有效地,并行地,并且彼此交织地,加载,许多较小的文件。

ES6模块的`API`是`静态的`。也就是说，`静态地定义`模块的公共`API`上的所有`top-level exports` ，并且稍后不能对其进行修改。

一些使用习惯于能够提供`动态API`定义，其中可以响应于运行时条件来`添加/移除/替换`方法。这些用途必须更改以适应ES6静态`API`，否则它们必须限制对`二级对象`的`属性/方法`的动态更改。

ES6模块是`单例`。也就是说，只有一个模块的实例，保持其状态。每次将该模块,导入到另一个模块时，都将获得对一个集中实例的引用。如果你想生产多个`模块`实例，你的`模块`将需要供给某种工厂来做。
在模块的公共`API`上公开的属性和方法不仅仅是`值`或`引用`的正常赋值。它们是内部模块定义中的`标识符`的实际绑定（几乎类似于`指针`）。

在ES6之前的模块中，如果你在公共`API`上放置一个属性，该属性包含一个原始值，比如数字或字符串，那么该属性赋值是通过值复制的，对应变量的任何内部更新都是`独立的`，不会影响公共副本上的`API对象`。

使用ES6，导出一个`本地私有变量`，即使它当前持有一个原始`字符串/数字/etc`，也会导出一个变量的绑定。如果模块更改变量的值，则`外部导入绑定`现在将解析为该`新值`。

导入模块与静态请求加载模块（如果尚未加载）相同。如果您在浏览器中，这意味着网络上的`阻塞负载`。如果你在服务器（即`Node.js`），它是从文件系统的`阻塞加载`。

然而，不要担心`性能的影响`。由于ES6模块具有静态定义，因此可以静态扫描导入要求，即使在使用模块之前，也会`预加载`。

ES6实际上没有指定或处理这些加载请求如何工作的`机制`。有一个单独的`模块加载器`的概念，其中每个`托管环境`（浏览器，`Node.js`等）提供适合于环境的默认加载器。模块的`导入`使用字符串值来表示`模块`（URL，文件路径等）的位置，但是这个值在程序中是不透明的，只对`Loader`本身有意义。

你可以定义自己的`自定义Loader`，如果你想要更`细粒度的`控制比默认的`Loader`提供 - 这基本上没有，因为它从程序的代码z中完全隐藏。

正如您所看到的，ES6模块将提供封装组织代码的全部用例，控制公共`API`和引用`依赖项导入`。但是他们有一个这样做的非常特别的方式，这与你已经做了模块多年可，能或不可能有着非常密切地的关系。

## CommonJS

有一个熟悉的，但不完全兼容的，模块语法，称为`CommonJS`，这是`Node.js生态系统`中,被人门所熟悉的。

由于缺乏一个更有说服力的方式来说这件事，从长远来看，ES6模块本质上必然取代所有以前的模块的`格式`和`标准`，甚至`CommonJS`，因为它们建立在语言的`语法支持`之上。这将，在时间上，不可避免地,赢得作为优越的方法，如果没有其他无处不在的原因。

然而，这一点，我们面对一条，相当长的路要走到。在服务器端`JavaScript世界中`有数十万个`CommonJS`样式模块，并且是浏览器世界中不同格式标准（UMD，AMD，ad hoc）的许多模块的`10`倍。转型需要多年才能取得重大进展。

在过渡期间，模块 `transpilers/converters`(转换器)将是绝对必要的。你可能只是习惯了那个新的实现。无论您是在常规模块，AMD，UMD，CommonJS或ES6中编写，这些工具都必须解析并转换为适合您代码运行环境的格式。

对于`Node.js`，这可能意味着（目前）目标是`CommonJS`。对于浏览器，它可能是UMD或AMD。在接下来的几年里，随着这些工具的成熟和最佳实践的出现，预计会有大量的流量。

从这里开始，我对模块的最好的建议是：无论什么形式，你已经虔诚地感受到了强烈的亲和力，也表现赞赏和理解ES6模块，如他们，并让你的其他模块褪色。它们是`JS`模块的未来，即使这种实现有点不好。


## 新的方式
启用ES6模块的两个主要的新关键字是`import` (导入)和`export`(导出)。 语法有很多`细微差别`，让我们进一步看看。

警告：一个很容易忽略的,重要细节：`import `和`export`必须,始终显示在,其各自用法的,顶级范围中。 例如，不能在`if`条件中放置`import `或`export`; 它们必须放在所有`块`和`函数`之外。

### `exporting` API Members  导出API成员

`export`关键字要么放在`声明之前`，要么作为`运算符`（排序）使用要`export`的`特定绑定列表`。 试想一下：

```
export function foo() {
    // ..
}

export var awesome = 42;

var bar = [1,2,3];
export { bar };
```
相同的表达`exports`的方法:

```
function foo() {
    // ..
}

var awesome = 42;
var bar = [1,2,3];

export { foo, awesome, bar };
```
这些都称为 *命名导出( named export)*   ，因为您实际上导出`变量/函数/等`的`名称绑定`。

任何你不标记`export`,会在模块的范围内保持`private`。 也就是说，尽管` var bar = ..`看起来像是在顶级全局范围声明，`顶级范围`实际上是`模块`本身; 模块中没有`全局范围`。

注意：模块仍然可以访问`window`和所有的"globals" ，挂起它，只是不是作为词汇顶级范围。 但是，你真的应该远离全局模块，如果可能的话。

您还可以在命名导出期间对模块成员进行“重命名”（也称为别名）：

```
function foo() { .. }

export { foo as bar };
```
导入此模块时，只有`bar `成员名称可用于导入; `foo`保持隐藏在模块内。

模块导出不仅仅是值或引用的正常赋值，因为您习惯于`=`赋值运算符。 实际上，当你导出一些东西，你正在导出一个绑定（有点像一个指针）到那个东西（变量等）。

在模块中，如果更改已导出绑定的变量的值，即使已经导入（参见下一部分），导入的绑定也将解析为当前（更新）的值。

试想一下：

```
var awesome = 42;
export { awesome };

// later
awesome = 100;
```

导入此模块时，无论是在`awesome = 100`设置之前还是之后，一旦分配已经发生，导入的绑定将解析为值`100`，而不是`42`。

这是因为绑定本质上是对`awesome`变量本身的引用或指针，而不是其值的副本。JS引入了ES6模块绑定,这是一个大多数前所未有的概念。

虽然您可以在模块的定义中清楚地多次使用`export `，但ES6绝对偏好模块,具有单个导出的方法，这称为` default export`。用TC39委员会的一些成员的话来说，如果你遵循这种模式，你就会被“简单的`import `语法”所奖励，而如果你不这么做，则反过来被更冗长的语法来“惩罚”。

默认导出将特定导出的绑定设置为导入模块时的默认值。绑定的名称字面上是默认的。正如你将在后面看到的，当导入模块绑定时，你也可以重命名它们，就像你通常使用默认导出一样。

每个模块定义只能有一个默认值。我们将在下一节中介绍`import`，如果模块具有`默认导出`，您将看到导入语法如何更简洁。

对于默认导出语法有一个微妙的细微差别，您应该密切注意。比较这两个片段：

```
function foo(..) {
    // ..
}

export default foo;
```
还有这一个：

```
function foo(..) {
    // ..
}

export { foo as default };
```

在第一个片段中，您正在导出一个绑定到此时的函数表达式值，而不是标识符`foo`。 换句话说，导出`export default ..`采用一个表达式。 如果以后将`foo`分配给模块中的其他值，则模块导入仍会显示最初导出的函数，而不是新值。

顺便说一下，第一个片段也可以写成：

```
export default function foo(..) {
    // ..
}
```

警告：虽然这里`function foo..`在技术上,是一个函数表达式，为了模块的内部范围的目的，它被视为一个函数声明，因为`foo`名称绑定在模块的顶级作用域 称为“提升”）。 出口默认类`Foo`也是如此。但是，虽然你可以做到`export var foo = ..`，但你目前不能做到`export default var foo = ..`（或`let`或`const`），在一个令人沮丧的不一致的情况下。 在撰写本文时，为了一致性，已经讨论了在ES6之后很快增加这种能力。

再次回忆第二个片段：

```
function foo(..) {
    // ..
}

export { foo as default };
```

在此版本的模块导出中，默认导出绑定实际上是`foo`标识符而不是它的`值`，所以您获得前面描述的绑定行为（即，如果您以后更改`foo`的值，你在`import`端看到的值也将`更新`）。

在默认导出语法中要非常小心这个微妙的问题，特别是如果你的逻辑需要更新导出值。 如果你从来没有计划更新默认导出的值，`export default ..`是很好的。 如果您计划更新值，则必须使用`export {.. as default}`。 无论哪种方式，确保评论你的代码如何来解释你的意图！

因为每个模块只能有一个默认值，所以您可能会试图使用一个对象的默认导出,来设计您的模块，其中包含所有的`API`方法，例如：

```
export default {
    foo() { .. },
    bar() { .. },
    ..
};
```

这种模式，似乎很密切地，映射了很多开发人员，已经构建了，他们的ES6之前的模块，所以，它似乎是，一个自然的方法。 不幸的是，它存在一些缺点，并正式地不鼓励这样做。

特别是，JS引擎，不能静态分析普通对象的内容，这意味着，它不能对静态导入性能，做一些优化。 每个成员单独和显式导出的优点是，引擎可以进行静态分析和优化。

如果你的API，已经有多个成员，似乎这些原则 - 每个模块一个默认导出，所有API成员名为exports - 存在冲突，不是吗？ 但是您可以有一个默认导出以及其他命名导出; 它们不是相互排斥的。

因此，替换掉当前的（不建议的）模式：

```

export default function foo() { .. }

foo.bar = function() { .. };
foo.baz = function() { .. };
```

你可以这么做：

```
export default function foo() { .. }

export function bar() { .. }
export function baz() { .. }
```

注意：在前面的代码段中，我使用名称`foo`作为默认函数的标签。 然而，名称`foo`，为了导出的目的被忽略 - `default`实际上是导出的名称。 当您导入此默认绑定时，您可以为其指定任何所需的名称，如下一部分中所示。

或者，有些人会喜欢：

```
function foo() { .. }
function bar() { .. }
function baz() { .. }

export { foo as default, bar, baz, .. };
```
当我们短期覆盖`import `时，混合`default `和命名`exports `的影响将更加清晰。但实质上它意味着最简洁的默认`import `形式,将只检索`foo()`函数。用户可以另外手动将`bar`和`baz`列为命名导入，如果他们想要的话。

你可以想象如果你有很多命名的导出绑定，你模块的用户将是多么乏味。这里有一个通配符导入表单，您可以在单个命名空间对象中导入所有模块的`exports `，但是无法将通配符导入到`顶级绑定`。

同样，ES6模块机制被有意设计为阻止具有大量输出的模块;相对来说，希望这样的方法有点困难，作为一种社会工程，它鼓励简单的模块设计，因为这将有利于`大/复杂`的模块设计。

我建议你不要混合`default export`与命名`exports`，这将是是不切实际或不需要，特别是如果你有一个大的API和重构单独的模块。在这种情况下，只需使用所有`named exports`，并记录您的模块的用户应该使用`import * as ..`（命名空间导入，在下一节中讨论）方法在单个命名空间中立即带来整个API。

我们之前提到过，但让我们更详细地回味一下。除了导出表达式值绑定的`export default ...`外，所有其他导出表单,都正在导出绑定到本地标识符。对于这些绑定，如果在导出后在模块中更改变量的值，则外部导入的绑定将访问更新的值：

```
var foo = 42;
export { foo as default };

export var bar = "hello world";

foo = 10;
bar = "cool";
```
当您导入此模块时，`default `和`bar`导出将绑定到局部变量`foo`和`bar`，这意味着它们将显示更新的`10`和`"cool"`值。 导出时的值是不相关。 导入时的值不相关。 绑定是实时链接，所以重要的是，当您访问绑定时的当前值。

警告：不允许双向绑定。 如果从模块导入`foo`，并尝试更改导入的`foo`变量的值，则会抛出错误！ 我们将在下一节再讨论。

您还可以重新导出另一个模块的`exports`，例如：

```
export { foo, bar } from "baz";
export { foo as FOO, bar as BAR } from "baz";
export * from "baz";
```

这些形式类似于只是首先从`"baz"`模块导入，然后明确列出，从模块导出的成员。 但是，在这些形式中，`"baz"`模块的成员绝不会导入本地作用域到模块; 他们始终不变。

### `import`ing API Members

要导入一个模块，不出意料的是，您会使用`import`语句。 正如`export`有几个细微差别，因此`import`也一样，通过您的选择花大量的时间考虑以下问题和实验。

如果要将模块的API的某些特定命名成员导入到顶级作用域，则可以使用以下语法：

```
import { foo, bar, baz } from "foo";
```
警告：这里的`{ .. }`语法可能看起来像一个`对象字面量`，甚至一个`对象解构语法`。 但是，它的形式是特殊的模块，所以要小心，不要与别的其他`{..}`模式混淆。

`"foo"`字符串称为 *module specifier* 。 因为整个目标是,静态可分析语法，所以模块说明符,必须是字符串文字; 它不能是一个保存字符串值的`变量`。

从你的ES6代码和JS引擎本身的角度来看，这个字符串文字的内容是完全不透明和无意义的。 模块加载器将此字符串解释为在哪里找到所需模块的指令，作为`URL路径`或`本地文件系统路径`。

列出的`foo`，`bar`和`baz`标识符必须与模块API上的`命名导出`（`静态分析`和`错误断言`适用）相匹配。 它们作为当前范围中的顶级标识符绑定：

```
import { foo } from "foo";

foo();
```
您可以重命名导入的绑定标识符，如下所示:

```
import { foo as theFooFunc } from "foo";

theFooFunc();
```

如果模块仅具有,要导入并绑定到标识符的`default export`，则可以选择跳过该绑定的`{ .. }`环境语法。 在这种优选情况下的导入，使用最简单和最简洁的导入语法形式：

```
import foo from "foo";

// or:
import { default as foo } from "foo";
```

注意：如上一节所述，模块`export`中的`default `关键字指定了一个命名导出，其中名称实际上是`default`，如第二个更详细的语法选项所示。 从`default `重命名，在这种情况下，`foo`，在后面的语法中是显式的，并且在前面的语法中是相同的但是
隐式的。

如果模块具有此类定义，您还可以导入默认导出以及其他命名导出。 从前面回忆这个模块定义：

```
export default function foo() { .. }

export function bar() { .. }
export function baz() { .. }
```

要导入该模块的默认导出及其两个命名导出:

```
import FOOFN, { bar, baz as BAZ } from "foo";

FOOFN();
bar();
BAZ();
```

ES6模块理念强烈建议的方法是，只从需要的模块导入特定绑定。如果一个模块提供了`10`个API方法，但是你只需要它们中的两个，有些人认为引入`整个API`绑定是很浪费的。

除了代码更加明确之外，一个好处是窄导入使得静态分析和错误检测（例如使用错误的绑定名称）更加`健壮`。

当然，这只是受`ES6设计理念影响的标准位置`;没有什么需要坚持这种方法。

许多开发人员会快速指出，这种方法可能更`繁琐`，要求您定期重新访问和更新`import `语句，每当您意识到,需要一个模块中的其他内容。这种折衷方式是为了方便。

在这种情况下，首选项可能是将模块中的所有内容导入单个`命名空间`，而不是将各个成员直接导入范围。幸运的是，`import`语句有一个语法变体，可以支持这种风格的模块消耗，称为`命名空间导入`。

考虑将一个`"foo"`模块导出为：

```
export function bar() { .. }
export var x = 42;
export function baz() { .. }
```

您可以将整个API导入到单个模块绑定的命名空间：

```
import * as foo from "foo";

foo.bar();
foo.x;          // 42
foo.baz();
```

注意：`* as ..`子句需要`*`通配符。 换句话说，你不能像`"foo"`一样为了只引入部分API，使用`import { bar, x } as foo from "foo"`，但仍然绑定到命名空间`foo`。 我会喜欢这样的东西，但相比较命名空间导入，对于ES6它是全部或不存在的。

如果要导入`* as ..`的模块具有默认导出，则在指定的命名空间中将其命名为`default`。 您还可以将命名空间绑定之外的默认导入命名为顶级标识符。 考虑一个`"world"`模块导出为：

```
export default function foo() { .. }
export function bar() { .. }
export function baz() { .. }
```

还有当前的导入：

```
import foofn, * as hello from "world";

foofn();
hello.default();
hello.bar();
hello.baz();
```

虽然这种语法是有效的，但是可以相当困惑的是，模块的一个方法（` default export`）绑定在顶级作用域，而其余的命名导出（和一个名为` default`）在一个不同命名（`hello`）标识符命名空间绑定为属性。

如前所述，我的建议是避免以这种方式设计模块导出，以减少您的模块的用户，将遭受这些奇怪怪癖的几率。

所有导入的绑定都是不可变的`和/或`只读的。 考虑以前的导入; 所有这些后续的赋值尝试都会抛出`TypeErrors`：

```
import foofn, * as hello from "world";

foofn = 42;         // (runtime) TypeError!
hello.default = 42; // (runtime) TypeError!
hello.bar = 42;     // (runtime) TypeError!
hello.baz = 42;     // (runtime) TypeError!
```
回想前面在“`export`ing API成员”部分中，我们讨论了`bar`和`baz`绑定如何绑定到`"world"`模块中的实际标识符。这意味着模块是否更改了这些值，`hello.bar`和`hello.baz`现在引用更新的值。

但是导入本地绑定的`不可变/只读`属性,强制限制不能从导入的`绑定`中更改它们，因此不能从`TypeError`s中更改它们。这很重要，因为没有那些保护，你的更改最终会影响模块的所有其他用户（记住：单例），这可能会产生一些非常令人惊讶的副作用！

此外，虽然模块可以从内部更改其API成员，但是您应该非常谨慎地小心地设计模块。 ES6模块旨在是静态的，因此偏离该原则应该很少，并且应该仔细和详细地记录。

警告：有一些模块设计原则，其中您实际上打算让用户更改您的API上的属性的值，或者模块API设计，为通过将其他“插件”添加到API命名空间来“扩展”。正如我们刚刚断言的，ES6模块API应该被认为，和设计为，静态和不可改变，这强烈地制约和阻止这些替代模块设计模式。你可以通过导出一个简单的对象，来解决这些限制，当然可以随意更改。但是要小心，在走这条路之前要三思。

作为`import `结果发生的声明`"hoisted"`（参见本系列的`Scope & Closures `标题）。试想一下：

```
foo();

import { foo } from "foo";
```

`foo()`可以运行，因为`import ..`语句的静态分辨率决定了编译过程中的`foo()`是什么，但它也将声明“提升”到模块范围的顶部，从而使其在整个模块中可用。

最后，最基本的导入形式如下所示：

```
import "foo";
```

此表单实际上不会将任何模块的绑定导入到范围中。 它加载（如果尚未加载），编译（如果尚未编译），并评估（如果尚未运行）`foo`模块。

一般来说，这种`import`可能不会是非常有用的。 可能存在这样的情况，其中模块的定义具有副作用（例如将东西分配给`window`/`global `对象）。 你也可以设想使用`import "foo"`作为模块的一种预加载，稍后可能需要。

##　Circular Module Dependency

A`imports `B. B`imports `A.这实际上如何工作？

我会说明设计,具有有意，循环依赖的系统的打击，通常是我试图避免的。 那已经说了，我知道有人为此做的原因，它可以解决一些粘性的设计情况。

让我们考虑ES6如何处理这个。 首先，模块`"A"`：

```
import bar from "B";

export default function foo(x) {
    if (x > 10) return bar( x - 1 );
    return x * 2;
}
```

现在, 模块 "B":

```
import foo from "A";

export default function bar(y) {
    if (y > 5) return foo( y / 2 );
    return y * 3;
}
```
这两个函数`foo(..)`和`bar(..)`将作为标准函数声明，如果它们在同一范围内，因为声明“提升”到整个范围，因此不管创建顺序都可以彼此可用。

使用模块，您具有完全不同范围的声明，因此ES6必须执行额外的工作，以帮助使这些循环引用起作用。

在粗略的概念意义上，这是循环`import `依赖性被验证和解决的方式：

- 如果首先加载`“A”`模块，第一步是扫描文件并分析所有`export`，因此它可以注册所有可用于`import`的绑定。然后它处理`import .. from "B"`，它表示它需要去获取`"B"`。

- 一旦引擎加载`"B"`，它对其导出绑定执行相同的分析。当它看到从`import .. from "A"`，所以它可以验证`import `是有效的。现在它知道`"B"`API，它也可以在等待`“A”`模块中验证`import .. from "B"`


实质上，相互导入以及,用于验证两个`import`语句的,静态验证,实际上组成了两个单独的模块范围（通过绑定），使得`foo(..)`可以调用`bar(..)`，反之亦然。这是对称的，如果它们最初被声明在相同的作用域。

现在让我们一起尝试使用这两个模块。首先，我们将尝试`foo(..)`：

```
import foo from "foo";
foo( 25 );              // 11
```
或者我们可以尝试 `bar(..)`:

```
import bar from "bar";
bar( 25 );              // 11.5
```
到`foo(25)`或`bar(25)`调用被执行时，所有模块的所有`分析/编译`已经完成。 这意味着`foo(..)`内部知道直接关于`bar(..) `,`bar(..)`内部直接知道关于`foo（..）`。

如果我们需要的是与`foo（..）`交互，那么我们只需要导入`"foo"`模块。 同样的，使用`bar(..)`和`"bar"`模块也一样。

当然，我们可以`import `和使用它们，如果我们想：

```
import foo from "foo";
import bar from "bar";

foo( 25 );              // 11
bar( 25 );              // 11.5
```

导入语句的,静态加载语义,意味着,通过导入相互依赖的`"foo"`和`"bar"`将确保在任何一个运行之前加载，解析和编译它们。 所以他们的循环依赖是使用如你所愿静态解决解决了。
## 模块加载

我们在这个`"Modules"`部分的开头声明，`import`语句使用托管环境（`浏览器`，`Node.js`等）提供的一个单独的机制，实际上,将`模块说明符字符串`,解析为一些有用的指令，用于查找和加载所需的模块。该机制是系统模块加载器。

环境提供的默认模块加载器会将`模块说明符`解析为`URL`（如果在浏览器中），并且（通常）作为本地文件系统路径（如果在诸如`Node.js`之类的服务器上）。默认行为,是以ES6标准模块格式编写的,假定加载的,文件。

此外，您可以通过HTML标记将模块加载到浏览器中，类似于加载当前脚本程序。在撰写本文时，尚不完全清楚该标记是`<script type ="module">`还是`<module>`。 `ES6`不控制该决定，但是相应标准机构中的讨论已经很好地与ES6并行。

无论标签看起来像什么，你都可以确保在它的覆盖下，它将使用默认加载器（或您预先指定的自定义加载器，我们将在下一节讨论）。

就像您在标记中使用的标记一样，模块加载器本身不是由ES6指定的。它是一个单独的，并行的标准（http://whatwg.github.io/loader/）目前由`WHATWG浏览器标准`组控制。

在撰写本文时，以下讨论反映了早期的通过API设计，并且事情可能会改变。

###  在模块外部加载模块

直接与模块加载器交互的一个用途是如果非模块需要加载模块。试想一下：

```
// normal script loaded in browser via `<script>`,
// `import` is illegal here

Reflect.Loader.import( "foo" ) // returns a promise for `"foo"`
.then( function(foo){
    foo.bar();
} );
```
`Reflect.Loader.import(..)`实用程序将整个模块导入到命名参数（作为`命名空间`），就像`import * as foo ..`我们前面讨论的`foo .`.命名空间`import`。

注意：`Reflect.Loader.import(..) `实用程序返回一个`promise`，一旦模块准备就绪。要导入多个模块，您可以使用`Promise.all([ .. ])`从多个`Reflect.Loader.import(..)`调用组合`promise`。有关Promises的更多信息，请参阅第4章中的“Promises”。

您还可以在实际模块中使用`Reflect.Loader.import(..) `来`动态地/有条件地`加载模块，其中`import`本身不工作。例如，如果一个特性测试显示它没有被当前引擎定义，你可以选择加载一个包含一些`ES7+`特性的`polyfill`的模块。

出于性能原因，您希望尽可能避免`动态加载`，因为它阻碍了`JS`引擎从其静态分析中触发早期抓取的能力。

## 定制加载

另一个用于直接与模块加载器交互的用途是，如果您想通过配置或甚至重新定义来定制其行为。

在写这篇文章的时候，正在开发的模块加载器API有一个`polyfill`（https://github.com/ModuleLoader/es6-module-loader）。虽然细节稀少，并且极有可能可改变，我们只可以探索最终可能会着陆的可能性。

`Reflect.Loader.import(..)`调用可能支持第二个参数，用于指定各种选项来`自定义导入/加载任务`。例如：

```
Reflect.Loader.import( "foo", { address: "/path/to/foo.js" } )
.then( function(foo){
    // ..
} )
```

这是充满期待的，定制将被提供（通过某种方式）用于挂载到加载模块的过程中，其中在加载之后，但在引擎编译模块之前，可以进行` translation/transpilation`。

例如，您可以加载不符合ES6的模块格式（例如，CoffeeScript，TypeScript，CommonJS，AMD）。 然后，您的翻译步骤可以将其转换为符合ES6的模块，以供引擎处理。

## Classes

从JavaScript的开始，语法和开发模式都尽最大努力（读作：struggled），以支持独立的面向类开发。有了像`new `和`instanceof`和一个`.constructor`属性，谁不是情不自禁的认为，JS有类，隐藏在原型系统内的某处？

当然，JS的`“classes”`与经典`类`几乎不相同。差异是有据可查的，所以我不会在这里进一步讨论这一点。

注意：要了解有关在假冒“classes”时使用的模式的更多信息，以及称为“委托”的原型的另一种视图，请参阅本系列的此系列的`this & Object Prototypes`标题的下半部分。

### 类

虽然JS的原型机制的作用不像传统`类`一样，但这并不能阻止对语言扩展语法糖的强烈需求，使表达“classes”看起来更像真正的`类`。引入ES6 `class `关键字及其关联机制。

这个特性是一个高度争议和引人注目的辩论的结果，并且代表一个，从几个强烈反对如何处理JS类的观点，折衷的较小的子集。大多数开发人员想要在JS中使用完整的类，会发现新语法的部分内容非常吸引人，但会发现重要的部分仍然缺失。不过不要担心。 `TC39`已经在开发额外的功能来增强ES6版本后的时间框架中的类。

新的ES6类机制的核心是`class`关键字，它标识一个`块`，其中内容定义了函数原型的成员。试想一下：

```
class Foo {
    constructor(a,b) {
        this.x = a;
        this.y = b;
    }

    gimmeXY() {
        return this.x * this.y;
    }
}
```
有些事情要注意：

- `class Foo`意味着创建一个名为`Foo`的（特殊的）函数，很像ES6之前的版本。
- `constructor(..)`标识`constructor(..)`函数的签名，以及它的主体内容。
- 类方法使用可用于对象字面量的,相同如第2章所述的“简明方法”语法。这还包括本章前面讨论的简洁`generator `形式，以及ES5 getter / setter语法。 但是，类方法是不可枚举的，而对象方法默认是可枚举的。
- 与对象字面量不同，类体中没有逗号分隔成员！ 事实上，他们甚至不允许。

前面代码片段中的类语法定义可以粗略地认为是这个ES6版本之前的的等价物，在那些已经完成的原型样式编码之前，这可能看起来相当熟悉：

```
function Foo(a,b) {
    this.x = a;
    this.y = b;
}

Foo.prototype.gimmeXY = function() {
    return this.x * this.y;
}
```

在前ES6形式或新的ES6类形式，这个“类”现在可以实例化和使用，正如你所期望的：

```
var f = new Foo( 5, 15 );

f.x;                        // 5
f.y;                        // 15
f.gimmeXY();                // 75
```

警告！虽然`class Foo`看起来很像函数`function Foo()`，但他们有重要的区别：

- `class Foo`的`Foo(..) `调用必须使用`new`，因为`Foo.call( obj )`的ES6的之前版本选项将不起作用。
- 虽然`function Foo`“被提权”（参见本系列的范围和封闭标题），但`class Foo`不是; `extends ..`子句指定一个不能“被提权”的表达式。所以，你必须先声明一个类，然后才能实例化它。
- 顶级全局作用域中的`class Foo`在该作用域中创建一个词法`Foo`标识符，但不同于`function Foo`,她不会创建该名称的全局对象属性。


建立的`instanceof`运算符仍然适用于ES6类，因为类只是创建了同名的构造函数。但是，ES6引入了一种使用`Symbol.hasInstance`（请参阅第7章中的“Well-Known Symbols”）来定制`instanceo`f如何发挥作用的方法。

另一种思考类的方法，我觉得更方便，是一个用于自动填充`原型`对象的宏。可选地，如果使用`extends`，它也连接`[[Prototype]]`关系（参见下一部分）。

ES6类实际上不是一个实体本身，而是一个包围其他具体实体的元概念，例如函数和属性，并将它们绑定在一起。

提示：除了声明形式，一个类也可以是一个表达式，如：`var x = class Y { .. }`。这主要用于将类定义（技术上，构造函数本身）作为函数参数或将其分配给对象属性。

`extends `and `super`

ES6类还具有用于在两个函数原型之间建立`[[Prototype]]`委托链接的语法糖 - 通常使用面向类别的熟悉术语扩展的错误标记的`“extends”`或混淆地标记为`“原型继承”`

```
class Bar extends Foo {
    constructor(a,b,c) {
        super( a, b );
        this.z = c;
    }

    gimmeXYZ() {
        return super.gimmeXY() * this.z;
    }
}

var b = new Bar( 5, 15, 25 );

b.x;                        // 5
b.y;                        // 15
b.z;                        // 25
b.gimmeXYZ();               // 1875
```
一个重要的新增加的关键字是`super`，这实际上在pre-ES6是`直接不可能的`（除了有一些打破权衡）。 在构造函数中，`super`自动引用`“父构造函数”`，在前面的示例中是`Foo(..)`。 在一个方法中，它引用`“父对象”`，这样你可以访问它一个属性/方法，如super.gimmeXY()。

`Bar extends Foo`当然意味着将`Bar.prototype`的`[[Prototype]]`链接到`Foo.prototype`。 所以，方法中的`super `像`gimmeXYZ()`方法，特殊的意义是`Foo.prototype`，而当在`Bar`构造函数中使用时，`super`意味着`Foo`。

注意：`super`不限于类声明。 它也适用于对象字面量，与我们在这里讨论的方式大致相同。 有关详细信息，请参阅第2章中的“Object `super`”。


## There Be `super` Dragons

注意到`super`行为根据它出现的位置而有所不同,并不是无关紧要。公平的，大多数时候，这不会是一个问题。但是，如果你偏离狭窄的规范，惊喜等待着。

可能存在这样的情况，在构造函数中，您将要引用`Foo.prototype`，例如直接访问它的一个`属性/方法`。但是，`super`在构造函数中不能以这种方式使用; `super.prototype`将不起作用。 `super(..)`意味着大致调用新的`Foo(..)`，但实际上不是一个可用的`Foo`本身的引用。

对称地，您可能需要在非构造函数方法中引用`Foo(..)`函数。 `super.constructor`将指向`Foo(..)`函数，但要注意，此函数只能使用`new`来调用。`new super.constructor(..)`将是有效的，但它在大多数情况下,不会非常有用，因为你不能使该调用,使用或引用,当前的这个对象上下文，这可能是你想要的。

此外，`super `看起来像它可能由一个函数的上下文驱动的，就像这样 - 也就是说，它们都是动态绑定的。然而，`super `不是`this`这样的动态。当构造函数或方法在声明（在`class `中）内部构造`super `引用时，该`super `静态地绑定到该特定类层次结构，并且不能被覆盖（至少在ES6中）。

这意味着什么？这意味着如果你在习惯上通过覆盖他们的的`this`,从one “class”拿，或从其他的类中“借”，调用`call(..)`或`apply(..)`，这可能很好，如果你所借的方法有一个`super `的话，就会产生惊喜。考虑这个类层次结构：

```
class ParentA {
    constructor() { this.id = "a"; }
    foo() { console.log( "ParentA:", this.id ); }
}

class ParentB {
    constructor() { this.id = "b"; }
    foo() { console.log( "ParentB:", this.id ); }
}

class ChildA extends ParentA {
    foo() {
        super.foo();
        console.log( "ChildA:", this.id );
    }
}

class ChildB extends ParentB {
    foo() {
        super.foo();
        console.log( "ChildB:", this.id );
    }
}

var a = new ChildA();
a.foo();                    // ParentA: a
                            // ChildA: a
var b = new ChildB();       // ParentB: b
b.foo();                    // ChildB: b
```

如你所见，`this.id`引用动态反弹，因此`：a`据考察在两种情况下，而不是`：b`。但是`b.foo()`的`super.foo()`引用没有动态反弹，所以它仍然被视为`ParentB`而不是期望的`ParentA`。

因为`b.foo()`引用了`super`，它静态绑定到`ChildB / ParentB`层次结构，不能针对`ChildA / ParentA`层次结构使用。对于这个限制ES6没有解决方案。

`super `似乎直观地工作，如果你有一个静态类层次结构，没有交叉授粉。但是公平起见，做这种感知编码的主要好处之一就是这种灵活性。简单地说，`class + super`需要你规避这样的技术。

选择归结为，缩小你对这些静态层次结构对象设计 - `class`，`extends`和`super `将是相当不错的 - 或放弃所有尝试“假”类，而不是拥抱动态和灵活，无类的对象和`[[Prototype]]`代理（参见本系列的`this＆Object Prototypes`标题）。

### 子类构造函数

类或子类不需要构造函数;如果省略，则在两种情况下替换默认构造函数。然而，默认的替换构造函数对于直接类和扩展类是不同的。

具体来说，默认的子类构造函数会自动调用父构造函数，并传递任何参数。换句话说，你可以认为默认的子类构造函数是这样的：

```
constructor(...args) {
    super(...args);
}
```
这是一个重要的细节要注意。 不是所有的类语言都有子类构造函数自动调用父构造函数。 C ++可以，但Java不行。 但更重要的是，在`pre-ES6`类中，这种自动的“父构造函数”调用不会发生。 转换到ES6类时要小心，如果你一直依赖这样的一直不发生的调用。

另一个也许令人惊讶的`偏差/限制`的ES6子类构造函数：在子类的构造函数中，您不能访问`this`直到`super(..)`已被调用。 原因比较微妙和复杂，但它归结为事实，父构造函数实际上是一个`创建/初始化`你的实例的`this`。 `Pre-ES6`，它的作用恰恰相反; 这个对象是由“子类构造函数”创建的，然后你调用一个“父构造函数”与“子类”的`this`上下文。

让我们来说明。 在`pre-ES6`的作用：

```
function Foo() {
    this.a = 1;
}

function Bar() {
    this.b = 2;
    Foo.call( this );
}

// `Bar` "extends" `Foo`
Bar.prototype = Object.create( Foo.prototype );
```
但是ES6等效项不允许使用：

```
class Foo {
    constructor() { this.a = 1; }
}

class Bar extends Foo {
    constructor() {
        this.b = 2;         // not allowed before `super()`
        super();            // to fix swap these two statements
    }
}
```

在这种情况下，修复很简单。 只需交换在子类`Bar`构造函数中的两个语句。 然而，如果你一直依靠ES6之前的能力来跳过调用“父构造函数”，请小心，因为这将不再允许。

`extend`ing Natives

对new `class` 和`extend` 设计最有希望的好处之一是能够（最终！）子类的本地内置，如`Array`。 试想一下：

```
class MyCoolArray extends Array {
    first() { return this[0]; }
    last() { return this[this.length - 1]; }
}

var a = new MyCoolArray( 1, 2, 3 );

a.length;                   // 3
a;                          // [1,2,3]

a.first();                  // 1
a.last();                   // 3
```
在ES6之前，使用手动对象创建和链接到`Array.prototype`的`Array`的一个假“子类”只是部分工作。 它错过了一个真正的数组的特殊行为，如自动更新`length `属性。 ES6子类应该完全使用如预期的“继承”和增强的行为！

另一个常见的前ES6“子类”限制是使用Error对象，在创建自定义错误“子类”。 当创建正确的错误对象时，它们会自动捕获特殊的堆栈信息，包括创建错误的行号和文件。 Pre-ES6自定义错误“子类”没有这样的特殊行为，这严重限制了它们的实用性。

ES6对于恢复：

```
class Oops extends Error {
    constructor(reason) {
        super(reason);
        this.oops = reason;
    }
}

// later:
var ouch = new Oops( "I messed up!" );
throw ouch;
```


在此之前的代码段中的,自定义错误对象,将像对待任何其他真正的错误对象一样，包括捕获`堆栈`(`stack`)。 这是一个很大的进步！

### new.target

ES6引入了一个新的概念，称为元属性(`meta property`)（见第7章），以`new.target`的形式。

如果这看起来很奇怪，它是`; `将关键字与一个`.`配对。 并且属性名称肯定是JS的一种不寻常的模式。

`new.target`是一个在所有函数中可用的新的“魔术值”，尽管在正常函数中它总是`undefined`。 在任何构造函数中，`new.target`总是指向`new `实际上直接调用的构造函数，即使构造函数在父类中，并且由子构造函数的`super(..)`调用委托。 试想一下：


```
class Foo {
    constructor() {
        console.log( "Foo: ", new.target.name );
    }
}

class Bar extends Foo {
    constructor() {
        super();
        console.log( "Bar: ", new.target.name );
    }
    baz() {
        console.log( "baz: ", new.target );
    }
}

var a = new Foo();
// Foo: Foo

var b = new Bar();
// Foo: Bar   <-- respects the `new` call-site
// Bar: Bar

b.baz();
// baz: undefined
```

new.target元属性在类构造函数中没有太大的用处，除了访问`静态属性/方法`（参见下一节）。

如果`new.target`未定义，则知道函数未使用`new`调用。 然后，如果必要，您可以强制执行`new `调用。

### static
当子类`Bar`扩展父类`Foo`时，我们已经观察到`Bar.prototype`是`[[Prototype]]` - 链接到`Foo.prototype`。 但另外，`Bar()`是`[[Prototype]]` - 链接到`Foo()`。 这部分可能没有这样明显的推理。

但是，在为一个类声明`静态方法`（而不仅仅是属性）的情况下，这是非常有用的，因为它们直接添加到该类的函数对象中，而不是函数对象的原型对象。 试想一下：
```
class Foo {
    static cool() { console.log( "cool" ); }
    wow() { console.log( "wow" ); }
}

class Bar extends Foo {
    static awesome() {
        super.cool();
        console.log( "awesome" );
    }
    neat() {
        super.wow();
        console.log( "neat" );
    }
}

Foo.cool();                 // "cool"
Bar.cool();                 // "cool"
Bar.awesome();              // "cool"
                            // "awesome"

var b = new Bar();
b.neat();                   // "wow"
                            // "neat"

b.awesome;                  // undefined
b.cool;                     // undefined
```

注意不要混淆`static `成员在类的原型链上。 它们实际上是在函数构造函数之间的`双/平行`链。

### Symbol.species Constructor  Getter

`static `非常有用的一个地方是为派生（子）类设置`Symbol.species` getter（在规范中在内部被称为`@@species`）。 这种能力允许子类向父类发信号通知什么构造函数应该使用 - 当不打算子类的构造函数本身 - 如果任何父类方法需要实例化一个新的实例。

例如，`Array`上的许多方法创建并返回一个新的`Array`实例。 如果你从`Array`定义一个派生类，但是你希望这些方法继续提供实际的Array实例，而不是你的派生类，这是有效的：

```
class MyCoolArray extends Array {
    // force `species` to be parent constructor
    static get [Symbol.species]() { return Array; }
}

var a = new MyCoolArray( 1, 2, 3 ),
    b = a.map( function(v){ return v * 2; } );

b instanceof MyCoolArray;   // false
b instanceof Array;         // true
```

为了说明父类方法如何使用一个子类的species 声明有点像`Array＃map(..)`在做，请试想一下：

```
class Foo {
    // defer `species` to derived constructor
    static get [Symbol.species]() { return this; }
    spawn() {
        return new this.constructor[Symbol.species]();
    }
}

class Bar extends Foo {
    // force `species` to be parent constructor
    static get [Symbol.species]() { return Foo; }
}

var a = new Foo();
var b = a.spawn();
b instanceof Foo;                   // true

var x = new Bar();
var y = x.spawn();
y instanceof Bar;                   // false
y instanceof Foo;                   // true
```

正如你通常期望的,父类`Symbol.species`确实`return this`推迟到任何派生类。 `Bar`然后覆盖,以便于手动声明`Foo`以用于此类实例创建。 当然，派生类仍然可以使用新的`new this.constructor(..)`来声明实例。


## Review

ES6引入了几个有助于代码组织的新功能：

- `Iterators `提供对数据或操作的顺序访问。 他们可以通过新的语言功能，如`for..o`f和`...`.
- `Generators` 是由``Iterators ``控制的本地暂停/恢复功能。 它们可以用于以编程方式（并且通过`yield / next(..)`消息传递）交互地生成要通过迭代的值。
- `Modules `允许使用公开导出的`API`对实现细节进行`私有封装`。 模块定义是基于文件的，单例，并在编译时`静态解析`。
- `Classes `在基于原型的编码上提供更清晰的语法。 添加`super`也解决了在`[[Prototype]]`链中的相对引用的棘手问题。

这些新工具应该是你，尝试通过采用ES6来改进JS项目的架构的第一站。
