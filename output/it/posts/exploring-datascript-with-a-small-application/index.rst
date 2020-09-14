.. title: Exploring Datascript with a small application
.. slug: exploring-datascript-with-a-small-application
.. date: 2020-09-02 11:43:16 UTC+02:00
.. tags: clojure clojurescript rum datascript datalog
.. category:
.. link:
.. description:
.. type: text

Overview
________

In a previous post I have briefly described how using ClojureScript, Rum and
Datascript helped me to explore a new way to prototype software starting by
investigating how the user is going to interact with it.

People, instead, showed more interest on Datascript (go figure :) ) so in this
post I am going to write a small application to explore how to use Datascript
and Datalog without digging too much into the nitty gritty details of both
because, I think, there are better resources out there. (See `Links section <Links>`_.)

The application
---------------

Instead of writing the usual TODO app, I would like to provide a way for users
to give feedback to my blog posts. Usually this is why you add a comment area
to your website but it would be nice if readers could write their feedback
for each section and not just the whole post.

For now I want to focus on the frontend part of it, to get a feel of how it
could work in the hands of the users. If it proves to be usable I can think of
a way to store the feedback to a durable storage, eventually add access control,
eventually an admin interface etc etc.

To keep things super simple I will target the structure of my blog posts which
is more or less like this:

.. code:: html

   <div class='body'>
     <div class='section' id='section-title'>
       <h2>Section title</h2>
       <p>Text<p/>
     </div>
   </div>

The div with class body is the main container; inside there are N section div
elements, each with an id which is the slugified title of the section, an h2
element with the title of the section and finally the content.

At start the application will fetch all post's sections, will store their id and
title in the database, and will attach a react component to them which will:
- provide a "Write your feedback" button which will toggle the feedback area composed by
- a form to write new comments
- a list of old comments with buttons to reply to or to delete them

The sources of the application are available `here <https://github.com/fpischedda/blog-feedback-app>`_.

I suggest to try the full application to get a feel of how it looks and works;
to run it please follow the instructions in the `README.md <https://github.com/fpischedda/blog-feedback-app/blob/master/README.md>`_ of the project.

Stack for the prototype
-----------------------

To build this application I have chosen to use my (currently) preferred
prototyping stack which is composed by:

- ClojureScript: the language
- Figwheel.main: build system and live code update
- Rum: React wrapper library
- Datascript: in memory Datalog database

For me, this work pretty well and I feel quite productive with it; the main
benefit is the fast feedback loop provided by figwheel, the easy of use of Rum,
a powerful, easy to use, database where to store the application state and last
but not least a REPL where I can quickly fiddle with every bit of the
application.

Data model
----------

For this simple application there are only two entities that we are going to
store in the database: section and comment; lets define a data model that will
hold the application state:

section

- id: string, HTML id of the section
- name: string, name/title of the section as extracted from the HTML

feedback/comment

- id: uuid
- author: string, author of the comment
- text: string, the content itself
- created-at: string, ISO format of the time of the creation
- parent: optional, ref to parent comment

The data model can be described with the following schema:

.. code:: clojure

  (def schema
    {:section/id      {:db/unique :db.unique/identity}
     :comment/id      {:db/unique :db.unique/identity}
     :comment/section {:db/valueType :db.type/ref
		       :db/cardinality :db.cardinality/one}
     :comment/parent  {:db/valueType :db.type/ref
		       :db/cardinality :db.cardinality/one}})

As you may have noticed, not all attributes of the entities are in the schema,
if fact it is sufficient to describe attributes with special meaning like ids
or references.

Data layer
----------

Usually I like to have a separate namespace for the data and presentation
layers, even for applications as simple as this one.
Lets have a look at the `app.db namespace <https://github.com/fpischedda/blog-feedback-app/blob/master/src/cljs/app/db.cljs>`_.

After defining the schema, the db gets created with the following form:

.. code:: clojure

  (def db (d/create-conn schema))

It is common to use `defonce` when defining app state to not reset it after
a code reload but, in this case, I prefer to `def` because, when experimenting,
it is quite useful to be able to change the schema and you may want to use the
newest one in your database.

