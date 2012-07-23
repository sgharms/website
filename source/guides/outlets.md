# Ember Application Structure

On a high-level, an Ember application is created by defining a series of
`Ember.route`s that correspond to application states.  These Routes can
be created such that they contain *other* `Route`s, a pattern that is called
"_nesting_."  When these states are entered, view templates and their
associated controllers are instantiated and bound by a recursive
inter-linking of views whose selection and content is determined by the
`Route`'s state.

This guide explains how routing works in Ember and also offers
demonstration code.  It proceeds from a definitions-oriented, bottom-up
approach.

## Navigation, Routing, and State Machines

In order to explain the Router, I wish to present a series of definitions.

### Navigation

Let us first consider the most visible activty associated with any web
application:  *navigation*.  Navigation is the act of following a link
whose `href` attribute corresponds to a URL or entering a URL into some
sort of HTTP processing application.  Navigation occurs when:

* Lauren types `http://ember.js.com` into her browser
* Erik writes a Ruby script that accesses an application's RESTful
  API at `http://example.com/api/widgets/list`

### Routing

*Navigation* is a different activity than *routing*.  *Routing* is the
act of mapping a URL onto a *route*.  A *route* represents a
configuration state of the application that _need not_ have any visual
similarity to the URL which was the result of navigation  Thus for the
url `http://example.com/post/1`, a *route* might be
"`root.posts.singlepost`."

### States

The aforementioned "configuration states" are representations of
"`Ember.State`s."  They represent a point in time where certain
variables are defined, objects instantiated, configuration made,
processing done, etc.

## Intersection of Navigation URLs, Routes, and States

Given that there is a correspondence between navigational endpoints
(URLs), their routes, and those routes' "states," it is obvious that an
Ember application that uses the Router is a "[state
machine](http://en.wikipedia.org/wiki/Finite-state_machine)." 

Reconsider the introduction to this document in light of these
discussions:

    On a high-level, an Ember application is created by defining a series of
    `Ember.route`s that correspond to application states.  These Routes can
    be created such that they contain other Routes, a pattern that is called
    "_nesting_."

Ergo Ember will support navigable URLs:
  * /cars
  * /shoes
  * /users

that, when parsed will correspond to some sort of "route" structure
whose names are arbitrarily linked to the endpoints' names:

    carsRoute: {
     route: '/cars' 
    }

which, in turn, will hand to a state the responsibility of creating the
necessary Model, Controller, View and instances required to support a
given set of functionality.

Also mentioned in the introduction was a concept called _nesting_.
Just as certain navigable URLs like `/cars` make sense, in keeping with
the [REST](http://en.wikipedia.org/wiki/Representational_state_transfer)
paradigm, it makes sense to have nested navigable URLs that correspond
to nested states e.g. `/cars/create` or `/cars/show/bmw-m5`.

With this common set of terms and understanding, let's build up a simple
example.

## Routing

A user navigates through your application by making choices about what
to view. For example, in a blog application, a user might choose between
a "Posts" and an "About" page.  In general, you want to have a default
for preferred choice (in this case, probably Posts).

Once the user has made their first choice, they're usually not done. In
the context of Posts, the user might eventually view an individual post
and its comments. Inside of an individual post, they can choose between
viewing a list of comments and a list of trackbacks.

Importantly, in all of these cases, the user's navigational choice is
informing what to display on the page. As you descend deeper into your
application state, those choices affect smaller areas of the page.

Let's translate this theoretical description of "browsing a web site"
into a practical demonstration.  Let's build something that translates
URLs to routes and routes to states.  That device, in Ember, is called
an `Ember.Router`.

The most basic Ember.Application that makes use of a Router looks like
this:

    window.App = Ember.Application.create({
      ready: function(){
        console.log("Created App namespace");
      },

      Router: Ember.Router.extend({
      })
    });

    App.initialize();

Let's try running it and see what happens.  *It didn't work.*  This is OK.

First we see the successful `console.log` message as the onReady event
fires.  We are then given a helpful error message `Uncaught Error:
assertion failed: Failed to transition to initial state "root" `.  OK,
so we need to create something called `root`.  According to the error,
this `root` thing should be a *state*.  

*But we will not create a State, we will create a Route* As a legacy of
Ember's history, `state` is a synonym for `route`.  Therefore, Ember is
telling us to build an `Ember.Route` called `root.`<sup>1</sup>  Also at
this point I'll remove the preliminary ready event console logging, feel
free to keep it in.

Let's do just that:

    window.App = Ember.Application.create({
        Router: Ember.Router.extend({
          root:  Ember.Route.extend({
          })
        })
    });
    App.initialize();

Let's try running it and see what happens.  *It didn't work.*  This is
OK.  This time we get a new, helpful error:  `Uncaught Error: assertion
failed: ApplicationView and ApplicationController must be defined on
your application`.  Apparently these two names are somehow special.
Let's create them:

    window.App = Ember.Application.create({
        ApplicationView: Ember.View.extend(),
        ApplicationController: Ember.Controller.extend(),
        Router: Ember.Router.extend({
          root:  Ember.Route.extend({
          })
        })
    });
    App.initialize();

And violà, we now have an Ember application that has a single route.  To
make things more interesting, we can add an `enter` callback and print
something to the console.

    window.App = Ember.Application.create({
        Router: Ember.Router.extend({
          root:  Ember.Route.extend({
            enter: function ( router ){
              console.log("The root state was entered.");
            }
          })
        })
    });
    App.initialize();

All routes should have a propery called `route` defined on them to make
it clear which navigable URL to which they map clear.  

    window.App = Ember.Application.create({
        Router: Ember.Router.extend({
          root:  Ember.Route.extend({
            route: '/',
            enter: function ( router ){
              console.log("The root state was entered.");
            }
          })
        })
    });
    App.initialize();

When this is added, a new error appears in the console:  `Uncaught
TypeError: Object hash has no method 'setURL' `.  

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




