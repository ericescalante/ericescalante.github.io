---
layout: post
title: Isomorphic React apps in PHP via dnode - 3
tags: [react, node, php]
comments: true
---

## Part 3 - Server and client-side rendering

Now the plot begins to thicken. It's time to work a little bit on the backend, behind the courtains. 

### Node.js server

{% highlight javascript linenos %}
var React = require('react');
var Router = require('react-router');
var routes = require('./routes');
var dnode = require('dnode');
var express = require('express');
var app = express();
var gutil = require('gulp-util');

regularPort = 3000;
dnodePort = 3001;

app.get('/developers/page/*', function(req, res) {
  var router = Router.create({
    location: req.url,
    routes: routes
  });
  return router.run(function(Handler, state) {
    gutil.log("Serving to browser via " + regularPort);
    var component = state.routes[1].handler;
    return component.fetchData(state.params, function(err, data) {
      var reactOutput = React.renderToString(React.createElement(Handler, {
        data: data,
        params: state.params
      }));
      return res.send(getHtml(reactOutput, data, state));
    });
  });
});

app.use(express["static"](__dirname + '/public'));

server = app.listen(regularPort, function() {
  return gutil.log("HTTP: Listening on port " + regularPort);
});

getHtml = function(reactOutput, data, state) {
  var response;
  response = '<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">';
  response += "<script>window.reactData = " + (JSON.stringify(data)) + "</script>";
  response += "<script>window.initialPage = " + state.params.page + "</script>";
  response += '<div id="app" class="container">';
  response += reactOutput;
  response += '</div>';
  return response += "<script src='http://localhost:" + regularPort + "/js/bundle.js'></script>";
};

{% endhighlight %}

*Feeble attempt at an explanation*: Ok, let's take this one step at a time. We start first by including our routes definition and setting  up a simple Express app. Finally we declare two variables that will tell node and dnode on which ports they should listen for connections.

On our Express app instance, we will define a function to handle GET requests to `/developers/page/*`. This function returns a react-router instance created using the current `req.url` and our routes configuration (line 12). Next we invoke the `run` function on our router instance and obtain the react component that will handle the route  and the current route state (params).

On line 22 we invoke the static function we coded into our DeveloperList component (`FetchData`, remember?) and pass a callback function to act upon the resulting data from the request to Github's API.

On line 24 we invoke React's `renderToString` method passing our DeveloperList component (represented by the `Handler` variable), the resulting data from `fetchData`, and finally the `state.params` provided by react router (page: 1 in this case). The resulting HTML we pass to a custom function called `getHtml`  that will set the data as JSON object in the page so react can use it the first time. This function also includes bootstrap and more importantly, the `bundle.js` file generated by browserify. All this unwholesome HTML concatenatin' can be replaced with a jade template for example, but just wanted to keep it all server stuff on a single file for this post :)

We wrap things up on line 28 by sending the output of our `getHtml` function as the response from our express app back to the browser.

### Browser logic

We  now have a working server and router to handle the initial request of a user. But we also need to configure our router to handle requests done once the page has been served, and do this without bothering the `server` script. We need now a `browser` script!


{% highlight javascript linenos %}
var React = require('react');
var Router = require('react-router');
var routes = require('./routes');

Router.run(routes, Router.HistoryLocation, function(Handler, state) {
  if (window.initialPage === parseInt(state.params.page)) {
    return React.render(
      <Handler data={window.reactData} params={state.params}/>, document.getElementById('app')
    );
  } else {
    var component = state.routes[1].handler;
    return component.fetchData(state.params, function(err, data) {
      React.render(
      <Handler data={data} params={state.params}/>, document.getElementById('app')
      );
    });
  }
});

{% endhighlight %}

*Explanation*: Thanks Cthulhu and all the Great Old Ones, things get more simple from now on. Ish. The first three lines should look familiar by now, so we can move along. On line 5 we create once again a router instance but this time we pass the browser url provided by `Router.HistoryLocation` together with our routes definition.

We need to tell react to render the resulting Handler (our trusty developers list) using either the JSON data that comes already hardcoded on the page thanks to node, or to do a new call to Github's API, in order to avoid doing an unnecessary request the first time. That little if on line 6 makes sure of just that. So, when it's the first render of the page, we just invoke `React.render` with the data present, otherwise we make a call to the `fetchData` function of our Handler and use the result of that as the data to be rendered. I still feel there must be a better way to handle this scenario. Looking forward to improving on that!

That's it! We have a this point an isomorphic/universal/$nextTrendyName node app, that makes use of the same react components on both the server and the client! It's Sunday and it's sunny outside, so time to head out and walk a bit in the sun. But first, let's hook all of this up to PHP. 

Because you only live once.
