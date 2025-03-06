# 资料
1. [代码](https://github.com/PrismLibrary/Prism)
2. [官方sample](https://github.com/PrismLibrary/Prism-Samples-Wpf)
3. [Sample博客解释](https://www.cnblogs.com/cbaa/category/1984777.html?page=2)
   

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
本例的知识点，全在ViewModel中，看代码：



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