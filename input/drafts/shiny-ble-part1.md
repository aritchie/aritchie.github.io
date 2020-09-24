Title: BluetoothLE (Part 1) - Getting Started - Shiny Style
Published: 7/1/2019
Tags:
    - Xamarin
    - BluetoothLE
    - OSS
    - Shiny
---

Bluetooth (specifically LE) has been a big project of mine for some time.  The reason I like BLE so much is that it is the one protocol that just works on all platforms.  You don't need internet, you don't need any weird permission (mostly) or to beg the user to do much beyond asking "can we use your bluetooth radio"?  

Over the last few years though, I've seen people really get BLE wrong.  In an era of web devs, working with bytes often seems to be a lost art.  BLE allows you to send 20 bytes per packet.

* Sending 8k worth of JSON and complaining things are slow
* 


## Android Pain
Android is a finnicky beast ESPECIALLY when it comes to BluetoothLE.  The protocol, while being asynchronous in nature, is anything but async on Android.  If you execute reads or writes on top of each other, it often leads to the dreaded GATT 133 error.


## Links
* [Initial Shiny Setup](introducingshiny)
* [Source Code](https://github.com/shinyorg/shiny)
* [Samples](https://github.com/shinyorg/shinysamples)
* [Documentation](https://shinylib.net)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.BluetoothLE.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.BluetoothLE/)