---
layout: post
title: A Not So MEAN Reminder...app
excerpt: "Create a Todo app using express mongo and node"
---

We're going to be creating a 2 model CRUD application utilizing [nodeJS](https://nodejs.org/en/), [expressJS](http://expressjs.com/en/index.html), [mongoDB](https://docs.mongodb.org/manual/?_ga=1.128034762.792441474.1449068883), and [mongoose](http://mongoosejs.com/). When I was first learning about mongo/mongoose and how it plays with express, all the resources out there used a lot of generators and there wasn't really a good resource for hand-rolling a CRUD application with express and mongoose. So I took what I knew about expressJS and scoured the mongoose documentation to make the application we'll be building today.

> Though experience in these technologies is not required for this blogpost, it is recommended to have a general understanding of express and document based noSQL databases. However! If you were to just copy and paste all this code in the right places, you will still have a functional application with no knowledge of either. You just need to make sure you have npm and mongo installed through homebrew. You might just learn something in that process =D. Another thing I'd like to mention: The folder and file structure for this application is one that I structured myself and by no means consider it as any kind of standardized way of organizing code for the MEAN stack. If you have any suggestions, please submit an issue to this [repo](https://github.com/andrewsunglaekim/reminders_mongo)

The domain for the app that we'll be building has 2 models, an `Author` has many `Reminder`'s. In mongo, we'll have a collection of `Author` documents with `Reminder` sub documents. You can get all of the code for this application here in this [repo](https://github.com/andrewsunglaekim/reminders_mongo). Here's an example of what an `Author` document might look like in our Mongo DB:

{% highlight javascript %}
{
  _id: someObjectId
  name: "Bob",
  reminders: [
    {_id: someObjectId, body: "Go grocery shopping"},
    {_id: someObjectId, body: "eat chips"},
    {_id: someObjectId, body: "learn mongoose"},
  ]
}
{% endhighlight %}
<hr>

I want to take a minute to talk about the bigger component parts of this application. Node, express, mongo and mongoose. Node is just a runtime environment that allows us to use javascript for server side applications. Express is a web framework built on node. Mongo is the database we'll be using in so far as this is what mongoose connects express to. We won't be writing any actual Mongo code. It is a document based non-relational database. Mongoose is the ORM we'll be using to communicate with our mongo database. All of these technologies are a part of the super popular MEAN stack. The only part of the MEAN stack we won't be using today is AngularJS, a very popular front end framework. The application we'll be building out today will be a server-side rendered web app.

<hr>

### Dependencies, Folders, and Files
We'll be creating everything from scratch with no generators. Minus a couple of dependencies we'll be handrolling most everything. Let's start by installing dependencies. In the terminal:

{% highlight bash %}
$ mkdir reminders_mongo
$ cd reminders_mongo
$ npm init
$ npm install express --save
$ npm install hbs --save
$ npm install body-parser --save
$ npm install method-override --save
$ npm install mongoose --save
{% endhighlight %}

> `--save` updates our package.json file to include our dependencies

The 5 dependencies we'll be using for this app are:

>  1. `express` - web framework
  2. `hbs` - handlebars view engine we'll be using
  3. `body-parser` - allows us to get parameter values from forms and JSON
  4. `method-override` - allows us to do put/delete requests in our hbs views
  5. `mongoose` - our Mongo ORM

Now let's create all of the folders and files we'll need for this app. In the terminal:

{% highlight bash %}
$ mkdir controllers
$ touch controllers/authorsController.js
$ mkdir db
$ touch db/schema.js
$ touch db/seeds.js
$ mkdir models
$ touch models/author.js
$ touch models/reminder.js
$ mkdir views
$ mkdir views/authors
$ touch views/layout.hbs
$ touch views/authors/index.hbs
$ touch views/authors/show.hbs
$ touch views/authors/new.hbs
$ touch views/authors/edit.hbs
$ touch index.js
{% endhighlight %}

> this is a large amount of files, it can seem a little daunting at first, but we'll flesh all of this out throughout the post.

Before we get to code in all these files, I want to mention that the comments in the code will sometimes be more informative than the blurbs between the code.

### Schema and Models

The first thing we should do is define the schema for our database. In `db/schema.js`:

{% highlight javascript %}
// requiring mongoose dependency
var mongoose = require('mongoose')

