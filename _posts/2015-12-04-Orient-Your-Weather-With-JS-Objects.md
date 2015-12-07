---
layout: post
title: Orient Your Weather with JS Objects
excerpt: "Create a client side weather app utilizing the Weatherunderground API that implements an OOJS stucture"
---

When using AJAX to make client side calls to an API, we often just make the call and just string some jQuery soup along to manipulate the DOM in some way. Other times we use a front end framework to render views from the API and just trust that it'll "just work". What we're going to be doing in this post, is hand rolling an object oriented approach to accessing data and rendering views from an API. The API we'll be using is the [Weather Underground API](http://www.wunderground.com/weather/api/). I really like this API for teaching new developers because the endpoints are really simple and easy to use.

> A general understanding of AJAX and javascript would be recommended for this blog post. In this post, we'll be doing read only from a 3rd party API. Getting information from an API, converting it into a model in our application and then rendering a view with that model. It will be a client side only application.

Visit [weather underground api](http://www.wunderground.com/weather/api/?MR=1) and click `Sign Up for FREE!`

After signing up and validating email, agree to terms and sign in. Once there, click on pricing(don't worry its free!). Make sure you click on the stratus plan(the free one, but still great!) and then click on purchase key. Fill out the form and purchase a key. If you then click on the documentation tab you'll see a url somewhere in the middle of the page like this:

{% highlight bash %}
http://api.wunderground.com/api/<your key here>/conditions/q/CA/San_Francisco.json
{% endhighlight %}

> Make sure you hold onto this key, we'll need it later

If we visit that link we'll see something like this:

<img src="/images/weatherjson.png">

> If we look at this `JSON` object. We can see a whole bunch of useful information we may want for our application like `temp_f` and `weather`. You can also change the url with different cities and states and see all the information change. We want to encapsulate parts of this data into a JS object.

Let's start building out our application now.

### Folders and Files
In the terminal:

{% highlight bash %}
$ mkdir wunderground_oojs
$ cd wunderground_oojs
$ mkdir models
$ mkdir views
$ mkdir js
$ touch index.html
$ touch js/script.js
$ touch js/models/forecast.js
$ touch js/views/forecast.js
{% endhighlight %}

### HTML Skeleton and Dependencies
Let's first fill out the skeleton of our app. In `index.html`:

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Weather.ly</title>
  <!-- load jquery dependency and all other script files in this app -->
  <script src="https://code.jquery.com/jquery-2.1.4.js"></script>
  <script src="js/models/forecast.js"></script>
  <script src="js/views/forecast.js"></script>
  <script src="js/script.js"></script>
</head>
<body>
  <!-- form containing 2 inputs to get a city and state, as well as submit button -->
  <form class="search">
    <input class="searchCity" type="text" name ="city" placeholder="City">
    <input class="searchState" type="text" name ="state" placeholder="State">
    <input type="submit" value="Get Weather">
  </form>
  <!-- empty div that will contain our forecast -->
  <div class="forecast">
  </div>
</body>
</html>
{% endhighlight %}

### Forecast Model
Now that our skeleton's all set up, let's define our `Forecast` model. In `js/models/forecast.js`:

{% highlight javascript %}
var Forecast = function(info){
  this.city = info.city;
  this.state = info.state;
}
{% endhighlight %}

> This is a basic constructor function that accepts an object as an argument to populate some attributes for our `Forecast` objects. We'll be supply these attributes by just filling out the form from our HTML and using jQuery to grab the values out of the form.

Well this is great, but wasn't the whole point of this post to access data from a third party API? It is! Let's extend the functionality of our `Forecast` constructor. In `js/models/forecast.js`:

{% highlight javascript %}
var Forecast = function(info){
  this.city = info.city;
  this.state = info.state;
}

Forecast.prototype.loadForecast = function(){
  var self = this
  // url to hit the API endpoint, make sure to put your key in!
  var url ="http://api.wunderground.com/api/yourKeyInfoGoesHere/conditions/q/" + this.state + "/" + this.city + ".json"
  // makes the AJAX get request using $.getJSON() passing in the URL above
  var request = $.getJSON(url)
  // in this promise `.then` we'll set some attributes for a Forecast by drilling
  //  through the response we get back from the API
  .then(function(response){
    self.tempF = response.current_observation.temp_f
    self.iconUrl = response.current_observation.icon_url
    self.description = response.current_observation.weather
  })
  // finally we return the request because we need to be able to continue to
  // chain promises on the return of this function
  return request
}
{% endhighlight %}

> Here we're extending the functionality of our `Forecast` model to include the `loadForecast()` function. When we instantiate a new Forecast, we can call `.loadForecast()` on it to populate some more attributes from the API call.

### Forecast View

Now we need an interface that will take instances of the model we've just defined to render a view. We'll do this by creating another constructor function that abstracts all of the view functionality/interface to a JS object. In `views/forecast.js`:

{% highlight javascript %}
var ForecastView = function(forecast){
  // set a Forecast model as a property of the view
  this.forecast = forecast;
  // sets the div with class forecast as the domain of this view
  this.$el = $(".forecast");
}

ForecastView.prototype = {
  // DOM manipulation using a Forecast
  forecastTemplate: function(forecast){
    var html = $("<div></div>");
    html.append("<h2>Weather For " + forecast.city + ", " + forecast.state + "</h2>");
    html.append("<img class='artist-photo' src='" + forecast.iconUrl + "'>");
    html.append("<p class='description'>" + forecast.description + "</p>")
    html.append("<p class='tempF'>Current Temp in Fahrenheit: " + forecast.tempF + "°</p>")
    return(html);
  },
  // function that changes the HTML of the $el(div with class forecast) to what was generated by the template
  render: function(){
    var self = this;
    self.$el.html(self.forecastTemplate(self.forecast).html());
  },
  // emptys all elements withing the $el (div with class forecast)
  clearContainer: function(){
    this.$el.empty()
  }
}
{% endhighlight %}

### Tying it all together

Sweet! So basically, if we have a city and state, we can render a view based off the information we get from the Weatherunderground API. Well, it just so happens we have 2 inputs for a city and state in our `index.html`. We just need to grab the values from those input fields, create a new `Forecast` object then render the corresponding view. In `js/script.js`:

{% highlight javascript %}
$(document).ready(function(){
  // sets a submit event to the element with class search(in our case the form)
  $(".search").on("submit", function(e){
    // prevents the default action, the default action here will cause a page refresh, which we don't want
    e.preventDefault()
    // grabs values from the the input fields and stores them to a variable
    var city = $(".searchCity").val()
    var state = $(".searchState").val()
    // creates a new forecast object using our Forecast constructor we define earlier.
    forecast = new Forecast({city: city, state: state})
    // calls .loadForecast to make the API call
    forecast.loadForecast().then(function(){
      // in the promise we create a new view passing in the forecast object
      view = new ForecastView(forecast)
      // empties forecast if one already exists
      view.clearContainer()
      // renders the new forecast
      view.render()
    })
  })
})

{% endhighlight %}

If we've done everything correctly to this point, we should be able to fill in a City and State and see something like this:

<hr>

<img src="/images/weather.png">

Wonderful! We have a nice little weather app we can run any time to get the current weather, how useful!

> This is just a small example of how you could parse an API call into a JS object. In this app, if we weren't going to change it at all going forward, jQuery soup is probably a better approach just because it'll be much quicker to code. OOJS separates your concerns and makes it more modular. If I want to add another property, I just add it to the model definition. Let's also imagine this is a view that needs to be generated in more than one place. Now we won't have to duplicate any code we can just use our constructors to execute that functionality.
