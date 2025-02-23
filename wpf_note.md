# 1. 深入浅出WPF
1. 3中project: 控制台、应用程序、类库。
2. 项目所编译出来的结果称为程序集Assembly。
3. 如果类派生自FrameworkElemetn,使用Name和X:Name是相同的，如果不是继承自FrameWorkElement需要使用X:Name；

# 公司电脑有笔记，记得update

## code snipet
例如for然后两次tab键，可以自动填充；
也可以新增自己的功能，在工具，代码片段填充管理器中新增；

## MVVM设计模式
Model: Represents the data and business logic. It is independent of the UI and contains the application's core logic.
View: Represents the UI. In WPF, this is typically defined in XAML files. The View binds to properties and commands in the ViewModel.
ViewModel: Acts as an intermediary between the Model and the View. It exposes data and commands to the View via binding. The ViewModel manipulates the Model and updates the View.

# 2. CommunityToolkit 
## 1. 介绍

**CommunityToolkit.MVVM**（以前称为 **Microsoft.Toolkit.MVVM**）是一个轻量级、高性能的 MVVM（Model-View-ViewModel）框架，专为 .NET 开发设计。它是 .NET 社区工具包（.NET Community Toolkit）的一部分，旨在简化 MVVM 模式的实现，特别适合 WPF、WinUI、UWP 和 .NET MAUI 等 XAML 平台。


## 2. **安装 CommunityToolkit.MVVM**
通过 NuGet 安装：
```bash
dotnet add package CommunityToolkit.Mvvm
```

---

## 3. **核心功能**

### (1) **ObservableObject**
`ObservableObject` 是一个基类，用于实现 `INotifyPropertyChanged` 接口，简化属性通知的代码。

**示例**：
```csharp
using CommunityToolkit.Mvvm.ComponentModel;

public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private string _name;

    [ObservableProperty]
    private int _age;
}
```
- 使用 `[ObservableProperty]` 属性标记字段，源生成器会自动生成对应的属性，并实现属性更改通知。

---

### (2) **RelayCommand**
`RelayCommand` 和 `AsyncRelayCommand` 用于实现命令模式，简化事件绑定。

**示例**：
```csharp
using CommunityToolkit.Mvvm.Input;

public partial class MainViewModel : ObservableObject
{
    [RelayCommand]
    private void SayHello()
    {
        MessageBox.Show($"Hello, {Name}!");
    }

    [RelayCommand]
    private async Task LoadDataAsync()
    {
        await Task.Delay(1000); // 模拟异步操作
        Age = 30;
    }
}
```
- 使用 `[RelayCommand]` 标记方法，源生成器会自动生成对应的命令属性。

---

### (3) **依赖注入**
CommunityToolkit.MVVM 支持依赖注入，可以通过构造函数注入服务。

**示例**：
```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly IDataService _dataService;

    public MainViewModel(IDataService dataService)
    {
        _dataService = dataService;
    }

    [RelayCommand]
    private void LoadData()
    {
        var data = _dataService.GetData();
        // 处理数据
    }
}
```

---

### (4) **Messenger**
`Messenger` 用于实现 ViewModel 之间的松耦合通信。

**示例**：
```csharp
using CommunityToolkit.Mvvm.Messaging;

// 定义消息
public class LoggedInUserMessage
{
    public string Username { get; set; }
}

// 发送消息
WeakReferenceMessenger.Default.Send(new LoggedInUserMessage { Username = "Alice" });

// 接收消息
WeakReferenceMessenger.Default.Register<LoggedInUserMessage>(this, (recipient, message) =>
{
    MessageBox.Show($"User {message.Username} logged in.");
});
```

---

## 4. **在 WPF 项目中使用 CommunityToolkit.MVVM**

### (1) **创建 ViewModel**
```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private string _name;

    [RelayCommand]
    private void SayHello()
    {
        MessageBox.Show($"Hello, {Name}!");
    }
}
```

### (2) **绑定 ViewModel 到 View**
在 XAML 中绑定 ViewModel：
```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:WpfApp"
        Title="MainWindow" Height="350" Width="525">
    <Window.DataContext>
        <local:MainViewModel />
    </Window.DataContext>
    <StackPanel>
        <TextBox Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}" />
        <Button Content="Say Hello" Command="{Binding SayHelloCommand}" />
    </StackPanel>
</Window>
```


# WPF项目实战合集
链接：https://github.com/HenJigg/my-todoapp
教程：https://www.bilibili.com/video/BV1nY411a7T8/?p=31&spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=ef7d0fcabfe8e08f21e9c986b7805614

