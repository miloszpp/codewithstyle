---
title: 'Building serverless web application with Angular 2, Webtask and Firebase'
tags:
  - angular
  - firebase
  - serverless
  - webtask
url: 121.html
id: 121
categories:
  - Angular
  - Best Of
  - Tutorials
  - Web
date: 2016-12-27 17:46:43
---

Recently I've been playing a lot with my pet project [Tradux](http://codewithstyle.info/tradux/). It is a simple trading platform simulator which I built as an exercise in Redux, event sourcing and serverless architectures. I've decided to share some of the knowledge I learned in the form of this tutorial. 

We're going to build (guess what?) a TODO app. The client (in Angular 2) will be calling a Webtask whenever an event occurs (task created or task marked as done). The Webtask will update the data in the Firebase Database which will be then synchronized to the client. Webtask is function-as-a-service offering which allows you to run pieces of code on demand, without having to worry about infrastructure, servers, etc. - i.e. serverless. 

![Architecture](/images/2016/12/Architecture.png "Architecture") 

The full source code is available on [Github](https://github.com/miloszpp/serverless-todo). 

**UPDATE:** recently I gave a talk on this topic during [#11 AngularJS Warsaw meetup](https://www.meetup.com/AngularJS-Warsaw/events/237216962/). During the talk I built a silghtly different demo application which additionally performs spell checking in the webtask. Check out the [Github repo](https://github.com/miloszpp/angularjs-warsaw-serverless) for the source code.

### Project skeleton

Let's start with a very simple client in Angular 2. We will use Angular CLI to scaffold most of the code.

```
npm install -g angular-cli
ng new serverless-todo
```

It takes a while for this command to run and it will install much more stuff than we need, but it's still the quickest and most convenient way to go. Let's create a single component.

```
cd serverless-todo
ng generate component tasks
```

Now, let's create the following directory structure. We'd like to share some code between the client and the webtask so we will put it in common directory.

```
src
|--app // the Angular 2 application
|--common // stuff shared between the client and webtasks
|  |--config.ts // Firebase and Webtask connection details
|  |--config-secret.ts // secret parts of our config - careful not to share this
|  `--model.ts // common interface definitions
`--webtasks // source code for webtasks
   |--add-task.ts // there will be only one webtask - it will be used for adding items to our list
```

Let's start with defining the following interfaces inside model.ts. The first one is a command that will be sent from the client to the webtask. The second one is the entity representing an item on the list that will be stored in the database.

```typescript
export interface AddTaskCommand {
    content: string;
}

export interface Task {
    content: string;
    created: number;
}
```

Finally, remember to add the Tasks component to app.component.html :

```html
<app-tasks></app-tasks>
```

### Adding Firebase to the Client

Before we proceed, you need to create a [Firebase](https://firebase.google.com/) account. Firebase is a cloud platform which provides useful services for developing and deploying web and mobile applications. We will focus on one particular aspect of Firebase - the Realtime Database. The Realtime Database is a No-SQL storage mechanism which supports automatic synchronization of clients. In other words, when one of the clients modifies a record in the database, all other clients will see the changes (almost in real-time). Once you created the account, let's modify the database access rules. By default, the database only allows authenticated users. We will change it to allow anonymous reads. You can find the Rules tab once you click on the Database menu item.

```
{
  "rules": {
    ".read": true,
    ".write": "auth != null"
  }
}
```

Firebase provides a generous limit in the free Spark subscription. Create an account and define a new application. Once you are done, put the following definition in config.ts :

```typescript
export const config = {
    addTaskUrl: "",
    firebase: {
        apiKey: "<<YOUR API KEY>>",
        authDomain: "<<YOUR FIREBASE PROJECT ID>>.firebaseapp.com",
        databaseURL: "https://<<YOUR FIREBASE PROJECT ID>>.firebaseio.com/",
        storageBucket: ""
    }
};
```

If you cannot find your settings, here is a helper for you. If you are really lazy, you can use the following settings, although I cannot guarantee any availability. ![firebase2](/images/2016/12/firebase2.png) Let's now add Firebase to our client. There is an excellent library called AngularFire2 which we are going to use. Run the following commands:

```
npm install --save firebase
npm install --save angularfire2
```

Modify the imports section of AppModule  inside app.module.ts  so that it looks like this (you can import AngularFireModule  from angularfire2  module):

```typescript
imports: \[
    BrowserModule,
    FormsModule,
    HttpModule,
    AngularFireModule.initializeApp(config.firebase)
  \],
```

Now you can inject AngularFire object to Tasks component (tasks.compontent.ts ):

```typescript
  public tasks: FirebaseListObservable<Task\[\]>;

  constructor(public http: Http, angularFire: AngularFire) {
    this.tasks = angularFire.database.list('tasks');
  }
```

You will also need some HTML to display tasks. I will include the form for adding tasks as well (tasks.component.html ):

```html
<h1>TODO list</h1>
<div>
  <form>
    <div class="form-group">
      <label for="content">Task</label>
      <input class="form-control" type="text" name="content" #content />
    </div>
    <button type="button" class="btn btn-default" (click)="addTask(content.value)">Add</button>
  </form>
</div>
<div>
  <ul>
    <li *ngFor="let task of tasks | async">
      { { task.content }} <small>{ { task.created | date }}</small>
    </li>
  </ul>
</div>
```

Our client is ready to display tasks, however there are no tasks in the database yet. Note how we can bind directly to FirebaseListObservable \- Firebase will take care of all the updates for us.

### Creating the Webtask

Now we need to create the Webtask responsible for adding tasks to the list. Before we continue, please create an account on [webtask.io](http://webtask.io/). Again, you can use it for free for the purposes of this tutorial. The website will ask you to run the following commands:

```
npm install wt-cli -g
wt init <<YOUR EMAIL>>
```

Creating Webtasks is amazingly easy. You just need to define a function which takes a HTTP context and a callback to execute when the job is done. Paste the following code inside webtasks/add-task.ts :

```javascript
import { config } from '../common/config';
import { firebaseSecret } from '../common/config-secret';
import { AddTaskCommand, Task } from '../common/model';

var request = require('request');

module.exports = function (context, callback) {
    const tasksUrl = `${config.firebase.databaseURL}/tasks.json?auth=${firebaseSecret}`;
    const command = <AddTaskCommand>context.body;
    console.log(\`Received command: ${JSON.stringify(command)}\`);

    const task: Task = {
        content: command.content,
        created: Date.now()
    };

    const requestOptions = {
        method: 'POST',
        url: tasksUrl,
        json: task
    };

    request(requestOptions, () => callback(null, "Finished"));
}
```

The above snippet parses the request body (note how it uses the same AddTaskCommand  interface that the client). Later, it creates a Task  object and calls Firebase via the REST API to add the object to the collection. You could use the Firebase Javascript client instead of calling the REST API directly, however I couldn't get it working in the Webtask environment. Obviously in a production app you would perform validation here. Note that you need to define the firebaseSecret  constant. You can find the private API key here: 

![zrzut-ekranu-2016-12-27-o-18-24-37](/images/2016/12/Zrzut-ekranu-2016-12-27-o-18.24.37-1024x566.png) 

Firebase complains that this is a legacy method but it's simply the quickest way to do that. Why do we need to pass the secret now? That's because we defined a database access rule which says that anonymous writes are not permitted. Using the secret key allows us to bypass the rule. Obviously, in a production app you would use some proper authentication. We are ready to deploy the Webtask. A Webtask has to be a single JavaScript file. Ours is TypeScript and it depends on many other modules. Fortunately, Webtask.io provides a bundler which can do the hard work for us. Install it with the following command:

```
npm i -g webtask-bundle
```

Now we can compile the TypeScript code to JavaScript, then run the bundler to create a single file and then deploy it using the Webtask CLI:

```
tsc add-task.ts && \
wt-bundle --output add-task-bundle.js add-task.js && \
wt create add-task-bundle.js && \
wt ls
```

Voila, the Webtask is now in the wild. The CLI will tell you its URL. Copy it and paste inside config.ts:

```typescript
export const config = {
    addTaskUrl: "<<YOUR WEBTASK URL>>",
    ...
}
```

### Calling the Webtask from the Client

There is just one missing part - we need to call the Webtask from the client. Go to the Tasks component and add the below method:

```typescript
  addTask(content: string) {
    const command: AddTaskCommand = { content };
    console.log(\`Adding task: ${JSON.stringify(command)}\`);
    this.http.post(config.addTaskUrl, command).subscribe(
      () =\> console.log('Success'),
      error => alert(\`Adding task failed with error ${error}\`)
    );
  }
```

This function is already linked to in HTML. Now, run the following command in console and enjoy the result!

```
ng serve
```

### Summary

In this short tutorial I showed how to quickly build a serverless web application using Webtasks. Honestly, you achieve the same result without the Webtasks and by talking directly to Firebase from the Client. However, having this additional layer allows you to perform complex validation or calculations. See [Tradux](http://codewithstyle.info/tradux/) for a nice example of a complex Webtask. You can very easily use Firebase to deploy your app.