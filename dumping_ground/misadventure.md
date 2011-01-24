Misadventure
===

*A little game written on top of Faux and Backbone.js*

**introduction**

[Misadventure][play] is a little game in the style of [Adventure][a]. Misadventure is written in Javascript and runs entirely in the browser. Misadventure's code makes use of the [Faux][f] and [Backbone.js][b] libraries. In this essay I will give you a brief tour of Misadventure's [code][source], showing how it uses Faux to structure its routes and templates as well as how it uses Backbone.js to organize its models and interactive view code.

**the game**

Open [http://unspace.github.com/misadventure][play] in your browser.

Misadventure starts inauspiciously:

<a target="_blank" href="http://min.us/mvkEt6y"><img src="http://i.min.us/jeaApo.png" border="0"/></a>

You notice that a *fragment* of [#/wake][wake] has been added to the URL, but the base URL hasn't changed. Since Single Page Interface architecture is all the rage these days, you already know that this means that the contents of the page are being loaded into the DOM without refreshing the entire page. That requires much less bandwidth and scales faster than rendering pages from the server.

Let's get back to using the game. You have two options. If you experiment, you discover that closing your eyes doesn't appear to do anything at this point, but if you stand up and look around, you'll get something like this:

<a target="_blank" href="http://min.us/mvkEt6y#2"><img src="http://i.min.us/jefdsa.png" border="0"/></a>

The fragment has changed again, now it's [#/42492610216140747/7624672284554068][l1]. You can move North by clicking the link or pressing the up arrow on the keyboard. Things appear to be the same, but the fragment has changed again, now it's [#/42492610216140747/5682321739861935][l2]. Moving North one more time takes you to what appears to be a different page:

<a target="_blank" href="http://min.us/mvkEt6y#3"><img src="http://i.min.us/jeflO8.png" border="0"/></a>

And the fragment has changed yet again, now it's [#/42492610216140747/3916709493533819][l3]. We just moved North. What happens if we move South? Let's investigate. First, each possible move is a standard HTML link. There's no magic Javascript. Let's look at the link to move South. It's the same URL, but the fragment looks familiar: [#/42492610216140747/5682321739861935][l2]. That's interesting, it's the exact same fragment that we were on a move ago.

And looking at the page, we see that the link to move South is colored <font color='red'>red</font>. There's no special magic going on. It's a standard link and it goes to a place that's still in the browser's history, so it is styled differently, just like any other link. We'll see how that works later, but let's try something else.

Use the back button or back command in your browser. It takes you to your previous location, just as if each location int he maze is a standard web page with its own unique link. That's because each location does have a unique link and behaves exactly like a standard web page. This means that all of the things you or any user expects from a page in a browser work here. You can navigate forward and back, bookmark locations, even mail links to friends.

Did you notice that in this document we're using links for the fragments? If you travel to the game through the links, you wind up in exactly the same maze in exactly the same location. The links are *stable*. We'll come back to how that works when we look at the code, but it's a key point, so we'll emphasize it:

> Everything you see in Misadventure has a unique URL and works with your browser's existing mechanisms like bookmarks or navigating backwards and forwards. The URLs are stable and are not tied to a temporary "session."

You can continue to navigate your way through the corn maze. Try using an arrow key instead of clicking a link. With patience and a little knowledge of how to recursively search a tree, you may eventually find your way to the exit and leave the corn maze:

<a target="_blank" href="http://min.us/mvkEt6y#4"><img src="http://i.min.us/jbJZZ8.png" border="0"/></a>

And by now you will not be surprised to discover that the final page has a fragment too: [#/42492610216140747/bed][bed]. Do you want to play again? Simply click [close your eyes][wake], and you'll find yourself playing in a brand new maze, with all different fragments. Have fun and when you come back, we'll take a look at the code that makes this work. 

**summary of what we've learned from misadventure's user experience**

Before we look at the code, here's a quick summary of what we've seen:

1. Every "page" has its own unique URL
2. Pages all have the same base URL, but the fragments change
3. The DOM is being refreshed in the browser
4. The URLs are stable and can be bookmarked or shared with other users
5. All standard navigation elements (e.g. back, forwards, links) work in standard ways
6. Misadventure also supports non-standard navigation in the form of arrow keys

Now we'll see how Misadventure uses Faux and Backbone.js to make this happen.

**code overview**

Misadventure is organized in a tree:

<a target="_blank" href="http://min.us/mveGGAQ"><img src="http://i.min.us/jeaE9S.png" border="0"/></a>

Ignoring `README.md`, `docs`, `images`, and `stylesheets`, we're going to look at [index.html][index] and the two most important directories in the project: [javascripts][js] and [haml][haml].

[index]: http://github.com/unspace/misadventure/tree/master/index.html
[js]: http://github.com/unspace/misadventure/tree/master/javascripts
[haml]: http://github.com/unspace/misadventure/tree/master/haml

`index.html` contains the web page you see when you open Misadventure for the first time. Here's the source:

    <!DOCTYPE html>
    <html lang="en">
      <head>
        <title>Misadventure</title>
        <meta charset="utf-8">
        <link href="./stylesheets/application.css" rel="stylesheet">
        <!-- hard requirements for Faux -->
        <script src="./javascripts/vendor/jquery.1.4.2.js"></script>
        <script src="./javascripts/vendor/documentcloud/underscore.js"></script>
        <script src="./javascripts/vendor/documentcloud/backbone.js"></script>
        <script src="./javascripts/vendor/haml-js.js"></script>
        <!-- fixes for using haml-js with IE -->
        <script src="./javascripts/vendor/ie.js"></script>
        <!-- Faux -->
        <script src="./javascripts/vendor/faux.js"></script>
        <!-- other libraries we happen to like -->
        <script src="./javascripts/vendor/seedrandom.js"></script>
        <script src="./javascripts/vendor/functional/to-function.js"></script>
        <script src="./javascripts/vendor/jquery.combinators.js"></script>
        <script src="./javascripts/vendor/jquery.predicates.js"></script>
        <!--misadventure's javascript -->
        <script src="./javascripts/models.js"></script>
        <script src="./javascripts/views.js"></script>
        <script src="./javascripts/controller.js"></script>
      </head>
  
      <body>
    
        <div id="container">
          <h1>Misadventure</h1>
          <div class="content"></div>
  
        </div>    
    
        <small id="get-code">Read about Misadventure and browse the code on <a href="http://github.com/unspace/misadventure">Github</a>.</small>
  
      </body>

      <!-- THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
      IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
      FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
      AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
      LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
      OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
      THE SOFTWARE.

      To the extent possible under law, Unspace Interactive has waived all copyright 
      and related or neighboring rights to the software, except for those portions
      that are otherwise licensed.

      This work is published from Canada. -->

    </html>
    
A few points of interest:

1. We include jQuery, Underscore, Backbone.js, amd haml-js before we include Faux;
2. Misadventure's own files are [models.js][mjs], [views.js][vjs], and [controller.js][cjs];
3. There's a very interesting DOM element, `<div class="content"></div>`.

Looking in the [haml][haml] directory, we see three [Haml][haml-lang] templates: `wake.haml`, `location.haml`, and `bed.haml`. We're going to see where those are used shortly.

controller.js
---

Let's look at [controller.js][cjs]. It's the last file to be loaded, and it starts the application for us. Eliding the comments, we have:

    ;(function () {

    var controller = new Faux.Controller({
      save_location: true,
      model_clazz: true,
      element_selector: '.content',
      partial: 'haml',
      partial_suffix: '.haml',
      title: 'Misadventure'
    });

    controller

      .begin({
        'seed=': {
          locations: function (locations) { return locations.seed; },
          '': function () { return Math.random().toString().substring(2); }
        },
        'locations=': {
          seed: function (seed) { return LocationCollection.find_or_create({ seed: seed }); }
        }
      })

        .method('wake')
  
        .method('bed', {
          route: ':seed/bed'
        })

        .method('location', {
          route: ':seed/:location_id'
        })
    
        .end();


    $(function() {
      Backbone.history.start();
      window.location.hash || controller.wake();
    });
	
    })();

The first statement creates a new instance of `Faux.Controller`. Faux controllers extend Backbone.js's [controllers][bc]. When the program is running, they parse fragments and manage the history so that invoking an URL through a link results in calling a controller method.

Faux the library is nothing more than a backbone controller that has a bunch of helpers for writing Backbone.js controller methods for us. The most important such helper is the `.method` method. Looking at `controller.js`, you can se that we call `.method` three times:

    .method('wake')

    .method('bed', { ...configuration... })

    .method('location', { ...configuration... })
    
We now have enough information to explain Misadventure's basic structure:

1. There are three controller methods invoked during gameplay: `.wake(...)`, `.location(...)`, and `.bed(...)`
2. Each of those controller methods has a route, and Backbone.js will invoke the controller method when the browser invokes the URL.
3. The controller methods that Faux writes will use the templates `wake.haml`, `location.haml`, and `bed.haml` to display content in the page.
4. The content will be injected into the `<div class="content"></div>` DOM element on the page.

We haven't explained anything else about views, models, parsing parameters out of URL, or even defining which URLs invoke which methods yet. All of these things are driven by convention and configuration (preferring convention over configuration, of course).

****what is a controller method?**

If you're comfortable with a web framework like Ruby on Rails, you already know what a controller method is: A controller method is the application code that services a request or performs an action. All of the controller methods defined in Misadventure display something in the page using a template.

These three controller methods are all associated with routes. Controller methods can also be invoked directly in Javascript. For example, invoking `controller.location({ seed: '42492610216140747', id: '7624672284554068' })` has exactly teh same effect as directing your browser to [#/42492610216140747/7624672284554068](http://unspace.github.com/misadventure/#/42492610216140747/7624672284554068). Faux even updates the browser's location bar so that bookmarking and navigation works correctly.

If you dive deeply into Faux, you'll also discover that controller methods can be associated with events or with DOM elements, such that inserting a DOM element with a specific `id` and/or classes automatically invokes a controller method to populate it with children. Misadventure doesn't use any of these techniques. They aren't "advanced" or "complicated," but Misadventure is a small app and doesn't need to use every tool in the toolbox.

Controller methods are part of Backbone.js. Faux builds on Backbone by providing a DSL for writing controller methods, and it works with Backbone's route support. But underlying it all is a perfectly standard MVC architecture.

Now let's look at how each method is configured.

**in the beginning**

Faux methods are configured with objects, usually object literals. To facilitate sharing configuration between methods, Faux provides a lexical scoping mechanism, `.begin({...})` and `.end()`. `.begin({...})` introduces configuration that applies to all of the `.method(...)` calls until `.end()` is called. You can nest calls to `.begin({...})`,and we see that in controllers.js.

Faux methods are also configured by default according to certain naming conventions. We will discuss some more later, but for now the most important one is that unless otherwise specified, the name of the method is part of its route.

The code above is equivalent to:

    controller

      .method('wake', {
        route: '/wake',            // <- by convention, from the name
        partial: 'haml/wake.haml', // <- by convention, from the name
        model_clazz: false,        // <- by convention, from the name
        clazz: false,              // <- by convention, from the name
        'seed=': {                 // <- 'inherited' from .begin(...)
          locations: function (locations) { return locations.seed; },
          '': function () { return Math.random().toString().substring(2); }
        },
        'locations=': {
          '': function () { return LocationCollection.find_or_create(); },
          seed: function (seed) {  // <- 'inherited' from .begin(...)
            return LocationCollection.find_or_create({ seed: seed }); 
          }
        }
      })

      .method('bed', {
        route: '/:seed/bed',
        partial: 'haml/bed.haml',  // <- by convention, from the name
        model_clazz: false,        // <- by convention, from the name
        clazz: BedView,            // <- by convention, from the name
        'seed=': {                 // <- 'inherited' from .begin(...)
          locations: function (locations) { return locations.seed; },
          '': function () { return Math.random().toString().substring(2); }
        },
        'locations=': {            // <- 'inherited' from .begin(...)
          seed: function (seed) { return LocationCollection.find_or_create({ seed: seed }); }
        }
      })

      .method('location', {
        route: '/:seed/:location_id'
        partial: 'haml/location.haml', // <- by convention, from the name
        model_clazz: Location,         // <- by convention, from the name
        clazz: LocationView,           // <- by convention, from the name
        'seed=': {                     // <- 'inherited' from .begin(...)
          locations: function (locations) { return locations.seed; },
          '': function () { return Math.random().toString().substring(2); }
        },
        'locations=': {                // <- 'inherited' from .begin(...)
          seed: function (seed) { return LocationCollection.find_or_create({ seed: seed }); },
          location: function (location) { return location.collection; } // <- by convention, from the name
        },
        'location_id': {               // <- by convention, from the name
          location: function (location) { return location.id; }
        },
        'location=': {                 // <- by convention, from the name
          'locations location_id': function (locations, location_id) { return locations.get(location_id); }
        }
      })

Now that we know the 'expanded' configuration for each method, we can see what they do.

**routes and parameters**

Each of the methods we're defining will be bound to a route. Faux binds controller methods to routes unless you explicitly write `route: false` in a method's configuration. In two of the methods we've explicitly specified the route. In the third, we've let Faux infer the route from the method name.

Many routes have parameters. Faux and Backbone.js both use the same convention, namely anything that looks like a variable name prefixed by `:` is a parameter. So when the browser invokes [#/42492610216140747/bed][bed], the controller matches this against the route `/:seed/bed` and it's a match. Faux then invokes our controller method `controller.bed({ seed: '42492610216140747' })`.

*Nota Bene: Faux and Backbone differ slightly in how parameters are passed to controller methods. In plain vanilla Backbone,* `#/42492610216140747/bed` *would invoke* `controller.bed('42492610216140747')`*, whereas Faux bundles parameters into a hash of names and values.*

So now we know that there are three controller methods, that each has a route, and that when the route is invoked, parameters are parsed out and provided to the controller method in a hash. We haven't discussed all the `'seed='` configurations, we'll get to that after we explain how the application launches.

**launching the application**

As you know from jQuery, this code:

    $(function() {
      Backbone.history.start();
      window.location.hash || controller.wake();
    });
    
...is run when the page has loaded. `Backbone.history.start();` initializes Backbone's support for managing URLs. The next line checks to see whether the current URL has a fragment. If it doesn't, it invokes the controller method `.wake()` to start the application in a new cornfield.

So now we know how the application is launched. Let's look at our controller methods in more detail.

controller.wake()
---

As you saw above, `controller.wake()` has this "extended" configuration:

    .method('wake', {
      route: '/wake',            // <- by convention, from the name
      partial: 'haml/wake.haml', // <- by convention, from the name
      model_clazz: false,        // <- by convention, from the name
      clazz: false,              // <- by convention, from the name
      'seed=': {                 // <- 'inherited' from .begin(...)
        locations: function (locations) { return locations.seed; },
        '': function () { return Math.random().toString().substring(2); }
      },
      'locations=': {
        '': function () { return LocationCollection.find_or_create(); },
        seed: function (seed) {  // <- 'inherited' from .begin(...)
          return LocationCollection.find_or_create({ seed: seed }); 
        }
      }
    })
        
We've discussed that because its route is configured to be `/wake`, `controller.wake()` is invoked by a fragment of `#wake`. It can also be invoked directly, as we saw when the page is first loaded. It has no parameters.

So what happens after it is invoked? This is where the additional configuration comes into play:

      'seed=': {
        locations: function (locations) { return locations.seed; },
        '': function () { return Math.random().toString().substring(2); }
      },
      
And:

      'locations=': {
        seed: function (seed) {
          return LocationCollection.find_or_create({ seed: seed }); 
        }
      }

These options name two parameters, `seed` and `locations`. They also describe how might calculate either one if it isn't provided. What they say is:

1. To calculate `seed`, if you have `locations`, return `locations.seed`
2. To calculate `seed`, if nothing else works, return `Math.random().toString().substring(2)`
2. To calculate `locations`, if you have `seed`, return `LocationCollection.find_or_create({ seed: seed })`

You can see that the convention is to provide a hash of variable(s) provided to functions that do the calculating. The special case is that if you provide an empty string as a key, it becomes the "default" calculation.

In our case, we aren't providing any parameters, so Faux can't calculate `seed` from `locations`, and it can't use `seed` to calculate `locations` (since it doesn't have seed). Since it doesn't have any other calculation that works, Faux will use the "default" calculation for `seed` of `Math.random().toString().substring(2)` `LocationCollection.find_or_create()`.

Let's say this produces `'19608841026141122'`. So our parameters went from `{}` (no parameters) to `{ seed: '19608841026141122' }`. What about `locations`? Well, now that we have`seed`, Faux can calculation `locations` using `LocationCollection.find_or_create({ seed: seed })`. So Faux now has parameters of `{ seed: '19608841026141122', locations: ... }`.

wake.haml
---

"ANd?" you may ask. Well, Faux knows this method is called `Wake`. Faux has already looked for a `Backbone.View` of `WakeView` Looking in `views.js`, we see that there is a `BedView` and a `LocationView`, but no `WakeView`. Likewise, there is no `Wake` or `WakeModel` defined. Therefore, Faux skips all other Backbone architecture and displays the parameters it has in the `wake.haml` template:

    %p.intro You have been abducted by aliens!

    %img{ src: './images/cornfield.gif' }

    %p.caption You wake up in a cornfield.

    %ol
      %li
        %a.stand_north{ href: route_to_location({ location: locations.centre }) } Stand up
         and look around.

      %li.reset
        Or you can 
        %a.close_eyes{ href: route_to_wake() } close your eyes
         and go back to sleep, maybe it will all go away.

The two things of interest in this template are `href: route_to_location({ location: locations.centre })` and `href: route_to_wake()`. Each of the controller methods we defined has a corresponding `route_to` helper method that is available locally in templates. So (obviously) the `route_to_location` helper returns the route that invokes `controller.location(...)` and the `route_to_wake` helper returns the route that invokes `controller.wake()`.

Let's look at `route_to_location({ location: locations.centre })`. We're passing in a parameter named `location`. We'll get to that in a moment, but the value is interesting: We take our `locations` parameter and get the `centre` property. (That happens to be the centre of the corn maze, but we'll cover locations shortly.)

So what happens when we call `route_to_location({ location: locations.centre })`? Let's take a sneak peek at our definition for `controller.location`:

      .method('location', {
        route: '/:seed/:location_id'
        partial: 'haml/location.haml', // <- by convention, from the name
        model_clazz: Location,         // <- by convention, from the name
        clazz: LocationView,           // <- by convention, from the name
        'seed=': {                     // <- 'inherited' from .begin(...)
          locations: function (locations) { return locations.seed; },
          '': function () { return Math.random().toString().substring(2); }
        },
        'locations=': {                // <- 'inherited' from .begin(...)
          seed: function (seed) { return LocationCollection.find_or_create({ seed: seed }); },
          location: function (location) { return location.collection; } // <- by convention, from the name
        },
        'location_id': {               // <- by convention, from the name
          location: function (location) { return location.id; }
        },
        'location=': {                 // <- by convention, from the name
          'locations location_id': function (locations, location_id) { return locations.get(location_id); }
        }
      })
    
Faux wants to generate a route. So it needs a `seed` and a `location_id`. We've defined how to make a `seed` out of `locations`. And Faux has inferred how to make `locations` out of a `location` from the name. So Faux figures the seed out from the location your supply. Faux also needs a `location_id`, and once again Faux has inferred the correct function from the names, and it can fill in the values.

(If you don't like to use such obvious naming conventions, you are free to define your own conversion functions, just as we did for `seed` and `locations`).

So having provided the centre `location`, Faux is able to calculate a route of `http://unspace.github.com/misadventure/#/7221645157845498/6682013437305772` using `route_to_location`.  With `route_to_wake`, no calculations are needed because the route, `/wake`, doesn't have any parameters.

This, incidentally, is the whole point of writing the separate calculations as part of the configuration instead of as a function. Faux can mix that in with conversions it infers by convention and thus support `route_to` helpers.

Consider the alternative. If we didn't have separate calculations, you would have to write `route_to_location({ seed: location.collection.seed, location_id: location.id })`. That's better than `'#/' + location.collection.seed + '/' + location.id`, but not much. Now all this code needs to know is that the route to a location requires a location. The specifics of how that is translated to the route is hidden. Perhaps some future refactoring might build enough information into the location's id that no seed is necessary.

[bc]: http://documentcloud.github.com/backbone/#Controller
[haml-lang]: http://haml-lang.com/
[a]: http://www.digitalhumanities.org/dhq/vol/001/2/000009/000009.html
[f]: https://github.com/unspace/faux
[play]: http://unspace.github.com/misadventure/
[r]: http://weblog.jamisbuck.org/2011/1/12/maze-generation-recursive-division-algorithm
[j]: http://weblog.jamisbuck.org/
[rb]: http://reginald.braythwayt.com
[source]: http://github.com/unspace/misadventure
[docco]: https://github.com/raganwald/homoiconic/blob/master/2010/11/docco.md "A new way to think about programs"
[cjs]: http://unspace.github.com/misadventure/docs/controller.html
[mjs]: http://unspace.github.com/misadventure/docs/models.html
[vjs]: http://unspace.github.com/misadventure/docs/views.html
[s]: http://yayinternets.com/
[ui]: http://unspace.ca
[b]: http://documentcloud.github.com/backbone/
[wake]: http://unspace.github.com/misadventure/#/wake
[l1]: http://unspace.github.com/misadventure/#/42492610216140747/7624672284554068
[l2]: http://unspace.github.com/misadventure/#/42492610216140747/5682321739861935
[l3]: http://unspace.github.com/misadventure/#/42492610216140747/3916709493533819
[bed]: http://unspace.github.com/misadventure/#/42492610216140747/bed