所谓的链式编程呢，就是像链子一样一环扣一环，在编程中的体现呢，就是一个方法接着一个方法的调用。如果你还不懂那就想想[Masonry](https://github.com/SnapKit/Masonry)，`make.width.greaterThanOrEqualTo(@200);`类似这种代码。这样就懂了吧，我的小乖乖~

记得以前学Java的时候最喜欢这种不断点方法调用了，以前一直在想如果在OC中要实现链式编程的话会是怎么样子的呢？嗯~大概应该是这样的吧:

```
// str是一个NSString
[[str substringFromIndex:10] substringToIndex:12];
```

想到这个我真的是日了🐶了，要让我不断用空格来调用，OMG，算了吧。知道后来Block的出现，我看到了曙光，因为我看到了Block竟然可以用`()`来调用Block，终于又看到了我可爱的`()`了。

但是接下来我还是一步一步的来看看如何做。那么我们假设我们现在要在OC中实现链式编程，那么最直接的做法就是在方法调用后直接返回对应的对象即可，那么假设我们有一个`NSCStudent`对象，他有`study`和`play`的方法，如果按照OC的方法来进行链式的话，应该就是这样`[[student study] play]`, 那么`NSCStudent`的方法实现大概应该是这样：

```
@implementation NSCStudent

- (instancetype)study {
    NSLog(@"The student study");
    return self;
}

- (instancetype)play {
    NSLog(@"The student play");
    return self;
}

@end
```

这样我们就可以用`[[student study] play]`来调用了。（路人甲：“不对啊，我试过，如果是这样实现也可以用student.study.play来调用啊”）保安，保安，有人来打乱，拖出去乱棍打死~。 好吧，我承认如果是这种实现可以，那么如果带参数的情况下呢，那你是不是日了🐶了。

其实这种方式也没有什么，就是'['和']'特别多，有的时候都分不清楚哪一层是哪一层了，这个是我我就怀念起了Java里面通过'.'来调用方法了。那么接下来我们想想如何像`Mansory`通过'.'和'()'来实现一些带参数的链式编程呢？

其实上面也提到了，既然Block可以通过'()'来调用，并且我们也能通过'.'来调用不带参数的方法，那么我们通过写一个不带参数的方法，并且返回Block，且把原来的参数放到Block中即可。我们把上面的`NSCStudent`中添加一个方法`playWith`并且返回Block，那么代码应该如下：

```
- (NSCStudent * (^)(NSString *))playWith {
    return ^id(NSString *name) {
        NSLog(@"The student play with %@", name);
        return self;
    };
}
```

接下来我们来分析一下如果我写了`student.study.playWith`的话会出现什么，当然首先肯定是`study`还是跟原来一样，那么调用`playWith`之后将返回一个Block，并且Block需要一个`NSString`的参数。那么按照之前的理论就是可以通过'()'来调用Block的话，我们可以这样写`student.study.playWith(@"Girl")`，并且从代码中我们可以看得出来调用Block的结果是返回了一个`NSCStudent`的Block。那么酱紫的话，劳资是不是有可以继续点来调用`NSCStudent`的方法了。

那么我就可以`student.study.playWith(@"Girl1").playWith(@"Girl2").playWith(@"Girl3").playWith(@"Girl4").playWith(@"Girl5")....`，我去，那么不是我可以群*了，咳咳，说错了，是我就可以无限调用函数了，其实就是这么简单。

那么如果在Swift中要实现链式编程的话，就更特么简单了。毕竟Swift就是通过'.'和'()'来调用的，只要在方法后返回自身就好了。那么我们来稍微看下代码吧:

```
struct NSCStudent {
    func study() -> NSCStudent {
        print("The student study")
        return self
    }
    
    func playWith(name: String) -> NSCStudent {
        print("The student play with \(name)")
        return self
    }
}

let student = NSCStudent()
student.study().playWith("Girl")
```

好了，就酱紫吧，各位小老婆们伺候我就寝吧~

> PS:具体代码可以从[Github](https://github.com/NSCookies)上获取。

> 如有问题或纠正, 可以联系[@叫什么都不如叫Pluto-Y](http://weibo.com/plutoy0504)或在[Github](https://github.com/NSCookies)