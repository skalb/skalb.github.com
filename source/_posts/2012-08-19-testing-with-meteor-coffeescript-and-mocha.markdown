---
layout: post
title: Testing with Meteor, CoffeeScript and Mocha
tags:
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
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  title: Testing with Meteor, CoffeeScript and Mocha
  description: Testing with Meteor, CoffeeScript and Mocha
  keywords: meteor, meteor js, mocha, javascript, jasmine, testing, unit testing,
    bdd, tdd, coffeescript
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _facebookcount-cache: '0'
  _twittercount-cache: '0'
  robotsmeta: index,follow
---
Despite no mention in the official <a href="http://docs.meteor.com/">Meteor docs</a>, adding unit tests is fairly easy. For some reason, they don't mention this but any files in the tests/ folder will be ignored by the Meteor server.

<!--more-->

What you'll need:
<ul>
	<li><a href="https://github.com/meteor/meteor">Meteor</a></li>
	<li><a href="https://github.com/isaacs/npm">Npm</a></li>
	<li><a href="https://github.com/visionmedia/mocha">Mocha</a></li>
</ul>
<div>Optional:</div>
<div>
<ul>
	<li><a href="http://growl.info/">Growl</a> (for notifications from mocha watch)</li>
</ul>
</div>
<div>Getting everything running:</div>
<div>``` bash
meteor create mocha
cd mocha
meteor add coffeescript
```

</div>
Setting up Mocha to watch our coffee files and send Growl notifications:

``` bash

mocha tests/*.coffee -w -G --compilers coffee:coffee-script

```

Now to create a sample test, test.coffee:

``` coffeescript
describe "Array", ->
  describe "#indexOf()", ->
    it "should return -1 when the value is not present", ->
      assert.equal -1, [1, 2, 3].indexOf(1)
      assert.equal -1, [1, 2, 3].indexOf(0)
```

Mocha should notice the file change and run the tests showing you the passing test.

<a href="https://github.com/skalb/meteor-examples/tree/master/mocha">Source code here.</a>

Some notes:
<ul>
	<li>The growl notification didn't work for me, but I assume that's because I'm using an old version.</li>
	<li>When using Mocha watch, sometimes a failing test result would be outputted many times</li>
	<li><a href="https://github.com/pivotal/jasmine">Jasmine</a> should also work fine</li>
</ul>
