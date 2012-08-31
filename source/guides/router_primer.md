# Understanding the Ember.js Router: A Primer

## Introduction

<!--- {{{1 -->

Using the [Ember.Router][EmberRouter] is the preferred pattern for building large
applications in [Ember.JS][EmberSite].  Ember's approach is to conceive
of your application as a collection of states which can be accessed via
both unique internal specifiers (a "route path") _as well as_ by URL locations
that "route" to those states.

This approach has many advantages:

  * Employs the [State Machine][StateMachine] pattern
  * Functional states of the application define the URL, not vice-versa
  * Nesting of routes has built-in support
  * On entry to these states, recursive chains of views can be
    automatically instantiated and inserted _without your having to manage them!_

This guide is designed to help the reader accomplish the following:

  * Understand key terminology
  * Set up an [Ember.Router][EmberRouter]
  * Lay the conceptual groundwork to ease understanding of the [Ember Application Structure][OutletGuide] guide

The [Ember Application Structure][OutletGuide] guide introduces both the
[Ember.Router][EmberRouter] as a means for routing requests *as well as*
documents how to present views based on those routed-to states.  While that
document completely addresses both of these crticial components, this primer
drills into the function of the Router and, it is hoped, makes the 
[Ember Application Structure][OutletGuide] guide more readily digestible.  As a first
step, in understanding the Router, we establish a common vocabulary of
activities and nouns.

<!--- }}}1 -->

## Definitions

<!--- {{{1 -->

In common parlance, several of the activities that are essential to the
router's function are conflated.  To ensure clarity this guide defines
*navigation*, *routing*, *state* / *route*, and *nesting* as specific
terms of art.

### Navigation

Let us first consider the most visible activty associated with any web
application:  *navigation*.  Navigation is (1) the act of following a link whose
`href` attribute corresponds to a URL or (2) entering a URL into some sort of
HTTP processing application.  Navigation occurs when:

* Lauren types `http://emberjs.com` into her browser
* Erik writes a Ruby script that accesses an application's RESTful
  API at `http://example.com/api/widgets/list`

In Ember, by default, this looks like `http://example.com/#/posts` or
`http://localhost:4567/#/cars`.  Throughout the rest of this guide, the default
location manager, "HashLocation,"  which signifies various routes by means of
`/#/`, will be used.  If this syntax is not desired, the `location` property on
the Router can be set to **any** `Ember.Object` that implements the methods specified
in the `Ember.Location` API.  For more information, consult the `Ember.Location`
[API][E.L.API].

[E.L.API]: https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/location/api.js "Ember.Location API Source Code"

### Routing, State, and Route

A *State* is a deterministic configuration of the application.  It can be
imagined as a set of variables, a set of visible views, a set of instantiated
objects, etc.  A state is referencedd by a unique specifier called a
*routePath*.  Ember *routePaths*  looks like `root.index` or
`posts.post.comment`.

When a *State* can be invoked by means of a navigable URL, it is said to
be a *Route*.  *Routing* is the act of mapping a URL onto a *routePath*.
Thus a correspondence between URL and State emerges:
`http://example.com/#/posts` would (sensibly) map to
`posts.index`.

### Nesting

As mentioned previously, Routes can be nested.  When a route has
sub-routes it is called the *parent* route and its sub-routes are called
*child* routes and / or *leaf* routes.

**Parent routes are not-routable**.  Their children bear the responsibility of
constructing the state.

<!--- }}}1 -->

## Intersection of Navigation URLs, Routes, and States

Given that there is a correspondence between navigational endpoints (URLs),
their routePaths, and those routes' "states," it is obvious that an Ember
application that uses the Router is a [State Machine][StateMachine] that uses
routePaths and / or URLs to change into the states.

## Summation of Definitions' Interplay and Roles

<!--- {{{1 -->

Ergo Ember will support navigable URLs:

  * http://example.com/app/#/posts
  * http://example.com/app/#/about
  * http://example.com/app/#/users

that, when parsed, will correspond to some sort of "route" structure
whose names are arbitrarily linked to the endpoints' names:

    carsRoute: {
     route: '/cars'
    }

which, in turn, will bear the responsibility of creating the necessary
Model, Controller, View and instances required to support a given set of
functionality.

With this general groundwork in place, we shall now consider the
behavior of a user on a site given this common vocabulary.  Thence we
shall undertake a series of exercises that will create a basic routable
application.

<!--- }}}1 -->

## Practical Application

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

The following snippet should be considered the standard, minimal, Ember
application:


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

