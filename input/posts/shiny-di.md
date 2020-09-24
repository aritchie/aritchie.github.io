Title: Startup Tasks, Modules, and Stateful Delegates - Shiny Style
Published: 7/1/2019
Tags:
    - Xamarin
    - OSS
    - Shiny
---

Shiny isn't all about backgrounding, DI, RX, and all of that cool stuff.  It actually provides a ton of utility functions as well.  

I love Autofac and a lot of the functions it has.  It really is a great DI framework though is known for being a tad on the slow side.  With Shiny, I went with Microsoft.Extensions.DependencyInjection as the DI platform of choice.  It is fast, has "enough" features, built on a great set of abstractions, and has monolith company backing it.  However, it is no Autofac in terms of features - so I wanted to carry a few of them forward.  Namely, modules and startables.  I've also added a very useful feature in stateful delegates which is my personal favorite!  I'll explain what this in this article as well.

## Modules
Inversion of Control (IoC) acts as the basic building blocks of your application.  However, as your application grows, registering all of these components within a single point of entry can lead to a fat file.

Modules help by alleviating this.  They are used to help decouple your libraries, so that you can bundle up a set of related components behind a neatly wrapped package to simplify deployment and management of your application. Modules can also help by entangling bits of configuration code internal to library itself instead of literating your startup file with all of these additional flags.

Below is an example of the data registration module that I use within one of my sample apps [Shiny TODOs](https://github.com/aritchie/stream-todo). 

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using Refit;
using Shiny;
using Shiny.Jobs;


namespace Todo.Data
{
    public class DataModule : ShinyModule
    {
        public override void Register(IServiceCollection services)
        {
            services.AddSingleton(_ => RestService.For<IApiClient>(Constants.BaseApiUri));
            services.AddSingleton<TodoSqliteConnection>();
            services.AddSingleton<IDataService, SqliteDataService>();

            services.RegisterJob(new JobInfo
            {
                Identifier = nameof(SyncJob),
                Type = typeof(SyncJob),
                BatteryNotLow = true,
                RequiredInternetAccess = InternetAccess.Any
            });
        }
    }
}

```

In this example, you can see I'm injecting a SQLite connection, a data service provider, a remote API, and even registering a background job to synchronize the data to/from the server.  You may ask why this is good?  Well - the SQLite connection, api client, and job are all outside of my domain - meaning, the application as a whole, really doesn't have knowledge of their existence (nor should they).  They are just pieces of work that this particular portion of the app needs to do its job without effecting things around it.  

All that is left is to wire this into your Shiny startup (make sure you reference your library if it is a separate project).

```csharp
using Shiny;


namespace Todo
{
    public class ShinyStartup : ShinyStartup
    {
        public override void ConfigureServices(IServiceCollection services)
        {
            services.RegisterModule<Todo.Data.DataModule>();
        }
    }
}
```



## Startup Tasks
The IStartable/AutoActivate in Autofac is awesome as long as you used it in a smart way.  This concept does not exist in Microsoft's DI extensions.  These startables are something I use fairly frequently in my applications for various background tasks.  

Startup tasks in Shiny are similar to IStartable's in Autofac if you've ever used them.  Since Shiny is essentially DI agnostic, I wanted this feature to be available to all things DI.

### How do startup tasks differ from a job?
This is a great & likely common question I suspect, but also very easy to answer.  Startup tasks happen at the point of the container build when all of your services are ready to go.  The difference is that these tasks don't run in the background.  What they offer is a way of hooking up general pipeline logic within your app.  WARNING: These startup tasks should execute very quickly as you will pay a startup cost.  Also note, startup tasks are NOT async.  There are many, many, many reasons for this.  They are designed to hook up internal events and maybe wire up some necessary infrastructure... nothing more! 

### What would I do with one of these?
I often like to wire up my auth service that has a "SignOut" event.  When that signout event fires, I may delete a local database, clear all notifications, etc. 

### How do I make one?

```csharp
public class YourStartupTask : Shiny.IShinyStartupTask
{
    public YourStartupTask()
    {
        // you can inject just like everything else in shiny
    }

    public void Start()
    {
        // do your hooking of events and init stuff here :)
    }
}

// in your shiny startup
public void ConfigureServices(IServiceCollection builder)
{
    builder.RegisterStartupTask<YourStartupTask>();
}
```


## Stateful Delegates
My good friend Dan Siegel had been asking for something to help him manage GPS state and even used a decent trick with something that was already built into Shiny - the strong typed settings library.  Take a look at this [article](shinysettings) to see what's going on there.  I decided to build this into all event delegates that Shiny uses. All you have to do is make your delegate inherit from INotifyPropertyChanged, make your "stateful" properties public get/set, and raise notifications on their state changes.  While the sample below can be accomplished easier otherways, as the stateful properties become larger, this pattern begins to pay off - so I hope you're as excited about this as I am!!

NOTE: this does not work on jobs.  Jobs have their own special type of state management using the JobInfo argument in Run.  This may change in the future.

Here's a great example:

```csharp
using System;
using System.Threading.Tasks;
using Samples.Models;
using Shiny.Notifications;
using Shiny.Locations;


namespace Samples.ShinyDelegates
{
    public class YourGeofenceDelegate : Shiny.NotifyPropertyChanged, Shiny.Locations.IGeofenceDelegate
    {
        readonly INotificationManager notifications;
        public YourGeofenceDelegate(INotificationManager notifications)
        {
            this.notifications = notifications;
        }


        bool welcomed;
        public bool HasAlreadyBeenWelcomed
        {
            get => welcomed;
            set => Set(ref welcomed, value);
        }


        public async Task OnStatusChanged(GeofenceState newStatus, GeofenceRegion region)
        {
            if (!this.HasAlreadyBeenWelcomed)
            {
                await this.notifications.Send("WELCOME", "Houston welcomes you the first ever Xamarin Developer Summit"); // yes, you can see where this was used :)
                this.HasAlreadyBeenWelcomed = true;
            }
        }
    }
}

```

So next time this guy fires, whether from the background, reboot, etc - that state will be remembered.  This is pretty epic in my opinion!

Note, I had considered serializing & deserializing stateful properties (get/set) per delegate trigger, but with things like GPS that can fire rapidly, this may introduce performance issues.  The advantage to serializing/deserializing per run - is that you don't need a viewmodelish type setup.  Just plain old-C# public get/sets.

## In Closing
Modules & startup tasks provide a great way to modulizing your application and providing a rich set of wiring services together without coupling them within your normal application logic (ie. ViewModels).  


## Links
* [Initial Shiny Setup](introducingshiny)
* [Source Code](https://github.com/shinyorg/shiny)
* [Samples](https://github.com/shinyorg/shinysamples)
* [Documentation](https://shinylib.net)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* [Shiny TODOs Sample App](https://github.com/aritchie/stream-todo)

