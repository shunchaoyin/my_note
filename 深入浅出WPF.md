
1. WPF（Windows Presentatation Foundation）是用来编写应用程序的表示层,也有其它表示层例如，Windows Forms,ASP.Net,Silverlight等，业务逻辑层和数据层有自己的开发技术，比如WCF(Windows Communication Foundation)和WF（Windows Workflow Foundation）
2. WPF采用的是数据驱动，而MVC（Model View Controler）和MVP（Model View presenter）采用的是事件驱动，这很容易导致界面逻辑和业务逻辑混合。
3. 区别Attribute和property，xmal中两者往往可以相互转换，例如<Grid Height= "100" />中表示Attribute,而<Grid><Height= "100"/></Grid>表示Property,一般来说Property表示对象的成员，而Attribute是编程语法层面的东西。
4. 同一个界面会存在多种不同的xmal的写法，如下可以简化XAML：
   1. 尽量使用Attribute形式，而不使用Property
   2. 利用默认值，尽量不赋值
   3. 利用XMAL的简写方式，有些标签可以省略。
5. 标记拓展（Markup Extension）使用大括号的标签，例如<TextBox Text =“{ Binding ElementName = "slider1" Path = "Value"}” />
   1. 标记拓展，可以嵌套
   2. 标记拓展，可以简写，例如{Binding Value}和{Binding Path= “Valu”}
   3. 标记扩展的类名均以Extension结尾，在XAML中可以省略，例如Text＝{"x:static}而不用写成Text＝{"x:StaticExtension}
6. XMAL中应该使用x:Name还是Name,NAME属性定义在FrameWorkElement中，这是WPF的所有基类，所有控件都有Name属性。但是如果改类不是FrameWorkElement类就要使用x:Name,尽量统一全部使用x:Name

## C#中WPF中X命名空间中的标记拓展

在 WPF 中，**x 命名空间**（`xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"`）提供了一些特殊的标记扩展（Markup Extensions），用于在 XAML 中实现动态绑定、资源引用、类型转换等功能。这些标记扩展是 WPF 开发中非常重要的工具。

| 标记扩展              | 描述                                                                 |
|-----------------------|----------------------------------------------------------------------|
| **x:Static**          | 引用静态字段、属性或常量。                                           |
| **x:Type**            | 引用类型（如类、结构体、枚举等）。                                   |
| **x:Null**            | 将属性设置为 `null`。                                                |
| **x:Array**           | 在 XAML 中定义数组。                                                 |
| **x:Reference**       | 引用 XAML 中的其他元素。                                             |
| **x:Key**             | 为资源或样式指定键值。                                               |
| **x:Name**            | 为元素指定名称，以便在代码中引用。                                   |
| **x:Code**            | 在 XAML 中嵌入 C# 代码。                                             |

通过灵活使用 `x` 命名空间中的标记扩展，可以显著提高 WPF 应用程序的开发效率和代码的可维护性。

## WPF的六种控件类型

在WPF（Windows Presentation Foundation）中，控件是构建用户界面的基本元素。WPF控件可以分为以下几类：

| **控件类型**  | **用途**                             | **常见控件**                                                                 |
|---------------|--------------------------------------|-----------------------------------------------------------------------------|
| **布局控件**  | 管理子控件的排列和布局               | `Grid`、`StackPanel`、`DockPanel`、`WrapPanel`、`Canvas`、`UniformGrid`     |
| **内容控件**  | 显示单一内容（文本、图像或其他控件） | `Button`、`Label`、`Window`、`GroupBox`、`TabItem`                          |
| **条目控件**  | 显示多个条目或数据集合               | `ListBox`、`ComboBox`、`ListView`、`TreeView`、`DataGrid`                   |
| **文本控件**  | 显示或编辑文本内容                   | `TextBox`、`RichTextBox`、`TextBlock`、`PasswordBox`                        |
| **范围控件**  | 表示一个范围内的值                   | `Slider`、`ProgressBar`、`ScrollBar`                                        |
| **特殊控件**  | 具有特定功能的控件                   | `Image`、`MediaElement`、`WebBrowser`、`Calendar`、`DatePicker`             |

