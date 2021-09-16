full login / cookie example

Super simple login page for the cookie example server. Note that the "name" property of our form elements will be used to identify data sent to the server.

```html
<!doctype html>
<body>
  <form action='/login' method='POST'>
    <input type='text' name='username'>
    <input type='password' name='password'>
    <input type='submit'>
  </form>
</body>
</html>
```

Below is an example server to process logins using cookies. You can assume `main.html`
is any arbitrary HTML page that shows content once the login is complete.

```js
const express = require( 'express' ),
      cookie  = require( 'cookie-session' ),
      app = express()

// use express.urlencoded to get data sent by defaut form actions
// or GET requests
app.use( express.urlencoded({ extended:true }) )

// cookie middleware! The keys are used for encryption and should be
// changed
app.use( cookie({
  name: 'session',
  keys: ['key1', 'key2']
}))

app.post( '/login', (req,res)=> {
  // express.urlencoded will put your key value pairs 
  // into an object, where the key is the name of each
  // form field and the value is whatever the user entered
  console.log( req.body )
  
  // below is *just a simple authentication example* 
  // for A3, you should check username / password combos in your database
  if( req.body.password === 'test' ) {
    // define a variable that we can check in other middleware
    // the session object is added to our requests by the cookie-session middleware
    req.session.login = true
    
    // since login was successful, send the user to the main content
    res.sendFile( __dirname + '/public/main.html' )
  }else{
    // password incorrect, redirect back to login page
    res.sendFile( __dirname + '/public/index.html' )
  }
})

// add some middleware that always sends unauthenicaetd users to the login page
app.use( function( req,res,next) {
  if( req.session.login === true )
    next()
  else
    res.sendFile( __dirname + '/public/index.html' )
})

// serve up static files in the directory public
app.use( express.static('public') )
```
