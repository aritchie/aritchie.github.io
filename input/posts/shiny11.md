Title: Shiny 1.1
Published: 3/12/2020
Image: images/shiny_logo.png
Tags:
    - Xamarin
    - OSS
    - Shiny
---

It's only been a month since the 1.0 release and 1.1 is already here!  1.1 is packed with a ton of updates though.  I've received a ton of feedback over the past few weeks.  One of the big goals I have going forward is to make the boilerplate discovery even easier which hopefully is shown starting with this release.  Make sure to read to the end, the new Push library is pretty epic in my opinion.


Easier Boilerplate
---

I found that users were often missing some of the unfortunate boilerplate code that is necessary to integrate with the OS.  With 1.1, I set out to make the discovery a bit easier to work with. 

```csharp
using Shiny; // important - this will link in the extension methods

// iOS 
public class YourAppDelegate : ApplicationDelegate
{
    public void FinishedLaunching() 
    {
        // 1.0
        iOSShiny.Init(new YourStartup());

        // 1.1
        this.ShinyFinishedLaunching(new YourStartup());
    }

    public override void PerformFetch(UIApplication application, Action<UIBackgroundFetchResult> completionHandler)
    {
        // 1.0
        Shiny.Jobs.JobManager.PerformFetch(completionHandler);

        // 1.1
        this.ShinyPerformFetch(completionHandler);
    }

    public override void HandleEventsForBackgroundUrl(UIApplication application, string sessionIdentifier, Action completionHandler)
    {
        // 1.0
        Shiny.Net.Http.HttpTransferManager.HandleEventsForBackgroundUrl(sessionIdentifier, completionHandler);
        
        // 1.1
        this.ShinyHandleEventsForBackgroundUrl(sessionIdentifier, completionHandler);    
    }
}

// Android Activity
public class MainActivity : FormsAppCompatActivity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        TabLayoutResource = Resource.Layout.Tabbar;
        ToolbarResource = Resource.Layout.Toolbar;

        base.OnCreate(savedInstanceState);

        Forms.Init(this, savedInstanceState);
        this.LoadApplication(new App());

        // new with 1.1
        this.ShinyOnCreate();
    }


    protected override void OnNewIntent(Intent intent)
    {
        base.OnNewIntent(intent);

        // new with 1.1
        this.ShinyOnNewIntent(intent);
    }


    public override void OnRequestPermissionsResult(int requestCode, string[] permissions, [GeneratedEnum] Permission[] grantResults)
    {
        base.OnRequestPermissionsResult(requestCode, permissions, grantResults);

        // 1.0
        Shiny.AndroidShinyHost.RequestPermissionsResult(requestCode, permissions, grantResults);

        // 1.1
        this.ShinyRequestPermissionsResult(requestCode, permissions, grantResults);
    }
}

```


App State Delegates
---
Shiny often needs to watch when the app is foregrounding and backgrounding to run stuff internally, so why not expose this functionality to everyone else.  
This delegate does behave a tad different than other async delegates in Shiny.  Why?  Because app state changes wait for no one.  You need to do something here quick and synchronously if you want it to get done!

```csharp
// to register in your Startup
services.AddAppState<YourAppStateDelegate>();

// and now for the delegate - you can DI as with all other Shiny things
public class AppStateDelegate : IAppStateDelegate
{
    public void OnStart() {}
    public void OnForeground() {}
    public void OnBackground() {}
}
```

Foreground on Jobs
---
Building on AppStates, you can configure jobs to run while your app is open.  This is useful if you've got a task that you might want calling home often while you are online.  Perhaps you want to try and synchronize a bunch of offline data while your user is moving around the application?  This is your chance.

To get jobs to run in the foreground, you need to do 2 things

1. In your Shiny startup - add services.UseJobForegroundService(TimeSpan.FromSeconds(30));.  The timespan is how often you want it to poll the jobs to run.
2. When registering your job, set the JobInfo.RunOnForeground parameter to true;


AndroidX
---
AndroidX is now supported in most of the Shiny packages where possible.  Simply upgrade your Android target framework in your head project to use Android 10 and voila, you will automatically have the new AndroidX Shiny libraries.

There is really only 1 significant upgrade other than freeing yourself from those evil support libraries.  I've replaced the default JobScheduler engine with the new shiny WorkManager.  The awesome part, you don't have to change a thing in your job code to get the benefits of this!


