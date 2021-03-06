关于 Swift 演变的趣味探讨"

> 作者：Erica Sadun，[原文链接](http://ericasadun.com/2015/12/15/interesting-discussions-on-swift-evolution/)，原文日期：2015-12-15
> 译者：[小袋子](http://daizi.me)；校对：[Cee](https://github.com/Cee)；定稿：[numbbbbb](http://numbbbbb.com/)
  









记得我曾分享过一些想法和建议，比如：

### newtype

一个是建议 Swift 推出一个 `newtype` 的关键词，它可以添加完全不同于原生的可扩展的派生类型。例如：

    
    newtype Currency = NSDecimal

这创建了一个拥有所有 `NSDecimal` 所有行为的 `Currency` 类型。然而，你不能让一个 `NSDecimal` 类型的元素和一个 `Currency` 类型的元素相加，因为 Swift 中有类型检测。此外，你也可以扩展 `Currency` 类型。这样看起来就更加有针对性，因为不需要子类化或者添加新的存储属性。



`newtype` 的另一个特性是能够创建柯里化类型：

    
    newtype Counter<A> = Dictionary<A, Int>

类型是部分确定的，具体行为可以在扩展中实现，从而能包含键（key）类型不相同但值类型都是 Int 的字典。

期待看到你们的评论。

### self

另外一个提议是将 `self` 作为强制前缀，取代上下文语境推断。Greg Parker 在回复中写道：

> 在 Objective-C 中 `self.property` 这种写法很不优雅。
>
> 第一种方法是只使用 `property`。但是同名变量（ivar）会产生歧义，Swift 没有这样的问题。
>
> 第二种方法是用 `property` 访问属性，用 `self->ivar` 去访问同名变量。这是不可行的，因为会和现有的大量代码冲突。Swift 也没有这样的问题。


### 前置条件与断言（Precondition vs Assert）

Dave Abrahams 提出了一个有关重命名断言和前置条件的建议，我立刻将其中的一些深刻见解记在笔记本上：

> 从语言设计层面来说，这两个函数扮演不同的角色：
> – assert：检查内部的错误代码。
> – precondition：检查客户端给你的参数是否有效。
>
> 两者的区别很大，第二个要求有公共文档，第一个不需要。
>
> 例如：在 Swift 的标准库中，我们保证永远不会出现内存错误，除非你调用 (Obj)C 代码或者使用一个明确地标着「unsafe」的结构。我们需要去检验客户端参数，为了避免给了非法的参数引起内存泄露，我们要在参数中文档化这些需求作为前置条件，并且使用（等价的）precondition() 去检验它。我们还有一系列的内部合理检查，用以确定我们代码假定的正确性，而类型系统还不能保证这个代码的假定。由于这些原因，我们使用（等价的）assert()，因为我们不想降低*你的*代码性能（使用合理的检查）。
>
> 下面是几个具体的例子：

      
      /// 一个集合，其中的元素类型为 Element
    
      public struct Repeat<Element> : CollectionType {
        ...
        /// 获取 `position` 位置的元素
        ///
        /// - 要求: `position` 是 `self` 中的有效位置并且 `position != endIndex`.
        public subscript(position: Int) -> Element {
          _precondition(position >= 0 && position < count, "Index out of range")
          return repeatedValue
        }
      }
      extension String.UTF8View {
        ...
       private func _encodeSomeContiguousUTF16AsUTF8(i: Int) -> (Int, UTF8Chunk) {
          _sanityCheck(elementWidth == 2)
          _sanityCheck(!_baseAddress._isNull)
       
          let storage = UnsafeBufferPointer(start: startUTF16, count: self.count)
          return _transcodeSomeUTF16AsUTF8(storage, i)
        }
      }

> 在第一个例子中，我们有一个判断客户的 collection 没有越界的前置条件。在这个例子中，我们其实可以不做检查，因为越界也不会导致内存错误（因为返回的都是同一个 repeatedValue），但是我们还是加上了这个检查，这样我们的用户可以快速发现他们的 bug 。
>
> 第二个例子中是一个私有函数，它只能在我们保证 elementWidth == 2 和 _baseAddress 不为 null 的条件下调用（_sanityCheck 在 stdlib 下等价于 assert）。因为这是私有函数，使用者就是我们自己，所以看起来这个检查可以省略。但是有时候会出意外，比如后续的开发者可能会错误地使用它，因此我们需要添加检查。因为我们在 debug 和 release 的环境下运行我们的测试，并且有较高的测试覆盖率，因此（如果错误使用函数）断言很可能在某处被触发。
>
> 读完上面的内容，你可能认为 assert() 只能在私有方法中使用，而 precondition() 只能在公共方法中使用。事实并非如此；你可以内联任何私有方法到继承的公有方法的方法体内，因此合理的检查依然有意义。前置条件检查也会偶尔在私有方法中使用，最简单的例子就是公有方法转私有方法，复制代码的时候可以把原来的前置条件检查提取成一个私有的辅助方法（Helper）。
>
> <sup>*</sup>注意，有些前置条件实际上不会被执行，所以你不能指望所有的前置条件都被执行。


> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。