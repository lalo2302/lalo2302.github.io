---
title: MVP - A conceptual guide
layout: post
categories: [Design Patterns, English]
---

Common software development has reached (I believe) a phase where implementation became a routine. You need a visual interface which gets input from the user, then some transformations depending on the business logic takes place to finally store it somewhere. But to do something that persists and evolves with time, you need to put a little more effort than just sit and code like a maniac for 3 hrs. Entire business exists thanks to an app or a webpage, so a formal strategy can let numerous people to work on the same codebase without breaking anything. Testing is a whole other topic, but to architecture your code correctly is the first step for a continous development.

Here I will present a way of organizing your problem (or solution) in a conceptual way. You will not see code, or specific examples. You need to see it in an abstract way, so later you can apply it anywhere.

As I said before, we have 3 parts on a simple application:

- **Visual interface** which gets input from the user
- Transformation depending on the **business logic**
- **Store** it somewhere

Or in other words

- **V**iew
- **P**resenter
- **M**odel

But for the sake of aesthetics (or something): **MVP**

## View

The view is the UI of the application. What the user sees and interacts with. This represents: 

- animations
- transitions
- forms
- colors
- any visual aspect

The view needs to be **dumb**. If a desicion needs to be made that concerns the **data**, it doesn't belong to the view. Its only responsability is to make the software look pretty. Nothing less, nothing more. You may encounter yourself in situations where you don't know where to put someting, nevertheless, the view is almost always a bad place for it.

But what happens when a visual thing depends on a desicion? This is where the communication between the view and the presenter comes in. It may represent a little more code to write, but the benefits are worth it.

## Presenter

Here is where the fun starts. A presenter knows that the view exists and can talk to it, but it doesn't know how it works. A presenter is an independet piece of code which doesn't know what a button is, not even what a color is. A presenter is a communicator, a desicion maker. Let's try to clarify it with an example.

The user clicks a button. This means that the view receives the action of the click, it may make a pretty "loading" animation, then the view communicates with its presenter that something happened, perhaps sending some data like text that the user put through a form. Now the presenter makes a desicion of what will happen then. Lets say that a field had the wrong information, and the system should alert the user about that. This goes beyond the presenter's responsability, so the presenter communicates to the view that it should to send an alert. 

This is what I mean that the view is dumb, and that the presenter is a desicion maker. What we just explained here is what Robert C. Martin describe as Single Responsability Principle[1]

## Model

Almost every application needs to handle data, it is a fundamental part of technology, and it needs to be treated somewhere in the code. This is what a "Model" means, to manage data. Personally I don't like to see the model as a single file because data is complex, and it can be stored in several places. You can have external storage, internal storage, temporal storage, so it is up to you to decide how this will be organized. I believe that while you have in mind that a model only deals with:

- How data is consumed
- How it is stored

you are ok.

Back into the previous example, lets put ourselves in the scenario where the user filled out the form correctly, and the presenter needs to handle the data. The presenter will **decide** to send it somewhere (internal, external, etc...), but again, what will happend to the data goes beyond its responsability, so it passes it to the model, waiting maybe for a response. How you handle that response is up to you and the technology you are using, you can pass the presenter as a parameter, or maybe pass a callback (function) for the model to execute it afterwards. 

Most of the times, I/O tasks happen asyncrounously, you need to internalize this. While model stuff is happening, the view is showing an animation or something to let the user know that something is going on in the backgorund, and remember that the presenter decided what goes when.

Just how we came into the model, the chain of events will be reversed. The model will finish, and it comunicates it to the presenter, maybe with some kind of information about the result of the request. The presenter then will decide what to do, in this case it will comunicate to the view that something happened, and to finalize the view will show what the presenter told it to.

# Wrapping up

MVP is a form of communication between entities that have a single responsability. Separating it this way makes your code more testable and more readable. 

- The view makes the system look pretty
- The presenter makes desicions about what to do
- The model handles data


We need to remember that someone else will look at our work. So it would be nice to give it to them in an organized way, it doesn't matter if you don't care about this particular project, or about that particular person, but to believe that you can manage everything is only a sweet dream. And if you can, I don't know why you are reading this, go on and conquer the world, please.

# One more thing

Particular things make you system unique from others. It may be that special way of showing a date, or how a list is sorted. This behaviour maybe wont fit into MVP pattern. That particular function may be used in several places, and if you follow the DRY mantra (I hope you do), you need to write it somewhere else. This is how helpers come to save the day. Try to see how those functions relate to each other, and create a helper file. 

This can be dangerous, you can suddenly have huge helper files with a bunch of functions. Be cautious to not start putting everything there, and refactor your helpers every now and then. Everything has its place, just sometimes is not crystal clear.

### References
[1] Martin, Robert C. (2003). Agile Software Development, Principles, Patterns, and Practices. Prentice Hall. p. 95. ISBN 978-0135974445.