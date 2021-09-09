# Express

- What is Express?
  - Provides convenient "middleware" services to easily handle dataflow and server logic.
  - Extremely popular, with lots of plugins that make common web tasks easy.
  
- What is middleware?
  - A simple function with the signature `function( request, response, next ){}`, where
    `next` is a function that is called when the middleware function has finished processing
    data.
  - Example: a simple logger of requested urls (adapted from express documentation).
  
```js
var express = require( 'express' )
var app = express()

app.use( function( req, res, next ) {
  console.log( 'url:', req.url )
  next()
})

app.get( '/', function (req, res) {
  res.send( 'Hello World!' )
})

app.listen(3000)
```

## Writing middleware for POST requests
- With this in mind, how would we write middleware to handle JSON information sent via POST request?

```js
const express = require('express')
const app = express()
const dreams = []

// for example only: routes for handling the post request
app.use( function( request, response, next ) {
  let dataString = ''

  request.on( 'data', function( data ) {
    dataString += data 
  })

  request.on( 'end', function() {
    const json = JSON.parse( dataString )
    dreams.push( json )
    // add a 'json' field to our request object
    request.json = JSON.stringify( dreams )
    next()
  })
})

app.post( '/submit', function( request, response ) {
  // our request object now has a 'json' field in it from our
  // previous middleware
  response.writeHead( 200, { 'Content-Type': 'application/json'})
  response.end( JSON.stringify( request.json ) )
})

const listener = app.listen( process.env.PORT, function() {
  console.log( 'Your app is listening on port ' + listener.address().port )
})
```

## But there's a middleware for grabbing  JSON, right?
- Yes indeedy. `body-parser` is the most popular one, but there are others.

- You can find middleware here: https://expressjs.com/en/resources/middleware.html

- *IMPORTANT:* For JSON data, the body-parser middleware will only take action if the data
  sent to the server is passed with a 'Content-Type' header of 'application/json'.
  For example:
```js
  fetch( '/submit', {
    method:  'POST',
    headers: { 'Content-Type': 'application/json' },
    body:    JSON.stringify( data )
  })
```

- Below below is an entire example server, less than 20 LOC not counting comments.

```js
const express    = require('express'),
      app        = express(),
      bodyparser = require( 'body-parser' ),
      dreams     = []

// automatically deliver all files in the public folder
// with the correct headers / MIME type.
app.use( express.static( 'public' ) )

// get json when appropriate
app.use( bodyparser.json() )

// even with our static file handler, we still
// need to explicitly handle the domain name alone...
app.get('/', function(request, response) {
  response.sendFile( __dirname + '/views/index.html' )
})

app.post( '/submit', function( request, response ) {
  dreams.push( request.body.newdream )
  response.writeHead( 200, { 'Content-Type': 'application/json'})
  response.end( JSON.stringify( dreams ) )
})

app.listen( process.env.PORT )
```

- We can also tell express to only use body-parser for a particular route,
  which is usually a more efficient way to use it. To do this we simply
  pass the middleware as the second argument to our route. You can do this
  with any route/middleware combination... the `post`, `get`, `put`, `delete`
  methods of the `app` object are all overloaded.
  
```js
app.post( '/submit', bodyparser.json(), function( request, response ) {
  dreams.push( request.body.newdream )
  response.writeHead( 200, { 'Content-Type': 'application/json'})
  response.end( JSON.stringify( dreams ) )
})
```
