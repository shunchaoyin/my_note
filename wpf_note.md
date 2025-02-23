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