// instantiate a name space for our Schema constructor defined by mongoose.
var Schema = mongoose.Schema,
    ObjectId = Schema.ObjectId

// defining schema for reminders
var ReminderSchema = new Schema({
  body: String
})

// defining schema for authors.
var AuthorSchema = new Schema({
  name: String,
  reminders: [ReminderSchema]
})

// setting models in mongoose utilizing schemas defined above, we'll be using
// these frequently throughout our app
mongoose.model("Author", AuthorSchema)
mongoose.model("Reminder", ReminderSchema)
{% endhighlight %}

We've defined the schema and set some models to our mongoose interface. Now let's go ahead and abstract the model interface into 2 model files.

<hr>

In `models/author.js`:

{% highlight javascript %}
require("../db/schema")
var mongoose = require('mongoose')

var AuthorModel = mongoose.model("Author")
module.exports = AuthorModel
{% endhighlight %}

In `models/reminder.js`:

{% highlight javascript %}
require("../db/schema")
var mongoose = require('mongoose')

var ReminderModel = mongoose.model("Reminder")
module.exports = ReminderModel
{% endhighlight %}

> In these files we're exporting the functionality of these mongoose models so that the file can act as the model definition. With these model definitions, we now have an interface that maps documents in our mongoDB to objects in our application.

### Seeds

With this interface, let's create our seed file to seed our database with some dummy content so we have documents to play with in our database. In `db/seeds.js`:

{% highlight javascript %}
// requires mongoose dependencies
var mongoose = require('mongoose')
// connects us to the reminders database in mongo
var conn = mongoose.connect('mongodb://localhost/reminders')
// require our model definitions we defined earlier
var AuthorModel = require("../models/author")
var ReminderModel = require("../models/reminder")
// removes any existing authors and reminders from our database
AuthorModel.remove({}, function(err){
})
ReminderModel.remove({}, function(err){
})

// instantiates 3 authors and 6 reminders in memory(but not saved yet) and
// shoves them into arrays
var bob = new AuthorModel({name: "bob"})
var susy = new AuthorModel({name: "charlie"})
var tom = new AuthorModel({name: "tom"})

var reminder1 = new ReminderModel({body: "reminder1!!"})
var reminder2 = new ReminderModel({body: "reminder2!!"})
var reminder3 = new ReminderModel({body: "reminder3!!"})
var reminder4 = new ReminderModel({body: "reminder4!!"})
var reminder5 = new ReminderModel({body: "reminder5!!"})
var reminder6 = new ReminderModel({body: "reminder6!!"})

var authors = [bob, susy, tom]
var reminders = [reminder1, reminder2, reminder3, reminder4, reminder5, reminder6]

// iterate through the authors to save them to the database after 2 reminders
// have been added as subdocuments to the author
for(var i = 0; i < authors.length; i++){
  authors[i].reminders.push(reminders[i], reminders[i+3])
  authors[i].save(function(err){
    if (err){
      console.log(err)
    }else {
      console.log("author was saved")
    }
  })
{% endhighlight %}

In order to run this file and seed our database we run this command:

{% highlight bash %}
$ node db/seeds.js
{% endhighlight %}

If we see 3 `"author was saved"`, our database was seeded with data! It hangs after executing(this seems like an issue that needs resolved), but you can exit this instance of code execution by hitting `ctrl + c`.

### Main application file - `index.js`

Now that we've seeded our database, let's fill in our applications main executable `index.js`:

{% highlight javascript %}
// express dependency for our application
var express = require('express')
// loads mongoose dependency
var mongoose = require('mongoose')
// loads dependency for middleware for paramters
var bodyParser = require('body-parser')
// loads dependency that allows put and delete where not supported in html
var methodOverride = require('method-override')
// loads module containing all authors controller actions. not defined yet...
var authorsController = require("./controllers/authorsController")
// connect mongoose interfaces to reminders mongo db
mongoose.connect('mongodb://localhost/reminders')
// invokes express dependency and sets namespace to app
var app = express()
// sets view engine to handlebars
app.set("view engine", "hbs")
// allows for parameters in JSON and html
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended:true}))
// allows for put/delete request in html form
app.use(methodOverride('_method'))
// connects assets like stylesheets
app.use(express.static(__dirname + '/public'))

