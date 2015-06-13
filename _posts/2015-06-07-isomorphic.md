---
layout: post
title: Isomorphic React apps in a PHP CMS
---
# Isomorphic React apps in a PHP CMS

## Introduction

Hi all, welcome to the first post of my first blog! Well, the second one if you count witty and eloquent [lorem impsum one](/2015/05/25/welcome.html). This post assumes some basic familiarity with [React](https://facebook.github.io/react/), [Node.js](https://nodejs.org/) and [php](http://php.net/). Finally, all javascript code snippets will be written using [Coffeescript](http://coffeescript.org/) because of it's terseness and simplicity.

The goal of this post is to document and share my learnings on how to create an isomporphic React application that can be integrated within an existing CMS (be it [Drupal](https://www.drupal.org/), [Wordpress](https://wordpress.com/), etc). When creating single-page applications, we want to aim for ismorphism whenever possible for this reasons:

- SEO: Search engines (why bother with the plural, we all know there's just one anyway) should be able crawl and index our content. 
- Performance: A prendered page can be cached server-side and provide a snappy user experience. React hooks itself up once the page is served and takes over the rendering of it's components.
- Compatibility: A browser with no javascript enabled can still display the page and allow for a simple navigation.
- Maintainability: Sharing code between the server and browser, allows for a smaller codebase, which is easier to keep healthy.

For these reasons, just adding React components to the existing user interface of our CMS would prevent us from reaping the full benefits, even though it would still provide our application with the reduced development times, maintainability and faster response time that React brings to the table.

## Our sample scenario

In order to keep our experiment as simple as posible, let's skip the backend creation and use and existing service, in this case Github. Our little app should provide  our PHP CMS with a paginated list of Github accounts :)
