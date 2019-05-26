---
title: How to make a Todo App? (part 1A)
---
In this series of blog posts, I will be making a simple todo web app + a mobile app (in flutter). This is so that, by making a todo app, as a very simple example, I can explain to you all the basic things you need to know to make any app yourself. We will be making this app with an increasing complexity in each blog post. In the first one, we will start with a JQuery frontend-only app that uses the browser's session storage. Then in the next one we will use a backend made in Flask, with bcrypt authentication and SQLite database, and we will later add to it NodeJS, MongoDB, PassportJS, Flutter, React, Redux, GraphQL, ML, Blockchain, Quantum Computing, and all the latest buzzwords! :D (After all, it is 2019 and it is no longer acceptable in modern society to use a todo app that doesn't have atleast a Neural Network in it.)
All the code will be in a github repo, so you can follow the development at any step.

In this part, we will be using JQuery and the browser's session storage to create a client-side todo app. We'll add the backend next time. :) But first, I'll explain all the terms like Frontend, Backend, API, JSON, etc, so even beginners will have no trouble understanding any of this.

# The structure of any app

## The backend, frontend, and the middle-end

To build an app we'll first have to understand how different parts of an app work together. An app consists of  two parts, the frontend and the backend. The frontend code is the part that is responsible for displaying stuff  on your screen and managing the logic of user actions. The backend is the part which handles the "business logic" of your app and stores all the data. The backend is usually on the server side and the frontend on the client's computer. The thing that links the Frontend and the Backend is called _the API_ [^1]. The Frontend code connects to the backend and sends it some data in a response. 

<center> <img src="/assets/1_api_diagram.jpg" width="100%"> </center>

The frontend is made using technologies like HTML, CSS, JavaScript, JQuery, and React. The backend can be made in Python using the Flask framework or in Javacript with NodeJS (using Express) etc. The database can be either on the same server as the web server (running on a different process and port number) or it can be on a seperate server if you're making a large app. Databases come in two flavours, SQL databases (MySql, SQLite, etc.) and the new, shiny, NoSQL databases (MongoDB etc). You may have studied SQL earlier, it is a way to query relational databases. In the SQL-based databases, you need to have a proper data format defined using tables beforehand. But in the NoSQL databases you can simply use the database as a large JSON object, and forget about tables and SQL and stuff. (I've explained JSON below, don't worry if you don't understand it now).

