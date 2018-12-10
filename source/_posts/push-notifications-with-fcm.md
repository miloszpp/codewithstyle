---
title: >-
  How I built a PWA with Angular and Firebase Part 3: Push Notifications with
  Firebase Cloud Messaging
url: 246.html
id: 246
categories:
  - Angular
  - Tutorials
  - Web
date: 2017-05-28 13:50:33
tags:
  - angular
  - pwa
  - firebase
---

This post is the last post of the series in which I will describe how I built my first PWA, [Friendtainer](http://friendtainer.com/). It will touch on many topics such as Angular 2, Ionic 2, Firebase, service workers, push notifications, serverless architectures. I hope you will find it useful when building your own PWAs. In this part of the series, we’ll look at the most interesting part of the app - push notifications and Firebase Cloud Messaging.

*   [Part 1: Explaining PWA and introduction](https://codewithstyle.info/how-i-built-a-progressive-web-app-with-angular-and-firebase-part-1/)
*   [Part 2: Ionic and app manifest](https://codewithstyle.info/how-i-built-a-pwa-with-angular-and-firebase-part-2-ionic-2/)
*   [Part 3: Push notifications with Firebase Cloud Messaging](https://codewithstyle.info/push-notifications-with-fcm/)

### Recap of the architecture

In [Part 1](http://codewithstyle.info/how-i-built-a-progressive-web-app-with-angular-and-firebase-part-1/) of this series I described the architecture of my app. Let's focus on the part related to push notifications. We will use Firebase Cloud Messaging to implement notifications. Why do we need it? As the name suggests, push notifications are being **pushed** directly to users' devices. If we wanted to implement it ourselves, we would need to somehow figure out how to find the device, connect to it and send data to it. FCM can do this work for us. We can tell it what's the message and who to deliver it to and it will take care of the delivery. 

![](/images/2017/05/maxresdefault-1024x576.jpg) 

However, we still need something to actually send the message (i.e. to tell FCM what and to whom should be delivered). FCM refers to the piece that pushes messages as **application server**. Our setup is **serverless** so we will not use a single, centralized server for pushing the notifications. Instead, we will use a function-as-a-service offering called [Webtask.io](https://webtask.io/). Webtask lets us run a piece of JavaScript code in the cloud, without having to care about where and how it's executed. _BTW, you can use Firebase Cloud Functions instead of Webtask. I decided to use Webtask because when I was working on Friendtainer, Functions were not available yet._ 

Friendtainer has a single webtask that's responsible for determining which users need to be shown a reminder. It's automatically started every 24 hours. For each user that needs to be shown a reminder, the webtask tells FCM to deliver a notification to that user. FCM finds the user's device and sends the notification. Since the device has a service worker installed and running in the background, it handles the notification and shows a notification banner. 

As I said, FCM takes care of message delivery to a specific device. However, we are responsible for pairing of users and devices. Although a user can have multiple devices (especially with a PWA that can be run on desktop, smartphone, tablet, etc.), we are going to assume one device per user - it simplifies things by a lot. There are three pieces required to get this working:

*   The client app - it will handle subscribing to notifications and associating FCM token with the user
*   The service worker (part of the client app) - will handle incoming messages and display the notifications
*   The webtask - will send messages

###  ![](/images/2017/05/Architecture.png)

### Service worker

Notifications only make sense if you are able to receive them even if the application is not in the foreground. This is possible to achieve in modern web applications thanks to service workers. Service worker is a piece of JavaScript code which can run in the background, independently of the website which loaded it. Currently, service workers are used mostly for two things:

*   providing caching and offline access
*   push notifications

If you are using Ionic, it has already generated the service worker for you. You will find that it's not empty. Ionic sets up offline caching of all static resources for you. Let's setup handling messages from GCM in our service worker:

```javascript
importScripts('https://www.gstatic.com/firebasejs/3.5.2/firebase-app.js');
importScripts('https://www.gstatic.com/firebasejs/3.5.2/firebase-messaging.js');

firebase.initializeApp({
  // get this from Firebase console, Cloud messaging section
  'messagingSenderId': '***' 
});

const messaging = firebase.messaging();

messaging.setBackgroundMessageHandler(function(payload) {
  console.log('Received background message ', payload);
  // here you can override some options describing what's in the message; 
  // however, the actual content will come from the Webtask
  const notificationOptions = {
    icon: '/assets/images/logo-128.png'
  };
  return self.registration.showNotification(notificationTitle, notificationOptions);
});
```

### Subscribing to notifications

Next, we need to extend the user interface of our app to allow users to register their intent to receive push notifications. I've created the below service.

```typescript
@Injectable()
// based on https://github.com/firebase/quickstart-js/
export class MessagingService {
  private messaging: firebase.messaging.Messaging;
  private unsubscribeOnTokenRefresh = () => {};

  constructor(
    @Inject(FirebaseRef) fb: any,
    private userService: UserService
  ) {
    this.messaging = fb.messaging();
  }

  public enableNotifications(): firebase.Thenable<any> {
    console.log('Requesting permission...');
    return this.messaging.requestPermission().then(() => {
        console.log('Permission granted');
        // token might change - we need to listen for changes to it and update it
        this.setupOnTokenRefresh();
        return this.updateToken();
      });
  }

  public disableNotifications(): firebase.Thenable<any> {
    this.unsubscribeOnTokenRefresh();
    this.unsubscribeOnTokenRefresh = () => {};
    return this.userService.removeFcmToken().then();
  }

  private updateToken(): firebase.Thenable<any> {
    return this.messaging.getToken().then((currentToken) => {
      if (currentToken) {
        // we've got the token from Firebase, now let's store it in the database
        return this.userService.setFcmKey(currentToken);
      } else {
        console.log('No Instance ID token available. Request permission to generate one.');
      }
    });
  }

  private setupOnTokenRefresh(): void {
    this.unsubscribeOnTokenRefresh = this.messaging.onTokenRefresh(() => {
      console.log("Token refreshed");
      this.userService.removeFcmToken().then(() => { this.updateToken(); });
    });
  }
    
}
```

Let me explain how it works. On the high level, our application needs to call Firebase in order to get a **registration token**. This token identifies the target that notifications will be pushed to. By target, I mean specific browser on a specific device. If you request permissions from Firefox and Chrome both installed on the same machine, you will get two different registration tokens. 

![](/images/2017/05/Messaging-workflow.png "Messaging workflow")   

When you call enableNotifications, the library checks if the browser permits showing notifications. If this is not the case, the user will be asked whether he wishes to receive notifications. The promise will succeed only if he accepts. Next, we call setupOnTokenRefresh  in order to handle the situation when the token is updated. Finally, we call userService.setFcmKey() . You should implement the UserService  yourself and this particular method should simply store the token in some user profile object (in the database). 

Basically, we need to associate users to FCM tokens beacause when sending a message, we can't specify the targeted user but only his FCM token. You should consider associating multiple tokens with single user. If you want to be able to push messages to all devices owned by a user, you need to associate all of them with the user's profile. Note that we also implemented disableNotifications . This method removes the association between the current user and the FCM token. 

**You should consider calling this method when the user is logging out of the application** \- otherwise, you may end up in a situation when some other user using the same device receives notifications targeted to the other user. Lastly, remember to pass messagingSenderId  when initializing Firebase in the app and add the following values in your manifest file:

```
"gcm\_sender\_id": "103953800507", // this value is always the same
"gcm\_user\_visible_only": true
```

### Sending notifications

The final missing piece is to send the notification. In Friendtainer it's the responsibility of a Webtask which is triggered on some schedule. You can use any FaaS provider for that (using Firebase Cloud Functions might be a good choice since than you will stay within the Firebase ecosystem). The actual sending is pretty easy. Let's see an example.

```typescript
const fcmOptions = {
    method: "POST",
    url: "https://fcm.googleapis.com/fcm/send",
    // get the key from Firebase console
    headers: { Authorization: \`key=${fcmServerKey}\` }, 
    json: {
        notification: { 
            title: "Message title",
            body: "Message body",
            click_action: "URL to your app?"
        },
        // userData is where your client stored the FCM token for the given user
        // it should be read from the database
        to: userData.fcmRegistrationKey
    }
};
return request(fcmOptions).catch(() => console.log(\`ERROR: Push failed for user ${userData.email}\`));
```

We need to retrieve the association between the user and the token from the database first. In the above code, it's stored in the userData  object. Next, we build an object describing what should go into the message and who to send it to. Finally, we make a simple web request to Firebase Cloud Messaging. It will then route our message and deliver it to the desired device.

### Summary

In this series I talked about building a mobile app using the latest greatest offerings of the PWA approach. We've covered how to make a web app look like a mobile app, make the web app _installable_ on a mobile device and how to implement web push notifications which behave exactly as native notifications. I think given all these features the web platform is now ready to compete with native applications and that in the long run we will see the demise of many mobile apps.