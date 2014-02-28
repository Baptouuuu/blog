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

So now that we know how it works and the basic codes, let's see in what it can be useful. For the example we will assume we have 2 web apps hosted on the same domain. The first is a project management app and the second one is about finance handling. As nowadays all apps are inter-connected, we want it too for our apps. So it be great if when we finish/start new tasks in the management app it's reported in the finance one.

In general, the first though is: let's do some ajax to send data and check if there's modification from the otherside.

The main problem with this approach is that you need to communicate with the server. But as you're part of the cool kids, you've designed your app offline first and learned to build your apps without the need to rely on the server for everything.
And this where the SharedWorker help us do what required a server before; keeping the paradigm of offline first.

With the worker you can transfer data between tabs only through javascript.

Cool but how do we do it?

```js
//page1.js
var worker = new SharedWorker('shared-worker.js');
worker.port.addEventListener('message', function (event) {

    var source = event.data.source,
        action = event.data.action;

    if (action === 'data:request') {
        this.postMessage({
            source: 'page1',
            action: 'data:request:answer',
            dest: source,
            data: retrieveLocallyStoredData()
        });
    }

}.bind(worker.port), false);
worker.port.start();
worker.port.postMessage({
    source: 'page1',
    action: 'identification'
});
```

```js
//page2.js
var worker = new SharedWorker('shared-worker.js');
worker.port.addEventListener('message', function (event) {

    var source = event.data.source,
        action = event.data.action;

    if (action === 'data:request:answer' && source === 'page1') {
        computeData(event.data.data);
    }

}.bind(worker.port), false);
worker.port.start();
worker.port.postMessage({
    source: 'page2',
    action: 'identification'
});
worker.port.postMessage({
    source: 'page2',
    action: 'data:request',
    dest: 'page1'
});
```

```js
//shared-worker.js
var ports = [],
    identities = {};

self.addEventListener('connect', function (event) {
    var port = event.source;

    ports.push(port);

    port.addEventListener('message', function (e) {

        if (e.data.action === 'identification') {
            identities[e.data.source] = ports.indexOf(port);
        }

        if (
            e.data.source &&
            e.data.dest &&
            e.data.action
        ) {
            ports[identities[e.data.dest]].postMessage(e.data);
        }

    }.bind(this), false);

    port.start();
}.bind(self), false);

self.addEventListener('close', function (event) {
    var idx = ports.indexOf(event.source);

    //keeps in sync source name and index
    for (var source in identities) {
        if (identities.hasOwnProperty(source)) {
            if (identities[source] === idx) {
                delete identities[source];
            } else if (identities[source] > idx) {
                identities[source]--;
            }
        }
    }

    ports.slice(idx, idx);

}.bind(self), false);
```

So lots of code here, I'll explain what it does.

In both pages we instanciate our worker and identify themselves via the first `postMessage`. By default the API does not allow us to name the different connections, so this is how I found to do it so far.

In the first page we listen messages coming from the worker and if the action is `data:request`, we send back to the source the data of our app.
In the second one, we send our data request to the other page and listen to the response.

The code of the worker is mainly boilerplate code to setup named ports. Then it only comes down to the check on `source`, `dest` and `action` attributes on the message data; if there are all available, we post the message data to the appropriate port. Simple as that.

With not that much of code you can setup directional communication between tabs/apps, and is extensible to as many sources that you want. However the code above does not cover the case where the user open the same app in multiple tabs, it would break the mechanism of identification. But it could be a good way to check if the user launched the same app twice, in the identification code in the worker if we see a source is already defined with the same name, we post a message back and alert the user to only use one instance of the app (and don't forget to remove the port from the `ports` array).

## Sugar

A nice improvement we could do in the worker would be to introduce a data transformer, so we could adapt the data structure depending on which source is requesting it. It would really help keep as much as possible agnostics the apps from one another.

But as we can only specify one file for a worker, it wouldn't be really easy to put complex data manipulation into this single file. Especially if the number of apps communicating starts growing.

But don't be afraid, there's a function to help us and it's called `importScripts`. In a page you add your scripts via the `script` tag, in the worker it's done through this function. You pass one or multiple files as arguments, and you can call the function as many times you want.

```js
importScripts('file1.js');
importScripts('file2.js', 'fileX.js');
```

## Restrictions

To prevent concurrency conflicts, the worker can access to only a few objects from the main thread.

* `navigator`
* `location` (read-only)
* `XMLHttpRequest`

`window`, `document`, `parent` and the DOM are **NOT** accessible inside the browser.

The worker file, and files loaded inside it, must be on the same domain and respect the same scheme (http or https) as the page instanciating it.

## Conclusion

Some last words on the subject, I hope I helped you understand what kind of stuff you can do with Shared Workers and raised some ideas in your minds.
The only problem I encountered was the debug part, Chrome Dev Tools allows to debug workers but not shared ones. So if someone knows how to do it, please leave a comment on the issue related to this article, thanks.

And for browser compatibility, please refer to our dearest friend: [caniuse.com](http://caniuse.com/#feat=sharedworkers). Unfortunately, it's pretty limited right now.