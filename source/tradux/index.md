---
title: 'TRADUX: real-time trading platform simulator'
url: 112.html
id: 112
comments: false
date: 2016-12-11 14:48:07
---

TRADUX is a real-time trading platform simulator written as an exercise in event sourcing, Redux and Firebase. The application is completly serverless. The frontend processes events (which represent added orders) which are synchronized by Firebase's real-time database. Application uses Redux to process events and produce updated state. Additionally, TRADUX uses [Webtask](https://webtask.io/) to periodically re-calculate state stored in the database. Therefore, whenever client starts up, it doesn't have to download all events since the begining of time. It's sufficient to load the pre-calculated state and listen only for new events. TRADUX is hosted [here](https://tradux-fa630.firebaseapp.com/). The source code can be found [here](https://github.com/miloszpp/tradux). _TRADUX is currently hosted on a free Firebase plan which might result in it being unavailable when usage limits are exceeded._

### Event sourcing

The main idea behind event sourcing is to use the database to store immutable events instead of mutable state. Given stream of events and an initial state, it's possible to compute the current state. This solves whole classes of problems (for example, no need for transactions). TRADUX uses just one type of event - one that represents adding an order. Given the history of added orders, it's possible to calculate:

*   currently active orders
*   transaction history
*   statistics (e.g. average prices)

### Redux

Redux is a technology which allows to manage UI state in an event-sourcing like manner. Using Redux was very convenient when combined with event sourcing. In TRADUX, events are pushed to the database. Additionally, the application listens for new events in the database. Whenever a new event is received, Redux action is dispatched and processed by the reducer. The reducer calculates active orders, transactions and average prices. Angular components subscribe to state changes and update UI whenever new state is computed.

### Firebase real-time database

The real-time database is a technology which allows effortless synchronization between web clients. Whenever an object is added to the database, all clients are notified and eventually see the same state of database. In TRADUX, Firebase enables interactivity between clients. Whenever new event is added to the database, all clients will see it and will be able to process it. Thanks to Firebase's validation rules, it's possible to prevent invalid data from being added to the database. That logic would normally belong to backend.

### Webtasks

Webtask is a technology that allows running arbitrary Node.js code in the cloud, without having to worry about infrastructure to run it. TRADUX uses Webtask to periodically re-calculate state and save it in the Database. Having such state snapshot enables much faster loading of clients - they don't need to calculate state from all the events in history. TRADUX uses one Cron task running every minute (a feature of Webtask). Thanks to Webtask bundler, it's possible to share codebase between the client app and the task.