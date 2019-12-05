---
title: 【译】Fucking SwiftUI
date: 2019-07-18 02:31:57
tags: SwiftUI
categories: iOS
---

> 原文：[Fucking Swift UI - Cheat Sheet](https://fuckingswiftui.com/) 
> 译者的话：翻译过程中，发现了原文中的几个错误，我向作者[@sarunw](https://twitter.com/sarunw)提出意见后，直接在译文中改掉了，如果您发现文中内容有误，欢迎与我联系。

<!--more-->

关于 SwiftUI，您在下文中看到的所有答案并不是完整详细的，它只能充当一份备忘单，或是检索表。

## 常见问题

关于 SwiftUI 的常见问题：

**是否需要学 SwiftUI？**

是

**是否有必要现在就学 SwiftUI？**

看情况，因为 SwiftUI 目前只能在 iOS 13、macOS 10.15、tvOS 13和 watchOS 6 上运行。如果您要开发的新应用计划仅针对前面提到的 OS 系统，我会说是。 但是，如果您打算找工作或是无法确保会在此 OS 版本的客户端项目上工作，则可能要等一两年，再考虑迁移成 SwiftUI，毕竟大多数客户端工作都希望支持尽可能多的用户，这意味着您的应用必须兼容多个 OS 系统。 因此，一年后再去体验优雅的 SwiftUI 也许是最好的时机。

**是否需要学 UIKit/AppKit/WatchKit？**

是的，就长时间来看，UIKit 仍将是 iOS 架构的重要组成部分。现在的 SwiftUI 并不成熟完善，我认为即使您打算用 SwiftUI 来开发，仍然不时需要用到 UIKit。

**SwiftUI 能代替 UIKit/AppKit/WatchKit 吗?**

现在不行，但将来也许会。SwiftUI 虽然是刚刚推出的，它看起来已经很不错。我希望两者能长期共存，SwiftUI 还很年轻，它还需要几年的打磨成长才能去代替 UIKit/AppKit/WatchKit。

**如果我现在只能学习一种，那么应该选择 UIKit/AppKit/WatchKit 还是 SwiftUI？**

UIKit。 您始终可以依赖 UIKit，它用起来一直不错，且未来一段时间仍然可用。如果您直接从 SwiftUI 开始学习，可能会遗漏了解一些功能。

**SwiftUI 的控制器在哪里？**

没有了。 如今页面间直接通过响应式编程框架 Combine 交互。Combine 也作为新的通信方式替代了 UIViewController。

## 要求

- Xcode 11 Beta（[从 Apple 官网下载](https://developer.apple.com/download/)）
- iOS 13 / macOS 10.15 / tvOS 13 / watchOS 6
- macOS Catalina，以便在画布上呈现 SwiftUI（[从 Apple 官网下载](https://developer.apple.com/download/)）

> **想要体验 SwiftUI 画布，但不想在您的电脑上安装 macOS Catalina beta 系统**
您可以与当前的 macOS 版本并行安装 Catalina。这里介绍了[如何在单独的 APFS 卷上安装 macOS](https://support.apple.com/en-us/HT208891)

## SwiftUI 中等效的 UIKit

### 视图控制器

| UIKit                      |              SwiftUI              |                             备注                             |
| :------------------------- | :-------------------------------: | :----------------------------------------------------------: |
| UIViewController           |               View                |                              -                               |
| UITableViewController      |           [List](#list)           |                              -                               |
| UICollectionViewController |                 -                 | 目前，还没有 SwiftUI 的替代品，但是您可以像[Composing Complex Interfaces's tutorial](https://developer.apple.com/tutorials/swiftui/composing-complex-interfaces)里那样，使用 List 的组成来模拟布局 |
| UISplitViewController      | [NavigationView](#navigationview) |             Beta 5中有部分支持，但仍然无法使用。             |
| UINavigationController     | [NavigationView](#navigationview) |                              -                               |
| UIPageViewController       |                 -                 |                              -                               |
| UITabBarController         |        [TabView](#tabview)        |                              -                               |
| UISearchController         |                 -                 |                              -                               |
| UIImagePickerController    |                 -                 |                              -                               |
| UIVideoEditorController    |                 -                 |                              -                               |
| UIActivityViewController   |                 -                 |                              -                               |
| UIAlertController          |          [Alert](#alert)          |                              -                               |

### 视图和控件

| UIKit                   |              SwiftUI              |                             备注                             |
| :---------------------- | :-------------------------------: | :----------------------------------------------------------: |
| UILabel                 |           [Text](#text)           |                              -                               |
| UITabBar                |        [TabView](#tabview)        |                              -                               |
| UITabBarItem            |        [TabView](#tabview)        |             [TabView ](#tabview) 里的 `.tabItem`             |
| UITextField             |      [TextField](#textfield)      |             Beta 5中有部分支持，但仍然无法使用。             |
| UITableView             |           [List](#list)           |          [VStack](#vstack) 和 [Form](#form) 也可以           |
| UINavigationBar         | [NavigationView](#navigationview) |          [NavigationView](#navigationview) 的一部分          |
| UIBarButtonItem         | [NavigationView](#navigationview) | [NavigationView](#navigationview) 里的 `.navigationBarItems` |
| UICollectionView        |                 -                 |                              -                               |
| UIStackView             |         [HStack](#hstack)         |                   ` .axis == .Horizontal `                   |
| UIStackView             |         [VStack](#vstack)         |                    `.axis == .Vertical `                     |
| UIScrollView            |     [ScrollView](#scrollview)     |                              -                               |
| UIActivityIndicatorView |                 -                 |                              -                               |
| UIImageView             |          [Image](#image)          |                              -                               |
| UIPickerView            |         [Picker](#picker)         |                              -                               |
| UIButton                |         [Button](#button)         |                              -                               |
| UIDatePicker            |     [DatePicker](#datepicker)     |                              -                               |
| UIPageControl           |                 -                 |                              -                               |
| UISegmentedControl      |         [Picker](#picker)         |    [Picker](#picker) 中的一种样式 `SegmentedPickerStyle`     |
| UISlider                |         [Slider](#slider)         |                              -                               |
| UIStepper               |        [Stepper](#stepper)        |                              -                               |
| UISwitch                |         [Toggle](#toggle)         |                              -                               |
| UIToolBar               |                 -                 |                              -                               |

### 框架集成 - SwiftUI 中的 UIKit

将 SwiftUI 视图集成到现有应用程序中，并将 UIKit 视图和控制器嵌入 SwiftUI 视图层次结构中。

| UIKit            | SwiftUI                                                      | 备注 |
| :--------------- | :----------------------------------------------------------- | :--: |
| UIView           | [UIViewRepresentable](#uiviewrepresentable)                  |  -   |
| UIViewController | [UIViewControllerRepresentable](#uiviewcontrollerrepresentable) |  -   |

### 框架集成 - UIKit 中的 SwiftUI

将 SwiftUI 视图集成到现有应用程序中，并将 UIKit 视图和控制器嵌入 SwiftUI 视图层次结构中。

| UIKit                                                        | SwiftUI |                             备注                             |
| :----------------------------------------------------------- | :------ | :----------------------------------------------------------: |
| UIView ([UIHostingController](#uihostingcontroller))         | View    | 没有直接转换为 UIView 的方法，但是您可以使用容器视图将 UIViewController 中的视图添加到视图层次结构中 |
| UIViewController ([UIHostingController](#uihostingcontroller)) | View    |                              -                               |

## SwiftUI - 视图和控件

### <span id="text">Text</span>

显示一行或多行只读文本的视图。

```swift
Text("Hello World")
```

样式:

```swift
Text("Hello World")
  .bold()
  .italic()
  .underline()
  .lineLimit(2)
```

`Text` 中填入的字符串也用作 `LocalizedStringKey`，因此也会直接获得 `NSLocalizedString` 的特性。

```swift
Text("This text used as localized key")
```

直接在文本视图里格式化文本。 实际上，这不是 SwiftUI 的功能，而是 Swift 5的字符串插入特性。

```swift
static let dateFormatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateStyle = .long
    return formatter
}()

var now = Date()
var body: some View {
    Text("What time is it?: \(now, formatter: Self.dateFormatter)")
}
```

可以直接用 `+` 拼接 `Text` 文本:

```swift
Text("Hello ") + Text("World!").bold()
```

文字对齐方式：

```swift
Text("Hello\nWorld!").multilineTextAlignment(.center)
```

[文档](https://developer.apple.com/documentation/swiftui/text)

### <span id="textfield">TextField</span>

显示可编辑文本界面的控件。

```swift
@State var name: String = "John"    
var body: some View {
    TextField("Name's placeholder", text: $name)
        .textFieldStyle(RoundedBorderTextFieldStyle())
        .padding()
}
```

[文档](https://developer.apple.com/documentation/swiftui/textfield)

### <span id="securefield">SecureField</span>

用户安全地输入私人文本的控件。

```swift
@State var password: String = "1234"    
var body: some View {
    SecureField($password)
        .textFieldStyle(RoundedBorderTextFieldStyle())
        .padding()
}
```

[文档](https://developer.apple.com/documentation/swiftui/securefield)

### <span id="image">Image</span>

显示图像的视图。

```swift
Image("foo") //图像名字为 foo
```

我们可以使用新的 SF Symbols：

```swift
Image(systemName: "clock.fill")
```

您可以通过为系统图标添加样式，来匹配您使用的字体：

```swift
Image(systemName: "cloud.heavyrain.fill")
    .foregroundColor(.red)
    .font(.title)
Image(systemName: "clock")
    .foregroundColor(.red)
    .font(Font.system(.largeTitle).bold())
```

为图片增加样式：

```swift
Image("foo")
    .resizable() // 调整大小，以便填充所有可用空间
    .aspectRatio(contentMode: .fit)
```

[文档](https://developer.apple.com/documentation/swiftui/image)

### <span id="button">Button</span>

在触发时执行操作的控件。

```swift
Button(
    action: {
        // 点击事件
    },
    label: { Text("Click Me") }
)
```

如果按钮的标签只有 `Text`，则可以通过下面这种简单的方式进行初始化：

```swift
Button("Click Me") {
    // 点击事件
}
```

您可以像这样给按钮添加属性：

```swift
Button(action: {
                
}, label: {
    Image(systemName: "clock")
    Text("Click Me")
    Text("Subtitle")
})
.foregroundColor(Color.white)
.padding()
.background(Color.blue)
.cornerRadius(5)
```

[文档](https://developer.apple.com/documentation/swiftui/button)

### <span id="navigationlink">NavigationLink</span>

按下时会触发导航演示的按钮。它用作代替 `pushViewController`。

```swift
NavigationView {
    NavigationLink(destination:
        Text("Detail")
        .navigationBarTitle(Text("Detail"))
    ) {
        Text("Push")
    }.navigationBarTitle(Text("Master"))
}
```

为了增强可读性，可以把 `destination` 包装成自定义视图 `DetailView ` 的方式：

```swift
NavigationView {
    NavigationLink(destination: DetailView()) {
        Text("Push")
    }.navigationBarTitle(Text("Master"))
}
```

> 但不确定是 Bug 还是设计使然，上述代码 在 Beta 5 中的无法正常执行。尝试像这样把 `NavigationLink ` 包装进列表中试一下：
>
> ```swift
NavigationView {
    List {
        NavigationLink(destination: Text("Detail")) {
            Text("Push")
        }.navigationBarTitle(Text("Master"))
    }
}
```

如果 `NavigationLink` 的标签只有 `Text` ，则可以用这样更简单的方式初始化：

```swift
NavigationLink("Detail", destination: Text("Detail").navigationBarTitle(Text("Detail")))
```

[文档](https://developer.apple.com/documentation/swiftui/navigationlink)

### <span id="toggle">Toggle</span>

在开/关状态之间切换的控件。

```swift
@State var isShowing = true // toggle 状态值

Toggle(isOn: $isShowing) {
    Text("Hello World")
}
```

如果 `Toggle` 的标签只有 `Text`，则可以用这样更简单的方式初始化：

```swift
Toggle("Hello World", isOn: $isShowing)
```

[文档](https://developer.apple.com/documentation/swiftui/toggle)

### <span id="picker">Picker</span>

从一组互斥值中进行选择的控件。

选择器样式根据其被父视图进行更改，在表单或列表下作为一个列表行显示，点击可以推出新界面展示所有的选项卡。

```swift
NavigationView {
    Form {
        Section {
            Picker(selection: $selection, label:
                Text("Picker Name")
                , content: {
                    Text("Value 1").tag(0)
                    Text("Value 2").tag(1)
                    Text("Value 3").tag(2)
                    Text("Value 4").tag(3)
            })
        }
    }
}
```

您可以使用 `.pickerStyle(WheelPickerStyle())`覆盖样式。

在 iOS 13 中， `UISegmentedControl` 也只是 `Picker` 的一种样式。

```swift
@State var mapChoioce = 0
var settings = ["Map", "Transit", "Satellite"]
Picker("Options", selection: $mapChoioce) {
    ForEach(0 ..< settings.count) { index in
        Text(self.settings[index])
            .tag(index)
    }

}.pickerStyle(SegmentedPickerStyle())
```

> 分段控制器在iOS 13中也焕然一新了。

[文档](https://developer.apple.com/documentation/swiftui/picker)

### <span id="datepicker">DatePicker</span>

选择日期的控件。

日期选择器样式也会根据其父视图进行更改，在表单或列表下作为一个列表行显示，点击可以扩展到日期选择器（就像日历 App 一样）。

```swift
@State var selectedDate = Date()

var dateClosedRange: ClosedRange<Date> {
    let min = Calendar.current.date(byAdding: .day, value: -1, to: Date())!
    let max = Calendar.current.date(byAdding: .day, value: 1, to: Date())!
    return min...max
}

NavigationView {
    Form {
        Section {
            DatePicker(
                selection: $selectedDate,
                in: dateClosedRange,
                displayedComponents: .date,
                label: { Text("Due Date") }
            )
        }
    }
}
```

不在表单或列表里，它就可以作为普通的旋转选择器。

```swift
@State var selectedDate = Date()

var dateClosedRange: ClosedRange<Date> {
    let min = Calendar.current.date(byAdding: .day, value: -1, to: Date())!
    let max = Calendar.current.date(byAdding: .day, value: 1, to: Date())!
    return min...max
}

DatePicker(
    selection: $selectedDate,
    in: dateClosedRange,
    displayedComponents: [.hourAndMinute, .date],
    label: { Text("Due Date") }
)
```

如果 `DatePicker` 的标签只有 `Text`，则可以用这样更简单的方式初始化：

```swift
DatePicker("Due Date",
            selection: $selectedDate,
            in: dateClosedRange,
            displayedComponents: [.hourAndMinute, .date])
```

可以使用 [`ClosedRange`](https://developer.apple.com/documentation/swift/closedrange)、[`PartialRangeThrough`](https://developer.apple.com/documentation/swift/partialrangethrough) 和 [`PartialRangeFrom`](https://developer.apple.com/documentation/swift/partialrangefrom) 来设置 `minimumDate` 和 `maximumDate` 。

```swift
DatePicker("Minimum Date",
    selection: $selectedDate,
    in: Date()...,
    displayedComponents: [.date])
DatePicker("Maximum Date",
    selection: $selectedDate,
    in: ...Date(),
    displayedComponents: [.date])
```

[文档](https://developer.apple.com/documentation/swiftui/datepicker)

### <span id="slider">Slider</span>

从有界的线性范围中选择一个值的控件。

```swift
@State var progress: Float = 0

Slider(value: $progress, from: 0.0, through: 100.0, by: 5.0)    
```

Slider 虽然没有 `minimumValueImage` 和 `maximumValueImage` 属性， 但可以借助 `HStack`实现。

```swift
@State var progress: Float = 0
HStack {
    Image(systemName: "sun.min")
    Slider(value: $progress, from: 0.0, through: 100.0, by: 5.0)
    Image(systemName: "sun.max.fill")
}.padding()
```

[文档](https://developer.apple.com/documentation/swiftui/slider)

### <span id="stepper">Stepper</span>

用于执行语义上递增和递减动作的控件。

```swift
@State var quantity: Int = 0
Stepper(value: $quantity, in: 0...10, label: { Text("Quantity \(quantity)")})
```

如果您的 `Stepper` 的标签只有 `Text`，则可以用这样更简单的方式初始化：

```swift
Stepper("Quantity \(quantity)", value: $quantity, in: 0...10)
```

如果您要一个自己管理的数据源的控件，可以这样写：

```swift
@State var quantity: Int = 0
Stepper(onIncrement: {
    self.quantity += 1
}, onDecrement: {
    self.quantity -= 1
}, label: { Text("Quantity \(quantity)") })
```

[文档](https://developer.apple.com/documentation/swiftui/stepper)

## SwiftUI - 页面布局与演示

### <span id="hstack">HStack</span>

水平排列子元素的视图。

创建一个水平排列的静态列表：

```swift
HStack (alignment: .center, spacing: 20){
    Text("Hello")
    Divider()
    Text("World")
}
```

[文档](https://developer.apple.com/documentation/swiftui/hstack)

### <span id="vstack">VStack</span>

垂直排列子元素的视图。

创建一个垂直排列的静态列表：

```swift
VStack (alignment: .center, spacing: 20){
    Text("Hello")
    Divider()
    Text("World")
}
```

[文档](https://developer.apple.com/documentation/swiftui/vstack)

### <span id="zstack">ZStack</span>

子元素会在 z轴方向上叠加，同时在垂直/水平轴上对齐的视图。

```swift
ZStack {
    Text("Hello")
        .padding(10)
        .background(Color.red)
        .opacity(0.8)
    Text("World")
        .padding(20)
        .background(Color.red)
        .offset(x: 0, y: 40)
}
```

[文档](https://developer.apple.com/documentation/swiftui/zstack)

### <span id="list">List</span>

用于显示排列一系列数据行的容器。

创建一个静态可滚动列表：

```swift
List {
    Text("Hello world")
    Text("Hello world")
    Text("Hello world")
}
```

表单里的内容可以混搭：

```swift
List {
    Text("Hello world")
    Image(systemName: "clock")
}
```

创建一个动态列表：

```swift
let names = ["John", "Apple", "Seed"]
List(names) { name in
    Text(name)
}
```

加入分区：

```swift
List {
    Section(header: Text("UIKit"), footer: Text("We will miss you")) {
        Text("UITableView")
    }

    Section(header: Text("SwiftUI"), footer: Text("A lot to learn")) {
        Text("List")
    }
}
```

要使其成为分组列表，请添加 `.listStyle(GroupedListStyle())`：

```swift
List {
    Section(header: Text("UIKit"), footer: Text("We will miss you")) {
        Text("UITableView")
    }

    Section(header: Text("SwiftUI"), footer: Text("A lot to learn")) {
        Text("List")
    }
}.listStyle(GroupedListStyle())
```

[文档](https://developer.apple.com/documentation/swiftui/list)

### <span id="scrollview">ScrollView</span>

滚动视图。

```swift
ScrollView(alwaysBounceVertical: true) {
    Image("foo")
    Text("Hello World")
}
```

[文档](https://developer.apple.com/documentation/swiftui/scrollview)

### <span id="form">Form</span>

对数据输入的控件进行分组的容器，例如在设置或检查器中。

您可以往表单中插入任何内容，它将为表单渲染适当的样式。

```swift
NavigationView {
    Form {
        Section {
            Text("Plain Text")
            Stepper(value: $quantity, in: 0...10, label: { Text("Quantity") })
        }
        Section {
            DatePicker($date, label: { Text("Due Date") })
            Picker(selection: $selection, label:
                Text("Picker Name")
                , content: {
                    Text("Value 1").tag(0)
                    Text("Value 2").tag(1)
                    Text("Value 3").tag(2)
                    Text("Value 4").tag(3)
            })
        }
    }
}
```

[文档](https://developer.apple.com/documentation/swiftui/form)

### <span id="spacer">Spacer</span>

一块既能在包含栈布局时沿主轴伸展，也能在不包含栈时沿两个轴展开的灵活空间。

```swift
HStack {
    Image(systemName: "clock")
    Spacer()
    Text("Time")
}
```

[文档](https://developer.apple.com/documentation/swiftui/spacer)

### <span id="divider">Divider</span>

用于分隔其它内容的可视化元素。

```swift
HStack {
    Image(systemName: "clock")
    Divider()
    Text("Time")
}.fixedSize()
```

[文档](https://developer.apple.com/documentation/swiftui/divider)

### <span id="navigationview">NavigationView</span>

用于渲染视图堆栈的视图，这些视图会展示导航层次结构中的可见路径。

```swift
NavigationView {            
    List {
        Text("Hello World")
    }
    .navigationBarTitle(Text("Navigation Title")) // 默认使用大标题样式
}
```

对于旧样式标题：

```swift
NavigationView {            
    List {
        Text("Hello World")
    }
    .navigationBarTitle(Text("Navigation Title"), displayMode: .inline)
}
```

增加 `UIBarButtonItem`

```swift
NavigationView {
    List {
        Text("Hello World")
    }
    .navigationBarItems(trailing:
        Button(action: {
            // Add action
        }, label: {
            Text("Add")
        })
    )
    .navigationBarTitle(Text("Navigation Title"))
}
```

用 [NavigationLink](#navigationlink) 添加 `show`/`push` 功能。


作为 `UISplitViewController`：

```swift
NavigationView {
    List {
        NavigationLink("Go to detail", destination: Text("New Detail"))
    }.navigationBarTitle("Master")
    Text("Placeholder for Detail")
}
```

您可以使用两种新的样式属性：`stack` 和 `doubleColumn` 为 NavigationView 设置样式。默认情况下，iPhone 和 Apple TV 上的导航栏上显示导航堆栈，而在 iPad 和 Mac 上，显示的是拆分样式的导航视图。

您可以通过 `.navigationViewStyle` 重写样式：

```swift
NavigationView {
    MyMasterView()
    MyDetailView()
}
.navigationViewStyle(StackNavigationViewStyle())
```

在 beta 3中，`NavigationView` 支持拆分视图，但它仅支持非常基本的结构，其中主视图为列表，详细视图为叶视图，我期待在下一个 release 版本中能有优化补充。

[文档](https://developer.apple.com/documentation/swiftui/navigationview)

### <span id="tabview">TabView</span>

使用交互式用户界面元素在多个子视图之间切换的视图。

```swift
TabView {
    Text("First View")
        .font(.title)
        .tabItem({ Text("First") })
        .tag(0)
    Text("Second View")
        .font(.title)
        .tabItem({ Text("Second") })
        .tag(1)
}
```

标签元素支持同时显示图像和文本， 您也可以使用 SF Symbols。

```swift
TabView {
    Text("First View")
        .font(.title)
        .tabItem({
            Image(systemName: "circle")
            Text("First")
        })
        .tag(0)
    Text("Second View")
        .font(.title)
        .tabItem(VStack {
            Image("second")
            Text("Second")
        })
        .tag(1)
}
```

您也可以省略 `VStack`：

```swift
TabView {
    Text("First View")
        .font(.title)
        .tabItem({
            Image(systemName: "circle")
            Text("First")
        })
        .tag(0)
    Text("Second View")
        .font(.title)
        .tabItem({
            Image("second")
            Text("Second")
        })
        .tag(1)
}
```

[文档](https://developer.apple.com/documentation/swiftui/tabview)

### <span id="alert">Alert</span>

一个展示警告信息的容器。

我们可以根据布尔值显示 `Alert` 。

```swift
@State var isError: Bool = false

Button("Alert") {
    self.isError = true
}.alert(isPresented: $isError, content: {
    Alert(title: Text("Error"), message: Text("Error Reason"), dismissButton: .default(Text("OK")))
})
```

它也可与 `Identifiable` 项目绑定。

```swift
@State var error: AlertError?

var body: some View {
    Button("Alert Error") {
        self.error = AlertError(reason: "Reason")
    }.alert(item: $error, content: { error in
        alert(reason: error.reason)
    })    
}

func alert(reason: String) -> Alert {
    Alert(title: Text("Error"),
            message: Text(reason),
            dismissButton: .default(Text("OK"))
    )
}

struct AlertError: Identifiable {
    var id: String {
        return reason
    }
    
    let reason: String
}
```

[文档](https://developer.apple.com/documentation/swiftui/alert)

### <span id="modal">Modal</span>

模态视图的存储类型。

我们可以根据布尔值显示 `Modal` 。

```swift
@State var isModal: Bool = false

var modal: some View {
    Text("Modal")
}

Button("Modal") {
    self.isModal = true
}.sheet(isPresented: $isModal, content: {
    self.modal
})
```

[文档](https://developer.apple.com/documentation/swiftui/view/3352791-sheet)

它也可与 `Identifiable` 项目绑定。

```swift
@State var detail: ModalDetail?

var body: some View {
    Button("Modal") {
        self.detail = ModalDetail(body: "Detail")
    }.sheet(item: $detail, content: { detail in
        self.modal(detail: detail.body)
    })    
}

func modal(detail: String) -> some View {
    Text(detail)
}

struct ModalDetail: Identifiable {
    var id: String {
        return body
    }
    
    let body: String
}
```

[文档](https://developer.apple.com/documentation/swiftui/view/3352792-sheet)

### <span id="actionsheet">ActionSheet</span>

操作表视图的存储类型。

我们可以根据布尔值显示 `ActionSheet` 。

```swift
@State var isSheet: Bool = false

var actionSheet: ActionSheet {
    ActionSheet(title: Text("Action"),
                message: Text("Description"),
                buttons: [
                    .default(Text("OK"), action: {
                        
                    }),
                    .destructive(Text("Delete"), action: {
                        
                    })
                ]
    )
}

Button("Action Sheet") {
    self.isSheet = true
}.actionSheet(isPresented: $isSheet, content: {
    self.actionSheet
})
```

它也可与 `Identifiable` 项目绑定。

```swift
@State var sheetDetail: SheetDetail?

var body: some View {
    Button("Action Sheet") {
        self.sheetDetail = ModSheetDetail(body: "Detail")
    }.actionSheet(item: $sheetDetail, content: { detail in
        self.sheet(detail: detail.body)
    })
}

func sheet(detail: String) -> ActionSheet {
    ActionSheet(title: Text("Action"),
                message: Text(detail),
                buttons: [
                    .default(Text("OK"), action: {
                        
                    }),
                    .destructive(Text("Delete"), action: {
                        
                    })
                ]
    )
}

struct SheetDetail: Identifiable {
    var id: String {
        return body
    }
    
    let body: String
}
```

[文档](https://developer.apple.com/documentation/swiftui/actionsheet)

## 框架集成 - SwiftUI 中的 UIKit

### <span id="uiviewrepresentable">UIViewRepresentable</span>

表示 UIKit 视图的视图，当您想在 SwiftUI 中使用 UIView 时，请使用它。

要使任何 UIView 在 SwiftUI 中可用，请创建一个符合 UIViewRepresentable 的包装器视图。

```swift
import UIKit
import SwiftUI

struct ActivityIndicator: UIViewRepresentable {
    @Binding var isAnimating: Bool
    
    func makeUIView(context: Context) -> UIActivityIndicatorView {
        let v = UIActivityIndicatorView()
        
        return v
    }
    
    func updateUIView(_ uiView: UIActivityIndicatorView, context: Context) {
        if isAnimating {
            uiView.startAnimating()
        } else {
            uiView.stopAnimating()
        }
    }
}
```

如果您想要桥接 UIKit 里的数据绑定 (delegate, target/action) 就使用 `Coordinator`， 具体见 [SwiftUI 教程](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit)。

```swift
import SwiftUI
import UIKit

struct PageControl: UIViewRepresentable {
    var numberOfPages: Int
    @Binding var currentPage: Int

    func makeUIView(context: Context) -> UIPageControl {
        let control = UIPageControl()
        control.numberOfPages = numberOfPages
        control.addTarget(
            context.coordinator,
            action: #selector(Coordinator.updateCurrentPage(sender:)),
            for: .valueChanged)

        return control
    }

    func updateUIView(_ uiView: UIPageControl, context: Context) {
        uiView.currentPage = currentPage
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    // This is where old paradigm located
    class Coordinator: NSObject {
        var control: PageControl

        init(_ control: PageControl) {
            self.control = control
        }

        @objc func updateCurrentPage(sender: UIPageControl) {
            control.currentPage = sender.currentPage
        }
    }
}
```

[文档](https://developer.apple.com/documentation/swiftui/uiviewrepresentable)

### <span id="uiviewcontrollerrepresentable">UIViewControllerRepresentable</span>

表示 UIKit 视图控制器的视图。当您想在 SwiftUI 中使用 UIViewController 时，请使用它。

要使任何 UIViewController 在 SwiftUI 中可用，请创建一个符合 UIViewControllerRepresentable 的包装器视图，具体见 [SwiftUI 教程](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit)。

```swift
import SwiftUI
import UIKit

struct PageViewController: UIViewControllerRepresentable {
    var controllers: [UIViewController]

    func makeUIViewController(context: Context) -> UIPageViewController {
        let pageViewController = UIPageViewController(
            transitionStyle: .scroll,
            navigationOrientation: .horizontal)

        return pageViewController
    }

    func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
        pageViewController.setViewControllers(
            [controllers[0]], direction: .forward, animated: true)
    }
}
```

[文档](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable)

## 框架集成 - UIKit 中的 SwiftUI

### <span id="uihostingcontroller">UIHostingController</span>

表示 SwiftUI 视图的 UIViewController。

```swift
let vc = UIHostingController(rootView: Text("Hello World"))
let vc = UIHostingController(rootView: ContentView())
```

[文档](https://developer.apple.com/documentation/swiftui/uihostingcontroller)

## 来源

- [API 文档](https://developer.apple.com/documentation/swiftui/)
- [官方教程](https://developer.apple.com/tutorials/swiftui/tutorials)
- [WWDC 2019](https://developer.apple.com/videos/wwdc2019/)
  - [介绍 SwiftUI: 创建您的第一个 App](https://developer.apple.com/videos/play/wwdc2019/204/)
  - [SwiftUI 基础](https://developer.apple.com/videos/play/wwdc2019/216/)
  - [SwiftUI 数据流](https://developer.apple.com/videos/play/wwdc2019/226/)
  - [使用 SwiftUI 构建自定义视图](https://developer.apple.com/videos/play/wwdc2019/237/)
  - [集成 SwiftUI](https://developer.apple.com/videos/play/wwdc2019/231/)
  - [SwiftUI 中的可访问性](https://developer.apple.com/videos/play/wwdc2019/238/)
  - [所有设备上的 SwiftUI](https://developer.apple.com/videos/play/wwdc2019/240/)
  - [watchOS 上的 SwiftUI](https://developer.apple.com/videos/play/wwdc2019/219/)
  - [掌握 Xcode 预览](https://developer.apple.com/videos/play/wwdc2019/233/)
