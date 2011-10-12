---
layout: post
title: "Why I'll (probably) never use CoffeeScript"
category: rails
author: dustin
---

We started a new project using [Rails 3.1](http://rubyonrails.org/), so i was checking out all the new stuff in this release.  Most of it looks great, since the new baked in features solve problems that we were using third party or home grown solutions for anyway.  On our last project (Rails 3.0), we used [Less](http://lesscss.org/) and [Jammit](http://documentcloud.github.com/jammit/).  On this one, we use [Sass](http://sass-lang.com/) and [Sprockets](https://github.com/sstephenson/sprockets).  Rails chose for us and that's great, because now we don't have to deal with it.  But there's one new feature that I just don't get: [CoffeeScript](http://jashkenas.github.com/coffee-script/).

I mean, I get it.  Javascript has a lot of pomp and isn't a whole lot of fun to code in.  But it's the only language that the browser runs and thats a _real_ constraint.  If I write my client-side code in CoffeeScript, the browser still receives and runs Javascript.  That creates a huge disconnect between what I'm writing and what I'm running.  Now maybe your code runs perfectly every time, but mine doesn't.  And when it doesn't, I look at what's happening in the debugger.  What do I see in the debugger if I use CoffeeScript?  Not my code, but a _derivative_ of my code.  What do I have to do to fix a bug?  Mentally decompile the running Javascript to the source CoffeeScript, make the change in the CoffeeScript, and hope that it compiles back down to the Javascript that fixes my problem.  Harsh.

The problem here is that CoffeeScript fails at it's fundamental purpose as an **abstraction**: removing me from the underlying complexity.  It in no way removes me from having to deeply understand Javascript.  In fact, it _increases_ complexity because now I have to know Javascript and CoffeeScript and be able to mentally translate between the two.  

Now, as forward thinking person, I do see the value in alternate client side languages.  I suspect that if lots of devs start using CoffeeScript or [Dart](http://www.dartlang.org/) or whatever, the browsers will eventually implement some kind of native support for these languages, which will be great.  But this is a long way off and probably requires one language to dominate and all browsers to support it.  I'm just not willing to let my own code suffer for years helping this to happen.  Maybe you are, and for that I appreciate your sacrifice.

You can check out our new CoffeeScript-free project at [wheretoeatat.com](http://wheretoeatat.com/)
