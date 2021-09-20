# Object prototypes, the new keyword, and Function.prototype madness

## What does the new keyword do?
Given the following code:
```js
const Robot = function( name ) {
  this.name = name
}
Robot.prototype.speak = function() {
  console.log( `My name is ${this.name}.` )
}
const bobBot = new Robot( 'Bob' )
bobBot.speak() // => My name is Bob.
```
... what does the use of `new` accomplish when creating our new `Robot`?

1. Automatically creates a new object and binds it to the value of the `this` keyword inside our Robot function
2. Sets the prototype of our new object to whatever is found in the `Robot.prototype` property
3. The value of `this` is automatically returned by our `Robot` function

*INCREDIBLY IMPORTANT:* all three of the above rules only apply if the `new` keyword is used in front of our invocation of the `Robot` function!

What happens if we run: `const maryBot = Robot( 'Mary' )` after the code above?

## Avoiding the magic
That's a lot of behind-the-scenes automagickal nonsense that happens. How can we avoid it and still use prototypes? `Object.create`

```js
const a = { foo:42 }
const b = Object.create( a )
console.log( a.foo ) // => foo!
```

Note that this method employs *none* of the hand-waving magic that the `new` keyword exploits. Once you how prototypes work, it is cleaner and easier to understand. 

## OK, so then what's a mixin? 
JavaScript only lets you have one prototype chain for delegation. In order to use multiple objects (you can think of this as faux multiple inheritance) you can use mixins. In the example below, we mixin behaviors for movement and logging into objects that defer to `RobotProto`.
```js
const Mover = {
  move( x,y=0,z=0 ) {
    this.x += x
    this.y += y
    this.z += z
    return this
  },
  reset() {
    this.x = this.y = this.z = 0
    return this
  }
}

const Logger = {
  log( ...args ) {
    args.forEach( arg => console.log( `${arg}: ${this[ arg ]}` ) )
    return this
  }
}

const RobotProto = {
  speak() { 
    console.log( `My name is ${this.name}` )
    return this
  },
  init( name ) { 
    this.name = name
    // creates shallow copies of all properties on first argument
    Object.assign( this, Mover, Logger )
    return this
  }
}

const bobBot = Object.create( RobotProto )
  .init( 'Bob' )
  .reset()
  .move( 1,2,3 )
  .speak()
  .log( 'x','y','z' )
```

## I like classes. What's the problem with classes.
No giant problems, really, and ES6 provides a nice syntax for them. However, using them hides that classes use prototypes behind the scenes... they don't behave like classes in traditional classical inheritance langauges (Java, C++ etc.). For example:

```js
  class Cheap {
    constructor( value ) {
      this.value = value
    }
    cost() {
      console.log( `I cost ${this.value}.` )
    }
  }
  
  class Expensive extends Cheap {
    constructor( value ) {
      super( value )
    }
  }
  
  const pricey = new Expensive( 42 )
```

If we look at `pricey` in the developer console, we see that it has to navigate up the prototype chain *twice* to find its `cost` method!!! Using a mixin instead would prevent use of the prototype chain, which is important in frequent operations (think particle systems, or functions that operate over giant datasets).


# Modules and bundling like a pro: snowpack + webpack

There are a variety of systems for modularizing JS code, and a huge number of libraries have been created over the users for this. The simplest method is to check for the existence of a global object and place our code into this object. If it doesn’t exist, create it first. For example:

```js
if( typeof window.app === 'undefined' ) window.app = {}

window.app.mymodule = {
  foo:1,
  bar() { console.log( this.foo ) }
}
```

Using this strategy, we can safely import any number of files via script tags and not have to worry about the order that they’re loaded in, assuming only one of them makes use of `window.onload` to start our app running.

However, this can lead to some spaghetti code. What if one particular module needs to know about the existence of another module so that it can call functions from it? We need some sort of dependency system for this. [Snowpack](https://www.snowpack.dev/) is one such tool for the client that takes advantage of ES modules in the browser.

## Before we get to snowpack, what about importing / exporting in node.js?
Node.js conveniently includes a `require` function to include files and a `module.exports` object for exporting objects / functions / values. This method of exporting / requiring modules is known as `CommonJS`. We've used require a lot in our servers, but you might not have used `module.exports` yet:

```js
/************ FILE 1: module.js, a module to export **********/

module.exports = function() {
  console.log( 'a function' )
}

/************ FILE 2: importing our module ************/
const demomodule = require( './module.js' )

demomodule() // logs 'a function'
```

Any module can `require` any other module, and node.js will resolve circular dependencies whenever possible.

## ES6 import / export
ES6 added `import` and `export` keywords, however, it's only been in the last few years that browsers have added support (it's also only available in the most recent versions of Node). This module syntax is shortened to the name ESM for convenience, to distinguish it from CommonJS.

#### The basics
1. Let's make a basic module that exports a single number and name it `module.js`:
```js
export default 42
```

The `default` keyword tells us that `42` is the standard export for this file. You can only have one `default` export per module, however, we'll see soon that named exports enable us to export multiple values.

2. Let's create an `app.js` file that imports this value:
```js
import anumber from `module.js`

console.log( 'anumber is:', anumber )
```
3. OK, so we can include `app.js` in a web page with two extra steps:
   - We need to run a server with CORS enabled (e.g. http-server . -p 30000), can't load via `file://`
   - We need to add the `type="module"` attribute to our `<script>` tag.

```html
<!doctype html>
<html lang='en'>
<head>
  <script src='app.js' type='module'></script>
</head>

<body></body>

</html>
```

