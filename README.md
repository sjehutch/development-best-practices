MVVM Cross 

Solution Components

For the DL Xamarin projects we will use the MvvmCross framework. This framework acts as a helper to build a cross-platform application using the MVVM pattern. For more information about MvvmCross:
* MvvmCross Overview 
* MvvmCross App Architecture 
* IoC /DI 
* Customize MvvmCross Behavior  Very useful!!
The projects within the Xamarin solution are:
* Data: Services and Models needed to access data.
* Core PCL: Portable class library that includes all the ViewModels and UI logic code.
    * The core project naming: <SolutionName>.Core.
    * Contains App.cs: Responsible for registering any types and services needed. Defines which ViewModel to display when app is first started.
* iOS project: Native Xamarin iOS project that holds all of the Views, Templates, Data Converters and other UI related files as well as the needed services implementations for this platform.
    * iOS project naming: <SolutionName>.iOS.
    * Contains Setup.cs: Responsible for ‘bootstraping’ MvvmCross, your ‘core’ and your ‘UI’.
* Test project: Contains Unit Tests for the ViewModels done with NUnit and Moq.
    * The Test project naming: <SolutionName>.Test.
ViewModels Guidelines

Here we include the guidelines for writing ViewModels for the Xamarin application.

*Note: these simple examples are not following the ViewModel region organization guidelines noted previously.
Properties

Are used for data-binding between View and ViewModel.

Example:
```
private string _username;
public string Username
{
    get => _username; 
    set => SetProperty(ref _username, value);
}
```
Commands

Allow View events to call ViewModel actions.
```
public ICommand ResetCommand => new MvxCommand(Reset);

private void Reset()
{
    Name = string.Empty;
}
```
ViewModel LifeCycle

The ViewModel lifecyce consists of:
1. Construction, using IoC for Dependency Injection:
   ```
   public class DetailViewModel : MvxViewModel
    {
        readonly IDetailRepository _repository;
    
        public DetailViewModel(IDetailRepository repository)
        {
            _repository = repository;
        }
    }
    ```
2. Initialize if receiving a navigation parameter, also see Navigation section for more.
   ```
   public async Task Initialize(MyObject parameter)
    {
        //Do something with parameter
    }
    ```
3. Additionally, we have the following methods coupled to the View lifecycle available from the ViewModel:
    void Appearing();

    void Appeared();

    void Disappearing();

    void Disappeared();
 
For more detailed documentation on this:

### View Lifecycle and ViewModel Lifecycle

#### Navigation
For Navigation we use the MvvmCross Navigation Service (available in Mvx 5.x and higher). The full documentation for the navigation service can be found at: MvvmCross Navigation
For simple ViewModel to ViewModel navigation, with no parameters, use:
   _navigationService.Navigate<NextViewModel>(); 


To pass a parameter to another ViewModel:
```
public class MyViewModel : MvxViewModel
{
    readonly IMvxNavigationService _navigationService;
    public MyViewModel(IMvxNavigationService navigationService)
    {
        _navigationService = navigationService;
    }
```

    public async Task SomeMethod()
    {
        _navigationService.Navigate<NextViewModel, MyObject>(new MyObject());
    }
}
```
public class NextViewModel : MvxViewModel<MyObject>
{
    public async Task Initialize(MyObject parameter)
    {
        //Do something with parameter
    }
}
```
Services

To write services for our application we follow this pattern. Create an interface and class that will implement this functionality as needed.
```
public interface IExampleService
{
    string Request();
}

public class ExampleService : IExampleService
{
    public string Request()
    {
        return "Hello World";
    }
}
```
For Platform-Specific code implementations, we must do the following:
First, we declare our Interface which can be injected into the ViewModel:
```
public interface IScreenSize
{
    double Height { get; }
    double Width { get; }
}
```
```
public class MyViewModel : MvxViewModel
{
    readonly IScreenSize _screenSize;

    public MyViewModel(IScreenSize screenSize)
    {
        _screenSize = screenSize;
    }

    public double Ratio
    {
        get { return (_screenSize.Width / _screenSize.Height); }
    }
}
```
We provide the platform-specific implementation in the platform project:
```
public class ScreenSizeIos : IScreenSize
{
    public double Height => 800.0; 
    public double Width => 480.0; 
}
```
Finally register the implementation in the UI project’s Setup.cs (iOS), like so:
```
protected override void InitializeFirstChance()
{
    Mvx.RegisterSingleton<IScreenSize>(new ScreenSizeIos());
    base.InitializeFirstChance();
}
```
Custom App Start

We configure the ViewModel for IMvxAppStart inside the App.cs in the Core PCL project. There might be an advanced scenario in which we need a custom app start e.g. launching the app from a notification. We can implement this in App.cs like so:
```
public class CustomAppStart : MvxNavigatingObject, IMvxAppStart
{
    public void Start(object hint = null)
    {
        var auth = Mvx.Resolve<IAuth>();
        if (auth.Check())
        {
            _navigationService.Navigate<HomeViewModel>();
        }
        else
        {
            _navigationService.Navigate<LoginViewModel>();
        }
    }
}
```
Then register this custom start:
```
RegisterAppStart(new CustomAppStart());
```
For more information on this and other custom scenarios: Customize App
