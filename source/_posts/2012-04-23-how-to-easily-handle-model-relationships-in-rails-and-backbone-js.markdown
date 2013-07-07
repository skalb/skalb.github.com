---
layout: post
title: How to (easily) handle model relationships in Rails and Backbone.js
categories:
- Backbone
- CoffeeScript
- Rails
- Ruby
status: publish
type: post
published: true
comments: true
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  keywords: ! 'Backbone, Backbone.js, JavaScript, Coffeescript, Ruby, EJS, Rails,
    Models, Views, Templates, Single page, '
  description: Tutorial building a very basic single page project management app using
    Rails and Backbone.
  title: How to (easily) handle model relationships in Rails and Backbone.js
  robotsmeta: index,follow
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _facebookcount-cache: '0'
  _twittercount-cache: '1'
---
While playing around with Backbone.js, I couldn't find an easy way to build an app that used the RESTful hierarchy of my models. I think [Spine's](http://spinejs.com/docs/relations) implementation is fairly straightforward.

I did find a relevant [active project](https://github.com/PaulUithol/Backbone-relational), but for my specific case the added complexity of an additional component and dependency didn't seem justified. Rails already does the hard part for me, I just need Backbone to call the correct Urls.

I wanted to learn more about Backbone, so I prototyped a very basic project management app using Rails and Backbone called Trackbone that I'll walk through in this post.

<!--more-->

Briefly, this is a single page three panel app with drill-downs:
<ul>
	<li>Many projects</li>
	<li>Project has many Features</li>
	<li>Featurehas many Bugs</li>
</ul>

[Demo on Heroku](http://young-flower-9677.herokuapp.com/)
[Source](https://github.com/skalb/trackbone/tree/version1)

**Rails backend:**

Rails controllers provide the REST API for our Backbone app. I haven't inlined them here since they only have a few modifications post-scaffolding, but you can view them [here](https://github.com/skalb/trackbone/tree/version1/app/controllers)

Modifications:
<ul>
<li>Projects#index is moved to HomeController</li>
<li>Location is not returned in response after #create</li>
<li>Model is returned after #update</li>
</ul>

``` ruby routes.rb
root :to => "home#index"

Trackbone::Application.routes.draw do
  resources :projects do
    resources :features do
      resources :bugs
    end
  end
end
```

For setting up Backbone, the rails-backbone gem provides a good [guide](https://github.com/codebrew/backbone-rails/blob/master/README.md).

Originally, I scaffolded Backbone here as well. On review, I'm not sure I would do that again. I think you'll end at a better design if you start from scratch.

**Creating Backbone Models:**

Features only exist in the context of a Project, so they should only be loaded for a specific Project and similarly for Bugs.

``` coffeescript javascripts/backbone/models/project.js.coffee
class Trackbone.Models.Project extends Backbone.Model
  paramRoot: 'project'

  defaults:
    name: null

  loadFeatures: ->
    @features = new Trackbone.Collections.FeaturesCollection([], {project_url: @url()});

class Trackbone.Collections.ProjectsCollection extends Backbone.Collection
  model: Trackbone.Models.Project
  url: '/projects'
```

On reflection, loadFeatures is poorly named. It's really more of an 'initialize', but anyways, calling that method will create a FeaturesCollection and pass in the Url for this project. You can see how this is used in the Features model

``` coffeescript javascripts/backbone/models/feature.js.coffee
class Trackbone.Models.Feature extends Backbone.Model
  paramRoot: 'feature'

  defaults:
    name: null

  loadBugs: ->
    @bugs = new Trackbone.Collections.BugsCollection([], {feature_url: @url()});

class Trackbone.Collections.FeaturesCollection extends Backbone.Collection
  model: Trackbone.Models.Feature
  initialize: (model, args) ->
    @url = ->
      args.project_url + "/features"
```

Using the project_url from args will prepend all RESTful requests made on the features model with projects/:project_id.

``` coffeescript javascripts/backbone/models/bug.js.coffee
class Trackbone.Models.Bug extends Backbone.Model
  paramRoot: 'bug'

  defaults:
    name: null

class Trackbone.Collections.BugsCollection extends Backbone.Collection
  model: Trackbone.Models.Bug
  initialize: (model, args) ->
    @url = ->
      args.feature_url + "/bugs"
```

**Backbone Routers:**

Next we need to create the Projects router which will be the entry point into our single page app.

``` coffeescript javascripts/backbone/routers/projects_router.js.coffee
class Trackbone.Routers.ProjectsRouter extends Backbone.Router
  initialize: (options) ->
    @projects = new Trackbone.Collections.ProjectsCollection()
    @projects.reset options.projects

  routes:
    ".*" : "index"

  index: ->
    @view = new Trackbone.Views.Projects.IndexView(projects: @projects)
    $("#list-projects").html(@view.render().el)

    @newProjectView = new Trackbone.Views.Projects.NewView(collection: @projects)
    $("#new-projects").html(@newProjectView.render().el)
```

Because we're not worrying about permalinks (yet! I've started looking into this, and hope to create a follow-up post), we only need one catch-all route. We then pass in the respective models to our views.

**Listing projects:**

``` html views/projects/index.html.erb
<h1>Trackbone</h1>
<hr>
<div class="container">
  <div id="new-projects"></div>
  <div id="list-projects"></div>
</div>
<div class="container">
  <div id="new-features"></div>
  <div id="list-features"></div>
</div>
<div class="container">
  <div id="new-bugs"></div>
  <div id="list-bugs"></div>
</div>

<script type="text/javascript">
  $(function() {
    window.router = new Trackbone.Routers.ProjectsRouter(
      { projects: <%= @projects.to_json.html_safe -%> }
    );
    Backbone.history.start();
  });
</script>
```

There is a container, new, and list div for each model. Backbone will load data from our Rest API, build HTML using the Views and Templates and inject that HTML into those DIV elements. Note that we are passing in projects from Rails to the router. This allows us to grab data from the server when the page is first requested and save an additional call on page load. Since this is a single page app, this is the only Rails view we need.

To display our Project data we need three templates: index to list projects, project for a specific item, and new to create a project.

``` html javascripts/backbone/templates/shared/item.jst.ejs
<td><a href="#" class="select"><%= name %></td>
<td>[Destroy](#" class="destroy)</td>
```

``` html javascripts/backbone/templates/projects/index.jst.ejs
<h1>Listing projects</h1>

<table id="projects-table">
  <tr>
    <th>Name</th>
    <th></th>
  </tr>
</table>

<br/>
```

``` html javascripts/backbone/templates/projects/new.jst.ejs
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

These should be fairly self-explanatory. Each template will also need a Backbone View as well.

``` coffeescript javascripts/backbone/views/projects/index_view.js.coffee
Trackbone.Views.Projects ||= {}

class Trackbone.Views.Projects.IndexView extends Backbone.View
  template: JST["backbone/templates/projects/index"]

  initialize: () ->
    @options.projects.bind('reset', @addAll)
    @options.projects.bind('sync', @render)

  addAll: () =>
    @options.projects.each(@addOne)

  addOne: (project) =>
    view = new Trackbone.Views.Projects.ProjectView({model : project})
    @$("tbody").append(view.render().el)

  render: =>
    $(@el).html(@template(projects: @options.projects.toJSON()))
    @addAll()

    return this
```

This view receives the entire list of projects, renders the index view and appends a project view for each project.

``` coffeescript javascripts/backbone/views/projects/new_view.js.coffee
Trackbone.Views.Projects ||= {}

class Trackbone.Views.Projects.NewView extends Backbone.View
  template: JST["backbone/templates/projects/new"]

  events:
    "submit #new-project": "save"

  save: (e) ->
    e.preventDefault()
    e.stopPropagation()

    name = $("#new-project #name").val()
    if name
      $("#new-project #name").val('')
      @collection.create(name: name)

  render: ->
    $(@el).html(@template())

    return this
```

The new view simply providers a handler for creating a new project within the collection. The create method actually does three things: creates the model, POSTs the model to the server, and adds the model to the collection.

``` coffeescript javascripts/backbone/views/projects/project_view.js.coffee
Trackbone.Views.Projects ||= {}

class Trackbone.Views.Projects.ProjectView extends Backbone.View
  template: JST["backbone/templates/shared/item"]

  events:
    "click .select" : "select"
    "click .destroy" : "destroy"

  tagName: "tr"
  className: "item"

  select: () ->
    window.toggleSelected(@el)
    @model.loadFeatures()
    do (@model) ->
      @model.features.fetch success: ->
        featuresView = new Trackbone.Views.Features.IndexView(features: @model.features)
        $("#list-features").html(featuresView.render().el)

        # We should probably only render this once instead of each load
        newFeaturesView = new Trackbone.Views.Features.NewView(collection: @model.features)
        $("#new-features").html(newFeaturesView.render().el)

        $("#list-bugs").html('')
        $("#new-bugs").html('')
    @model.features.fetch()

  destroy: () ->
    @model.destroy()
    this.remove()

    return false

  render: ->
    $(@el).html(@template(@model.toJSON() ))
    return this
```

This view is really the heart of the app. Every time a Project is selected, we'll display the Features for that Project by calling fetch() and binding to success().

I'm not going to inline the templates and views for Features and Bugs since they are more or less identical, but feel free to browse through all the [client side code](https://github.com/skalb/trackbone/tree/version1/app/assets/javascripts/backbone)

Again, here's a working [demo](http://young-flower-9677.herokuapp.com/).

Please provide any feedback you have in the comments. Was this useful? Too long? Too much/too little code inline? I'm currently working on a couple more entires that will build on this one as I'm learning more about Backbone.js, so feedback is definitely useful.
