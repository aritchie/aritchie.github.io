Title: Common Pitfalls with Shiny
Published: 2/9/2019
Image: images/shiny_logo.png
Tags:
    - Xamarin
    - OSS
    - Shiny
---

During the course of the LOOOOONG Shiny beta.  I learned quite a bit of how people write there software and even how they think the operating systems work.  Some of these issues I'm able to deal with, others - I am not.  Here's a list of DO's and DONT's I've seen over the past few months.


1. DON'T broadcast to the UI from a delegate

Why? If the task is truly executing in the background, you don't have a UI to broadcast to you.  This can lead to errors.  Also, delegates generally execute on a different thread and sometimes quite rapidly, thus threading your UI thread.  Record these events in table OR in most cases, Shiny has a "foreground" equivalent method.  ie. Instead of using IGpsDelegate to send events, you can inject IGpsManager and use WhenReading to display data.


2. DON'T hook events or have long running tasks in delegates

Why? Android broadcast receivers and iOS delegates only give you a few seconds to execute


3. DO use jobs if you have something that is going to run for a 20-30 seconds

Background fetch on iOS and job manager on Android give give you a fair bit of time to do things. However, they execute when certain criteria


4. DON'T assume that your network calls will work at any time


5. DO use logging (and check those logs for job and delegate errors)

Shiny does trap all user code exceptions from delegates and jobs.  It also will log these to whatever logging store you may have


6. Treat jobs as periodic tasks, not scheduled tasks

There is a huge difference here.  Scheduled means it will run at a time of your choosing.  Periodic in this case, means when the OS thinks it really is a good time to run based on things like 
    * Time of day - will the user be using their phone in the little bit
    * Has your app been a "good citizen" meaning that it isn't chewing CPU, memory, and battery while it runs in the background