---
layout: post
title: Testing with Meteor, CoffeeScript and Mocha
categories:
- CoffeeScript
- JavaScript
- Meteor
- Mocha
- Programming
- Testing
status: publish
type: post
published: true
comments: true
---

Adding unit tests is straightforward in [Meteor](http://docs.meteor.com/#structuringyourapp). Any files in the tests/ folder will be ignored by the Meteor server.

<!--more-->

What you'll need:

- [Meteor](https://github.com/meteor/meteor)
- [Npm](https://github.com/isaacs/npm)
- [Mocha](https://github.com/visionmedia/mocha)

Optional: [Growl](http://growl.info/) (for notifications from mocha watch)

Getting everything running:

``` bash
meteor create mocha
cd mocha
meteor add coffeescript
```

Setting up Mocha to watch our coffee files and send Growl notifications:

``` bash

mocha tests/*.coffee -w -G --compilers coffee:coffee-script

```

Now to create a sample test:

``` coffeescript test.coffee
describe "Array", ->
  describe "#indexOf()", ->
    it "should return -1 when the value is not present", ->
      assert.equal -1, [1, 2, 3].indexOf(1)
      assert.equal -1, [1, 2, 3].indexOf(0)
```

Mocha should notice the file change and run the tests showing you the passing test.

[Source code here.](https://github.com/skalb/meteor-examples/tree/master/mocha)

Some notes:
- The growl notification didn't work for me, but I assume that's because I'm using an old version.
- When using Mocha watch, sometimes a failing test result would be outputted many times
- [Jasmine](https://github.com/pivotal/jasmine) should also work fine

