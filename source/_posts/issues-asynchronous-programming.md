---
title: Issues with asynchronous IO in web applications
tags:
  - scala
url: 76.html
id: 76
categories:
  - Articles
  - Other topics
  - Scala
date: 2016-05-01 13:40:40
---

Building servers with non-blocking IO has been quite popular these days. [Tests have shown](http://www.ducons.com/blog/tests-and-thoughts-on-asynchronous-io-vs-multithreading#conclusions) that it does actually improve scalability of web applications. However, my experience show that it comes at a cost. In this post I am going to discuss some negative aspects of writing asynchronous code based on Scala's Futures.

### Stacktraces

Debugging exceptions in asynchronous programs is a pain. When issuing an asynchronous IO operation you provide a callback that should be executed when the operation returns. In most implementations, this callback might be executed on any thread (not necessarly the same thread that invoked the operation). Since call stack is local to the thread, the stacktrace that you get when handling an exception is not very informative. It will not trace back to the servlet so you may have hard time figuring out where what actually happened.

Exception in thread "main" java.lang.IllegalArgumentException
	at HelloScala$$anonfun$1.apply(HelloScala.scala:28)
	at HelloScala$$anonfun$1.apply(HelloScala.scala:28)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.liftedTree1$1(Future.scala:24)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.run(Future.scala:24)
	at scala.concurrent.impl.ExecutionContextImpl$AdaptedForkJoinTask.exec(ExecutionContextImpl.scala:121)
	at scala.concurrent.forkjoin.ForkJoinTask.doExec(ForkJoinTask.java:260)
	at scala.concurrent.forkjoin.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1339)
	at scala.concurrent.forkjoin.ForkJoinPool.runWorker(ForkJoinPool.java:1979)
	at scala.concurrent.forkjoin.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:107)

### Thread-local variables

Some libraries use a mechanism called ThreadLocal  variables (available in Java and C#, in Scala known as DynamicVariable ). By definition, these libraries do not work well with asynchronous code, for the same reason that we get poor stacktraces. I have already discussed [one of such situations](http://codewithstyle.info/accessing-request-parameters-from-inside-a-future-in-scalatra/) on my blog. Another one is [Mapped Diagnostic Context](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/MDC.html) from the [Logback](http://logback.qos.ch) framework. MDC is a nice mechanism that allows you to attach additional information to your logs. Since the information is contextual, it will be available even to logs written from within external libraries. However, as one might expect, MDC is implemented with thread-local variables. Therefore, it doesn't work well with Scala's futures. There is a way to [get MDC with Futures](http://stackoverflow.com/questions/28306429/track-context-specific-data-across-threads) working by writing a custom ExecutionContext  (Scala's threadpool) that is aware of contextual data and propagates across threads.

### Missed exceptions

Unless you are very careful, it is quite easy to not wait for a Future to complete but instead to fork execution into two branches. When an exception is thrown in a Future that nobody is waiting for, it will most likely just go unnoticed.

    def postData(url: String, data: String): Future\[Unit\] = // ...

    def saveToDb(data: String): Future\[Unit\] = // ...

    postData("http://example.com", "message")
    saveToDb("another message")

Above code will compile. However, saveToDb  will most likely be called before postData  returns since execution has been forked. Any exception thrown inside postData  will most likely be missed. The correct way to write the above code would be:

    postData("http://example.com", "message") flatMap { _ =>
      saveToDb("another message")
    }

### Caching

Caching gets more complicated in an asynchronous web application, unless the library you use for caching is designed to work with async code. One of the most common patterns in caching libraries is to let you provide a function that should be executed when a value in cache is missing. See the below example of [Guava Cache](https://github.com/google/guava/wiki/CachesExplained):

cache.get(key, new Callable<Value>() {
    @Override
    public Value call() throws AnyException {
      return doThingsTheHardWay(key);
    }
  });

If doThingsTheHardWay returned a Future (was asynchronous) then you would have to block the thread and wait for the result. Mixing blocking and non-blocking code is generally discouraged and may lead to undesirable situations such as deadlocks.

### Code readbility

Asynchronous code adds complexity. In Scala, you need to use all sorts of Future combinators such as flatMap , map  or Future.sequence  in order to get your code to compile. The issue is partially addressed by async/await  language extensions/macros (available for example in [Scala](https://github.com/scala/async) and [C#](https://msdn.microsoft.com/pl-pl/library/hh191443.aspx)) but it can still make your code less readable and harder to reason about.