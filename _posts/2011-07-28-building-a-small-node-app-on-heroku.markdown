---
layout: post
title: "Geek fun: building a lil' node.js app"
categories: [heroku, node, javascript]
author: judy
---

We recently wrote a little service in Node.js and [deployed it on Heroku's Cedar stack](http://devcenter.heroku.com/articles/node-js). All it does is return UPS shipping rates when we do an http get with certain parameters. So instead of hitting UPS directly every time an [item page](https://www.givmo.com/items/ad22badf6f3d59481ea9b9bd54f45352) loads, we hit the node app and it either pulls the rate out of Redis or asks UPS, meaning that shipping rates should come up a lot faster. We didn't *really* need to spin up a Node app for this but hey, this is how we have fun :D.

It's a pretty basic app but there were a couple modules we used for the various pieces:

1. Query string parsing - [qs](https://github.com/visionmedia/node-querystring)
2. Caching - [RedisToGo](http://devcenter.heroku.com/articles/redistogo) and [redis](https://github.com/mranney/node_redis)
3. XML templating and parsing - [Mu](https://github.com/raycmorgan/Mu) and [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js)

And as an auxiliary bonus, anyone can use it to get UPS public rates.

### Node modules

#### qs

Heroku runs Node v0.4.7 and there's an included [query string module](http://nodejs.org/docs/v0.4.7/api/querystring.html) that parses the query string into an object. But we wanted to send nested parameters like 'destination\[state\]=NJ&destination\[zip\]=07305'. [qs](https://github.com/visionmedia/node-querystring) adds back the parsing of nested parameters that apparently got removed in 0.3.x. For example, hit http://localhost:8080/?destination\[state\]=NJ&destination\[zip\]=07305 in your browser while running this with node locally:

{% highlight js %}
var http = require('http'),
    qs = require('qs'),
    url = require('url');
http.createServer(function (request, response) {
  var query = url.parse(request.url).query;
  response.end(JSON.stringify(qs.parse(query)));
}).listen(8080);
{% endhighlight %}

#### redis

We don't charge flat shipping rates, it's just whatever UPS says it is and it's based on a [number of factors](http://blog.givmo.com/2011/08/why-is-shipping-so-expensive/). But since there are only 7 zones in the continental US and most things on Givmo are within 1-15 pounds, there's a small set of probably base rates, ripe for caching.

We decided to use redis partly because it seemed like the node community preferred it and partly because we thought it'd be fun. So we added the [Nano plan for RedisToGo on Heroku](http://addons.heroku.com/redistogo) and [node_redis](https://github.com/mranney/node_redis), hooking them up with [this little snippet](http://blog.jerodsanto.net/2011/06/connecting-node-js-to-redis-to-go-on-heroku/). Each key is set to expire after 24 hrs.

#### mustache and xml2js
The XML going back and forth is one of the annoying aspects of the UPS api. For the request, we made an XML file with [mustache template tags](http://mustache.github.com/mustache.5.html). A snippet:

{% highlight xml %}
{% literal %}
<ShipTo>
  <Address>
    {{#destination}}
    <PostalCode>{{zip}}</PostalCode>
    <StateProvinceCode>{{state}}</StateProvinceCode>
    {{/destination}}
    {{#residential}}<ResidentialAddress />{{/residential}}
  </Address>
</ShipTo>
{% endliteral %}
{% endhighlight %}

Then [mu](https://github.com/raycmorgan/Mu) to compile and render the template. When UPS returned an XML response, we used [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js) because it was simple and we liked how it mapped [XML to a js object](http://blog.nodejitsu.com/6-must-have-nodejs-modules).

### GET public rates

Anyone can use this to get UPS public rates. Why would you want to use it instead of the real UPS api though?

1. Faster ([except when AWS goes down and takes out Heroku/the internet](http://techcrunch.com/2011/08/08/amazon-ec2-outage/)).
2. Easier: No need to get a key or deal with xml.
3. Just for fun :)

Here are some ways to use it: 

[From your browser](http://hollow-spring-510.herokuapp.com/?destination[state]=NJ&destination[zip]=07305&origin[state]=NC&origin[zip]=27713&residential=false&weight=4)

With Node.js:

{% highlight js %}
var http = require("http"),
  qs = require("qs");

var query = qs.stringify({
  origin: { state: 'NC', zip: '27713' },
  destination: {state: 'NY', zip: '10001' },
  weight: '20',
  residential: 'true'
});

var options = {
  host: 'hollow-spring-510.herokuapp.com',
  path: '/?' + query
};

http.get(options, function(res){
  res.setEncoding('utf8');
  res.on('data', function(chunk){
    console.log(chunk);
  });
}).on('error', function(e){
  console.log("Error: " + e.message);
});
{% endhighlight %}

With Ruby: 

{% highlight ruby %}
require 'rest_client'

params = RestClient::Payload.generate(
  :origin => {:state => 'NC', :zip => '27713'},
  :destination => {:state => 'NY', :zip => '11211'},
  :residential => true,
  :weight => 2
)

puts RestClient.get "http://hollow-spring-510.herokuapp.com/?#{params}"
{% endhighlight %}