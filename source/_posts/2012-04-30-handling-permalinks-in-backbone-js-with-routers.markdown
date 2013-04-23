---
layout: post
title: Handling permalinks in Backbone.js with Routers
categories:
- Backbone
- CoffeeScript
- Rails
status: publish
type: post
published: true
comments: true
meta:
  _edit_last: '1'
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  keywords: ! 'Backbone, Backbone.js, JavaScript, Coffeescript, Ruby, EJS, Rails,
    Models, Views, Templates, Single page, '
  title: Handling permalinks in Backbone.js with Routers
  description: Handling permalinks in Backbone.js with Routers
  _syntaxhighlighter_encoded: '1'
  _facebookcount-cache: '0'
  _twittercount-cache: '1'
  robotsmeta: index,follow
---
This post is part of a series:
<ul>
	<li><a title="How to (easily) handle model relationships in Rails and Backbone.js" href="http://www.skalb.com/2012/04/23/how-to-easily-handle-model-relationships-in-rails-and-backbone-js/">First post</a></li>
	<li><a title="Extending and refactoring views in Backbone.js" href="http://www.skalb.com/2012/04/26/extending-and-refactoring-views-in-backbone/">Second post</a></li>
</ul>
One of the missing features in my prototype was handling of permalinks. To make things easy, I originally removed all the routes and added click handlers instead. In retrospect that was a mistake. Instead of having the app logic tangled up with click handlers, it would have been much more straightforward to define routes and use links.

Here’s the <a href="https://github.com/skalb/trackbone">source</a> and <a href="http://young-flower-9677.herokuapp.com/">demo</a>.

<!--more-->

In this post, I’m going to explain the  Backbone router step by step. First, I need to actually define what routes I want:

I need:
<ul>
	<li>Home page -> Load list of projects</li>
	<li>Selected project -> Load list of projects and features</li>
	<li>Selected feature -> Load list of projects, features, and bugs</li>
</ul>
``` coffeescript
routes:
  ".*"  : "showProjects"
  "projects/:project_id" : "showProjects"
  "projects/:project_id/features/:feature_id/*" : "showProjects"
```

Notice that all my routes point to the same function. The only difference is that the project_id and feature_id variables will be undefined. JavaScript won’t complain if your function call doesn’t match the function signature. It will just set them undefined.

Now let’s look at showProjects.

``` coffeescript
showProjects: (project_id, feature_id, bug_id) ->
  @renderViews(@projects, project_id, "Projects")

  if project_id
    @loadChildren(@projects, project_id, [feature_id, bug_id], "loadFeatures")
```

My original goal was to create a helper method that would support an abitrary length chain of loading children. That is, loadChildren will call loadFeatures which will then call loadChildren again etc. The final implementation isn’t quite that generic since it requires me to construct the individual item_ids at the start.

As I was finishing this prototype , I managed to remove all duplicate code by creating some helper methods.

``` coffeescript
clearType: (model) ->
  $("#list-#{model}").html('')
  $("#new-#{model}").html('')

renderView: (selector, view) ->
  $(selector).html(view.render().el)

renderViews: (items, item_id, type) ->
  indexView = new Trackbone.Views.IndexView(items: items, id: item_id, type: type)
  @renderView("#list-#{type.toLowerCase()}", indexView)

  newView = new Trackbone.Views.NewView(collection: items, type: type)
  @renderView("#new-#{type.toLowerCase()}", newView)
```

And now let’s look at loadChildren:

``` coffeescript
loadChildren: (items, item_id, child_ids, callback) ->
  item = items.get(item_id)
  item.loadChildren()
  item.children.fetch(
    success: =>
      @[callback](item.children, child_ids.shift(), child_ids)
  )
  item.children.fetch()
```

Okay, why am I indexing into the object with a string instead of just passing the function in directly. Well, because the fat arrow wasn’t working as expected. Either I was doing something wrong or there's a bug in the coffee-rails interpreter because js2coffee gave me a different result.

The full router:

``` coffeescript
class Trackbone.Routers.ProjectsRouter extends Backbone.Router
  initialize: (options) ->
    @projects = new Trackbone.Collections.ProjectsCollection()
    @projects.reset options.projects

  routes:
    ".*"  : "showProjects"
    "projects/:project_id" : "showProjects"
    "projects/:project_id/features/:feature_id/*" : "showProjects"

  clearType: (model) ->
    $("#list-#{model}").html('')
    $("#new-#{model}").html('')

  loadChildren: (items, item_id, child_ids, callback) ->
    item = items.get(item_id)
    item.loadChildren()
    item.children.fetch(
      success: =>
        @[callback](item.children, child_ids.shift(), child_ids)
    )
    item.children.fetch()

  renderView: (selector, view) ->
    $(selector).html(view.render().el)

  renderViews: (items, item_id, type) ->
    indexView = new Trackbone.Views.IndexView(items: items, id: item_id, type: type)
    @renderView("#list-#{type.toLowerCase()}", indexView)

    newView = new Trackbone.Views.NewView(collection: items, type: type)
    @renderView("#new-#{type.toLowerCase()}", newView)

  showProjects: (project_id, feature_id, bug_id) ->
    @renderViews(@projects, project_id, "Projects")

    if project_id
      @loadChildren(@projects, project_id, [feature_id, bug_id], "loadFeatures")

  loadFeatures: (features, feature_id, child_ids) ->
    @renderViews(features, feature_id, "Features")

    if feature_id
      @loadChildren(features, feature_id, child_ids, "loadBugs")
    else
      @clearType("bugs")

  loadBugs: (bugs, bug_id, child_ids) ->
    @renderViews(bugs, bug_id, "Bugs")
```

I also needed to tweak my item view to include the correct url for the select link.

``` coffeescript
Trackbone.Views.Projects ||= {}

class Trackbone.Views.ItemView extends Backbone.View
  template: JST["backbone/templates/item"]

  events:
    "click .destroy" : "destroy"

  tagName: "tr"
  className: "item"

  destroy: () ->
    @model.destroy()
    this.remove()

    return false

  render: ->
    name = @model.get("name")
    id = @model.get("id")
    url = "#{@model.collection.url()}/#{id}"
    $(@el).html(@template(name: name, id: id, url: url))
    if (@options.selected)
      window.toggleSelected(@el)
    return this

```

Again, here’s the <a href="https://github.com/skalb/trackbone">source</a> and <a href="http://young-flower-9677.herokuapp.com/">demo</a>.