```xml
Object
    └── DispatcherObject
        └── DependencyObject
            └── Visual
                └── UIElement
                    └── FrameworkElement
                        ├── Control (内容控件、条目控件、文本控件、范围控件、特殊控件)
                        │   ├── ContentControl (内容控件)
                        │   │   ├── Button
                        │   │   ├── Label
                        │   │   ├── Window
                        │   │   ├── GroupBox
                        │   │   └── TabItem
                        │   ├── ItemsControl (条目控件)
                        │   │   ├── ListBox
                        │   │   ├── ComboBox
                        │   │   ├── ListView
                        │   │   ├── TreeView
                        │   │   └── DataGrid
                        │   ├── TextBoxBase (文本控件基类)
                        │   │   ├── TextBox
                        │   │   └── RichTextBox
                        │   ├── RangeBase (范围控件基类)
                        │   │   ├── Slider
                        │   │   ├── ProgressBar
                        │   │   └── ScrollBar
                        │   └── 特殊控件
                        │       ├── Image
                        │       ├── MediaElement
                        │       ├── WebBrowser
                        │       ├── Calendar
                        │       └── DatePicker
                        └── Panel (布局控件)
                            ├── Grid
                            ├── StackPanel
                            ├── DockPanel
                            ├── WrapPanel
                            ├── Canvas
                            └── UniformGrid
```

### 1. **布局控件**
   - 用于管理子控件的排列和布局。
   - 常见控件：
     - `Grid`：网格布局，支持行和列的定义。
     - `StackPanel`：按水平或垂直方向排列子控件。
     - `DockPanel`：子控件可以停靠在面板的边缘。
     - `WrapPanel`：子控件按顺序排列，超出边界时自动换行。
     - `Canvas`：通过绝对坐标定位子控件。
     - `UniformGrid`：均匀分布的网格布局。
### 尽量不要使用`Canvas`布局，除非哪些不需要改动的布局，如图标

### UI设计时尽量XAML中不要使用“Margin"

### TextBox 和 TextBlock 的主要区别
| 特性         | TextBox                    | TextBlock                       |
|--------------|----------------------------|---------------------------------|
| 用途         | 用户输入和编辑文本        | 显示只读文本                    |
| 编辑功能     | 支持用户编辑              | 不支持用户编辑                  |
| 文本格式化   | 仅支持纯文本              | 支持简单格式化文本（通过Inlines）|
| 性能         | 较慢（适合交互式输入）    | 较快（适合静态显示）            |
| 常用场景     | 表单输入、多行文本编辑    | 标签、静态文本显示              |
| 绑定支持     | 支持双向绑定（Text属性）  | 支持单向绑定（Text属性）        |
| 事件支持     | 支持TextChanged等事件     | 无特定事件                      |

### `List<T>` 和 `ObservableCollection<T>` 

**主要区别**

| 特性                     | `List<T>`                          | `ObservableCollection<T>`          |
|--------------------------|------------------------------------|------------------------------------|
| **命名空间**             | `System.Collections.Generic`      | `System.Collections.ObjectModel`  |
| **数据绑定支持**         | 不支持自动通知 UI 更新            | 支持自动通知 UI 更新              |
| **性能**                 | 更高                               | 较低（因为需要处理通知机制）      |
| **适用场景**             | 后台数据处理、非 UI 绑定场景       | UI 绑定场景（如 WPF、Xamarin）    |
| **事件通知**             | 无                                 | 提供 `CollectionChanged` 事件     |

---
- `List<T>` 是一个通用的高性能集合，适合后台数据处理。
- `ObservableCollection<T>` 是为 UI 数据绑定设计的集合，支持自动更新通知。



### 依赖属性（Dependency Property）的存在原因

---

| **原因/优势**            | **传统 CLR 属性的局限性**                          | **依赖属性的解决方案**                          | **示例/说明**                                                                 |
|--------------------------|--------------------------------------------------|-----------------------------------------------|-------------------------------------------------------------------------------|
| **支持数据绑定**         | 无法直接支持数据绑定。                            | 内置数据绑定支持，实现双向绑定和动态更新。      | `<TextBox Text="{Binding UserName}" />`                                       |
| **支持动画**             | 无法直接被动画系统修改。                          | 可以被动画系统直接操作，实现平滑动画效果。      | `<DoubleAnimation Storyboard.TargetProperty="Opacity" From="0" To="1" />`     |
| **支持样式和模板**       | 无法通过样式和模板动态设置值。                    | 可以通过样式、模板和触发器动态设置值。          | `<Style TargetType="Button"><Setter Property="Background" Value="LightBlue"/></Style>` |
| **属性值继承**           | 无法实现属性值的继承。                            | 支持属性值继承，父元素的值可以传递给子元素。    | `<StackPanel TextElement.FontSize="20"><Button Content="Button 1"/></StackPanel>` |
| **内存优化**             | 每个对象都会为属性分配内存，即使未设置值。        | 只有在显式设置时分配内存，未设置时使用默认值。  | `DependencyProperty.Register("MyProperty", typeof(string), typeof(MyControl))` |
| **属性值优先级**         | 无法处理多个值源（如本地值、样式、动画等）的优先级。 | 支持多值源，并根据优先级决定最终值。            | 优先级顺序：动画 > 本地值 > 模板属性 > 样式触发器 > 样式设置器 > 默认值。      |
| **属性更改通知**         | 需要手动实现属性更改通知（如 `INotifyPropertyChanged`）。 | 内置属性更改通知机制，自动触发通知。            | `PropertyMetadata(OnMyPropertyChanged)`                                       |
| **只读属性支持**         | 无法实现只读属性。                                | 支持只读依赖属性，通过 `RegisterReadOnly` 注册。| `DependencyProperty.RegisterReadOnly("IsActive", typeof(bool), typeof(MyControl))` |

