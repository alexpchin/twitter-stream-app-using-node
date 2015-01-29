Twitter Stream App Using Node
=============================

- Intro to Express.js the JS version of Sinatra

## Objectives

* Build RESTful sites with express.js
* Build realtime sites with websockets

## Recap

* What is Node?

Node is a low-level, non-blocking, event-driven platform which allows you to write JavaScript on the server-side.

* What is npm?

npm is Node's package manager. It's used to manage dependencies. Think of it like RubyGems.

## What is express.js?

Express.js is a simple web framework for Node.js. It provides many features for you to start using straight away (Routing, Sessions) that you would have to do yourself if using vanilla Node. Think of it like **Sinatra for Node**.

1. `mkdir twitter-stream && twitter-stream` (Not a typo. With zsh you don't need `cd`)
2. `npm init` (Hit enter to accept the defaults and see the new [package.json](https://docs.npmjs.com/cli/init) file
3. `npm install express --save` (`--save` will mean it gets added to the project dependencies, this is similar to a gem but you can see it!)
4. `touch app.js` in twitter-stream directory

Check out the package.json file:

```
"dependencies": {
  "express": "^4.11.1"
}
```

Let's start coding!

```
// app.js
    
var express = require('express');
var app     = express();
var port    = process.env.PORT || 3000;
    
app.get('/', function(req, res) {
  res.send('Hello World');
});


app.listen(port);
console.log('Server started on ' + port);
```

** Notice the get verb here, this can also be post, put, delete etc... **

Then run the app using:

```
node app.js
```

Navigate to `http://localhost:3000` and voila! 

Now this is pretty awesome (is it? )but it doesn't really do anything. Plus, what if we want to start creating pages instead of just using sending text? 

## Templates

Express comes with a default templating engine called [jade](http://jade-lang.com). It's similar to [HAML](http://haml.info) so it should be familiar. There is another common templating engine called [EJS](http://www.embeddedjs.com/) (Embedded JavaScript) which is similar to ERB.

Instead of just sending some text when we hit our site let's have it serve an index page.

First thing to do is to install jade.

### Install Jade 
```
npm install jade --save
```
You can install from a project with:

```
npm uninstall jade --save
```

---

You might want to `cmd+shift+p` and Install the Jade Sublime Package at this point

---

Now let's tell our app we want to use jade and where we are going to put the templates. 

We also have to change what happens when a user GETs '/'. Let's get it to render our index template instead of sending 'Hello World'.

### Change app.js to render index

```
# app.js

var express = require('express');
var app     = express();
var port    = process.env.PORT || 3000;

app.set('views', './views');
app.set('view engine', 'jade');

app.get('/', function(req, res) {
    res.render('index');
});

app.listen(port);
console.log('Server started on ' + port);
```

## Create some views

Create a jade index page:

```
touch views/index.jade
```

And add this code:

```    
html
  head
    title Welcome to express.js!
  body
    h1 Express and jade
    .container
      p This is a paragraph of text. Yay!
```       

## Middleware

You may have seen this word floating around or seen it when you did Sinatra (Rack Middleware). It's a bit of a funny concept but think about it as _something_ that sits in-between express and you. Kind of. Let me show you with an example.

In our Hello World app we are logging out the server port once it has started. That is it. We get no other information about requests or errors like we have in Rails. We can use some _Middleware_ to achieve this.

```
# app.js
.
.
.
app.set('view engine', 'jade');

// Middelware
app.use(function(req, res, next) {
  console.log('%s request to %s from %s', req.method, req.path, req.ip);
  next();
});

app.get('/', function(req, res) {
.
.
.
```
    
Let's go through this. After setting up our app and before our routes we tell our app to use a new function we are providing. That's all Middleware is! When writing custom Middleware, it's best practice to pass in the **req** object, the **res** object and finally **next**, _even if we don't use it!_ In this case, we are simply logging out the request method ('GET'), the request path ('/') and the request IP ('127.0.0.1' - localhost). 

### Not in the Console log of chrome but in your terminal console!
If you try and hit another path ('/fail') this will also show in the logger.

You may also notice the formatting of the string. This is 'node-speak' for this:

```
console.log(req.method + ' request to ' + req.path + ' from ' + req.ip);
```

Finally, **next**. This tells express to continue processing the request or the next piece of middleware. 

---

**You can edit the request from inside of a middleware function so if you are writing your own, be careful!**

---

## Better logging with Morgan 

Now our logging middleware is ok but it's not great. If this was Sinatra or Rails we would go off and find the best logging gem so let's do the same with npm.

We're going to be using a module called _morgan_. So let's install it in the normal way. Don't forget to add `--save` so it get's added to the dependencies!

```
npm install morgan --save
```

Then we require it in our app.js (at the top):

```
var morgan = require('morgan');
```

and finally _use_ it, not forgetting to **remove our own logging middleware**!

```
app.use(morgan('dev'));
```

Restart the server and check out the logging!

## Sinatra/reloader equivalent... Nodemon

Introduce [nodemon](https://github.com/remy/nodemon) so you don't have to keep restarting server, it does it for you! 

```
npm install nodemon -g
```

Another syntax is:

```
npm install -g nodemon 
```

The `-g` here has basically installed nodemon globally (more similar to the way that you install gems normally).

You start the app now with:

```
nodemon app.js
```

## Routing

Let's add some routes. This should all be familiar but let's go through it a little.

[ExpressJS 4.0](https://scotch.io/tutorials/learn-to-use-the-new-router-in-expressjs-4) comes with the new Router. Router is like a mini express application. It doesn’t bring in views or settings, but provides us with the routing APIs like `.use`, `.get`, `.param`, and `route`.

First we define our _router_. This is what handles our routing. It's normally better to use this way of doing routes (and extracting them in to their own files) as it makes applications more modular and you won't have a 500 line app.js.

```
var morgan  = require('morgan');
var express = require('express');
var app     = express();
var port    = process.env.PORT || 3000;
var router  = express.Router();
```

Needs to be under the definition of `var app`!
Then we add our routes.

```
router.get('/', function(req, res) {
  res.render('index', { header: 'index!'});
});

router.get('/contact', function(req, res) {
  res.render('contact', { header: 'contact!'});
});

router.get('/about', function(req, res) {
  res.render('about', { header: 'about!'});
});
```

At the bottom of the page add:

```
app.use('/', router);
```

As we saw before we are rendering our template and then passing in a local variable (_header_) to use in our template. Like instance variables defined in our controller we can then pass to our views in Rails.

## Adding WebSockets

So that's the basics of how to get up and running with expressjs. Now let's do something a bit more creative that just standard CRUD work.

### Recap

I want to take you back to week 1 (or 2, or 3) and talk about how the web works. In very simple terms! You have a client (your computer) and the server (somewhere). When you want some information your browser sends a request to get some data and then the server responds. This can be with a GET request or an AJAX request but it's always the client saying give me some data.

**Question: What are the issues with this?**

* The client is in 'control' (the server might have updates but the client doesn't know about them)
* The client has to request things that they don't know about

In comes polling! The client can keep 'polling' the server to see if it has any more data.

**Question: What are the issues with this?**

* It's slow! Polling every n seconds isn't ideal (Chat app example)
* If you poll too often your bandwidth will go through the roof and slow it all down

### Solution: Enter Websockets

WebSockets solves all this. It maintains an open connection from Server <-> Client that we can use to 'push' information down, like push notifications on your phone (Gmail through Mail.app example).

Other things like each time you send any HTTP request there is loads of extras (type, user-agent, cookies, date etc) but once the websocket connection is established there is none of that.

## Simple example: WebSocket Number Station

First thing we need is to install the [socket.io](http://socket.io) module.

`npm install socket.io --save`

Then require it in our app, with a few changes. First let's add a new require for the _http_ module which gives us the server that socket.io needs to listen to.

```
var morgan  = require('morgan');
var express = require('express');
var app     = express();
var server  = require('http').createServer(app);
```

Under the `console.log('Server started on ' + port);` add: 

```
var io 		= require('socket.io')(server);
```

We also need to also change at the bottom from `app` to `server`:

```
server.listen(port);
```


## Add Twitter Streaming API

Great! We're also going to using a module called [twit](https://github.com/ttezel/twit) to use with the Twitter Streaming API.

```
npm install twit --save
```

And add to your app.js

```
var Twit = require('twit');
```

Now you can either use your own twitter accounts or create fake ones for this purpose. 

### Create twitter app

Go to (Twitter)[https://apps.twitter.com] and create a new 'app'. 

- **Name:** twitter-stream-in-node
- **Description:** Small app to stream tweets from twitter.
- **Website:** http://127.0.1.1

Navigate to **Keys and Access Tokens** and copy the keys and generate **Your Access Token** and instantiate new Twit object.

```
var twitter = new Twit({
  consumer_key: 'abc123',
  consumer_secret: 'abc123',
  access_token: 'abc123', 
  access_token_secret: 'abc123'
});
```

## process.env.VARIABLE

Do you remember environment variables ENV['something']. In JS, we can use:

```
process.env.VARIABLE
```

You can console log this to see if it has worked:

```
console.log(twitter);
```

Now we connect to the Twitter Streaming API (using the twit module), get all the statuses (tweets) and filter them on our keyword.

```
var stream = twitter.stream('statuses/filter', { track: 'javascript' });
```

Now we set up our websocket . There are a number of reserved words (connect, connection, message, disconnect) that can't be used elsewhere. We want out tweets to stream when we connect to the page so we open a _connect_ channel.

Inside, we set up our tweet socket and finally we _emit_ our _tweet_ on the _tweets_ channel.

```
io.on('connect', function(socket) {
  stream.on('tweet', function(tweet) {
    socket.emit('tweets', tweet);
  });
});
```

## Client Side 

Now that's the server side sorted, now let's do the client. Open up our index.jade and add two things.

First thing is to include our socket.io library, we're going to use the CDN version.

```
script(src='/socket.io/socket.io.js')
```

Notice that the path is relative... That's being done for you...

## Let's check in Chrome's console

```
> io
< function lookup(uri,opts){if(typeof uri=="object"){opts=uri;uri=undefined}opts=opts||{};var parsed=url(uri);var source=parsed.source;var id=parsed.id;var io;if(opts.forceNew||opts["force new connection"]||false===opts.multiplex){debug("ignoring socket cache for %s",source);io=Manager(source,opts)}else{if(!cache[id]){debug("new io instance for %s",source);cache[id]=Manager(source,opts)}io=cache[id]}return io.socket(parsed.path)}

```

Then in `index.jade` add in our receiving code below `body`:

```
script(type='text/javascript').
  var socket = io();

  socket.on('connection', function() {
    console.log('Connected!');
  });

  socket.on('tweets', function(tweet) {
    console.log(tweet);
  });
```

We use one of the reserved events ('connection') to log out the fact we are connected and then we hook up to the _tweets_ channel and start logging out what is received.

This is great! We now have own tweets streaming but only to the console. Let's get it on the page with some jQuery.


## Back to the serve-side
Go back to our app.js and tidy up the tweet data we're sending through.

```
stream.on('tweet', function (tweet) {
var data = {};
  data.name = tweet.user.name;
  data.screen_name = tweet.user.screen_name;
  data.text = tweet.text;
  data.user_profile_image = tweet.user.profile_image_url;
  socket.emit('tweets', data);
});
```

Note the change to: `socket.emit('tweets', data);`
Now we can add this using jQuery. Append vs Prepend.

```
# index.jade

html
  head
    title Welcome to Express.js!
    link(href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css", rel="stylesheet")
    link(href="/css/styles.css", rel="stylesheet")
  body
    h1= header
    .container#tweet-container
  script(src='https://code.jquery.com/jquery-2.1.1.js')
  script.
    var socket = io('http://localhost');
    socket.on('connection', function() {
      console.log('Connected!');
    });
    
    socket.on('tweets', function(tweet) {
      var html = '<div class="row"><div class="col-md-6 col-md-offset-3 tweet"><img src="' + tweet.user_profile_image + '" class="avatar pull-left"/><div class="names"><span class="full-name">' + tweet.name + ' </span><span class="username">@' +tweet.screen_name + '</span></div><div class="contents"><span class="text">' + tweet.text + '</span></div></div></div>';
      $('#tweet-container').preappend(html);
    });

```

Notice we've added links to Bootstrap, jQuery and our own stylesheet.

## Style

Create folders for css and javascript and explain about static files. Add to app.js.

```
app.use(express.static(__dirname + '/public'));
```

Then create:

```
mkdir public
mkdir public/css
touch public/css/style.css
```

Then add this code:

```
# /css/styles.css

body {
  background-color: #00aff0;
  color: #323332;
}

h1 {
  text-align: center;
  color: #fff;
  text-shadow: 0 0 2px #323332;
  line-height: 3;
}

.tweet {
  padding: 5px;
  border: 1px solid rgba(50,51,50,0.5);
  margin-bottom: 10px;
  background-color: #fdfdfd;
  -webkit-border-radius: 3px;
  -moz-border-radius: 3px;
  border-radius: 3px;
}

.names, .contents {
  margin-left: 60px;
}
```

Demo! Profit!