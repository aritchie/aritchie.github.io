Title: Geofencing with a Pinch of Notifications - Shiny Style
Published: 5/21/2019
Tags:
    - Xamarin
    - OSS
    - Shiny
---

GPS & Geofencing is a common need for mobile and IoT platforms alike.  However, mobile platforms with backgrounding in this area are always painful and that is being nice.  We've tried several plugins over the years, but they have all some sort of pain point.  Shiny aims to solve all of these as it provides a lot of base infrastructure to make things... shiny ;)

We'll talk about GPS in a future article.  In this article, we'll focus on the geofencing and add in some notifications to make things awesome!

## What Are Geofences?
Geofencing is a location-based virtual boundary that uses GPS.  When you enter or exit this boundary, an event is executed in your application code.  A good example of this is a user coming close to your store and triggering a local notification - "Hey Peoples - come buy some stuff.  I'll make a deal you can't refuse".  When the user leaves the area "We're so sorry to see you leave.  Come back again and I'll give you 99% off with this magic code".  It's beautiful.... can be annoying, but if you do it right - you can create a good conversion opportunity.

## Geofencing - Why?
Geofencing is very efficient on battery for the mobile operating systems.  One of the pitfalls is that it isn't exact in terms of fire times.  Sometimes it can take a couple of minutes before the event is actually triggered.  I often get asked why use a geofence over raw GPS.  The explanation I use is, are you in the mall or not - if you are in the mall, you likely won't have a good GPS signal either.  The calculation for determining the circular area of a geofence is also not 2 lines of code if you need to do the effort yourself using raw GPS.  Geofences are good for when you enter and exit a particular area, GPS is better for real time "where are you".

## Pitfalls
I've also seen a number of "gone bad" geofence routines.  This one implementation tried to register over 300 geofences at startup.... so ya... don't do that.  In fact, iOS limits you to 20 and Android caps it at 60.  You may think "wow that is limiting", but really - it isn't.  If you can't get the job done in 20 geofence registrations, honestly, in my opinion - you're doing something wrong.  I generally set a geofence on the center of a structure/area that I'm interested in and set a circular distance of 200 meters.  This is the perfect balance between battery and quick OS response time in my experience.

## Making It Happen With Shiny

Do all of your normal Shiny setup - you can read my [Introducing Shiny](introducingshiny) to get going on the general setup stuff.

[Shiny.Locations](https://www.nuget.org/packages/Shiny.Locations/) comes as a separate nuget package, but provides functionality for GPS & Geofencing.  
You'll also want to pickup [Shiny.Notifications](https://www.nuget.org/packages/Shiny.Notifications/) to complete the sample in this article.

### In Your Shared code

First, let's create our geofence delegate.  This is the guy that catches those events in the background.  As with all delegates in Shiny, you can inject your own services as long as they are registered with the Shiny container in your startup file.  

```csharp
using Shiny.Locations;

namespace MyNamespace
{
    public class MyGeofenceDelegate : IGeofenceDelegate
    {
        readonly INotificationManager notifications;

        public MyGeofenceDelegate(INotificationManager notifications)
        {
            this.notifications = notifications;
        }


        public async Task OnStatusChanged(GeofenceState newStatus, GeofenceRegion region)
        {
            if (newState == GeofenceState.Entered)
            {
                await this.notifications.Send(new Notification 
                { 
                    Title = "WELCOME!",
                    Message = "It is good to have you back " + region.Identifier 
                });
            }
            else 
            {
                await this.notifications.Send(new Notification 
                { 
                    Title = "GOODBYE!", 
                    Message = "You will be missed at " + region.Identifier
                });
            }
        } 
    }
}
```

Now, let's hook this guy up to Shiny, so we can get everything running

```csharp
using System;
using Shiny;
using Shiny.Locations;
using Shiny.Notifications;
using Microsoft.Extensions.DependencyInjection;


public class SampleStartup : ShinyStartup
{
    public override void ConfigureServices(IServiceCollection builder)
    {
        builder.UseGeofencing<MyGeofenceDelegate>();
        builder.UseNotifications(true);
    }
}
```

### Android
Other than the normal Android setup for Shiny, you need to add the following to your manifest.xml - we'll need a few bluetooth permissions here
```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### iOS
Again, same typical iOS initialization.  Just add the following to your info.plist.  The UIBackgroundModes is required for geofencing.

```xml
<key>NSLocationAlwaysUsageDescription</key>
<string>Your message</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Your message</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Your message</string>

<key>UIBackgroundModes</key>
<array>
    <string>location</string>
</array>
```


## Registering an actual Geofence

Your viewmodel is the best place to register a geofence.  

Please note that while registering a geofence will request the necessary user permissions through the OS, if the user declines the necessary permission, the method will toss an exception.  It is a good practice to use IGeofenceManager.RequestAccess to know the state of things yourself.

```csharp
using Shiny;
using Shiny.Locations;
using Shiny.Notifications;


public class YourViewModel
{
    public YourViewModel()
    {
        // shiny doesn't usually manage your viewmodels, so we'll do this for now
        var geofences = ShinyHost.Resolve<IGeofenceManager>();
        var notifications = ShinyHost.Resolve<INotificationManager>();

        Register = new Command(async () => 
        {
            // this is really only required on iOS, but do it to be safe
            var access = await notifications.RequestAccess();
            if (access == AccessState.Available)
            {
                await this.geofences.StartMonitoring(new GeofenceRegion(
                    "CN Tower - Toronto, Canada",
                    new Position(43.6425662, -79.3892508),
                    Distance.FromMeters(200)
                )
                {
                    NotifyOnEntry = true,
                    NotifyOnExit = true,
                    SingleUse = false
                });
            }
        });
    }

    public ICommand Register { get; }
}
```

## In Closing
Geofencing + Notifications is a powerful combination for things like marketing.  Hopefully, Shiny helps make this combo easy for you!

## Links
* [Initial Shiny Setup](introducingshiny)
* [Source Code](https://github.com/shinyorg/shiny)
* [Samples](https://github.com/shinyorg/shinysamples)
* [Documentation](https://shinylib.net)
* Shiny.Core - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* Shiny.Locations - [![NuGet](https://img.shields.io/nuget/v/Shiny.Locations.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Locations/)
* Shiny.Notifications - [![NuGet](https://img.shields.io/nuget/v/Shiny.Notifications.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Notifications/)

