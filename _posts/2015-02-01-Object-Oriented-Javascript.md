---
layout: post
title: A Beginner's Approach to Object Oriented Javascript
excerpt: Rudimentary OOJS
---

Initially, I had wanted to write a continuation on the Devise Gem. I'm still going to build upon the same app I started last week, however, for this blog post I'm going to talk about object oriented programming in Javascript, specifically, how it applies in the Rails Framework. A disclaimer before I get into it... I am no javascript guru. In fact, this is only my third week using the language, but I imagine putting my thoughts to paper will substantiate some of this knowledge milling around in my head. That being said, some of my syntax won't be the cleanest and perhaps slightly damp at times. Also, semicolons suck, and I will forget to put them in... a lot.

I'll be using the Ruby on Rails framework, postgresql database, javascript and jQuery. If you want to follow along and get a better understanding of where I'm starting from, read [this](http://andrewsunglaekim.github.io/page2/).The finalized code will be on github [here](https://github.com/andrewsunglaekim/reminder). 

So the overall goal will be to convert our ruby model into a JSON object so we can integrate our ruby models into our javascript. The first thing I want to do is make our Post model a little bit more robust and less boring, so we can have a better idea of what's happening when we start writing our javascript.

Once you're in the reminder repo, in the terminal write:

    $ rails g migration AddColumnsToPosts

Find the migration, it should be the most recent migration file under `db/migrate` directory and shove this content into it:

{% highlight ruby %}
class AddColumnsToPosts < ActiveRecord::Migration
  def change
    add_column :posts, :title, :string
    add_column :posts, :time_of_day, :string
  end
end
{% endhighlight %}

Next lets just store some dummy content into our seed file, so we have something to play with in JS. In the seed file `db/seeds.rb` place the following content:

{% highlight ruby %}
User.destroy_all
Post.destroy_all

users = User.create([
  {email: 'bob@email.com', password: 'password'},
  {email: 'bob1@email.com', password: 'password'}
])

users[0].posts.create([
  {title: "Breakfast", body: "Eat Breakfast", time_of_day: "morning"},
  {title: "Lunch", body: "Eat Lunch", time_of_day: "afternoon"},
  {title: "Dinner", body: "Eat Dinner", time_of_day: "evening"}
])

users[1].posts.create([
  {title: "Grocery Shopping", body: "Go buy some schtuff", time_of_day: "morning"},
  {title: "Exercise", body: "Go work out", time_of_day: "afternoon"},
  {title: "slumber", body: "crash hard in bed", time_of_day: "evening"}
])

{% endhighlight %}

And then run the migration and seed the database:

    $ rake db:migrate
    $ rake db:seed

Cool, now that we got our database setup, lets incorporate some JS!

In the terminal:

    $ touch app/assets/javascripts/post.js
    $ touch app/assets/javascripts/post_view.js

Open the first file `app/assets/javascripts/post.js` in your text editor and put the following content.

{% highlight javascript%}
// lets test to make sure our JS is working
$(document).ready(function(){
  alert("js is working");
})
{% endhighlight %}

Great, now lets fire up the rails server and open `localhost:3000`in your browser. You should see something like this:

<img src="/images/js_working.png">

Great, now we know the javascript works. Lets change up a couple of things we had in the previous version of this app. The first change will be getting rid of the nested resource for posts in `config/routes.rb`:

{% highlight ruby %}
Rails.application.routes.draw do
  root "posts#index"
  devise_for :users, :path => 'accounts' 
  resources :posts
end
{% endhighlight %}

Next let's set up our view in `app/views/posts/index.html.erb`, replace all the content with:
{% highlight erb %}
	<h1>Listing your posts</h1>

<%= form_for @post do |f| %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_field :body %>
  </p>
  <p>
    <%= f.label :time_of_day %><br>
    <%= f.text_field :time_of_day %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
<%#-- These buttons will be the focus of our javascript --%>
<button class="firstExample">First Example </button>
<button class="secondExample">Second Example </button>
<button class="thirdExample">Third Example </button> 

<%#-- We'll shove in some content here by clicking on the above buttons using javascript --%>
<div class="jsContent">
</div>

<ul>
  <% @posts.each do |post| %>
  <li><%= post.body %></li>
  <% end %>
</ul>
{% endhighlight %}

Let's go into `app/controllers/posts_controller.rb` and change the index method to the following:

{% highlight ruby %}
class PostsController < ApplicationController
  before_action :authenticate_user!
  def index
    @post = Post.new
    @posts = current_user.posts
    # the respond_to helper method allows us to access ruby @posts into json format
    respond_to do |format|
      format.html
      format.json{render json: @posts}
    end  
  end
...
# more controller actions follow	
{% endhighlight %}

Great everything's ready except for our JS. Let's go into `app/assets/javascripts/post.js` and put this content in it: (The next two blocks of code are kind of the meat and potatos of it all. They are the most pertinent to this blog)

{% highlight javascript %}
// this is our constructor function... i think
var Post = function(){
}

// the ajax call to parse our ruby into a JSON object
Post.prototype = {
  // the function to make the ajax call
  load: function(callback){
    $.ajax({
      type: 'GET',
      dataType: 'json',
      url: window.location.origin + '/posts.json',
      success: callback
    });
    // or you can use
    // $.getJSON(window.location.origin + '/posts', callback)
  }
}
{% endhighlight %}

Now finally, let's go to `app/assets/javascripts/post_view.js` and put this in it: 

{% highlight javascript %}
var PostView = function(jsonObject){
  // to make drilling in a little bit easier, instantiate some attributes
  this.model = jsonObject;
  this.firstObject = jsonObject[0]
  this.secondObject = jsonObject[1]
  this.thirdObject = jsonObject[2]
  this.secondObjectTimeOfDay = jsonObject[1].time_of_day
}

// we will define how we want our content to appear on the page. Separate our concerns and keep our views relatively modular
PostView.prototype = {
  firstExampleFunction: function(){
    $(".jsContent").append("<p>hello</p>")
    $(".jsContent").append("<p>You're first task is to " + this.firstObject.body + "</p")
  },
  secondExampleFunction: function(){
    $(".jsContent").append("<p>You should also " + this.secondObject.title  + " at some point in the " + this.secondObjectTimeOfDay + "</p>")
  },
  thirdExampleFunction: function(){
    $(".jsContent").append("<p>But save room for dinner in the " + this.thirdObject.time_of_day + "</p>")
  },
}

// making a new instance of our Post model
var p = new Post();

// now we call the load function defined in post.js. The response is the JSON object retrieved from the ajax call in the post model
p.load(function(response){
  var newPostView = new PostView(response);

  // here we'll set up our event listeners and call the functions defined in our PostView.prototype
  $(".firstExample").on("click", function(){
    newPostView.firstExampleFunction();
  })
  $(".secondExample").on("click", function(){
    newPostView.secondExampleFunction();
  })
  $(".thirdExample").on("click", function(){
    newPostView.thirdExampleFunction();
  })
})
{% endhighlight %}

After clicking each of the three buttons your screen should look something like: 

<img src="/images/js_success.png">

Great! We've just implemented some rudimentary object-oriented javascript! I encourage you to dive into `app/assets/javascripts/post_view.js` file and mess around with drilling into the object and other ways to manipulate the DOM. I hope this post helped bridge the gap between rails and object oriented javascript a little bit. For more in depth documentation, check out [The MDN Introduction to Object-Oriented Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript) 














