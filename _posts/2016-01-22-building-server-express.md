---
layout: post
title: Building Servers with Node.js and Express, Part 1
---

Being able to write servers in Node.js was an incredible step forward for web development, as it allowed us to have a truly seamless integration with the client and the server, and build full stack applications entirely in JavaScript.  Writing servers in vanilla Node is not bad, but when you compare it with building an express server, it almost feels archaic.  

Express is a node module that greatly reduces the amount of boilerplate code you need to build a Node server, and makes for a much more readable, intuitive codebase.  In this blog, I want to go over an a super basic Express server, and then a slightly more advanced one.

To start, I want to note that there are generally many ways to accomplish a given task, and with Express we are given a lot of freedom regarding the setup of our server structure and the particular Express methods we use.  If you want to follow allong with the snippets, create a new directory.  Inside the directory, use "npm init" to initialize a package.json (If you don't have node, you'll need that first).  Next run "npm install --save express", which means we are telling npm to install Express in the current directory and add it to our dependencies in our package.json.  Next, create a file called server.js, copy the code snippets in, and execute "node server.js" in the correct directory.

###Basic
{% highlight js %}
//Require Express and create an instance of Express
var express = require('express')
var app = express()

//Handle GET requsts to the base route "/"
app.get("/", function(req, res) {
  res.send("This should appear in the broswer")
});

//Tell our app which port to listen on
app.listen(3000, function() {
  console.log("server listening on port 3000");
});
{% endhighlight %}

Theres a few important things going on here that tell us a lot about what Express can do. First, we require express, create a variable called "app", and set that equal to the invocation of Express.  This means app will have all of the Express methods available to it through dot access, as we see in the example.  

The second thing that happens is a very basic case of request handling.  Here, we specify three main things: The type of the request (GET), the specific route that we want to listen for ("/" the base route of our server), and the request handler.  The arguments to the request handler are the http request and response objects.  It is inside the callback function that we will process any data the user may have attached to the request object (ex. form data).  After we process the data, we can send some data back to the user by invoking the send method on the response object.  Here we send back a simple string, but you can imagine sending back a much more complex set of data if we needed to, using the exact same syntax.

The last thing we do is tell our app which port to listen on.  This could be any available port on your personal machine or the machine that is hosting your server.  3000 is a common choice.  

If you run the server, then go to your web browser and type "localhost:3000" into the address bar, you should see the response string displayed on the screen.  In the next example, we'll build a server that looks more like something you might see in an actual application. Specifically, we'll build something that has a couple of the basic ingredients of building a server for a social network.

###Better
{% highlight js %}
//Require node modules and create and instance of Express
var express = require('express')
var bodyParser = require('body-parser')
var morgan = require('morgan');
var app = express()

//Use bodyParser for processing POST request body data
app.use(bodyParser.urlencoded({extended: true}));
app.use(bodyParser.json());

//Use morgan for logging request information to the console
app.use(morgan('dev'));

//Request Handlers
app.post('/signin', function(req, res) {
  var userData = req.body.userData;
  //Do some database operation to validate user info..
  if (validated) {
    res.status(200).send(someSuccessData);
  } else {
    res.status(422).send("username/password incorrect!")
  }
});

app.get("/profile/:id", function(req, res) {
  var userID = req.params.id
  //Fetch profile data from the DB using the provided ID.
  res.status(200).send(userProfileData)
});

app.get("/friendList/:id", function(req, res) {
  var userID = req.params.id
  //Find user's friend list in the DB w/ provided ID.
  res.status(200).send(userFriendList);s
});

//Tell our app which port to listen on
app.listen(3000, function() {
  console.log("server listening on port 3000");
});
{% endhighlight %}

Now we have a server that can actually handle multple tasks.  First we require two additional node modules.  The first is "body-parser", which we use to convert request body information into readable JSON format.  If you have ever done a similar task with Node.js, you'll remember working with "response.on('data', function(chunk){body += chunk}..." and building up the data object in chunks.  Body-parser is awesome because it does all of that work for us in two lines, and stores all the data we need inside the req.body object.  The second new module we use is "morgan", which deals with tracking server request metadata.  There are a number of awesome ways to use morgan.  Here we go for the simplest, by using morgan in 'dev' mode, we will log the meta data for each request made to our server to the console (like ip address, request type, url, response codes, etc.).

The next thing we've done is add a few more request handlers, each of which correspond to a different request type/ url combination.  The first one handles post requests made to the route "/signup".  Here we would want to grab the user info from the req.body object and then check if that username/password combo exists in our database.  We conditionally send our results back based on if we find a match in the database.  One thing to note is how we can chain methods on the res object to send both a status code indicating the outcome of the request, and some data to go along with it.  

We have two more request handlers which demonstrate how one might go about fetching different kinds of data for a user that is already signed into the site.  I won't go into much detail on these except for noting that we are using request parameters here.  If you look at the end of each url you will see a "/:id".  This is a common practice for sending some small amount of data along with a get request.  On the client side, the application would simply append the user's id to the end of the request url before actually sending out the request.  When the server receives the request, it can
parse out one or more of the parameters using the req.params method.  I'll discuss using these a bit more in part 2 of this post.

This example has kept it pretty simple, but you can imagine easily adding more request handlers for different actions like adding friends, liking, messaging, posting content, etc.  Pretty quickly it will start to resemble a full fledged server.

In Part 2 of this post, I'll go in depth on a significant more complex example of an Express.  We'll go into things like middleware, maintaining modularity, security, integrating with a database, and more.  Hope you enjoyed reading!