4. If you open the page and developer's console, you should see the number `42` printed.

#### Slightly more complex
In this next example, we'll export a file named `utilities.js`, that will provide us with a couple of different functions to use. We'll export these using `named` exports, instead of `default` exports like we did in our previous example.

```js
// utilities.js
const log = ( obj, prop ) => console.log( `${prop}: ${obj[ prop ]}`)

const square = num => num * num

export { log, square }
```

Now we can import and use those functions in our `app.js` file:

```js
// app.js
import { log, square } from './utilities.js' 

const app = { myvalue: square(42) }

log( app, 'myvalue' )
```

We can import `app.js` using the same HTML / server we used in the previous example. In the next example we'll look at how to make sure this script can run in all browsers, using `browserify` and `babel`, a transpiler for JavaScript.

## Demo w/ Three.js and ES Modules
Let's walk through using various ways to import modules. We'll use [three.js](http://threejs.org/), the most popular library for 3D in the browser, as an example module to play with.

1. Open some type of bash shell. Create a new directory `mkdir mydir` and cd into it.
3. Install three.js via npm. `npm install three --save`
4. OK, let’s make our main html file. We’ll include single JS file (named `main.js`). We *have to include the 'module' type attribute in order to be able to import other libraries inside of this file*. Don't forget this!

```html
<!doctype html>
<html lang='en'>
  <head>
    <style> body { margin:0 } </style>
    <script src='./main.js' type="module"></script>
  </head>
  <body></body>
</html>
```

6. Now we need to create our main JavaScript file. Before we learn how import three from the module we just installed, let's look at using it from a CDN:

```js
// import our three.js reference
import * as THREE from 'https://unpkg.com/three/build/three.module.js'

const app = {
  init() {
    this.scene = new THREE.Scene()

    this.camera = new THREE.PerspectiveCamera()
    this.camera.position.z = 50 

    this.renderer = new THREE.WebGLRenderer()
    this.renderer.setSize( window.innerWidth, window.innerHeight )

    document.body.appendChild( this.renderer.domElement )
    
    this.createLights()
    this.knot = this.createKnot()

    // ...the rare and elusive hard binding appears! but why?
    this.render = this.render.bind( this )
    this.render()
  },

  createLights() {
    const pointLight = new THREE.PointLight( 0xffffff )
    pointLight.position.z = 100
    this.scene.add( pointLight )
  },

  createKnot() {
    const knotgeo = new THREE.TorusKnotGeometry( 10, .1, 128, 16, 5, 21 )
    const mat     = new THREE.MeshPhongMaterial({ color:0xff0000, shininess:2000 }) 
    const knot    = new THREE.Mesh( knotgeo, mat )

    this.scene.add( knot )
    return knot
  },

  render() {
    this.knot.rotation.x += .025
    this.renderer.render( this.scene, this.camera )
    window.requestAnimationFrame( this.render )
  }
}

window.onload = ()=> app.init()
```

## Using Snowpack
Snowpack let's us use our ES Module syntax but figures out exactly what JS (and CSS too!) we need to put on the server and wraps it all up nicely for us. Otherwise you can quickly get into a situation where you're having to upload your entire `node_modules` folder to your server... take a look inside of that folder, it's usually pretty gnarly. Snowpack will figure out exactly what JS is needed for a given project and make it easily available for you.

1. Go ahead and run `npm install snowpack --save-dev` in your project directory.

2. Instead of loading three from our CDN, use the following line of code in `main.js` (comment out or replace the previous module loader):

```js
import * as THREE from 'three'
```

3. Launch the Snowpack dev server. In a terminal, make sure the working directory is the top-level of your project and run: `npx snowpack dev`. This should automatically open your webpage at `http://localhost:8080`.

4. Take a look at all the files that have been loaded in the Sources tab of your browser's developer tools to get a sense of the magic that is happening.

5. OK, pretty cool, but chances are you'll want to run your own server to serve up your site, and you definitely don't want to be running the Snowpack dev server in production. We can tell Snowpack to wrap everything up nice and neat for us using `npx snowpack build`. Snowpack creates a `build` folder; if you launch a standard server you can view all your files. You could also upload your build folder to Glitch / Heroku etc. 

## Using Webpack

While Snowpack works *great* for browsers that handle ES Module syntax, [~7% of browsers still don't support this](https://caniuse.com/?search=module). In order to support such browsers, we need to combine all our JS files into a single file (often called bundling) which we can then load. 

Before the widespread adoption of ES Modules, Webpack was a popular tool for doing this type of bundling. Snowpack let's us load Webpack as a plugin to handle bundling for us. 

1. Install the plugin: `npm install --save-dev @snowpack/plugin-webpack`

2. We need to generate a configuration file for Snowpack to setup Webpack as a plugin. Run `npx snowpack init` in your project directory to create a blank template.

3. Change the generated `snowpack.config.js` file to include the plugin:

```js
module.exports = {
  mount: {
    /* ... */
  },
  plugins: [
    [ '@snowpack/plugin-webpack', { entry: './main.js' } ]
  ],
  packageOptions: {
    /* ... */
  },
  devOptions: {
    /* ... */
  },
  buildOptions: {
    /* ... */
  },
};
```

The `entry` configuration point to our main JavaScript fle. From there, Webpack figures out every JS file that needs to be bundled up.

4. Now run `npx snowpack build` again and take a look at the resulting `build/index.html` file. No modules required, just regular script tags that should be supported in older browsers.
