---
layout: post
title: Creating a document sharing site with Meteor.js
tags:
- Backbone
- CoffeeScript
- Handlebars
- JavaScript
- Meteor
- Programming
status: publish
type: post
published: true
comments: true
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  title: Creating a document sharing site with Meteor.js
  description: Create and deploy a real time document sharing website using Meteor.js
    and Backbone.js
  _twittercount-cache: '27'
  _facebookcount-cache: '4'
  keywords: Backbone.js,CoffeeScript,Handlebars,JavaScript,Meteor.js
  _avia_elements_avia_options_sentence: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  _avia_elements_theme_compatibility_mode: a:3:{s:15:"_slideshow_type";s:11:"fade_slider";s:19:"_slideshow_autoplay";s:5:"false";s:19:"_slideshow_duration";s:1:"5";}
  robotsmeta: index,follow
---
**Background:**

“Meteor is a set of new technologies for building top-quality web apps in a fraction of the time, whether you're an expert developer or just getting started.”
- <a href="http://www.meteor.com">Meteor.com</a>

**Goal:**

Create and deploy a real time document sharing website. The final product is at: <a href="http://docshare-tutorial.meteor.com">docshare-tutorial.meteor.com</a>.

<!--more-->

**Updated: Source code at:<a href="https://github.com/skalb/docshare-tutorial">https://github.com/skalb/docshare-tutorial</a>**

**Spec:**
<ul>
	<li>Single page app with two sections</li>
	<li>Section 1</li>
<ul>
	<li>List of documents each with edit and delete buttons</li>
	<li>Create new document button with name input</li>
</ul>
	<li>Section 2</li>
<ul>
	<li>Text area of the document currently being edited</li>
</ul>
</ul>
**Prerequisites:**
<ul>
	<li>Install Meteor</li>
</ul>
``` bash
$ curl install.meteor.com | /bin/sh
```

**Step 1: Getting things started**

Lets create the app:

``` bash
meteor create docshare-tutorial
```

Now, start the meteor server:

``` bash
cd docshare-tutorial
meteor
```

You should see the default site at<a href="http://localhost:3000/">http://localhost:3000</a>:

Lastly, add the other packages we are going to use

``` bash
meteor add coffeescript
meteor add backbone
```

**Step 2: Setting up the project**

Go ahead and delete<span style="text-decoration: underline;">docshare-tutorial.js</span>and empty out the contents of <span style="text-decoration: underline;">docshare-tutorial.html</span>.

Meteor lets you separate client and server code in 2 different ways:
1) Using the Meteor.is_client and Meteor.is_server flags
2) Place client and server Javascript in the /client and /server folders, respectively. Any Javascript at the root level with run on both.

I prefer method 2 since it feels a bit cleaner to me, but feel free to instead combine everything into one file. Create <span style="text-decoration: underline;">docshare-tutorial.coffee</span> at the root and <span style="text-decoration: underline;">client.coffee</span> in /client folder and <span style="text-decoration: underline;">server.coffee</span> in the /server folder.

**Step 3: Server**

Collections in Meteor are schemaless. We want our documents collection to be available to both the server and the client so we’ll add it to the root level.

``` coffeescript docshare-tutorial.coffee
 @Documents = new Meteor.Collection("documents")

```

Our document object will have two fields: name and text. Let’s create a sample document on startup.

<span style="text-decoration: underline;">server.coffee</span>:

``` coffeescript
Meteor.startup ->
  if Documents.find().count() is 0
    Documents.insert
      name: "Sample doc"
      text: "Write here..."
```

Now, if you restart the meteor server, you’ll be able to access that document in the browser. Try this in the developer console:

``` coffeescript
Documents.findOne()
```

You will see an object with the properties we just created. This is the only time you should need to restart the meteor server.

**Step 4: Client HTML**

Here’s we’ll define our head and body. The body will render two templates: documentList and documentView.

<span style="text-decoration: underline;">docshare-tutorial.html</span>:

``` html
<head>
  <title>docshare</title>
</head>
<body>
  {{> documentList}}
  <hr>
  {{> documentView}}
</body>

```

Next, create the two templates needed to display the documents: documentList and document.

<span style="text-decoration: underline;">docshare-tutorial.html</span>:

``` html
<template name="documentList">
  <h1>Welcome to document sharing!</h1>
  <div>
    {{#each documents}}
      {{> document}}
    {{/each}}
  </div>
  <div id="createDocument">
    Name: <input type="text" id="new-document-name" placeholder="New document" /><input type="button" id="new-document" value="create"/>
  </div>
</template>

```

