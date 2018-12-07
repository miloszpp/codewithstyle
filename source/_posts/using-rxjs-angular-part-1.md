---
title: 'Using RxJS with Angular: part 1'
url: 482.html
id: 482
categories:
  - Angular
  - Web
date: 2017-10-16 20:05:42
tags:
---

Recently I wrote about using RxJS as part of my [Functional Programming in JavaScript course](https://codewithstyle.info/functional-programming-javascript-plain-words/). Let's see how to combine RxJS with the Angular framework. ![RxJS and Angular](/images/2017/10/angular-rxjs.png) As a quick reminder, [RxJS](https://github.com/Reactive-Extensions/RxJS) is a functional-reactive programming JavaScript library. It helps you write maintainable, readable code by allowing you to express your application as a manipulation on streams of events. If you have no experience with RxJS, I recommend you to read the [part of my course dedicated to it](https://codewithstyle.info/functional-javascript-part-8-functional-reactive-programming-rxjs/).

### Angular support for RxJS

It's especially easy to overlook RxJS support in Angular when you are coming from the AngularJS (1.x) background where asynchrony was based on promises. For example, the $http  service in AngularJS returned a Promise which would resolve once the remote server responded to our request. In Angular (2+) it's still possible to work in exactly the same way. The HttpClient service (Http service before Angular 4.3) returns Observables. However, Observables are easily convertible to Promises with the toPromise operator. In some cases, that's ok. However, there are cases where we can benefit from sticking to Observables. What does it actually mean that HttpClient returns an Observable? An Observable represents a stream of events. Given a callback (provided as an argument to subscribe call) it will run it every time a new event is produced. When we make a remote server call our "stream of events" contains only a single event - the response coming back from the server. It's therefore a very specific kind of Observable, but an Observable nonetheless.

### Async pipe

Besides having Observables baked into the API, Angular also supports consuming them with the async pipe. Let's see an example.

    @Component({
      selector: 'app-root',
      template: `
        <ul>
          <li *ngFor="let post of posts | async">{ {post.title}}</li>
        </ul>
      `
    })
    export class AppComponent implements OnInit {
      posts: Observable<Post[]>;
    
      constructor(private httpClient: HttpClient) { }
    
      ngOnInit(): void {
        this.posts = this.httpClient.get<Post[]>("https://jsonplaceholder.typicode.com/posts");
      }
    }
    
    interface Post {
      title: string;
      body: string;
    }

Pay attention to the template. We are binding to the bands property. Since it's an Observable we can't bind to it directly. However, the AsyncPipe comes to rescue. If it weren't for the AsyncPipe, we would need to manually subscribe to the Observable. What's more, we would need to remember to unsubscribe from it when the component is destroyed.

### Issues with AsyncPipe

You need to be careful when using the Async pipe, though. Let's have a look at the following example.

    @Component({
      selector: 'app-root',
      template: `
        <p>{ { (post | async)?.title }}</p>
        <p>{ { (post | async)?.body }}</p>
      `
    })
    export class AppComponent implements OnInit {
      post: Observable<Post>;
    
      constructor(private httpClient: HttpClient) { }
    
      ngOnInit(): void {
        this.post = this.httpClient.get<Post>("https://jsonplaceholder.typicode.com/posts/1");
      }
    }

Here we are fetching a single post and want to display it's details. Hence, we use the AsyncPipe  twice. Surprisingly, if we check the Network tab in our browser's developer tools, we will discover that two HTTP requests have been made instead of one. ![](/images/2017/10/AngularRxjsExample.png) To explain this we need to understand the difference between hot and cold observables. **Cold observables** are the ones that "trigger" the observed stream when they are being subscribed to. HttpClient returns cold observables. We are using the AsyncPipe twice here which invokes the subscription twice. In consequence, an HTTP request is fired twice. On the other hand **hot observables** are ones that are already "triggered". When we subscribe we will only see new events. The fact of subscribing has no side effects. It's possible to fix our problem here by converting the cold observable to a hot one. However, it has its drawbacks too. The best option is actually to use good old Promises.

    this.post = this.httpClient
        .get<Post>("https://jsonplaceholder.typicode.com/posts/1")
        .toPromise();

### Summary

In this post you have seen how to deal with Observables returned by the HttpClient service in Angular. I have also shown that it sometimes makes sense to fall back to plain Promises. In the next part we will unleash the real power of RxJS by combining our Observable with another event stream.