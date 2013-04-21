rocker.js
=========

This is a javascript client used to communicate with a [Rocker server](https://github.com/victorjonsson/PHP-Rocker).
This script **works both in a browser and as a nodejs module**. *(The Rocker server is a restful backend service (LAMP), [here you can read more about PHP-Rocker](https://github.com/victorjonsson/PHP-Rocker))*

#### Example

```js

var Rocker = require('rocker'),
    server = new Rocker('https://api.website.com/');

var userMeta = {
    birth : '1980-12-04',
    gender : 'male',
    country: 'Finland'
};
server.createUser('john@gmail.com', 'John', 'password', userMeta, function(status, user) {
    if( status == 409 ) {
        console.log('E-mail taken by another user');
    } else {
        console.log();
    }
});

```

### Methods

- **setSecret( s )** — In case using encrypted authentication (more info below)
- **setUser( email, password )** — Set authentication credentials
- **me( callback )** — Get user data belonging to authenticated user
- **createUser( email, nick, pass, meta, callback )** — Create a new user
- **request( obj )** — Request an API operation. Example

```js

var server = new Rocker('https://api.website.com/');
server.setUser('admin@gmail.com', 'thepassword');
server.request({
    path : 'email/send',
    method : 'POST',
    data : {
        to : 'anna@facebook.com',
        subject : 'Hi there!',
        body : 'I like your smile :)'
    },
    auth : true, // send authorization header
    onComplete : function(status, json, http) {
        console.log(status); // the http status code
        console.log(json); // json data returned by server
        console.log(http); // XHR object in the browser, http response object in nodejs
    }
});

```

### RC4 encrypted authentication

If you're using RC4 encrypted authentication you will have to give the shared secret to
your client before making any requests that requires authentication. Example:

```js
var server = new Rocker('https://api.website.com/');
server.setSecret('AodXqLN.3Amejksao!elM');
server.setUser('admin@gmail.com', 'thepassword');
server.me(function(user) {
  console.log('My name is ' + user.nick );
});
```