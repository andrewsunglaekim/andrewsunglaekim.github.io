---
layout: post
title: Many Actives to Many Records
excerpt: Has many through Active Record /w command line
---

>This post will require you to have a basic knowledge of Active Record and postgreSQL. In this post we're going to be going over a many-to-many relationship in Active Record. We'll have 3 models: Doctor, Patient and Appointment.

Sometimes when relating models, there will be cases where many instances of a model are associated with many instances of a different model. In these cases its important to introduce a third model to tie the associations together. We may even want to add additional attributes to this third model. In this blog post, we will have a Doctor model and a Patient model, and they will be associated through a third Appointment Model.

---
Let's make the file structure for our app and create all the directories and files we'll need in the terminal.

{% highlight bash %}
$ mkdir lib
$ mkdir db
$ touch required_gems.rb
$ touch lib/doctor.rb
$ touch lib/patient.rb
$ touch lib/appointment.rb
$ touch db/connection.rb
$ touch db/schema.sql
$ touch db/seed.rb
$ touch Gemfile
$ touch app.rb
{% endhighlight %}

Let's start by bundling our dependencies for this app. Put these dependencies in the `Gemfile` file:
{%highlight ruby%}
source 'https://rubygems.org'

gem 'pry'
gem 'pg'
gem 'activerecord'
{% endhighlight %}

Now we run `$ bundle install` in the terminal to install the dependencies for this app.
Lets start by setting the schema for our database in `db/schema.sql`:

{% highlight SQL %}
-- to reset the tables when you load the schema
DROP TABLE IF EXISTS doctors;
DROP TABLE IF EXISTS patients;
DROP TABLE IF EXISTS appointments;

-- creates doctors table in database
CREATE TABLE doctors(
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL
);

-- creates patients table in database
CREATE TABLE patients (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL,
  sex VARCHAR(10) NOT NULL
);

-- creates appointments table in database, each row
-- belongs_to a doctor and a patient
CREATE TABLE appointments(
  id SERIAL PRIMARY KEY,
  time VARCHAR(100) NOT NULL,
  patient_id INTEGER NOT NULL,
  doctor_id INTEGER NOT NULL
);
{% endhighlight %}

Let's go ahead and create our database and load our schema in `db/schema.sql`:

{% highlight bash %}
$ createdb hospital
$ psql -d hospital < db/schema.sql
{% endhighlight %}

Lets make a quick update to our `required_gems.rb` so we can call it on any files that will require gems:

{% highlight ruby %}
require 'active_record'
require 'pg'
require 'pry'
{% endhighlight %}

Ok, we're going to fill in the file `db/connection.rb` that establishes connection with the database:

{% highlight ruby%}

# establish a connection to the hopistal database
# using postgres
ActiveRecord::Base.establish_connection(
	database: 'hospital',
	adapter: 'postgresql'
)
{% endhighlight %}

Let's define our ActiveRecord models in `lib/doctor.rb`:

{% highlight ruby %}
# Doctor class definition
class Doctor < ActiveRecord::Base
  # has_many association with appointments and
  # a dependent destroy
  has_many :appointments, dependent: :destroy

  # has_many patients association through the has_many
  # appointments association
  has_many :patients, through: :appointments
end
{% endhighlight %}

in `lib/patient.rb`:

{% highlight ruby %}
# Patient class definition
class Patient < ActiveRecord::Base
  # has_many association with appointments and
  # a dependent destroy
  has_many :appointments, dependent: :destroy

  # has_many doctors association through the has_many
  # appointments association
  has_many :doctors, through: :appointments
end
{% endhighlight %}

in `lib/appointment.rb`:

{% highlight ruby %}
class Appointment < ActiveRecord::Base
  # belongs_to association for both patient and doctor
  belongs_to :patient
  belongs_to :doctor
end
{% endhighlight %}

> Couple things to note. For dependent destroy, if any Patient or Doctor objects get destroyed, their related appointments are destroyed as well. Without the appointment model the doctor model and patient model can not be associated in this code.

Let's go ahead and fill out our seed file `db/seed.rb`, so we have some objects to play with in pry(ruby REPL):

{% highlight ruby %}
require_relative 'connection'
require_relative '../lib/doctor'
require_relative '../lib/patient'
require_relative '../lib/appointment'

Doctor.destroy_all
Patient.destroy_all

