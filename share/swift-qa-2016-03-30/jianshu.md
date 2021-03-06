每周 Swift 社区问答：关于 for-loop 的问题"


作者：[shanks](http://codebuild.me)

发现每周整理的问题比较零散，并且每个问题浅尝辄止，没有深入去讨论和分析。这周开始，尝试对一个问题进行深入讨论，加深每个知识点的影响。后面有余力把 stackoverflow 上的类似问题都归档一下，慢慢形成一些知识整理。这种形式感觉会好一些。
今天我们就谈谈关于 for 循环的问题。




对应的代码都放到了 github 上，有兴趣的同学可以下载下来研究：[点击下载](https://github.com/SwiftGGTeam/SwiftCommunityWeeklyQA/tree/master/20160330)

<!--more-->


## Question: For loop decrement in Swift 2.2
[点击打开问题原地址](https://forums.developer.apple.com/thread/43416)
### 问题描述

此问题是关于 for 循环的。大家都比较习惯 c 语言的 for 循环方式。如下：

    for var i=10; i>=0; --i {
        print("\(i)")
    }

楼主只知道， 在 Swift 中，使用 `...` 操作符表示一个序列，`0...10` 表示 0 到 10。但是以上代码是从10 到 0，而 `10...0` 会报错。

    for i in 10...0 {
        print("\(i)")
    }
### 问题解答

答案是需要 stride 方法的协助，或者使用 reverse 方法进行反转：

    for i in 10.stride (through: 0, by: -1) {
        print("\(i)")
    }
    
    for i in (0...10).reverse() {
        print("\(i)")
    }
    
    (0...10).reverse().forEach{print($0)}

大家在最新的 Xcode7.3 中，使用 Playground 运行c 语言的 for 语法，会收到 2 个警告：

#### 第一个警告：'--' is deprecated: it will be removed in Swift 3.
 
 -- 和 ++ 操作符将在 Swift 3 中消失，这是 Lattner 的提议，具体可以参见：[Remove the ++ and -- operators](https://github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md)。
 
Lattner 首先说了 2 点 -- 和 ++ 操作符的好处：

* 这 2 个操作符比 += 1 和 -= 1 这样的写法更短，同时也比`x.advance()`这样的迭代方法也更简洁。在需要返回一个值的时候，+= 不能返回值，要另外写一行来表示返回，而 ++ 是可以的：

    func add() -> Int {
    	var a = 1
    	return a += 1 // 这样只能返回空，而不能返回 2。所以这一行代码报错
    }
    
    func add1() -> Int {
        var a = 1
        return ++a // 返回 2
    }
    
    add1()
    
    var f: Float = 1.1
    f++
    f
 
* 第二个好处，传承了 c 语言系(C++, Objective-C, Java, C#, Javascript, etc)的语法, 熟悉这些语言的程序员当然希望能有这些熟悉的操作符了。
 
 
不过接下来，Lattner 提了 7 点使用 ++ 和 -- 操作符的缺点（比优点多：），7 >> 2，所以得去掉）：


* Swift 如果作为入门语言，这些操作符的理解会增加一些学习难度，除非你知道，这些操作符是来自于其他语言的。

* 其实 `x++` 比 `x += 1` 也短不了多少

* 对应 =，+= 这一类的操作符，Swift 在处理他们和 return 之间的关系时候， Swift 返回的空（Chris 说这样设计，很多原因，不过没说具体，待考证），但是递增和递减操作符，却是返回具体的值。这与返回空的设计，不太一致。

* Swift 中提供了很多替代的实现方案，比如： for-in, ranges, enumerate, map 等。

* 递增操作符放在变量前后有一些差别，增加了理解的难度（这也是面试遇到过的基础题。。），虽然很取巧，但是的确增加了理解的难度。

* Swift 在代码的执行顺序上有着良好的定义，所以类似于`foo(++a, a++)`不太好懂。

* 递增递减操作符只适用于少量的类型：整型，浮点类型和迭代器，并不适用于一些复杂的类型，比如矩阵。

基于以上的整理，准备在 Swift 3 中，去掉递增和递减操作符。
 
 
 
#### 第二个警告：C-style for statement is deprecated and will be removed in a future version of Swift.
    
C 语言的 for 语句语法形式将在未来的 Swift 版本中消失。这个提议是由 Erica Sadun(我们翻译组一直翻译他的文章)提出的：[Remove C-style for-loops with conditions and incrementers](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md)。

Erica 首先说到了 for-loop 的好处，作为一个熟悉的语法，降低了学习 Swift 循环语句的门槛。仅此而已，后面写了 6 个缺点：

* for-in 和 stride 已经提供了相同的语法，并且使用的是更加 Swift 风格的方式。
* for-loop 没有 for-in 语法简洁。
* for-loop 不适用于集合类型和其他一些核心的 Swift 类型。
* for-loop 鼓励使用递增递减操作符，而这种操作符将来会去掉。
* 如果 for-loop 不存在的话，我都怀疑 Swift 3 是否会记起它！
    理由还是很充分的，所以大家尽量少用 for-loop，多用 for-in, 不然 Swift 3 发布以后，就编译不过啦。