// app server located on port 4000
app.listen(4000, function(){
  console.log("app listening on port 4000")
})

// routes for all requests to this express app that map to an action/function
// in our authorsController
app.get("/authors", authorsController.index)
{% endhighlight %}

> We'll be adding other routes to this as the post goes on, but the dependencies will remain the same.

We haven't defined an `authorsController` yet, let alone an index method residing in that controller. Let's fix that now. In `controllers/authorsController.js`:

{% highlight javascript %}
// requires our model definitions
var AuthorModel = require("../models/author")
var ReminderModel = require("../models/reminder")

// instantiates an authorsController which will contain all of our controller actions
var authorsController = {
  // the index action will make a DB query to find all author documents in our
  // authors collection, when it does it will render the authors/index view and
  // pass the author objects to the template
  index: function(req, res){
    AuthorModel.find({}, function(err, docs){
      res.render("authors/index", {authors: docs})
    })
  }
}

// exports the controller so we can use the file as the controller.
// re: index.js: var authorsController = require("./controllers/authorsController")
module.exports = authorsController;
{% endhighlight %}

Let's go ahead and set what the layout view of our application will be. In `views/layout.hbs`:

{% highlight html %}
<!doctype html>
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="/css/styles.css">
  </head>
  <body>
    <header>
      <h1><a href="/authors">Reminder.ly?</a></h1>
    </header>
    {% raw %}{{{body}}}{% endraw %}
  </body>
</html>
{% endhighlight %}

In the index action we're rendering the `authors/index` view. So let's update that file now. In `views/authors/index.hbs`:

