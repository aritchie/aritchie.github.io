Title: Shiny 1.0 Release
Published: 1/27/2020
Image: images/shiny_logo.png
Tags:
    - Xamarin
    - OSS
    - Shiny
---


The Road To 1.0
---
TLDR;
Its been a while since I blogged because I've really been digging in to finish up a lot of the core parts of Shiny.  This has been such a fun OSS project to build.  I remember when I was discussing this over a year ago with a couple of colleagues.  Shiny had a horrid name when I first started called 'ACR Core'.  The name Shiny came to me one night on twitter when I was making fun of people complaining about updating to the latest Android Support libraries right away and having issues.... shocking.... when people see new NuGets, they must update them regardless of if they need them or not.  "They must have their Shinies" (said in the voice of Golem from LOTR).  The name stuck, but became more of an oxymoron because nothing in Shiny is what you see in your UI... EPIC!  

When I started doing Xamarin plugins a few years ago, they were just one off pieces of code.  The issue was that I always need some sort of persistence, permissions, settings, logging, etc.  It would also be a struggle to get people to wire things up properly for things to work.  I needed a way that would force a bit structure, bring all of the components I needed to services, and bring the users services into a solid unit while also providing a testable end product.  Thus Shiny had to become more of a framework than a plugin. The project intent is to create a simple way to create background services and work with some of the complex hardware on mobile devices (ie. BLE) while having a clean testing pattern and tie-in to your infrastructure. 


What's In The Release
---
* Background Jobs
* GPS, Geofencing, & Motion Activity
* Local Notifications with all the bells and whistles (blog post to follow)
* HTTP Transfers - Background File Uploads and Downloads with metrics to track things like speed, estimate time remaining, etc
* Sensors (RX Style) - pedometer, heart rate, accelerometer, etc
* A lot of the Essentials stuff (file system access, environment, settings, permissions, etc)
* PS - for those that don't like DI - there are now Static Shims (blog post to follow)


What's Next?
---

1. **Bluetooth & Beacons** - BLE just simply wasn't where I wanted it to be in terms of functionality and background ability, but I wanted to release everything else.  It's close and is the next priority.
2. **Integrations** 
    * MvvmCross
    * ReactiveUI
    * Xamarin Essentials 
        Shiny has a lot of overlap with Essentials.  I had debated just taking it on as a dependency as opposed to continuing to use my implementations that I've collected over the years, but Microsoft simply can't change course, introduce breaking changes, or release as fast as Shiny can.  That isn't a knock against them at all.  They've got a larger community and a process they have to follow.  
3. **Documentation** - It needs to be done.  It is a lot of work.  It will get there when it gets there
4. **Universal Windows Platform** - It is in the core packages, but I want to make it more of a first class citizen.  With the Surface Neo coming, I feel like there is a bunch of potential here.
5. **Tizen, watchOS, & macOS** - There is lots of Shiny that I want to bring to these platforms.  There has been some interest and to reach these guys isn't too difficult.
6. **WASM, Blazor, & Uno** - These are definitely on the radar.  I think WASM is going to be a huge game changer in our industry.  The only reason I haven't done it yet is that I'm not looking to be an early adopter around an OSS project of this size without help (and especially since WASM isn't even in a stable state yet).

Thanks
---

A big thank you to all of those who have contributed in any way, shape, or form to Shiny.  I want to send out a few of special thank yous to people

* [Dan Siegel](https://twitter.com/DanJSiegel) 

> My good friend Dan has been helping really get the word out about Shiny over the last few months.  He's blogged, tweeted, contributed many issue reports, and even developed a beautiful and direct integration with Prism (Shiny.Prism).  Dan has been the go-to for fixing devops stuff and also did me the solid of starting the twitter hashtags #ShutUpAllan and #TakeMyBugReport.  In turn, I now refer to Dan as Troy McClure from the Simpsons.  Go listen to one of his YouTube feeds.  BIG CHEERS DAN

* [James Montemagno](https://twitter.com/JamesMontemagno)

> James has made a ton of mentions in the community standups, podcasts, and various shows he runs. He even tried Shiny on out one of his livestreams while building the Hanselmann app which included some bugfixing on my part - BUT IT WAS A BETA :)  Thanks so much James

* [Keith Dahlby](https://twitter.com/dahlbyk)

> Keith has been contributing issue reports, pull requests with fixes & features for a few months.  He's just been a general good open source citizen and a fantastic person to have working in the project.  Thanks Keith!

* [Claudio Sanchez](https://twitter.com/ClaudioASanchez) and [Danny Ackerman](https://twitter.com/dannyackerman)

> The only long term sponsors I've ever had!  It's very humbling you that you guys contribute money because you find value out of the bugs...cough.... code I write :)


## Links
* [All Shiny Articles](/tags/Shiny)
* [Samples](https://github.com/shinyorg/shinysamples/tree/master/Samples)
* [Documentation](https://shinylib.net)
* Shiny.Core - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* Shiny.Locations - [![NuGet](https://img.shields.io/nuget/v/Shiny.Locations.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Locations/)
* Shiny.Notifications - [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* Shiny.BluetoothLE - [![NuGet](https://img.shields.io/nuget/v/Shiny.BluetoothLE.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.BluetoothLE/)
* Shiny.Beacons - [![NuGet](https://img.shields.io/nuget/v/Shiny.Beacons.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Beacons/)
* Shiny.Sensors - [![NuGet](https://img.shields.io/nuget/v/Shiny.Sensors.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Sensors/)
* Shiny.Net.Http - [![NuGet](https://img.shields.io/nuget/v/Shiny.Net.Http.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Net.Http/)
* Shiny.SpeechRecognition - [![NuGet](https://img.shields.io/nuget/v/Shiny.SpeechRecognition.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.SpeechRecognition/)
* Shiny.Testing - [![NuGet](https://img.shields.io/nuget/v/Shiny.Testing.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Testing/)
* Shiny.Push - [![NuGet](https://img.shields.io/nuget/v/Shiny.Push.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Push/)
* Shiny.NFC - [![NuGet](https://img.shields.io/nuget/v/Shiny.NFC.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.NFC/)