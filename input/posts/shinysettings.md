Title: Settings in a New Light - Shiny Style
Image: images/shiny_logo.png
Published: 5/3/2019
Tags:
    - Xamarin
    - OSS
    - Shiny
---
<img src="images/shiny_logo.png" width="100" /> 

I know - Preferences & Settings are a dime a dozen these days, but I'm such a big advocated of decoupled software that I had to do things differently.  With Shiny, I had a chance to move one of the features that I loved from my ACR Settings plugin over and give it a good home in a DI universe of awesome!

You've probably seen code similar to this before:
```csharp

public static class AppSettings
{
    public static string ApiBaseUri
    {
        get => Settings.Get("ApiBaseUri");
        set => Settings.Set("ApiBaseUri", value);
    }
}
```

### Why is this not ideal?
Several reasons 
* It is not a testable piece of code - not that settings usually have to be testable, but they are often used in tests
* It creates some coupling (yes we can get around this though)
* It uses some boilerplate code
* It isn't necessarily fun to wire through your app
* You can't react to changes in your settings


### Settings in a different light
Often people associate the INotifyPropertyChanged interface the ViewModel interface.  It is, but it can be used for many other cool things.  Let's look at this in a different way:

```csharp
public class AppSettings : Shiny.NotifyPropertyChanged
{
    public AppSettings()
    {
        // values set here are like defaults and not pushed settings 
        // they are overwritten by the settings binding if they were set
        this.RememberText = "Hello"; 
    }


    string rememberText;
    public string RememberText 
    { 
        get => this.rememberText
        set => this.Set(ref this.rememberText, value);
    }


    bool isEnabled;
    public bool IsEnabled
    { 
        get => this.isEnabled;
        set => this.Set(ref this.isEnabled, value);
    }


    DateTime? lastUpdated;
    public DateTime? LastUpdated 
    { 
        get => this.lastUpdated;
        private set => this.Set(ref this.lastUpdated, value);
    }
}
```

Now, you can definitely argue that viewmodels contain their own set of boilerplate, but we've got so many tools to take care of things like this now a days such as ReactiveUI.Fody & PropertyChanged.Fody courtesy of Simon Cropp!

### Messaging in a new REACTIVE light:

Using the AppSettings class above, let's set that LastUpdated according to changes in the Remember Me

```csharp

// adding to the constructor above
public AppSettings()
{
    // whenever a property changes, update the date (and remember it to)
    this.WhenAnyProperty(x => x.IsEnabled).Subscribe(_ => this.LastUpdated = DateTime.Now);
    this.WhenAnyProperty(x => x.RememberText).Subscribe(_ => this.LastUpdated = DateTime.Now);
}


// to register this as a service with Shiny - add the following to your Startup

public class MyStartup : Shiny.ShinyStartup
{
    public override void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<AppSettings>();
    }
}
```
That magical little method WhenAnyProperty is from a bunch of RX addons with Shiny.Core.


### What about doing this on my ViewModel?

Sure, just make sure to unbind the ViewModel from settings when it is going away - Below is an example using Prism:

```csharp
public class MyViewModel : Prism.Mvvm.BindableBase, Prism.AppModel.IPageLifecycleAware
{
    readonly ISettings settings;


    public MyViewModel(ISettings settings)
    {
        this.settings = settings;
    }


    public void OnAppearing()
    {
        settings.Bind(this);
    }


    public void OnDisappearing()
    {
        settings.UnBind(this);
    }



    string rememberText;
    public string RememberText 
    { 
        get => this.rememberText
        set => this.SetProperty(ref this.rememberText, value);
    }


    bool isEnabled;
    public bool IsEnabled
    { 
        get => this.isEnabled;
        set => this.SetProperty(ref this.isEnabled, value);
    }
}
```

## Closing Thoughts
Pretty epic right!?  This can serve in a bunch of capacities - you can use it as a good global store that you can monitor for changes (ie. Message Bus).   Give it a try - give us feedback - and may all your apps be Shiny!

## Links
* [Initial Shiny Setup](introducingshiny)
* [Source Code](https://github.com/shinyorg/shiny)
* [Samples](https://github.com/shinyorg/shinysamples)
* [Documentation](https://shinylib.net)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)