---

### DependencyObject 和DependencyProperty
- **`DependencyObject`**:
  - 是依赖属性的载体，负责存储和管理依赖属性的值。
  - 提供 `GetValue` 和 `SetValue` 方法，用于操作依赖属性。
  - 支持属性值继承和属性更改通知的触发。

- **`DependencyProperty`**:
  - 是依赖属性的定义，负责注册依赖属性并定义其行为（如默认值、回调函数等）。
  - 提供属性值优先级规则和属性更改通知机制。

- **协作方式**:
  - `DependencyProperty` 定义属性的规则和行为。
  - `DependencyObject` 存储属性的值，并根据 `DependencyProperty` 的规则管理属性。


### **示例代码**
```csharp
public class MyControl : DependencyObject
{
    // 注册依赖属性
    public static readonly DependencyProperty MyPropertyProperty =
        DependencyProperty.Register(
            name: "MyProperty",
            propertyType: typeof(string),
            ownerType: typeof(MyControl),
            typeMetadata: new FrameworkPropertyMetadata(
                defaultValue: "Default Value",
                propertyChangedCallback: OnMyPropertyChanged
            )
        );

    // CLR 属性包装器
    public string MyProperty
    {
        get { return (string)GetValue(MyPropertyProperty); }
        set { SetValue(MyPropertyProperty, value); }
    }

    // 属性更改回调
    private static void OnMyPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        Console.WriteLine($"MyProperty changed from {e.OldValue} to {e.NewValue}");
    }
}
```
```xml
<Window x:Class="MyApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:MyApp">
    <Grid>
        <local:MyControl MyProperty="Hello, World!" />
    </Grid>
</Window>
```

```csharp
MyControl control = new MyControl();
control.MyProperty = "New Value";  // 设置属性值
Console.WriteLine(control.MyProperty);  // 获取属性值

//输出
MyProperty changed from Default Value to Hello, World!
MyProperty changed from Hello, World! to New Value
```

---

### **附加属性？**
**附加属性（Attached Property）** 是 WPF（Windows Presentation Foundation）和 UWP（Universal Windows Platform）等 XAML 框架中的一种特殊类型的依赖属性。它允许一个对象为另一个对象定义属性，即使该属性不属于定义它的类。附加属性通常用于布局、动画和数据绑定等场景。

附加属性的特点
- **不属于定义它的类**: 附加属性定义在一个类中，但可以附加到其他类的对象上。
- **全局可用**: 可以在任何对象上使用附加属性。
- **支持依赖属性功能**: 附加属性具备依赖属性的所有功能，如数据绑定、动画、样式等。

场景描述
Human 类: 表示一个人，具有姓名、年龄等属性。
School 类: 表示一所学校，具有名称、地址等属性。
附加属性: 为 Human 对象附加一个 School 属性，表示这个人所在的学校。



```csharp
using System;
using System.Windows;

public class Human : DependencyObject
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public class School
{
    // 1. 注册附加属性
    public static readonly DependencyProperty SchoolProperty =
        DependencyProperty.RegisterAttached(
            name: "School",                     // 属性名称
            propertyType: typeof(string),        // 属性类型（学校名称）
            ownerType: typeof(School),          // 所有者类型
            defaultMetadata: new FrameworkPropertyMetadata(
                defaultValue: "Unknown School",  // 默认值
                propertyChangedCallback: OnSchoolChanged // 属性更改回调
            )
        );

    // 2. 定义 Get 方法
    public static string GetSchool(DependencyObject obj)
    {
        return (string)obj.GetValue(SchoolProperty);
    }

    // 3. 定义 Set 方法
    public static void SetSchool(DependencyObject obj, string value)
    {
        obj.SetValue(SchoolProperty, value);
    }

    // 4. 属性更改回调方法
    private static void OnSchoolChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        var human = d as Human;
        string newSchool = e.NewValue as string;
        string oldSchool = e.OldValue as string;

        // 处理属性更改逻辑
        if (human != null)
        {
            Console.WriteLine($"{human.Name} 的学校从 {oldSchool} 更改为 {newSchool}");
        }
    }
}
```

