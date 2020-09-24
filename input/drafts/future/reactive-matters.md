Title: Reactive Matters to Mobile
Published: 2/9/2019
Tags:
    - Xamarin
    - Reactive
---
Reactive matters

On mobile, async operations are a must.  Getting off that main UI thread so you can wait for slow network I/O requests to be serviced or because you need to suck in 20megs of JSON junk.  For anyone who is old enough to remember the faded screen of windows forms death because it was hanging the UI thread can attest to this.


Why not events?
Well - events are better than asking all the time, but events can leak memory, don't filter, don't transform

Task are enough!
For request/response style setups - they can be.  

So RX
RX is async & events on steriods.  Every other major platform embraces RX with open arms.  Flutter uses streams which is an event stream that allows for much richer sets of event logic than .NET offers out of box.  Also, there is RX-Dart on top of that.  Let's not forget RXJava, and of course - RXJS.  I'm not sure why RX.NET is snubbed by the .NET presentationware devs, but don't knock it until you've written more than "hello world".  It is often seen as over complex.  My argument to this is, try to write anything like Throttle, Sample, Window, Average, Buffer, etc /w filtering & transforms on events.... then talk to me.

Let's go a step further with ReactiveUI
I love ReactiveUI.  Below is something we use on a MVVM ViewModel for "as you type" searches and comes right off the [ReactiveUI website]().  It isn't hard to read if you take a look at it.  Look at the beauty of this thing

```csharp
this.WhenAnyValue(x => x.SearchQuery)
    .Throttle(TimeSpan.FromSeconds(0.8), RxApp.TaskpoolScheduler)
    .Select(query => query?.Trim())
    .DistinctUntilChanged()
    .Where(query => !string.IsNullOrWhiteSpace(query))
    .ObserveOn(RxApp.MainThreadScheduler)
    .InvokeCommand(ExecuteSearch);
```

So RX isn't a big deal in .NET?
There are smart people fighting for it to live on.  One of them in Geoffrey Huntley.  I love this guy.  He's been a HUGE open source advocate over the years.  I refer to him as the Angry Kangaroo (he doesn't know that - so shhh)