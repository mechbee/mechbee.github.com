---
layout: post
title: "How to auto reload gems that you're working on in Rails 3"
categories: [rails, dustin]
---

We recently moved our [CouchDB ORM](https://github.com/givmo/couch_record) code into its own Ruby gem because we're thinking about making it open source.  While working on the new ORM gem in the context of the Givmo Rails app, I wanted the the gem code to reload on every request like the rest of the code for the app.  For a while I was making frequent changes and restarting the server every time got old pretty fast.

I eventually figured out how to do this, but it wasn't super easy to find, so here's the short, sweet version of what you'll need to do:

<script src="https://gist.github.com/1051936.js"> </script>

First in development.rb, you need to add the lib directory for your gem to the autoload path. This is because gems normally only get loaded once, on startup. Since we want it to reload, we use the Rails autoload system. Files only autoload if the relevant constant is missing, so the last step is to tell rails to unload the root constant for our gem on every request.

Then in your Gemfile, tell bundler to load the gem in question from wherever you're working on it. In this case, it's in a sister directory to the Rails app.