In a web api, all the communication is done over HTTP, so that means the resuests will be sent as individual GET and POST requests [^2]. What that means, in simple terms is that we will be sending requests to the server in mainly two ways, a GET request to fetch some data from the server (like viewing some content), or a POST request, to send some new data to the server (or to change the server's state).

All the data transfer that happens between the server and client is done in a particular format called the _JSON format_ (JavaScript Object Notation). Let us first see how data is represented in this format.

## JSON Format

The JSON format is a lightweight data description format. It is a text format and is human readable (as opposed to a binary format). This format is used because it is very convinient to pass around in a web app to and from the server. It can be read and understood by the developer easily, can be parsed by the language easily, etc. In JSON, we can have  the following types of any data, _string, number, bool, null, array, and object_.

<center> <img src="/assets/2_json_types.jpg" width="100%"> </center>

The data types string, number, and bool[^3] are basic data types. The `null` type is used to show that there is no data associated with that field. The types Object and Array can contain other data types, even themselves. So you can have infinitely complex data types, with arbitrary nesting, Array within an Array, Array of Objects, Objects containing Arrays containing Objects etc.

You would think what is the use of such complicated data types, like "Objects containing Arrays containing Objects"?! Actually such highly nested data structures are pretty common in JSON! Let's imagine what kind of data our todo app would need to store. It would store an Array of Todos, and each Todo could be an object with the following _attributes_: `"todoid": number, "title": string, "finished": bool, "timestamp": number`. If we're making a bit advanced todo app, it may have "tags" which stores an array of strings or tag_id's. So we would then have the data as an array of objects containing strings, numbers, bools, and other arrays.

Some sample JSON data for our todo app:

```json
[
    {
        "todoid": 1,
        "title": "Get notes for endsem from teacher",
        "finished": true,
        "timestamp": 1558859316.3211691,
        "tags": ["study", "endsem"],
    },
    {
        "todoid": 2,
        "title": "Solve problem 225C on codeforces",
        "finished": false,
        "timestamp": 1558864813.0850985,
        "tags": ["codeforces"],
    },
]
```

## Back to the API

Our app communicates with the API by sending it GET and POST rquests on certain URLs, which are called _API endpoints_ or _routes_. These URLs can be like `/alltodos, /newtodo, /markAsDone` etc. The server itself is accessed using an ip address or domain name like `http://best_todo_app_ever.com/alltodos` or `http://12.34.56.78/alltodos` 

In our todo app, you can imagine the conversation between the server and client like this: (the parts in square brackets are the english translation)

```
client:

--> GET /alltodos

    ["show me the list of all todos stored in the server rn"]

server:

--> [{"todoid": 1,"title": "Get notes for endsem from 
    teacher","finished": true,"timestamp": 
    1558859316.3211691,"tags": ["study", "endsem"],},
    {"todoid": 2,"title": "Solve problem 225C on 
    codeforces","finished": false,"timestamp": 
    1558864813.0850985,"tags": ["codeforces"],},]

    [the server sent the data in JSON format]

----------------------------------------------------------

client:

--> POST /newtodo
    json data: {"title": "finish math assignment", 
    "finished": false, "timestamp": 1558865911.8238328,}

    ["add a new todo with the data I sent you"]

server:

--> OK, todo added. Here's the new todoid: 3

----------------------------------------------------------

client:

--> POST /deltodo
    data: {"todoid":3}

server:

--> OK, done

----------------------------------------------------------

client:

--> GET /todoDetails?id=2

[the request can also contain parameters in the URL 
itself, like the id=2 here]

server:

--> {"todoid": 2,"title": "Solve problem 225C on 
    codeforces","finished": false,"timestamp": 
    1558864813.0850985,"tags": ["codeforces"],}

----------------------------------------------------------
```

Note that in this model, only the client can initiate communication with the server by sending it a request. The server cannot send data to the client at an arbitrary time by itself. That is only possible with the new websockets technology, not with standard HTTP. If the client wants to check for new data on the server, it must do polling in the background.

## The web server

Okay, so now how will we make this API, and how will the client access it? To make a web API like this, you will need a software to listen for HTTP requests. This kind of software is called a _web server_, and luckily, it is already present in the standard libraries of many languages so you won't have to implement it yourself. What we will have to implement are simply functions that handle each of the API endpoints, or routes. For example, _Flask_, a python library allows us to create an API like this quite easily. Let us say we want to create an API with two endpoints, `/a` and `/b`, one which handles GET and one that handles POST. In flask it is as simple as this:

```py
from flask import *
app = Flask(__name__)

@app.route("/a", methods=["GET"])
def handle_a():
    # do stuff to handle request 'a'
    # get query parameters in request.args
    return response

@app.route("/b", methods=["POST"])
def handle_b():
    # do stuff to handle request 'b'
    # process some json data from request.json
    return response

if __name__ == "__main__":
    app.run()
```

There is very minimal boilerplate code, just 3-4 lines, and we simply have to declare the functions to handle the two routes and we can return the response directly from those functions. Flask makes it really easy to make web apps like this[^4].

But what is really happening here? Here we are using Flask to declare a set of routes. When we run this file with python, it fires up a default development web server (which is not too powerful or secure) to listen for requests and it runs these functions as soon as it recieves a request and returns the response. However if you want to deploy this on a real server in production environement, the default web server is simply not good enough. We then need to use something like GUnicorn or Apache or Nginx to actually behave as a web server which forks multiple copies of the Flask code and runs it as it recieves requests. These production web servers can usually be much faster and more performant than Flask's server.

All our buisness logic lives in these functions, we can connect to our database from here, process the data, do whatever we want.

NodeJS and Express is similar, however NodeJS is quite fast and can even be used directly as the web server.

## The frontend

The frontend, for a web app is coded in HTML+CSS+Javascript. The HTML is purely Markup, it provides only the content of the web page. CSS is decoration, it styles the web page, and the JavaScript provides code that actually has the logic. To make it easier to write code in the frontend, we have some libraries there too, for example JQuery. It is a javascript library that provides certain common functionality like adding elements, hiding things, making transitions, etc. Bootstrap is a library for CSS that makes it easier to make good looking websites. There are certain frameworks like React which make it possible to create much better frontend code, with components and a "reactive" coding style but I'll not explain that here. (we'll meet React in a later post...)

