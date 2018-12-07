---
title: Understand pure functions with Angular pipes
url: 536.html
id: 536
categories:
  - Uncategorized
tags:
---

There is an interesting concept in functional programming called **pure functions**. It turns out that Angular takes advantage of it and it's actually essential to know if you plan to create custom **pipes**.

Change detection
----------------

In one of my previous posts I've explained [how change detection works in Angular](https://codewithstyle.info/change-detection-angular-versus-angularjs). Thanks to _zones_ Angular triggers change detection after every single asynchronous event fired within the application. Each component has its own change detector which will compare values used in property bindings with their previous values. In case of `ngFor` directive, change detector will look into the contents of the array used in the `let ... of` clause. Objects contained by the array after the event will be compared with objects contained by the array before the event. Thanks to this, changes to the elements in the array will be reflected in the DOM tree.

    @Component({
      selector: 'app-root',
      template: `
        <ul>
          <li *ngFor="let book of books">{ { book.title }}</li>
        </ul>
        <button (click)="changeTitle()">Change title</button>
      `
    })
    export class AppComponent {
      books = [
        { id: 1, title: "1984" },
        { id: 2, title: "Gone with the wind" },
        { id: 3, title: "Pride and prejudice" }
      ];
    
      changeTitle() {
        this.books[1].title = "Left hand of darkness";
      }
    }
    

The above example demonstrates this mechanism. Even though we are messing with internals of one of the array elements, the list will be updated.

Pipes
-----

Let's create a custom pipe that will implement showing only books with long titles.