Another important tool for experimentation is to quickly populate the database
with mock data; I find it very useful when it comes the time to work on the UI
or when I need to test queries and transactions.

Here is an example query and the result when some data is in the database:

.. code:: clojure

  (defn all-sections [db]
    (d/q '[:find ?id ?name
           :where
           [?s :section/id ?id]
           [?s :section/name ?name]
           ]
      db))

  (comment
    (all-sections @db) ;; => #{["overview" "Overview"] ["application-design" "Application design"]}
    )

Digging a bit more in to the db namespace, we can find an example of how using
aggregates looks like in the section-comments-count function, here is the code:

.. code:: clojure

  (defn section-comments-count
    "If the query cannot find any entity that matches then it cannot reasonably
     count anything and it will return nil; with the use of `or` we can handle
     this case and return 0."
    [db section-id]
    (or
      (d/q '[:find (count ?c) .
               :in $ ?section-id
               :where
               [?s :section/id ?section-id]
               [?c :comment/section ?s]
               ]
          db section-id)
      0))

As stated in the docstring, when there are no matching entities, the query will
return `nil`, for this reason I use an `or` to return the correct value (zero).

Also, please note the `dot` after the call to `count`, this signals that we
want a scalar value back instead of a set with one element containing the count.

Building the UI
---------------

After a quick tour of the data layer is now the turn of the UI part which
can be found in the `app.core namespace <https://github.com/fpischedda/blog-feedback-app/blob/master/src/cljs/app/core.cljs>`_.
This namespace make more sense if read bottom to top while the db namespace is
better read top to bottom.

At start, the application looks for all section elements, stores the section id
and name of the section in the detabase and attaches a feedback component to it.
This can be seen it the following code fragment:

.. code:: clojure

  (defn init-app
    "fetches all elements with a section class, parse all sections,
     remove prevoius feedback related elements (useful when reloading script)
     and add them to the db, finally attach a `feedback` button to all
     sections' container div"
    [db]
    (let [sections (dom/getElementsByClass "section")]
      (doseq [section sections]
        (attach-feedback-component-to-section section db))
      (start-listener!))) ;; start-listener will be addressed later

Each component will receive the database instance and will react to changes;
this is possible because a database is basically an atom so using rum/reactive
mixin works out of the box, here is the main component's code:

.. code:: clojure

  (rum/defc feedback-component < rum/reactive
    "feedback area is composed by an input text for the author,
     a textarea for the message and a div containing all previous messages
     for the current section identified by `section-id`"
    [section-id db]
    (let [db (rum/react db) ;; <- the component will react to changes of the db
          area-id (str "feedback-area-" section-id)
          comments (sorted (db/section-comments db section-id))
          comment-count (db/section-comments-count db section-id)]
      ;; view code omitted for brevity
      ))

The following binding does most of the magic:

.. code:: clojure

  db (rum/react db)

Simplifying, it subscribes to the db atom and derefs it, like we would do
with @db or (deref db); a this point it will be possible to use the returned
reference in queries like:

.. code:: clojure

  comments (sorted (db/section-comments db section-id))

Closing words
-------------

The rest of the application details are out of scope and it should be trivial
to figure them out by reading the documentation of libraries used to build the
application.

I hope that this quick tour of a simple application may have given you some
pointers to start to work with Datascript and eventually Rum.

Links section
-------------

- `Learn Datalog today <http://www.learndatalogtoday.org/>`_
- `A shallow dive into DataScript internals <https://tonsky.me/blog/datascript-internals/>`_
- `DOMAIN MODELING WITH DATALOG by Norbert Wojtowicz <https://www.youtube.com/watch?v=oo-7mN9WXTw&amp;feature=youtu.be>`_
- `Mook, a novel approach to frontend state management with ClojureScript and React <https://lambdam.com/blog/2020-09-mook/>`_ : This great article is not Datalog specific but provides a good motivation to use Datascript plus some more interesting bits of information

.. raw:: html

  <style>
    .invisible {
      display: none;
    }
  </style>
  <script type="text/javascript" src="/js/app.js"></script>
