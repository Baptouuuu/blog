# App architecture

**Work in progress**...

Lately I put my thoughts on where the web is at and what we are trying to accomplish with it; and more precisely on the rise of web applications. This article is about how, in my opinion, we should architecture our new projects; exisiting ones may be hard to mutate toward the structure I'll express.


## Current state

Before jumping right into my conclusions, I want to state about projects structures I came across in past year. I'll talk only about 2 of them:

* professional web app (work on during my graduation internship)
* e-commerce website (biggest project since I graduated)

The common thing with those two different kind of projects is how the Javascript was mixed up in the html to add dynamic behaviours in some pages, without a clean and structured code. It was often features added after the page was created, and were about manipulating datas in the UI.
Aside of that, there was pages designed from day one with js features and so some code design was able to be made.

But the problem here is, as a whole, how to keep a project coherent and maintanable when we build front code like that?

For the past years backend code as matured to a point where we have stable frameworks based around trusted design patterns. We learned to use them and smartly built our backend code with clear separation of concerns (like with MVC pattern). But when talking about our front code, for now, let's be honest, is a mess!

So I asked myself why don't we extend the separation of concerns to the whole application? Backend, frontend code and what sits in the middle.

The technologies we are using have their proper goal:

* Backend code: store/restitute data
* HTTP: stateless protocol to transfer this data
* HTML: format to present data in a structured way
* Javascript: manipulate what's displayed
* CSS: handle how data is displayed

Instead, we are trying to manipulate what and how is displayed data through javascript. We try to handle client side stateful data in our server side code, via sessions, throught a stateless protocol.

So, why don't we stick with their original intentions?


## Single Page Application



## Offline first



## Role of the server



## Conclusion



*[SPA]: Single Page Application