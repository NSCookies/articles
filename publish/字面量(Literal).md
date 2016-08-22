所谓的字面量就是其本身就涵盖自己本身的值以及能包含自己的类型的值，比如我们常见的有数字，字符串以及布尔值，就像下面这些:

```
// Objective-C
NSString *aString = @"Hello";
NSNumber *aNumber = @5;
BOOL aBool = YES;
NSArray *arr = @[@"Hello", @"World"];
NSDictionary *dic = @{@"key1": @"value1", @"key2": @"value2"};

// Swift
let aString = "World"
let aNumber = 6
let aBool = false
let arr = ["Hello", "World"]
let dic = ["key1": "value1", "key2": "value2"]
```

字面量在我们工作中几乎是无处不在，然而其实`Objective-C`与`Swift`中这两个的区别还是挺大的。`Objective-C`几乎不可以自定义字面量转换成类型，毕竟`OC`中的字面量转换成类型是在`Clang`中进行支持的。而在`Swift`中是通过一些列的`字面量转换协议`(`Literal Convertible Protocols`)来进行字面量转换成具体类型就好了。所以`Swift`中我们可以通过自定义某一个字面量转换成具体类型。

那么`Objective-C`是如何实现这些，我们一次来说吧，就说说这些常见的字面量。
 > 这里我们通过`clang -rewrite-objc NSCLiteralController.m`来进行看看`Clang`都为我们做了什么
 
我们来看看结果吧:

```
#ifndef __NSCONSTANTSTRINGIMPL
struct __NSConstantStringImpl {
  int *isa;
  int flags;
  char *str;
#if _WIN64
  long long length;
#else
  long length;
#endif
};
#ifdef CF_EXPORT_CONSTANT_STRING
extern "C" __declspec(dllexport) int __CFConstantStringClassReference[];
#else
__OBJC_RW_DLLIMPORT int __CFConstantStringClassReference[];
#endif
#define __NSCONSTANTSTRINGIMPL
 
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_1 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"Hello",5};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_2 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"Hello",5};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_3 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"World",5};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_4 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"key1",4};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_5 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"value1",6};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_6 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"key2",4};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_7 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"value2",6};


// 真实代码通过转换后的c++源码
NSString *aString = (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_1;
NSNumber *aNumber = ((NSNumber *(*)(Class, SEL, int))(void *)objc_msgSend)(objc_getClass("NSNumber"), sel_registerName("numberWithInt:"), 5);
BOOL aBool = ((bool)1);
NSArray *arr = ((NSArray *(*)(Class, SEL, const ObjectType *, NSUInteger))(void *)objc_msgSend)(objc_getClass("NSArray"), sel_registerName("arrayWithObjects:count:"), (const id *)__NSContainer_literal(2U, (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_2, (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_3).arr, 2U);
NSDictionary *dic = ((NSDictionary *(*)(Class, SEL, const ObjectType *, const id *, NSUInteger))(void *)objc_msgSend)(objc_getClass("NSDictionary"), sel_registerName("dictionaryWithObjects:forKeys:count:"), (const id *)__NSContainer_literal(2U, (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_5, (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_7).arr, (const id *)__NSContainer_literal(2U, (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_4, (NSString *)&__NSConstantStringImpl__var_folders_fm_xbb226bx49n28m4sg6sbf_hr0000gn_T_NSCLiteralController_a32604_mi_6).arr, 2U);
```

或许有小伙伴看到上面一堆看不懂的东东直呼，尼玛，太长了，不看。然而请稍安勿躁，难道我还会骗你吗？（我去，上次你说去看那美女洗澡，结果是80岁的老太太。）咳咳，开玩笑。不要急我们一点一点分析，先从具体的运行代码开始分析。
 
