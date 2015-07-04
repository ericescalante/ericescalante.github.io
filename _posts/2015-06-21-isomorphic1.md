---
layout: post
title: Isomorphic React apps in a PHP CMS - 1
comments: true
---

## Part 1 - React components

In this section we'll cover the react components, please make sure you have checked out the <a href="https://github.com/ericescalante/isomorphic-post-code" class='repo' target="_blank">sample repo</a> so you can run the code as you read.

The application will implement three react components: a developer list, a mini profile and a pager. 

### MiniProfile
<div class="profiles">
  <image src="/public/miniprofile.png" style="width:100px;"/>
</div> 

{% highlight html linenos %}
var React = require('react');

var MiniProfile = React.createClass({
  displayName: 'MiniProfile',
  render: function() {
    var user = this.props.user;
    var css = {width:120, padding:10};
    return (
      <div className="pull-left">
        <img src={user.avatar_url} style={css} className="img-circle pull-left"/>
        <div className="text-center">
          <a href={user.html_url} target="_blank">
            <span className="label label-info">{user.login}</span>
          </a>
        </div>
      </div>
      );
    }
});

module.exports = MiniProfile;
{% endhighlight %}

*Explanation*: This component has only one dependency, so we declare a React instance on line 1. Using the react instance we proceed to create our component with `React.createClass` and give it the name MiniProfile. The `displayName` property allows the name of our component to be shown on the React plugin for chrome :)
Next, our component implements the `render` method, with the user received as a property via `this.props.user` and lovely JSX. 
Finally, we export our MiniProfile so it's available for inclusion on other scripts, as we'll see on the developer list.

### Pagination
<div class="profiles">
  <image src="/public/pagination.png" style="width:100px;"/>
</div> 

{% highlight html linenos %}
var React = require('react');
var cx = require('classnames');
var Link = require('react-router').Link;

var Pagination = React.createClass({
  displayName: 'Pagination',
  render: function() {
    var linkCss = {marginTop: 20};
    var buttonCss = {margin: 5};
    var prevPage = {page: this.props.currentPage - 1};
    var nextPage = {page: this.props.currentPage + 1};
    return (
      <div className="row clearfix text-center" style={linkCss}>
        <Link to="index" params={prevPage} style={buttonCss}>
          <button className={prevClasses}>
          </button>
        </Link>
        <Link to="index" params={nextPage} style={buttonCss}>
          <button className="glyphicon glyphicon-chevron-right btn btn-primary">
          </button>
        </Link>
      </div>
    );
  }
});
 
{% endhighlight %}

*Explanation* This component has two additional dependencies beside just React. On line 2 we create an instance of [classnames](https://www.npmjs.com/package/classnames) used to add css classnames based on a condition (not showing the left chevron on the first page in this case). The second dependency is for [Link](http://rackt.github.io/react-router/#Link) a component that comes with [react-router](https://github.com/rackt/react-router). Link will help us creating links that allow updating the browser's url while navigating through our developer list via ajax and also work as traditional anchor tags when javascript is not available, as in the case of a search robot or a clunky browser. The JSX bit just adds a 'previous' and 'next' chevrons so we can move back and forth in the list.

### Developer list
<div class="profiles">
  <image src="/public/developerlist.png" style="width:600px;"/>
</div> 

{% highlight html linenos %}
var React = require('react');
var request = require('request');
var MiniProfile = require('./miniprofile');
var Pagination = require('./pagination');

var DeveloperList = React.createClass({
  displayName: 'DeveloperList',
  statics: {
    fetchData: function(params, callback) {
      'Fetching from Github';
      var options = {
        url: "https://api.github.com/users?since=" + (params.page * 30),
        headers: {
          'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0'
        },
        withCredentials: false
      };
      return request(options, function(error, response, body) {
        callback(null, JSON.parse(body));
      });
    }
  },
  render: function() {
    var profiles = []
    this.props.data.forEach(function(user, index) {
      profiles.push(<MiniProfile key={index} user={user}/>);
    });
      
    return (
      <div>
        <div className="row">
          {profiles}
        </div>
        <Pagination currentPage={parseInt(this.props.params.page)}/>
      </div>
    );
  }
});

module.exports = DeveloperList;
{% endhighlight %}

*Explanation*: It all comes together on our developer list component. Lines 1 - 4 define the dependencies for this component, starting with React. Then we create a [request](https://www.npmjs.com/package/request) instance, which we'll use later on to make our call to Github's API. The last to lines create instances of our previously defined MiniProfile and Pagination components.
Of note here, is that we have created our data fetching function as a `static`, in order to be able to call it later on from React Router. The data resulting from this function will be passed by the router as a property of our developer list, so we can make use of it in line 26 in a `forEach` as a source to build individual MiniProfiles for each developer in our `render` function. To finish things off, we include our Pagination component and pass the current page as a property.


That's it for the first part, on the next post we'll go over the process hooking our developer list component to a router and making it work both on the server and the browser!
