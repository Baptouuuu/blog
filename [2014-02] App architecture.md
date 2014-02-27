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

In the desing I think we should use, let's start with the front end and the principle of the Single Page Applications. Well, when I talk about this I think more about the idea that front end logic should be done via front end code. Seems logic, right?

So to be a bit more descriptice, we need to start building our apps via the UI as this part is the most likely to change when creating a project. If we want to be efficient, we must isolate the html (and css) in the app structure design. The idea behind that is to cut the current process where we have designers making html templates and then developers integrating them in backend code to generate them with appropriate data.

That's where the principle of SPA comes into place!

Wouldn't be easier if the raw templates were left as is with data placeholders and then data would be injected into them via Javascript? We have now at our disposal plenty of js frameworks to help us to just do that. But I think there're all missing a key point: separation of concerns. In general, for templating there are using js strings to represent the html and they all put to much logic in the template!
We must keep aside the template and the app logic so the designer can come back to it's html and still find it readable; and don't have to bother a developer to know where to modify the html. I'm sure both sides would like that.

## Offline first



## Role of the server



## Conclusion



*[SPA]: Single Page Application