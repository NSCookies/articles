[Tagged Pointer](https://en.wikipedia.org/wiki/Tagged_pointer) What the hell is this?冷静！冷静！不要激动，从名字来上看应该是一个“打标记"的指针？其实想想好像以前见过这个名字, 对以前一定看过这个。于是乎我有开始搜索大脑以及程序员的资源库——Google。

soga，原来是这个回事。好了，这节就到这里了。(赶紧说，别废话!)既然你诚心诚意的发问了,为了防止世界被破坏,为了维护世界的和平...&%*%&*^&&^$。谁用拖鞋扔我！

好吧，且听我静静装逼，不对，且听我慢慢说来。所谓的`Tagged Pointer`其实就是一个带着"真实"的数据信息的指针。说到底他也就是一个伪指针。为什么这么说呢？

我们来先看两个例子：

```language-objectivec
NSNumber *num1 = [NSNumber numberWithInt:10];
NSNumber *num2 = [NSNumber numberWithInt:10];
NSLog(@"pointer of num 1: %p", num1);
NSLog(@"pointer of num 2: %p", num2);
NSNumber *num3 = [NSNumber numberWithInt:0xfffff];
NSNumber *num4 = [NSNumber numberWithInt:0xfffff];
NSLog(@"pointer of num 3: %p", num3);
NSLog(@"pointer of num 4: %p", num4);

// 其输出结果为:
2016-07-31 23:32:08.066 OCCodes[3773:205937] pointer of num 1: 0xb0000000000000a2
2016-07-31 23:32:08.066 OCCodes[3773:205937] pointer of num 2: 0xb0000000000000a2
2016-07-31 23:32:08.066 OCCodes[3773:205937] pointer of num 3: 0xb000000000fffff2
2016-07-31 23:32:08.066 OCCodes[3773:205937] pointer of num 4: 0xb000000000fffff2
```

非常奇怪，正常我们创建两个实例的时候，它应该前后两个实例地址的指针应该是不一样的，可是例子中的两组实例中，没组的实例的指针都是一样的。而且很奇怪的是指针的地址都是这种`0xb....2`形式，并且2旁边的数值正好是`NSNumber`存的值。难道这些都是巧合吗？

答案是：并不是的。其实这个正是`Tagged Pointer`的真实面目，表面上看他是一个指针，可是实际上他并不是真正的指针，而是包含了`NSNumber`具体信息的伪指针。但是这种"指针"只能存储数值较小的情况下，例如如下的例子则不能保存成`Tagged Pointer`的形式：

```language-objectivec
NSNumber *num5 = [NSNumber numberWithDouble:M_PI];
NSNumber *num6 = [NSNumber numberWithDouble:M_PI];
NSLog(@"pointer of num 5: %p", num5);
NSLog(@"pointer of num 6: %p", num6);

// 其输出结果为:
2016-07-31 23:39:34.375 OCCodes[3822:210131] pointer of num 5: 0x7fe70140e030
2016-07-31 23:39:34.375 OCCodes[3822:210131] pointer of num 6: 0x7fe70170aab0
```

可以看出当如果存储的数值所需的字节数较大的情况下，则以正确的指针来进行保存，而只有当存储的数值所需的位数较少时，则以`Tagged Pointer`的形式进行存储。那么，有人就要问了。为什么不都以真实的指针来进行存储呢？你猜！(瞬间一双拖鞋就过来了！)

好吧，其实是在iPhone 5s引入的过程中其实也引入了64位的处理器，而在引入64位处理器的过程中，指针所占的位数也相应的变为了64位(而在以前的32位处理器的时候指针都是以32位的形式存储)。而如果我需要创建一个`NSNumber`的实例，则我需要一个8个字节(64位)的指针以及在堆中分配一定空间来进行存储(大概是在8字节左右)。也就是说我需要16个字节才能存储一个`NSNumber`的对象。而对于一个4个字节的存储空间来说以及可以存储20+亿的整数了。

这个时候`Tagged Pointer`就闪亮登场了。它的目的就在于，当处于64位处理器的系统中将所有信息都存在栈的指针中，并通过一定的的特殊标记位来说明该指针为一个`Tagged Pointer`，剩余的一些指针的位数则用来存储`NSNumber`的具体数值(如上方的`0xb000000000fffff2`)。

其实这么做除了能节省CPU的空间以外，还能提高处理`NSNumber`的速度，因为他不需要再从堆中申请和释放空间，同时在读取和存储上的速度也加快了。

> 根据WWDC2013的[Session 404 Advanced in Objective-C](https://developer.apple.com/videos/play/wwdc2013/404/)中提到了，在内存上读取的速度快了3倍，创建时的速度比以前快乐106倍。

同时除了`NSNumber`能以这种这种形式存储外，`NSDate`也是做了同样的优化。而以上所说的这些`Tagged Pointer`是针对64位处理器来说的，在32位系统的情况下，这些还是以指针+堆中的对象进行存储的。在64位处理器并且其存储的实际数值没法用`Tagged Pointer`保存的情况下，也是与32位系统中一样的存储方式。

由于`Tagged Pointer`是一个伪指针，所以在使用过程中有两点需要注意的：
* 对于`NSObject`中的`isa`指针其实是没有的，毕竟它不是一个真正的对象
* 对于在MRC下，其retainCount也是不正确的，它采用了一个非常大的整数来进行返回，这样也能保证其能在内存中长时间的存在

总的来说，`Tagged Pointer`的出现是为了在64位系统中提升速度的。而`Tagged Pointer`则是通过一定的标志位以及将真实信息存入在指针中。所以`Tagged Point`并不是一个真正意义上的指针，而是包含了真实信息的指针。就是这样啦。

顺带献上Swift代码，请各位大王笑纳:
```language-swift
// MARK: Tagged Pointer
let num1 = NSNumber(long: 0xffff)
let num2 = NSNumber(long: 0xffff)
let pointFormatStr1 = NSString(format: "tagged point :%p", num1)
let pointFormatStr2 = NSString(format: "tagged point :%p", num2)


let num3 = NSNumber(float: Float(M_PI))
let num4 = NSNumber(float: Float(M_PI))
let pointFormatStr3 = NSString(format: "tagged point :%p", num3)
let pointFormatStr4 = NSString(format: "tagged point :%p", num4)

let date1 = NSDate(timeIntervalSince1970: 10)
let date2 = NSDate(timeIntervalSince1970: 10)
let pointFormatStr5 = NSString(format: "tagged point :%p", date1)
let pointFormatStr6 = NSString(format: "tagged point :%p", date2)

let date3 = NSDate(timeIntervalSinceNow: 10)
let date4 = NSDate(timeIntervalSinceNow: 10)
let pointFormatStr7 = NSString(format: "tagged point :%p", date3)
let pointFormatStr8 = NSString(format: "tagged point :%p", date4)

```

> PS:具体代码可以从[Github](https://github.com/NSCookies)上获取。

> 如有问题或纠正, 可以联系[@叫什么都不如叫Pluto-Y](http://weibo.com/plutoy0504)或在[Github](https://github.com/NSCookies)