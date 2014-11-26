MediaHubAPI
===========

What follows is a higher level description of the API that a Hub makes available to clients.  This API is meant to be implemented on top of the [Socket.io protocol](https://github.com/Automattic/socket.io-protocol), so it will be defined in terms of that protocol.  Socket.io solves many of the issues that would be faced in attempting to implement our own WebSocket solution, and can be substituted out in the future if needed.

## Client messages

A word on security:

Most Web Applications will run their authentication through HTTPS, with the server generating some type of token for the client.  The client then makes a seperate WebSocket connection and passes the token with each communication sent through the WebSocket.  This process requires respecting [Same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy), remembering to allow for [CORS](https://en.wikipedia.org/wiki/Same-origin_policy#Cross-Origin_Resource_Sharing) when we want to allow grant access to clients not hosted on the same domain as the Hub.  Although this project is starting with browser based clients, it does not plan on being limited to them.  Therefore utilizing browser based security methods seems to be bad idea.  WebSockets are not subject to Same-origin policy and therefore restricting/authenticating clients will be done within the socket.  All sockets should be opened under wss:// so as to mask the password sent from the client during the "auth" event.

##### `["auth", "<some password>"]`
Upon connection of the socket, the client should emit an `"auth"` event.  The only data passed with the event will be the Hub's set password.  This will be compared with stored value on the server.  If equal the server replies `[true]`, otherwise the socket is closed.  If 10 seconds has elapsed and no `"auth"` event has been recieved, then the Hub will close the socket connection.

##### `["saveScene", <Scene Object>]`
Save a Scene to the database.

##### `["listScenes"]`
Return a list of Scenes that can be subscribed to.  Will be an array of strings.

##### `["subScene", "<id of scene>"]`
Subscribe to a Scene.  Server will reply with the current Scene object.  Additionally this will subscribe the client to updates to the Scene as it changes. 

##### `["unsubScene", "<id of scene>"]`
Unsubscribe from a Scene.  Server will reply with `[true]` upon success.

## Hub messages

##### `["sceneUpdate", "<id of scene>", "<json patch>"]`
A delta of the JSON ([JSON Patch RFC 6902](http://tools.ietf.org/html/rfc6902)) to be applied to the specified scene.

##### `["show", "<id of scene>", "<name of media object>", <display information>]`
Instructions to show a specific media object listed in a scene.  `<display information>` may be any valid datatype.  It is expected that the subscribed client will know how to process it.
