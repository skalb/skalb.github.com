---
layout: post
title: Extending and refactoring views in Backbone.js
tags:
- Backbone
- CoffeeScript
- EJS
- Handlebars
- JavaScript
- Programming
- Rails
- Ruby
status: publish
type: post
published: true
comments: true
meta:
  _edit_last: '1'
  description: Tutorial on how to extend views in Backbone.js
  title: Extending and refactoring views in Backbone
  robotsmeta: index,follow
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  keywords: Web, Tutorial, HTML, JavaScript, Templates, Backbone, CoffeeScript, Handlebars,
    Rails, Ruby
  _facebookcount-cache: '0'
  _twittercount-cache: '2'
  _syntaxhighlighter_encoded: '1'
---
In a <a href="http://www.skalb.com/2012/04/23/how-to-easily-handle-model-relationships-in-rails-and-backbone-js/" >previous post</a>, I built an example single page app using Backbone. One thing that bothered me was how similar the views are, yet didn't share any code. I think part of this was that I originally scaffolded the entire app and worked backwards.

<!--more-->

For example, here's Project vs Feature Index Templates:
``` html
<h1>Listing projects</h1>

<table id="projects-table">
  <tr>
    <th>Name</th>
    <th></th>
  </tr>
</table>

<br/>
```

vs.

``` html
<h1>Listing features</h1>

<table id="features-table">
  <tr>
    <th>Name</th>
    <th></th>
  </tr>
</table>

<br/>
```

This is easily refactored to:
``` html
<h1>Listing <%= type %></h1>

<table id="items-table">
  <tr>
    <th>Name</th>
    <th></th>
  </tr>
</table>

<br/>

```
Similarly, the New View changed from:
``` html
<h1>New project</h1>

<form id="new-project" name="project">
  <div class="field">
    <label for="name"> name:</label>
    <input type="text" name="name" id="name" value="<%= name %>" >
  </div>

  <div class="actions">
    <input type="submit" value="Create Project" />
  </div>

</form>
```
to
``` html
<h1>New <%= type %></h1>

<form id="new-item">
  <div class="field">
    <label for="name"> name:</label>
    <input type="text" name="name" id="name" value="<%= name %>" >
  </div>

  <div class="actions">
    <input type="submit" value="Create <%= type %>" />
  </div>

</form>
```

Great, that was easy and now I just reduced my total Templates. To share functionality between the Views I needed to create a base View class:

``` coffeescript
Trackbone.Views ||= {}

class Trackbone.Views.IndexView extends Backbone.View
  template: JST["backbone/templates/index"]

  initialize: () ->
    @options.items.bind('reset', @addAll)
    @options.items.bind('sync', @addAll)

  addAll: () =>
    # This shouldn't be needed, but for some reason
    # lists are rendered twice
    @$("tbody").html('')

    @options.items.each(@addOne)

  addOne: (item) =>
    item.collection = @options.items
    @$("tbody").append(@getView({model: item}).render().el)

  render: =>
    $(@el).html(@template(type: @options.type))
    @addAll()

    return this
```

First thing to note is that this View has an initialize method. But that method will never be called automatically because we're going to extend this View into a new class and create an instance of the subclass instead. Also note that we're calling a function getView() that isn't defined in this class.

``` coffeescript
#= require ../index_view

Trackbone.Views.Projects ||= {}

class Trackbone.Views.Projects.IndexView extends Trackbone.Views.IndexView
  initialize: () ->
    super
    @options.type = "Projects"

  getView: (options) =>
    new Trackbone.Views.Projects.ProjectView(options)
```

We can call the into the parent class by using super in this view's initialize. This is just a <a href="http://coffeescript.org/#classes">CoffeeScript shortcut</a> to apply the same arguments to the parent's constructor. I've also explicitly required the parent View class since Rails does not guarantee which order JavaScript files will be loaded in the browser. The getView function here creates the correct ItemView based off the Project model.

The New item View shown below is generic enough that it did not need to be extended for each model.

``` coffeescript
Trackbone.Views.Projects ||= {}

class Trackbone.Views.NewView extends Backbone.View
  template: JST["backbone/templates/new"]

  events:
    "submit #new-item": "save"

  save: (e) ->
    e.preventDefault()
    e.stopPropagation()

    name = @.$("#new-item #name").val()
    if name
      $("#new-item #name").val('')
      @collection.create(name: name)

  render: ->
    $(@el).html(@template(type: @options.type))

    return this
```

The Item View is a bit trickier because it contained the select handling for when an item was clicked. To be able to reuse the handling, I had to make the load methods consistently named loadChildren as shown below.

``` coffeescript
class Trackbone.Models.Project extends Backbone.Model
  paramRoot: 'project'

  defaults:
    name: null

  loadChildren: ->
    @children = new Trackbone.Collections.FeaturesCollection([], {project_url: @url()});

class Trackbone.Collections.ProjectsCollection extends Backbone.Collection
  model: Trackbone.Models.Project
  url: '/projects'
```

One thing to point out in the Item View base class is that I believe I could have used the <a href="http://coffeescript.org/#fat_arrow">fat arrow</a> to retain the correct context.

``` coffeescript
Trackbone.Views.Projects ||= {}

class Trackbone.Views.ItemView extends Backbone.View
  template: JST["backbone/templates/item"]

  events:
    "click .select" : "select"
    "click .destroy" : "destroy"

  tagName: "tr"
  className: "item"

  select: () -> 
    if @model.loadChildren
      window.toggleSelected(@el)
      @model.loadChildren()
      do (@model, @renderChildren) ->
        @model.children.fetch(
          success: @renderChildren(@model.children)
        )
        @model.children.fetch()

  destroy: () ->
    @model.destroy()
    this.remove()

    return false

  render: ->
    $(@el).html(@template(@model.toJSON() ))
    return this
```

Now the actual Project view only had to define how to render it's children.

``` coffeescript
#= require ../item_view

Trackbone.Views.Projects ||= {}

class Trackbone.Views.Projects.ProjectView extends Trackbone.Views.ItemView
  template: JST["backbone/templates/item"]

  renderChildren: (children) -> 
    featuresView = new Trackbone.Views.Features.IndexView(items: children)
    $("#list-features").html(featuresView.render().el)

    # We should probably only render this once instead of each load
    newFeaturesView = new Trackbone.Views.NewView(collection: children, type: "Features")
    $("#new-features").html(newFeaturesView.render().el)

    $("#list-bugs").html('')
    $("#new-bugs").html('')
```

Again, here's the <a href="http://young-flower-9677.herokuapp.com/">demo</a> and <a href="https://github.com/skalb/trackbone/tree/version2">source</a>.

I think this is a big improvement over the first version, though it's not quite as good as it could be. Having the loadChildren logic in the views doesn't really make sense, but I'm leaving those changes for when I implement permalinks.
