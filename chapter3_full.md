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

注意：我调用这些隐式接口不是因为它们没有在规范中明确调用 - 他们确实是！ - 但是因为它们没有暴露为代码可访问的直接对象。 JavaScript在ES6中不支持任何“接口”的概念，所以对自己的代码是纯粹的遵守常规。 然而，JS期望一个迭代器  ，一个`for..of`循环，例如 - 你提供的，必须遵循的接口或将失效的代码。

还有一个`Iterable`接口，它描述为必须能够产生迭代器的对象：

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


+
内置迭代器将始终返回此形式的值，但是更多属性当然可以根据需要显示在返回值上。

例如，自定义迭代器可以向结果对象添加附加元数据（例如，数000000000000000据来自何处，取回所花费的时间，缓存期满长度，适当的下一请求的频率等）。

注意：在技术上，如果否则将被认为缺少或未设置，例如在值未定义的情况下，value是可选的。 因为访问res.value将产生未定义是否存在该值或完全不存在，属性的存在/不存在更多是实现细节或优化（或两者），而不是功能问题。
