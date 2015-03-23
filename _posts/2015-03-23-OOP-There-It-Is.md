---
layout: post
title: OOP There it is(ruby)
excerpt: Basic Ruby OOP
---

I've recently taken a job as a junior instructor for the web development immersive at General Assembly. So I get to rehash some of the concepts I had just learned while taking the class. For this post, you should have RVM and ruby installed. Follow this link [here](https://www.ruby-lang.org/en/documentation/installation/) for Ruby and [here](https://rvm.io/rvm/install) for RVM. My co-worker [Randy](https://twitter.com/rmlatz) came up with the title, I'm not that creative.

Let's talk about Object Oriented Programming. What Makes it so useful?

- Allows you to keep your code DRY
- Modularizes your code
- Separates your concerns

In ruby, everything is an object. Whether its a simple string, integer or any other data type. What if real world things we want to emulate in code can't exist within the constraints of these data types. Well, why don't we just create our own types of objects? Enter defining your own ruby class.

Lets make our own ruby class! Let's start by creating a file in the terminal:

{% highlight bash %}
$ touch person.rb
{% endhighlight %}

Lets place the following in the file `person.rb`:

{% highlight ruby %}
class Person
  def initialize
    @name = "bob"
  end
end

bob = Person.new
puts bob
{% endhighlight %}

Now, let's run the program in the terminal:

{% highlight bash %}
$ ruby person.rb
{% endhighlight %}

You should see something like this outputted:

{% highlight bash %}
#<Person:0x007fdd3b157cd0>
{% endhighlight %}

Great! We've created a class, and created an instance of that class. There's an instance variable @name so that all Person objects will have an attribute of name, bob. But it can't really do anything.. Lets add some stuff to that same file:

{% highlight ruby %}
class Person
  def initialize
    @name = "bob"
  end

  def name
    @name
  end

  def say_hello
    puts "#{@name} says hello!"
  end
end

bob = Person.new
sam = Person.new
puts bob.name
puts sam.name
bob.say_hello
sam.say_hello
{% endhighlight %}

The output should be:
{% highlight bash%}
bob
bob
bob says hello!
bob says hello!
{% endhighlight %}

Well, we have access to the instance variable now. And now instances of the Person classes can now say_hello. Hmmm... well it's not very useful that every instance of the Person object is named bob. Lets make it so that when we create a new instance of the Person class we can pass in a name:

{% highlight ruby %}
class Person
  def initialize(name)
    @name = name
  end

  def name
    @name
  end

  def say_hello
    puts "#{@name} says hello!"
  end
end

bob = Person.new("bob")
sam = Person.new("sam")
bob.say_hello
sam.say_hello
{% endhighlight %}

The output should be:
{% highlight bash%}
bob says hello!
sam says hello!
{% endhighlight %}

Awesome! now we can create a Person with a name. Lets flesh this program out a bit:

{% highlight ruby %}
class Person

  # getter methods functionally equivalent to the name method
  attr_reader :age, :sex, :location

  # initialize method for when an object of class Person gets instantiated
  def initialize(name, age, sex, location)
    @name = name
    @age = age
    @sex = sex
    @location = location
  end

  # getter method
  def name
    @name
  end

  # the say_hello method is just a method to show using our instance variables
  def say_hello
    puts "Hi! My name is #{@name}. Im a #{@age} year old #{@sex} from #{@location}"
  end
end

bob = Person.new("bob", 52, "male", "DC")
erica = Person.new("erica", 25, "female", "NY")
bob.say_hello
erica.say_hello
{% endhighlight %}

Output should be:
{% highlight bash %}
Hi! My name is bob. Im a 52 year old male from DC
Hi! My name is erica. Im a 25 year old female from NY
{% endhighlight %}

This is a very rudimentary class structure, but shows you some of the basic principles of Object Oriented Ruby. If you'd like more in depth knowledge read [this tutorial on Ruby OOP](http://www.tutorialspoint.com/ruby/ruby_object_oriented.htm)
