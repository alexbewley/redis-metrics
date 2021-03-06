redis-metrics
=============

Easy metric tracking and aggregation using Redis.

This module was originally created to provide an easy way of storing and
viewing aggregated counter and trend statistics.

In a sense, the libary tries to provide sugar-coated method calls for storing
and fetching Redis data to report counts and trends. The first design goal is to
make counting simpler.

Read on below or read more on the [documentation site](http://getconversio.github.io/redis-metrics/).

Install
-------

```console
$ npm install --save redis-metrics
```

Use
----- 

Basic counter:

```javascript
// Create an instance
var RedisMetrics = require('redis-metrics');
var metrics = new RedisMetrics();

// If you need to catch uncaught exceptions, add an error handler to the client.
metrics.client.on('error', function(err) { ... });

// Create a counter for a "pageview" event and increment it three times.
var myCounter = metrics.counter('pageview');
myCounter.incr();
myCounter.incr();
myCounter.incr();

// Fetch the count for myCounter, using a callback.
myCounter.count(function(cnt) {
  console.log(cnt); // Outputs 3 to the console.
});

// Fetch the count for myCounter, using promise.
myCounter.count().then(function(cnt) {
  console.log(cnt); // Outputs 3 to the console.
});
```

Time-aware counter.

```javascript
// Create an instance
var RedisMetrics = require('redis-metrics');
var metrics = new RedisMetrics();

// If you need to catch uncaught exceptions, add an error handler to the client.
metrics.client.on('error', function(err) { ... });

// Use the timeGranularity option to specify how specific the counter should be
// when incrementing.
var myCounter = metrics.counter('pageview', { timeGranularity: 'hour' });

// Fetch the count for myCounter for the current year.
myCounter.count('year')
  .then(function(cnt) {
    console.log(cnt); // Outputs 3 to the console.
  });

// Fetch the count for each of the last two hours.
// We are using moment here for convenience.
var moment = require('moment');
var now = moment();
var lastHour = moment(now).subtract(1, 'hours');
myCounter.countRange('hour', lastHour, now)
  .then(function(obj) {
    // "obj" is an object with timestamps as keys and counts as values.
    // For example something like this:
    // {
    //   '2015-04-15T11:00:00+00:00': 2,
    //   '2015-04-15T12:00:00+00:00': 3
    // }
  });

// Fetch the count for each day in the last 30 days
var thirtyDaysAgo = moment(now).subtract(30, 'days');
myCounter.countRange('day', thirtyDaysAgo, now)
  .then(function(obj) {
    // "obj" contains counter information for each of the last thirty days.
    // For example something like this:
    // {
    //   '2015-03-16T00:00:00+00:00': 2,
    //   '2015-03-17T00:00:00+00:00': 3,
    //   ...
    //   '2015-04-15T00:00:00+00:00': 1
    // }
  });

// Fetch the count for the last 60 seconds...
// ... Sorry, you can't do that because the counter is only set up to track by
// the hour.
```

Redis Namespace
----
By default keys are stored in Redis as `c:{name}:{period}`. If you prefer to use a different Redis namespace than `c`, you can pass this in as an option:
```
var myCounter = metrics.counter('pageview', { timeGranularity: 'hour', namespace: 'stats' });`
```


Test
----
Run tests including code coverage:

    $ npm test

Documentation
-------------
The internal module documentation is based on [jsdoc](http://usejsdoc.org) and
can be generated with:

    $ npm run docs