Here we are using the built in Handlebars iterator #each to render the individual document objects. Finally, there’s a input field and create button to add a new document.

Now add the template to list the documents names each with an edit and delete button. We’ll also use a template method to determine which document is selected.

<span style="text-decoration: underline;">docshare-tutorial.html</span>:

``` html
<template name="document">
  <div class="document {{selected}}">
    <p>
    {{ name }}
    <input type="button" id="edit-document" value="Edit"/>
    <input type="button" id="delete-document" value="Delete"/>
    </p>
   </div>
</template>
```

Lastly, let’s add the actual text field that the users can edit.

<span style="text-decoration: underline;">docshare-tutorial.html:</span>

``` html

<template name="documentView">
  {{#if selectedDocument}}
  {{#with selectedDocument}}
    <div>
      <p>{{ name }}</p>
        <textarea id="document-text" rows="10" cols="80">{{ text }}</textarea>
    </div>
   {{/with}}
   {{/if}}
</template>

```

Note that this will only be rendered if a document is currently selected. Having both an #if and a #with seems redundant, but I didn’t see a better way. Based off the Handlebars documentation, I should only need the #if, but that doesn’t work.

**Step 5: Client Coffeescript**

First, we’ll setup a Backbone router to allow us to keep track of which document we’re viewing. This will allow us to support page refreshes and permalinking to documents.

<span style="text-decoration: underline;">client.coffee</span>:

``` coffeescript
DocumentsRouter = Backbone.Router.extend(
  routes:
    ":document_id": "main"

  main: (document_id) ->
    Session.set "document_id", document_id

  setDocument: (document_id) ->
    @navigate(document_id, true)
)

Router = new DocumentsRouter

Meteor.startup ->
  Backbone.history.start pushState: true
```

Basically this will store the document_id into the Meteor session whenever the user navigates to a URL or form /:document_id. If you’re not familar with Backbone, don’t worry about this, or read up at <a href="http://backbonejs.org/">http://backbonejs.org/</a>

We are also using Meteor.startup again here but for a different purpose. On the client side it will run after DOM is loaded every time. I think it would be more clear if this method didn’t mean different things based on context.

Next, we need to define where the documentList template gets its data and handle the create new button

<span style="text-decoration: underline;">client.coffee</span>:

``` coffeescript
Template.documentList.documents = ->
  Documents.find({},
    sort:
      name: 1
  )

Template.documentList.events =
  'click #new-document': (e) ->
    name = $('#new-document-name').val()
    if name
      Documents.insert(
        name: name
        text: ""
      )
```

Notice that we’re sorting the documents by name. Each time a new document is added it will be correctly injected into the DOM. The entire list is not recreated. There is also a bit of validation to make sure the name exists.

Documents.insert (and all collection operations) are non-blocking when called client side. Meteor will go ahead and insert the data to the local client and can optionally call a callback with an object identifier or error message after the real operation finishes. This is invisible to the user, of course.

Next, define the selected property and event handlers for edit and delete:

<span style="text-decoration: underline;">client.coffee</span>:

``` coffeescript
Template.document.events =
  'click #delete-document': (e) ->
    Documents.remove(@_id)
  'click #edit-document': (e) ->
    Router.setDocument(@_id)

Template.document.selected = ->
  if Session.equals("document_id", @_id) then "selected" else ""
```

Note that in handlers for events the this object is the actual Document object.

Next, define the selectedDocument using the id stored in the session and update the text of that document when the user presses a key.

<span style="text-decoration: underline;">client.coffee</span>:

``` coffeescript
Template.documentView.selectedDocument = ->
  document_id = Session.get("document_id")
  Documents.findOne(
    _id: document_id
  )

Template.documentView.events =
  'keyup #document-text': (e) ->
    # @_id should work here, but it doesn't
    sel = _id: Session.get("document_id")
    mod = $set: text: $('#document-text').val()
    Documents.update(sel, mod)
```

Meteor acknowledges in their docs: “For now, the event handler gets the template data from the top level of the current template, not the template data from the template context of the element that triggered the event. This will be changing.” This is why we have to pull the id from the session.

Lastly, add the css style for the selected div:
<span style="text-decoration: underline;">docshare-tutorial.css:</span>
[css]
.selected {
  background-color: yellow;
}
[/css]

**Conclusion:**
That’s it, done! 50 lines of HTML and 50 lines of Coffeescript for a very basic Google docs clone.

To test it out open two browsers and type!