我们可以看到字符串的字面量被转换成了`struct __NSConstantStringImpl`的类型,也就是字符串的字面量转换成了一个结构常量，然后通过类型转换成了NSString类型。而`NSNumber`则是通过`[NSNumber numberWithInt:]`的函数将其转换成了`NSNumber`类型,而如果小伙伴试试其他的数字，比如float或者double等数字的话，会发现`clang`会替我们自动选择最适合的构造函数进行转换，不过目前有点纠结的一点就是`long double`类型是没有办法通过`@`来进行转换， 并且如果之前看过[Tagged Pointer](http://www.nscookies.com/tagged-pointer/)一节的小伙伴可以尝试一下，会发现其实这里的`NSNumber`其实是一个`Tagged Pointer`，感兴趣的小伙伴可以自己去看下。

而至于数组和字典则是在调用其自己的构造函数，`[NSArray arrayWithObjects:count:]`和`[NSDictionary dictionaryWithObjects:forKeys:count:]`,然后其中的拿一些一大长串的都是字符串字面量的结构体的常量名称。跟上面说到的字符串的字面量一样。

而这里比较特别的是一个布尔类型，其实OC中的BOOL类型只是通过typedef转换了一下，其实其本质就是一个`signed char`， 而其中的`YES`和`NO`就是通过一些宏定义定义的，具体的代码可以查看`objc/objc.h`中查看，下面我贴出这些代码：

```
/// Type to represent a boolean value.
#if (TARGET_OS_IPHONE && __LP64__)  ||  TARGET_OS_WATCH
#define OBJC_BOOL_IS_BOOL 1
typedef bool BOOL;
#else
#define OBJC_BOOL_IS_CHAR 1
typedef signed char BOOL; 
// BOOL is explicitly signed so @encode(BOOL) == "c" rather than "C" 
// even if -funsigned-char is used.
#endif

#if __has_feature(objc_bool)
#define YES __objc_yes
#define NO  __objc_no
#else
#define YES ((BOOL)1)
#define NO  ((BOOL)0)
#endif
```

当然初次之外，我们还可以用的比较多的是所谓的`Boxed Expressions`,那么具体是什么样子的呢？其实很简单就是'**@**'加上个括号，`@( <expression> )`的形式，其实这种形式非常常见，可以用在字符串，算术表达式，枚举以及struct中。

```
typedef NS_ENUM(NSInteger, NSCLietralEnum) {
    NSCLietralEnum1,
    NSCLietralEnum2,
    NSCLietralEnum3,
    NSCLietralEnum4
};

typedef struct __attribute__((objc_boxable)) _NSCPosition {
    float x, y;
} NSCPosition;

// Boxed Expressions
NSNumber *piOverTwo = @(M_PI / 2);
NSNumber *favoriteColor = @(NSCLietralEnum1);
NSValue *value = @(self.view.frame);
NSCPosition pos;
NSValue *value2 = @(pos);
```
看到上面代码我们可以注意到其实需要一个struct，需要其通过`__attribute__((objc_boxable))`加以修饰，否则就不可以通过`@()`进行字面量的转换。当然还有关于数组和字典中下标的内容这里就不说了，其实也就是通过判断其是读还是写，将其转换成对应的`msg_send()`的方法调用。这里就不说了，有兴趣的小伙伴可以自己做一些试验。

最后说说我们所谓的"自定义下标"，其实所谓的自定义下标就是通过实现两个方法即可。 这里我们用一个字典来实现这两个方法，如果想实现自己的方法的话，可以例如通过数据库去获取信息啊之类的。看下面代码：

```
@interface NSCSubscriptableObject : NSObject

//  Object subscripting
- (id)objectForKeyedSubscript:(id <NSCopying>)key;
- (void)setObject:(id)obj forKeyedSubscript:(id <NSCopying>)key;

@end

@implementation NSCSubscriptableObject {
    NSMutableDictionary     *_dictionary;
}

- (id)init {
    self = [super init];
    if (self) {
        _dictionary = [NSMutableDictionary dictionary];
    }
    return self;
}

- (id)objectForKeyedSubscript:(id <NSCopying>)key {
    return _dictionary[key];
}

- (void)setObject:(id)obj forKeyedSubscript:(id <NSCopying>)key {
    _dictionary[key] = obj;
}

@end

NSCSubscriptableObject *subsObj = [[NSCSubscriptableObject alloc] init];

subsObj[@"string"] = @"Value 1";
subsObj[@"number"] = @2;
subsObj[@"array"] = @[@"Arr1", @"Arr2", @"Arr3"];

NSLog(@"String: %@", subsObj[@"string"]);
NSLog(@"Number: %@", subsObj[@"number"]);
NSLog(@"Array: %@", subsObj[@"array"]);
```

好了好了，关于`Objective C`的字面量内容说的有点多了。主要是连得内容也多，如果有兴趣的小伙伴可以看[Objective-C Literals](http://clang.llvm.org/docs/ObjectiveCLiterals.html)来看更多详细的内容。

具体的代码也可以从[Github](https://github.com/NSCookies)下载，如果对Swift的字面量没兴趣的小伙伴就滚吧(靠，你说什么，找死啊~)。咳咳，说错了。 请各位大神留下来看一眼吧，求求你们了。

其实`Swift`的非常简单，只要完成其指定协议即可。而至于`Swift`中都挺提供了哪些`字面量转换协议`，大家可以通过下面这些来参考一下：

* ArrayLiteralConvertible
* BooleanLiteralConvertible
* DictionaryLiteralConvertible
* ExtendedGraphemeClusterLiteralConvertible
* FloatLiteralConvertible
* NilLiteralConvertible
* IntegerLiteralConvertible
* StringLiteralConvertible
* StringInterpolationConvertible
* UnicodeScalarLiteralConvertible

只要某一个对象实现了具体的转换协议，我们就可以通过对应的字面量进行转换了，下面我们来看一个例子：

```
// Swift 字面量转换协议
class NSCPerson : StringLiteralConvertible {
    let name: String
    
    init(withName name: String) {
        self.name = name
    }
    
    required convenience init(stringLiteral value: String) {
        self.init(withName:value)
    }
    
    required convenience init(unicodeScalarLiteral value: String) {
        self.init(withName:value)
    }
    
    required convenience init(extendedGraphemeClusterLiteral value: String) {
        self.init(withName:value)
    }
    
}

let person: NSCPerson = "NSCookies"
print("person.name:\(person.name)")

// 输出
// person.name:NSCookies
```

是不是非常简单，简单到就是实现一个接口而已。来小伙伴可以给我鼓鼓掌，我会更爱你们哟~（pia~一脱鞋过来）呜呜~，不要这么暴力，年轻人还是斯文一点比较好。

好吧，言归正传。当我们需要自定义字面量转换成具体类型的时候，可以根据上面的协议中挑选需要的字面量协议类型即可，剩下的就交给`Swift`自带的机制吧。

可以看到这个例子中用字符串的字面量来创建了`NSCPerson`对象。我特别喜欢`Swift`这个特性， 因为这个特性可以给我们带来许多的方便，例如我们可以用在创建`NSURL`时通过字符串来创建，我们可以通过一个ID去创建一个对象，创建的同时可以通过本地缓存或者数据缓存来获得等等等，非常有用。小伙伴要好好利用这个特性。

> PS:具体代码可以从[Github](https://github.com/NSCookies)上获取。