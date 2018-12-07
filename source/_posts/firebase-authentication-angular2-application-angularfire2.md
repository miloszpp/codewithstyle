---
title: Firebase authentication in Angular2 application with Angularfire2
tags:
  - angular2
  - firebase
  - serverless
url: 160.html
id: 160
categories:
  - Angular
  - Tutorials
  - Web
date: 2017-02-22 17:21:01
---

Implementing authentication in web apps is a tedious and repetitive task. What's more, it's very easy to do it wrong and expose security holes in our app. Fortunately, **Firebase Authentication** comes to rescue offering authentication as a service. It means that you no longer need to implement storage and verification of credentials, email verification, password recovery, etc. In this post I'll explain how to add email/password authentication to an Angular2 application. Site note: Firebase Authentication can be very useful when building a [serverless application](http://codewithstyle.info/building-serverless-web-application-angular-2-webtask-firebase/). For reference, here is a working example illustrating this article: [https://github.com/miloszpp/angularfire-sdk-auth-sample](https://github.com/miloszpp/angularfire-sdk-auth-sample).  

### Overview

Firebase offers two ways of implementing authentication:

*   **FirebaseUI Auth** \- a library of ready-to-use GUI components (such as login/registration forms, password recovery, etc.)
*   **Firebase Authentication SDK** \- a more flexible approach in which we need to implement above components ourselves; the role of Firebase is to store and verify user credentials; let's focus on this one

We'll implement three components:

*   **Register** component will show a registration form and will ask Firebase to create an entry for a user upon submission
*   **Login** component will show a login form and will ask Firebase to verify provided credentials upon submission
*   **Home** component will show the currently logged user (provided there is one)

We'll use the excellent [Angularfire2](https://github.com/angular/angularfire2) library. It provides an Angular-friendly abstraction layer over Firebase. Additionally, it exposes _authentication state_ as an **observable**, making it very easy for other components to subscribe to events such as login and logout.

### Preparations

To begin with, let's install Angularfire2 and Firebase modules:

npm install firebase angularfire2

Next, we need to enable email/password authentication method in the Firebase console. ![Firebase: enabling email/password authentication](http://codewithstyle.info/wp-content/uploads/2017/02/Zrzut-ekranu-2017-02-21-o-21.27.24-1024x414.png) Finally, let's load Angularfire2 in our **app.module.ts**:

import { AuthProviders, AuthMethods } from 'angularfire2';

const authConfig = {
    provider: AuthProviders.Password,
    method: AuthMethods.Password
};

@NgModule({
  declarations: \[ /* ... */ \],
  imports: \[
    AngularFireModule.initializeApp(config.firebase, authConfig),
    /\* ... */
  \],
  providers: \[\],
  bootstrap: \[AppComponent\]
})
export class AppModule { }

### Login component

Firstly, let's inject AngularFire into the component:

public model: LoginModel;
public authState: FirebaseAuthState;

constructor(public af: AngularFire) {
  this.model = { email: "", password: "" };
  this.af.auth.subscribe((auth) => {
    this.authState = auth;
  });
}

As you can see, this.af.auth  is an observable. It fires whenever an event related to authentication occurs. In our case, it's either logging in or logging out. FirebaseAuthState  stores information about currently logged user. Next, let's add two methods for logging in and logging out:

public submit() {
  this.af.auth.login(this.model).catch(alert);
}

public logout() {
  this.af.auth.logout();
}

As you can see, we simply propagate calls to the Angularfire2 API. When logging in, we need to provide email and password (encapsulated in model). Finally, we need some HTML to display the form:

<form (ngSubmit)="submit()" *ngIf="!authState">
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" class="form-control" id="email" name="email" placeholder="Email" \[(ngModel)\]="model.email" />
  </div>
  <div class="form-group">
    <label for="password">Password</label>
    <input type="password" class="form-control" id="password" name="password" placeholder="Password" \[(ngModel)\]="model.password" />
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
</form>
<div *ngIf="authState">
  Logged as { { authState.auth.email }}
  <button (click)="logout()">Logout</button>
</div>

The form is only visible when the user is not logged in (authState  will be undefined). Otherwise, we show the user name and the logout button.

### Register component

We've allowed our users to logged in but so far there are no registered users! Let's fix that and create a registration component. Firstly, we need to inject the AngularFire service just like we did in the login controller. Next, let's create a method to be called when the user provides his registration details:

public submit() {
  this.af.auth.createUser(this.model).then(() => /* success! */, alert);
}

Finally, here goes the HTML form:

<form (ngSubmit)="submit()">
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" class="form-control" id="email" name="email" placeholder="Email" \[(ngModel)\]="model.email" />
  </div>
  <div class="form-group">
    <label for="password">Password</label>
    <input type="password" class="form-control" id="password" name="password" placeholder="Password" \[(ngModel)\]="model.password" />
  </div>
  <button type="submit" class="btn btn-default">Register</button>
</form>

### Summary

In this tutorial I showed you how to take advantage of Firebase Authentication and use it in an Angular 2 application. This example doesn't exploit the full potential of Firebase Authentication - it can also do email verification (with actual email sending and customizable email templates), password recovery and logging in with social accounts (Facebook, Twitter, etc.). I will touch on these topics in the following articles. Let me know if you have any feedback regarding this post - feel free to post a comment!