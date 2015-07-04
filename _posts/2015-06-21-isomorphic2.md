---
layout: post
title: Isomorphic React apps in a PHP CMS - 2
comments: true
---

## Part 2 - Route configuration

Now that we have coded our components, we need to link them up to a route handler, so when a user goes to `http://localhost:3000/developers/page/1` we can catch that page 1 parameter and pass it as a property to our developers list.

### Route configuration 

{% highlight html linenos %}
var React = require('react');
var Route = require('react-router').Route;

module.exports = [
  <Route>
    <Route name='index' path="/developers/page/:page" handler={require('./developer_list.js')} ignoreScrollBehavior/>
  </Route>
];
{% endhighlight %}

*Explanation*: At the top we declare our React dependency and then go ahead and create an instance of react-router. The next step is to export a route declaration, in which we declare that the path `/developers/page/:page` will be handled by the `developers_list.js` component.

 The `ignoreScrollBehavior` is a handy little parameter that prevents the browser from scrolling up when a user clicks on our pagination links. The page paramater will later tickle down via `state.params.page` to the `props` object of our react components.

As a nice bonus of using React Router we have managed to create three *state-less components*. It's considered [a best practice](https://facebook.github.io/react/docs/thinking-in-react.html) to keep as little state as possible in your components, since every change of state triggers a new rendering of the component. As you can see, none of the components implement a `getInitialState` function and only work with the `props` handed down to them from above.

Later on, we'll include this route definition script in our `server` script and `browser` script files in order to process urls coming via http requests and browser locations respectively.
