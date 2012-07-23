# Ember Application Structure

## Introduction

<!--- {{{1 -->

Using the [Ember.Router][EmberRouter] is the preferred pattern for building large
applications in [Ember.JS][EmberSite].  Ember's approach is to conceive
of your application as a collection of states which can be accessed via
both unique internal specifiers _as well as_ by URL paths that "route"
to those states.

This approach has many advantages:

  * Employment of the [State Machine][StateMachine] pattern
  * Functional states of the application define the URL, not vice-versa
  * Based on entry of these states, recursive chains of views can be
    automatically instantiated and inserted _without your having to manage them!_
  * Built-in support for the nesting of routes

This guide is designed to help the reader accomplish the following:

  * Understand key terminology
  * Set up an [Ember.Router][EmberRouter]
  * Have the Router create and display views whose content is informed by the active
[Ember.Route][EmberRoute]

As a first step, we establish a common vocabulary of activities and
nouns.

<!--- }}}1 -->

## Definitions

<!--- {{{1 --> 

In common parlance, several of the activities that are essential to the
router's function are conflated.  To ensure clarity this guide defines
*navigation*, *routing*, *state* / *route*, and *nesting* as specific
terms of art.

### Navigation

Let us first consider the most visible activty associated with any web
application:  *Navigation*.  Navigation is the act of following a link
whose `href` attribute corresponds to a URL or entering a URL into some
sort of HTTP processing application.  Navigation occurs when:

* Lauren types `http://ember.js.com` into her browser
* Erik writes a Ruby script that accesses an application's RESTful
  API at `http://example.com/api/widgets/list`

In Ember this looks like `http://example.com/#/posts` or `http://localhost:4567/#/cars`.

### Routing, State, and Route

A *State* is a deterministic configuration of the application.  It can
be imagined as a set of variables, a set of visible views, a set of
instantiated objects, etc.  A state can be transitioned into by means of
a unique trigger that activates this configuration.  In Ember this looks
like `root.index` or `posts.post.comment`.

When a *State* can be invoked by means of a navigable URL, it is said to
be a *Route*.  *Routing* is the act of mapping a URL onto a *route*.
Thus a correspondence between URL and State emerges:
`http://example.com/#/posts` would sensibly map to
`posts.index`.<sup>1</sup>

### Nesting


As mentioned previously, Routes can be nested.  When a route has
sub-routes it is called the *parent* route and its sub-routes are called
*child* routes and / or *leaf* routes.  

**Parent routes are not-routable**.  They transfer the responsibility of
instantiating the state to a series of child states.

<!--- }}}1 --> 

## Intersection of Navigation URLs, Routes, and States

Given that there is a correspondence between navigational endpoints
(URLs), their routes, and those routes' "states," it is obvious that an
Ember application that uses the Router is a [State Machine][StateMachine].

## Summation of Definitions' Interplay and Roles

