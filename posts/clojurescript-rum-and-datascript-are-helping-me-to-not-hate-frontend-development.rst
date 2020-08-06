.. title: ClojureScript, Rum and DataScript, my new goto tools for rapid software prototyping
.. slug: clojurescript-rum-and-datascript-my-new-goto-tools-for-rapid-software-prototyping
.. date: 2020-08-01 10:08:14 UTC+02:00
.. tags: clojurescript rum react datascript frontend
.. category:
.. link:
.. description:
.. type: text

Foreword
________

In this blog post I am going to illustrate how, during the development of a
pet project, I have changed the way I approach software development and
prototyping, going from "backend first" to "frontend first, backend eventually".

Background
----------

At $work I have always spent most of my time in the backend and infrastructure
side of things but sometimes, especially when working on side projects, I have
to go "full stack" and take care of frontend tasks, even if I kind of suck at
it.
For this reason I am always inclined to start with some sort of backend code,
starting with a data model, core business logic and a REST API to (eventually)
build a frontend on top of it...which most of the times means I will lose
interest/momentum and the project dies.

At some point something clicked
-------------------------------

One day (probably a lazy Sunday) I had an idea: I want an incredibly stupid
way to collect random thoughts, links or any kind of text blob, I want to own
it, it must live on my VPS and I have to finish it today!
This time thoughts decided to follow a different path:
- how important what I am going to write here is? can I afford to lose data?
- what if I don't need to store anything in a durable storage?
- ok, aim for least amount of features and see later if I need more

The outcome is a service that will keep all data in volatile memory, losing it
at each restart and this is perfectly fine for what I needed at that time!
I think this small tool planted the seed for what, I think, will be be my
preferred method for future projects.

What has been so different with development of this tool is that I have tried
and (more or less succeeded) to address the problem at hand without much
ceremony, and it served its purpose quite well.
At some point I've started to add more features "just because" but, at the end
of the day, the core remained the same and now I even want to remove these new
features because they are not adding value but are, effectively, distracting.

Next project
------------

Fast forward few months, I am again thinking about a new project.
My girlfriend needed/wanted a tool to make it easy to share a watermarked
version of her drawings, even better if the shared link required a password to
access the image.

Compared to the previous project, this one is slightly more complex, especially
the user facing part, but I am confident that if I address the most basic
requirements first, I should be able to give her a rough beta in few hours.
Here are the features I have decided to include in the "MVP":
- user can upload a file
- user can chose a watermark size
- user can eventually chose a password to protect the image
- once the user clicks "Upload", the system applies the watermark and
- render a page with the list of uploaded/watermarked files, each with a link to it

So far so good, the tool is there, ugly but working. What if we open this
tool to other people? WOW! We can make our own product! Lets do it!

Armed with great energy I have started working on the new version but this time
I have started building a full featured backend even if, at this point,
is not 100% clear how a user is going to interact with it, how valuable it
could be and all the usual unknowns.
So why I am insisting with the old approach when it showed, multiple times, it
is quite ineffective? I guess it is a matter of habits (because I don't want
to call it stupidity).

A new approach
--------------

I often fail to make progress on my projects when it comes the turn of the
frontend code, after having written most of the backend, maybe this time I
can try the other way around!
Common sense suggests to create mocks of the interface using tools like
`Balsamiq <https://balsamiq.com/wireframes>`_, `Moqups <https://moqups.com/>`_ and so on but, for some reasons I am totally unable to
use those kind of tools plus I would like to have a taste of how to UI feels
in the hands a user...what should I do now?

Recently I have started experimenting with `ClojureScript <https://www.clojurescript.org>`_ and `Rum <https://github.com/tonsky/rum>`_ (a library
that wraps React) and so far I feel quite productive with it, why should I not
use these tools to write the mock? The plan now looks like this:

- quickly build an ugly but functional UI
- mock the app state/data to reflect what I need in the UI
- understand what feels wrong, get feedback from other people
- plan next iteration
- repeat until happy

Essential to this approach is the quick feedback loop you can get by using
ClojureScript + Rum (or any other React wrapper like `Reagent <https://reagent-project.github.io/>`_), immutable
persistent data structures and a REPL; I "suspect" that, if we exclude the
REPL, a similar setup can be achieved with other stacks but I am quite happy
with the tools I have chosen.

How has it been so far?
-----------------------

I am not a frontend developer and every time I try to create some sort of UI
I am pretty sure I will fail, lose interest and put the project on hold but,
this time, it has been quite different.
Being able to iterate quickly and having a fast feedback loop, helped me a lot
to proceed at "great speed" and not losing focus.
Each time I wanted to add a new feature or change how a component had to work,
the process has been pretty straightforward:
- create a new branch
- try out the new idea by small, even tiny iterations with live feedback
- adjust the data model as needed
- repeat
- if happy merge to master otherwise scrap everything and forget

Managing the app state
----------------------

One of the points that I would like to spend so time on is the app state.
Even if keeping all the state in one big map is quite handy and flexible,
at some point, it becomes harder and harder to maintain, especially in the
case where there are many ( > 4 or 5) entity types with cross references and
the size of the data gets bigger and bigger; it can still be done but the
state management code starts to get bigger and harder to maintain compared to
the UI code; for my case, the UI code had to know too much about the data model
itself impacting the separation of concerns and this was driving away the focus
on the main goal which is to explore the space of possibilities for the UI I
want to build.

Introducing Datascript into my tool-set
---------------------------------------

With the growing pain of state management I have decided to invest a little bit
of time to find out which solutions are already available in this space and
fortunately I have stepped into `Datascript <https://github.com/tonsky/datascript>`_, an in memory database developed by
the same person who gave us Rum :)
This database has all the features I need plus some more; I am sure that I am
still missing some handy features but so far I have been able to:

- build a consistent data store
- which supports cross references between entities
- which I can easily query (using Datalog)
- with support to joins

`Datalog <https://en.wikipedia.org/wiki/Datalog>`_ itself is quite simple even if a bit "alien" until you know how to
use it; instead of writing about it here I prefer to give a link to the best
learning material I have found `<http://www.learndatalogtoday.org/>`_.

Armed with Datascript (for the data store) and Datalog (for the queries) the
UI code can now interact with the app state without knowing much about it!
This is a great relief because now I can focus again on building a UI for my
project and spend the right amount of time (< 5%) on the data store.

Another good point of this approach is that, when the backend will be ready,
it will be a matter of translating what I will get out of its APIs to the
UI's internal data store and everything will continue to work as expected.

Summary
-------

The goal of this post was to write down my current approach to software
development with the hope to start a discussion with other developers which
maybe be interested to explore and talk about novel methodologies which have
worked great for them.
If by reading this blob of text you are now also curious about ClojureScript
and its ecosystem, I am even more happy!

Links
-----

ClojureScript: `<https://clojurescript.org/>`_
Rum: `<https://github.com/tonsky/rum>`_
DataScript: `<https://github.com/tonsky/datascript>`_
Datalog: `<https://en.wikipedia.org/wiki/Datalog>`_
Datalog course: `<http://www.learndatalogtoday.org/>`_
Reagent: `<https://reagent-project.github.io/>`_
