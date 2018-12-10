---
title: 'SBT: how to build and deploy a simple SBT plugin?'
tags:
  - sbt
  - scala
url: 88.html
id: 88
categories:
  - Other topics
  - Scala
  - Tutorials
date: 2016-07-03 18:46:42
---

Few weeks ago when I was working on [my pet project](http://codewithstyle.info/scala-ts-scala-typescript-code-generator/), I wanted to make it an SBT plugin. Since I had to spend some time studying SBT docs, I decided to write a short tutorial explaining how to write and deploy a SBT plugin.

### Make sure your project can be built with SBT

First of all, your project needs to be buildable with SBT. This can be achieved simply - any project that follows the [specific structure](http://www.scala-sbt.org/0.13/docs/Directories.html) can be built with SBT. additionally, we are going to need a build.sbt  file with the following contents at the top-level:

```scala
lazy val root = (project in file(".")).
  settings(
    name := "my-sbt-plugin",
    version := "0.2.0",
    organization := "com.github.miloszpp",
    scalaVersion := "2.10.6",
    sbtPlugin := true,
    sbtVersion := "0.13.11"
  )
```

Note that we are using Scala version 2.10 despite that at the time of writing 2.11 is available. That's because SBT 0.13 is build against Scala 2.10. You need to make sure that you are using matching versions, otherwise you might get compile errors.

### Implement the SBT plugin

Our example plugin is going to add a new command to SBT. Firstly, let's add the following imports:

```scala
import sbt.Keys._
import sbt._
import complete.DefaultParsers._
```

Next, we need to extend the AutoPlugin  class. Inside that class we need to create a nested object called autoImport. All SBT keys defined inside this object will be automatically imported into the project using this plugin. In our example we are defining a key for an input task - which is a way to define an SBT command that can accept command line arguments.

```scala
object MySBTPlugin extends AutoPlugin {

  object autoImport {
    val hello = inputKey\[Unit\]("Says hello!")
  }
```

Now we need to add an implementation for this task:

```scala
override lazy val projectSettings = Seq(
    hello := {
      val args = spaceDelimited("").parsed
      System.out.println(s"Hello, ${args(0)}")
    }
  )
```

And that's it.

### Test the SBT plugin locally

SBT lets us test our plugins locally very easily. Run the following command:

```
sbt compile publishLocal
```

Now we need an example project that will use our plugin. Let's create an empty project with the following directory structure:

```
src
project
   plugins.sbt
build.sbt
```

Inside plugins.sbt , let's put the following code:

```
addSbtPlugin("com.github.miloszpp" % "scala-ts" % "0.2.0")
```

Note that this information needs to match organization , name  and version  defined in your plugin. Next, add the following lines to build.sbt:

```scala
import sbt.Keys._

lazy val root = (project in file(".")).
  settings(
    scalaVersion := "2.10.6",
    sbtVersion := "0.13.11"
  ).
  enablePlugins(com.mpc.scalats.sbt.MySBTPlugin)
```

Make sure that you use the fully qualified name of the plugin object. You **can **use Scala version older than 2.10 in the consumer project. Now you can test your plugin. Run the following command:

sbt "hello Milosz"

Note the use of quotes - you are passing the whole command, along with its parameters to SBT.

### Make it available to others

If you would like to make your plugin available to other users, you can use OSS Repository Hosting. They are hosting a public Maven repository for open source projects. Packages in this repository are automatically available to SBT users, without further configuration. The whole procedure is well described [here](http://central.sonatype.org/pages/ossrh-guide.html). One of the caveats for me was to change the organization  property to com.github.miloszpp (I host my project on GitHub). You can't just use any string here because you need to own the domain - otherwise, you can use the GitHub prefix.