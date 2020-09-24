Title: Notifications - Shiny Style
Published: 1/28/2020
Image: images/shiny_logo.png
Tags:
    - Xamarin
    - OSS
    - Shiny
---
What good are background tasks without being able to tell your user something went down?  We all use push notifications, but local notifications still have many uses like geofences & beacons.  Shiny Notifications (in my opinion) is the best of breed plugin in local notifications for Xamarin.  

Setup & Permissions
---

```csharp
// though this is using the static host, you really should DI for good things such as testing
ShinyNotifications.RequestAccess();
```

Sending
---

```csharp
ShinyNotifications.Schedule(new Notification {

});
```

Notification Delegate
---

The notification is your integration point to the notification stuff.  Below is an example of a notification delegate.  

```csharp
public class NotificationDelegate : INotificationDelegate
{
    public Task OnEntry(NotificationResponse response) 
    {
        // this is when the user taps on your notification
    }

    public Task OnReceived(Notification notification)
    {
    }
```

Notification Categories & Text Replies
---
Notification categories and actions are really slick ways of having your user interact or even enter replies to your app without ever entering your app


```csharp
services.UseNotifications<NotificationDelegate>(
    true,
    new NotificationCategory(
        "Test",
        new NotificationAction("Reply", "Reply", NotificationActionType.TextReply),
        new NotificationAction("Yes", "Yes", NotificationActionType.None),
        new NotificationAction("No", "No", NotificationActionType.Destructive)
    )
);

// to process the action
```

### Android
<img src="/images/shiny-notifications/android-reply-button.gif" alt="Android Button" width="250" />
<img src="/images/shiny-notifications/android-reply-text.gif" alt="Android Reply" width="250" />

### iOS
<img src="/images/shiny-notifications/ios-reply-button.gif" alt="iOS Button" width="250" />
<img src="/images/shiny-notifications/ios-reply-text.gif" alt="iOS Reply" width="250" />


## Links
* [Initial Shiny Setup](introducingshiny)
* [Source Code](https://github.com/shinyorg/shiny)
* [Samples](https://github.com/shinyorg/shinysamples)
* [Documentation](https://shinylib.net)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Notifications.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Notifications/)