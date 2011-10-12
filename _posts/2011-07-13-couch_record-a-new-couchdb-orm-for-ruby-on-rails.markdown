---
layout: post
title: "couch_record: a new CouchDB ORM for Ruby on Rails"
categories: [rails, couchdb]
author: judy
---

We'll write a more detailed post about it later but we're excited to open this up now: couch\_record, a new couchDB ORM for Ruby on Rails, using couchrest and based on our experiences with couchrest\_model and, to a lesser extent, couchrest\_rails.

### Why do we need another CouchDB ORM?

When we were deciding which ORM to use, it seemed like [couchrest_model](https://github.com/couchrest/couchrest_model) had the most traction and most activity. But there were a number of aspects we were unhappy with (we wrote about one of them [here](http://blog.givmo.com/2011/05/performance-issues-with-couchrestmodel-and-activemodel/)).

Then recently we added [New Relic](http://addons.heroku.com/newrelic) and noticed some really problematic performance issues with our [items](https://www.givmo.com/items) page (to the point where we were restarting our app every 20 minutes to keep response times reasonable). We pinpointed the issue to a memory leak from [couchrest_model](https://github.com/couchrest/couchrest_model). We were a couple versions behind and the latest version fixed our problem but also introduced a number of new things that didn't work with the rest of our app. So we decided to roll our own with these goals:

* **Be Fast** - Your app has to map a lot of objects, so try to do as little as possible, especially when reading records from CouchDB

* **Be Simple** - You're going to end up looking at (debugging into) your ORM when something's not working as you expect, so the less code there is and the easier it is to understand, the better.

* **ActiveModel** - Use ActiveModel modules wherever possible to support ActiveModel functionality

* **Don't Support Everything** - CouchRecord is not a drop in replacement for ActiveRecord, CouchRest::Model or anything else

We made this for ourselves so it very specifically addresses our needs and the tests and documentation are a bit oblique but we'd love for other people to try it out and find it useful in their own projects!