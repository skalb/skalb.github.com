---
layout: post
title: Multi-color heat map in JavaScript
tags:
- Data
- JavaScript
- Programming
status: publish
type: post
published: true
comments: true
meta:
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  title: Multi-color heat map in JavaScript
  keywords: JavaScript, Data, heat map
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  robotsmeta: index,follow
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  _facebookcount-cache: '0'
  _twittercount-cache: '0'
  _wpas_done_all: '1'
---
While working on my <a href="http://flightgrid.herokuapp.com/">flight grid site</a>, I was looking for a way to add a multi-color heat map.

I found the following <a href="http://www.designchemical.com/blog/index.php/jquery/jquery-tutorial-create-a-flexible-data-heat-map/">example</a>, but for just one color.

I modified it to work for two colors as shown below (warning messy code!):

<!--more-->

``` javascript
<script type="text/JavaScript">
$(document).ready(function(){
	// Function to get the Max value in Array
    Array.max = function( array ){
        return Math.max.apply( Math, array );
    };

    // get all values
    var counts= $('#heat-map-3 tbody td').not('.stats-title').map(function() {
        return parseInt($(this).text());
    }).get();
	
	// return max value
	var max = Array.max(counts),
      middle = max * 1.0 / 2;
	
	// add classes to cells based on nearest 10 value
	$('#heat-map-3 tbody td').not('.stats-title').each(function(){
		var val = parseInt($(this).text());
    if (val < middle) {
      var pos = parseInt(Math.round((val/middle)*100)).toFixed(0);
  		relative = parseInt(pos / 100.0 * 225).toFixed(0);
  		clr = 'rgb('+relative+','+255+','+relative+')';
  		$(this).css({backgroundColor:clr});
    }
    else {
      var pos = parseInt(100 - Math.round(((val-middle)/(max-middle))*100)).toFixed(0);
      relative = parseInt(pos / 100.0 * 225).toFixed(0);
      clr = 'rgb('+255+','+relative+','+relative+')';
      $(this).css({backgroundColor:clr});
    }
	});
});
</script>
```

Here's a working <a href="http://www.skalb.com/wp-content/uploads/jquery-data-heat-map.htm">example</a>.
