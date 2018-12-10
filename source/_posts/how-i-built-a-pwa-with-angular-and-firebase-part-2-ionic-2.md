---
title: 'How I built a PWA with Angular and Firebase Part 2: Ionic and app manifest'
url: 250.html
id: 250
categories:
  - Angular
  - Web
date: 2017-05-09 18:47:22
tags:
  - angular
  - firebase
  - pwa
  - ionic
---

This post is part of a series in which I will describe how I built my first PWA, [Friendtainer](http://friendtainer.com). It will touch on many topics such as Angular 2, Ionic 2, Firebase, service workers, push notifications, serverless architectures. I hope you find it useful when building your own PWAs. In this part of the series, we'll look at how to create a PWA app using the Ionic 2 framework. Additionally, I will talk about the app manifest and how to use it to allow adding the app icon to the device's home screen.

*   [Part 1: Explaining PWA and introduction](https://codewithstyle.info/how-i-built-a-progressive-web-app-with-angular-and-firebase-part-1/)
*   [Part 2: Ionic and app manifest](https://codewithstyle.info/how-i-built-a-pwa-with-angular-and-firebase-part-2-ionic-2/)
*   [Part 3: Push notifications with Firebase Cloud Messaging](https://codewithstyle.info/push-notifications-with-fcm/)

Overview
--------

As I mentioned in the [previous post](http://codewithstyle.info/how-i-built-a-progressive-web-app-with-angular-and-firebase-part-1/), there are a couple of requirements that a web app has to comply with in order to be called a PWA. In this post we'll focus on the items in bold.

*   **Looks like a native app on mobile devices**
*   Loads fast and supports offline mode
*   **Can be launched from the phone’s home screen**
*   **Runs full-screen**
*   May support push notifications (so the phone will display notifications even if the browser is not running)

Ionic
-----

Ionic is a web/mobile application framework based on Angular. It's primary goal is to facilitate building hybrid mobile applications. Hybrid mobile apps are apps consisting of two parts - a native wrapper and a web app. The wrapper acts as a bridge between the web app and the device, providing an API to native functionalities. Although we are not going to build a hybrid app but a PWA, we can still benefit from Ionic. It contains a rich library of UI components which help us build a web app that looks like a native app. What's more, Ionic detects the platform it's running on and the components look different on iOS, Android and Windows Phone. 

![](/images/2017/05/Zrzut-ekranu-2017-05-09-o-20.25.18.png)
**Friendtainer on Android**

![](/images/2017/05/Zrzut-ekranu-2017-05-09-o-20.24.38.png) 
**Friendtainer on iPhone**

Ionic comes with it's own command line tool which you can use to generate the project skeleton as well as particular pages.

Getting started with Ionic
--------------------------

In order to get started with Ionic you need to download it first:

```
npm install -g ionic
```

Now you can use the ionic  command line tool. Run this command to generate the project skeleton:

```
ionic start myApp sidemenu
```

Now you can start adding Ionic pages like this:

```
ionic generate page Contacts
```

Ionic page is basically an Angular component with some added functionality. Every Ionic app has a root component called AppComponent  which hosts the navigation and the current page. Pages form a stack where the topmost page is the one that's currently visible to the user. It's super-easy to create pages using the rich component library provided in Ionic. The [documentation](https://ionicframework.com/docs/components/#overview) is great, providing live examples which demonstrate the look and feel on all supported platforms. Ionic make it very convenient to develop and test your app. Just run the following command to run it locally in development mode:

```
ionic serve
```

Building production package
---------------------------

Once your app is ready you need to deploy it. You need to build a package that contains optimized versions of all the JavaScript files and static assets. Ionic's build process is geared towards supporting different target platforms such as iOS or Android. This only applies to hybrid apps. In our case, we want to produce a package that has no dependency on the bridging API. For that we need to specify browser  as the target platform when building the package. Before deploying your app to production, it's highly recommended to optimize it. Optimization involves bundling and minifying the JavaScript source code, running Angular AOT (Ahead Of Time) compilation and setting the app in prod mode. As a result, the optimized package is much smaller and has much better performance. In order to build the app with the optimizations, use the prod  flag:

```
ionic build --prod browser
```

The build process can take some time. Once it's done, you can find your package inside the platforms/browser  directory. It's just a bunch of static files so you can just drop it on a HTTP server. Or, use Firebase hosting as I did.

App manifest
------------

So far, Ionic helped us deal with providing native-like experience inside the app. How about some elements external to the app? Specifically, I mean home screen integration and the full-screen mode. The web app manifest specification allows us to control those aspects of the user experience. Manifest file is basically a JSON file containing some metadata about your application. We can use this file to specify the name of the app, the icon to be used on the home screen and whether it should run full screen or not. Once we provide all this information, some modern browsers (including Chrome) will display a prompt suggesting the user visiting your app to add an icon for it to the home screen. 

![](/images/2017/04/Zrzut-ekranu-2017-04-14-o-12.00.43.png) 
**Chrome home screen prompt**

Ionic apps created using the CLI tool already include a manifest file so you just need to modify it. You can find it under the following path: src/manifest.json Below you can find the manifest file for Friendtainer. As you can see, it contains the app title, references to the icon image in several sizes and some info about colors.

```
{
  "name": "Friendtainer",
  "short_name": "Friendtainer",
  "start_url": "index.html",
  "display": "standalone",
  "icons": \[{
    "src": "assets/images/logo-500.png",
    "sizes": "500x500",
    "type": "image/png"
  },{
    "src": "assets/images/logo-256.png",
    "sizes": "256x256",
    "type": "image/png"
  },{
    "src": "assets/images/logo-128.png",
    "sizes": "128x128",
    "type": "image/png"
  }\],
  "background_color": "#FFFFFF",
  "theme_color": "#68A9FF",
  "gcm\_sender\_id": "103953800507",
  "gcm\_user\_visible_only": true
}
```

Full-screen mode is achieved by setting the display  option to standalone. Android phones will also provide a splash screen for your app based on the settings found in the manifest file. The splash screen is composed of the app title and the app image. Background and theme colors are also used when showing the splash screen.

![](/images/2017/05/Zrzut-ekranu-2017-05-09-o-20.34.18.png) 
**Splashscreen based on the above manifest** 

The remaining two settings starting with the gcm  prefix are related to Google Cloud Messaging. I will describe them in more detail once we discuss push notifications. **To be continued...**