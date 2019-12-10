# Rails - React 

 We're going to create *two applications* , a frontend application with react that sends api requests to our backend built with Rails ! 

## Rails back-end ! 

Doctor `has_many` Patients `:through` Appointments

The has_many :through association is one of the most useful associations in Rails. It is a way of relating two models together *through* another model. For instance, a mentor may have many mentees and a mentee may have many mentors, all linked together through mentorships. The three models there would be mentors, mentees, and mentorships. 

Or doctors may have many patients and patients may have many doctors and they could all be linked together through their appointments. 

____________

First step, create our new project, called `doctor_app`. We don't need the test framework, so we'll leave that out of our project. We're also gonna tell Rails to configure our app to use Postgresql.

        $ rails new doctor_app --skip-test-unit -d postgresql

Next, we will create the three models we need for our has_many :through association. To make our lives easier, we'll use scaffolding in order to create the associated views and controllers at the same time.

        $ rails g scaffold doctor name:string --skip-test-framework
        $ rails g scaffold patient name:string --skip-test-framework
        $ rails g scaffold appointment reason:string doctor_id:integer patient_id:integer --skip-test-framework
        
With our new models, we need to run the migrations to create the necessary tables in our database, so:

        $ rails db:migrate db:setup
        
And now let's make the associations. This is done in the model files. First up, `doctor.rb` (in app/models directory) should look like this:

        class Doctor < ActiveRecord::Base
          has_many :appointments
          has_many :patients, through: :appointments
        end
        
Notice how the syntax makes it very clear what this association is all about. Pretty cool! Then, the `patient.rb` model file:

        class Patient < ActiveRecord::Base
          has_many :appointments
          has_many :doctors, through: :appointments
        end
				
And, finally, set up the `appointments.rb` model file like this:

        class Appointment < ActiveRecord::Base
          belongs_to :doctor
          belongs_to :patient
        end

We're almost ready to take a look at the app. Just a couple of steps before we fire it up in our browser. We want the appointments index page to be the root of our app, so open up `config/routes.rb` and add this line somewhere (toward the top is nice):

        root 'appointments#index'
        
Then, start the local server in the `doctor_app` main directory:

        $ rails s
        
Open your browser, and go to the address `http://localhost:3000`. This is what you should see:

![]()

Cool! Could it be that easy? Well, almost. If we click "New Appointment", we get this:

![]()

(It may look a little different, depending on your browser.) You'll notice that you're only allowed to enter numbers, though, which isn't exactly what we want.

![]()

Click on "Create Appointment" and you indeed create an appointment, but with useless numbers...

![]()

Those numbers are Doctor and Patient ID's, though none have been created yet. What we need are a bunch of doctor and Padawan names. Let's put some links in to create some doctors and patients. We need to head over to the views and do some work there. First up, `app/views/appointments/index.html.erb`. Add the following link code to the bottom of that file:

        <%= link_to 'New Doctor', new_doctor_path %>
        <%= link_to 'New Patient', new_patient_path %>
        
Take a look, reload http://localhost:3000 and we should see:

![]()

Ah ha, now we can start making some doctors and patients! First, let's make a bunch of doctors:

![]()

