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
        console.log(user);
    }
});

```

### Methods

- **setSecret( s )** — In case using encrypted authentication (more info below)
- **setUser( email, password )** — Set authentication credentials
- **me( callback )** — Get user data belonging to authenticated user
- **createUser( email, nick, pass, meta, callback )** — Create a new user
- **saveFile( content, name, callback, base64Decode, imageVersions)** — Save a file related to authenticated user (more info below)
- **fileUpload( inputElement, callback, beforeUploadCallback, imageVersions )** — Browser only (more info below)
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

Here you can [read more about which operations that is available out-of-the-box](https://github.com/victorjonsson/PHP-Rocker/wiki/API-Reference)

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

### Storing files

Browser example (requires support for FileReader):

```html
<html>
<head></head>
<body>
  <input type="file" id="file-upload" />
  <script src="js/rocker.js"></script>
  <script>
    var server = new Rocker('https://api.website.com/');
    server.setUser('admin@website.com', 'password...');
    server.fileUpload(

        // input of type file
        document.getElementByd('file-upload'),

        // Callback when operation finished
        function(status, obj) {
            if( status == 'success' ) {
                // all went well....
                console.log('File available at '+obj.location);
            }
            else if( status == 'browser-error' ) {
                // error in browser
                alert(obj); // alert error message
            }
            else if( status == 'server-error' ) {
                // something went wrong on backend
                console.log(obj);
            }
        },

        // Callback called before sending file to server
        function(file) {
            // file variable being an object representing the file
            // return false to prevent file from being sent to server
        },

        // Object declaring image versions that should be generated
        // in case the file is an image
        {
            thumb : '80x80',
            small : '160x0',
            medium : '468x0'
        }
    );
  </script>
</body>
</html>
```

Nodejs example:

```js

var fs = require('fs'),
    Rocker = require('rocker'),
    server = new Rocker('https://api.website.com/');

// Give client user credentials
server.setUser('admin@website.com', 'password....');

// Load image into base64 encoded string (only binary files needs to be base64 encoded)
var imgBase64 = new Buffer(fs.readFileSync('my-image.jpg', 'binary'), 'binary').toString('base64');

// Declare callback
var onFileSent = function(status, response) {
    if( status == 201 ) {
        // All is fine :)
        console.log('Image located at '+ response.location);
    } else {
        console.log(status);
        console.log(response);
    }
};

// Send file to server
server.saveFile(
    'file/my-image.jpg', // you can user what ever file name you want for your file
    onFileSent, // the callback
    true, // Boolean telling the server if file content should be base64 decoded
    {thumb : '100x100'} // object with image versions (generated in case saved file is an image)
});

```