---
layout: post
title: Isomorphic React apps in a PHP CMS - 4
comments: true 
---

## Part 4 - Rendering from PHP with dnode

Or Ruby, or Java. Heck, even perl! Our proud little app can integrate with any language that has a dnode implementation! 

When installing the php dnode package with composer, I was surprised to find installed a [React](http://reactphp.org/) dependency that actually has nothing to do with facebook's! It seems to be a great library for evented i/o for php.

Also, when I first attempted this experiment, I tried using [Facebook's php library based on v8](https://github.com/reactjs/react-php-v8js) or straight [php v8](http://php.net/manual/en/book.v8js.php) but it soon became obvious that another solution was needed.

### In our node server
{% highlight javascript linenos %}
dserver = dnode({
  renderIndex: function(remoteParams, remoteCallback) {
    var router = Router.create({
      location: '/developers/page/' + remoteParams.page,
      routes: routes
    });
    return router.run(function(Handler, state) {
      gutil.log("Serving to PHP via " + dnodePort);
      var component = state.routes[1].handler;
      return component.fetchData(state.params, function(err, data) {
        var reactOutput = React.renderToString(React.createElement(Handler, {
          data: data,
          params: state.params
        }));
        return remoteCallback(getHtml(reactOutput, data, state));
      });
    });
  }
});

dserver.listen(dnodePort);

gutil.log("dnode: Listening on port " + dnodePort);
{% endhighlight %}

*Explanation*: These lines come after the declaration of our `getHtml` function in our `server` script (around line 48 over there). To begin things, we declare the name of the function we want to expose via dnode, in this case `renderIndex`. When the php side of dnode calls, it will provide the parameters and a remote function to be invoked as a callback once  we're done. 

Inside our `renderIndex` function, on line 2, we'll perform the now familiar task of getting a route handler for `/developers/page/` (appending the page number sent by php). Then we go ahead and invoke the handler's `fetchData` and use the resulting data to have `React.renderToString` provide the final html. Finally on line 18 we invoke the remote callback with the output of our handy `getHtml` function. Bam! 

### PHP index script.

<div class="profiles">
  <image src="https://raw.githubusercontent.com/ericescalante/isomorphic-post-code/master/screenshots/screenshot2.png" style="width:600px;"/>
</div> 

{% highlight html linenos %}
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Sample PHP page rendering Node/ReactJS</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
  </head>

  <body style="background:#FAFAFA;padding:20px">
    <div class="container">
      <div class="jumbotron">
      <h1>Sample PHP page</h1>
      <p>The content below is rendered server-side with PHP/node/react via dnode.</p>
      <p>Subsequent pagination is done with react client-side via Ajax.</p>
      <p>This could be a Drupal, Wordpress, or any other kind of PHP app :)</p>
    </div>

    <div class="panel panel-default" style="padding:20px;">
<?php 
  require 'vendor/autoload.php';

  $path =  explode('/', $_SERVER['REQUEST_URI']);

  $options  = array('page' => array_pop($path));
  $loop = new React\EventLoop\StreamSelectLoop();
  $dnode = new DNode\DNode($loop);
  $dnode->connect(3001, function($remote, $connection) use($options) {
    $remote->renderIndex($options, function($result) use ($connection) {
        echo $result;
        $connection->end();
    });
  });
  $loop->run();

?>
      </div>
    </div>
  </body>
</html>

{% endhighlight %}

*Explanation*: The markup is just a simple html page, but close your eyes for a second and picture this implemented properly on a large, living app. 

The magic starts at line 26, where we tell our dnode php instance to connect to port 3001, the one we defined in our node server script as `dnodePort = 3001`. This gives us a `$remote` object from which we can make calls to functions implemented on the other side of this funny wormhole, back in node. 

So, when we do `$remote->renderIndex` that `renderIndex` is the function we just defined on server.coffee a couple of paragraphs above! When I saw this running the first time my mind was totally blown (away)! - yes, I know I need to go out more. 

Next, we just pass the current page inside an `$options` array and get back a `$result` variable containing the output of the node app, including a link to our `bundle.js` that includes the browser-side react-routing code! 

Our PHP app now serves pre-rendered react components! Once the page is ready, all user interaction is handled by the react components client side, communicating via ajax to the node app. Note that we can send the response from our node app in array so we can have body and head elements separately (like open graph tags, for example).

### We made it!
This ends this short series and I hope it was fun and useful for people learning node, gulp, react, and curious about remote procedure calls, etc. 

Don't forget to check out the <a href="https://github.com/ericescalante/isomorphic-post-code" class='repo' target="_blank">repo</a> and get in touch with suggestions, questions, or complaints about my cat sneaking into your balcony and causing mischief!
