---
title: 'Using RxJS with Angular: part 2'
url: 488.html
id: 488
categories:
  - Angular
  - Web
date: 2017-10-30 20:14:40
tags:
---

In the previous post about Angular and RxJS we discussed the AsyncPipe and how it can be used to consume Observables in Angular templates. ![](/images/2017/10/angular-rxjs.png) This time we will focus on the essence of functional-reactive programming. Let's see how we can reinvent the way we look at how data flows in our program. Check out the [first part](https://codewithstyle.info/using-rxjs-angular-part-1/) of this article where I explain the AsyncPipe. _Credits to [Piecioshka](https://piecioshka.pl/blog/) whose questions inspired me to look into RxJS in more detail!_

Creating your own Observables
-----------------------------

So far we've only seen one kind of Observable - the one returned by the HttpClient. As mentioned, it's not a particularly exciting Observable. It would only contain a single event - the arrival of HTTP response from the server. However, one may argue that there are many other streams of events out there in a GUI application. Consider a button with some click handler. A sequence of button clicks can be looked at as at a stream of events - hence, an Observable. It's very easy to create an observable from a button in Angular:

    @Component({
      selector: 'app-click-stream',
      template: `
        <button (click)="clickStream.next()">Fetch data</button>
      `
    })
    export class ClickStreamComponent implements OnInit {
      clickStream: Observable<any> = new Subject<any>();
    
      constructor() { }
    
      ngOnInit() {
        this.clickStream.subscribe(() => {
          console.log("Button clicked!");
        });
      }
    }

We've created a Subject and call next on it after every click. This makes subscribers run the handler on every click. The type of clickStream is Observable<any> because we're not interested in any data associated with the click but merely with the click itself. Nothing exciting here so far. We could've accomplished exactly the same with a regular event handler.

Combining Observables
---------------------

As button caption suggests, let's fetch some data whenever the button is clicked. We could do it this way:

    ngOnInit() {
      this.clickStream.subscribe(() => {
        this.httpClient.get<Post[]>("https://jsonplaceholder.typicode.com/posts")
          .subscribe(posts => {
            this.posts = posts;
          });
      });
    }

But there are issues with this approach. It doesn't allow us to take advantage of the AsyncPipe we've talked about in the previous post. Besides, we should take care about unsubscribing of all the Observables created with button clicks. What we would much prefer is to somehow combine the click stream with the inner Observable. It turns out that RxJS lets us do exactly this!

    import "rxjs/add/operator/mergeMap";
    
    @Component({
      selector: 'app-click-stream',
      template: `
        <button (click)="clickStream.next()">Fetch data</button>
        <span *ngFor="let post of postStream | async">{ { post.title }}</span>
      `
    })
    export class ClickStreamComponent {
      private url = "https://jsonplaceholder.typicode.com/posts";
    
      clickStream: Observable<any> = new Subject<any>();
      postStream: Observable<Post[]>;
    
      constructor(private httpClient: HttpClient) {
        this.postStream = this.clickStream.mergeMap(() => this.httpClient.get<Post[]>(this.url));
      }
    }

We've used the mergeMap operator. It takes a function that for each value produced by an Observable creates a new Observable. It then merges all the resulting Observables into a single one which we can now safely use with the AsyncPipe! ![](/images/2017/10/mergeMap.png "mergeMap")   On each click our clickStream Observable produces an (empty) value. We take this value and call httpClient.get which gives us an Observable that will produce a single value. If we merge these Observables we will get a stream of values returned from the server.

Why do this?
------------

I hope you agree that the mergeMap approach is much more readable and concise than the nested subscribe approach. However, this is not the only benefit. Having our data fetching mechanism represented a single observable allows us to use the whole arsenal of RxJS operators. A very common use case would be to make some sort of safe guard that would prevent the user from firing bazillion of HTTP requests by clicking furiously on the button. With RxJS it's super easy!

    // import "rxjs/add/operator/debounceTime";
    
    this.postStream = 
          this.clickStream
            .debounceTime(500)
            .mergeMap(() => this.httpClient.get<Post[]>(this.url));
    

The debounceTime operator waits 500 milliseconds after each button click. If there is a new click coming within this timespan than it drops the previous one. Thanks to that we will only make the request for the last click. Imagine implementing this without RxJS... Another approach would be to use the switchMap operator.

    this.postStream = 
          this.clickStream.switchMap(() => 
            this.httpClient.get<Post[]>(this.url));
    

It works like mergeMap but if there is a new click while the previous request is still not resolved than the previous request will be dropped (cancelled).

Summary
-------

I wanted to show you that with RxJS we can change the way we think about the data flow in Angular applications. Doing this allows us to make use of some interesting RxJS operators but it also lets us eliminate mutable state from our component. Such components are easier to maintain and harder to break.