---
layout: post
title: Routing in Polymer Elements
---

Lately, I've been working a lot with [Polymer](https://www.polymer-project.org/), a fantastic library for making web components a lot easier to produce and manage.

It really is a great library, which I see more as a wrapper than a framework. It provides an excellent binding and change notification system and allows for truly reusable components to be made.

However, the one thing which always seemed out of place to me, was the use of [page.js](https://visionmedia.github.io/page.js/) in the [Polymer Starter Kit](https://github.com/PolymerElements/polymer-starter-kit).

To remove this out-of-place dependency, [carbon-route](https://elements.polymer-project.org/elements/carbon-route) has been introduced.

`carbon-route` allows you to deal with your routes the same as any other logic in your application, through Polymer elements.

## An example

An example is the best way to show what this does, so here we go:

```html
<carbon-location route="{% raw %}{{route}}{% endraw %}"></carbon-location>

<carbon-route
	route="{% raw %}{{route}}{% endraw %}"
	pattern="/:page"
	data="{% raw %}{{routeData}}{% endraw %}"
	tail="{% raw %}{{subRoute}}{% endraw %}">
</carbon-route>
```

Take straight from the docs, this simply binds in `window.location` through the `carbon-location` element and produces a route object for us.

The route object essentially holds the state, such as the current path, query string, parameters and so on (now bound to `route` in the example).

The first two parameters are obvious, `route` is the current route state and `routeData` holds our parameters (`:page`).

The cool part here is `subRoute`. 

## Sub routing

In `page.js`, we could only really specify entire routes and what page (e.g. in an `iron-pages`) is associated with each.

With `carbon-route`, we now have the ability to match a route and pass the remainder down to be routed/handled further by other elements.

In `index.html`:

```html
<carbon-route
	route="{% raw %}{{route}}{% endraw %}"
	pattern="/:page"
	data="{% raw %}{{routeData}}{% endraw %}"
	tail="{% raw %}{{subRoute}}{% endraw %}">
</carbon-route>

<iron-pages
	attr-for-selected="data-route"
	selected="{% raw %}{{routeData.page}}{% endraw %}">

	<my-page
		data-route="test"
		route="{% raw %}{{subRoute}}{% endraw %}">
	</my-page>

</iron-pages>
```

Here, if we visit `/test` then `routeData.page` will be `test`. If we visit `/test/123`, it will still be `test`. We are matching only the first part of the path essentially, any unmatched goes in our `subRoute` state (as in, `subRoute.path` will be `/123`).

This means we can leave all paths underneath `/test` upto `my-page` to handle, excellent!

In `my-page.html`:

```html
<carbon-route
	route="{% raw %}{{route}}{% endraw %}"
	pattern="/:id"
	data="{% raw %}{{routeData}}{% endraw %}">
</carbon-route>
```

So, as you see, `my-page` has no idea about `/test`, it only cares about paths visited below that (in this case, `/123`). Here, `routeData.id` will become `123` for `/test/123`.

## Everything is observed

Now, the really cool thing I love about `carbon-route` and `carbon-location` is the binding.

The route object that `carbon-location` produces is bound into `window.location`, so we can deal with all location changes within Polymer:

```javascript
this.set('route.path', '/foo'); // Causes a location change to /foo
```

Remember too, that this all uses the HTML5 history API so there are no actual changes in page.

Even better, the route data is also bound:

```javascript
this.set('routeData.id', 500); // Causes a location change to /test/500
```

## Another example

This leads to some really cool stuff, like the following:

```html
<paper-tabs
	selected="{% raw %}{{routeData.page}}{% endraw %}"
	attr-for-selected="data-route">
	
	<paper-tab data-route="home">Home</paper-tab>
	<paper-tab data-route="contact">Contact</paper-tab>
	<paper-tab data-route="about">About</paper-tab>

</paper-tabs>
```

This very nicely results in the location bar changing to the associated route when a tab becomes active. So if we click `Home`, we change location to `/home` because of the binding `routeData.page` has.

## To finish

As you can see, this is such a nice addition as we can deal with routes within our Polymer app, keeping with the flow. No longer do we have to resort to external means to handle it, but can put it where it belongs, with the rest of our code.
