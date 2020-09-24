Title: Shiny - Easy Mode Statics
Published: 1/28/2020
Tags:
    - Xamarin
    - OSS
    - Shiny
---

## Easy Mode

While I was working on Shiny, a LOT of people emailed me telling me "Component.This" doesn't exist (they could have read the samples, but whatevs).  They wanted a Xamarin Essentials approach.  I disagree with this approach to a degree since it doesn't provide some enterprise practices that I require in my day-to-day, but I'm not everyone and people have different requirements.  Thus I decide to bring the static "shims" into Shiny as well as bring a registration model similar to Xamarin Forms.  You still need some of the Shiny startup boilerplate, but there are now ways to cut that down a great deal as well.  

## Static Access

Let's start with the static accessors.  Essentially, if we have a service called Shiny.Jobs.IJobManager, we now have an equivalent ShinyJobManager with all of the functions of the interface.  No more DI necessary.

```csharp
// examples
await ShinyJobManager.Schedule(new JobInfo(...));
ShinySettings.Set("1");
ShinyEnvironment.AppVersion

ShinyGeofences.StartMonitoring(...)
ShinyGps.StartListener(...)
ShinyMotionActivity.Query(...)
```

Don't like the boilerplate?  I can only get rid of so much, but I've made service registration match the Xamarin Forms assembly attribute.  


## Registering with Assemblies

I wanted a fast way to skip the startup file and give a familiar style, so I borrowed a page from Xamarin Forms.  You can register you services very similar to how XF does it, so that you can use your services inside of Shiny delegates and jobs

In the usual "init" places, simply do the following

```csharp
iOSShinyHost.Init(ShinyStartup.FromAssemblyRegistration(typeof(App).Assembly));
```

Now, somewhere in your shared project - add attributes similar to the following to wire up some of your stuff
```csharp
[assembly: ShinySqliteIntegration(true, true, true, true, true)]
[assembly: ShinyJob(typeof(SampleJob), "MyIdentifier", BatteryNotLow = true, DeviceCharging = false, RequiredInternetAccess = Shiny.Jobs.InternetAccess.Any)]
[assembly: ShinyAppCenterIntegration(Constants.AppCenterTokens, true, true)]
[assembly: ShinyService(typeof(YourOwnService))]
[assembly: ShinyService(typeof(IYourOwnService, typeof(YourOwnService))]
```

## Auto-Registering Everything!
For any and all Shiny libraries, we automatically setup the services for you.  You don't even need a startup file (similar to registering with assembly attributes).  Note that if you do you use DI in your delegates, you will need to use assembly attribute registrations if you want to take a "bite of both pies".  With auto assemblies, we also go out and detect the appropriate delegates for each service type.  If one isn't found and it is required, we throw an exception during Init.

Auto-registration is by far the easiest way to get up and running, but it comes at the cost of startup performance.  On Android, you may notice it during startup.  You also may run into linker specific issues - my suggestion is to disable linking on Shiny libraries.

```csharp
// iOS - Normal AppDelegate addition
iOSShinyHost.Init(ShinyStartup.AutoRegister());

// Android - YourAndroidApplication.OnCreate
AndroidShinyHost.Init(ShinyStartup.AutoRegister());
```

We don't do this for jobs though.  You need to schedule those yourself just like before.


## Closing Thoughts

If speed and testability are things that are important to you, I don't recommend this approach for everyone.  Use the traditional mechanisms with DI.

## Links
* [Initial Shiny Setup](introducingshiny)
* [Samples](https://github.com/shinyorg/shinysamples/tree/master/Samples)
* [Documentation](https://shinylib.net)
* Shiny.Core - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)