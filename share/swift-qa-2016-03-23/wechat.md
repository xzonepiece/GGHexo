每周 Swift 社区问答 2016-03-23（第十四期）"


作者：[shanks](http://codebuild.me)

本周苹果发布了 Swift2.2, 语言又往前迈进了一小步，开源后的第一个版本，社区也会更加活跃。本周共整理了 4 个问题。涉及问题有：重写问题，结构体和枚举的结合问题，协议编程和面向对象编程问题，defer不执行的问题。


本周整理问题如下:

* [Overriding function signature--documented?](#Question1)
* [Swift syntax for struct: variations ?](#Question2)
* [Why protocol is better than class in swift? ](#Question3)
* [Defer block is not executed](#Question4)





对应的代码都放到了 github 上，有兴趣的同学可以下载下来研究：[点击下载](https://github.com/SwiftGGTeam/SwiftCommunityWeeklyQA/tree/master/20160323)

<!--more-->

<a name="Question1"></a>
## Question1: Overriding function signature--documented?
[点击打开问题原地址](https://forums.developer.apple.com/thread/43120)
### 问题描述

楼主的问题是，B 是 A 的子类，在 D 中定义了一个方法`arbitraryFactory() -> A`，返回 A 的对象，E 继承 D，定义方法`arbitraryFactory() -> B`。必须加上`override`关键字才能通过编译。但是 D 中的方法`identify(anID: A)`，E 中定义的方法`identify(anID: B)`就不用使用`override`，楼主查不到有文档对此进行了说明，跪求大神解释：

    class A {
    }
    class B : A {
    }
    class D {
        func arbitraryFactory() -> A {
            return A()
        }
        func identify(anID: A) {
            print(anID)
        }
    }
    class E : D {
        override func arbitraryFactory() -> B {  // subclass as return type similar enough to require "override"
            return B()
        }
        
        func arbitraryFactory() -> E { // different enough not to require "override"
            return E()
        }
        
        func identify(anID: B) { // subclass as input different enough not to require "override"
            print(anID)
        }
    }

### 问题解答

看了一下[官方文档](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Inheritance.html)对`override`关键词的解释。的确没有讲的太深入。重写在其他语言中也是一个常见的语法，苹果可能觉得其他语言都讲了很多，没必要再讲的太细吧。。
楼下的同学做了一下解释，我觉得讲的比较好：
决定一个函数定义的标志是函数名，输入参数和返回类型。E 中定义的方法`arbitraryFactory() -> E`,返回值是 E 的对象，跟父类 D 中的同名方法返回值 A 没有关系，所以不使用`override`关键字来修饰。但是另外一个方法`arbitraryFactory() -> B` 就不一样，B 是 A 的子类，编译器认为这2个方法是相同的函数类型。必须使用`override`表明是重写。不过在调用时候，需要赋值的变量需要显式声明类型，因为方法名称重了。以下代码会报错：

    //let f = E().arbitraryFactory()
    
    /*:
     正确的写法:
     */
    let f1: A = E().arbitraryFactory()
    let f2: B = E().arbitraryFactory()
    let f3: E = E().arbitraryFactory()
在函数名和返回值相同的情况下，输入参数不能通过继承关系来推断是否一样，也就是说，`identify(anID: A)`和`identify(anID: B)`是不同的函数类型。




<a name="Question2"></a>

## Question2: Swift syntax for struct: variations ?
[点击打开问题原地址](https://forums.developer.apple.com/thread/42887)
### 问题描述

楼主的问题，如何在结构体里面实现以下的伪代码功能，根据属性的值，定义不同的新属性。以下代码是楼主能想到的代码，但是通不过编译：

    import UIKit
    
    struct Item {
        var selectionNum: Int
        var selectionName : String
        var itemCase : Int
        switch itemCase {
        case 0 :
        var paramNum: Int
        var paramImage: NSImage
        case 1 : var paramText: String
    }

### 问题解答

Swift 的枚举，就是为了这种应用场景而设计的，可以使用关联值来解决此问题：

    struct Item {
        var selectionNum: Int
        var selectionName : String
        enum ItemCase {
            case Image (a: Int, b: NSURL?)
            case Text (c: String)
        }
        var anItemCase : ItemCase
        init(num: Int, name : String, aCase : ItemCase) {
            selectionNum = num
            selectionName = name
            anItemCase = aCase
        }
    }
    var example : Item = Item(num: 0, name: "", aCase: .Image(a:1, b:nil))
请阅读翻译组翻译的经典长文：[Swift 中枚举高级用法及实践](http://swift.gg/2015/11/20/advanced-practical-enum-examples/), 枚举知识一网打尽。


<a name="Question3"></a>
## Question3:Why protocol is better than class in swift?
[点击打开问题原地址](http://stackoverflow.com/questions/36145986/why-protocol-is-better-than-class-in-swift)
### 问题描述

楼主的问题是，为什么面向协议编程优越感强于面向对象编程？

### 问题解答

本来准备提笔写一些自己的理解，结果发现翻译组的小乔同学分析了，就不班门弄斧了，同时还有ray的文章也可以看看：
 
 * [Swift面向协议编程初探 | WWDC 2015学习笔记](http://wxgbridgeq.github.io/blog/2015/07/21/protocol-oriented-programming-first/)
 * [Introducing Protocol-Oriented Programming in Swift 2](https://www.raywenderlich.com/109156/introducing-protocol-oriented-programming-in-swift-2)


<a name="Question4"></a>
## Question4: Defer block is not executed
[点击打开问题原地址](http://stackoverflow.com/questions/36169415/defer-block-is-not-executed)
### 问题描述

以下代码中，defer 里面的打印信息不会输出。作者比较疑惑。2个例子都是这样的。

    func A() {
        print ("Hello")
       
        guard 1 == 2 else {
            return
        }
        defer {
            print ("World")
        }
    }
    
    A()
    
    import Foundation
    
    enum MyError: ErrorType {
        case TriggerDefer
    }
    
    func throwsMyError() throws {
        let myzero = Int(arc4random_uniform(1))
        
        guard myzero > 1 else {
            throw MyError.TriggerDefer
        }
    }
    
    func A1() throws {
        try throwsMyError()
        
        defer {
            print ("Hello World")
        }
    }
    
    try A1()

### 问题解答

问题出在 defer 出现的位置，上例中，guard 代码段里面的 return 已经执行了。后面的 defer 语句不会执行。正确的答案见以下代码, 就可以在控制台输出中看到 Hello World 了

    func A3() {
        print ("Hello")
        defer {
            print ("World")
        }
        guard 1 == 2 else {
            return
        }
    }
    A3()

记住：defer 语句写到函数开头就好了。

































