---
title: Creating Angular NPM packages with ng-packagr
url: 604.html
id: 604
categories:
  - Angular
  - Web
date: 2018-01-25 22:03:28
tags:
---

In this short post, I would like to share some of the experiences I've had when creating an NPM Angular package using `ng-packagr`. ![](/images/2018/01/Change-detection-in-AngularJS-vs-Angular-—-kopia-—-kopia.png)

Angular Package Format and `ng-packagr`
---------------------------------------

First of all, when creating an Angular library, it's important to understand that you need to include a bit more than just plain TypeScript files inside the NPM package. It should contain JavaScript code that is ready to be consumed in various ways - e.g. by Webpack, Rollup or Angular CLI. There are some best-practices describing what to include in an Angular NPM package - it's called [Angular Package Format](https://docs.google.com/document/d/1CZC2rcpxffTDfRDs6p1cfbmKNLA6x5O-NtkJglDaBVs/preview). It's actually not trivial to produce a package that complies with the standard. Fortunately, we can use the excellent [ng-packagr](https://www.npmjs.com/package/ng-packagr) tool for that. The documentation for the package is pretty good and it's actually quite easy to get going with it.

AOT metadata generation issue
-----------------------------

It worked pretty well for me too, except for two issues on which I've spent too much time. Both of them only manifested themselves when I tried to build the consuming project (the project to which my new common package is a dependency) using Angular Ahead of Time compilation (AOT). When trying to build my consumer project I've got the following error:

    ERROR in Error: Unexpected value 'SomeModuleName in ...........path-to-module.d.ts' imported by the module 'AppModule in ...............path-to-module.ts'. Please add a @NgModule annotation.
    

After some digging, I've found out that the root cause was that when building my package I wasn't generating appropriate metadata to be further used by the AOT compiler. This should be normally done by `ng-packagr`. It turned out that the tool could not generate the metadata properly because I was using invalid paths inside the `public_api.ts` file. Inside my package, I was making heavy use of `index.ts` files where I re-exported all the relevant symbols from the module. The file structure looked like this:

    + src
    |--+ module1
    |  |-- module1.module.ts
    |  |-- index.ts
    |  + module2
    |  |-- module2.module.ts
    |  └-- index.ts
    └ public_api.ts
    

And the `public_api.ts` was referencing the modules like this:

    import * from './src/module1';
    import * from './src/module2';
    

It turns out that this is not enough to generate the metadata properly. I had to change the imports to include `index`.

    import * from './src/module1/index';
    import * from './src/module2/index';
    

After this change, the AOT compilation started to work properly. The only place where I could find this information was a comment by **JoeQueR** on [GitHub](https://github.com/dherges/ng-packagr/issues/355). Thanks, [JoeQueR](https://github.com/JoeQueR)!

Module's `forRoot` and AOT
--------------------------

Another problem I had isn't that much related to `ng-packagr` itself but rather to the `NgModule.forRoot` convention. `forRoot` is a static method which allows isolating services provided by given module so that they are only provided once. The `forRoot` method should be only invoked when importing the module in the `app` module or in the `core` module. This approach helps you avoid issues with lazy loading. In my common package, I've implemented a `forRoot` method which conditionally provided some additional services based on an argument passed to the method. It looked a bit like below:

    export class SomeModule {
      public static forRoot(includeSomeService: boolean): ModuleWithProviders {
        return {
          ngModule: ModalModule, 
          providers: [
            Service1, 
            ...(includeSomeService ? [ Service2 ] : [])
          ]
        };
      }
    }
    

It turns out that you cannot do this! All the providers should be included in generated metadata and in this case, the final provider list will only be known at runtime so it's not possible to figure it out during compile time. Therefore, it's not possible to implement such scenario. The `providers` array has to be determined statically.