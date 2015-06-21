---
layout: post
title: Isomorphic React apps in a PHP CMS
---

## Introduction

In this four-part series, we're going to cover how to go about making a PHP benefit from the use of ReactJS components both server and client-side. The case for this is, for example, if we have a Wordpress or Drupal CMS we want to keep around for it's content management functions, but want to extend with new and modern functionality.

Being able share some of the javascript code both on the server and client provides many advantages, such as:

- SEO: Search engines should be able crawl and index our content. No need to implement/pay for a pre-rendering service to achieve this.
- Performance: A pre-rendered page can be cached server-side and provide a snappy user experience. React hooks itself up once the page is served and takes over the rendering of it's components based on user interactions.
- Compatibility: A browser with no javascript enabled can still display the page and allow for a simple navigation.
- Usability: A user is be able to bookmark the current state of the page and is able to return to it or share it.
- Maintainability: Sharing code between the server and browser, allows for a smaller code base, which is easier to keep healthy.

For these reasons, just adding React components to the existing user interface of our CMS would prevent us from reaping the full benefits, even though it would still provide our application with the reduced development times, maintainability and faster response time that React brings to the table.

## Tools used

This post assumes some basic familiarity with the following:

* [Node.js](https://nodejs.org/)
* [React](https://facebook.github.io/react/)
* [gulp](http://gulpjs.com/)
* [browserify](http://browserify.org/)
* [coffeescript](http://coffeescript.org/)
* [php](http://php.net/) 
* [composer](https://getcomposer.org/)

  
## Sample application

The goal of our brave little sample app is to connect to Github's API and fetch a list of developers and display their username and avatar. We'll also provide a link to their Github profile page and show a simple back and forth pager at the bottom.

The series will be spread as follows:

* Intro
* [Part 1](/2015/06/21/isomorphic1/): React components
* Part 2: Server-side rendering
* Part 3: Rendering from PHP with dnode

I've created a repository with the finished app so you can download, analyze and tinker with it as you read along. 
Please be gentle, it's my first attempt at a coding post+repo!
You can find it at: [ericescalante/isomorphic-post-code](https://github.com/ericescalante/isomorphic-post-code)


