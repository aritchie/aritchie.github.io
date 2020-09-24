Title: Beacons - Shiny Style
Image: images/shiny_beacons.png
Published: 06/10/2019
Tags:
    - Xamarin
    - OSS
    - Shiny
---

## What are Beacons
Beacons are low powered IoT devices that emit a signal over Bluetooth advertising that contains a specific (and very small piece of data) that is like an address.  You attach that address to a "location" of some type.  They are like an invisible light house.  If you have a device that can "see" those signals, you can find out how close you are to it.  

Some beacons include sensors like temperature, accelerometers, ambient light, pressure, and more.  Head over to [Estimote](https://estimote.com/products/) to take a look.  These guys are essentially the Rolls Royce of the beacon industry.

Beacons from a software perspective actually come in several flavours/protocols.  The main one in practice is iBeacon which is a protocol created by Apple.  There is a beacon protocol by Google called Eddystone. Eddystone doesn't seem to have caught on like iBeacons did.  Eddystone is currently not a supported with Shiny.

## Terminology
Beacons are riddled with terms and acronyms.  Let me hopefully clear up the ones that matter.

* **Identifier** - this is a string, for you to set how you see fit (within reasons obviously)
* **UUID (Universally Unique Identifier)** - This is a GUID in .NET terms.  You could equate this to the city where you live
* **Major** - This is a ushort (uint16).  This would equate to the street on which you live
* **Minor** - This is also a ushort (uint16).  This would equate to your street number.  Thereby, giving you the greatest precision in identification.
* **Ranging** - Refers to scanning for all beacons within a certain address range.  You can see any and all beacons within your filter parameters as well as how close you are to it.  Think [GhostBuster PKE Meter](https://ghostbusters.fandom.com/wiki/P.K.E._Meter)
* **Monitoring** - The art of scanning for a "filtered" set of beacon address in the background.  You don't know how close you are to the target in this mode, only if you are entering the region or leaving it.  If you watch the show "Chernobyl" on HBO - think the equivalent being the dosimeters on the belts of these guys (the clicking - you are in the bad zone) - [Watch Here](https://www.youtube.com/watch?v=uXafEIdkx6c)
    * _iOS_ - You can monitor 20 beacons (this includes any geofences your app uses as well, so careful here) max
    * _UWP/Android_ - Technically, there is no limit here, but I would stick to 20 as well.
    

## Working with Beacons - Shiny Style

Shiny provides the first fully managed beacon implementation for Android & UWP.  It's reach is only limited currently by the reach of Shiny.BluetoothLE.  For iOS, it simply makes use of the iOS API because Apple hijacks all beacon packets (you can't touch them with raw BLE).  

### Getting Started

Again, to save some time - the best place to go for getting the initial setup with shiny is [RIGHT HERE](introducingshiny).  For this article, I'm going to register [Shiny.Beacons](https://www.nuget.org/packages/Shiny.Beacons/) and [Shiny.Notifications](https://www.nuget.org/packages/Shiny.Notifications/) for backgrounding in the second part of this post under monitoring.

### Android
Other than the normal Android setup for Shiny, you need to add the following to your manifest.xml - we'll need a few bluetooth permissions here.  Bluetooth on Android requires location permissions as well as Bluetooth since Android 6 due to the use of beacons.

Let's get started with the traditional OS setup (yes UWP is supported, but is left out for now)

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH" />
```

### iOS
Again, same typical iOS initialization.  Just add the following to your info.plist.  The UIBackgroundModes is required for Beacons since they are location context even though they are bluetooth devices - so just like [Geofences](shiny-geofencing).

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

### In Your Shared Code

```csharp
using System;
using Shiny;
using Shiny.Beacons;
using Shiny.Notifications;
using Microsoft.Extensions.DependencyInjection;


namespace YourNamespace
{
    public class SampleStartup : ShinyStartup
    {
        public override void ConfigureServices(IServiceCollection builder)
        {
            // only necessary for the monitoring - for ranging only you can use UseBeacons()
            builder.UseBeacons<MyBeaconDelegate>();
            builder.UseNotifications();
        }
    }
}
```



### Ranging

Ranging is particularily easy - you can do this right in your viewmodel.  With ranging, you are given the specific beacon with its proximity.  You still have to set a beacon region to monitor that filters by at least your global UUID.  

```csharp

public class YourViewModel
{
    IDisposable scanSub;


    public YourViewModel()
    {
        this.Start = new Command(() =>
        {
            this.scanSub = Shiny
                .ShinyHost
                .Resolve<IBeaconManager>()
                .WhenRanged(new BeaconRegion(YourUuid))
                .Subscribe(x => Device.BeginInvokeOnMainThread(() =>                
                    this.Beacons.Add(x)
                ))
        });

        this.Stop = new Command(() => this.scanSub?.Dispose());
    }


    public ICommand Start { get; }
    public ICommand Stop { get; }
    public IList<Beacon> Beacons { get; private set; }
}

```


### Monitoring
With monitoring, you aren't given the specific beacon or how close the phone is to the beacon in terms of proximity.  You are basically handed back the filter you used.  This is to protect the privacy of the user (which I actually agree with).  This is a pretty easy task to perform on iOS, but brutally painful to do in Android.  Shiny makes this an absolutely delightful to do.

```csharp
using System;
using System.Threading.Tasks;
using Samples.Models;
using Shiny.Beacons;
using Shiny.Notifications;


namespace YourNamespace
{
    public class MyBeaconDelegate : IBeaconDelegate
    {
        readonly INotificationManager notifications;
        public MyBeaconDelegate(INotificationManager notifications) 
        {
            this.notifications = notifications;
        }


        public async Task OnStatusChanged(BeaconRegionState newStatus, BeaconRegion region)
        {
            await this.notifications.Send(
                $"Beacon Region {newStatus}",
                $"{region.Identifier} - {region.Uuid}/{region.Major}/{region.Minor}"
            );
        }
    }
}
```

## In Closing
Give Beacons a try - they are great for a variety of business applications from marketing to employee management

## Links
* [Initial Shiny Setup](introducingshiny)
* [Samples](https://github.com/shinyorg/shinysamples/tree/master/Samples/Beacons)
* [Documentation](https://shinylib.net)
* Shiny.Core - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* Shiny.Beacons - [![NuGet](https://img.shields.io/nuget/v/Shiny.Beacons.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Beacons/)
* Shiny.Notifications - [![NuGet](https://img.shields.io/nuget/v/Shiny.Notifications.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Notifications/)
* [Estimote Beacons](https://estimote.com/products/)
