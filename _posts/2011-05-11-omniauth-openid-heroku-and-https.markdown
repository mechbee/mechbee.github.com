---
layout: post
title: Omniauth, openid, heroku, and https
category: heroku
author: judy
---

We recently added the option to log in or sign up with Facebook, Twitter, or Google using the handy [omniauth](https://github.com/intridea/omniauth) gem.

Facebook and Twitter went swimmingly. We had already been integrating with Facebook and Twitter so that people could tweet or post to Facebook to tell their friends about the items they're giving away. Adding a sign up/log in pathway was just an extension of that functionality.

But we ran into a problem with Google and OpenID: they worked fine in development but kept failing with "invalid credentials" in production.

There are two main differences with our production environment:
1. It's on Heroku which has a [read-only system](http://devcenter.heroku.com/articles/read-only-filesystem)
2. It always runs over https

We looked at possible problems with writing to the filesystem on Heroku first. Even though we had the [filesystem path set to ./tmp](http://kent.posterous.com/omniauth-rails-3-heroku-google-apps-tempfile), we were still worried about it and thought about [switching to a memcache store](http://www.arailsdemo.com/posts/18). But then we tested another app on Heroku with the same filesystem store writing to ./tmp and that worked fine.

Okay maybe it was a problem with ssl. This turned out to be really hard to google because we kept trying some combination of "Omniauth openid heroku https" (thus the blatant google mongering of this title) but in fact the problem was with Rack. We're using rails 3.0.3, omniauth 0.2.5, devise 1.3.4, and rack 1.2.2. The [1.3.0.beta version](https://github.com/rack/rack/blob/1.3.0.beta/lib/rack/request.rb) of rack has a notably different request.rb from the [1.2.2 version](https://github.com/rack/rack/blob/1.2.2/lib/rack/request.rb). Notice the scheme and port methods.

*request.rb* in rack 1.2.2:

{% highlight ruby %}
def scheme;          @env["rack.url_scheme"]                  end
def port;            @env["SERVER_PORT"].to_i                 end
{% endhighlight %}

*request.rb* in rack 1.3.0.beta:

{% highlight ruby %}
def scheme
  if @env['HTTPS'] == 'on'
    'https'
  elsif @env['HTTP_X_FORWARDED_SSL'] == 'on'
    'https'
  elsif @env['HTTP_X_FORWARDED_PROTO']
    @env['HTTP_X_FORWARDED_PROTO'].split(',')[0]
  else
    @env["rack.url_scheme"]
  end
end

def port
  if port = host_with_port.split(/:/)[1]
    port.to_i
  elsif port = @env['HTTP_X_FORWARDED_PORT']
    port.to_i
  elsif ssl?
    443
  elsif @env.has_key?("HTTP_X_FORWARDED_HOST")
    80
  else
    @env["SERVER_PORT"].to_i
  end
end
{% endhighlight %}

When we go to /auth/google in production, rack 1.2.2 doesn't realize we're coming from https and the return\_to for OpenID gets set to http. In development, this was fine because everything is http but in production, OpenID does a verification on the return_to url and raises a ProtocolError when the schemes don't match.

*[idres.rb:199](https://github.com/openid/ruby-openid/blob/master/lib/openid/consumer/idres.rb)*:

{% highlight ruby %}
[:scheme, :host, :port, :path].each do |meth|
  if msg_return_to.send(meth) != app_parsed.send(meth)
    raise ProtocolError, "return_to #{meth.to_s} does not match"
  end
end
{% endhighlight %}

Then the OpenID strategy in omniauth fails with the not so useful "invalid credentials" message.

*[open_id.rb:91](https://github.com/intridea/omniauth/blob/v0.2.5/oa-openid/l%20ib/omniauth/strategies/open_id.rb)*

{% highlight ruby %}
@openid_response = env.delete('rack.openid.response')
if @openid_response && @openid_response.status == :success
  super
else
  fail!(:invalid_credentials)
end
{% endhighlight %}

We ended up monkeypatching it with [this code](https://gist.github.com/825769) and it seems to be working so far.