And then, create a slew of patients, all eager to become powerful doctors (go back to http://localhost:3000 to get to the "New patient" link):

![]()

Now we need to make it easy to pair the doctors and patients up through appointments, so let’s use their names rather than their ID’s through drop down menus. That code is found in the appointments _form.html.erb partial. Let’s change the end of that file from this:

      <div class="field">
    	  <%= f.label :doctor_id %><br />
    	  <%= f.number_field :doctor_id %>
      </div>
      <div class="field">
    	  <%= f.label :patient_id %><br />
    	  <%= f.number_field :patient_id %>
      </div>
      <div class="actions">
    	  <%= f.submit %>
      </div>
    <% end %>

to this:

          <div class="field">
            <%= form.select :doctor_id, Doctor.all.collect { |p| [ p.name, p.id ] } %>
          </div>
          <div class="field">
            <%= form.select :patient_id, Patient.all.collect { |p| [ p.name, p.id ] } %>
          </div>
          <div class="actions">s
            <%= f.submit %>
          </div>
        <% end %>
        
Now we have drop down menus, populated with all of our doctors and patients, all ready to be synched up through their appointments!

![]()

Create an appointment, and let’s see what happens…

![]()

Hmm, we’re almost there, but we’re only seeing the doctor and patient ID’s in the appointment show view. Not very satisfying. Let’s change that so we can see their names.

In `views/appointments/show.html.erb`, change these lines:

        <p>
          <b>Doctor:</b>
          <%= @appointment.doctor_id %>
        </p>
        
        <p>
          <b>Patient:</b>
          <%= @appointment.patient_id %>
        </p>

to read like this:

        <p>
          <b>Doctor:</b>
          <%= @appointment.doctor.name %>
        </p>
        
        <p>
          <b>Patient:</b>
          <%= @appointment.patient.name %>
        </p>

And reload…

![]()

Much better! But, clicking “Back” to get back to the appointments index, we see those darn ID’s again.

![]()

To change that, let’s change the appropriate `views/appointments/index.html.erb` lines to read from this:

        <% @appointments.each do |appointment| %>
          <tr>
        	<td><%= appointment.doctor_id %></td>
        	<td><%= appointment.patient_id %></td>

to this:

        <% @appointments.each do |appointment| %>
          <tr>
        	<td><%= appointment.doctor.name %></td>
        	<td><%= appointment.patient.name %></td>

And reloading the appointments index, we see:

![]()

That’s what we want to see! Make a few more associations, and we can see all the appointments we need:

![]()


_________________

## next up we're going to change our methods a bit ! 

We only care about certain methods this time , so let's do few changes :

1. index : 

```
 @appointments = Appointment.all
    render json: @appointments
    

```

2. show :
```
def show
@appointment = Appointment.find(params[:id])
render json: @appointment

  end
  
 ```
3. update :
  
  ```
  appointment = Appointment.find(params[:id])
if appointment.update(appointment_params)
  render json: appointment
else 
        render json: "{'msg': 'failed'}"

end
  end
  ```
  
*try to do destroy on your own now !*

1. We'll add the following code at the top of our controller :

```
 skip_before_action :verify_authenticity_token
```


2. Test all you're routes using postman :

Send request for get, post , put and delete 

now if all is good ,  let's keep our rails server running and jump into react for the second part of our applications ~ 


_________________________________

## React front-end :

``` npm init create-react-app appointments ``` 
``` npm create-react-app appointments```

1. let's create an appointment component:  


```

import React, { Component } from "react";
const axios = require("axios");

class Appointment extends Component {
  constructor(props) {
    super();

    this.state = {
      appointments: []
    };
  }

  componentDidMount() {
    axios("http://localhost:3000/appointments").then(response => {
      console.log(response.data);
      this.setState(prevState => ({
        appointments: response.data
      }));
    });
  }

  render() {
    console.log(this.state.appointments);

    const allAppointments = this.state.appointments.map((appt, order) => {
      return <div key={order}>{appt.reason}</div>;
    });

    return (
      <div>
        <h1>Hello</h1>
        {allAppointments}
      </div>
    );
  }
}

export default Appointment;



```

1. and now let's head back to our App.js component and change it to the following : 

```
import React from "react";
import "./App.css";
import Appointment from "./Appointment";

function App() {
  return (
    <div className="App">
      <Appointment />
    </div>
  );
}

export default App;



```

______________________

### CORS ! 

Ooh no the infamous cors error ! how can we fix this ? 

1. let's go back to our rails application and add a new gem to our gemfile : 

```
gem 'rack-cors'

```
 
run 

 ```bundle exec rake middleware```
 
 ```bundle install ```
    
    
1. Go to your config/application.rb and add the following lines : 

*after ```config.load_defaults 6.0``` *

```
 config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end
``` 

We are all set now! 


