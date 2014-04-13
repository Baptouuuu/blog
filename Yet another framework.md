# Yet another framework [JS]

Yesterday I finally merged my last 6 months (not full time) of work on my framework to `master`, releasing the first version `0.5.0`. In this article I want to explain why I started this project.

## Origin

I want to start at the very beginning. More than a year ago I started a project management tool fully written in javascript to replace a discontinued tool named Wunderkit. It was also a time I learned to write javascript so it was a good opportunity to improve my skills on this language.

As I wanted to really know how javascript was working I did this project in vanilla js. I worked on it for about one year and I've rewritten the app 3 times as I was never really satisfied how the code was structured.

In the meantime, I discovered the backend framework [Symfony2](http://symfony.com) to build the website/api for my project. Working with it was an eye opener to a lot of design patterns and how different components are working together inside a unique framework. And I felt like: "Where the hell is a framework like this for javascript?".

So I searched for frameworks to build my project upon. In the end the sole one close enough was [angular](http://angularjs.org) (no need to introduce it I hope). I liked the fact there are a mechanism for services and that it encourages you to use them.

But I was still missing the clear code structure and organisation I was familiar with Symfony. So here's how and why I started this project: to reproduce the same kind of framework as Symfony but in javascript.

## Principles

The main problem I encountered when looking to other frameworks was the hole integretaed thing. You can't really reuse part of the framework outside it, that's the first thing that bugged me up and I didn't want that for this project. So I started to work like I was building different libraries with no relation between them and glue them up when there're all ready (same thing is done with Symfony).

So all the libraries in Sy can be reused outside out the framework: Configurator, HTTP wrapper, Mediator, Logger, Generator, Registry, StateRegistry, Service container, Storage mechanism, Translator and View mechanism (and this should be just the start :)). Each one of them has it's own documentation in the framework [repository](http://github.com/baptouuuu/sy/tree/master/docs/).

The other that I really didn't like with all other frameworks was that you never now (and nobody tells it) where you need to put the instances of your main objects. I've seen controller instances put in the global scopes, or each time you want to instanciate a top object you do it in a closure.

With Sy it's simple, you don't have to wonder about it, the framework takes care of it. Because it impose you a clear code structure in where you need to put your classes, the framework can determine where to find what it needs and instanciate in an internal object a new instance of a controller for example. In the end, you only define your classes in the `App` global object (and sub-objects) and the framework will create the instances when needed. No need to wonder if some some code will override a variable in another file, or to loose performance by using to much closures.

Another point I wanted for this project was about performance. Nowadays end user don't see performance (ui responsiveness, etc...) as a plus in your app, but it's now a key feature. That's why all the code written in Sy is here to help you reduce the need of closures. Most of the javascript I read in my work rely (way too much) on them, because it's easy it doesn't it's good. I remember a while back watching a Google I/O event about how to optimize code for V8, there was a part about the garbage collection and how it worked; that day I realized how closures can become a hell to the GC when looking for objects to be destroyed and how it could impact the performance (there's also a google doc on js [optimizations](https://developers.google.com/speed/articles/optimizing-javascript)). So if you look at the core code of my framework you'll see almost no closure, in fact it's quite easy to avoid them.

The last point is about reading code, I wanted to keep things simple as if there were no framework at all. I wanted to lower the learning curve when reading code built upon Sy and feel like it's just vanilla javascript. When I look at the code now, I feel like homemade code using external libraries to abstract some work. I didn't want to first understand framework's principles before understanding the real code in front of me. And I hope you'll have this feeling too. Plus, with the code structure imposed you know what's where, don't need to search where some code could be located.

## Conclusion

So to resume a bit, I'm building this framework to give you guys a good code structure using known design patterns. But at the same time leave things simple by letting you have the feeling you're writing vanilla js.

I hope I'm on the good path to accomplish this, and you'll like this approach too.

Feel free to express your thoughts on this subject in the associated [issue](https://github.com/Baptouuuu/blog/issues/5).