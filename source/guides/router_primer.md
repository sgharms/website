# Understanding the Ember.js Router: A Primer

## Introduction

<!--- {{{1 -->

Using the [Ember.Router][EmberRouter] is the preferred pattern for building large
applications in [Ember.JS][EmberSite].  Ember's approach is to conceive
of your application as a collection of states which can be accessed via
both unique internal specifiers (a "route path") _as well as_ by URL locations
that "route" to those states.

This approach has many advantages:

  * It Employs the [State Machine][StateMachine] pattern
  * Functional states of the application define the URL, not vice-versa
  * Nesting of routes has built-in support
  * On entry to these states, recursive chains of views can be
    automatically instantiated and inserted _without your having to manage them!_

This guide is designed to help the reader accomplish the following:

  * Understand key terminology
  * Set up an [Ember.Router][EmberRouter] in a series of small,
    incremental improvements upon a minimal, viable application

<!--- }}}1 -->

## Definitions

<!--- {{{1 -->

In common parlance, several of the activities that are essential to the
router's function are conflated.  To ensure clarity this guide defines
several terms of art.

### Navigation

Navigation is (1) the act of following a link whose `href` attribute
corresponds to a URL or (2) entering a URL into some sort of HTTP
processing application.  Navigation occurs when:

* Lauren types `http://emberjs.com` into her browser
* Erik writes a Ruby script that accesses an application's RESTful
  API at `http://example.com/api/widgets/list`

### State

A *State* is a deterministic configuration of the application.  It can be
imagined as a set of variables, a set of rendered views, a set of instantiated
objects, etc.  `State`s are collected into [StateManager][StateManager]s.

### State Manager

A container that holds an arbitrary number of unique states.

### Router

While Ember supports having many `StateManagers`, that address a wide variety
of purposes (handling what's in a status console, toggling lightbox effects,
etc.), a StateManager dedicated to bringing about certain application states is
a Router.  Fittingly, in the Ember source, a `Router` has an is a sublcass of
`StateManager`.

### Route

Just as a Router is a special case of a `StateManager`, a `Router`'s
consitituent `State`s are given the special designation of `Route`.

### RoutePath 

A `Route` is referred to by a unique specifier called a routePath.  Ember
*routePaths*  look like `root.index` or `posts.post.comment`.

### Routing

Routing is the act of mapping a URL onto a routePath.  Thus a correspondence
between URL and State emerges: `http://example.com/#/posts` would (sensibly)
map to `posts.index`.

### Routable URL

A routable URL is a URL that triggers a transition to a Route.  In Ember, by
default, this looks like `http://example.com/#/posts`,
`http://localhost:4567/#/cars`, `http://localhost:4567/#/post/show/1`.  

The relevant part of the url is the part that is featured after the `#/`.  In
the descriptions above, the Routable URLs will trigger the Router to change
state to either `posts`, `cars`, or `post.show`.  If the use of this
"hash-slash" delimiter is not desirable, it can easily be
changed<sup>[1](#location-manager)</sup>.

### Nesting

As mentioned previously, Routes can be nested.  When a route has sub-routes it
is called the *parent* route and its sub-routes are called *child* routes and /
or *leaf* routes.

**Parent routes are not-routable**.  Their children bear the responsibility of
constructing the state.  Otherwise said, all routable states **must be** leaf
routes.  A typical RoutePath for a nested route would be `post.show` and its
matching URL would be `#/post/show/1`.

<!--- }}}1 -->

## Intersection of Navigation URLs, Routes, and States

Given that there is a correspondence between navigational endpoints (URLs),
their routePaths, and those routes' "states," it is obvious that an Ember
application that uses the Router is a [State Machine][StateMachine] that uses
routePaths and / or URLs to change into the states.

## Summation of Definitions' Interplay and Looking Ahead

<!--- {{{1 -->

Given the preceeding definitions, we should be able to imagine an Ember
application that uses the Router.  While the exact how-to will come next, if we
wanted to talk about shoes and cars, we would imagine an application that does
something when handed URLs like:

  * http://example.com/app/#/shoes
  * http://example.com/app/#/shoe/blue-suede
  * http://example.com/app/#/cars

Given the association between a routable URL to a routePath thence to a
Route, it is logical to expect that we will build a Route that looks, as
a naive implementation, like:

    carsRoute: {
     route: '/cars'
    }

We also know that `Route`s are aggregated into a `Router`.  The only piece
missing, therefore, is something that translates the routable URL and then
loads up the applicate state associated with the routable URL's `Route`.

In this example, it turns out that the Router is both the collection of Routes,
but also bears the responsibility of creating the necessary Model, Controller,
View and instances required to support a given set of functionality in a Route.

In the subsequent sections, therefore, this guide will demonstrate how to
construct a Router, an abstraction that:

* Defines a collection of `State`s, i.e. `Route`s
* Puts the browser into those states based on the routable URL
* Delegates model, controller, and view construction to its contained routes

With this general groundwork in place, we shall now undertake a series of
exercises that will create a basic routable application.

<!--- }}}1 -->

## Practical Application

<!-- {{{1 -->

### Method

<!-- {{{2 -->

#### Text

<!-- {{{3 -->

This guide attempts to allow the reader to follow the incremental building-up
of a router-driven application.  You should be able to follow along on your
tablet on the train and, with some focus, follow along.  As such the code is
*frequently* re-listed.

<!-- }}}3 -->

#### Hands-on Code

<!-- {{{3 -->

If you are in a situation where you have access to a computer, this guide also
has reference to a [GitHub](http://github.com) project whose commits match each
of these incremental steps.  As such, you can clone the repository, check out
the commit under discussion, adjust the code, try things out, and then move to
the next step.

The code is housed in a minimal application framework built on Ruby and Sinatra
called [Halbert](https://github.com/sgharms/halbert).  If the terms "git,"
"Sinatra," and "Ruby" are all confusing to you, feel free to merely follow
along with the text and / or copy-and-paste the code listings into your
development environment.

<!-- }}}3 -->

<!-- }}}2 -->

### Step One: Minimum Non-Viable Router

<!--- {{{1 -->

This step:

1.  Creates an Ember.Application
1.  Gives preliminary debugging output
1.  Produces one critical error

```javascript
window.App = Ember.Application.create({
  ready: function(){
    console.log("Created App namespace");
  },

  Router: Ember.Router.extend({
  })
});

App.initialize();
```

The result should look like the following when we run this application with our
console open:

<img src="/images/routing-primer/app-without-root-route.png">

An error appears:   `Uncaught Error: assertion failed: Failed to transition to
initial state "root" `.

The Router class of of an Ember application **must** contain an Ember.State
called `root`.  As a legacy of Ember's history and inheritance chain, `State` is
a synonym for `Route` (see discussion above).  As such, the error message can be
read as saying that the application could not find an `Ember.Route` called
`root`.  Let's add it.

The amended code will look like the following:

```javascript
window.App = Ember.Application.create({
  ready: function(){
    console.log("Created App namespace");
  },
  Router: Ember.Router.extend({
    root:  Ember.Route.extend({
    })
  })
});

App.initialize();
```

Let's give it a reload and see what happens....

<!--- }}}1 -->

### Step Two: Minimum Non-Viable Router Part II

<!--- {{{1 -->

Running the previous code *again* produces an error:  `Uncaught Error: assertion
failed: ApplicationView and ApplicationController must be defined on
your application`.

<img src="/images/routing-primer/no-app-controller-defined.png">

An Ember application using the router **must** define **both**
`ApplicationController` and `ApplicationView`.  In conjunction with a `root`
route that has a routable child, these are the only requirements for an
Ember application that makes use of the Router.

<!--- }}}1 -->

### Step Three: Minimum Viable Router

<!--- {{{1 -->

Let's address the error of the previous section and, in to doing, create the
standard, minimal, Ember application:


<!--- {{{2 -->
```javascript
window.App = Ember.Application.create({

    ApplicationView: Ember.View.extend(),
    ApplicationController: Ember.Controller.extend(),

    Router: Ember.Router.extend({
      root:  Ember.Route.extend({
        index:  Ember.Route.extend({
          route:  '/'
        })
      })
    })
});
App.initialize();
```

<!--- }}}2 -->

<!--- {{{2 -->
This will load cleanly, according to the console.

Lamentably this is a rather visually dull site because no templates chocked full
of beautiful HTML and styled by beautiful CSS have been wired up (yet).  While
we'll remedy that in a hurry but there are other ways of printing diagnostic
data.  It behooves the Ember developer to know some of these techniques for
debugging and Router "sketching" purposes.


First, we can add `console.log` actions to the `enter` or `exit` properties of an
`Ember.Route`.

Another tool for providing output that confirms that our implementation of the
router is correct is to enable logging of the router's decision process.  To do
so we set the `enableLogging` property to `true` within the Router.  When the
browser's debug console is open, the router will print helpful error messages
beginning with `STATEMANAGER`.

Lastly, as a point of formatting, when one to examines the Ember source one
sees *liberal* use of vertical whitespace.  Just as vertical whitespace helps
separate logical "paragraphs" of operations in code, so too is it appropriate
to use vertical whitespace to create logical groupings of controllers and
views.  Both of these, per the developer's &aelig;sthetic sensibility, might
might be set off from simple properties.  Here's a minimum viable application
with diagnostic data enabled.

<!--- {{{2 -->
```javascript
window.App = Ember.Application.create({

    ApplicationView: Ember.View.extend(),
    ApplicationController: Ember.Controller.extend(),

    Router: Ember.Router.extend({
      enableLogging:  true,
      root:  Ember.Route.extend({
        enter:  function(){
          console.log("entered root");
        },
        index:  Ember.Route.extend({
          enter:  function(){
            console.log("entered index");
          },
          route:  '/'
        })
      })
    })
});
App.initialize();
```

An initial load of this application looks like the following:


<figure>
  <img src="/images/routing-primer/initial-load-router.png">
</figure>
<!--- }}}2 -->

<!--- }}}1 -->

### Steps One Through Three In Review:  Required Components

<!--- {{{1 -->

#### Root "Route"

<!--- {{{2 -->
While the guidelines given in Step Two are sufficient to **create** a
baseline routing application, this section provides the "why" an
application must meet the requirements.

By default, when the Router is instantiated it automatically moves to
the state `root`.  This is the "default landing place" for the Router.
`root` **is not a route** itself.  It is the box which contains the
routes, but it is not a route.  It bears repeating:  **You cannot make
this default landing place routable.**  As such if `root` is not present
the Router should, and does, report an error.

If an application has only one route, as does our example thus far, it
should have a sub-Route, by convention, called `index` that answers to
the navigable hash-bang url `/`; that is, the default URL to navigate to
the application.

This Route can either wire-up views or it can redirect to another Route.

<!--- }}}2 -->

#### ApplicationView and ApplicationController

<!--- {{{2 -->
Ember is opinionated, and we believe that to be A Good Thing.

Ember.Views should only be concerned with presentation and
event-handling logic: e.g. recognizing a click event, hiding a widget on
the screen, etc.  Logic outside of this scope should be handled on an
instance of a controller which has a tight coupling to the view.

We can see the tightness of this coupling by virtue of the naming convention.
For views that will be wired up by the Router, when given a base name (e.g
'BaseName'), there should be an `App.BaseNameView` and an
`App.BaseNameController.` These router-wire-able types of views will be
automatically instantiated by the call to `#initialize()` into instances
corresponding to the convention such that `BaseNameView`'s instance is
`App.baseNameView` and the controller, similarly, becomes
`App.baseNameController`.

As we seek to "wire up" these view instances in the router, they will need
a primordial "thing" on which to hang.  They will need a special view that
is guaranteed to exist.  These instances need a primordial `Em.View` on which
to "hang."  By Ember convention that primoridal, base view is an instance of
`App.ApplicationView`  called `App.applicationView`.  Its associated controller
is, as the conventions should help you surmise, `App.applicationController`.
It is for this reason that Ember threw an error in Step II when we defined
a router-based application *without* these two critical classes.

<!--- }}}2 -->

<!--- }}}1 -->

<!--- }}}1 -->

### Step Four:  A New Hope

<!--- {{{1 -->

Let us now create a new Router-implementing Ember application.  We need
something slightly less trivial than what has been demonstrated before so that
we may discern the state-transitioning features afforded by the Router.

<!--- {{{2 -->
```javascript
window.App = Ember.Application.create({
    ApplicationView: Ember.View.extend(),
    ApplicationController: Ember.Controller.extend(),

    Router: Ember.Router.extend({
      enableLogging:  true,

      root:  Ember.Route.extend({
        enter: function ( router ){
          console.log("The root state was entered.");
        },
        index:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The index sub-state was entered.");
          },
          route: '/'
        }),
        shoes:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The shoes sub-state was entered.");
          },
          route: '/shoes'
        }),
        cars:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The cars sub-state was entered.");
          },
          route: '/cars'
        })
      })
    })
});
App.initialize();
```

This application follows the format of the previous application but with the
addition that the routes now each have an `enter` callback which will be called
when that state is entered.  There is also an `exit` callback.

<!--- }}}2 -->

### Step Five:  Programmatically Affecting State Change

<!--- {{{1 -->

Routes can be transitioned into programmatically by invoking
`Ember.Router#transitionTo('stateName')`.

In the web console of a browser running the code provided in the previous
section, you can try out the first method by invoking the `transitionTo` method
on the `App.Router` class's instance, `App.router` like so:
`App.router.transitionTo('root.cars')`.  Two important things happen:

1. The [State Manager][StateManager] logging output states that the `cars
sub-state was entered`
1. The URL slug becomes affixed with `#/cars`

The first point is the expected result under the principle of least surprise,
we changed the state.  The second result is that the slug updated the URL to
express a URL path that uniquely maps to a state in the state machine.

We can change to the other state with: `App.router.transitionTo('root.shoes')`.
Again, as expected, we enter that Route state and see a change in the URL slug to
`#/shoes`.

It is worth noting that when transferring between sibling states (`cars` to
`shoes`) the parent state was *not* entered.  This is confirmed both my the
state manager's debugging  output as well as by the non-execution of the
`root.index`'s `enter()` callback.

Recall that in the introduction we said that each state was uniquely identified
by a routePath (e.g. `root.cars`) that could be uniquely bonded to a URL.  By
changing a state by means of `transitionTo` and a `routePath` we affect a
change in the URL.  Contrawise one should expect that a change in the URL
navigated to should afect a change in the state loaded &#x2014; *and that is
exactly what happens!*

That is the purest essence of the Ember.Router:  an
[Ember.StateManager][StateManager] whose states are uniquely addressable via
`routePath`s **or** URLs.

<figure>
  <img src="/images/routing-primer/move-between-routes-transitionto-urlslug.png">
</figure>

The exact methods by which the Router knows how to translate a URL slug into a
series of objects required for a state is controlled my a method on Route
called `deserialize`.  The means by which a state expresses its required
components based on the contents of the url is controlled by a method called
`serialize.`  We'll see more about the control and customization of objects
based on route states later on in this guide, but what's important to take away
is that Ember provides means for easily controlling what needs to happen based
on the slug and for fabricating a string (the slug) which were it to be passed
to the router via URL navigation or bookmark would allow it to pull up the
right data objects needed to fulfill the expectations of the Route.

Let's set that concern aside for a moment and look at how to move our
interaction with the app from merely logging diagnostic data based on Route /
State change to something more visually stimulating.

<!--- }}}1 -->

### Step Six:  Wiring Up Views in Routes: The {{outlet}}

<!--- {{{1 -->

Above we mentioned that the primordial view is the `ApplicationView`.  We can
imagine that its associated template would likely have some skeletal, static
HTML in it:  a logo, a static footer, and some other assets.

But there are also going to be components whose structural position always
remains the same, but whose content changes.  We could imagine that a "top
navigation" area (or "View") will always happen in the same place, but whose
data source might contain different content when loaded (e.g. "logged-in
navigation" or "admin navigation");

The place where this appropriate-to-the-State View "plugs in" is an "`outlet`."
While "outlet" has many subtle meanings in English, in Ember case it means the
place where Views get plugged in.

We denote an outlet by putting the Em.Handlebars helper `{{outlet}}` inside of
our Views' template code.  A template can have multiple outlets which are
separated by name (e.g. `{{outlet topNav }}` and `{{outlet leftNav}}`).  In many
templates, however, there will be only one `outlet`.

For didactic purposes this guide will **not** be putting outlet declarations
inside of another file.  Tutorials more generally focused on Ember's basic
use cover the use of how a `templateName` property in a View can be used
to refer to a `<script>`-based Handlebars template.  Since the views in this
document are contrived, and to make for easier phone / tablet reading, the
template code will be compiled *in situ* in the Views.

The following listing creates a view at the top level, the requisite
`ApplicationView`, and gives it a simple template with a single `outlet`
statement.

<!--- {{{2 -->
```javascript
window.App = Ember.Application.create({
    ApplicationView: Ember.View.extend({
      template:  Ember.Handlebars.compile("<p>App.View:</p><p>{{outlet}}</p>"),
    }),
    ApplicationController: Ember.Controller.extend(),

    Router: Ember.Router.extend({
      enableLogging:  true,

      root:  Ember.Route.extend({
        enter: function ( router ){
          console.log("The root state was entered.");
        },
        index:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The index sub-state was entered.");
          },
          route: '/'
        }),
        shoes:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The shoes sub-state was entered.");
          },
          route: '/shoes'
        }),
        cars:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The cars sub-state was entered.");
          },
          route: '/cars'
        })
      })
    })
});
App.initialize();
```

<!--- }}}2 -->

When this application is loaded, the in-line template
(`ApplicationView.template`) is rendered to the screen.  While the `template`'s
Handlebars code presented the opportunity to connect something to its
`{{outlet}}` "hook,"  we didn't do that.  Unsurprisingly, no view  was
rendered.

`Em.Route`s afford a way to connect outlets programmatically.  One does so in a
method called, fittingly, `connectOutlets`.  This method is provided as a
mixed-in method from the Routable mixin.  While not strictly knowlegede at this
juncture, it should remind the reader that the Router is a `StateManager` &plus;
`Routable`.

Let's suppose that in the `cars` state one wants to wire up a view into
`ApplicationView`'s `outlet` that displays something `car`-related.

Let's first look at the signature of `connectOutlets`:

    connectOutlets:  function(router){...}

`connectOutlets` is passed one argument by default: the router itself &endash;
the selfsame router in which the method is defined!  While this may seem unusual
at first glance, merely accepting this convention does not impair effective use
of the router. 

This parameter should be used as a means for accessing the controllers and
views that were automatically instantiated by the router during the
Em.Application#initialize() call.  While `App.router.myController` is the same
thing as `router.get('myController')`, the former makes assumptions about the
namespace of the application and should be avoided.  By using the `router`
parameter, developers ensure that they're not peeking into the mechanics of the
Ember.Application internal machinery.

Let's flesh out our connectOutlets method some more:

    connectOutlets:  function(router){
        router.get('applicationController').connectOutlet('shoes');
    }

Recall earlier that we noted that a given view might have multiple outlets?
That's why the name of this method is `connectOutlet`**s**.   Within this
method, one will invoke (0:many) calls to `connectOutlet` on a controller that
is bonded (by naming convention) to a view.  One can read this line of code as:

> "Router, find me the instance of your class App.ApplicationController called
> App.applicationController which, by convention has an instance of the view
> App.ApplicationView called App.applicationView whose template declaration
> defines an outlet.  I wish to take an instance of App.ShoesView and inject its
> template data into the outlet."

`connectOutlet` has very flexible arguments invocations accepting either: 

* a single argument, a string (common) which Ember will apply some conventions to to derive:
  * A view source
  * A controller
  * Ember will assume the view with the outlet will have only one `{{outlet}}` call 
  * Ember will assume the view you're injecting will have the data it needs in its controller's `content` property
* three arguments defining:
  * a named outlet: if both `{{outlet alpha}}` and `{{outlet beta}}` are
present, a string `'beta'` would tell Ember you want to inject onto the `beta`
outlet * a name whence is derived a viewSource and controller
  * a data object that will be assigned to the controller's `content` attribute 
* object of parameters that defines, even more granularly, the specifics of
wiring up a view to an outlet.  

<!--- {{{2 -->
```javascript
window.App = Ember.Application.create({
    ApplicationView: Ember.View.extend({
      template:  Ember.Handlebars.compile("<p>App.View:</p><p>{{outlet}}</p>"),
    }),
    ApplicationController: Ember.Controller.extend(),

    ShoesView:  Em.View.extend({
      template:  Ember.Handlebars.compile("<p>App.ShoesView:</p><p>{{outlet}}</p>"),
    }),
    ShoesController: Ember.Controller.extend(),

    Router: Ember.Router.extend({
      enableLogging:  true,

      root:  Ember.Route.extend({
        enter: function ( router ){
          console.log("The root state was entered.");
        },
        index:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The index sub-state was entered.");
          },
          route: '/'
        }),
        shoes:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The shoes sub-state was entered.");
          },
          route: '/shoes',
          connectOutlets:  function(router){
            router.get('applicationController').connectOutlet('shoes');
          }
        }),
        cars:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The cars sub-state was entered.");
          },
          route: '/cars'
        })
      })
    })
});
App.initialize();
```
<!--- }}}2 -->

The application should look something like this:

<img src="/images/routing-primer/initial-view-wireup.png">

To show the flexibility the Router affords via `connectOutlets`, here's an example of how to use `{{outlets}}` in more complex views:

<!-- {{{2 -->
```javascript
window.App = Ember.Application.create({
    ApplicationView: Ember.View.extend({
      template:  Ember.Handlebars.compile("<p>App.View:</p><p>{{outlet alpha}}</p><hr/>{{outlet beta}}"),
    }),
    ApplicationController: Ember.Controller.extend(),

    ShoesView:  Em.View.extend({
      template:  Ember.Handlebars.compile("<p>App.ShoesView:</p><p>{{outlet list}}</p>"),
    }),
    ShoesController: Ember.Controller.extend(),

    ListOfShoesController:  Em.ArrayController.extend(),
    ListOfShoesView:  Em.View.extend({
      template:  Em.Handlebars.compile("{{#each shoe in controller}}<li>{{shoe.name}}</li>{{/each}}"),
    }),

    FooterController:  Em.ObjectController.extend(),
    FooterView:  Em.View.extend({
      template:  Em.Handlebars.compile("This is the footer: {{message}}"),
    }),

    Router: Ember.Router.extend({
      enableLogging:  true,

      root:  Ember.Route.extend({
        enter: function ( router ){
          console.log("The root state was entered.");
        },
        index:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The index sub-state was entered.");
          },
          route: '/'
        }),
        shoes:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The shoes sub-state was entered.");
          },
          route: '/shoes',
          connectOutlets:  function(router){
            router.get('applicationController').connectOutlet('beta', 'footer', { message: "agony"} );
            router.get('applicationController').connectOutlet('alpha', 'shoes');

            var listOfShoes = [ { name: "Rainbow Sandals" }, { name:  "Strappy shoes" }, 
                { name:  "Blue Suede" } ];
            router.get('shoesController').connectOutlet('list', 'listOfShoes', listOfShoes);

          }
        }),
        cars:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The cars sub-state was entered.");
          },
          route: '/cars'
        })
      })
    })
});
App.initialize();
```
<!-- }}}2 -->

Using the information thus far provided, the documentation for connectOutlet,
and some patience, the flexibility and power of the `connectOutlets` method
should help you see ways of building powerful views whose components can be
easiely swapped out: no longer must you troll through endless template code
removing (or adding) conditional clauses.  Simply create a new view / controller
pair, unplug the old, and plug in the new.  

The Router's design with `{{outlet}}`s opens many exciting new possibilities:
A/B testing, "admin" interfaces versus "regular" interfaces, portable layouts,
etc.

Now that we've demonstrated how to wire up a substantial view quickly, we need
to provide a way, via the views, to affect Route state change.  This is
addressed in the following section.


<!--- }}}1 -->

## Moving from Route to Route

<!--- {{{1 -->

Recall that we have already demonstrated that routes can be changed with calls
to `App.get('router').transitionTo('someState')`.  This change can also be
triggered by  executing the `transitionTo` action as a result of a
browser event: typically, click.  This section will show how to change state as
a result of an event.  The subsequent section will cover how to effect state
change as a result of the URL slug &mdash; the case that truly lives up to the name
"Router."

Frequently a click event will cause a change in state.  A click on a post's
title in a blog application should transition state to one where that route is
shown.  A click on a link for "Dinner" should change the way that the menu's
consitiuent views are wired up such that **steak frites** are on the menu
versus **croque madame**.  

Looking at our toy application, let's transition from the `shoes` state to the
`cars` state.  We'll add a link in the `ShoesView` that will take us to the
`CarsView`.  I'll edit the temlate for `ShoesView` to be the following:

      template:  Ember.Handlebars.compile(
      "<p>App.ShoesView:</p><p>{{outlet list}}</p>" + 
      "<p><a {{action goToCars href=true}}>Go To Cars</a>"),

The meat of the addition is:

      {{action goToCars href=true}}

The `{{action}}` Handlebars helper is used throughout Ember's Handlebars
templates to handle eventing in Ember (click, keyDown, [**etc.**][EventList]).
If we change this line and navigate to the `/#/shoes` view, we receive an error
from Ember warning that `goToCars` was not defined.

[EventList]: http://docs.emberjs.com/symbols/Ember.View.html

In Router-driven applications if an action is not intercepted by a view, that
event will bubble up to the Route in which that view was rendered.  If that
Route is a sub-route of another Route the transition will be sought there all
the way up to the `Router` definition. 

In this case, Ember looks to App.ShoesView to handle the click, it does not.
It then bubbles upward until it hits the current Route and then bubbles out to
any Route containing that Route.  In this case `goToCars` is not found
**anywhere**.  As such, Ember raises an error.

To alleviate this, we must define the `goToCars` transition somewhere in the
event bubbling chain.  But what should `goToCars` be defined to?  It should be
defined to a function generated by `Ember.Route.transitionTo`.  When passed a
single argument of a routePath, this method generates a function which
transitions states.  In this case, our `goToCars` should be defines to be
`Ember.Route.transitionTo('root.cars')`.  We can put this inside the `shoes`
route or in the `root` route.  For more generally applicable transitions, we
can our transition at the root Route.  For more specific transitions, it might
be more appropriate to have it with a very limited visibility in a very
specific sub-sub-sub-route.

As such we need to add the line to the `Router` declaration:

    goToCars:  Ember.Route.transitionTo('root.cars')

With these two changes in place, a new link appears for 'Go To Cars' when
visiting `/#/shoes`.  Clicking on it, as the console debugging log shows,
transports the view construct to `root.cars`.

While the state URL slug have both changed, we have not implemented
`connectOutlets` in `root.cars`.  As such the view looks the same as
`/#/shoes`.  Let's `connectOutlets` for `root.cars`.

    connectOutlets:  function(router,context){
      router.get('applicationController').connectOutlet('alpha', 'car');
    }

This also will require the addition of a `CarView` and `CarController`.  They
are the following:

    CarController:  Em.ObjectController.extend(),
    CarView:  Em.View.extend({
      template:  Em.Handlebars.compile("In the year 2012 we locomote by exploding dead sauruses.  Try some <a {{action goToShoes href=true}}>shoes</a>"),
    }),

You'll see that CarView has a link **back** to the `root.shoes` route.  This
will require the addition of a `goToShoes` transition.  To show the flexibility
of where these transitions can be placed, this transition will be localized
into the `root.cars` route versus the more global definition of `goToShoes`.

      goToShoes:  Em.Route.transitionTo('root.shoes'),

The final code to allow movement between states by virtue of DOM click events is
the following:

<!-- {{{2 -->
```javascript
/* Define the namespace, models, views, and controllers */
window.App = Ember.Application.create({
    ApplicationView: Ember.View.extend({
      template:  Ember.Handlebars.compile("<p>App.View:</p><p>{{outlet
alpha}}</p><hr/>{{outlet beta}}"),
    }),
    ApplicationController: Ember.Controller.extend(),

    ShoesView:  Em.View.extend({
      template:  Ember.Handlebars.compile("<p>App.ShoesView:</p><p>{{outlet
list}}</p><p><a {{action goToCars href=true}}>Go To Cars</a>"),
    }),
    ShoesController: Ember.Controller.extend(),

    ListOfShoesController:  Em.ArrayController.extend(),
    ListOfShoesView:  Em.View.extend({
      template:  Em.Handlebars.compile("{{#each shoe in
controller}}<li>{{shoe.name}}</li>{{/each}}"),
    }),

    FooterController:  Em.ObjectController.extend(),
    FooterView:  Em.View.extend({
      template:  Em.Handlebars.compile("This is the footer: {{message}}"),
    }),

    CarController:  Em.ObjectController.extend(),
    CarView:  Em.View.extend({
      template:  Em.Handlebars.compile("In the year 2012 we locomote by
exploding dead sauruses.  Try some <a {{action goToShoes
href=true}}>shoes</a>"),
    }),

    ShoeDetailController:  Em.ObjectController.extend(),
    ShoeDetailView:   Em.View.extend({
      template:  Em.Handlebars.compile("Detail:  {{content}}")
    }),

    Router: Ember.Router.extend({
      enableLogging:  true,
      goToCars:  Em.Route.transitionTo('root.cars'),

      root:  Ember.Route.extend({
        enter: function ( router ){
          console.log("The root state was entered.");
        },
        index:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The index sub-state was entered.");
          },
          route: '/'
        }),

        shoes:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The shoes sub-state was entered.");
          },
          route: '/shoes',
          connectOutlets:  function(router){
            router.get('applicationController').connectOutlet('beta', 'footer',
{ message: "agony"} );
            router.get('applicationController').connectOutlet('alpha', 'shoes');

            var listOfShoes = [ { name: "Rainbow Sandals" }, { name:  "Strappy
shoes" }, 
                { name:  "Blue Suede" } ];
            router.get('shoesController').connectOutlet('list', 'listOfShoes',
listOfShoes);
          },
        }),

        cars:  Ember.Route.extend({
          goToShoes:  Em.Route.transitionTo('root.shoes'),
          enter: function ( router ){
            console.log("The cars sub-state was entered.");
          },
          route: '/cars',
          connectOutlets:  function(router,context){
            router.get('applicationController').connectOutlet('alpha', 'car');
          }
        })
      })
    })
});
App.initialize();
```
<!-- }}}2 -->

<!--}}}1-->

## Cleaning up Our Data Sources

<!--- {{{1 -->

In this example I have provided data to each of the routes by using some sort
of locally-defined data strucutre e.g. the variables `shoeList` and
`listOfShoes`.  Clearly, there is a better way to get data.

Ember's opinion is that you should ask your Model classes how to retrieve
instances or collections of that model.  This should look *somewhat* similar to
those coming from Rails.  In the same way as one might express `Post.all` or
`Post.find(id)`, Ember perfers a similar method of declaration: the "fetching"
work should occur as a "class method" on an Ember model.

Class method, above, appears in quotes to note that we fully understand that no
such thing exists in the Javascript language per se.  Let's structure the code
for gathering the "list of shoes."   In the `root.shoes` route we have the
following code: 

    var listOfShoes = [ { name: "Rainbow Sandals" }, { name:  "Strappy shoes" }, { name:  "Blue Suede" } ];
    router.get('shoesController').connectOutlet('list', 'listOfShoes',
listOfShoes);

One way of reading this, if we may think back to a C-influenced style of
thinking, is this:

1.  Construct an array of objects
1.  Said array of objects can be "pointed to" by a reference.  For clarity
    purposes this will be called `0x000016`
1.  Give the reference to that array of objects to the variable `listOfShoes`
1.  Give the `ShoeController` arguments of (list, name, contextReference) where
    contextReference points to the reference of our array of objects, that is
    to the object at address `0x000016`.

Let's make it such that the reference object comes back from something
semantically appropriate.  The Ember preference would be to call the method:
`App.Shoe.all()` which returns a unique reference address which, for the
lifetime of the app, will be the source of data for the `listOfShoesController`
which will inform the `listOfShoesView`.

Why all this discussion of references?  If one were to replace
listOfController.content with another array (a different array reference), the
view, which is watching the old reference, **would not know to update itself**.
This is a very common error for those learning to use the Router and to
integrate it with model-held data methods.

Here's the code for the `all()` method on `App.Shoe`:

```javascript

App.Shoe = Ember.Object.extend();
App.Shoe.reopenClass({
  _listOfShoes:  Em.A(),

  all:  function(){
    var allShoes = this._listOfShoes;

    // Mock an ajax call
    setTimeout( function(){
      allShoes.clear();
      allShoes.pushObjects(
        [ 
          { id: 'rainbow',   name: "Rainbow Sandals" },
          { id: 'strappy',   name: "Strappy shoes" },
          { id: 'bluesuede', name: "Blue Suede" } 
        ]
      );
    }, 2000);

    return this._listOfShoes;
  }
});
```

After the `App.create` (but before the `App.initialize`!), we add the previous
listing.  From the declaration inside `root.shoes` here's what happens:

1.  Declare a class `App.Shoe`
1.  Provide a unique, "private" attribute  called `_listOfShoes`
1.  Provide an `all()` method.

Then the router is initialized and all the Routes are created and attached to
the `App.router` instance variable.  When the `root.shoes` route is entered.
`connectOutlets` fires and the following happens:

1.  `App.Shoe.all` is called
1.  A local reference (`allShoes`) to a persistent address (`App.Shoe._listOfShoes`) is gained 
1.  An asynchronous call is made (mocking an AJAX call) that updates in 2
    seconds time
1.  The reference address to `App.Shoe._listOfShoes` is returned, a pointer to an
    empty array (`Em.A()`)
1.  _Asynchronously_ the local reference is updated ( it is empted by the
    `clear()` method and then loaded with the hard-coded array set

Thanks to Ember's bindings, when this is viewed in the browser, the
`listOfShoesView` renders an empty array ("do an each of my controller's content,
nothing in there, show nothing").  Two seconds later that reference's contents
change and the view is notified of this change ("do an each of my controller's
content, there are items in there, render them").

It might have been tempting to do the following in `all`:

```javascript
App.Shoe = Ember.Object.extend();
App.Shoe.reopenClass({
  _listOfShoes:  Em.A(),

  all:  function(){
    var allShoes = this._listOfShoes;

    // Mock an ajax call
    setTimeout( function(){
      this._listOfShoes = 
        [ 
          { id: 'rainbow',   name: "Rainbow Sandals",
              price: '$60.00', description: 'San Clemente style' },
          { id: 'strappy',   name: "Strappy shoes",
              price: '$300.00', description: 'I heard Pénèlope Cruz say this word once.' },
          { id: 'bluesuede', name: "Blue Suede",
              price: '$125.00', description: 'The King would never lie:  TKOB⚡!' } 
        ]
      );
    }, 2000);

    return this._listOfShoes;
  }
});
```
By **assigning** a new object reference ID, we have left the view looking at
the **old** reference address but have put our new data at a **new** address.
**Your view will not update** if you make this error.

<!--- {{{2 -->
<!--- }}}2 -->

<!--- }}}1 -->

While we're here, let's add a find method that will let us look up a single
Shoe by its ID:

```javascript
  find:  function(id){
    return this._listOfShoes.findProperty('id', id);
  },
```
With a model in place that can do our data fetching, let's make it possible
to go from viewing a list of shoes, to viewing the details about a shoe.

## From a List of Shoes to a Shoe

Here's the battle plan:

1.  Make a click on the shoe listing do something
1.  Make a something that receives that noficiation change
1.  Make the notification say somethign like: such-and-such shoe's details
    should be shown

In Ember, this is surprisingly easy and should feel a bit familiar if you've
been following along with the guide thus far.

## Add Action Signaling

In the handlebars template for listOfShoesView, we'll add an action call and
turn the list items' content into anchor tags;

    <li><a{{action showShoe shoe href=true}}>{{shoe.name}}</a></li>

## Add a trigger

In the `root.shoes` route add:

    showShoe:  Em.Route.transitionTo('root.shoe.showShoe')

## Implement the showShoe Route

Place this at the root level of `App.Router`, on the same level as `cars` or
`shoes`.

```javascript
    shoe:  Em.Route.extend({
      index:  Em.Route.extend({
        route:  '/shoe/',
      }),

      show:  Em.Route.extend({
        route:  '/shoes/:name',
        connectOutlets:  function(router, context){
        }
      })
    }),
```

With this in place, if you view `#/shoes`, will have clickable list items.

Try this out and you'll see that:

1.  The url for the clicked-on entry takes us to `#/shoes/Strappy shoes`
1.  The views did not change

We've seen #2 before, it's because our connectOUtlets is not doing anything.
the first case is a bit different though.  The issue comes about because the
link items don't know how to express their underlying `shoe` as a url, they
need to learn how to `serialize` themselves.  If a Shoe were trying to express
how to find it in URL form, how would it point to itself?  We will write that
method for the route as such:

    serialize:  function(router, context){
      return {
        name: context.id
      }
    }

It should not be surprising that a Route that explains how to serialize its
objects should also need to explain how to **de**serialize its items when a
meaningful URL is given.


    deserialize:  function(router, context){
      return App.Shoe.find( context.name );
    },

Serialize works by receiving the router as an argument and the Ember object
that needs to know how to "URL-ify" itself.  In this case the Shoe returns a POJsO
whose `name` property points to it the Shoe's id.  Why did it choose `name`?
It's because `:name` is specified as a slug on the `root.shoe.show`'s `route`
property.  Conversely, deserialize will take a slug ending in #/shoes/:name.
`name` is set as a property on context, it contains the slugs (the parts in
place of the colon-led strings) as keys.  As such one can expect that
`#/shoe/strappy` will create, in context, a key of name which has the value of
'strappy.'   

From this 'context' object we can look to our find method to re-vivify the
object entered in `serialize`.

With these components in place, we should not be able to move from `shoes` so
`/shoes/strappy`.

Here's the end result:

<!-- {{{2 -->

```javascript
window.App = Ember.Application.create({
    ApplicationView: Ember.View.extend({
      template:  Ember.Handlebars.compile("<p>App.View:</p><p>{{outlet alpha}}</p><hr/>{{outlet beta}}"),
    }),
    ApplicationController: Ember.Controller.extend(),

    ShoesView:  Em.View.extend({
      template:  Ember.Handlebars.compile("<p>App.ShoesView:</p><p>{{outlet list}}</p><p><a {{action goToCars href=true}}>Go To Cars</a>"),
    }),
    ShoesController: Ember.Controller.extend(),

    ListOfShoesController:  Em.ArrayController.extend(),
    ListOfShoesView:  Em.View.extend({
      template:  Em.Handlebars.compile("{{#each shoe in controller}}<li><a{{action showShoe shoe href=true}}>{{shoe.name}}</a></li>{{/each}}"),
    }),

    FooterController:  Em.ObjectController.extend(),
    FooterView:  Em.View.extend({
      template:  Em.Handlebars.compile("This is the footer: {{message}}"),
    }),

    CarController:  Em.ObjectController.extend(),
    CarView:  Em.View.extend({
      template:  Em.Handlebars.compile("In the year 2012 we locomote by exploding dead sauruses.  Try some <a {{action goToShoes href=true}}>shoes</a>"),
    }),

    ShoeDetailController:  Em.ObjectController.extend(),
    ShoeDetailView:   Em.View.extend({
      template:  Em.Handlebars.compile("Detail for {{name}}: [{{price}}] {{description}} ")
    }),

    Router: Ember.Router.extend({
      enableLogging:  true,
      goToCars:  Em.Route.transitionTo('root.cars'),

      root:  Ember.Route.extend({
        index:  Ember.Route.extend({
          enter: function ( router ){
            console.log("The index sub-state was entered.");
          },
          route: '/'
        }),

        shoes:  Ember.Route.extend({
          showShoe:  Em.Route.transitionTo('root.shoes.ashoe'),
          route: '/shoes',

          index:  Em.Route.extend({
            route: '/',
            connectOutlets:  function(router){
              router.get('applicationController').connectOutlet('alpha', 'shoes');
              router.get('applicationController').connectOutlet('beta', 'footer', { message: "agony"} );

              router.get('shoesController').connectOutlet('list', 'listOfShoes', App.Shoe.all() );
            },
          }),

          ashoe:  Em.Route.extend({
            route:  '/:name',
            connectOutlets:  function(router, context){
              var ac = router.get('applicationController');
              ac.connectOutlet('alpha', 'shoeDetail', context );
              ac.connectOutlet('beta', 'footer', { message: "I feel de agony of de footer"} );
            },

            deserialize:  function(router, context){
              var ashoe = App.Shoe.find( context.name );
              return ashoe;
            },

            serialize:  function(router, context){
              return {
                name: context.id
              }
            }
          })
        }),


        cars:  Ember.Route.extend({
          goToShoes:  Em.Route.transitionTo('root.shoes'),
          enter: function ( router ){
            console.log("The cars sub-state was entered.");
          },
          route: '/cars',
          connectOutlets:  function(router,context){
            router.get('applicationController').connectOutlet('alpha', 'car');
          }
        }),

      })
    })
});

App.Shoe = Ember.Object.extend();
App.Shoe.reopenClass({
  _listOfShoes:  Em.A(),

  find:  function(id){
    return this._listOfShoes.findProperty('id', id);
  },

  all:  function(){
    var allShoes = this._listOfShoes;

    // Mock an ajax call
    setTimeout( function(){
      allShoes.clear();
      allShoes.pushObjects(
        [ 
          { id: 'rainbow',   name: "Rainbow Sandals",
              price: '$60.00', description: 'San Clemente style' },
          { id: 'strappy',   name: "Strappy shoes",
              price: '$300.00', description: 'I heard Pénèlope Cruz say this word once.' },
          { id: 'bluesuede', name: "Blue Suede",
              price: '$125.00', description: 'The King would never lie:  TKOB⚡!' } 
        ]
      );
    }, 0);

    return this._listOfShoes;
  }
});


App.initialize();
```
<!-- }}}2 -->

<!-- }}}1 -->

## Back and Forth

So now we can go from `#/shoes` to #/shoes/strappy or #/shoes/bluesuede by
clicking.  We can also go directly to these URLs and have the state
re-vivified.  The only thing left is to add some click candy to the
ShoeDetailView template.  This should be a rehash from other actions we've
implemented thus far.

Change the template for `ShoeDetailView`

      template:  Em.Handlebars.compile("<p>Detail for {{name}}: [{{price}}] {{description}}</p><a {{action goToShoes}}>Back to Shoes</a>")

...and add an action to transfer states...


      goToShoes:  Em.Route.transitionTo('root.shoes.index'),

And we can remove the `goToShoes` trigger in the `cars` Route.

## Todo, Route nesting....

## Conclusion

With this guide you should not have a feel for how to construct an application
using Ember.JS.  You have learned how to define a router, how to define a model
class, and how each Ember application is conceived of as a finite series of
states.  We move between states by allowing the URL to dictate the state to
load, by handing an event on a trigger, or by direct console-based invocation
of Em.Routable methods which effect changes in state.


----

## Footnotes

<a id="location-manager"></a>

1.  The default location manager, "HashLocation,"  signifies various routes by
means of `/#/`.  To change this behavior, set the `location` property on the
Router to **any** `Ember.Object` that implements the methods specified in the
`Ember.Location` API.  For more information, consult the `Ember.Location`
[API][E.L.API].

[E.L.API]: https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/location/api.js "Ember.Location API Source Code"

<!-- {{{1 -->

[EmberSite]: http://emberjs.com/ "Ember.JS Homepage"
[StateMachine]: http://en.wikipedia.org/wiki/Finite-state_machine "Wikipedia definition of a State Machine"
[EmberState]: https://github.com/emberjs/ember.js/blob/master/packages/ember-states/lib/state.js "Ember.State Source Code"
[EmberRouter]: https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/router.js "Ember.Router Source Code"
[EmberRoute]: https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/route.js "Ember.Route Source"
[OutletGuide]: http://emberjs.com/guides/outlets  "Ember Application Structure Guide"
[StateManager]:  https://github.com/emberjs/ember.js/blob/master/packages/ember-states/lib/state_manager.js "Ember StateManager"
<!-- }}}1 -->

<!-- vim: set fdm=marker ft=markdown tw=79: -->