## 控件模板（ControlTemplate） 和 数据模板（DataTemplate）
控件模板
```xml
  <Window.Resources>
        <!-- 定义按钮的控件模板 -->
        <ControlTemplate x:Key="RoundButtonTemplate" TargetType="Button">
            <Grid>
                <!-- 圆形背景 -->
                <Ellipse Fill="LightBlue" Stroke="DarkBlue" StrokeThickness="2" />
                <!-- 按钮内容 -->
                <ContentPresenter HorizontalAlignment="Center" VerticalAlignment="Center" />
            </Grid>
        </ControlTemplate>
    </Window.Resources>
    <Grid>
        <!-- 使用自定义模板的按钮 -->
        <Button Template="{StaticResource RoundButtonTemplate}" Content="Click Me" Width="100" Height="100" />
    </Grid>
```
数据模板
```XML
 <Window.Resources>
        <!-- 定义数据模板 -->
        <DataTemplate x:Key="PersonTemplate">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}" FontWeight="Bold" />
                <TextBlock Text=" - " />
                <TextBlock Text="{Binding Age}" Foreground="Gray" />
            </StackPanel>
        </DataTemplate>
    </Window.Resources>
    <Grid>
        <!-- 使用数据模板的 ListBox -->
        <ListBox ItemsSource="{Binding People}" ItemTemplate="{StaticResource PersonTemplate}" />
    </Grid>
```
```CSHARP
using System.Collections.Generic;
using System.Windows;

namespace WpfApp
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            // 绑定数据
            DataContext = new
            {
                People = new List<Person>
                {
                    new Person { Name = "Alice", Age = 25 },
                    new Person { Name = "Bob", Age = 30 },
                    new Person { Name = "Charlie", Age = 35 }
                }
            };
        }
    }

    public class Person
    {
        public string Name { get; set; }
        public int Age { get; set; }
    }
}
```
## 数据绑定（Data Binding）
1.1 绑定组件

    源（Source）：数据的提供者，可以是对象、属性、集合等。
    目标（Target）：数据的接收者，通常是 UI 元素的依赖属性（如 TextBox.Text）。
    绑定路径（Path）：指定从源中获取数据的路径（如属性名称）。

1.2 绑定模式（Binding Mode）

    OneWay：数据从源流向目标（默认模式）。
    TwoWay：数据在源和目标之间双向流动。
    OneWayToSource：数据从目标流向源。
    OneTime：数据仅在初始化时从源流向目标。

1.3 更新触发机制（UpdateSourceTrigger)

    PropertyChanged：目标属性更改时立即更新源。
    LostFocus：目标失去焦点时更新源。
    Explicit：显式调用 UpdateSource 方法时更新源。

```xml
 <StackPanel VerticalAlignment="Center" HorizontalAlignment="Center">
        <TextBox Text="{Binding Age, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" Width="100" />
        <Slider Value="{Binding Age, Mode=TwoWay}" Minimum="0" Maximum="100" Width="200" />
    </StackPanel>
```
## 资源字典（ResourceDictionary）
在 WPF 中，**资源字典（ResourceDictionary）** 是一种用于集中管理和重用资源（如样式、模板、画笔、动画等）的机制。通过资源字典，可以将资源定义在单独的文件中，并在多个地方引用，从而提高代码的可维护性和复用性。

---

### 1. **创建资源字典文件**

  1. 在项目中右键点击，选择 **添加 -> 新建项**。
  2. 选择 **资源字典（ResourceDictionary）**，并为其命名（如 `Styles.xaml`）。
  3. 点击 **添加**，生成一个 `.xaml` 文件。

### 示例：`Styles.xaml`
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- 定义资源 -->
    <SolidColorBrush x:Key="PrimaryBrush" Color="LightBlue" />
    <SolidColorBrush x:Key="SecondaryBrush" Color="DarkBlue" />

    <!-- 定义样式 -->
    <Style x:Key="PrimaryButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource PrimaryBrush}" />
        <Setter Property="Foreground" Value="White" />
        <Setter Property="FontSize" Value="14" />
        <Setter Property="Padding" Value="10 5" />
    </Style>
</ResourceDictionary>
```
---

### 2. **在应用程序中引用资源字典**

### 方法 1：在 `App.xaml` 中引用
将资源字典添加到 `App.xaml` 中，使其在整个应用程序中可用。

#### 示例：`App.xaml`
```xml
<Application x:Class="WpfApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <!-- 引用资源字典 -->
                <ResourceDictionary Source="Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

#### 方法 2：在特定窗口中引用
如果资源字典仅在特定窗口或页面中使用，可以在该窗口或页面的 `Resources` 中引用。

#### 示例：`MainWindow.xaml`
```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="350" Width="525">
    <Window.Resources>
        <!-- 引用资源字典 -->
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Window.Resources>
    <Grid>
        <!-- 使用资源 -->
        <Button Content="Primary Button" Style="{StaticResource PrimaryButtonStyle}" HorizontalAlignment="Center" VerticalAlignment="Center" />
    </Grid>
</Window>
```

---

### 3. **在代码中动态加载资源字典**

如果需要根据条件动态加载资源字典，可以在代码中实现。

### 示例：动态加载资源字典
```csharp
using System.Windows;

namespace WpfApp
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            // 动态加载资源字典
            var resourceDictionary = new ResourceDictionary();
            resourceDictionary.Source = new System.Uri("Styles.xaml", System.UriKind.Relative);
            Resources.MergedDictionaries.Add(resourceDictionary);
        }
    }
}
```

---

### 4. **合并多个资源字典**

可以将多个资源字典合并到一个资源字典中，方便统一管理。

### 示例：合并资源字典
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <!-- 引用多个资源字典 -->
        <ResourceDictionary Source="Styles.xaml" />
        <ResourceDictionary Source="Brushes.xaml" />
        <ResourceDictionary Source="Templates.xaml" />
    </ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```

---

### 5. **使用资源字典中的资源**

在 XAML 或代码中，可以通过 `StaticResource` 或 `DynamicResource` 引用资源字典中的资源。

### 示例：使用资源
```xml
<Button Content="Primary Button" Style="{StaticResource PrimaryButtonStyle}" />
```