<!--- {{{1 --> 

Ergo Ember will support navigable URLs:
  * http://example.com/app/#/posts
  * http://example.com/app/#/about
  * http://example.com/app/#/users

that, when parsed will correspond to some sort of "route" structure
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

## Routing

<!--- {{{1 --> 

A user navigates through your application by making choices about what
to view. For example, in a blog application, a user might choose between
a "Posts" and an "About" page.  In general, you want to have a default
for preferred choice (in this case, probably Posts).

Once the user has made their first choice, they're usually not done. In
the context of Posts, the user might eventually view an individual post
and its comments. Inside of an individual post, they could choose between
viewing a list of comments and a list of trackbacks.

Importantly, in all of these cases, the user's navigational choice is
informing what to display on the page. As they descend deeper into your
application's state, those choices affect smaller areas of the page.

Let's undertake an exercise to see how we might implement such an
application.

<!--- }}}1 --> 

## Practical Application

### Step One: Minimum Non-Viable Router

<!--- {{{1 -->
This step:

1.  Creates an Ember.Application
1.  Gives preliminary debugging output 
1.  Produces one critical error

    window.App = Ember.Application.create({
      ready: function(){
        console.log("Created App namespace");
      },

      Router: Ember.Router.extend({
      })
    });

    App.initialize();

When we run this application with our console open, we see the onReady
event fire.  In the console `Created App namespace` appears.  But an
error appears as well:   `Uncaught Error: assertion failed: Failed to
transition to initial state "root" `.  

The Router class of of an Ember application **must** contain an
Ember.State called `root`.  As a legacy of Ember's history and
inheritance chain, `State` is a synonym for `Route` (see discussion
above).  As such, an `Ember.Route` called `root` must be
added.<sup>2</sup>  

The amended code will look like the following:

    window.App = Ember.Application.create({
        Router: Ember.Router.extend({
          root:  Ember.Route.extend({
          })
        })
    });
    App.initialize();

<!--- }}}1 -->

### Step Two: Minimum Non-Viable Router Part II

<!--- {{{1 -->

Running the previous code *again* produces an error:  `Uncaught Error: assertion
failed: ApplicationView and ApplicationController must be defined on
your application`.  

An Ember application using the router must define both
ApplicationController and ApplicationView.  In conjunction with a `root`
route that has a routable child, these are the only requirements for an
Ember application that makes use of the Router.

The following snippet should be considered the standard, minimal, Ember
application:


<!--- {{{2 -->
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

<!--- }}}2 -->

Since this application has no views associated with it, it is hard to
see that it is working.  We can add `console.log` actions to the `enter`
property of the Ember.Routes (as specified in their superclass
Ember.State's API documentation).

<!--- {{{2 -->
    window.App = Ember.Application.create({
        ApplicationView: Ember.View.extend(),
        ApplicationController: Ember.Controller.extend(),
        Router: Ember.Router.extend({
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

<!--- }}}2 -->

<!--- }}}1 -->

### Step Three:  Embellishment

<!--- {{{1 -->

Some further embellishment of the routing application should help
provide some additional insight as to what a filled-out Router looks
like.  It will also be helpful for the following step where the
programmatic traversal of routes is demonstrated.

<!--- {{{2 --> 
    window.App = Ember.Application.create({
        ApplicationView: Ember.View.extend(),
        ApplicationController: Ember.Controller.extend(),

        Router: Ember.Router.extend({
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

<!--- }}}2 --> 

<!--- }}}1 --> 

### Step Four:  Programmatically Affecting State Change

Routes can be transitioned to programmatically by invoking them.  This
is accomplished by aquiring the router object.  As part of
Ember.Application#initialize(), the Router class is instantiated as
App.router.  

In this example, one can affect a transition with
`App.get('router').transitionTo('root.cars')`.  The console output will
confirm that the sibling routes were moved through, but note that the
parent state **was not** moved through.

With these tools in place, you're now able to see the that you have
routable hash URLS linked to distinct application states.  This in
and of itself is very powerful, but thus far our only visual feedback
has been to print diagnostic data to the console.  In the next section
we will join the routing machine to the view application.

## Managing Views from Within the Router





In the next section, we'll cover how you control these areas of the
page. For now, let's look at how to structure your templates.

When the user first enters the application, the application is on the
screen, and it has an empty outlet that the router will control. In
Ember, an `outlet` is an area of a template that has its child template
determined at runtime based on user interaction.

<figure>
  <img src="/images/outlet-guide/application-choice.png">
</figure>

The template for the Application (`application.handlebars`) will look
something like this:

```
<h1>My Application</h1>

{{outlet}}
```

By default, the router will initially enter the _list of posts_ state,
and fill in the outlet with `posts.handlebars`. We will see later how
this works exactly.

<figure>
  <img src="/images/outlet-guide/list-of-posts.png">
</figure>

As expected, the _list of posts_ template will render a list of posts.
Clicking on the link for an individual post will replace the contents of
the application's outlet with the template for an individual post.

The template will look like this:

```
{{#each post in controller}}
<h1><a {{action showPost context="post" href=true}}>{{post.title}}</a></h1>
<div>{{post.intro}}</div>
{{/each}}
```

When clicking on a link for an individual post, the application will
move into the _individual post_ state, and replace `posts.handlebars` in
the application's outlet with `post.handlebars`.

<figure>
  <img src="/images/outlet-guide/individual-post.png">
</figure>

In this case, the individual post also has an outlet. In this case, the
outlet will allow the user to choose between viewing comments or
trackbacks.

The template for an individual post looks like this:

```
<h1>{{title}}</h1>

<div class="body">
  {{body}}
</div>

{{outlet}}
```

Again, the `{{outlet}}` simply specifies that the router will make the
decision about what to put in that area of the template.

Because `{{outlet}}` is a feature of all templates, as you go deeper
into the route hierarchy, each route will naturally control a smaller
part of the page.

## How it Works

Now that you understand the basic theory, let's take a look at how the
router controls your outlets.

### Templates, Controllers, and Views

First, for every high-level handlebars template, you will also have a
view and a controller with the same name. For example:

* `application.handlebars`: the template for the main application view
* `App.ApplicationController`: the controller for the template. The
  initial variable context of `application.handlebars` is an instance of
  this controller.
* `App.ApplicationView`: the view object for the template.

In general, you will use view objects to handle events and controller
objects to provide data to your templates.

Ember provides two primary kinds of controllers, `ObjectController` and
`ArrayController`. These controllers serve as proxies for model objects
and lists of model objects.

We start with controllers rather than exposing the model objects
directly to your templates so that you have someplace to put
view-related computed properties and don't end up polluting your models
with view concerns.

You also connect `{{outlet}}`s using the template's associated
controller.

### The Router

Your application's router is responsible for moving your application
through its states in response to user action.

Let's start with a simple router:

```javascript
App.Router = Ember.Router.extend({
  root: Ember.State.extend({
    index: Ember.State.extend({
      route: '/',
      redirectsTo: 'posts'
    }),

    posts: Ember.State.extend({
      route: '/posts'
    }),

    post: Ember.State.extend({
      route: '/posts/:post_id'
    })
  })
});
```

This router sets up three top-level states: an index state, a state that
shows a list of posts, and a state that shows an individual post.

In our case, we'll simply redirect the index route to the `posts` state.
In other applications, you may want to have a dedicated home page.

So far, we have a list of states, and our app will dutifully enter the
`posts` state, but it doesn't do anything. When the application enters
the `posts` state, we want it to connect the `{{outlet}}` in the
application template. We accomplish this using the `connectOutlets`
callback.

```javascript
App.Router = Ember.Router.extend({
  root: Ember.State.extend({
    index: Ember.State.extend({
      route: '/',
      redirectsTo: 'posts'
    }),

    posts: Ember.State.extend({
      route: '/posts',

      connectOutlets: function(router) {
        router.get('applicationController').connectOutlet('posts', App.Post.find());
      }
    }),

    post: Ember.State.extend({
      route: '/posts/:post_id'
    })
  })
});
```

This connectOutlet call does a few things for us:

* It creates a new instance of `App.PostsView`, using the
  `posts.handlebars` template.
* It sets the `content` property of `postsController` to a list of all
  of the available posts (`App.Post.find()`) and makes `postController`
  the controller for the new `App.PostsView`.
* It connects the new view to the outlet in `application.handlebars`.

In general, you should just think of these objects as operating in
tandem. You will always provide the content for a view's controller when
you create a view.

## Transitions and URLs

Next, we will want to provide a way for an application in the `posts`
state to move into the `post` state. We accomplish this by specifying a
transition.

```javascript
posts: Ember.State.extend({
  route: '/posts',
  showPost: Ember.State.transitionTo('post'),

  connectOutlets: function(router) {
    router.get('applicationController').connectOutlet('posts', App.Post.find());
  }
})
```

You invoke this transition by using the `{{action}}` helper in the
current template.

```
{{#each post in controller}}
  <h1><a {{action showPost context="post" href=true}}>{{post.title}}</a></h1>
{{/each}}
```

When a user clicks on a link with an `{{action}}` helper, Ember will
dispatch an event to the current state with the specified name. In this
case, the event is a transition.

Because we used a transition, Ember was also able to generate a URL for
this link. Ember uses the `id` property of the context to fill in the
`:post_id` dynamic segment of the `post` state.

Next, we will need to implement `connectOutlets` on the `post` state.
This time, the `connectOutlets` method will receive the post object
specified as the context to the `{{action}}` helper.

```javascript
post: Ember.State.extend({
  route: '/posts/:post_id',

  connectOutlets: function(router, post) {
    router.get('applicationController').connectOutlet('post', post);
  }
})
```

To recap, the `connectOutlet` call performs a number of steps:

* It creates a new instance of `App.PostView`, using the
  `post.handlebars` template.
* It sets the `content` property of `postController` to the post that
  the user clicked on.
* It connects the new view to the outlet in `application.handlebars`.

You don't have to do anything else to get the link (`/posts/1`) to work
if the user saves it as a bookmark and comes back to it later.

If the user enters the page for the first time with the URL `/posts/1`,
the router will perform a few steps:

* Figure out what state the URL corresponds with (in this case, `post`)
* Extract the dynamic segment (in this case `:post_id`) from the URL and
  call `App.Post.find(post_id)`. This works using a naming convention:
  the `:post_id` dynamic segment corresponds to `App.Post`.
* Call `connectOutlets` with the return value of `App.Post.find`.

This means that regardless of whether the user enters the `post` state
from another part of the page or through a URL, the router will invoke
the `connectOutlets` method with the same object.

## Nesting

Finally, let's implement the comments and trackbacks functionality.

Because the `post` state uses the same pattern as the `root` state, it
will look very similar.

```javascript
post: Ember.State.extend({
  route: '/posts/:post_id',

  connectOutlets: function(router, post) {
    router.get('applicationController').connectOutlet('post', post);
  },

  index: Ember.State.extend({
    route: '/',
    redirectsTo: 'comments'
  }),

  comments: Ember.State.extend({
    route: '/comments',
    showTrackbacks: Ember.State.transitionTo('trackbacks'),

    connectOutlets: function(router) {
      var postController = router.get('postController');
      postController.connectOutlet('comments', postController.get('comments'));
    }
  }),

  trackbacks: Ember.State.extend({
    route: '/trackbacks',
    showComments: Ember.State.transitionTo('comments'),

    connectOutlets: function(router) {
      var postController = router.get('postController');
      postController.connectOutlet('trackbacks', postController.get('trackbacks'));
    }
  })
})
```

There are only a few changes here:

* We specify the `showTrackbacks` and `showComments` transitions only in
  the states where transitioning makes sense.
* Since we are setting the view for the outlet in `post.handlebars`, we
  call `connectOutlet` on `postController`
* In this case, we get the content for the `commentsController` and
  `trackbacksController` from the current post. The `postController` is
  a proxy for the underlying Post, so we can retrieve the associations
  directly from the `postController`.

Here's the template for an individual post.

```
<h1>{{title}}</h1>

<div class="body">
  {{body}}
</div>

<p>
  <a {{action showComments href=true}}>Comments</a> |
  <a {{action showTrackbacks href=true}}>Trackbacks</a>
</p>

{{outlet}}
```

And finally, coming back from a bookmarked link will work fine with this
nested setup. Let's take a look at what happens when the user enters the
site at `/posts/1/trackbacks`.

* The router determines what state the URL corresponds with
  (`post.trackbacks`), and enters the state.
* For each state along the way, the router extracts any dynamic segments
  and calls `connectOutlets`. This mirrors the path a user would take as
  they move through the application. As before, the router will call the
  `connectOutlet` method on the post with `App.Post.find(1)`.
* When the router gets to the trackbacks state, it will invoke
  `connectOutlets`. Because the `connectOutlets` method for `post` has
  set the `content` of the `postController`, the trackbacks state will
  retrieve the association.

Again, because of the way the `connectOutlets` callback works with
dynamic URL segments, the URL generated by an `{{action}}` helper is
guaranteed to work later.

## Asynchrony

One final point: you might be asking yourself how this system can work
if the app has not yet loaded Post 1 by the time `App.Post.find(1)` is
called.

The reason this works is that `ember-data` always returns an object
immediately, even if it needs to kick off a query. That object starts
off with an empty `data` hash. When the server returns the data,
ember-data updates the object's `data`, which also triggers bindings on
all defined attributes (properties defined using `DS.attr`).

When you ask this object for its `trackbacks`, it will likewise return
an empty `ManyArray`. When the server returns the associated content
along with the post, ember-data will also automatically update the
`trackbacks` array.

In your `trackbacks.handlebars` template, you will have done something
like:

```
<ul>
{{#each trackback in controller}}
  <li><a {{bindAttr href="trackback.url"}}>{{trackback.title}}</a></li>
{{/each}}
</ul>
```

When ember-data updates the `trackbacks` array, the change will
propagate through the `trackbacksController` and into the DOM.

You may also want to avoid showing partial data that is not yet loaded.
In that case, you could do something like:

```
<ul>
{{#if controller.isLoaded}}
  {{#each trackback in controller}}
    <li><a {{bindAttr href="trackback.url"}}>{{trackback.title}}</a></li>
  {{/each}}
{{else}}
  <li><img src="/spinner.gif"> Loading trackbacks...</li>
{{/if}}
</ul>
```

When ember-data populates the `ManyArray` for the trackbacks from the
server-provided data, it also sets the `isLoaded` property. Because all
template constructs, including `#if` automatically update the DOM if the
underlying property changes, this will "just work".


----

Footnotes:

1.  Ember.JS is even coded in a fashion such that it codifies these
    definitions.  An `Ember.Route` is the result of taking an
    Ember.State and mixing in a Mix-In called `Ember.Routable.`

1.  For those wondering why this is the case, `Ember.Route =
    Ember.State.extend(Ember.Routable);`.  Ain't polymorphism grand?
2.  This would imply that it is possible to perform multiple
    `connectOutlet` actions in the `connectOutlets` function.  This is
just so.

3.  The statement that: "Top-level `Ember.Route`s should be given a
    default `index` sub-route" might lead some clever readers to wonder
    whether they *must* be given this route.  The answer is no, but a
    default sub-route called `index` or something else obvious helps get
    around several problems.  A precis of why this is so follows, but can be
    safely ignored for guide purposes.  

    The problem is that in a router, events in a higher level of the
    hierarchy are available to the lower level.  This seemingly unnecessary
    requirement that this "index" route wrap the `route` and
    `connectOutlets` is a way to allow the Ember.Router to be defined as a
    nesting structure (which is a powerful aid to the programmer,
    conceptually) but ensures that the routes keep proper privacy about the events
    that they respond to.

    Granted it is also possible to define events higher-up in the
    nesting that you would like available to all descendants (e.g. an action
    on root that all sidebars should be aware of).


[EmberSite]: http://emberjs.com/ "Ember.JS Homepage"
[StateMachine] http://en.wikipedia.org/wiki/Finite-state_machine "Wikipedia definition of a State Machine"
[EmberRouter]: #
[EmberRoute]: #



#===============================================================================
#===============================================================================
TRASH

This is caused by the Ember.Router not being able to find a very special
method called `connectOutlets.`  If the yin of translating URLs to
routes is the `route` property, the yang of binding it to application
state is the method `connectOutlets`.  By default `connectOutlets` is
defined as Ember.K, that is, a function that returns the context in
which it was found.  

The Router needs more than this :).

*This method is the linch-pin in understanding Ember.Router*.  

Because of efforts undertaken by the Ember team, several conventions
over configuration and some logical guesses occur in `connectOutlets`
and `connectOutlet`.  This allows the method to be exceedingly terse,
and powerful.

In d06f4a4a, Trek Glowacki put in some great documentation that will
help us out in the section "Changing View Hierarchy in Response To State
Change".  

Given the following connectOutlets definition:

    connectOutlets: function(router, context) {
      router.get('oneController').connectOutlet('another');
    },

Trek wrote:

    This will detect the '{{outlet}}' portion of `oneController`'s view
    (an instance of `App.OneView`) and fill it with a rendered instance of
    `App.AnotherView` whose `context` will be the single instance of
    `App.AnotherController` stored on the router in the `anotherController`
    property.

*Whoa!*  That is a lot of magic that is not clearly explained.  Here's
how it works.  Let's follow Trek's description and figure out what we
need to provide Ember.

First, Trek tells that a whole lot of things happen as a result of this
*one* line of code.  The only parts of the code that are configurable
happen to be in the parentheses.  `oneController` and `another`.  The
entire logical edifice Trek described follows as a result of applying
the conventions to these to parameters.

### Starting with `one`

It works like this:  the `App wordController` is assumed to have a
`App.WordView` which, in its `templateName` named file or in the
compiled Handlebars code specified by `template` contains the
`{{outlet}}` helper.  Obviously, then, we can see how Trek was able to
write this:

    This will detect the '{{outlet}}' portion of `oneController`'s view
    (an instance of `App.OneView`)

### Conjunction

Trek then describes that the {{outlet}} will be filled with the
"rendered instance of App.AnotherView.  The code that specifies this
"interpolate into `{{outlet}}`" is `connectOutlet` (Nota Bene:  this is
the _singular_ form!<sup>2</sup>).


### The Rest

Accordingly, the rest of the assignment must be "magicked" into place by
providing the word "another."

    `App.AnotherView` whose `context` will be the single instance of
    `App.AnotherController` stored on the router in the `anotherController`
    property.

### The Shopping List

Based on Trek's description it would seem we need the following:

* App.OneController
* App.OneView which has a template with `{{outlet}}`
* App.AnotherController
* App.AnotherView

Let's put these four things into our code and see what happens:

  window.App = Ember.Application.create({
    AnotherController:  Ember.Controller.extend(),
    AnotherView:  Ember.View.extend(),

    OneController:  Ember.Controller.extend(),
    OneView:  Ember.View.extend({
      template:  Ember.Handlebars.compile("This is OneView: {{outlet}}")
    }),

    Router: Ember.Router.extend({
      root:  Ember.Route.extend({
        route: '/',
        enter: function ( router ){
          console.log("The root state was entered.");
        },
      })
    })
  });
  App.initialize();

We get a new error:  "Uncaught TypeError: Cannot call method
'connectOutlet' of undefined."  This comes about because the root route
needs to have its default route "wrapped."  Top-level `Ember.Route`s
should be given a default `index` sub-route to function.  So let's wrap
this guy up.<sup>3</sup>

    window.App = Ember.Application.create({
      AnotherController:  Ember.Controller.extend(),
      AnotherView:  Ember.View.extend({
        template:  Ember.Handlebars.compile("This is Another...")
      }),

      ApplicationController:  Ember.Controller.extend(),
      ApplicationView:  Ember.View.extend({
        template:  Ember.Handlebars.compile("This is ApplicationView:
    {{outlet}}")
      }),

      Router: Ember.Router.extend({
        root:  Ember.Route.extend({
          index:  Ember.Route.extend({
            route: '/',
            enter: function ( router ){
              console.log("The root state was entered.");
            },
            connectOutlets:  function ( router, context ){
              router.get('applicationController').connectOutlet('another');
            }
          })
        })
      })
    });
    App.initialize();

`connectOutlets` should look like the following:

      connectOutlets: function(router, context) {
        router.get('oneController').connectOutlet('another');
      }

`connectOutlet`'s first argument is `router`, that means the `Router`
in whose scope this method is being typed!  `context` is an object full
of data that will be used to fill out the view (parameters pulled out of
the URL, or things looked up and fabricated in order for the view to be
informative).


Making this method seem  _even more_ mysterious is that it does a number
of things _by convention_.  


Let's try to make Trek's `connectOutlets` work in our code.  Let's
define the smallest possible OneController and OneView:

    OneController: Ember.ObjectController.extend(),
    OneView:  Ember.View.extend({
      template:  Ember.Handlebars.compile("<p>OneView has:</p> {{outlet}}")
    }),





--------------------------------------------------------------------------------
