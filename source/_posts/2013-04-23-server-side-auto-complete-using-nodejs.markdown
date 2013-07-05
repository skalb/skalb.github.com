---
layout: post
title: "Server side airport autocomplete using NodeJS"
date: 2013-04-23 15:47
comments: true
categories:
- JavaScript
- NodeJS
- Express
status: publish
type: post
published: true
---

This post shows how to enable autocomplete on a web form that consumes a JSON api running on nodeJS.

[Example](http://airport-autocomplete.herokuapp.com/)

[Source](https://github.com/skalb/airport-autocomplete)

<!--more-->

To enable the autocomplete, we need two main components to the app:

* [jQueryUI autocomplete](http://jqueryui.com/autocomplete/)
* [node-autocomplete](https://github.com/marccampbell/node-autocomplete)

The easiest way to add jQueryUI is to include a reference in our layout:

``` html layout.jade
!!!
html
  head
    title= title
    link(rel='stylesheet', href='http://code.jquery.com/ui/1.9.2/themes/base/jquery-ui.css')
    link(rel='stylesheet', href='/stylesheets/style.css')
    script(src='http://code.jquery.com/jquery-1.8.3.min.js')
    script(src='http://code.jquery.com/ui/1.9.2/jquery-ui.min.js')
    script(src='http://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.4.3/underscore-min.js ')
    script(src='/javascripts/index.js')
  body!= body
```

node-autocomplete and other necessary libraries can be added to package.json. Note: I'm required to use older verions of some packages because Heroku does not support the latest version of Express.

``` json package.json
{
  "name": "airport-autocomplete",
  "version": "0.0.1",
  "dependencies": {
    "express": "2.5.x",
    "jade": "*",
    "autocomplete": "*",
    "underscore": "*",
    "jquery": "*"
  },
  "engines": {
    "node": "0.8.x",
    "npm": "1.1.x"
  }
}
```

The dataset for this example is going to be airports and their codes. I wanted to be able to intelligently handle for the case where the user enters either the airport code or the name since most travel sites operate this way.

[Airports.dat](/assets/airports.dat) - format name|code

We need to keep lookups in memory to present the user with the properly capitalized airport name. During the actual autocomplete search, we will only use lowercased entries and input. Therefore, we need to keep track of three things:

* The actual autocomplete object where we load both codes and names into
* One lookup to retrieve the full name from the code
* Another lookup to retrieve the full name from a lowercased airport name

``` javascript routes/index.js
fs.readFile('data/airport-codes.dat', function(err, data) {
  var airports = [];

  _.each(data.toString().split('\n'), function(a) {
    var parts = a.trim().split("|"),
        airportName = parts[0],
        airportNameLower = airportName.toLowerCase(),
        airportCode = parts[1];
        airportCodeLower = airportCode.toLowerCase(),
        fullName = airportCode + " - " + airportName;

    airportNames[airportNameLower] = fullName;
    airportCodes[airportCodeLower] = fullName;

    airports.push(airportNameLower);
    airports.push(airportCodeLower);
  });

  namesAC.initialize(function(onReady) {
    onReady(airports);
  });
});
```

In the nodeJS route that returns the airport data, we'll use the input to test if it matched either an airport name or an aiport code and return those results.

``` javascript routes/index.js
exports.airports = function(req, res) {
  var airport = req.query["term"].toLowerCase(),
      results = namesAC.search(airport);

  var airportResults = _.map(results, function(a) {
    return airportNames[a] || airportCodes[a];
  });

  res.send(airportResults);
};
```
Here's what a [sample request](http://airport-autocomplete.herokuapp.com/airports?term=RIO) for RIO would return:

``` json
[
    "RBR - Rio Branco, AC, Brazil",
    "RCU - Rio Cuarto, CD, Argentina",
    "GIG - Rio De Janeiro, RJ, Brazil",
    "RGL - Rio Gallegos, Argentina - Internacional",
    "RIG - Rio Grande, RS, Brazil",
    "RGA - Rio Grande, TF, Argentina",
    "ROY - Rio Mayo, CB, Argentina",
    "RVD - Rio Verde, GO, Brazil",
    "RCH - Riohacha, Colombia"
]
```

The last part is to wire up the actual jQuery UI autocomplete. I've also added a handler on the open to highlight the term from the search.

``` javascript public/javascripts/index.js
(function () {
  var termTemplate = "<span class='ui-autocomplete-term'>$1</span>";

  $(document).ready(function() {
    $("#origin").autocomplete({
      source: "/airports",
      minLength: 2,
      open: function(e,ui) {
            var acData = $(this).data('autocomplete');
            acData
                .menu
                .element
                .find('a')
                .each(function() {
                    var me = $(this);
                    var regex = new RegExp( '(' + acData.term + ')', 'gi' );
                    me.html( me.text().replace(regex, termTemplate) );
                });
        }
    });
  });
})();
```