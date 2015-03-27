---
layout: post
title: Get Ready! Get Set! Ruby!
excerpt: Getter and Setter methods in Ruby Classes
---

So we need a way to have programmatic access to our instance variables within a ruby class structure. If you want to learn a little more about ruby classes before learning about getter and setter methods read [this](http://andrewsunglaekim.github.io/OOP-There-It-Is/)

Let's create a ruby class in a new file we'll call `person.rb`:

{% highlight ruby %}
class Person
  def initialize(name, age, location)
    @name = name
    @age = age
    @location = location
  end
end

bob = Person.new("bob", 25, "DC")
puts bob.name
{% endhighlight %}

Great, so we establish a Person class that has the instance variables `@name`, `@age` and `@location`. We also created a new Person object and stored it into the variable bob. So bob has all these great instance variables, but as soon as we run this file we get an error like this one:

{% highlight bash %}
$ ruby person.rb
person.rb:10:in `<main>': undefined method `name' for #<Person:0x007ff5c3127760 @name="bob", @age=25, @location="DC"> (NoMethodError)
{% endhighlight %}

So, even though we initialize our Person objects with some instance variables we don't have access to them. Let's create access to them now through getter methods in `person.rb`:

{% highlight ruby %}
class Person
  def initialize(name, age, location)
    @name = name
    @age = age
    @location = location
  end

  # getter methods for name, age and location below
  def name
    @name
  end

  def age
    @age
  end

  def location
    @location
  end
end

bob = Person.new("bob", 25, "DC")
puts bob.name
puts bob.age
puts bob.location
bob.location="NY"
puts bob.location
{% endhighlight %}

Awesome, now we have read access to our instance variables `@name`, `@age` and `@location`. But we can't set or change the instance variables. If we run our program `person.rb`, we'll see this error in our terminal:

{% highlight bash %}
$ ruby person.rb
bob
25
DC
person.rb:26:in `<main>': undefined method `location=' for #<Person:0x007fcaba08f098 @name="bob", @age=25, @location="DC"> (NoMethodError)
{% endhighlight %}

Lets fix this by adding the ability to change or set instance variables. But it doesn't make sense to be able to change certain attributes. Maybe we'll have a birthday method that increments age, but we shouldn't be able to just change your age to whatever you want. You should however be able to change a person's name or location in case they do change. So let's add some setter methods for `@name` and `@location`. I'll throw in a `celebrate_birthday` method as well that will allow us to manipulate the age for kicks. So in our `person.rb`:

{% highlight ruby %}
class Person
  def initialize(name, age, location)
    @name = name
    @age = age
    @location = location
  end

  # getter methods for name, age and location below
  def name
    @name
  end

  def age
    @age
  end

  def location
    @location
  end

  # setter methods for name and location below
  def name=(name)
    @name = name
  end

  def location=(location)
    @location = location
  end

  # person has a birthday method
  def celebrate_birthday
    @age += 1
  end
end

bob = Person.new("bob", 25, "DC")
puts bob.name
puts bob.age
puts bob.location
bob.name="bob the second"
bob.location="NY"
bob.celebrate_birthday
puts bob.name
puts bob.age
puts bob.location
{% endhighlight %}

So if we run our `person.rb` file, we should get something like this as output:
{% highlight bash %}
$ ruby test.rb
bob
25
DC
bob the second
26
NY
{% endhighlight %}

Great! so now we have getter and setter methods for everything we want. It turns out, ruby developers noticed that they were writing this type of code over and over again so they decided to write some helper methods to not have to write out all of this code every time they wanted getter and setter methods. I'm going to comment out the original methods and write the helper methods directly above. Let's write them now in `person.rb`:

{% highlight ruby %}
class Person
  def initialize(name, age, location)
    @name = name
    @age = age
    @location = location
  end

  # getter methods for name, age and location below
  attr_reader :name
  # def name
  #   @name
  # end

  attr_reader :age
  # def age
  #   @age
  # end

  attr_reader :location
  # def location
  #   @location
  # end

  # setter methods for name and location below
  attr_writer :name
  # def name=(name)
  #   @name = name
  # end

  attr_writer :location
  # def location=(location)
  #   @location = location
  # end

  # person has a birthday method
  def celebrate_birthday
    @age += 1
  end
end

bob = Person.new("bob", 25, "DC")
puts bob.name
puts bob.age
puts bob.location
bob.name="bob the second"
bob.location="NY"
bob.celebrate_birthday
puts bob.name
puts bob.age
puts bob.location
{% endhighlight %}

If you run `person.rb` again, you'll notice the same output. It turns out that often times, instance variables will want both getter and setter methods so there is another helper method `attr_accessor` So the finalized version of our code in `person.rb` will look like this:

{% highlight ruby %}
class Person
  # getter and setter methods for name and location
  attr_accessor :name, :location

  # getter method for age
  attr_reader :age

  # initialize method when new Person object gets instantiated
  def initialize(name, age, location)
    @name = name
    @age = age
    @location = location
  end

  # still need this method for changing age
  def celebrate_birthday
    @age += 1
  end
end

bob = Person.new("bob", 25, "DC")
puts bob.name
puts bob.age
puts bob.location
bob.name="bob the second"
bob.location="NY"
bob.celebrate_birthday
puts bob.name
puts bob.age
puts bob.location
{% endhighlight %}

Great! Now we know all about `attr_reader`, `attr_writer`, and `attr_accessor` with respect to getter and setter methods!
