# 资料
1. [代码](https://github.com/PrismLibrary/Prism)
2. [官方sample](https://github.com/PrismLibrary/Prism-Samples-Wpf)
3. [官方Doc](https://docs.prismlibrary.com/docs/)
4. [Sample博客解释](https://www.cnblogs.com/cbaa/category/1984777.html?page=2)
   

## 1.BootstrapperShell

App.xaml.cs:

```csharp
protected override void OnStartup(StartupEventArgs e)
        {
            base.OnStartup(e);

            var bootstrapper = new Bootstrapper();
            bootstrapper.Run();
        }
```
上面创建一个bootstrapper，并启动了它。

bootstrapper又是什么呢？
```csharp
class Bootstrapper : PrismBootstrapper
    {
        protected override DependencyObject CreateShell()
        {
            return Container.Resolve<MainWindow>();
        }
 
        protected override void RegisterTypes(IContainerRegistry containerRegistry)
        {
             
        }
    }
```
旧版本 Prism（如 Prism 7.x 及之前）中，Bootstrapper 的主要职责包括：初始化依赖注入容器、配置模块化架构、创建主窗口（Shell）等核心任务。需要手动创建 Bootstrapper 实例并调用 Run()。

从 Prism 8 开始，框架进行了简化，Bootstrapper 的功能被合并到 App.xaml.cs 中，也就是说，bootstrapper已经合并到app.xaml.cs中了。
```csharp
public partial class App : PrismApplication // 继承自 PrismApplication
{
    protected override Window CreateShell()
    {
        return Container.Resolve<MainWindow>();
    }

    protected override void RegisterTypes(IContainerRegistry containerRegistry)
    {
        // 注册自定义类型或服务
    }
}
```
## 新旧版本代码对比

| 步骤         | 旧版本（Prism 7.x）                          | 新版本（Prism 8+）                          |
|--------------|---------------------------------------------|---------------------------------------------|
| 入口点       | App.xaml.cs 中手动创建 Bootstrapper 并启动。 | 直接继承 PrismApplication，自动处理启动逻辑。 |
| 主窗口创建   | 在 Bootstrapper.CreateShell() 中实现。       | 在 App.CreateShell() 中实现。               |
| 依赖注册     | 在 Bootstrapper.RegisterTypes() 中实现。     | 在 App.RegisterTypes() 中实现。             |
| 模块化配置   | 通过 IModuleCatalog 配置模块。               | 类似，但集成到 App 类中。                   |

## 2.Region

```csharp
public partial class App : PrismApplication
{
    protected override Window CreateShell()
    {
        return Container.Resolve<MainWindow>();
    }

    protected override void RegisterTypes(IContainerRegistry containerRegistry)
    {
        
    }
}
```
直接解析了MainWindow这个文件，而这个文件，现在放在Views的文件夹下。
  <ContentControl prism:RegionManager.RegionName="ContentRegion" />
定义了一个控件，控件里放了一下Region，命名为ContentRegion

## 3. 自定义Region

在例2中，我们使用了一个Region

<ContentControl prism:RegionManager.RegionName="ContentRegion" />

上面使用了ContentControl，但在prism中，不是每个控件都能定义为Region的。比如Stack Panel就不行，Grid也不行。如果要用到这样的控件作为区域，就需要定义一个RegionAdapter。
下面就以Stack Panel为例：
```csharp

  public class StackPanelRegionAdapter : RegionAdapterBase<StackPanel>
    {
        public StackPanelRegionAdapter(IRegionBehaviorFactory regionBehaviorFactory)
            : base(regionBehaviorFactory)
        {

        }

        protected override void Adapt(IRegion region, StackPanel regionTarget)
        {
            region.Views.CollectionChanged += (s, e) =>
            {
                if (e.Action == System.Collections.Specialized.NotifyCollectionChangedAction.Add)
                {
                    foreach (FrameworkElement element in e.NewItems)
                    {
                        regionTarget.Children.Add(element);
                    }
                }

                //handle remove
            };
        }

        protected override IRegion CreateRegion()
        {
            return new AllActiveRegion();
        }
    }
```
有了Adapter，就可以在Stack Panel中定义Region
```xml
 <StackPanel prism:RegionManager.RegionName="ContentRegion" />
```
## 4. View Discovery
前三节算是弄明白了Region是什么，但是定义了区域，怎样向区域中添加内容呢？内容是UserControl，即ViewA。

添加内容的方式有2种，一种叫View Discovery，一种叫View Injection。

第一种，先定义一个ViewA，内容很简单，只加一行字：
```xml
 <TextBlock Text="View A" FontSize="38" />
```
然后我们在主页面上添加一个Region，如同前面反复看到好几次的：
```xml
  <ContentControl prism:RegionManager.RegionName="ContentRegion" />
```
现在的问题就是，怎样把ViewA的内容显示在ContentRegion中呢？ 这里用的是发现：

```csharp
public partial class MainWindow : Window
    {
        public MainWindow(IRegionManager regionManager)
        {
            InitializeComponent();
            //view discovery
            regionManager.RegisterViewWithRegion("ContentRegion", typeof(ViewA));
        }
    }
```

这里是在RegionManager 中进行注册，注册的内容是区域和视图的关联关系。一个区域可以注册多个视图，但是显示哪个，就由后面的激活决定。后面的样例应该是有的。

## 5. View Injection
这里稍微复杂了点，定义视图A的过程是一样的：
但是与前面不同的是，需要点击按钮之后才能显示，这个在MainWindow的后台文件中体现：
 

```csharp
public partial class MainWindow : Window
    {
        IContainerExtension _container;
        IRegionManager _regionManager;

        public MainWindow(IContainerExtension container, IRegionManager regionManager)
        {
            InitializeComponent();
            _container = container;
            _regionManager = regionManager;
        }

        private void Button_Click(object sender, RoutedEventArgs e)
        {
            var view = _container.Resolve<ViewA>();
            var region = _regionManager.Regions["ContentRegion"];
            region.Add(view);
        }
    }
```

这里所称的注入，首先是在构造函数中注入了容器和区域管理器，然后在Button的后台中，先生成View的实例，再找到Region，最后向Region中Add（View）。

由些看来，View Injection的方法更透明，也就是说这个过程更可控，在下面的情况下，建议使用Injection。

Explicit or programmatic control over when a view is created and displayed, or when you need to remove a view from a region, for example, as a result of application logic.
To display multiple instances of the same views into a region, where each view instance is bound to different data.
To control which instance of a region a view is added (for example, if you want to add customer detail view to a specific customer detail region). Note that
this scenario requires scoped regions described later in this topic.


## 6 Activation Deactivation
例5中刚说到视图精确控制，这次说明这样的灵活控制是怎样做的，显示或不显示，或切换视图。
主页上显示了主按钮和一个ContentControl

```xml
<DockPanel LastChildFill="True">
        <StackPanel>
            <Button Content="Activate ViewA" Click="Button_Click"/>
            <Button Content="Deactivate ViewA" Click="Button_Click_1"/>
            <Button Content="Activate ViewB" Click="Button_Click_2"/>
            <Button Content="Deactivate ViewB" Click="Button_Click_3"/>
        </StackPanel>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" HorizontalAlignment="Center" VerticalAlignment="Center" />
    </DockPanel>
```

然后在后台代码中，对显示内容进行了精确控制

```csharp
   public partial class MainWindow : Window
    {
        IContainerExtension _container;
        IRegionManager _regionManager;
        IRegion _region;

        ViewA _viewA;
        ViewB _viewB;

        public MainWindow(IContainerExtension container, IRegionManager regionManager)
        {
            InitializeComponent();
            _container = container;
            _regionManager = regionManager;

            this.Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            _viewA = _container.Resolve<ViewA>();
            _viewB = _container.Resolve<ViewB>();

            _region = _regionManager.Regions["ContentRegion"];

            _region.Add(_viewA);
            _region.Add(_viewB);
        }

        private void Button_Click(object sender, RoutedEventArgs e)
        {
            //activate view a
            _region.Activate(_viewA);
        }

        private void Button_Click_1(object sender, RoutedEventArgs e)
        {
            //deactivate view a
            _region.Deactivate(_viewA);
        }

        private void Button_Click_2(object sender, RoutedEventArgs e)
        {
            //activate view b
            _region.Activate(_viewB);
        }

        private void Button_Click_3(object sender, RoutedEventArgs e)
        {
            //deactivate view b
            _region.Deactivate(_viewB);
        }
    }
```
这里出现的新概念就是Activate 和DeActivate。控制显示和不显示。

## 7. Modules App.Config
在项目中添加模块化文件。模块文件怎样在主项目中注册。本例 说明方式一，使用了App.config文件。

例如MouduleA中定义了一些UserControl,ViewA,需要在其他模块使用，需要在模块A中定义导出界面：

Prisim Sample 7 Modules App.Config
在项目中添加模块化文件。模块文件怎样在主项目中注册。本例 说明方式一，使用了App.config文件。


```csharp
namespace ModuleA
{
    public class ModuleAModule : IModule
    {
        public void OnInitialized(IContainerProvider containerProvider)
        {
            var regionManager = containerProvider.Resolve<IRegionManager>();
            regionManager.RegisterViewWithRegion("ContentRegion", typeof(ViewA));
        }

        public void RegisterTypes(IContainerRegistry containerRegistry)
        {
            
        }
    }
}
```
在其他模块中调用使用配置文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <section name="modules" type="Prism.Modularity.ModulesConfigurationSection, Prism.Wpf" />
  </configSections>
  <startup>
  </startup>
  <modules>
    <module assemblyFile="ModuleA.dll" moduleType="ModuleA.ModuleAModule, ModuleA, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="ModuleAModule" startupLoaded="True" />
  </modules>
</configuration>
```
直接使用
```xml
<Window x:Class="Modules.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:prism="http://prismlibrary.com/"
        Title="Shell" Height="350" Width="525">
    <Grid>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" />
    </Grid>
</Window>
```

### 7 Modules Code
也可以使用代码调用
```csharp
namespace Modules
{
    public partial class App : PrismApplication
    {
        protected override Window CreateShell()
        {
            return Container.Resolve<MainWindow>();
        }

        protected override void RegisterTypes(IContainerRegistry containerRegistry)
        {

        }

        protected override void ConfigureModuleCatalog(IModuleCatalog moduleCatalog)
        {
            moduleCatalog.AddModule<ModuleA.ModuleAModule>();
        }
    }
}
```

### 7 Modules Directory
这种方式用扫描目录的方式来增加模块，具备最大的灵活性

仍然在App.xaml.cs中增加了以下代码
```csharp

protected override IModuleCatalog CreateModuleCatalog()
        {
            return new DirectoryModuleCatalog() { ModulePath = @".\Modules" };
        }
```
这意思是到Modules这文件夹下扫描所有的模块，扫到的就注册进来。
但是模块项目的生成输出必须指定到这个文件夹下，所要在项目的生成事件中使用以下脚本：

```cmd
xcopy "$(TargetDir)$(TargetName)*$(TargetExt)" "$(SolutionDir)$(SolutionName)\bin\Debug\netcoreapp3.1\Modules\" /Y /S
```
在测试运行之前，请先生成整个项目，否则外部模块还没有复制完成，找不到模块。

### 7 Modules LoadManual
这种模块是手动载入的，需要的时候手动加载。

在app.xaml.cs中注册为按需加载，代码

```csharp
protected override void ConfigureModuleCatalog(IModuleCatalog moduleCatalog)
        {
            var moduleAType = typeof(ModuleAModule);
            moduleCatalog.AddModule(new ModuleInfo()
            {
                ModuleName = moduleAType.Name,
                ModuleType = moduleAType.AssemblyQualifiedName,
                InitializationMode = InitializationMode.OnDemand
            });
        }
```
其中最后行是指定手动加载。

在加载时，例中用了一个按钮来手动加载
```csharp
 public partial class MainWindow : Window
    {
        IModuleManager _moduleManager;

        public MainWindow(IModuleManager moduleManager)
        {
            InitializeComponent();
            _moduleManager = moduleManager;
        }

        private void Button_Click(object sender, RoutedEventArgs e)
        {
            _moduleManager.LoadModule("ModuleAModule");
        }
    }
```
 
## 7 Module xaml
这一节使用xaml标记甚为不解。
本节注册module 的方式同directory一节很类似。在那一节中，用工厂方法创建一模块目录：
```csharp
protected override IModuleCatalog CreateModuleCatalog()
        {
            return new DirectoryModuleCatalog() { ModulePath = @".\Modules" };
        }
这里用了同一个方法，返回也相同，但是实现不同。

protected override IModuleCatalog CreateModuleCatalog()
        {
            return new XamlModuleCatalog(new Uri("/Modules;component/ModuleCatalog.xaml", UriKind.Relative));
        }
```csharp
这里也指定了目录，同时指定了一个用xaml写的配置文件，配置文件的内容也似曾相识：
ModuleCatalog.xaml文件
```xml
<m:ModuleCatalog xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:m="clr-namespace:Prism.Modularity;assembly=Prism.Wpf">

    <m:ModuleInfo ModuleName="ModuleAModule" 
                  ModuleType="ModuleA.ModuleAModule, ModuleA, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />

</m:ModuleCatalog>
```
注意上面代码中/Modules不是模块输出目录，而是项目目录。

## 8. ViewModelLocator 跳过简单（绑定后缀ViewModel和View的文件）
```xml
prism:ViewModelLocator.AutoWireViewModel="True"
```
## 9 ChangeConvention
上个例子跳过了ViewModelLocator，因是采用约定的方式最为方便。
如果有人要修改约定，自定义view和viewModel的默认自动定位方式，怎么办呢？

在app.xaml.cs重写以下方法：

```csharp
protected override void ConfigureViewModelLocator()
        {
            base.ConfigureViewModelLocator();

            ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver((viewType) =>
            {
                var viewName = viewType.FullName;
                var viewAssemblyName = viewType.GetTypeInfo().Assembly.FullName;
               var viewModelName = $"{viewName}ViewModel, {viewAssemblyName}";
               
                return Type.GetType(viewModelName);
            });
        }
```
其中，{viewname}ViewModel这里直接在同一目录下将view名称+ViewModel的文件名默认关联。

如果要修改路径或文件名，修改这个变量即可。
## 10-CustomRegistrations
作用同上节，这里是用修改注册的方式自定义View和ViewModel的关联。

```csharp
protected override void ConfigureViewModelLocator()
        {
            base.ConfigureViewModelLocator();

            // type / type
            //ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), typeof(CustomViewModel));

            // type / factory
            //ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), () => Container.Resolve<CustomViewModel>());

            // generic factory
            //ViewModelLocationProvider.Register<MainWindow>(() => Container.Resolve<CustomViewModel>());

            // generic type
            ViewModelLocationProvider.Register<MainWindow, CustomViewModel>();
        }
```
## 11-UsingDelegateCommands
本例的知识点，全在ViewModel中，不太好理解的，为何button的状态与IsEnable的属性相关，两者又没有直接绑定，
看代码：



```csharp
  public class MainWindowViewModel : BindableBase
  {
      private bool _isEnabled;
      public bool IsEnabled
      {
          get { return _isEnabled; }
          set
          {
              SetProperty(ref _isEnabled, value);
              ExecuteDelegateCommand.RaiseCanExecuteChanged();
          }
      }

      private string _updateText;
      public string UpdateText
      {
          get { return _updateText; }
          set { SetProperty(ref _updateText, value); }
      }


      public DelegateCommand ExecuteDelegateCommand { get; private set; }

      public DelegateCommand<string> ExecuteGenericDelegateCommand { get; private set; }        

      public DelegateCommand DelegateCommandObservesProperty { get; private set; }

      public DelegateCommand DelegateCommandObservesCanExecute { get; private set; }


      public MainWindowViewModel()
      {
          ExecuteDelegateCommand = new DelegateCommand(Execute, CanExecute);

          DelegateCommandObservesProperty = new DelegateCommand(Execute, CanExecute).ObservesProperty(() => IsEnabled);

          DelegateCommandObservesCanExecute = new DelegateCommand(Execute).ObservesCanExecute(() => IsEnabled);

          ExecuteGenericDelegateCommand = new DelegateCommand<string>(ExecuteGeneric).ObservesCanExecute(() => IsEnabled);
      }

      private void Execute()
      {
          UpdateText = $"Updated: {DateTime.Now}";
      }

      private void ExecuteGeneric(string parameter)
      {
          UpdateText = parameter;
      }

      private bool CanExecute()
      {
          return IsEnabled;
      }
  }
```

构造函数中建立了4个命令。

其中，33行这个是原始的，CanExecute要靠10行的代码来通知改变。后面的三个，是使用语法糖

ObservesCanExecute(() => IsEnabled)实现了自动通知。
这里可以试一下，注释掉 ExecuteDelegateCommand.RaiseCanExecuteChanged();然后运行的话，勾选Enabled，其他三个命令都改变状态，而第一个按钮就没有收到变更通知。

39行构造的命令为泛型命令，支持一个泛型的参数。参数在xaml中使用CommandParammeter提供。


## 12. UsingCompositeCommands
这个案例适用于插件和主程序的交付。

本例中，主页是一个按钮，绑定了一个复合命令，然后下面一个TabControl

```xml
<Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <Button Content="Save" Margin="10" Command="{Binding ApplicationCommands.SaveCommand}"/>

        <TabControl Grid.Row="1" Margin="10" prism:RegionManager.RegionName="ContentRegion" />
    </Grid>
```
然后在另外一个模块构建一个视图及ViewModel，VM中建立一个命令，用于更新TabView的内容。关键是，这个命令注册成为复合命令的子命令。复合命令也是一个模块。

```csharp
  public TabViewModel(IApplicationCommands applicationCommands)
        {
            _applicationCommands = applicationCommands;

            UpdateCommand = new DelegateCommand(Update).ObservesCanExecute(() => CanUpdate);

            _applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
        }
```
最后，将TabView 的三个实例依次添加到ContentRegion

```csharp
 public void OnInitialized(IContainerProvider containerProvider)
        {
            var regionManager = containerProvider.Resolve<IRegionManager>();
            IRegion region = regionManager.Regions["ContentRegion"];

            var tabA = containerProvider.Resolve<TabView>();
            SetTitle(tabA, "Tab A");
            region.Add(tabA);

            var tabB = containerProvider.Resolve<TabView>();
            SetTitle(tabB, "Tab B");
            region.Add(tabB);

            var tabC = containerProvider.Resolve<TabView>();
            SetTitle(tabC, "Tab C");
            region.Add(tabC);
        }
```
然后我们看到，在每个TabView中的命令，只更新自己的View。而复合命令，更新了所有的View，虽然只注册了一个命令。但这个命令应用于多个实例的时候，在每个实例上都会起使用。

## 13-IActiveAwareCommands
本例和12的唯一区别，仅仅是在ViewModel中增加了一个IActiveAware，这决定了只有在Acitve状态的视图中才会执行自己ViewModel中的命令。 
有点看不太懂了，主程序会判断当前界面是否是焦点，会自动订阅模块的IsActiveChanged事件。


## 14-UsingEventAggregator
这次是事件聚合器的应用。两个模块分别在主程序中显示，并且通过事件聚合实现信息交互。

### 第一步：定义事件聚合器

首先定义一个事件聚合器类，这个类应该是一个可访问的公共区域。以下代码展示了如何在一个核心项目中定义事件聚合器：

```csharp
using Prism.Events;

namespace UsingEventAggregator.Core
{
    public class MessageSentEvent : PubSubEvent<string>
    {
    }
}
```
这个类继承了 PubSubEvent<T>，其中 T 可以是任何类型。

### 第二步：发送消息
在 ModuleA 中定义一个消息发送界面：
```xml
<StackPanel>
    <TextBox Text="{Binding Message}" Margin="5"/>
    <Button Command="{Binding SendMessageCommand}" Content="Send Message" Margin="5"/>
</StackPanel>
```

```csharp
public class MessageViewModel : BindableBase
{
    IEventAggregator _ea;

    private string _message = "Message to Send";
    public string Message
    {
        get { return _message; }
        set { SetProperty(ref _message, value); }
    }

    public DelegateCommand SendMessageCommand { get; private set; }

    public MessageViewModel(IEventAggregator ea)
    {
        _ea = ea;
        SendMessageCommand = new DelegateCommand(SendMessage);
    }

    private void SendMessage()
    {
        _ea.GetEvent<MessageSentEvent>().Publish(Message);
    }
}
```
### 第三步：接收消息
在 ModuleB 中定义一个接收消息的界面：
```xml
<Grid>
    <ListBox ItemsSource="{Binding Messages}" />
</Grid>
```
```csharp
public class MessageListViewModel : BindableBase
{
    readonly IEventAggregator _ea;

    private ObservableCollection<string> _messages;
    public ObservableCollection<string> Messages
    {
        get => _messages;
        set => SetProperty(ref _messages, value);
    }

    public MessageListViewModel(IEventAggregator ea)
    {
        _ea = ea;
        Messages = new ObservableCollection<string>();

        _ea.GetEvent<MessageSentEvent>().Subscribe(MessageReceived);
    }

    private void MessageReceived(string message)
    {
        Messages.Add(message);
    }
}
```


## 15-FilteringEvents
例14演示了怎样事件聚合器怎样发布与接收信息。

例15增加了一个事件的过滤功能，即设定一个条件，符合的才接收。

```csharp
 _ea.GetEvent<MessageSentEvent>().Subscribe(MessageReceived, ThreadOption.PublisherThread, false, (filter) => filter.Contains("Brian"));
 ```
上面的注册时，只有消息中包括"Brian"的的消息才会接收。


## 16-RegionContext
主程序RegionContext中有一个 **RegionName="ContentRegion"**，ModuleA中在PersonList视图中又定义了一个**RegionName="PersonDetailsRegion"**，并且指定了该区域的**RegionContext="{Binding SelectedItem}**，这样区域中有了Data。
相当于ModuleA 中的主页面给子页面提供了数据。


本例的核心是RegionContext，意思是一个区域的上下文。但与DataContext似乎并不相同。

RegionContext与DataContext的区别：

DataContext：视图的本地数据上下文（通常绑定ViewModel）
RegionContext：区域间的共享数据通道（传递选中项、参数等）

典型场景

### 主窗体布局

在主窗体上只有一个区域：

```xml
<Grid>
    <ContentControl prism:RegionManager.RegionName="ContentRegion" />
</Grid>
```
在此区域上注册一个视图，该视图中包含一个子区域：

```xml
<Grid x:Name="LayoutRoot" Background="White" Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="100"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>

    <ListBox x:Name="_listOfPeople" ItemsSource="{Binding People}" DisplayMemberPath="FirstName"/>
    <ContentControl Grid.Row="1" Margin="10"
                    prism:RegionManager.RegionName="PersonDetailsRegion"
                    prism:RegionManager.RegionContext="{Binding SelectedItem, ElementName=_listOfPeople}"/>
</Grid>
```
上部是一个列表，下面插入一个区域，用来显示选中项的细节。RegionContext 绑定到 SelectedItem。

但是定义了一个RegionContext，绑定到SelectedItem。如果这是上下文，这个子视图还有自己的ViewModel怎么办呢？

ViewModel 绑定
子视图的 ViewModel 绑定 SelectedPerson：
```csharp
 public class PersonDetailViewModel : BindableBase
    {
        private Person _selectedPerson;
        public Person SelectedPerson
        {
            get { return _selectedPerson; }
            set { SetProperty(ref _selectedPerson, value); }
        }

        public PersonDetailViewModel()
        {

        }
    }
```
系统决定了View 和ViewModel的自动绑定了，Detail绑定的是SelectedPerson，需要实现一个转换。


View 后台代码
在 View 的后台代码中实现 RegionContext 的转换：
```csharp
public partial class PersonDetail : UserControl
    {
        public PersonDetail()
        {
            InitializeComponent();
            RegionContext.GetObservableContext(this).PropertyChanged += PersonDetail_PropertyChanged;
        }

        private void PersonDetail_PropertyChanged(object sender, System.ComponentModel.PropertyChangedEventArgs e)
        {
            var context = (ObservableObject<object>)sender;
            var selectedPerson = (Person)context.Value;
            ((PersonDetailViewModel) DataContext).SelectedPerson = selectedPerson;
        }
```
这段代码的意思是，Detail 绑定了 SelectedItem，如果 SelectedItem 发生了变化，就将变化的值更新到 ViewModel 中。

## 17-BasicRegionNavigation
本例是基础的导航应用，简单过了。

在窗口中布局了2个按钮，分别导航到子模块的两个区域。
```csharp
public DelegateCommand<string> NavigateCommand { get; private set; }

        public MainWindowViewModel(IRegionManager regionManager)
        {
            _regionManager = regionManager;

            NavigateCommand = new DelegateCommand<string>(Navigate);
        }

        private void Navigate(string navigatePath)
        {
            if (navigatePath != null)
                _regionManager.RequestNavigate("ContentRegion", navigatePath);
        }
```


## 18-NavigationCallback
同17相比，在导航方法中增加了回调函数

```csharp
private void Navigate(string navigatePath)
        {
            if (navigatePath != null)
                _regionManager.RequestNavigate("ContentRegion", navigatePath, NavigationComplete);
        }

        private void NavigationComplete(NavigationResult result)
        {
            System.Windows.MessageBox.Show(String.Format("Navigation to {0} complete. ", result.Context.Uri));
        }
```

## 19-NavigationParticipation
意思是对导航过程的参与，触发事件，类似离开导航目标和进入导航的回调

在VM中，增加一个接口 ，然后实现导航事件
```csharp
public class ViewAViewModel : BindableBase, INavigationAware
    {
      ……

        private int _pageViews;
        public int PageViews
        {
            get { return _pageViews; }
            set { SetProperty(ref _pageViews, value); }
        }

        public ViewAViewModel()
        {

        }

        public void OnNavigatedTo(NavigationContext navigationContext)
        {
            PageViews++;
        }

        public bool IsNavigationTarget(NavigationContext navigationContext)
        {
            return true;
        }

        public void OnNavigatedFrom(NavigationContext navigationContext)
        {
            
        }
    }
```
需要注意的是主程序的RegionName对应的类型是TabControl，每次导航都会add view.
```xml
        <TabControl prism:RegionManager.RegionName="ContentRegion" Margin="5"  />

```
### INavigationAware 接口详解

| 方法              | 触发时机                 | 典型应用场景                           |
|-------------------|--------------------------|----------------------------------------|
| OnNavigatedTo     | 导航到当前视图时触发     | 初始化数据、更新状态、埋点统计         |
| IsNavigationTarget| 决定是否复用当前实例     | 控制实例复用策略（如根据参数过滤）     |
| OnNavigatedFrom   | 离开当前视图时触发       | 资源释放、取消异步操作、保存临时状态   |
---
## 20-NavigateToExistingViews
`                                                                           
上一个例子介绍了INavigationAware中的OnNavitationTo，这次是第二个实现函数。

IsNavitationTarget，这个名字有点误导，真实的作用是，当从其它页面导航至本页面的时候，首先会调用IsNavigationTarget，IsNavigationTarget返回一个bool值，是重复使用这个视图的实例还是再创建一个。true就重建，false就用以前的实例。

代码中，VM中设置了一个变量，pageView，用来记录进入视图的次数。而在IsNavigationTarget方法中，return PageViews / 3 != 1，即仅当pageViews为3的时候创建新实例。

## 21-PassingParameters
这个例子是说明导航中传递参数，类似Asp.net中实现。

例子的模板，是例16中使用regionContext实现过的。在例16中，数据是在RegioContext中解析。

本例中，
```csharp
public PersonListViewModel(IRegionManager regionManager)
        {
            _regionManager = regionManager;

            PersonSelectedCommand = new DelegateCommand<Person>(PersonSelected);
            CreatePeople();
        }

        private void PersonSelected(Person person)
        {
            var parameters = new NavigationParameters();
            parameters.Add("person", person);

            if (person != null)
                _regionManager.RequestNavigate("PersonDetailsRegion", "PersonDetail", parameters);
        }
```
本例中，导航命令中的使用了参数，而参数是一个字典形式。就是解析参数包（类似viewbag）并获得数据。

## 22-ConfirmCancelNavigation
导航到一个视图，如果在离开这个视图时需要确认，在VM中实现以下接口

```csharp
public class ViewAViewModel : BindableBase, IConfirmNavigationRequest
    {
        public ViewAViewModel()
        {

        }

        public void ConfirmNavigationRequest(NavigationContext navigationContext, Action<bool> continuationCallback)
        {
            bool result = true;

            if (MessageBox.Show("Do you to navigate?", "Navigate?", MessageBoxButton.YesNo) == MessageBoxResult.No)
                result = false;

            continuationCallback(result);
        }

        public bool IsNavigationTarget(NavigationContext navigationContext)
        {
            return true;
        }

        public void OnNavigatedFrom(NavigationContext navigationContext)
        {
            
        }

        public void OnNavigatedTo(NavigationContext navigationContext)
        {
            
        }
    }
```
---

 ## 23-RegionMemberLifetime
没有看懂。
在导航中跳转时，视图是缓存的。如果要求某视图在离开后就销毁，需要实现下面的接口

```csharp
public class ViewAViewModel : BindableBase, INavigationAware, IRegionMemberLifetime
    {
        
        public bool KeepAlive=>false;
        
}

public MainWindowViewModel(IRegionManager regionManager)
{
    _regionManager = regionManager;
    _regionManager.Regions.CollectionChanged += Regions_CollectionChanged;

    NavigateCommand = new DelegateCommand<string>(Navigate);
}
private void Regions_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
{
    if (e.Action == NotifyCollectionChangedAction.Add)
    {
        var region = (IRegion)e.NewItems[0];
        region.Views.CollectionChanged += Views_CollectionChanged;
    }
}

private void Views_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
{
    if (e.Action == NotifyCollectionChangedAction.Add)
    {
        Views.Add(e.NewItems[0].GetType().Name);
    }
    else if (e.Action == NotifyCollectionChangedAction.Remove)
    {
        Views.Remove(e.OldItems[0].GetType().Name);
    }
}

```
•	Regions_CollectionChanged 方法：当区域集合发生变化时（例如添加新的区域），订阅新添加区域的视图集合的 CollectionChanged 事件。

•	Views_CollectionChanged 方法：当视图集合发生变化时（例如添加或移除视图），更新 Views 集合。添加视图时，将视图的类型名称添加到 Views 集合中；移除视图时，从 Views 集合中移除视图的类型名称。


## 24-NavigationJournal
本例是在上一案例中导航参数的基础上增加了导航的历史记录功能，就是向前向后的功能。

导航本身很简单，以下代码就实现了：

```csharp
 public void OnNavigatedTo(NavigationContext navigationContext)
        {
            _journal = navigationContext.NavigationService.Journal;
 }
        
private void GoBack()
        {
            _journal.GoBack();
        }
```
在导航上下文中，NavigationContext就保存了导航服务，服务中包括了历史记录。只要调用相关方法就可以。但是有些视图是带有参数的，那么需要同样记得当前所用参数，然后一并重新打开。

---
## 25 没有找到代码
新版本中去掉了一些过重的功能

## 26 NotificationDialogViewModel

定义了 NotificationDialogViewModel 类，它是一个 WPF 应用程序中用于处理对话框逻辑的视图模型，

对话框（Dialog）通常是一个独立的窗口，常用来显示重要信息或要求用户进行确认、输入等操作。它往往是模态的，会暂时阻塞背后的窗口交互，直到用户关闭对话框。
UserControl 则是一个可复用的控件组件，可以嵌入在其他界面或窗口中，与主窗口同时交互，不会像对话框那样阻塞用户对其他界面的操作。它更适合创建在应用程序内部共享或重复使用的界面片段。

## 27 StylingDialog
没有看出和26有任何区别。

## 28 UsingCustomWindow
对话框（Dialog）采用指定的样式。、


## 29-InvokeCommandAction
本例是演示行为转命令的，事实上前面已经用到了。

```csharp

   <i:Interaction.Triggers>
                <!-- This event trigger will execute the action when the corresponding event is raised by the ListBox. -->
                <i:EventTrigger EventName="SelectionChanged">
                    <!-- This action will invoke the selected command in the view model and pass the parameters of the event to it. -->
                    <prism:InvokeCommandAction Command="{Binding SelectedCommand}" TriggerParameterPath="AddedItems" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
```

这个原因是，不是所有控件都可以直接绑定Command的，只有Button类，MenuItem类，ListBoxItem类可以。

但是这个例子就是ListBox啊，如果不用转命令而是直接绑定怎么办呢？

在 WPF 中，ListBox 这类控件本身没有直接支持 Command 属性的机制（如 Button 的 Click 事件对应 Command 属性），因此需要通过行为（Behaviors）或事件绑定机制（如 Prism 的 InvokeCommandAction）将事件转换为命令。不过，如果不想用事件转命令，也可以通过 属性绑定 + 属性变更通知 的方式实现类似效果。

方案 1：直接绑定 SelectedItem（推荐）
```xml
<ListBox 
    Grid.Row="1" 
    Margin="5" 
    ItemsSource="{Binding Items}"
    SelectedItem="{Binding SelectedItem, Mode=TwoWay}"  <!-- 关键绑定 -->
    SelectionMode="Single" />
```

方案 2：事件转命令
通过行为（Behaviors）将 SelectionChanged 事件转换为命令：
```xml
<ListBox 
    Grid.Row="1" 
    Margin="5" 
    ItemsSource="{Binding Items}" 
    SelectionMode="Single">
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="SelectionChanged">
            <prism:InvokeCommandAction 
                Command="{Binding SelectedCommand}" 
                TriggerParameterPath="AddedItems" />
        </i:EventTrigger>
    </i:Interaction.Triggers>
</ListBox>
```

## 事件转命令与直接绑定的对比

### 两种方案的对比

| 特性             | 方案 1（SelectedItem 绑定） | 方案 2（事件转命令）       |
|------------------|-----------------------------|----------------------------|
| 代码简洁性       | 更简洁，无需事件触发器      | 需要 XAML 事件绑定         |
| MVVM 纯度        | 更高（直接绑定属性）        | 需依赖行为库               |
| 参数传递灵活性   | 直接传递选中项对象          | 可传递事件参数             |
| 多选模式支持     | 需要绑定 SelectedItems      | 需处理 AddedItems          |
| 依赖库           | 无                          | 需 Prism/Behaviors         |

优先方案 1：如果只需要获取选中项并处理变更逻辑，直接绑定 SelectedItem 更简单高效。
选择方案 2：如果需要事件参数（如 AddedItems/RemovedItems）或需要兼容旧代码逻辑。
