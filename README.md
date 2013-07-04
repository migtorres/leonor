# WHY I LIKE AMD 

### (...AND NOT THE RAILS ASSET PIPELINE)



# AMD

- Asynchronous 
- Module 
- Definition



**Asynchronous module definition (AMD)** is a JavaScript API for defining modules such that the module and its dependencies can be asynchronously loaded. 

([Wikipedia](http://en.wikipedia.org/wiki/Asynchronous_module_definition))



**Asynchronous Module Definition (AMD)** is the most widely supported JavaScript module format. It's used by **cujo.js**, **jQuery**, **dojo**, **Mootools**, and several dozens of other libraries and frameworks. **AMD** is specifically designed for browser environments, but you can also use it in non-browser environments.

([Authoring AMD modules](http://know.cujojs.com/tutorials/modules/authoring-amd-modules))



## Is AMD worth it?
- There are many different ways for organizing and loading client-side JavaScript.
- The **AMD** approach is only one among many.
- Read: [CommonJS Is The Future...](http://old.alexmaccaw.com//posts/rails_js_packaging) and [Why Use Make](http://bost.ocks.org/mike/make/)
- Rails, in its Asset Pipeline, uses a different approach. 



## Why do I not like the asset pipeline - 1

It is not truly modular and there is no built-in way of protecting variables from the global name-space.



### Does this really matter?



Depends...

One could correctly argue that there is no big chance of variable names such as **backbone** or **jquery** conflicting, 

- but things are more subtle than how they appear...
- ... have a look at these two examples from our code-base:



## 1

```js

//= require_tree .

var roundToDecimals = function(number, places) {
    // more code
};

var addCommasToNumber = function(number) {
    // more code
};
```

([ mypolygon-v2 - application.js](https://github.com/unepwcmc/mypolygon-v2/blob/master/app/assets/javascripts/application.js))

These are global variables, with very generic names.

Still not a big issue? Maybe, but as an application grows, chances of conflicts grow too... **and there is something more...**



This way of organizing code does not help to keep things tidy... and maintainable. 
Every single piece of functionality should have its right place inside an application. 
**roundToDecimals**, for example, could live in a reusable and generic numbers module. This can be achieved within the **Asset Pipeline** too, but an AMD approach makes it more obvious. 



## 2

In our code every coffeScript file is compiled with a top-level function wrapper, that protects the global namespace, but then, in order to escape this wrapper, modules start with something like:

```coffee
  window.Pica ||= {}
```

This is not elegant. **Why?**

- One should not use **window** to pass around globals in the first place.
- One should not be concerned about the existence of **window.Pica** (why || ?).
- One is not taking advantage of new ECMASript 5 features such as [object.freeze](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
- In general, it is a matter of **personal taste**.



## Why do I not like the asset pipeline - 2

In the **Asset Pipeline**, the order of the files required in a manifest matters. And, unsurprisingly, one discovers quite soon that **"//= require_tree ."** is a bad practice.



Order also matters for the **script tags** in an HTML file, so what is the big deal here?

- **JavaScript** started off as a small scripting language.
- But nowadays it's huge and with so many files (modules) around the place, one should not be concerned about the loading order. **Less stuff to remember the better!**



## Why do I not like the asset pipeline - 3

The **Asset Pipeline** does not make it easy to split the **css** and **JavaSript** across different pages of a Web application.



**Nevertheless** the **Asset Pipeline** has one big advantage: if using **Rails** its built for **Rails**. 



## How do **AMD** modules work instead?



There are different implementations of the **AMD** API:

- [requirejs](http://requirejs.org/)
- [curljs](https://github.com/cujojs/curl)



From the excellently explained [Authoring **AMD** modules](http://know.cujojs.com/tutorials/modules/authoring-amd-modules):

```js
define(dependencyIds, factoryFunction);
```

As you can see from the first parameter, **dependencyIds**, you can pass an array of ids into **define**. These are the ids of other modules that your module requires to do its work. The second parameter, **factoryFunction**, is a function that creates your module and will be run exactly once. The factory is called with the dependent modules as parameters. Furthermore, **it is guaranteed to run only after all of the dependencies are known to be available**. In practice, the factory typically runs just before it's needed.

```js
// module app/mime-client
define(['rest', 'rest/interceptor/mime'], function (rest, mime) {
    var client;

    client = rest.chain(mime);

    return client;
});
```



A simple configuration file example from a **requirejs** implementation

```coffee
  # The path where your JavaScripts are located
  baseUrl: "static/js/"

  # Specify the paths of vendor libraries
  paths:
    jquery: "vendor/jquery-1.8.3.min"
    jquery_ui: "vendor/jquery-ui-1.10.0.custom"
    laconic: "vendor/laconic"
    bootstrap: "vendor/bootstrap"
    underscore: "vendor/miso-0.4.0/lib/lodash"
    backbone: "vendor/backbone"
    text: "vendor/text-2.0.3"
    d3: "vendor/d3"

  # Not AMD-capables per default,
  # so we need to use the **AMD** wrapping of RequireJS.
  shim:
    jquery_ui:
      deps: ["jquery"]
      exports: "jQuery"

    laconic:
      exports: "laconic"

    bootstrap:
      exports: "bootstrap"

    underscore:
      exports: "_"

    backbone:
      deps: ["jquery", "underscore"]
      exports: "Backbone"

    d3:
      exports: "d3"

  # Start loading the main app file. Put all of
  # your application logic in there.
  requirejs ['main']
```

Note: non compatible **JavaScript** libraries are declared in the **shim** section.



The fact that **AMD** modules get loaded asynchronously does not mean that modules can not be optimized.

Usually for production we pass the code through an optimizer, such as:

- [rjs](http://requirejs.org/docs/optimization.html)
- [cram](https://github.com/cujojs/cram)



From the [docs](https://github.com/cujojs/cram/blob/master/docs/introduction.md): 

**cram.js** concatenates the modules of your application into larger bundles that may be loaded much more efficiently than when loaded as separate modules. cram.js can create AMD- compatible bundles that may be loaded by an **AMD** loader, such as **curl.js** or bundles with an integrated loader.



An optimizer needs some configuration too. A simple example from a **requirejs** implementation:

```js
{
    mainConfigFile: '../js/app.js',
    baseUrl: '../js', 
    name: 'vendor/almond',
    include: [
        'd3',
        'jquery',
        'jquery_ui',
        'backbone',
        'underscore'
        'laconic',
        'text',
        // All other app dependencies 
        // are traced down from the following 2:
        'router',
        'main'
    ],
    insertRequire: ['main']
}
```
Note: the reference to the **mainConfigFile** and how the order of the elements in the include array is irrelevant.



## What are the main advantages of this kind of approach?



##1

The global name-space does not get polluted. 



##2

The order in which **JavaScript** modules are loaded does not matter.



##3

For big **JavaScript** applications one can optimize the payload and memory management by selectively loading different **AMD** module bundles for different parts of the app.



##4

For small **JavaScript** applications or more traditional **JavaScript** enhanced Websites one can optimize the code by minifying all of it in a single file. 



##5

It's the future. **Harmony** will have [modules](http://wiki.ecmascript.org/doku.php?id=harmony:modules). Not **AMD** modules, but one might as well get used to the concept.



##6

It is a good way for approaching programs. It helps to write cleaner, more flexible, testable and better organized code.



From the words of [John Hann](https://github.com/unscriptable):

(AMD) is the simplest format I've seen that doesn't require a build step. **It was designed for browsers**. You can get started with **AMD** by just downloading an **AMD** loader and writing some code. Hit F5 or Cmd-R and see your first module load.

([interview link](http://davidwalsh.name/unscriptable))



## AMD and the Asset Pipeline

In the end it all boils down to **JavaScript**, so you could use AMD modules in the **Asset Pipeline** by just writing, in application.js, something like:

```js
//= require require
```
... and take it from there

Or alternatively, use a gem such us [requirejs-rails](https://github.com/jwhitley/requirejs-rails).



Would this approach work well within the wider **Rails** environment, at all levels, client side testing included?

- I do not know yet.



References and further reading

- [Interview with John Hann](http://davidwalsh.name/unscriptable)
- [Authoring AMD modules](http://know.cujojs.com/tutorials/modules/authoring-amd-modules)
- [WHY AMD?](http://requirejs.org/docs/whyamd.html)