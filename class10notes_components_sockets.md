# Quizzes

# Things left to cover
  - D3
  - Realtime communication tech
    - WebSockets
    - WebRTC
  - Web applications
    - WebAssembly
    - WebWorkers
  - Server-Side rendering / templates
  - Misc
    - Typescript
    - CSS Preprocessors
    - JS Unit tests
    
# React Hooks
[React Hooks Documentation](https://reactjs.org/docs/hooks-overview.html)

> Hooks are functions that let you “hook into” React state and lifecycle features from function components.

```js
import React, { useState, useEffect } from 'react'

const Todo = props => (
  <li>{props.name} : 
    <input type="checkbox" defaultChecked={props.completed} onChange={ e => props.onclick( props.name, e.target.checked ) }/>
  </li>
)

const App = () => {
  const [todos, setTodos] = useState([ ]) 

  function toggle( name, completed ) {
    fetch( '/change', {
      method:'POST',
      body: JSON.stringify({ name, completed }),
      headers: { 'Content-Type': 'application/json' }
    })
  }

  function add() {
    const value = document.querySelector('input').value

    fetch( '/add', {
      method:'POST',
      body: JSON.stringify({ name:value, completed:false }),
      headers: { 'Content-Type': 'application/json' }
    })
    .then( response => response.json() )
    .then( json => {
       setTodos( json )
    })
  }
  
  // make sure to only do this once
  if( todos.length === 0 ) {
    fetch( '/read' )
      .then( response => response.json() )
      .then( json => {
        setTodos( json ) 
      })
  }
    
  useEffect( ()=> {
    document.title = `${todos.length} todo(s)`
  })

  return (
    <div className="App">
    <input type='text' /><button onClick={ e => add()}>add</button>
      <ul>
        { todos.map( (todo,i) => <Todo key={i} name={todo.name} completed={todo.completed} onclick={ toggle } /> ) }
     </ul> 
    </div>
  )
}

export default App
```

- We can use the `useEffect` to only run a block of code once

```js
  useEffect(()=> {
    fetch( '/read' )
      .then( response => response.json() )
      .then( json => {
        setTodos( json ) 
      })
  }, [] )
```

Need to access a parent component within a child? [Look into Refs](https://reactjs.org/docs/refs-and-the-dom.html)

## Quick refresher on Svelte
- Declarative reactive programming
- Changes to variables should trigger changes to UI according to component templates
- Simple button changing script to run at https://svelte.dev/tutorial
- Unlike React, Svelte does not use a virtual DOM
  - Svelte compiles code / React uses virtual DOM
  - Svelte is more efficient, but in most cases React is "fast enough"

```js
<script>
	let loggedIn = false
	
	const login = function() {
	  // assignment triggers UI update
	  loggedIn = !loggedIn
	}
</script>

{#if loggedIn === true }
  <button on:click={login}>log out</button>
{:else}
  <button on:click={login}>log in</button>
{/if}
```

## Socket Servers w/ Svelte (realtime chat)

### What are Web Sockets?
  - TCP and require a handshake to establish connection
  - realtime communication tech (chat, multiplayer games etc.)
  - remember to implement heartbeats to maintain connections
  - socket.io is a great library that abstracts sockets across programming langauges, but they're easy to use in JS as is.

### A quick server for both http and WebSockets (ws)

[ws](https://www.npmjs.com/package/ws) - A node package for creating servers that understand the WebSocket protocol

```js
/* 
1. Open up a socket server
2. Maintain a list of clients connected to the socket server
3. When a client sends a message to the socket server, forward it to all
connected clients
*/

const express = require('express'),
      app     = express(),
      ws      = require('ws'),
      http    = require('http')

app.use( express.static('public') )

const server = http.createServer( app ),
      socketServer = new ws.Server({ server }),
      clients = []

socketServer.on( 'connection', client => {
    
  // when the server receives a message from this client...
  client.on( 'message', msg => {
	  // send msg to every client EXCEPT the one who originally sent it
    clients.forEach( c => if( c !== client ) c.send( msg ) )
  })

  // add client to client list
  clients.push( client )
})

server.listen( 3000 )
```

### Svelte Client
- Make template project using [svelte-template](https://github.com/sveltejs/template):

```
npx degit sveltejs/template svelte-app
cd svelte-app
npm i
```

Then edit your App.js component:

```js
// App.js
<script>
  let ws, msgs = []
  window.onload = function() {
    ws = new WebSocket( 'ws://127.0.0.1:3000' )
    // when connection is established...
    ws.onopen = () => {
		  ws.send( 'a new client has connected.' )
      ws.onmessage = msg => {
	      // add message to end of msgs array,
	      // re-assign to trigger UI update
        msgs = msgs.concat([ msg.data ])
      }
    }
  }

  const send = function() {
    const txt = document.querySelector('input').value
    ws.send( txt )
	  // re-assigning to msgs variable triggers UI update
    msgs = msgs.concat([ txt ])
  }
</script>

<input type='text' on:change={send} />

{#each msgs as msg }
  <h3>{msg}</h3>
{/each}
```

# D3
[See the course handout on D3 + SVGs](https://github.com/cs4241-21a/cs4241-21a.github.io/blob/main/using_svg_and_d3.md)
