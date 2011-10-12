---
layout: post
title: "Performance issues with CouchRest::Model and ActiveModel"
categories: [rails, couchdb]
author: dustin
---

For [Givmo](https://www.givmo.com), we have an admin page that lists about 100 records at a time and it was taking upwards of 10 seconds to load.  It doesn't do anything super complicated, so I didn't really have a good starting point for what to optimize.  Time to fire up the profiler.

The page in question loads 100 records (documents) from a [CouchDB](http://couchdb.apache.org) database and displays them to the user in a table.  Nothin' crazy.  We use [CouchRest Model](https://github.com/couchrest/couchrest_model) to do our ORM, which makes our model classes behave like ActiveModels.  This makes them more or less compatible with tools that like to work with ActiveRecord.

I wrapped the offending code with the [ruby-prof gem](https://github.com/rdp/ruby-prof) and generated some profiling data.  After some digging, I found that all the time was being spent in 2 methods:

* [ActiveModel::AttributeMethods#respond_to?](http://api.rubyonrails.org/classes/ActiveModel/AttributeMethods.html#method-i-respond_to-3F)
* [ActiveModel::AttributeMethods#method_missing](http://api.rubyonrails.org/classes/ActiveModel/AttributeMethods.html#method-i-method_missing)


CouchRest::Model doesn't create accessors for the model's properties.  Instead, it keeps a list of all the properties (if you're looking, it's in [couchrest_model/lib/couchrest/model/properties.rb](https://github.com/givmo/couchrest_model/blob/master/lib/couchrest/model/properties.rb)) and lets activemodel's #method\_missing deal with it ([activemodel/lib/active_model/attribute_methods.rb](https://github.com/rails/rails/blob/v3.0.3/activemodel/lib/active_model/attribute_methods.rb)). So when a record is being loaded from the database and you have a property called 'email', it first checks #respond\_to? to see if there's a setter (#email=) and then calls the setter which triggers #method_missing.

Both #respond\_to? and #method\_missing have to do roughly the same thing: check if there's a property named 'email' in the list of properties.  They both call the same method to do this: #match\_attribute_method?.  This method (and its children) could use some optimization.  It maps the array of Properties to an array of property names (creating a new array of names every call) and then loops through the new array looking for the name.  For 100 records with 10 properties, this gets called about 20,000 times.  Ouch.

So, for this particular page, I cut CouchRest::Model out of the picture.  The time to render the page went from 10 seconds to 0.077 seconds.  That's 2 orders of magnitude folks.

Incidentally, the query to get the data from the [Cloudant](http://cloudant.com/) server, over the internet, though my measly cable modem takes about 0.5 seconds.  Something doesn't feel right with it taking 20 times longer to map the data into objects than it does to load it from the database.

Oh and we're on Rails 3.0.3 and couchrest_model 1.0.0.beta8.