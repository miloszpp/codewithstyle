---
title: Change detection in Angular versus AngularJS
url: 496.html
id: 496
categories:
  - Angular
  - Web
date: 2017-11-08 21:05:26
tags:
---

**Change detection** is the mechanism responsible for data binding in Angular. Thanks to it you don't need to manually manipulate the DOM tree. Instead, you can make changes to the model and they are automatically reflected in the view. In this post I'll try to briefly explain the differences of how change detection works in **AngularJS** (version 1.x) versus **Angular** (version 2+). Why should you care about this? It's generally a good idea to understand how the framework you're using works under the hood. The improvements in change detection contributed heavily to performance boost of Angular 2+. **Scroll to the bottom for a table summarizing major differences.**

When to detect changes?
-----------------------

Let's think about how the process of change detection could look like in big picture.

1.  The application starts up and models are initialized with some values. These values are also reflected in the view.
2.  The user interacts with the application. They click a button or type some text into an input field. An event is emitted.
3.  An event handler is called for that event. The code can update some property on the model.
4.  The model and the view are not in sync any more. Now is the time for the framework to detect changes and update the view accordingly.

![Change detection diagram](/images/2017/10/Change-detection-workflow.png) Now we know when change detection should be performed - whenever there is a chance that some model will be modified. In the above example it's a handler for a user-generated event that can make changes to the model. However, there are other situations that can result in model updates such as:

*   arrival of response to a HTTP request
*   execution of callback passed to `setInterval` function

Wrapping up, model changes (hence change detection) always start with some sort of asynchronous event.

Digest cycle vs zones
---------------------

We already know **when** change detection should be called. However, we don't know **how** the framework ensures that it will be called. This is the first of the differences between Angular and AngularJS. In AngularJS change detection was invoked by calling `$scope.digest()`. Most of the times we didn't have to call it manually because the framework did it for us. Directives such as `ngClick` or the `$http` service would make sure to run the digest cycle (i.e. change detection) after executing the handlers we've provided (that could potentially make changes to the model). The consequence of this was that in some rare cases you had to tell Angular to run the digest cycle by calling `scope.$digest` or `scope.$apply` manually. Angular takes a different, more robust approach. Instead of invoking CD manually, it takes advantage of a concept called **zones**. _Zone.js_ is a library which allows you to inject your own code into some low-level calls. In particular, it allows Angular to run change detection code automatically after an asynchronous event is handled.

Multiple passes vs single pass
------------------------------

Let's now focus on the change detection mechanism itself. It's actually quite simple - the framework keeps track of values in the model before and after running your event handlers. Change detection simply looks at these values and compares them. Once it finds some differences, it's able to tell that a change in the model has occurred. Such mechanism is called **dirty checking**.

Digest loop in AngularJS
------------------------

Whenever we use data binding or pass an expression to `ng-if` or `ng-for` we create a watcher. During the `$digest` call, AngularJS iterates over all watchers and compares old values of watched expressions with new values. The tricky part is that **AngularJS allows two-way data binding**. That means that after a change in the model is reflected in the view, it might turn out that the change in the view triggers another change in the model. Therefore, a single pass through the watcher array may not be enough. AngularJS repeats the process until it detects no more changes in the model. Unfortunately, it's quite easy to write code in a way that the model never stabilizes. What's more, having to iterate over the watchers so many times is not great in terms of performance.

    $scope.$watch('foo', function() {
      $scope.foo = $scope.foo + 1;
    });
    

Above you can see an example of infinite digest loop taken from Angular documentation.

Unidirectional data flow in Angular
-----------------------------------

Angular is smarter about this. It doesn't support two-way data binding any more. Therefore, it can assume that a single pass of change detection will always be sufficient. This concept has a name - it's called **unidirectional data flow**. It's unidirectional because the data always flow from the model to the template. It's not possible for a view change resulting from change detection to trigger a change in the model - this would be data flowing in the other direction. Hey, but isn't `[(ngModel)]` a two-way data binding? Actually, it's not. It's just a syntactic sugar for a simultaneous event binding and property binding. Importantly, it still works with single pass change detection. Unidirectional data flow is a major simplification when compared with AngularJS. It makes your application more predictable. It's also much better in terms of performance.

Dynamic vs static dirty checking
--------------------------------

Finally, AngularJS and Angular differ in how the dirty checking mechanism is implemented. In AngularJS the mechanism is dynamic. It means that watchers are created and added to the array during runtime, on demand. Angular takes a different approach. It generates a change detector for every single component. Such change detector is based on the template so it only compares the values of expressions used in property bindings. With [Ahead of Time compilation](https://codewithstyle.info/ahead-of-time-compilation-angular) enabled, Angular can generate change detectors during build time! Such change detectors are much easier to optimize for the JavaScript Virtual Machine than the dynamic mechanism used by AngularJS. We say that **change detection in Angular is VM-friendly**. This has major performance implications.

Summary
-------

In this article I've explained basics of change detection in Angular. Having understood this it's easier to understand some of the design choices taken in Angular versus AngularJS. To wrap up, here is a table that summarizes the major differences.

AngularJS

Angular

Change detection is invoked internally by the framework (by calling `$digest`). Sometimes it is necessary to trigger change detection manually.

Change detection is invoked automatically using zones. The framework hooks into internal browser API calls and performs CD after asynchronous events.

Two-way data binding is supported which means that single pass of change detection is not enough. AngularJS runs digest cycles until the model stabilizes.

Two-way data binding is not supported hence single pass of change detection is enough. Unidirectional data flow is enforced - the data flows from the model to the view, never the other way around.

Dirty checking is dynamic. Watchers are created at runtime.

Every component has its own custom change detector. With AOT enabled change detectors can be generated at build time. Change detection code is VM-friendly.