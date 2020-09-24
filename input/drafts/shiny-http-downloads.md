Title: HTTP Background Downloads - Shiny Style
Published: 7/1/2019
Tags:
    - Xamarin
    - OSS
    - Shiny
---

Large background downloads where the OS kills your app is annoying and not trivial to fix.  Fortunately, Shiny has your back with [Shiny.Net.Http](https://www.nuget.org/packages/Shiny.Net.Http/).  This library allows your downloads to continue and allow the OS to manage things efficiently.  A few other things that aren't an out-of-box-experience (OOBE) with downloads (HttpClient or anything else of that matter) is tracking downloading progress, download speed, and time remaining until completion.  Shiny really provides an ALL-IN-ONE place to manage all of your HTTP transfers.  It also does uploads, but that is a different can of worms for another time.


## Get A Download Going

So in your ViewModel or Page (whatever floats your boat) - let's queue up a new download. 

```csharp

var fileSystem = Shiny.ShinyHost.Resolve<IFileSystem>();
var transferManager = Shiny.ShinyHost.Resolve<IHttpTransferManager>();

var path = Path.Combine(fileSystem.AppData.FullName, "yourLocalFilename.ext");
var request = new HttpTransferRequest("https://yourdownload.com/something.ext" path, false)
{
    UseMeteredConnection = true
};
await transferManager.Enqueue(request);
```

A few things to note in this examples - first off, the filesystem... you need to store the file in your appdata folder for your current platform.  Second, note the use of metered connection.  If you don't want to obliterate your users data, you can set this to false to prevent downloading on a cell connection.


## Progress
I have to admit, I'm impatient.  If I know how long something is going to take, I can go do something else while I wait.

```csharp

var vm = new HttpTransferViewModel
{
    Identifier = transfer.Identifier,
    Uri = transfer.Uri,
    IsUpload = transfer.IsUpload,

    Cancel = ReactiveCommand.CreateFromTask(async () =>
    {
        var confirm = await dialogs.Confirm("Are you sure you want to cancel all transfers?", "Confirm", "Yes", "No");
        if (confirm)
        {
            await this.httpTransfers.Cancel(transfer.Identifier);
            await this.Load.Execute();
        }
    })



    this.httpTransfers
.WhenUpdated()
.WithMetrics()
.SubOnMainThread(
transfer =>
{
var vm = this.Transfers.FirstOrDefault(x => x.Identifier == transfer.Transfer.Identifier);
if (vm != null)
{
    ToViewModel(vm, transfer.Transfer);
    vm.TransferSpeed = Math.Round(transfer.BytesPerSecond.Bytes().Kilobytes, 2) + " Kb/s";
    vm.EstimateTimeRemaining = Math.Round(transfer.EstimatedTimeRemaining.TotalMinutes, 1) + " min(s)";
}
},
ex => this.dialogs.Alert(ex.ToString())
)
.DisposeWith(this.DeactivateWith);
}


static void ToViewModel(HttpTransferViewModel viewModel, HttpTransfer transfer)
{
viewModel.PercentComplete = transfer.PercentComplete;
viewModel.PercentCompleteText = $"{transfer.PercentComplete * 100}%";
viewModel.Status = transfer.Status.ToString();
}

```

## Background Operations

Lastly, you still need to hook up to completion events.

```csharp

namespace YourNamespace
{
    public class YourHttpDelegate : IHttpDelegate
    {
        public async Task OnError(HttpTransfer transfer, Exception ex)
        {
        }


        public async Task OnCompleted(HttpTransfer transfer)
        {
        }
    }
}
```

## Links
* [Initial Shiny Setup](introducingshiny)
* [Source Code](https://github.com/shinyorg/shiny)
* [Samples](https://github.com/shinyorg/shinysamples)
* [Documentation](https://shinylib.net)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Core.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Core/)
* [![NuGet](https://img.shields.io/nuget/v/Shiny.Net.Http.svg?maxAge=2592000)](https://www.nuget.org/packages/Shiny.Net.Http/)