This will load cleanly, according to the console.  Lamentably this is a rather
visually dull site because no visual data has been wired up.  We'll remedy that
in a hurry.

<!--- }}}2 -->

Since this `ApplicationView` has no `template` attribute associated with it, it
is desirable to add some *other* visual output to demonstrate that things are
working.  We can add `console.log` actions to the `enter` property of the
`Ember.Route`s (as specified in their superclass, [`Ember.State`][EmberState],'s
API documentation).  

Another tool for providing output that confirms that our implementation of the
router is correct is to enable logging of the router's decision process.  To do
so we set the `enableLogging` property to `true` within the router.  When the
browser's debug console is open, the router will print helpful error messages
beginning with `STATEMANAGER`.

Lastly, as a point of formatting, when one to examines the Ember source one sees
*liberal* use of vertical whitespace.  Just as vertical whitespace helps
separate logical "paragraphs" of operations in code, so too is it appropriate to
use vertical whitespace to create logical groupings of routes.

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

### Aside:  Required Components

<!--- {{{1 -->

#### Root "Route"

<!--- {{{2 -->
While the guidelines given in Step Two are sufficient to create a
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

<!--- }}}2 -->

#### ApplicationView and ApplicationController

<!--- {{{2 -->
Ember is opinionated, and we believe that to be A Good Thing.
Ember.Views should only be concerned with presentation and
event-handling logic: e.g. recognizing a click event, hiding a widget on
the screen.  Logic outside of this scope should be handled on an
instance of a controller.

By Ember convention this should be the same name as the Ember.View
subclass (minus the "View" substring) and with "Controller" appended.
Thus an ApplicationView should have an ApplicationController.

The latter will be instantiated by the Application's #initialize()
function *automatically*, provided the conventions are followed and will
be initialized as the controller class name with the first letter
lower-case e.g. App.applicationController.

While this View/Controller structuring convention is handy, it is
required in the case of ApplicationController and ApplicationView.
ApplicationView is the top-level view of the entire application.  It is
the View whose template contains the `{{outlet}}` call(s) into which
other views will be injected.  It presents the hangars onto which other
views can append themselves.  As such, Ember requires these classes to
be defined on your Ember.Application subclass *before*
Ember.Application#initialize() is called.

Given that Controllers are generally in the business of managing a model
instance (a collection of things or a thing) two proxying subclasses of
Ember.Controller exist:  `ObjectController` and `ArrayController`.
Depending on the nature of your application, one of these is likely to
provide extra help to your implementation.


<!--- }}}2 -->

<!--- }}}1 -->

### Step Three:  Embellishment

<!--- {{{1 -->

Some further embellishment of the routing application should help
provide some additional insight as to what a filled-out Router looks
like.  It will also be helpful for the following step where the
programmatic traversal of routes is demonstrated.

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

<!--- }}}2 -->

<!--- }}}1 -->

### Step Four:  Programmatically Affecting State Change

<!--- {{{1 -->

Routes can be transitioned to programmatically by invoking them.  This
is accomplished by aquiring the router object.  As part of
Ember.Application#initialize(), the Router class is instantiated as
App.router.

In this example, one can affect a transition with
`App.get('router').transitionTo('root.cars')`.  The console output will
confirm that the sibling routes were moved through, but note that the
parent state **was not** moved through.

<figure>
  <img src="/images/routing-primer/transition-to-in-router.png">
</figure>

With these tools in place, you're now able to see the that you have
routable hash URLS linked to distinct application states.  This in and
of itself is very powerful, but thus far our only visual feedback has
been to print diagnostic data to the console.  In the [Ember Application
Structure][OutletGuide] guide, the binding of views and controllers to
Routes is demonstrated.

## Conclusion

You now have an understanding of the Router.  Grasping the linking of
Routes to views should prove a snap!  That's the focus of the [Ember
Application Structure][OutletGuide] guide.  As always, it pays to make
sure you understand the Router's function and assumptions.

[EmberSite]: http://emberjs.com/ "Ember.JS Homepage"
[StateMachine]: http://en.wikipedia.org/wiki/Finite-state_machine "Wikipedia definition of a State Machine"
[EmberState]: https://github.com/emberjs/ember.js/blob/master/packages/ember-states/lib/state.js "Ember.State Source Code"
[EmberRouter]: https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/router.js "Ember.Router Source Code"
[EmberRoute]: https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/route.js "Ember.Route Source"
[OutletGuide]: http://emberjs.com/guides/outlets  "Ember Application Structure Guide"

<!-- vim: set fdm=marker ft=markdown: -->
