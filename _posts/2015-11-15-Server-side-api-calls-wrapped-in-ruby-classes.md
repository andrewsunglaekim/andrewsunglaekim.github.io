---
layout: post
title: Server Side Api Calls Wrapped in Ruby Classes
excerpt: "Make server side api calls, wrap that JSON in a ruby class"
---

There's a lot of information out their on the web packaged in API's. Ridiculous amounts. Using rails, we can access a large portion of that information by making server side API calls. Also, we can encapsulate objects into ruby classes to provide an interface for that data in our views.

> In this blog post, we'll be utilizing the [weather underground api](www.wunderground.com/weather/api/) to create a simple app that gets current weather and temperature when you enter in a city and state. You can fork an example of this app, [Quick Weather.ly](https://github.com/ga-dc/rails_weather_api). A general understanding of [ruby classes](http://andrewsunglaekim.github.io/OOP-There-It-Is/) and [rails](http://guides.rubyonrails.org/getting_started.html) will be needed to follow this blog.

Let start by creating a brand new rails application:

{%highlight bash%}
$ rails new weather
{% endhighlight%}

We'll also add some dependencies we'll need in our `Gemfile`:

{% highlight ruby %}
gem 'httparty'
gem 'figaro'
{% endhighlight %}

> The [httparty gem](https://github.com/jnunemaker/httparty) will allow us to make HTTP requests on our server.

> The [figaro gem](https://github.com/laserlemon/figaro) allows us to hide our tokens and keys from version control but still allow us to use them in our application. We don't want our keys public facing!

Let's install dependencies and install figaro as well.

{% highlight bash %}
$ bundle install
$ bundle exec figaro install
{% endhighlight %}

> You'll notice here that `config/application.yml` file was created here and also added to the `.gitignore` file. This file is where we will store our API key for the [weather underground api](www.wunderground.com/weather/api/) and any future keys/tokens we'll need for our application.

Let's create a simple route in our `config/routes.rb` :

{% highlight ruby %}
Rails.application.routes.draw do
  root 'weather#get_weather'
end
{% endhighlight %}

> Our site will be simple and have only one path, the root url, that gets the current weather of a city.

Since we have a route that maps to a weather controller, we need to make that now as well. Create the controller file `app/controllers/weather_controller.rb` and place in the following contents.

{% highlight ruby %}
class WeatherController < ApplicationController
  def get_weather
    if params[:city] && params[:state]
      @forecast = Forecast.new(params[:city], params[:state])
    else
      @forecast = Forecast.new("washington", "dc")
    end
  end
end
{% endhighlight %}

Ok. Wait a minute. What the heck is this `Forecast` class? I haven't defined that yet.

> At a high level though, it seems like we might be getting some parameter values to create a new one, and if we don't, it'll just default to making one for Washington, DC.

So we need to define the `Forecast` model... We'll be utilizing the `httparty` gem to make requests to an API inside of this class definition. Before we define the class, let's take a quick detour and retrieve a key for the API we'll be using. Visit [weather underground api](http://www.wunderground.com/weather/api/?MR=1) and click `Sign Up for FREE!`

After signing up and validating email, agree to term and sign in. Once there, click on pricing(don't worry its free!). Make sure you click on the stratus plan(the free one, but still great!) and then click on purchase key. Fill out the form and purchase a key. If you then click on the documentation tab you'll see a url somewhere in the middle of the page like this:

{% highlight bash %}
http://api.wunderground.com/api/<your key here>/conditions/q/CA/San_Francisco.json
{% endhighlight %}

If we visit that link we'll see something like this:

<img src="/images/weatherjson.png">

> If we look at this `JSON` object. We can see a whole bunch of useful information we may want for our application like `temp_f` and `weather`. You can also change the url with different cities and states and see all the information change. We want to encapsulate parts of this data into a ruby object.

Let's define that `Forecast` class now. Let's begin by creating a model file for forecasts. `$ touch app/models/forecast.rb` in that file, place the following code:

{% highlight ruby %}
class Forecast
  # creates getter methods for temp_f, weather, city and state.
  attr_reader :temp_f, :weather, :city, :state

  # initialize method takes 2 arguments city and state
  def initialize(city, state)

    # create the url using the city and state arguments. Also utilizing ENV
    # variable provided by figaro. Key value should be in 'config/application.yml'
    url = "http://api.wunderground.com/api/#{ENV["wunderground_api_key"]}/conditions/q/#{state.gsub(/\s/, "_")}/#{city.gsub(/\s/, "_")}.json"

    # utilizing httparty gem to make get request to the url prescribed in the
    # line above and storing the response into the variable below.
    response = HTTParty.get(url)

    # instantiating temp_f and weather by parsing through the JSON response
    @temp_f = response["current_observation"]["temp_f"]
    @weather = response["current_observation"]["weather"]

    # storing arguments as instance varibles in the model
    @city = city
    @state = state
  end
end
{% endhighlight %}

> One thing we can notice right away is that this class does not inherit from any other class. Another thing to note is that the url string is very similar if not identical to the one we entered into the browser.

Currently our model won't work because we haven't defined `ENV["wunderground_api_key"]`. We need to make sure we update our `config/application.yml` file with this information:

{% highlight yaml %}
wunderground_api_key: your_key_info_goes_here
{% endhighlight %}

> You can find your key by clicking on `Key Settings` in the weather underground API site.

Assuming you have a working key, we can now hop into the `rails console` and test our model out. We can see something like this if we instantiate a new forecast and pass in `washington` and `dc` as arguments:

<hr>

<img src="/images/forecast_rails_c.png">

This is great. Now the information in our `app/controllers/weather_controller.rb` makes a little bit more sense:

{% highlight ruby %}
class WeatherController < ApplicationController
  def get_weather
    if params[:city] && params[:state]
      @forecast = Forecast.new(params[:city], params[:state])
    else
      @forecast = Forecast.new("washington", "dc")
    end
  end
end
{% endhighlight %}

> If there are parameters for a city and state, it will create a new forecast based on that city and state. If there isn't parameter values being passed in, then it will create a default forecast of Washington, DC.

Before we test this route at, let's actually create the view that will have the form for city and state.

{% highlight bash %}
$ mkdir app/views/weather
$ touch app/views/weather/get_weather.html.erb
{% endhighlight %}

Great, let's put the form and some displays in there. In `app/views/weather/get_weather.html.erb`:

{% highlight erb %}
<form method="get" action="/">
  <label>City:</label>
  <input name="city" type="text">
  <label>State:</label>
  <input name="state" type="text">
  <input type="submit">
</form>


<p>Temperature in <%= "#{@forecast.city}, #{@forecast.state}" %> is <%= @forecast.temp_f %></p>
<p>Current Weather in <%= "#{@forecast.city}, #{@forecast.state}" %> is <%= @forecast.weather %></p>

{% endhighlight %}

Great lets fire up the rails server and test our application `$ rails s` and navigate to `http://localhost:3000`. We'll see something like this:

<img src="/images/no_params.png">
<hr>
If we enter `Miami` for the city and `Fl` for the state we'll see this sort of result:

<img src="/images/params_weather.png">

> You're results may very as this API is updated relatively frequently. Whats cool is that this data is a real time reflection of the API when the request was made. Every time we click on `Submit` we get that information from the API.

This is a very small example of how we can leverage API and ruby classes to encapsulate data from JSON responses. If you have a JSON endpoint that has data you want to access, then you can create a ruby class to encapsulate that data into objects. Limitless possibilities.
