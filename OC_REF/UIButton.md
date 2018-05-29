【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# UIButton
控制你自定义代码执行来响应用户交互的控制组件。

## 概述
当你点击一个按钮或者选择一个获取焦点的按钮时，它会执行任何与其绑定的动作。你使用文本标签，图片或者两者来传达按钮的用途。按钮的外观时可配置的，所以你可以给按钮着色，或者格式化标题来匹配应用的设计。你可以用代码或者IB把按钮添加到界面上。
![](/Users/csip/Documents/docs/notes/translate/datas/uibutton_1.png)

当往界面上添加按钮的时候，执行一下步骤：
1. 在创建按钮的时候设置类型。
2. 提供标题字符串或者图片；为按钮设置合适的size。
3. 给按钮连接一个或多个动作。
4. 设置自动布局规则来控制按钮在界面上的大小和位置。
5. 提供可访问性信息和本地化字符串。

## 响应按钮点击
当用户单击按钮的时候，按钮用[Target-Action](https://developer.apple.com/library/content/documentation/General/Conceptual/CocoaEncyclopedia/Target-Action/Target-Action.html)设计模式通知你的应用。给按钮分配一个动作方法并指派哪些事件触发你的方法调用，而不是直接持有触摸事件。在运行时，按钮会处理所有到来的触摸事件并调用你的方法作为响应。

你可以使用[addTarget:action:forControlEvents:](#addTarget)方法或者IB来链接按钮和你的动作方法。动作方法签名采用列举在Listing1中的三种形式之一。选择需要响应按钮点击而提供信息的形式。

> Listing 1

```
- (IBAction)doSomething;
- (IBAction)doSomething:(id)sender;
- (IBAction)doSomething:(id)sender forEvent:(UIEvent*)event;
```

## 配置按钮外观
按钮的类型定义它的基本外观和行为。 你可以使用[buttonWithType:](#buttonWithType)方法或者`storyboard`在创建按钮的时候指定其类型。按钮创建后，你就不能改变它的类型了。最常用的按钮类型是`Custom`和`System`类型，但也可以在适当的时候使用其他类型。

> **Note**
> 如果要为应用中的所有按钮配置外观，需要使用外观代理对象。`UIButton`类实现了[appearance](#)类方法，你可以通过这个方法获取应用中所有按钮的外观代理。

### 按钮状态
按钮有五个定义外观的状态：`default`，`highlighted`，`focused`，`selected`和`disabled`。当你在界面上添加按钮的时候，它处于`default`状态，这意味着按钮可用并且用户没有与其交互。当用户与按钮交互的时候，他的状态会变成其他的值。例如，当用户点击带有标题的按钮时，按钮就会变成`highlighted`状态。

当使用编程或者IB方式配置按钮时，你可以分别为每个状态指定属性。在IB中，使用属性检查器`State Config control`来选择合适的状态并配置其他属性。如果你没有为特定的状态设置属性，`UIButton`类则会为其提供一个合理的默认行为。例如，一个不可用的按钮通常都是暗灰色的并且当点击的时候不会呈现亮色。这个类的其他的属性，例如[adjustsImageWhenHighlighted](#adjustsiwh)和[adjustsImageWhenDisabled](#adjustsiwd)属性，允许你在某些特殊情况下改变默认行为。

### 内容
按钮的内容包含一个你指定的标题字符串或者图片。你指定的内容用来配置由按钮自己管理的[UILabel](#)和[UIImageView](#)对象。你可以使用[titleLabel](#titleLabel)和[imageView](#imageView)属性来获取这些对象，并且可以直接修改他们的值。这个类的方法也提供了配置字符串和图片外观的快捷方法。

通常的，你可以用标题或者图片来配置按钮，并相应地调整其大小。按钮也可以有一个背景图片，它的位于你指定的内容的背后。你可以同时为按钮设置一个标题和图片，它的外观就如图2显示的那样。你可以使用显式的属性来获取按钮的当前内容。
![图2](/Users/csip/Documents/docs/notes/translate/datas/uibutton_2.png)

当设置按钮内容的时候，你必须为每一个状态分别设置标题，图片和外观属性。如果你没有为特定状态自定义外观，按钮会使用与默认状态相关的值，并添加任何合适的定制。例如，在`highlighted`状态，如果没有提供自定义图片，基于图片的按钮会在默认图片的上方画一个亮点。

### 底色
你可以使用[tintColor](#tintColor)为自定义按钮指定底色。这个属性为按钮的图片或者文本设置颜值。如果你没有设置底色，按钮会使用父视图的底色。

### 内凹边界
你可以在自定或系统按钮中使用内凹添加或移除内容周围的空间。你可以分别设置按钮标题（[titleEdgeInsets](#titleEdgeInsets)），图片（[imageEdgeInsets](#imageEdgeInsets)）的内凹值或者标题和图片[contentEdgeInsets](#contentEdgeInsets)同时设置。当应用时，内凹会影响按钮相应的内容矩形，自动布局引擎正是用它来决定按钮的位置的。

对于`info`，`contact`，`disclosure`等按钮就不能调整内凹边界了。

## IB属性
Table 1 列出了在IB中配置按钮的核心属性

|Attribute|Description|
|---------|-----------|
|类型|按钮类型。这个属性决定很多按钮属性的默认设置。这个属性的值无法在运行时改变，但是你可以使用[buttonType](#buttonType)属性获取类型值|
|状态配置|状态选择器。在这个控制器中选择一个值后，对按钮属性的改变就会应用到指定的状态上|
|标题|按钮的标题。你可以用一个简单地字符串或者属性字符串设置按钮的标题|
|（标题字体和属性）|应用到按钮标题上的字体和其它属性。具体的配置选项取决于你为按钮设置的是一个简单字符串还是一个属性字符串。如果是简单字符串，那么你就可以自定义字体，文本颜色和阴影颜色。对于属性字符串，你可以指定对齐方式，文本方向，刻痕，连字符和很多其他选项。|
|图片|按钮的前景图片。通常的，你会给按钮前景设置一个模板化图片，但是你可以在Xcode工程中设置任何图片。|
|背景|按钮背景图片。背景图片在标题和前景图片的底部。|

Table 2 列出了影响按钮外观的属性。

|Attribute|Description|
|----|----|
|阴影偏移|按钮阴影的偏移和行为。阴影只影响标题字符串。当按钮状态改变为高亮或者从高亮改变为其他状态时，启用高亮反转选项来改变阴影高亮显示。|
|绘画|按钮的绘制行为。|
|换行|按钮文本的换行选项。使用这个属性去定义如何改变按钮标题来适应可用空间。|

Table 3 列出了边缘内凹属性。使用按钮边缘内凹来改变按钮内容的矩形。

|Attribute|Description|
|---|---|
|边界|要配置的边缘内凹。你可以分别设置按钮所有内容的边缘内凹，标题和图片。|
|内凹|内凹值。正值会缩小对应的边界，让它更靠向中心。负值会扩张边界，让它更远离中心。在运行时可以使用[contentEdgeInsets](#contentEdgeInsets)，[titleEdgeInsets](#titleEdgeInsets)和[imageEdgeInsets](#imageEdgeInsets)属性获取这些值。|

关于按钮继承过来的IB属性，请看[UIControl](#)和[UIView](#)。

## 国际化
为了国际化一个按钮，需要为按钮标题文本设置一个本地化字符串。（你也许会适当的本地化一个按钮图片。）

当使用`storyboard`构建界面的时候，使用基于`Xcode`的国际化功能去配置你项目支持的本地化。当你添加本地化时，Xcode会创建一个本地化的字符串文件。当编程配置你的界面的时候，可以使用系统的内置支持来加载本地化字符串和资源。关于界面国际化的更多信息，请看[国际化和本地化指南](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/Introduction/Introduction.html#//apple_ref/doc/uid/10000171i)。

## 可访问性
默认按钮是可访问的。对于按钮默认的可访问特性是启用按钮和用户交互。

当在设备上启用VoiceOver时，可访问性签，特征和提示会回传给用户。按钮标题会重写他的可用性标签；甚至如果你给标签设置一个自定义值，VoiceOver也会说出标题的值。一旦用户点击按钮，VoiceOver会说出它的信息。例如，当用户点击相机的一个可选按钮，VoiceOver会这样说：

* “Options. Button. Shows additional camera options.”

更多关于iOS控制访问的信息，请看在[UIControl](#)中的可访问性信息。关于让你的界面可访问的一般信息，请看[iOS可访问性编程指南](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/iPhoneAccessibility/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008785)。


# 话题
## 创建按钮
### <span id="buttonWithType:">+ buttonWithType:</span>
`方法`创建并返回一个指定了类型的新按钮。

#### 声明
```
+ (instancetype)buttonWithType:(UIButtonType)buttonType;
```

#### 参数
**buttonType**：按钮类型。查看可能的值请看[UIButtonType](#UIButtonType)。

#### 返回值
新创建的按钮。

#### 讨论
这个方法是带有特殊配置创建按钮对象的便利构造器。如果你子类化`UIButton`，这个方法返回的实力不是你的子类。如果你想创建一个特定的子类，你需要直接调用 `alloc/init`方法来创建按钮。

当创建一个自定义按钮——按钮的类型是[UIButtonTypeCustom](#UIButtonTypeCustom)的时候，按钮的初始frame是(0, 0, 0, 0)。在把按钮添加到界面上之前，你需要把frame更新为适当的值。

### <span id="UIButtonType">UIButtonType</span>
`枚举`指定按钮的样式。

#### 声明
iOS
```
typedef enum UIButtonType : NSInteger {
    UIButtonTypeCustom = 0,
    UIButtonTypeSystem,
    UIButtonTypeDetailDisclosure,
    UIButtonTypeInfoLight,
    UIButtonTypeInfoDark,
    UIButtonTypeContactAdd,
    UIButtonTypeRoundedRect = UIButtonTypeSystem
} UIButtonType;
```
tvOS
```
typedef enum UIButtonType : NSInteger {
    UIButtonTypeCustom = 0,
    UIButtonTypeSystem,
    UIButtonTypeDetailDisclosure,
    UIButtonTypeInfoLight,
    UIButtonTypeInfoDark,
    UIButtonTypeContactAdd,
    UIButtonTypePlain,
    UIButtonTypeRoundedRect = UIButtonTypeSystem
} UIButtonType;
```

#### 常量
**UIButtonTypeCustom：**没有按钮样式
**UIButtonTypeSystem：**系统风格按钮，同显示在导航栏和工具栏上的按钮一样。
**UIButtonTypeDetailDisclosure：**详情按钮。
**UIButtonTypeInfoLight：**高亮背景的信息按钮。
**UIButtonTypeInfoDark：**深色背景的信息按钮。
**UIButtonTypeContactAdd：**联系方式添加按钮。
**UIButtonTypePlain：**没有模糊背景视图的标准系统按钮。
**UIButtonTypeRoundedRect：**圆角矩形按钮。

## 配置按钮标题
### <span id="titleLabel">titleLabel</span>
`属性`为按钮展示`currentTitle`属性值的的视图
#### 声明
```
@property(nonatomic, readonly, strong) UILabel *titleLabel;
```
#### 讨论
尽管这个实现是只读的，但是它的属性是可读写的。使用这些属性初步的配置按钮文本。例如：
```
UIButton *button = [UIButton buttonWithType: UIButtonTypeSystem];
button.titleLabel.font = [UIFont systemFontOfSize:12];
button.titleLabel.lineBreakMode = NSLineBreakTruncatingTail;
```
不用使用标签对象设置文本颜色和阴影颜色。而是，使用这个类的[setTitleColor:forState](#stcfs)和[setTitleShadowColor:forState:](#stscfs)方法来设置这些值。使用[setTitle:forState](#stfs)设置标签的文本（`button.titleLabel.text`无法设置文本）。

甚至如果按钮还没有显示出来，`titleLabel`属性也会返回一个值。对于系统按钮这个属性的值是`nil`。

---

### <span id="tfs">- titleForState:</span>
返回与指定状态相关的标题。

#### 声明
```
- (NSString *)titleForState:(UIControlState)state;
```
#### 参数
*state：*使用标题的状态。可用的状态值在[UIControlState](#)中声明。
#### 返回值
特定状态的标题。如果指定状态没有标题，这个方法会返回一个与[UIControlStateNormal](#)状态相关的标题。

---

### <span id="stcfs">- setTitle:forState:</span>
为特定状态设置标题
#### 声明
```
- (void)setTitle:(NSString *)title forState:(UIControlState)state;
```
#### 参数
*title：*特定状态使用的标题
*state：*特定标题使用的状态，参看[UIControlState](#)

---

### <span id="atttfs">- attributedTitleForState:</span>
返回与指定状态相关的样式化标题。

---

### <span id="satfs">- setAttributedTitle:forState:</span>
为指定状态设置样式化标题。

---

### <span id="tcfs">- titleColorForState:</span>
返回特定状态使用的标题颜色。

---

### - setTitleColor:forState:
为特定状态设置标题颜色。

---

### - titleShadowColorForState:
返回特定状态的阴影颜色。

---

### <span id="stscfs">- setTitleShadowColor:forState:</span>
为特定状态设置阴影颜色。

---

### reversesTitleShadowWhenHighlighted
布尔值，当按钮高亮时，确定是否改变标题阴影。

---

## 配置按钮展示
###<span id="adjustsiwh">adjustsImageWhenHighlighted</span>
布尔值，确定当按钮高亮时是否改变图片。

###<span id="adjustsiwd">adjustsImageWhenDisabled</span>
布尔值，确定当按钮不可用时是否改变图片。

### showsTouchWhenHighlighted
布尔值，确定点击按钮时是否发光。

### - backgroundImageForState:
返回特定状态的背景图片。

### - imageForState:
返回特定状态的图片。

### - setBackgroundImage:forState:
为特定状态设置背景图片。

### - setImage:forState:
为特定状体设置图片。

### tintColor
应用到按钮标题和图片上的底色。

## 配置边缘内凹
###<span id="titleEdgeInsets">titleEdgeInsets</span>
设置标题边框。
###<span id="imageEdgeInsets">imageEdgeInsets</span>
设置图片边框。
###<span id="contentEdgeInsets">contentEdgeInsets</span>
设置按钮内容边框。

## 获取当前状态
### <span id="buttonWithType">buttonWithType</span>
按钮类型。

### currentTitle
当前标题。

### currentAttributedTitle
获取当前样式化标题。

### currentTitleColor
当前标签颜色。

### currentTitleShadowColor
当前标题阴影颜色。
### currentImage
按钮展示的当前图片。
### currentBackgroundImage
按钮展示的当前背景图片。
### imageView
按钮的图片视图。

## 获取尺寸
### - backgroundRectForBounds:
返回接收者绘制背景的矩形。
### - contentRectForBounds:
返回接收者绘制整个内容的矩形。
### - titleRectForContentRect:
返回接收者绘制标题的矩形。
### - imageRectForContentRect:
返回接收者绘制图片的矩形。

---
### <span id="addTarget">addTarget:action:forControlEvents:</span>
将目标对象和动作方法与空间关联起来。
#### 声明
```
- (void)addTarget:(id)target action:(SEL)action forControlEvents:(UIControlEvents)controlEvents;
```
#### 参数
*target：*目标对象，即调用动作方法的对象。如果你设置为nil，`UIKit`会为对象在响应链上查找响应指定动作的消息，并把该消息传递给这个对象。
*action：*称作动作方法的选择器标识符。你可以指定一个匹配`UIControl`中任何方法签名的选择器。这个参数必须非空。
*controlEvents：*一个用于标识调用动作方法的特定控制事件的位掩码。通常指定至少一个常量。常量列表，请看[UIControlEvents](#)

