## Swift: 通过`weak self`理解内存泄漏

在Halo，我们在思考做健壮的软件设计上花了和神经科学和体育运动一样多的时间。对于那些想深入Swift引用计数的人来说，下面是我最近调试过程中的一些笔记。

由于Swift使用引用计数( ARC )来确定是否应该释放内存，工程师应该知道何时使用强引用和弱引用来避免内存循环引用。

最常见的做法是将代码中的每个对象都看成是，具有父与子关系联系的对象。父对象保留对其子对象的强引用，每个子对象保留对其父对象的弱引用(在必要的时候)。这将允许父对象不让子对象的内存被释放掉，但反之亦然。

这个做法在大部分情况下都有效，但是仍然有一些微妙的方法来创建带有闭包的循环引用(的确，我很难发现这一点)。

### 发现错误

Check out the following five view controller implementations. Two of them are correct. The rest either leak memory, compile with a warning, or both:

<script src="https://gist.github.com/alanf/cee87d89be6863eeab479b57f63e2811.js"></script>

Answers follow — don’t read ahead if you’re still reasoning about each implementation would behave!

## Answers

ViewControllerA leaks memory, it is a canonical example of a retain cycle. It has a strong reference to a timer, and the timer has a closure which has a strong reference to ViewControllerA (it references self within the event handler’s animation block).

ViewControllerB is correct. It has a strong reference to a timer, but the Swift compiler does not create a strong reference to self since self is never referenced in the event handler code or the nested closure.

ViewControllerC compiles with a warning and leaks memory. The warning is because self is never referenced, although it’s named as a weak parameter to the closure. More on why it leaks memory shortly.

ViewControllerD is correct. It correctly uses [weak self] to avoid a reference cycle.

ViewControllerE leaks memory and is an example of how there’s a subtle programmer error lurking for those who don’t know the implicit behavior of Swift closures.

Even though self isn’t referenced in the outer closure, a strong reference to self is created because Swift does not “look ahead” to see if the nested closures describe a strong or weak reference to self. The Swift compiler creates a strong reference because self is referenced by the nested closure when [weak self] is specified, which leads to a retain cycle.

There is no warning that a strong reference to self is created but never referenced “strongly”. Even in ViewControllerC, where self is never referenced in the closure, the Swift compiler does not optimize away the behavior of strongly referencing self on behalf of the nested closure.

## This has bitten others

A cursory search shows that this subtle behavior has bitten others. A proposal to at least warn about this case was discussed on the Swift mailing list in 2015 but has not advanced beyond the mailing list discussion.

So use Swift closures with caution, and please encourage the Swift community to at least warn of these scenarios (I asked the mailing list proposal author if he would revisit his proposal).