In Android, the task of HTML+CSS is taken up by XML and JavaScript code is replaced with Java. However we also have another framework for App development called _Flutter_, which is quite similar to React, and it makes it much easier to write Android code! It is cross platform too so the code will run on iOS as well. It's quite a cool thing, and we'll cover that too in a later blog post.

# Let's make an app!

First we start out with just the basic components of our app in an HTML file

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>To-Do App</title>
</head>
<body>
  <div id="main">
    <div id="heading">
      <h1>To-Do's</h1>
    </div>

    <ul>
      <li>
        <div class="todo">
          <input type="checkbox" class="todo-check">
          <span class="todo-title">
            Get Stuff Done
          </span>
        </div>
      </li>
      <li>
        <div class="todo">
          <input type="checkbox" class="todo-check">
          <span class="todo-title">
            Get Stuff Done - 2
          </span>
        </div>
      </li>
    </ul>

    <div id="new">
      <input type="text" id="new_todo">
      <button>+</button>
    </div>
  </div>
</body>
</html>
```

It looks right now like this:

<center> <img src="/assets/ss1.1.png"> </center>

We've added those div's with the id's and classes so that we can later add CSS to style those elements individually. We could have used a CSS library like bootstrap, that provides a lot of predefined classes for us to use. But there's an even simpler alternative, called [water.css](https://github.com/kognise/water.css/), that just requires us to add _one line of code_!

We'll just add this line to the `<head>`:

```html
<link rel='stylesheet' href='https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css'>
```

Now it looks like:

<center> <img src="/assets/ss1.2.png"> </center>

Much better, I think!

We can focus on the rest of the styling later on, but right now let's just get all the JQuery code to work. First add JQuery from a CDN:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
```

The JQuery library basically provides us a single function called `$` that is just magic. It can do a lot of stuff. with just this one function.

For Example:

```js
// select an HTML DOM element with id #new_todo:
$("#new_todo")
// select all elements with class todo-title and set the style to color: red
$(".todo-title").css("color", "red");
// find all div elements, create a new <p>Hello world</p> element and append it to all of them
$("div").append( $("<p>Hello world</p>") );
```

If you want to test how all this works and experiment with this a bit, open up the chrome JS console on the web page we're making with Ctrl+Shift+J and test any of this code.

<center> <img src="/assets/ss1.3.png"> </center>

Now, to make our "+" button work, we will add an `onclick` attribute to the button:

```html
<button onclick="add_todo()">+</button>
```

This will call the `add_todo()` function in javascript whenever the button is pressed. We'll now make this function in javascript that reads text from the input box and then adds it to the list.

```js
function add_todo() {
  var title = $("#new_todo").val();
  $("#todo-list").append(
    $("<li/>").append(
      $("<div class='todo'/>").append(
        $("<input type='checkbox' class='todo-check'>")
      ).append(
        $("<span class='todo-title'/>").text(title)
      )
    )
  );
  $("#new_todo").val("");
}
```

The first line uses JQuery's selector syntax to select the element with id `new_todo` and gets its value. The next lines find the `#todo_list` element, and adds a new todo element to it and sets its text equal to the value we had found earlier. Then we clear the text box with the last line.


We just test this code, and right now, it is working as intended.

<center> <img src="/assets/ss1.4.png"> </center>

If you're still confused, experiment with this in your own JS console. You can also look up JQuery's own documentation at its official website. It has a lot of helpful examples.

Now you can try implementing the rest of the features yourself, like deletion or crossing out todos. This post has gotten rather long so I'll continue with the rest of the implementation with session storage in the next post, in part 1B.

<br><br><br><br>

-------------------------------------------------------

<br><br><br><br>


[^1]: Actually, the concept of an API is more general than this, but I'm only explaining API in the context of a web API here. Check out wikipedia for the more general definition.

[^2]: You can get more info about [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) at the awesome documentation at Mozilla Developer network. Check out [HTTP data flow](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#HTTP_flow) and an [HTTP example session](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session) and [HTTP Request Methods ](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) (like GET and POST).

[^3]: A Number in JavaScript is actually always a IEEE floating point number, so there are no seperate float and int data types.

[^4]: If you know python, you can very easily start writing web apps in no time using Flask, just check out the [Flask Quickstart Guide](http://flask.pocoo.org/docs/1.0/quickstart/)!