# Code Review文档

### What
```
代码评审是指在软件开发过程中，对源代码的系统性检查。
```

### Why
* 提高质量
* 及早发现潜在缺陷与BUG，降低事故成本。
* 促进团队内部知识共享，提高团队整体水平
* 评审过程对于评审人员来说，也是一种思路重构的过程。帮助更多的人理解系统。

### Who/When/Where
```
Every Commit
```

### How
```
通过Phabricator软件，参照下面的review条款，进行review 
```

[!Phabricator客户端安装](https://www.jianshu.com/p/b347c44749ca)

``` 
代码必须通过其他人的review才能提交
```


### 条款
### 1.代码长度
#### 1.1 单一文件建议不超过1000行（不强制 建立Extension 或者 更改架构） 
#### 1.2 单一方法不得超过60行 (强制 拆分成多个方法获得拆分出某些公共方法或者多个类，降低耦合)
#### 1.3 单列不得超过140列 （更改参数对齐方式）


### 2.书写规范

```
原则上，尽量符合苹果系统库的书写格式
```

#### 2.1 变量声明，应该让*号放在变量名的前面

```
UIImage *image = nil;
```

#### 2.2 属性声明，关键字的顺序应该是可空性，原子性，内存管理，读写性

```
@property (nullable, nonatomic, strong, readonly) UIImage *image; 
```

**注意**

```
对于内存关键字基本类型用assign，指针类型的选择weak或strong，
不可变容器类用copy（防止多态指向可变类型，造成数值被改变<按需求
>）, block用copy（防止在栈中被释放引发的野指针问题）
```

#### 2.3 在适当的时候添加空格

```
@property (nullable, nonatomic, copy) NSArray<UIImage *> *animationImages; 
- (instancetype)initWithImage:(nullable UIImage *)image;
```

#### 2.4 对于复杂的业务逻辑添加必要的注释，模型的属性声明必须添加注释，枚举必须添加注释，公共方法（Utils,Helper）必须添加注释

#### 2.5 驼峰命名，类名首字母大写，加必要前缀避免命名冲突，变量名首字母小写，宏变量全大写，单词间用下划线连接，全局变量以k开头(不强制，但要能够区分出来)，属于某个类的全局变量以类名前三个字母做前缀，避免起过于简单，无意义的变量名

```
@interface WYStudent: NSObject
@end

WYStudent *student;

static NSString* const kSomeGlobalVariable = @"abc";

#define SCREEN_HEIGHT 320.0f

const NSTimeInterval WYViewClassAnimationDuration = 0.3; 
```

#### 2.6 访问属性尽量使用点，不使用调用
* 推荐做法

```
if (someVaribale.someProperty > 5) {
}

someVaribale.someProperty = 6
```

* 不推荐做法

```
if ([someVaribale someProperty] > 5) {
}

[someVaribale setSomeProperty: 6];
```

#### 2.7 容器类加上必要的泛型约束
* 推荐做法

```
NSArray<UIImage *> *animationImages;
NSDictionary<NSString *, UIImage *> *imageDicts;
```

* 不推荐做法

```
NSArray *animationImages;
NSDictionary *imageDicts;
```

#### 2.8 尽量使用NonNull，Nullable，instancetype等关键字对参数和返回值进行约束，原则上让编译器对风险代码做出提示

#### 2.9 尽量使用@selector(), 不使用NSSelectorFromString()

### 2.10 三元运算符不可嵌套，第二个参数必要时省略
* 不推荐做法

```
result = a > b ? x = c > d ? c : d : y;

value = object ? object : [self createObject]; 
```

* 建议写法

```
res1 = (x = c > d)? c : d;
res2 = a > b? res1: y;

value = object ? : [self createObject];
```

### 2.11 switch中，一行的case不加括号，多余一行加括号，枚举的switch，省略default, 不需要fall-through的要加break

```
switch (condition) { 
    case 1: 
        // ... 
        break; 
    case 2: { 
        // ... 
        // Multi-line example using braces 
        break; 
       } 
    case 3: 
        // ... 
        break; 
    default:  
        // ... 
        break; 
} 
```

### 2.12 使用NS_ENUM定义枚举，枚举量命名为枚举类型名+枚举量的名字
```
typedef NS_ENUM(NSUInteger, ZOCMachineState) { 
    ZOCMachineStateNone, 
    ZOCMachineStateIdle, 
    ZOCMachineStateRunning, 
    ZOCMachineStatePaused 
}; 
```

### 2.13 初始化尽量使用字面量
* 不推荐做法

```
NSArray *names = [NSArray arrayWithObjects:@"Brian", 
@"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
 
NSDictionary *productManagers = [NSDictionary 
dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", 
@"Kamal", @"iPad", @"Bill", @"Mobile Web", nil]; 


NSNumber *shouldUseLiterals = [NSNumber 
numberWithBool:YES]; 
```

* 推荐做法

```
NSArray *names = @[@"Brian", @"Matt", @"Chris", 
@"Alex", @"Steve", @"Paul"]; 

NSDictionary *productManagers = @{@"iPhone" : @"Kate", 
@"iPad" : @"Kamal", @"Mobile Web" : @"Bill"}; 

NSNumber *shouldUseLiterals = @YES; 
```

### 2.14 对于内存占用过大的对象使用懒加载

### 2.15 花括号的使用

* 不推荐写法

```
if (cond) 
{

}
```

* 推荐写法

```
if (cond) {

}
```

### 2.16 使用必要的#pragma mark对代码进行分区

### 3.参数校验（不能因为参数导致程序crash）
* 错误写法

``` 
- (void) addSomeItem: (id) obj {
	[someList addObject: obj]; //造成Crash
}
```

* 正确写法

```
- (void) addSomeItem: (id) item {
	if (!item) {
		return
	}
	[someList addObject: obj];
}
```

**注意**

```
类似的情况，还包括数组越界的判断，如：
if (idx < arr.count) {
	id someValue = arr[idx];
}

建议用统一的安全取下标方法，防止crash
```

### 4.UIView不使用Tag
* 错误写法

```
UIView *someView = [[UIView alloc] init];    
[someView setTag:1001];


// 使用view
UIView *view = [superView viewWithTag:1001];
[view removeFromSuperView];
```

* 正确写法

```
UIView *someView = [[UIView alloc] init]; 
somePointer = someView;


// 使用view
[somePointer removeFromSuperView];
```

### 5.禁止尤达表达式
```
尤达表达式是指，拿一个常量去和变量比较而不是拿变量去和常量比较。  
```

* 推荐做法

```
if ([someValue isEqual:@42]) { 
}
```

* 不推荐做法

```
if ([@42 isEqual:someValue]) { 
}
```

**注意**  

```
使用尤达表达式可以避免以下错误，如：
//该写法可以编译通过，但结果可能并不是你想要的
if (somePointer = nil) {  
}  
//该写法不会编译通过,但影响了可读性  
if (nil = somePointer) {  
}

对于这种情况建议使用如下格式：
if (somePointer) {
}

if (!somePointer) {
}
```

### 6.不使用Hard Coding和Magic Number，使用枚举
* 错误写法

```
if (someValue == @"stu") {
}
```

* 正确写法

```
static NSString * const kStuType = @"stu";
if someValue == kStuType {
}
```

**注意**

```
类似的情况还包括，NSNotification需要的key，一些类型特殊值，
一些经常变动的文案，原则上改动常量，应该不影响业务逻辑，两部
分代码应该解耦
```

### 7.block用法

```
对于self持有的block，block方法块中又引用了self的代码，进行相
应的处理
```

* 错误写法

```
self.someBlock = ^{
	NSLog(@"%@", self); 
};
```

* 正确写法

```
__weak typeof(self) weakSelf = self; 
self.completionHandler = ^{ 
    NSLog(@"%@", weakSelf); 
}; 

多行

__weak typeof(self) weakSelf = self; 
self.completionHandler = ^{ 
	 __weak typeof(self) strongSelf = self;
    NSLog(@"%@", strongSelf); 
    [strongSelf someAction];
}; 

```

### 8.注意repeat模式的timer可能造成循环引用

```
很多情况，timer的target为self，而timer被self持有
这时候会造成self无法释放，要确保页面不显示之前，将timer 
停止，并且置为nil
```

### 9.注意通知未移除可能导致的野指针错误
### 10.注意delegate内存管理应该采用weak，避免循环引用
### 11.注意其他的循环引用情况，比如cell strong table等
### 12.避免出现重复代码

* 不推荐写法

```
if (cond) {
	someValue = cond
	someValue2 = cond
	someLogic1
	someLogic2
}else{
	someValue = !cond
	someValue2 = !cond
	someLogic1
	someLogic2
}
```

* 推荐写法

```

someValue = cond
someValue2 = cond
someLogic1
someLogic2   如果过长建议封装一个新的方法
```
