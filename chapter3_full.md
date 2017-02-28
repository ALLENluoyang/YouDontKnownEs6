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
