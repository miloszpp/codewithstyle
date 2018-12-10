---
title: 'Scala-ts: Scala to TypeScript code generator'
tags:
  - scala
  - typescript
url: 86.html
id: 86
categories:
  - Other topics
  - Projects
  - Scala
date: 2016-06-23 15:04:15
---

I have started using TypeScript a few weeks ago at work. It turns out to be a great language which lets you avoid many problems caused by JavaScript's dynamic typing, facilitates code readibility and code refactoring and does that at relatively small cost thanks to modern, concise syntax. 

Currently we are using TypeScript for writing the frontend part of a web application which communicates with backend in Scala. The backend part exposes a REST API. 

One of the drawbacks of such desing is the need for writing Data Transfer Objects definitions for both backend and frontend and making sure that they match each other (in terms of JSON serialization). In other words, you need to define the types of objects being transferred between backend and frontend in both Scala and TypeScript. 

Since this is a rather tedious job, I came up with an idea to write a simple code generation tool that can produce TypeScript class definitions based on Scala case classes. 

I've put the project on Github. It's also available via SBT and Maven. Here is the link to the project: [https://github.com/miloszpp/scala-ts](https://github.com/miloszpp/scala-ts)