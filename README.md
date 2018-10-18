# A Simple RESTful App Server

*Goals:* Learn the basics of HTTP and REST by building a simple API.

*Prerequisites:* Exposure to Node.js and Express, basic understanding of client-server communication.

*References:*

- [Git Documentation](https://git-scm.com/doc)
- [Introduction to the Linux Terminal](https://www.digitalocean.com/community/tutorials/an-introduction-to-the-linux-terminal)
- [MDN: Getting Started with the Web](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web)
- HTML5 Family references on iLearn Resources

## Cloning the lab repository

*In a Web browser:*

These instructions assume that you are looking at the GitHub page for this repository within a Web browser, after accepting the assignment via GitHub Classroom.

1. From the GitHub page for this repository, click on the green "Clone or Download" button to reveal the repository URL.
2. Copy this URL.

*In a BASH terminal:*

1. Clone the Lab repository by entering the following command, replacing *PATH_TO_REPO* by pasting the URL you copied above. `git clone PATH_TO_REPO Lab05`
2. Enter the newly created repository folder by *changing your working directory* (i.e., "going into the folder"). `cd Lab05`
3. List the contents of your working directory to see that it contains a README.md file (in fact, the same README that you are currently reading) as well as an HTML file. `ls -a`
4. Install our server's module dependencies (listed in _package.json_): `npm install`

## Create a simple model for working with person data

Technically speaking, we're not creating a fully fledged Model here. Rather, we are simply "modeling" (pun intended) some of the essential operations expected of a peristent data model.

Under the currently empty models folder, create a file named `person.js` that will hold our "person" data. For purposes of this Lab activity, a person is simply a String containing a person's name.

Although a real-world application would almost certainly need data persistence, for this example we will store all data in memory so that it is ephemeral (i.e., will be reset when the server is restarted). Create a list containing a few person names...

```js
let people = [
    "Alice"
  , "Bob"
  , "Carol"
]
```

In Node.js, the built-in `module.exports` variable (or simply `exports`) allows us to control what data and functions are visible to other modules that `require` this one. For this example, we will keep our `people` variable private, and instead we will only _export_ a collection of functions for operating on this list.

To that end, let's create an object whose methods are solely designed to manipulate this list of person names, and then export this object. We'll have methods for the following tasks:

- look at the entire list of names
- get the one name at a specific position
- add a new name to the list
- alter the existing name at some position

```js
module.exports = {
    list: () => people
  , find: (i) => people[i]
  , create: (name) => people.push(name) - 1
  , rename: (i, name) => people[i] = name
}
```

These methods provide most of the CRUD functionality expected of persistent data. CRUD stands for Create-Read-Update-Delete. The only thing missing right now is Delete, but we won't worry about that today.

# Using the Express Router to create a simple controller

In Express.js, the Router object allows us to create custom server routes for whatever purposes we can imagine. In this example, we will create routes that roughly correspond to the CRUD operations on our data. Then we will attach these routes to our server application.

For purposes of this exercise, we assume plain-text data. This means that the server's responses wil contain data in plain-text form _and_ the client requests will send any data or parameters in plain-text form as well.

Under the currently empty models folder, create a new file named `controller.js` that will configure the desired routes to interact with our data via the CRUD interface methods created in our model script. Begin this file by loading the required modules...

```js
// in controller.js
person = require("../models/person.js")
express = require("express")
```

Next, let's access Express' Router object...

```js
router = express.Router()
```

Now we're ready to create our first route, which will be a _Read_ operation corresponding to the `list` method of our model.

```js
router.route('/persons')
  .get((req, res) => res.send(person.list().toString()))

module.exports = router;
```

That's it! All that's left is to connect this Router to our server application. Edit the "app.js" file in the following ways:

- load the controller module
- load the "body-parser" module which will allow us to work with the request data
- attach the plain-text parser to our application
- attach the controller to our application

```js
// in app.js

parser = require("body-parser");
controller = require("./controllers/controller.js");

app.use(parser.text());
app.use("/api", controller);
```

Now we're ready to test our application! Save the file, launch the server with `node server/app.js`, and then enter the appropriate route "localhost:3000/api/persons" into your browser's address bar. If successful, then you should see the plain text "Alice,Bob,Carol" displayed in your browser. Good job!

# Filling out our Controller with more routes

We still have several additional operations to add to our application.

- add a new person name
- read a single person name
- replace an existing person's name

In order to add a new name, we will rely on the HTTP method called __POST__. Behind the scenes, the controller must get the new name from the body of the client's request message and then add this name to the in-memory list of people by calling the appropriate method of our model.

```js
  // add this immediately after the get route
  .post((req, res) => {
    res.send(person.create(req.body).toString());
  });
```

To test this, we must be able to send a __POST__ request rather than a simple __GET__ request. Do this using either the `curl` command in your shell or by using the _Postman_ application.

```
$ curl -XPOST -H "Content-type: text/plain" -d 'Dave' http://localhost:3000/api/persons
$ curl -XPOST -H "Content-type: text/plain" -d 'Erin' http://localhost:3000/api/persons
```

Note that the server response will contain the index of this newly added person name, as plain-text. If everything was successful, the responses should be "3" and "4" respectively, and you should see "Alice,Bob,Carol,Dave,Erin" when you try to load the "localhost:3000/api/persons" route again.

Next, we'll add a new route with two methods so that we can read and update a specific name at a given position in our list of people.

```js
// add this new route after the previous route
router.route('/persons/:id')
  .get((req, res) => res.send(person.find(req.params.id)))
  .put((req, res) => {
    person.rename(req.params.id, req.body);
    res.send(204);
  });
```

You can test this functionality with the following `curl` commands (or the equivalent in Postman).

```
$ curl -X GET http://localhost:3000/api/players/4
$ curl -X PUT -H "Content-type: text/plain" -d 'Eve' http://localhost:3000/api/players/4
$ curl -X GET http://localhost:3000/api/players/4
```

Test your application and make sure that the server does not crash and that the responses make sense. The first request should return the name "Erin", the second should return nothing (but should replace "Erin" with "Eve" in the people list in-memory), and the third should then return "Eve".

# Deliverables

Be sure to commit and push all your work to complete this Lab. In class tomorrow, we will discuss this Lab as well as all the concepts touched on here, as well as even more... including JavaScript Object Notation (JSON).

```
$ git add server
$ git commit -m "Completed Lab 5."
$ git push
```

## Future work

Try replicating this Lab with slightly different data, or slightly different routes. Make sure that you understand how the pieces connect and how to do this again from scratch.

