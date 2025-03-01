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



