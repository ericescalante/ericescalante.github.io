---
layout: post
title: Isomorphic React apps in a PHP CMS
---

## Part 1 - React components

In this section we'll cover the react components, please make sure you have checked out the [sample repo](https://github.com/ericescalante/isomorphic-post-code) so you can run the code as you read.

The application will implement three react components: a developer list, a mini profile and a pager. We're gonna use [cjsx](https://github.com/jsdf/coffee-react) in order to be able to code our components using the [coffeescript](http://coffeescript.org/) version of react's [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html). This allows us to write shorter, more concise code and not have to worry about closing parenthesis and semicolons. 

### MiniProfile
<div class="profiles">
  <image src="/public/miniprofile.png" style="width:100px;"/>
</div> 
{% highlight coffeescript linenos %}
React = require 'react'

MiniProfile = React.createClass
  displayName: 'MiniProfile'
  render: ->
    user = this.props.user
    <div className="pull-left">
      <img src={user.avatar_url} style={width:120, padding:10} className="img-circle pull-left"/>
      <div className="text-center">
        <a href={user.html_url} target="_blank"><span className="label label-info">{user.login}</span></a>
      </div>
    </div>

module.exports = MiniProfile
{% endhighlight %}

*Explanation*: This component has only one dependency, so we declare a React instance on line 1. Using the react instance we proceed to create our component with `React.createClass` and give it the name MiniProfile. The `displayName` property allows the name of our component to be shown on the React plugin for chrome :)
Next, our component implements the `render` method, with the user received as a property via `this.props.user` and lovely CJSX. 
Finally, we export our MiniProfie so it's available for inclusion on other scripts, as we'll see on the developer list.

### Pagination
<div class="profiles">
  <image src="/public/pagination.png" style="width:100px;"/>
</div> 
{% highlight coffeescript linenos %}
React = require 'react'
cx = require 'classnames'
Link = require('react-router').Link

Pagination = React.createClass
  displayName: 'Pagination'
  render: ->
    prevClasses = cx
      'glyphicon': true
      'glyphicon-chevron-left': true
      'btn btn-primary': true
      'hidden': this.props.currentPage == 1
    <div className="row clearfix text-center" style={marginTop: 20}>
      <Link to="index" params={page: this.props.currentPage - 1} style={margin: 5}>
        <button className={prevClasses}>
        </button>
      </Link>
      <Link to="index" params={page: this.props.currentPage + 1} style={margin: 5}>
        <button className="glyphicon glyphicon-chevron-right btn btn-primary">
        </button>
      </Link>
    </div>

module.exports = Pagination
{% endhighlight %}

*Explanation* This component has two additional dependencies beside just React. On line two we create an instance of [classnames](https://www.npmjs.com/package/classnames) used to add css classnames based on a condition (not showing the left chevron on the first page in this case). The second dependency is for [Link](http://rackt.github.io/react-router/#Link) a component that comes with [react-router](https://github.com/rackt/react-router). Link will help us creating links that allow updating the browser's url while navigating through our developer list via ajax and also work as traditional anchor tags when javascript is not available, as in the case of a search robot or a clunky browser. The CJSX bit just adds a 'previous' and 'next' chevrons so we can move back and forth in the list.

Note: the `params` parameter on lines 14 and 18 should have double curly brackets but [jekyll](http://jekyllrb.com/) says no. 

### Developer list
<div class="profiles">
  <image src="/public/developerlist.png" style="width:600px;"/>
</div> 
{% highlight coffeescript linenos %}
React = require 'react'
request = require 'request'
MiniProfile = require './miniprofile.cjsx'
Pagination = require './pagination.cjsx'

DeveloperList = React.createClass
  displayName: 'DeveloperList'
  statics:
    fetchData: (params, callback) ->
      'Fetching from Github'
      options =
        url: "https://api.github.com/users?since=#{params.page*30}"
        headers:
          'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0'
        withCredentials:false
      request options, (error, response, body) ->
        return callback null, false if response.statusCode != 200
        callback null, JSON.parse body
  render: ->
    profiles = []
    this.props.data.forEach (user, index) ->
      profiles.push <MiniProfile key={index} user={user}/>

    <div>
      <div className="row">
        {profiles}
      </div>
      <Pagination currentPage={parseInt this.props.params.page}/>
    </div>

module.exports = DeveloperList
{% endhighlight %}

*Explanation*: It all comes together on our developer list component. Lines 1 - 4 define the dependencies for this component, starting with React. Then we create a [request](https://www.npmjs.com/package/request) instance, which we'll use later on to make our call to Github's API. The last to lines create instances of our previously defined MiniProfile and Pagination components.
Of note here, is that we have created our data fetching function as a `static`, in order to be able to call it later on from React Router. The data resulting from this function will be passed by the router as a property of our developer list, so we can make use of it in line 21 in a `forEach` as a source to build individual MiniProfiles for each developer in our `render` function. To finish things off, we include our Pagination component and pass the current page as a property.


That's it for the first part, on the next post we'll describe the process hooking our developer list component to a router and making it work both on the server and the browser!