{% highlight html %}
<a href="/authors/new">New Author</a>
<!-- Iterates through each author -->
{% raw %}{{#each authors}}
  <div class="author">
    <div class="authorName"><a href="/authors/{{_id}}">{{name}}</a></div>
    <!-- For each author, it will iterate through their reminders -->
    {{#each reminders}}
      <p class="reminder">{{body}}</p>
    {{/each}}
  </div>
{{/each}}{% endraw %}
{% endhighlight %}

Let's start up our application by running the following in the terminal:

{% highlight bash %}
$ nodemon index.js
{% endhighlight %}

> if you don't have nodemon installed you should! `npm install nodemon --g`. This will allow you to not restart your server every time you make changes to your file.

Now, if you visit `http://localhost:4000/authors` you should see something like this:

<img src="/images/mongoindex.png">

### Author's show and create

Great, we have a an index view for all our authors and we can see all of their reminders! If we click on either the `New Author` button or any author, we'll get an error `cannot GET`. Let's update our main application file to include those routes now. In `index.js`:

{% highlight javascript %}
app.get("/authors/new", authorsController.new)
app.get("/authors/:id", authorsController.show)
{% endhighlight %}

These controller actions(functions) don't exist yet in our code base. Let's add them now. In `controllers/authorsController.js`:

{% highlight javascript %}
var authorsController = {
  // already defined this one above,
  index: function(req, res){
    AuthorModel.find({}, function(err, docs){
      res.render("authors/index", {authors: docs})
    })
  },
  // in this action we'll just be rendering a view and don't need to query the database for anything
  new: function(req, res){
    res.render("authors/new")
  },
  // the show action we'll make a DB query to find a single author document by ID
  // in our authors collection, when it does it will render the authors/new
  // view and pass the author object to the template.
  show: function(req, res){
    AuthorModel.findById(req.params.id, function(err, doc){
      res.render("authors/show", {author: doc})
    })

  }
// more code follows
{% endhighlight %}

Great, now that we have these controller actions, let's now update the views to render some relevant information. In `views/authors/new.hbs`:

{% highlight html %}
<!-- Creates a basic form to create an author -->
<h2>Create new Author</h2>
<form action="/authors" method="post">
<label>Name:</label>
<input type="text" name="name">
<input type="submit">
</form>
{% endhighlight %}

In `views/authors/show.hbs`:

{% highlight html %}
{% raw %}
<h2>Reminders for {{author.name}}</h2>
<div>

  <!-- iterates through each reminder that belong to this author -->
  {{#each author.reminders}}
    <div class="reminders-show-each">
      <p>{{body}}</p>
      <!-- form to delete a reminder from this author (route and controller action not defined yet)-->
      <form method="post" action="/authors/{{../author._id}}/reminders/{{_id}}?_method=delete">
          <input type="submit" value="Done!">
      </form>
    </div>
  {{/each}}
</div>

<div id="reminder-new">
  <h3>New Reminder</h3>
  <!-- form to create a reminder from this author (route and controller action not defined yet)-->
  <form action="/authors/{{author._id}}/reminders" method="post">
    <label>Body</label>
    <input type="text" name="body">
    <input type="submit">
  </form>
</div>


<div id="author-edit">
  <!-- link to edit this author (route and controller action not defined yet)-->
  <a href="/authors/{{author._id}}/edit">Edit Author</a>
  <a href="/authors">Back</a>
</div>
{% endraw %}
{% endhighlight %}

> We used `{%raw%}{{../author._id}}{%endraw%}` to reference the parent object's id. IE the author's. Because we were in a loop for each reminder, `{% raw %}{{_id}}{% endraw %}` would be referring to the reminder's id. You'll also notice a bit of weirdness in the action of our delete request. `?_method=delete`. Because `PUT`/`DELETE` requests aren't native to HTML forms, we have to pass in the request as a query string. And with the `method-override` dependency, we're able to do a `DELETE` request. We won't be defining reminder CRUD til the latter sections in this post.

### Updating/Deleting authors

Before we get to creating and destroying reminder's for our authors, let's finish up CRUD functionality on our author. In the author's show page we had a link `<a href="/authors/{% raw %}{{author._id}}{%endraw%}/edit">Edit Author</a>` to an edit page for an author. Lets define that route, controller action, and view now.

In `index.js`:

{%highlight javascript%}
app.get("/authors/:id/edit", authorsController.edit)
{%endhighlight%}

In `controllers/authorsController.js`:

{%highlight javascript%}
edit: function(req,res){
  // the edit action will make a DB query to find an author documents by ID in our
  // author's collection, when it does it will render the authors/edit view and
  // pass the author object to the template
  AuthorModel.findById(req.params.id, function(err, doc){
    res.render("authors/edit", {author: doc})
  })
}
{%endhighlight%}

> when defining these controller actions, just place them in addition to the actions that already exist inside of our `controller/authorsController.js` Be careful for commas!

In `views/authors/edit.hbs`:

{% highlight html %}
{% raw %}
<!-- form to update author -->
<h2>Edit {{author.name}}</h2>
<form action="/authors/{{author._id}}?_method=put" method="post">
<label>Name:</label>
<input type="text" name="name" value="{{author.name}}">
<input type="submit">
</form>

<form method="post" action="/authors/{{../author._id}}?_method=delete">
    <input type="submit" value="Delete Author!!">
</form>
{%endraw%}
{% endhighlight %}

We've define the functionality to render the edit form. But currently the form for both edit and delete doesn't do anything. Lets fix that now by updating our application file to include the route. In `index.js`:

{%highlight javascript%}
app.put("/authors/:id", authorsController.update)
app.delete("/authors/:id", authorsController.delete)
{%endhighlight%}

Now our are app can respond to these put and delete requests. Unfortunately because we haven't defined the callbacks for these requests they'll probably error out. Let's define them now. In `controllers/authorsController.js`:

{% highlight javascript %}
update: function(req,res){
  // the update action will make a DB query to find an author document by ID in our
  // author's collection, when it does it will set the name of the author to the
  // value specified in the form. If it saves without error, it will redirect to the author's show page
  AuthorModel.findById(req.params.id, function(err, docs){
    docs.name = req.body.name
    docs.save(function(err){
      if(!err){
        res.redirect("/authors/" + req.params.id)
      }
    })
  })
},
delete: function(req, res){
  // the delete action will remove an author documents by ID. If there's no error
  // it will redirect to the author's index page.
  AuthorModel.remove({_id: req.params.id}, function(err){
    if(!err){
      res.redirect("/authors")
    }
  })
}
{%endhighlight%}

Great! We got full CRUD functionality for our authors now. This is great, but the point of the app is ... reminders. We don't have the ability to create them or delete them yet. If we look at `views/authors/show.hbs`, we can see a couple of forms, one that deletes reminders and one that creates them. Let's define the routes and controller actions for those forms now. In `index.js`:

{% highlight javascript %}
app.post("/authors/:id/reminders", authorsController.addReminder)
app.delete("/authors/:authorId/reminders/:id", authorsController.removeReminder)
{% endhighlight %}

In `controllers/authorsController.js`:

{% highlight javascript %}
addReminder: function(req, res){
  // the addReminder action will make a DB query to find an author document by ID in our
  // author's collection, when it does it will create a new reminder and push it
  // to the reminders subdocuments. If it saves without error, it will redirect to the author's show page  
  AuthorModel.findById(req.params.id, function(err, docs){
    docs.reminders.push(new ReminderModel({body: req.body.body}))
    docs.save(function(err){
      if(!err){
        res.redirect("/authors/" + req.params.id)
      }
    })
  })
},
// this is an alternate syntax, this action will find and author by it's id and,
// update it based on the object passed in as the second argument. In this case,
// the object being passed in pulls a reminder based on the id from the url and
// removes it from the subdocuments. In the callback, if there's no errors, it
// will redirect to the author's show page.
removeReminder: function(req, res){
  AuthorModel.findByIdAndUpdate(req.params.authorId, {
    $pull:{
      reminders: {_id: req.params.id}
    }
  }, function(err, docs){
    if(!err){
      res.redirect("/authors/" + req.params.authorId)
    }
  })
}
{% endhighlight %}

Great! That's the whole app. We now have the ability to create and delete reminders! Because we pieced a lot of this application incrementally. I want to consolidate some of this code, the routes in `index.js` and the actions in `controllers/authorsController.js`

#### Routes

{% highlight javascript %}
app.get("/authors", authorsController.index)
app.get("/authors/new", authorsController.new)
app.post("/authors", authorsController.create)
app.get("/authors/:id", authorsController.show)
app.get("/authors/:id/edit", authorsController.edit)
app.put("/authors/:id", authorsController.update)
app.delete("/authors/:id", authorsController.delete)
app.post("/authors/:id/reminders", authorsController.addReminder)
app.delete("/authors/:authorId/reminders/:id", authorsController.removeReminder)
{% endhighlight %}

#### Controller

{% highlight javascript %}
var authorsController = {
  index: function(req, res){
    AuthorModel.find({}, function(err, docs){
      res.render("authors/index", {authors: docs})
    })
  },
  new: function(req, res){
    res.render("authors/new")
  },
  create: function(req, res){
    var author = new AuthorModel({name: req.body.name})
    author.save(function(err){
      if (!err){
        res.redirect("authors")
      }
    })
  },
  show: function(req, res){
    AuthorModel.findById(req.params.id, function(err, doc){
      res.render("authors/show", {author: doc})
    })

  },
  edit: function(req,res){
    AuthorModel.findById(req.params.id, function(err, doc){
      res.render("authors/edit", {author: doc})
    })
  },
  update: function(req,res){
    AuthorModel.findById(req.params.id, function(err, docs){
      docs.name = req.body.name
      docs.save(function(err){
        if(!err){
          res.redirect("/authors/" + req.params.id)
        }
      })
    })
  },
  delete: function(req, res){
    AuthorModel.remove({_id: req.params.id}, function(err){
      if(!err){
        res.redirect("/authors")
      }
    })
  },
  addReminder: function(req, res){
    AuthorModel.findById(req.params.id, function(err, docs){
      docs.reminders.push(new ReminderModel({body: req.body.body}))
      docs.save(function(err){
        if(!err){
          res.redirect("/authors/" + req.params.id)
        }
      })
    })
  },
  removeReminder: function(req, res){
    AuthorModel.findByIdAndUpdate(req.params.authorId, {
      $pull:{
        reminders: {_id: req.params.id}
      }
    }, function(err, docs){
      if(!err){
        res.redirect("/authors/" + req.params.authorId)
      }
    })
  }
}
{% endhighlight %}

If you want some additional practice, add the ability to edit/update reminders. Add a third model! Play with it and break things, see what sorts of things you can do with mongoose and express.

---

Mongoose plays really well with JSON on express. Instead of doing `res.render(template, docs)` in your controller actions. Try out `res.json(docs)` and see the JSON response in the browser. Good job, you just made an api endpoint.
