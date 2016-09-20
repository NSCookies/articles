[类簇](https://developer.apple.com/library/content/documentation/General/Conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html#//apple_ref/doc/uid/TP40010810-CH4-SW1)用通俗一点讲就是一个`public`的抽象类加上一些`private`的私有类构成的，它是对一些实现细节进行隐藏，而对外公开的行为进行统一的一种设计。相信大家在平常工作中多少有注意到一些蛛丝马迹。例如我们常用的`NSNumber`, `NSArray`, `NSDictionary`以及`NSString`等，而这些都是总所周知。

然而就`UIButton`是不是类簇，本王就纠结了。To be or not to be, that's a question. 这时候就应该装逼了，搬出莎士比亚这句话。 顺带就带着这个来说说类簇的问题。关于为什么纠结呢？因为很多地方包括书籍都提到`UIButton`是类簇，而我再[Stack Overflow](http://stackoverflow.com/questions/5045672/create-uibutton-subclass)却找到这样一段话:

> UIButton is not a class cluster at all. A class cluster is represented by a public abstract class, that means no instance variables, with a bunch of private concrete subclasses that provide the implementation of the abstract methods of the abstract class. UIButton on the other hand is a concrete class, none of its methods is abstract, and it has instance variables to store the value you pass through its arguments. The only problematic part is that +buttonWithType can instantiate subclasses instead of UIButton directly, thus it can be seen as a factory method, not a class-cluster...

然后我就懵逼了，根据类簇大体的概念我们知道至少说如果`UIButton`是一个抽象类的话，那么应该还存在一些`private`的私有类来实现具体的细节。那么我们的任务就是就是找到这些私有类，之前看了[@我就叫Sunny怎么了 ](http://weibo.com/u/1364395395?topnav=1&wvr=6&topsug=1)的一篇文章[从NSArray看类簇](http://blog.sunnyxx.com/2014/12/18/class-cluster/)后发现至少`UIButton`中没办法用这种方法，于是就想说用LLDB断点一下，用了下面这条命令：

```
breakpoint set -F '+[UIButton buttonWithType:]'
```

得到了这样一段汇编：

```
UIKit`+[UIButton buttonWithType:]:
->  0x1063251e5 <+0>:    pushq  %rbp
..... // 省略
    0x106325297 <+178>:  movq   0x972c02(%rip), %rdi      ; (void *)0x0000000106cbabf8: UIButton
    0x10632529e <+185>:  movq   0x93cf6b(%rip), %rsi      ; "alloc"
    0x1063252a5 <+192>:  movq   0x9c0f34(%rip), %rbx      ; (void *)0x0000000105650800: objc_msgSend
    0x1063252ac <+199>:  callq  *%rbx
    0x1063252ae <+201>:  movq   0x93ce2b(%rip), %rsi      ; "initWithFrame:"
..... // 省略
    0x10632531c <+311>:  movq   0x97427d(%rip), %rdi      ; (void *)0x0000000106cbad10: UIPopoverButton
    0x106325323 <+318>:  movq   0x93cee6(%rip), %rsi      ; "alloc"
    0x10632532a <+325>:  movq   0x9c0eaf(%rip), %rbx      ; (void *)0x0000000105650800: objc_msgSend
    0x106325331 <+332>:  callq  *%rbx
    0x106325333 <+334>:  movq   0x9557a6(%rip), %rsi      ; "initWithFrame:buttonType:"
    0x10632533a <+341>:  movq   -0x38(%rbp), %rcx
..... // 省略
    0x1063253c5 <+480>:  movq   0x9741c4(%rip), %rdi      ; (void *)0x0000000106cbac48: UIRoundedRectButton
    0x1063253cc <+487>:  jmp    0x106325553               ; <+878>
    0x1063253d1 <+492>:  movb   %r14b, -0x81(%rbp)
    0x1063253d8 <+499>:  movq   0x972ac1(%rip), %rdi      ; (void *)0x0000000106cbabf8: UIButton
    0x1063253df <+506>:  jmp    0x106325553               ; <+878>
    0x1063253e4 <+511>:  movb   %r14b, -0x81(%rbp)
    0x1063253eb <+518>:  movq   %rbx, -0x80(%rbp)
    0x1063253ef <+522>:  movq   0x973262(%rip), %rdi      ; (void *)0x0000000106cb45a0: UINavigationButton
    0x1063253f6 <+529>:  movq   0x93ce13(%rip), %rsi      ; "alloc"
    0x1063253fd <+536>:  movq   0x9c0ddc(%rip), %rbx      ; (void *)0x0000000105650800: objc_msgSend
    0x106325404 <+543>:  callq  *%rbx
    0x106325406 <+545>:  movq   0x9556cb(%rip), %rsi      ; "initWithTitle:style:"
    0x10632540d <+552>:  xorl   %edx, %edx
    0x10632540f <+554>:  xorl   %ecx, %ecx
    0x106325411 <+556>:  jmp    0x106325475               ; <+656>
    0x106325413 <+558>:  movb   %r14b, -0x81(%rbp)
    0x10632541a <+565>:  movq   %rbx, -0x80(%rbp)
    0x10632541e <+569>:  movq   0x973233(%rip), %rdi      ; (void *)0x0000000106cb45a0: UINavigationButton
    0x106325425 <+576>:  movq   0x93cde4(%rip), %rsi      ; "alloc"
    0x10632542c <+583>:  movq   0x9c0dad(%rip), %rbx      ; (void *)0x0000000105650800: objc_msgSend
    0x106325433 <+590>:  callq  *%rbx
    0x106325435 <+592>:  movq   0x95569c(%rip), %rsi      ; "initWithTitle:style:"
    0x10632543c <+599>:  xorl   %edx, %edx
    0x10632543e <+601>:  movl   $0x1, %ecx
    0x106325443 <+606>:  jmp    0x106325475               ; <+656>
    0x106325445 <+608>:  movb   %r14b, -0x81(%rbp)
    0x10632544c <+615>:  movq   %rbx, -0x80(%rbp)
    0x106325450 <+619>:  movq   0x973201(%rip), %rdi      ; (void *)0x0000000106cb45a0: UINavigationButton
    0x106325457 <+626>:  movq   0x93cdb2(%rip), %rsi      ; "alloc"
    0x10632545e <+633>:  movq   0x9c0d7b(%rip), %rbx      ; (void *)0x0000000105650800: objc_msgSend
    0x106325465 <+640>:  callq  *%rbx
    0x106325467 <+642>:  movq   0x95566a(%rip), %rsi      ; "initWithTitle:style:"
..... // 省略
    0x10632549e <+697>:  movq   0x9740f3(%rip), %rdi      ; (void *)0x0000000106cbacc0: UITexturedButton
    0x1063254a5 <+704>:  jmp    0x106325553               ; <+878>
    0x1063254aa <+709>:  movb   %r14b, -0x81(%rbp)
    0x1063254b1 <+716>:  movq   %rbx, -0x80(%rbp)
    0x1063254b5 <+720>:  movq   0x9740d4(%rip), %rdi      ; (void *)0x0000000106cbac48: UIRoundedRectButton
    0x1063254bc <+727>:  movq   0x93cd4d(%rip), %rsi      ; "alloc"
    0x1063254c3 <+734>:  movq   0x9c0d16(%rip), %r14      ; (void *)0x0000000105650800: objc_msgSend
..... // 省略
    0x106325538 <+851>:  movq   0x974069(%rip), %rdi      ; (void *)0x0000000106cbad60: _UIPlacardButton
    0x10632553f <+858>:  jmp    0x106325553               ; <+878>
    0x106325541 <+860>:  movb   %r14b, -0x81(%rbp)
    0x106325548 <+867>:  movq   %rbx, -0x80(%rbp)
    0x10632554c <+871>:  movq   0x97405d(%rip), %rdi      ; (void *)0x0000000106cbadb0: _UIShortPlacardButton
    0x106325553 <+878>:  movq   0x93ccb6(%rip), %rsi      ; "alloc"
    0x10632555a <+885>:  movq   0x9c0c7f(%rip), %rbx      ; (void *)0x0000000105650800: objc_msgSend
    0x106325561 <+892>:  callq  *%rbx
    0x106325563 <+894>:  movq   0x93cb76(%rip), %rsi      ; "initWithFrame:"
..... // 省略
```

看到这里小伙伴不要害怕，吃口粑粑冷静一下。我不是要大家看每一句话是什么意思，大家可以注意一下每一行最后又`Button`关键字的地方，会发现一堆我们平时没见过的一些`Button`,如`UIPopoverButton`, `UIRoundedRectButton`, `UINavigationButton`, `UITexturedButton`, `_UIPlacardButton`以及`_UIShortPlacardButton`等。至少我们发现了我们所说的`private`的私有类，那么我们就可以知道其实这里的`UIButton`是一个类簇。

那么我们也用官网的图来说说类簇的用处：

![NSNumber](https://developer.apple.com/library/content/documentation/General/Conceptual/CocoaEncyclopedia/Art/cluster3.gif)

可以从上图看到虽然官方的实现细节根据不同的类型有不同的实现，但是我们所需要记得以及交互的接口都可以通过`NSNumer`来进行交互。而如果我们自己在设计类簇的时候也可能通过这种方式来进行隐藏一些细节的实现。

但是在设计自己的类簇的过程中需要注意一下几点：

* 首先要定义好抽象基类
* 其次需要指明子类需要重写的方法
* 最后提供一个比较好的文档说明方面其他人读写

最后就到这吧，小伙伴各回各家，各找各妈吧~

> PS:具体代码可以从[Github](https://github.com/NSCookies)上获取。

> 如有问题或纠正, 可以联系[@叫什么都不如叫Pluto-Y](http://weibo.com/plutoy0504)或在[Github](https://github.com/NSCookies)