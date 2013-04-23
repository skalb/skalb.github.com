---
layout: post
title: Using Sinon Spys with Meteor Collections
categories:
- JavaScript
- Meteor
- Mocha
- Sinon
- Testing
status: publish
type: post
published: true
comments: true
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  robotsmeta: index,follow
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _facebookcount-cache: '0'
  _twittercount-cache: '0'
---
Following my testing described <a title="Testing Backbone Routers in Meteor with Mocha" href="http://www.skalb.com/2012/08/19/testing-backbone-routers-in-meteor-with-mocha/">here</a>, I wanted to verify that Meteor interacts with my Routers correctly.

<!--more-->

The Meteor.Collection object exposes methods for querying the MongoDB collection as described in the docs. I want to test if my index callback will retrieve all the items it should.

Add the new module we need:

``` bash
npm install sinon
```

Here's how to use the spy:

``` coffeescript lib/sample_router_factory.coffee
items =
  find: sinon.spy()
```

Now, let's write our test:

``` coffeescript tests/sample_router_factory_test.coffee
it "should retrieve all items", ->
  router.navigate('', true)
  items.find.called.should.equal true
```

This will fail, since we need to first modify our Router to accept a collection as an argument and then call find on it.

``` coffeescript lib/sample_router_factory.coffee
root = exports ? this

class SampleRouterFactory
  constructor: (@Backbone, @Items) ->

  getRouter: () ->
    SampleRouter = @Backbone.Router.extend(
      routes:
        "": "index"

      index: =>
        items = @Items.find()
    )
    new SampleRouter

root.SampleRouterFactory = SampleRouterFactory
```

This <em>should</em> pass, but yet doesn't. Why? Thinking about the assumption our test is making when using the a Backbone router, it likely depends on Backbone initialization. We can try this in our test, also:

``` coffeescript
Backbone.history.start
  silent: true
  pushState: true
```

This doesn't work either because Backbone assumes we are in the browser when accessing **window**. I'm pretty sure this is not the approach I want. We don't need to add a dependency on Backbone's internal behavior to test a method of my Router. My previous test verifies that the root path will call an index method, so now I only need to call that index method and validate that behavior. This has the added benefit of simplifying the test.

Here's the final version of my tests:

``` coffeescript tests/sample_router_factory_test.coffee
util = require('util')
should = require('should')
Backbone = require('backbone')
sinon = require('sinon')
SampleRouterFactory = require('../client/lib/sample_router_factory').SampleRouterFactory

describe "SampleRouter", ->
  items =
    find: sinon.spy()

  factory = new SampleRouterFactory(Backbone, items)
  router = factory.getRouter(Backbone)

  it "should have an index route", ->
    router.routes[''].should.equal('index')

  it "should retrieve all items when navigating to index", ->
    router.index()
    items.find.called.should.equal true
```

A bit roundabout, but in the end what I was trying to accomplish. Source <a href="https://github.com/skalb/meteor-examples/tree/master/mocha-router">here</a>.