Other Core Updates
---
A lot of these services already exist in Xamarin Essentials, but are needed by Shiny internals, so I continue to build on these.  These services often have different twists vs Essentials.  For instance, PowerManager & Connectivity both implement INotifyPropertyChanged very much like an MVVM viewmodel and in fact, they behave exactly like that.  You can use old school .NET events to wire up to PropertyChanged or bring in some RX muscle to watch changes on the properties. 

In 1.1,

Connectivity now has the ability to return the current cellular carrier of your device.  PowerManager adds the ability to check if the energy save mode is on.  

```csharp
ShinyConnectivity.CellularCarrier; // AT&T, etc

ShinyPowerManager.IsEnergySaverEnabled; // Boolean

```

Notifications
---
There are several minor, but nice updates to the local notifications library
* A better way to set sounds either from custom or system
* AndroidX upgrade - remove those pesky support libraries!
* Android now allows you to use Big text styles and large icons - check the Notification.AndroidOptions

Location Updates
---
Mostly bugfixes for this one with some minor updates
* We are now checking for the new Android 10 background location permission as necessary
* The addition of a pure GPS to Geofence module.  Some customers have stated they don't like the requirement on Google Play Services for many reasons, thus, I wanted to give this option.  This is now also the fallback option (automagically) when Google Play Services is not available!

```csharp
// in your shiny startup file - this is to force use the GPS geofence module
services.UseGpsDirectGeofencing<YourGeofenceDelegate>();

// otherwise, the regular geofence module will fallback with the usual
services.UseGeofencing<YourGeofenceDelegate>();
```

Push
---
It turns out, a lot of people started to use AppCenter push notifications and like most push centers, it will be disappearing into the wind.  What if you could have a push notification engine that had "one easy abstraction to rule them all"?  This is what I set out to do. 

This module is still in beta, but very close to release.

Currently, Shiny supports the default native OS push mechanisms as well as Azure Push Notification Hubs.  Firebase is coming soon!


```csharp
// install Shiny.Push.AzureNotifications nuget into your netstandard project
services.UsePushAzureNotificationHubs<PushDelegate>("Your listener connection string", "your hub name");

// don't want any of those provider and like going your own - simply install 
services.UsePushNotifications<PushDelegate>();
```

Note that there are several other arguments you can (should) pass to the notification provider such as categories & actions if you use them in your notifications.  Also, you can have Shiny request permission to use push notifications right off the app start.


The push delegate is where you can integrate with data in your backend
```csharp
namespace Samples.ShinyDelegates
{
    public class PushDelegate : IPushDelegate
    {
        public Task OnEntry(PushEntryArgs args)
        {
            // the user tapped on a notification
        }

        public Task OnReceived(IDictionary<string, string> data)
        {
            // you received a push, you can work with the payload data here OR maybe go retrieve data from your server
        }

        public Task OnTokenChanged(string token)
        {
            // you should store this and send it off to your server so you send notifications in the future
        }     
    }
}
```

The push abstraction has a few "neat" additions.  If you have a UI that listens for silent notifications like I often do, you need an easy way for listening to those notifications. 

```csharp
// this uses the static host, but you really should DI this!
ShinyPush.WhenReceived().Subscribe(x => {
    // x is a dictionary that contains the data section of your push notification
})
```


### Closing
There's still more - feel free to check out the changelog in the links below!  
Do you have any suggestions?  Do you have any device or background mode that is difficult to work with?  Send over a suggestion to [GitHub](https://github.com/shinyorg/shiny)!

For those that are watching, I have been actively working on macOS and Tizen implementations.  Still some work to go to get all of those parts stabilized before them make it into Shiny.


## Links
* [1.1 Change Log](https://github.com/shinyorg/shiny/blob/master/ChangeLog.md)
* [All Shiny Articles](/tags/Shiny)
* [Samples](https://github.com/shinyorg/shinysamples/tree/master/Samples)
* [Documentation](https://shinylib.net)
* Shiny.Core - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* Shiny.Locations - [![NuGet](https://img.shields.io/nuget/v/Shiny.Locations.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Locations/)
* Shiny.Notifications - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* Shiny.Net.Http - [![NuGet](https://img.shields.io/nuget/v/Shiny.Net.Http.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Net.Http/)
* Shiny.Push - [![NuGet](https://img.shields.io/nuget/v/Shiny.NFC.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Push/)
* Shiny.Push.AzureNotificationHubs - [![NuGet](https://img.shields.io/nuget/v/Shiny.Push.AzureNotificationHubs.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Push.AzureNotificationHubs/)