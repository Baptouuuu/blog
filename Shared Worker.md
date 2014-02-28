# Shared Worker [JS]

Last week I was thinkig one of my personal projects and what kind of evolution I could make. And a came across one requiring two browser tabs (on the same domain) to communicate with each other. And I remembered one article read long ago about shared web worker that could help me do that.
I've spent the whole week trying to find this article and couldn't find it back. The only resources I did find was a short article on [SitePoint](http://www.sitepoint.com/javascript-shared-web-workers-html5/) dating back to 2011 and the [specifications](http://www.whatwg.org/specs/web-apps/current-work/multipage/workers.html#shared-workers) on [whatwg.org](http://whatwg.org).

A strange thing is the disappearance of the documention on the [Mozilla Developper Network](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker).

So I decided to do an article on the subject with a use case and code examples.

## Definition

First things first, what is a **Shared Worker**?

It's an object you instanciate on your page, taking the path to a javascript file as first argument. It will execute the specified script in a separated thread and is aimed to process data without affecting the main thread (the one managing your page). You communicate between the threads through messages.

Another key point is that every new instanciation on the same file will not create a new thread, but will connect to the first thread.

>         (Worker thread)
>           /        \
>          /          \
> (Tab 1 thread) (Tab 2 thread)

If you don't see yet in what this is really cool, wait for the use case.

So first let's look at how to instanciate a shared worker:
```js
var worker = new SharedWorker('shared-worker.js');
worker.port.addEventListener('message', function (event) {
    //here we listen all the messages the worker is sending to us
},false);
worker.port.start();
```

And the worker by itself looks like this:
```js
//shared-worker.js
self.addEventListener('connect', function(connectEvent) {
    var port = event.source;

    port.addEventListener('message', function (messageEvent) {
        //here we listen messages coming from the page thread
    }, false);

    port.start();
}, false);
```

Before jumping to the use case, let's explain the code above.

As you can see there's a lot of `port`, think of it as a connection. Remember, multiple pages can connect the same worker. So when you listen to events or send messages it's done on the `port` object.

In the examples,I've shown 2 event names but there is a third named `close` and is available in the worker only. So in the order, we instanciate our worker, we define our messages listeners then say "ok connect me to the worker". This last one will trigger the `connect` event in the worker, in the listener we listen to the messages that may come through this connections and finally says "I'm ready too,let's do some work". Now the threads can communicate with each other.

The `close` event, not appearing in the code, remains really important. As the worker is only destroyed when all the tabs connected to the worker are closed, if you don't close properly the connection to the worker and reload your page and re-instanciate the worker, when it will post a message you will receive it twice and I'm sure you don't want that.

So here's the code to close the connection:
```js
//in the page
worker.port.close();
```

```js
//shared-worker.js
var ports = [];
self.addEventListener('connect', function (connectEvent) {
    var port = event.source;
    ports.push(port);
    //listen to messages...
}.bind(self), false);

self.addEventListener('close', function (event) {
    var idx = ports.indexOf(event.source);
    ports.slice(idx, idx);
}.bind(self), false);
```

Above we the keep the track of all the opened connections, and remove them when the `worker.port.close()` is called; you will in the use case why this is useful.


## Use case