```xml
<Window x:Class="MyApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:MyApp">
    <Grid>
        <local:Human x:Name="Student" Name="Alice" Age="15" local:School.School="Greenwood High" />
    </Grid>
</Window>
```

#### **代码示例**:
```csharp
class Program
{
    static void Main(string[] args)
    {
        // 创建一个 Human 对象
        Human student = new Human { Name = "Alice", Age = 15 };

        // 设置附加属性
        School.SetSchool(student, "Greenwood High");

        // 获取附加属性
        string schoolName = School.GetSchool(student);
        Console.WriteLine($"{student.Name} 的学校是 {schoolName}");
    }
}
//输出
Alice 的学校从 Unknown School 更改为 Greenwood High
Alice 的学校是 Greenwood High
```

---
### 附加属性和依赖属性的snippet：propdp,propa

### **WPF 附加事件（Attached Event）**

附加事件是一种特殊的路由事件，允许某个类定义事件，但其他类的对象可以订阅和处理该事件。其核心特性如下：
- **事件拥有者与事件处理者解耦**：事件定义在一个类中，但可以被其他任意元素订阅。
- **基于路由事件机制**：附加事件本质是路由事件的扩展，遵循相同的冒泡、隧道和直接传播策略。
- **常见应用**：WPF 内置的许多事件（如 `Mouse.MouseDown`、`Keyboard.KeyDown`）本质是附加事件。

| **特性**         | **路由事件**                     | **附加事件**                     |
|-------------------|----------------------------------|----------------------------------|
| **定义位置**       | 定义在触发事件的类中（如 `Button` 的 `Click`） | 定义在其他类中（如 `Mouse` 类的 `MouseDown`） |
| **使用场景**       | 处理元素自身的行为（如按钮点击）    | 跨元素监听事件（如全局鼠标事件）   |
| **语法**          | 直接通过元素名访问（如 `<Button Click="...">`） | 通过附加属性语法访问（如 `<Grid Mouse.MouseDown="...">`） |

示例 1：在 `Grid` 中统一处理子元素的鼠标事件**
```xml
<Grid Mouse.MouseDown="Grid_MouseDown">
    <Button Content="Button 1" Margin="10" />
    <Button Content="Button 2" Margin="10" />
    <TextBox Width="100" Margin="10" />
</Grid>
```

```csharp
private void Grid_MouseDown(object sender, MouseButtonEventArgs e)
{
    var sourceElement = e.OriginalSource as FrameworkElement;
    if (sourceElement != null)
    {
        MessageBox.Show($"在 Grid 中捕获到 {sourceElement.GetType().Name} 的鼠标点击");
    }
}
```
**场景**：创建一个 `MessageAttachedEvent` 类，允许任意元素触发自定义消息事件。

#### **步骤 1：定义附加事件**
```csharp
public class MessageAttachedEvent
{
    // 注册附加事件
    public static readonly RoutedEvent MessageEvent =
        EventManager.RegisterRoutedEvent(
            "Message",
            RoutingStrategy.Bubble,
            typeof(RoutedEventHandler),
            typeof(MessageAttachedEvent));

    // 提供添加和移除处理程序的方法（附加事件语法）
    public static void AddMessageHandler(DependencyObject obj, RoutedEventHandler handler)
    {
        if (obj is UIElement element)
        {
            element.AddHandler(MessageEvent, handler);
        }
    }

    public static void RemoveMessageHandler(DependencyObject obj, RoutedEventHandler handler)
    {
        if (obj is UIElement element)
        {
            element.RemoveHandler(MessageEvent, handler);
        }
    }

    // 触发事件的方法
    public static void RaiseMessage(UIElement element, string message)
    {
        RoutedEventArgs args = new RoutedEventArgs(MessageEvent, element)
        {
            // 自定义数据可存储在 EventArgs 子类中
            Source = element
        };
        element.RaiseEvent(args);
    }
}
```
```xml
<Window xmlns:local="clr-namespace:YourNamespace"
        local:MessageAttachedEvent.Message="Window_Message">
    <StackPanel>
        <Button Content="发送消息" Click="Button_Click" />
    </StackPanel>
</Window>
```
处理事件与触发事件**
```csharp
// 事件处理
private void Window_Message(object sender, RoutedEventArgs e)
{
    MessageBox.Show("接收到自定义消息事件！");
}

// 触发事件
private void Button_Click(object sender, RoutedEventArgs e)
{
    var button = sender as Button;
    MessageAttachedEvent.RaiseMessage(button, "Hello from Button");
}
```

- **附加事件本质是静态路由事件**：通过 `EventManager.RegisterRoutedEvent` 注册，但通过附加属性模式暴露。
- **事件处理程序存储**：附加事件的处理程序最终会被添加到目标元素的 `RoutedEventHandlers` 列表中。

---



