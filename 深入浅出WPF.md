
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



