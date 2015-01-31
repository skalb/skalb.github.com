---
layout: post
title: Multi-color heat map in JavaScript
categories:
- Visualizations
- JavaScript
status: publish
type: post
published: true
comments: true
---

While working on my [flight grid site](http://flightgrid.herokuapp.com/), I was looking for a way to add a multi-color heat map.

I found the following [example](http://www.designchemical.com/blog/index.php/jquery/jquery-tutorial-create-a-flexible-data-heat-map/), but for just one color.

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

Here's a working [example](http://www.skalb.com/assets/jquery-data-heat-map.html).
