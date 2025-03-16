# 文档资料
1. [官方文档](https://learn.microsoft.com/zh-cn/dotnet/communitytoolkit/mvvm/)
2. [官方代码Demo](https://github.com/CommunityToolkit/MVVM-Samples)
3. 


# MVVM设计模式
Model: Represents the data and business logic. It is independent of the UI and contains the application's core logic.
View: Represents the UI. In WPF, this is typically defined in XAML files. The View binds to properties and commands in the ViewModel.
ViewModel: Acts as an intermediary between the Model and the View. It exposes data and commands to the View via binding. The ViewModel manipulates the Model and updates the View.

# 2. CommunityToolkit 
## 1. 介绍

**CommunityToolkit.MVVM**（以前称为 **Microsoft.Toolkit.MVVM**）是一个轻量级、高性能的 MVVM（Model-View-ViewModel）框架，专为 .NET 开发设计。它是 .NET 社区工具包（.NET Community Toolkit）的一部分，旨在简化 MVVM 模式的实现，特别适合 WPF、WinUI、UWP 和 .NET MAUI 等 XAML 平台。

# 3. **核心功能**
## 1. MVVM 源生成器
[源生成器](https://learn.microsoft.com/zh-cn/dotnet/csharp/roslyn-sdk/#source-generators)

MVVM 工具包包含全新的 Roslyn 源生成器，有助于在使用 MVVM 体系结构编写代码时大幅减少样本。 它们可以简化需要设置可观察属性、命令等的方案。 如果你不熟悉源生成器，可以在上面链接详细了解。 以下是工作原理的简化图：
![alt text](image.png)

### (1) **ObservableProperty 特性**
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

生成的代码实际上比这要复杂一些，原因是它还公开了一些方法，你可以通过实现这些方法挂钩到通知逻辑，并在属性即将更新时和刚刚更新后运行其他逻辑（如果需要）。 也就是说，生成的代码实际上类似如下：
```csharp
public string? Name
{
    get => name;
    set
    {
        if (!EqualityComparer<string?>.Default.Equals(name, value))
        {
            string? oldValue = name;
            OnNameChanging(value);
            OnNameChanging(oldValue, value);
            OnPropertyChanging();
            name = value;
            OnNameChanged(value);
            OnNameChanged(oldValue, value);
            OnPropertyChanged();
        }
    }
}

partial void OnNameChanging(string? value);
partial void OnNameChanged(string? value);

partial void OnNameChanging(string? oldValue, string? newValue);
partial void OnNameChanged(string? oldValue, string? newValue);
```
如果想要运行一些仅需要引用属性已设置的新值的逻辑，则可使用前两个重载。 如果你有一些更复杂的逻辑并且还必须更新所设置的旧值和新值的某些状态，则可使用其他两个重载。
```csharp
[ObservableProperty]
private string? name;

partial void OnNameChanging(string? value)
{
    Console.WriteLine($"Name is about to change to {value}");
}

partial void OnNameChanged(string? value)
{
    Console.WriteLine($"Name has changed to {value}");
}
```
 如果未实现它们（或者只实现一个），编译器将删除整个调用，以便在不需要此附加功能时避免造成性能下降。

### 通知依赖属性
假设有一个 FullName 属性，你希望每当 Name 更改时都为之引发通知。 可以使用 NotifyPropertyChangedFor 特性实现这一点，如下所示：
```csharp
[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FullName))]
private string? name;

这将导致生成的属性等效于：
public string? Name
{
    get => name;
    set
    {
        if (SetProperty(ref name, value))
        {
            OnPropertyChanged("FullName");
        }
    }
}
```
### 通知依赖命令
假设你有一个命令，其执行状态依赖于此属性的值。 也就是说，每当此属性发生更改时，命令的执行状态都应失效并再次计算。 换句话说，应再次引发 ICommand.CanExecuteChanged。 可以通过使用 NotifyCanExecuteChangedFor 特性来实现此目的。
```CSHARP
[ObservableProperty]
[NotifyCanExecuteChangedFor(nameof(MyCommand))]
private string? name;

等效于
public string? Name
{
    get => name;
    set
    {
        if (SetProperty(ref name, value))
        {
            MyCommand.NotifyCanExecuteChanged();
        }
    }
}
```
### 请求属性验证
没有看懂呀！！！！！
如果在继承自 ObservableValidator 的类型中声明属性，则也可以使用任何验证特性对该属性进行批注，然后请求生成的资源库触发对该属性的验证。 可以使用 NotifyDataErrorInfo 特性实现此目的：
```csharp
[ObservableProperty]
[NotifyDataErrorInfo]
[Required]
[MinLength(2)] // Any other validation attributes too...
private string? name;
等效于：
public string? Name
{
    get => name;
    set
    {
        if (SetProperty(ref name, value))
        {
            ValidateProperty(value, "Value2");
        }
    }
}
```
生成的 ValidateProperty 调用将验证属性并更新 ObservableValidator 对象的状态，以便 UI 组件可以对其做出反应，并相应地显示任何验证错误。

### 发送通知消息
如果在继承自 ObservableRecipient 的类型中声明属性，则可以使用 NotifyPropertyChangedRecipients 特性指示生成器还要插入代码，以针对属性更改发送一则说明属性已更改的消息。 这样，注册的接收者便可以动态响应更改。 也就是说，应考虑以下代码：

```csharp
[ObservableProperty]
[NotifyPropertyChangedRecipients]
private string? name;
等效于
public string? Name
{
    get => name;
    set
    {
        string? oldValue = name;

        if (SetProperty(ref name, value))
        {
            Broadcast(oldValue, value);
        }
    }
}
```
生成的 Broadcast 调用将使用当前 viewmodel 中正在使用的 IMessenger 实例向所有注册订阅者发送新的 PropertyChangedMessage<T>。

使用场景
```csharp
1. 跨 ViewModel 通信
// ViewModelA 修改 Name
Name = "NewValue"; 

// ViewModelB 监听变更
Messenger.Register<PropertyChangedMessage<string>>(
    this,
    (r, msg) => 
    {
        if (msg.PropertyName == "Name")
        {
            Console.WriteLine($"Name changed from {msg.OldValue} to {msg.NewValue}");
        }
    });

2. 多窗口同步数据
// 主窗口修改数据
SelectedItem = newItem;

// 详情窗口自动更新显示
Messenger.Register<PropertyChangedMessage<Item>>(
    this,
    (r, msg) => 
    {
        if (msg.PropertyName == "SelectedItem")
        {
            LoadDetails(msg.NewValue);
        }
    });
```


### 添加自定义属性（没有看懂）
在某些情况下，在生成的属性上添加一些自定义属性可能会很有用。 为此，你只需在带注释的字段上使用属性列表中的 [property: ] 目标，MVVM Toolkit 就会自动将这些属性转发到生成的属性。

例如，请考虑如下所示的字段：
```csharp
[ObservableProperty]
[property: JsonRequired]
[property: JsonPropertyName("name")]
private string? username;
```
这将生成一个 Username 属性，其中包含 [JsonRequired] 和 [JsonPropertyName("name")] 这两个属性。 可根据需要使用任意多个针对属性的属性列表，所有这些属性列表都将转发到生成的属性。

1. 序列化控制

    使用 [JsonPropertyName] 指定 JSON 字段命名
    用 [JsonRequired] 确保属性必须参与序列化
    例如：生成的 Username 属性在序列化时会变成 JSON 中的 "name" 字段

2. 数据验证

    添加验证特性如 [Required]、[Range] 到生成的属性
    与 ASP.NET Core 模型验证或客户端验证框架配合使用
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

