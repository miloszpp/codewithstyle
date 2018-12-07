---
title: 'How I built a PWA with Angular and Firebase Part 1: Introduction'
tags:
  - angular2
  - firebase
  - pwa
  - severless
url: 232.html
id: 232
categories:
  - Angular
  - Web
date: 2017-04-14 18:29:30
---

This post is part of a series in which I will describe how I built my first PWA. It will touch on many topics such as Angular 2, Ionic 2, Firebase, service workers, push notifications, serverless architectures. I hope you find it useful when building your own PWAs. I've spent last month working on my new project. It's called [Friendtainer](http://friendtainer.com) and it's a smartphone app which helps you meet with your friends regularly. You can add contacts and decide how often you want to meet. If you fail to record a meeting within given timespan you will start receiving reminders until you record a meeting.

*   [Part 1: Explaining PWA and introduction](https://codewithstyle.info/how-i-built-a-progressive-web-app-with-angular-and-firebase-part-1/)
*   [Part 2: Ionic and app manifest](https://codewithstyle.info/how-i-built-a-pwa-with-angular-and-firebase-part-2-ionic-2/)
*   [Part 3: Push notifications with Firebase Cloud Messaging](https://codewithstyle.info/push-notifications-with-fcm/)

### PWAs (Progressive Web Apps)

I decided to create the app as a Progressive Web Application (PWA). What is a PWA? It's a new term coined by Google which basically means a web application which provides a native-like experience on mobile devices. There are a few indicators of whether a web app is a PWA:

*   Looks like a native app on mobile devices
*   Loads fast and supports offline mode
*   Can be launched from the phone's home screen
*   Runs full-screen
*   May support push notifications (so the phone will display notifications even if the browser is not running)

What are the advantages of a PWA over a native mobile app? **Discoverability**! You don't need to put your app in the App Store or the Play Store. All your users need to do is to go to a specific URL. Besides, you can use your web skills and don't need to learn platform-specific technologies such as Swift or the Android framework.

### Modern web technologies at work

As you can see, there are quite a few requirements that the app should satisfy. Let's take them one by one.

*   _Looks like a native app on mobile devices_ \- I know Angular best so I would like to use it to build the app. [Ionic 2](http://ionic.io/2) is an excellent framework based on Angular 2 which provides a set of UI components that make your app look like a native app. The framework detects the device it's running on and applies different styling depending on whether it's the iOS or the Android operating system.

![](http://codewithstyle.info/wp-content/uploads/2017/04/Zrzut-ekranu-2017-04-14-o-12.06.43.png)

*   _Loads fast and supports offline mode_ \- this part is taken care of by [Service Workers](https://developers.google.com/web/fundamentals/getting-started/primers/service-workers). Service Workers is a relatively new technology which allows you to run JavaScript in the background. What it means is that your web app can register a piece of JavaScript code that will keep running even if the user closes the browser! Service Workers can also intercept HTTP calls made by the app. Therefore, it's possible to implement offline caching of all calls made by the app using them.
*   _Can be launched from the phone's home screen_ and _Runs full-screen_ - another technology called [The Web App Manifest](https://developers.google.com/web/fundamentals/engage-and-retain/web-app-manifest/) is responsible exactly for this. It's quite easy to achieve - all you need to do is write a JSON file which defines various properties of the app. Given a proper manifest, Chrome on Android will show a suggestion to the user to install the app on the home screen.

![](http://codewithstyle.info/wp-content/uploads/2017/04/Zrzut-ekranu-2017-04-14-o-12.00.43.png)

*   _Push notifications_ \- I wanted my app to send friend meeting reminders to the users so Push Notifications sounds great! Thanks to Service Workers and the [Push API](https://developer.mozilla.org/en/docs/Web/API/Push_API) it's possible for a web app to produce native-like notifications on Android phones. However, something has to send the notifications. I decided to use Firebase Cloud Messaging as a transport and Webtask for hosting the code that would actually invoke the notifications.

### Architecture

I wanted to develop my app fast and not spend time on setting up infrastructure so I decided to go for serverless architecture with Firebase Database instead of a dedicated backend. Firebase Database is a no-SQL, real-time database that web applications can directly connect to. It is tightly integrated with Firebase authentication and has a powerful validation rule engine. Therefore, it's possible to avoid having a separate application server. ![](http://codewithstyle.info/wp-content/uploads/2017/04/Architecture.png "Architecture") The app will consist of the following components:

*   The frontend written in Ionic 2 utilizing Service Workers for offline caching and receiving push notifications while the app is not in the foreground. It will also have a Web Manifest file to enable adding to the home screen.
*   The data will be stored in a Firebase Database. Validation rules will enforce that a user can only modify their own data. Firebase will also provide authentication.
*   A scheduled [Webtask](https://webtask.io/) (function-as-a-service) will run daily, read data from the Firebase Database and send out push notifications and email reminders. Firebase also offers a similar functionality (it's called Functions) but at the moment of writing they don't support scheduled execution.

If you are not that familiar with Firebase or Webtasks, take a look at [my previous post](http://codewithstyle.info/building-serverless-web-application-angular-2-webtask-firebase/) where I talk more about it. **Check out [Part 2: Ionic 2 and app manifest](http://codewithstyle.info/how-i-built-a-pwa-with-angular-and-firebase-part-2-ionic-2/)**