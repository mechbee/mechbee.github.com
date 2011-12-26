---
layout: post
title: "Organizing a large backbone and rails app, some thoughts"
categories: [javascript, rails]
author: judy
---

We recently started working on a new web app with backbone 0.5.3, rails 3.1, and couchdb. It's not the still-raw-and-pulsing bleeding edge but it's pretty new stuff. There are some nice examples of backbone apps, e.g.  [DocumentCloud](https://github.com/documentcloud/documentcloud), [CloudApp](http://cloudapp.github.com/engine/), [CloudEdit](http://cloudedit.jamesyu.org/), [Todo](http://documentcloud.github.com/backbone/examples/todos/index.html), etc., but conventions and best practices aren't there yet in the way that they are for Rails.

Our app is larger than any of these examples so we thought we'd share how we're currently organizing it, and handling routing, forms, and nested models.

###Folders and Files###

Our app files and directory structure are very similar to that of [backbone-rails](https://github.com/codebrew/backbone-rails) with a few differences. backbone-rails creates four folders under app/assets/javascripts/backbone:

    routers/
    models/
    templates/
    views/
    
Our app is in stealth mode right now so let's say it's a web app to connect Jarls, Guild Masters, and other quest givers with adventurers who want to do quests. Quest givers and quest takers have similar but different UI requirements. For example, they both look at quests but quest givers can edit the quest and see all the quest takers who are interested in the quest, etc. So instead of one backbone folder for the entire app, we've got folders for both groups and a folder for shared assets:
    
    app/assets/javascripts/quest_givers 
    app/assets/javascripts/quest_takers
    app/assets/javascripts/shared

###Routing###

There are two types of routes the backbone app needs to know about. 

1. backend rails routes so that models can sync with the server 
2. front-end routes for navigating around, triggering views and actions, etc
 
For the backend routes, we could just set it on the Collection e.g. 

{% highlight js %}
class QuestGiver.Collections.QuestCollection extends Backbone.Collection
  url: '/quest'
{% endhighlight %}

Instead we added an application.js.erb file to our rails app and filled a Skyrim.Routes object using the rails helpers. Something like: 

{% highlight js %}
var Skyrim.Routes = {}
Skyrim.Routes['quest_givers_quests_path'] = '<%= quest_givers_quests_path %>'
{% endhighlight %}
    
Then our Collection url is:

    url: Skyrim.Routes.quest_givers_quests_path

Front-end routes are used in a couple places: 

* in the backbone routers, typically: 

{% highlight js %}
class QuestGivers.Routers.QuestsRouter extends Backbone.Router
  routes: 
    "quests" : "index"
    "quests/new" : "newQuest"
    "quests/:id" : "show"
{% endhighlight %}
  
* in the backbone views for example when switching from new to show after a model is saved:

{% highlight js %}
class QuestGiver.Views.Quests.NewView extends Backbone.Views
  save: (e) ->
    @collection.create(@model.toJSON(),
      success: (quest) =>
        @model = quest
        window.location.hash = "/#{@model.id}"
    )
{% endhighlight %}
    
* in the jst templates in places like the href of anchor tags e.g. :

{% highlight html %}
<a href="#/<%= id %>">Show Quest</a>
{% endhighlight %}

However we wanted our routes defined in one place so we added a Routes object:

{% highlight js %}
window.QuestGivers.Routes =
  quests: '/quests'
  new_quest: '/quests/new'
  quest: '/quests/:id'
{% endhighlight %}

Then we used a slightly modified version of [router.js](https://gist.github.com/877784) and pass QuestGivers.Routes to addRoutes(). Now our routers, views, and templates look like: 

{% highlight js %}
class QuestGivers.Routers.QuestsRouter extends Backbone.Router
  initialize: (options) ->
    @routes = {}
    @routes[QuestGivers.Routes.quests] = 'index'
    @routes[QuestGivers.Routes.new_quest] = 'newQuest'
    @routes[QuestGivers.Routes.quest] = 'show'
    @_bindRoutes()
{% endhighlight %}

Don't forget to \_bindRoutes() when adding routes this way or they won't get triggered.
    
{% highlight js %}
class QuestGiver.Views.Quests.NewView extends Backbone.Views
  save: (e) ->
    @collection.create(@model.toJSON(),
      success: (quest) =>
        @model = quest
        window.location.hash = QuestGivers.Routes.quest(@model.id)
    )
{% endhighlight %}

{% highlight html %}
<a href="<%= QuestGivers.Routes.quest(id) %>">Show Quest</a>
{% endhighlight %}

###Forms and Nested Models###

We're not using the [backbone-rails gem](https://github.com/codebrew/backbone-rails) but we are using the [datalink plugin](https://github.com/codebrew/backbone-rails/blob/master/vendor/assets/javascripts/backbone_datalink.js) to connect models with form fields and the [rails-ified Backbone.sync](https://github.com/codebrew/backbone-rails/blob/master/vendor/assets/javascripts/backbone_rails_sync.js). We're also using [backbone-deep-model](https://github.com/powmedia/backbone-deep-model) so that we can do:

{% highlight js %}
class QuestGiver.Models.Quest extends Backbone.DeepModel
  defaults: 
    name: null
    address: new Skyrim.Models.Address()
    
class Skyrim.Models.Address extends Backbone.Model
  defaults:
    address: null
    city: null
    state: null
    zipcode: null

model = new QuestGiver.Models.Quest()
model.get("address.city")
{% endhighlight %}

###Going Forward###

We're just getting started so maybe in a few months we'll look back on this and shake our heads in wonder. We'll see where it goes!