# seed doctors into database
Doctor.create([
  {name: "Bob Stephens", email: "bob@email.com"},
  {name: "Sarah Smith", email: "sarah@email.com"},
  {name: "Jack Bauer", email: "jack@email.com"},
  {name: "Eric Cartman", email: "eric@email.com" },
  {name: "Wilma McKnight", email: "wilma@email.com"}
])

# seed patients into database
Patient.create([
  {name: "Tom Jones", email: "tom@email.com", sex: "male"},
  {name: "Regina Filangey", email: "regina@email.com", sex: "female"},
  {name: "Matthew McFly", email: "mattew@email.com", sex: "male"},
  {name: "Shannon Sean", email: "shannon@email.com", sex: "female"},
  {name: "Daryl Dixon", email: "daryl@email.com", sex: "male"}
])

# seed appointments into database
Appointment.create([
  {time: "11:30 am", patient_id: Patient.all[0].id, doctor_id: Doctor.all[0].id},
  {time: "12:30 pm", patient_id: Patient.all[0].id, doctor_id: Doctor.all[1].id},
  {time: "10:30 am", patient_id: Patient.all[0].id, doctor_id: Doctor.all[2].id},
  {time: "9:30 am", patient_id: Patient.all[0].id, doctor_id: Doctor.all[3].id},
  {time: "9:10 am", patient_id: Patient.all[1].id, doctor_id: Doctor.all[3].id},
  {time: "10:10 am", patient_id: Patient.all[1].id, doctor_id: Doctor.all[2].id},
  {time: "1:30 pm", patient_id: Patient.all[1].id, doctor_id: Doctor.all[2].id},
  {time: "2:30pm", patient_id: Patient.all[2].id, doctor_id: Doctor.all[2].id},
  {time: "3:30pm", patient_id: Patient.all[2].id, doctor_id: Doctor.all[4].id},
  {time: "5:30pm", patient_id: Patient.all[2].id, doctor_id: Doctor.all[2].id},
  {time: "8:30 am", patient_id: Patient.all[2].id, doctor_id: Doctor.all[0].id},
  {time: "7:30 am", patient_id: Patient.all[3].id, doctor_id: Doctor.all[0].id},
  {time: "10:45 am", patient_id: Patient.all[3].id, doctor_id: Doctor.all[1].id},
  {time: "10:50 am", patient_id: Patient.all[3].id, doctor_id: Doctor.all[1].id},
  {time: "10:55 am", patient_id: Patient.all[4].id, doctor_id: Doctor.all[1].id},
  {time: "11:45 am", patient_id: Patient.all[3].id, doctor_id: Doctor.all[4].id},
  {time: "12:50 pm", patient_id: Patient.all[2].id, doctor_id: Doctor.all[3].id},
  {time: "2:30 pm", patient_id: Patient.all[4].id, doctor_id: Doctor.all[3].id},
  {time: "5:30 pm", patient_id: Patient.all[0].id, doctor_id: Doctor.all[4].id},
  {time: "3:30 pm", patient_id: Patient.all[1].id, doctor_id: Doctor.all[1].id},
  {time: "4:30 pm", patient_id: Patient.all[1].id, doctor_id: Doctor.all[1].id},
  {time: "6:30 pm", patient_id: Patient.all[4].id, doctor_id: Doctor.all[0].id}
])
{% endhighlight %}

Alright lets go ahead an run our seed file in the terminal:
{% highlight bash %}
$ ruby db/seed.rb
{% endhighlight %}
In the `app.rb` file place these contents so that it lets us hop into pry with a connection to our database:

{% highlight ruby %}
# require all depedencies
require_relative 'required_gems'
require_relative 'db/connection'
require_relative 'lib/doctor'
require_relative 'lib/patient'
require_relative 'lib/appointment'

# start the REPL // also if you were to make a more
# verbose app, this is also where you may want to
# include the UI for your app. Maybe like an apointment
# scheduling app for a doctor?
binding.pry
{% endhighlight %}

Lets run our `app.rb` file:
{% highlight bash%}
$ ruby app.rb
{% endhighlight %}
Running this command should startup pry(ruby REPL). I encourage you to use the REPL and test out the association and see some of the helper methods in actions. Some commands to try out in pry:

- Doctor.first.appointments
- Doctor.first.patients
- Patient.last.appointments
- Patient.last.doctors

> Note that the only patients that are associated with a single doctor are ones in which the doctor has an appointment with the patient and vice versa. A doctor and a patient are only associated in so far as that they must have an appointment that belongs to both of them.

Hopefully, this post can shed some light on the has_many through association.
