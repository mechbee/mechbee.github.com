---
layout: post
title: "Obj to three.js, getting your materials to show up"
categories: [javascript]
author: judy
---

We were playing around with [three.js](https://github.com/mrdoob/three.js/) and nearly died of despair when we could not get the materials to show up on our otherwise perfect looking model for no discernible reason. In hindsight, it all seems so clear but at the time, we were not so zen.

So if someone gives you have an obj file with some jpgs that are supposed to be the textures for the model and asks you to render it in your browser with three.js, here's what to do:

* Use the [blender exporter](https://github.com/mrdoob/three.js/tree/master/utils/exporters/blender) or the [python obj exporter](https://github.com/mrdoob/three.js/tree/master/utils/exporters/obj) to turn your obj file into a js file.

* Open that js file and you will see an array of materials at the top that looks like:

{% highlight js %}
"materials": [{
  "DbgColor" : 15658734,
  "DbgIndex" : 0,
  "DbgName" : "material_12.001",
  "colorAmbient" : [0, 0.0111, 0],
  "colorDiffuse" : [0.0, 0.009412000468552117, 0.0],
  "colorSpecular" : [1.0, 1.0, 1.0],
  "mapDiffuse" : "room_50k_cleaned_noroof_tex012.jpg",
  "mapDiffuseWrap" : ["repeat", "repeat"],
  "shading" : "Lambert",
  "specularCoef" : 1,
  "transparency" : 0.0,
  "vertexColors" : false
},{
  "DbgColor" : 15597568,
  "DbgIndex" : 1,
  "DbgName" : "material_13.001",
  "colorAmbient" : [0, 0.0111, 0],
  "colorDiffuse" : [0.0, 0.009412000468552117, 0.0],
  "colorSpecular" : [1.0, 1.0, 1.0],
  "mapDiffuse" : "room_50k_cleaned_noroof_tex013.jpg",
  "mapDiffuseWrap" : ["repeat", "repeat"],
  "shading" : "Lambert",
  "specularCoef" : 1,
  "transparency" : 0.0,
  "vertexColors" : false
}]
{% endhighlight %}

Notice how the `colorAmbient` values are all close to 0. Change all of these to something like `[0.5, 0.5, 0.5]`:
    
    "colorAmbient" : [0.5, 0.5, 0.5]
    
* Using the [three.js boilerplate](https://github.com/jeromeetienne/threejsboilerplate), move your js file into the js folder and the texture images into the images folder.

* Open up the index.html of the boilerplate and get rid of the sample object. 

* Add an ambient light: 

{% highlight js %}
var ambient = new THREE.AmbientLight( 0xFFFFFF );
scene.add(ambient);
{% endhighlight %}

* Load your js file with the JSONLoader and MeshFaceMaterial: 

{% highlight js %}
new THREE.JSONLoader().load('js/your_js_file_path_.js', function(geometry){
  var mesh = new THREE.Mesh( geometry, new THREE.MeshFaceMaterial());
  scene.add( mesh );
}, 'images');
{% endhighlight %}

Finally start the local server and navigate to [localhost:8000](http://localhost:8000) in your browser. 

    make server