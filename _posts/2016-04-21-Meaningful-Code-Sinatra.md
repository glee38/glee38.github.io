---
layout: post
title: Meaningful Code
categories: Development
excerpt: "The even more exhilerating story about the birth of my second code child. Please click the link above to read more."
comments: true
draft: false
---

# Meaningful Code

If you've read my first post, then you know that about a month ago, I gave birth to my first ever code child - Teavana CLI Gem. I was in labor for days, but quickly forgot about my exhaustion upon the sweet arrival of my first code baby. Today, I am excited to share the birth of my second code child, Meaningful Code! The labor period was a little longer this time around, but the birth was that much more rewarding. 

After learning about a smooth little fella who goes by the name "Sinatra", I was challenged to create a Sinatra app from scratch. I knew this was the perfect opportunity to create a platform for something I was passionate about, and with my newly aquired skills, I was more than excited to see my idea to fruition. Below, I will take you along my journey, once again sharing my struggles and triumphs, as well as a video walkthrough of my website. I hope you enjoy!

## Getting started

For this particular project, I was given the following requirements:

1. Build an MVC Sinatra Application.
2. Use ActiveRecord with Sinatra.
3. Use Multiple Models.
4. Use at least one `has_many` relationship
5. Must have user accounts. The user that created the content should be the only person who can modify that content
6. Models must have validations to ensure that bad data isn't created
7. Any validation failures must be shown to user with an error message

By now, I had grown accustomed to building app structures and environments from scratch, and I no longer felt intimidated by the technicalities of the project. My biggest concern was what my app would be about. I knew I wanted to make something that would not only help me understand code better, but also help others in some way. 

## What is Meaningful Code?

After a much thought, I finally came up with an idea for my Sinatra app. I wanted to create a website to aid budding nonprofit organizations with their web and/or mobile development needs. My goal was to build a platform in which developers and nonprofits could unite for a common cause.

Nonprofit organizations that did not have the resources to hire developers would upload their web/mobile development needs in the form of "projects" to the website, while developers could sign up and volunteer to work on a project (or projects) of their choosing. My hope was that by creating a platform to bring people together, we would, together, make a meaningful and lasting difference in the lives of others.

## Features
Once I had my idea, it was time to set it into motion. I began by whiteboarding the basics of my application:

#### As a guest (someone who is not logged in or signed up):

1. Navigation Bar
  + About Us
      + who we are
      + our mission
  + Our nonprofits
  + Our developers
2. Index page
  + Tell us who you are:
    + Developer
      + sign up
      + login
    + Nonprofit
      + sign up
      + login

#### As a nonprofit:

1. Homepage
  + View all available developers
  + Projects due today
  + Open projects
  + Closed projects
2. Your Profile 
  + includes tagline, projects, our cause, donate, edit your profile
3. Your Projects
  +  View projects
    +  Can click on project to see:
      +  title
      +  description
      +  due date
      +  current developers working on said project
    +  Manage your projects
    +  Delete your projects
    +  Projects due today, open, and closed
  +  Manage projects
  +  Delete projects
3. Messages
  + Received messages
  + Sent messages
  + New message
  + All messages
4. Create New Project
5. More
  + All nonprofits
  + All projects
  + All developers
  + Your developers
  + Logout 

#### As a developer:

1. Homepage
  + View all available projects
  + Projects due today
  + Open projects
  + Close projects
2.  Your Profile
  + includes github, about me, your projects, edit your profile
3. Messages
  + Received messages
  + Sent messages
  + New message
  + All messages
4. More
  + All nonprofits
  + All projects
  + All developers
  + Your nonprofits
  + Logout 

## Beginning the Labor
Next, it was time to lay out my associations and validations for each model.

#### 1. Developer
The developers would not be able to create/edit projects posted by the nonprofits, but they would be able to "take" projects to work on.

Assocations & Validations:

```ruby
class Developer < ActiveRecord::Base
  has_secure_password
  validates_presence_of :name, :github, :username, :email, :password_digest, :about_me
  validates_uniqueness_of :username, :email, :github
  validates_format_of :email, :with => /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\Z/i, :on => :create

  has_many :developer_projects
  has_many :projects, through: :developer_projects
  has_many :nonprofits, through: :projects
  has_many :messages

  include Slugify
  extend Slugify
end
```

Attributes:

```ruby
  create_table :developers do |t|
    t.string :name
    t.string :github
    t.string :username
    t.string :password_digest
    t.string :email
    t.text   :about_me
  end
```

#### 2. Nonprofit
The nonprofits would be able to create and edit projects. 

Assocations & Validations:

```ruby
class Nonprofit < ActiveRecord::Base
  has_secure_password
  validates_presence_of :name, :cause, :tagline, :website, :username, :email, :password_digest
  validates_uniqueness_of :username, :email, :website
  validates_format_of :email, :with => /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\Z/i, :on => :create

  has_many :projects
  has_many :developers, through: :projects
  has_many :messages

  include Slugify
  extend Slugify

  def self.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end
end
```

Attributes:

```ruby
  create_table :nonprofits do |t|
    t.string :name
    t.text   :cause
    t.text   :tagline
    t.string :website
    t.string :username
    t.string :password_digest
    t.string :email
  end
```

#### 3. Projects
Assocations & Validations:

```ruby
class Project < ActiveRecord::Base
  validates_presence_of :name, :project_specs, :due_date
  
  belongs_to :nonprofit
  has_many :developer_projects
  has_many :developers, through: :developer_projects

  def slug
    self.name.downcase.strip.gsub(" ", "-")
  end

  def self.find_by_slug(slug)
    self.all.find {|obj| obj.slug == slug}
  end
end
```


#### 4. Developer_Projects
The purpose of this join table was to provide a way for a developer to work on many projects, and for a projects to have many developers.

Assocations:

```ruby
class DeveloperProject < ActiveRecord::Base
  self.table_name = "developer_projects"
  belongs_to :developer
  belongs_to :project
end
```

Attributes:

```ruby
  create_table :developer_projects do |t|
    t.integer :developer_id
    t.integer :project_id
  end
```

#### 5. Messages
**NOTE**: The messaging feature was added last, and though it was not required, I felt that it only made sense for developers and nonprofit organizations to have a method of communication with one another within the scope of the website. I would have loved to build a more complex and efficient version of this feature, but due to time constraints and my limited knowledge of code at the time, I felt that a basic implementation of this feature would suffice for the time being.

Associations & Validations:

```ruby
class Message < ActiveRecord::Base
  validates_presence_of :recipient, :subject, :content

  belongs_to :developer
  belongs_to :nonprofit
end
```

Attributes:

```ruby
  create_table :messages do |t|
    t.string  :sender
    t.string  :recipient
    t.string  :subject
    t.text    :content
    t.integer :developer_id
    t.integer :nonprofit_id
    t.date    :date
  end
```

## MVCs
Once I had sucessfully set up my associations and databases, I dove into the model, view, controller portion of the application. This part turned out to be pretty extensive, but I will attempt to break down the process as clearly as possible. 

### MODELS
The "model" portion was structured as follows:

```
meaningful-code
  - models
    - concerns
      - slugify.rb
    - developer.rb
    - developer_project.rb
    - message.rb
    - nonprofit.rb
    - project.rb
```

All associations, validations, and inheritance of each class from `ActiveRecord::Base` (as seen in the 'Beginning the Labor' section) lived here. 

`slugify.rb` simply served as a module containing methods to find and convert urls to RESTful conventions:

```ruby
module Slugify  
  def slug
    self.username.downcase.strip.gsub(" ", "-")
  end

  def find_by_slug(slug)
    self.all.find {|obj| obj.slug == slug}
  end
end
```


### VIEWS - NAVBAR
Next, I moved on to the views. Because I wanted to focus on the functionality of the website, I decided that I would utilize the bootstrap framework to add front end design to my application. Once I found a template that best suited my needs and aesthetic, I added the following customizations to `layout.rb`:

```html
<!doctype html>
<html>
  <head>
    <title>Meaningful Code</title>
    <link rel="stylesheet" href="https://bootswatch.com/journal/bootstrap.min.css">
  </head>
  <body>

    <nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="navbar-brand" href="/">Meaningful Code</a>
    </div>

    <% if dev_logged_in? %>

    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li><a href="/developers/<%=current_dev.slug%>">Your Profile<span class="sr-only">(current)</span></a></li>

        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">Your Projects <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="/developers/<%=current_dev.slug%>/projects">View Projects</a></li>
            <li><a href="/developers/<%=current_dev.slug%>/projects/edit">Manage Projects</a></li>
          </ul>
        </li>

         <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">Messages <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="/developers/<%=current_dev.slug%>/messages/recieved">Recieved Messages</a></li>
            <li><a href="/developers/<%=current_dev.slug%>/messages/sent">Sent Messages</a></li>
            <li class="divider"></li>
            <li><a href="/developers/<%=current_dev.slug%>/messages/new">New Message</a></li>
            <li class="divider"></li>
            <li><a href="/developers/<%=current_dev.slug%>/messages">All Messages</a></li>
          </ul>
        </li>

        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">More <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="/nonprofits/all">All Nonprofits</a></li>
            <li><a href="/projects">All Projects</a></li>
            <li><a href="/developers/all">All Developers</a></li>
            <li class="divider"></li>
            <li><a href="/developers/<%=current_dev.slug%>/nonprofits">Your Nonprofits</a></li>
            <li class="divider"></li>
            <li><a href="/developers/logout">Logout</a></li>
          </ul>
        </li>
    </div>
  </div>
</ul>
   <ul class="breadcrumb">
    <li><a href="/developers/<%=current_dev.slug%>/homepage">Home</a></li>
    <li class="active">Developer</li>
    <li class="active"><%=current_dev.name%></li>
  </ul>
</nav>

    <% elsif np_logged_in? %>

    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li><a href="/nonprofits/<%=current_np.slug%>">Your Profile<span class="sr-only">(current)</span></a></li>

        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">Your Projects <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="/nonprofits/<%=current_np.slug%>/projects">View Projects</a></li>
            <li><a href="/nonprofits/<%=current_np.slug%>/projects/edit">Manage Projects</a></li>
            <li><a href="/nonprofits/<%=current_np.slug%>/projects/delete">Delete Projects</a></li>
          </ul>
        </li>

         <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">Messages <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="/nonprofits/<%=current_np.slug%>/messages/recieved">Recieved Messages</a></li>
            <li><a href="/nonprofits/<%=current_np.slug%>/messages/sent">Sent Messages</a></li>
            <li class="divider"></li>
            <li><a href="/nonprofits/<%=current_np.slug%>/messages/new">New Message</a></li>
            <li class="divider"></li>
            <li><a href="/nonprofits/<%=current_np.slug%>/messages">All Messages</a></li>
          </ul>
        </li>

        <li><a href="/<%=current_np.slug%>/projects/new">Create New Project<span class="sr-only">(current)</span></a></li>

        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">More <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="/nonprofits/all">All Nonprofits</a></li>
            <li><a href="/projects">All Projects</a></li>
            <li><a href="/developers/all">All Developers</a></li>
            <li class="divider"></li>
            <li><a href="/nonprofits/<%=current_np.slug%>/developers">Your Developers</a></li>
            <li class="divider"></li>
            <li><a href="/nonprofits/logout">Logout</a></li>
          </ul>
        </li>
    </div>
  </div>
</ul>
   <ul class="breadcrumb">
    <li><a href="/nonprofits/<%=current_np.slug%>/homepage">Home</a></li>
    <li class="active">Nonprofit</li>
    <li class="active"><%=current_np.name%></li>
  </ul>
</nav>

    <% else %>
    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li><a href="/about">About Us <span class="sr-only">(current)</span></a></li>
        <li><a href="/nonprofits/all">Our Nonprofits</a></li>
        <li><a href="/developers/all">Our Developers</a></li>
      </ul> 
    </div>
  </div>
</nav>
<%end%>
    <div class="container">
      <%= yield %>
    </div>
    <script src="https://bootswatch.com/bower_components/jquery/dist/jquery.min.js"></script>
    <script src="https://bootswatch.com/bower_components/bootstrap/dist/js/bootstrap.min.js"></script>
  </body>
</html>
```

WEW. That looks like a LOT. But this is what the layout is basically saying:

+ If a developer is logged in, display the following navbar:

![navdev](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 3.29.02 PM.png)


+ If a nonprofit is logged in, display the following navbar:

![navnp](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 3.29.33 PM.png)

+ If no one is logged in, display the following:

![navdefault](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 3.28.30 PM.png)

Once I had the basic layout for my website, I moved on to my controllers.

### CONTROLLERS

As I began to add functionality to the website, my controller files became increasingly intricate. I knew I had to account for the possibility of hackers, nonexisting urls, and overall user friendliness. Once I had set up the barebones for my controllers, my file structure was as follows:

```
meaningful-code
  - app
    - controllers
      - application_controller.rb
      - developers_controller.rb
      - nonprofits_controller.rb
      - projects_controller.rb
```

The `ApplicationController` was by far the most simple controller, as it only really had to account for the landing page, as well as defining some helpful methods to determine what kind of user was logged in. 

```ruby
require './config/environment'

class ApplicationController < Sinatra::Base

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions
    set :session_secret, "secretlymeaningful"
  end

  get "/" do
    if dev_logged_in?
      redirect "/developers/#{current_dev.slug}"
    elsif np_logged_in?
      redirect "/nonprofits/#{current_np.slug}"
    else
      erb :index
    end
  end

  get "/about" do
    erb :about
  end

  helpers do
    def dev_logged_in?
      !!session[:dev_id]
    end

    def current_dev
      @current_dev ||= Developer.find(session[:dev_id])
    end

    def np_logged_in?
      !!session[:np_id]
    end

    def current_np
      @current_np ||= Nonprofit.find(session[:np_id])
    end
  end

end
```

The developers and nonprofits controllers were far more dense - both had routes that were very similar to one another (with some additional routes for nonprofits to account for creating/editing projects), and both had conditionals to protect the website from being hacked in any way. 

Here is a snippet of the `NonprofitsController`:

```ruby
  get '/nonprofits/:slug/messages/:m_id/reply' do
    @message = Message.find_by_id(params[:m_id])
    erb :'nonprofits/reply', :layout => false
  end

  get '/nonprofits/:slug/messages/:m_id' do
    @message = Message.find_by_id(params[:m_id])
    @np = Nonprofit.find_by_slug(params[:slug])

    if !@np.nil?
      if np_logged_in?
        if current_np.slug == @np.slug
          if !@message.nil?
            erb :'/nonprofits/show_message'
          else
            erb :'messages/failure'
          end
        else
          redirect "/nonprofits/#{current_np.slug}/messages"
        end
      else
        redirect "/nonprofits/#{@np.slug}"
      end
    else
      redirect "/nonprofits/failure"
    end
  end

  post '/nonprofits/:slug/messages/:m_id' do
    @np = Nonprofit.find_by_slug(params[:slug])
    @message = Message.find_by_id(params[:m_id])
    @dev = @message.developer
    @new_message = Message.new

      if !params[:message][:content].empty?
        @new_message.developer = @dev
        @new_message.nonprofit = current_np
        @new_message.date = Date.today
        @new_message.content = params[:message][:content]
        @new_message.subject = "Re: #{@message.subject}"
        @new_message.recipient = @dev.email
        @new_message.sender = current_np.email
        @new_message.save
        erb :'/nonprofits/show_messages', locals: {message: "Message successfully sent!"} 
      else
        erb :'nonprofits/reply', locals: {message: "Message cannot be empty."} 
      end
  end

  get "/nonprofits/:slug/projects/closed" do
    @np = Nonprofit.find_by_slug(params[:slug])

    if !@np.nil?
      erb :"nonprofits/closed_projects", :layout => false
    else
      redirect "/nonprofits/failure"
    end
  end

  get "/nonprofits/:slug/projects/edit" do
    @np = Nonprofit.find_by_slug(params[:slug])
    if !@np.nil?
      if np_logged_in?
        if current_np.slug == @np.slug
          erb :"nonprofits/edit_projects"
        else
          redirect "/nonprofits/#{current_np.slug}/projects"
        end
      else
        redirect "/nonprofits/#{@np.slug}"
      end
    else
      redirect "/nonprofits/failure"
    end
  end

  post "/nonprofits/:slug/projects/edit" do
    @np = Nonprofit.find_by_slug(params[:slug])
    if !@np.nil?
      if np_logged_in?
        if current_np.slug == @np.slug
          erb :"nonprofits/edit_projects"
        else
          redirect "/nonprofits/#{current_np.slug}/projects"
        end
      else
        redirect "/nonprofits/#{@np.slug}"
      end
    else
      redirect "/nonprofits/failure"
    end
  end

```

In the above snippet, you can see various routes for the messaging feature as well as for editing and creating projects. `locals` defined a message variable that would display depending on whether or not its condition was met, and some of the routes had the `:layout => false` (because AJAX was being used on the page).

The `DevelopersController` was similar, with the exception of creating/editing projects. For the developers, "editing" a project meant being able to remove (delete) it from their list of "taken" projects:

```ruby
get '/developers/:slug/projects' do
    @dev = Developer.find_by_slug(params[:slug])
    if !@dev.nil?
      erb :'/projects/dev_projects'
    else
      redirect "developers/failure"
    end
  end

  get "/developers/:slug/projects/closed" do
    @dev = Developer.find_by_slug(params[:slug])

    if !@dev.nil?
      erb :"developers/closed_projects", :layout => false
    else
      redirect "/developers/failure"
    end
  end

  get '/developers/:slug/projects/edit' do
    @dev = Developer.find_by_slug(params[:slug])
    if !@dev.nil?
      if dev_logged_in?
        if current_dev.slug == @dev.slug
          erb :'/developers/edit_projects'
        else
          redirect "/developers/#{current_dev.slug}"
        end
      else
        redirect "/developers/#{@dev.slug}"
      end
    else
      redirect "/developers/failure"
    end
  end

  post '/developers/:slug/projects/edit' do
    @dev = Developer.find_by_slug(params[:slug])
    if !@dev.nil?
      if dev_logged_in?
        if current_dev.slug == @dev.slug
          erb :'/developers/edit_projects'
        else
          redirect "/developers/#{current_dev.slug}"
        end
      else
        redirect "/developers/#{@dev.slug}"
      end
    else
      redirect "/developers/failure"
    end
  end

  patch '/developers/:slug/projects/edit' do
    @dev = Developer.find_by_slug(params[:slug])

      @project_ids = @dev.project_ids
      @to_delete = params[:dev][:project_ids]
      @project_ids.delete_if {|p| @to_delete.include?(p.to_s)}
      @dev.update(project_ids: @project_ids)

      erb :"/developers/edit_projects", locals: {message: "Project(s) successfully removed."} 
  end

  get '/developers/:slug/messages' do
    @dev = Developer.find_by_slug(params[:slug])
    if !@dev.nil?
      if dev_logged_in?
        if current_dev.slug == @dev.slug
          erb :'/developers/show_messages'
        else
          redirect "/developers/#{current_dev.slug}/messages"
        end
      else
        redirect "/developers/#{@dev.slug}"
      end
    else
      redirect "/developers/failure"
    end
  end

  get '/developers/:slug/messages/sent' do
    @dev = Developer.find_by_slug(params[:slug])
    if !@dev.nil?
      if dev_logged_in?
        if current_dev.slug == @dev.slug
          erb :'/developers/sent_messages'
        else
          redirect "/developers/#{current_dev.slug}/messages"
        end
      else
        redirect "/developers/#{@dev.slug}"
      end
    else
      redirect "/developers/failure"
    end
  end

  get '/developers/:slug/messages/recieved' do
    @dev = Developer.find_by_slug(params[:slug])
    if !@dev.nil?
      if dev_logged_in?
        if current_dev.slug == @dev.slug
          erb :'/developers/recieved_messages'
        else
          redirect "/developers/#{current_dev.slug}/messages"
        end
      else
        redirect "/developers/#{@dev.slug}"
      end
    else
      redirect "/developers/failure"
    end
  end
```

Once again, the above snippet shows the routes containing various conditionals, as well as their interaction with projects and messages. 

Last but not least, the `ProjectsController` was quite simple with the following routes:

```ruby
class ProjectsController < ApplicationController
  get "/projects" do
    erb :"projects/all_projects"
  end

  get "/:slug/projects/new" do
    @project = Project.new
    @np = Nonprofit.find_by_slug(params[:slug])

    if !@np.nil?
      if np_logged_in?
        if current_np.slug == @np.slug
          erb :"projects/new"
        else
          redirect "/nonprofits/#{current_np.slug}/projects"
        end
      else
        redirect "/nonprofits/#{@np.slug}/projects"
      end
    else
      redirect "/nonprofits/failure"
    end
  end

  post "/:slug/projects" do
    @project = Project.new(params[:project])
    @project.valid?

    if np_logged_in? && @project.valid?
      @project.nonprofit = current_np
      @project.save
      redirect "nonprofits/#{current_np.slug}/projects/#{@project.slug}"
    else
      erb :"projects/new"
    end
  end

  get "/projects/failure" do
    erb :'projects/failure'
  end

end
```

### BACK TO VIEWS
Defining the routes inevitably meant that I would also simultaneously have to define the views. Though the defining routes had seemed repetitive in many ways, defining the views proved to be the most repetitive part of this process. I had not yet learned about partials, and with my limited knowledge, I found myself writing certain html snippets over and over.

Once the views were finished however, I was so excited with the amount that I had learned during the process. Repetitive? Yes. Still worth it? A million times yes.

Here are a view things I learned while coding my views:

1. How to use AJAX to load partial views.
2. How to display those awesome error messages to the user when filling out a form.
3. Many helpful methods that I may have not otherwise discovered or implemented
4. Some refreshers on html styling

Here is a snippet of a view that used AJAX:

`app/views/nonprofits/homepage.erb`

```html
<h1>Welcome, <%=current_np.name%>!</h1><br>

<a href="/developers/all"><p class="text-muted">View all available developers</p></a>

<h3><p class="text-warning">Projects Due Today:</h3><br>
<%if current_np.projects.count == 0%>
  There are no projects to display.<br>
<%else%>
  <% @due = [] %>
  <%current_np.projects.each do |p|%>
    <%if p.due_date == Date.today%>
      <% @due << p %>
      <li><h4><p class="text-muted"><a href="/nonprofits/<%=p.nonprofit.slug%>/projects/<%=p.slug%>"><%=p.name%></a></p></h4></li>
    <%end%>
  <%end%>
<%end%>

<%if @due.count == 0%>
  There are no projects to display.<br>
<%end%><br>

<h3>Open Projects:</h3><br>
<%if current_np.projects.count == 0%>
  There are no projects to display.<br>
<%else%>
  <%current_np.projects.each do |p|%>
    <%if p.due_date >= Date.today%>
    <li><h4><p class="text-muted"><a href="/nonprofits/<%=p.nonprofit.slug%>/projects/<%=p.slug%>"><%=p.name%></a></p></h4></li>
    <%end%>
  <%end%>
<%end%><br>

<div id="closed"></div>
<button type="button" onClick = "loadDoc(), this.style.visibility= 'hidden';">Show closed projects</button>
<br></br>
<br></br>

<script>
function loadDoc() {
  var xhttp = new XMLHttpRequest();
  xhttp.onreadystatechange = function() {
    if (xhttp.readyState == 4 && xhttp.status == 200) {
      document.getElementById("closed").innerHTML = xhttp.responseText;
    }
  };
  xhttp.open("GET", "/nonprofits/<%=@np.slug%>/projects/closed", true);
  xhttp.send();
}
</script>
```

And here are some images of the views in action!

#### When logged in as as nonprofit:

`app/views/projects/np_projects.erb`:

![mc1](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.27.16 PM.png)

After clicking on "Show closed projects" with AJAX load:

![mc2](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.27.35 PM.png)

`app/views/projects/show_project.erb`:

![mc3](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.28.14 PM.png)

`app/views/nonprofits/show_messages.erb`:

![mc4](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.28.45 PM.png)

#### When logged in as a developer

`app/views/projects/show_project.erb`:

![mc5](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.29.51 PM.png)

`app/views/nonprofits/show_nonprofit.erb`:

![mc6](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.30.14 PM.png)

After clicking "contact" with AJAX load:

![mc7](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.30.30 PM.png)

`app/views/projects/edit_projects.erb`:

![mc8](/Users/gracelee/Desktop/meaningful_code/Screen Shot 2016-05-01 at 4.30.44 PM.png)

And there you have it!

## CONCLUSION
I think the most exciting part of all of this was finally being able to create something (though my knowledge was limited) that I was passionate about and knew could potentionally be of use to other people. Seeing my website fully functioning and tangible was a feeling that has officially been added to the "best feelings ever" compartment in my heart.

If you would like to view the final product in action, [here](https://www.youtube.com/watch?v=gX31GLx8Org) is a video walkthrough of the website.

If you would like to share in my happiness, please feel free to download my baby by doing the following:

```
1. fork repo at https://github.com/glee38/meaningful-code
2. git clone onto your computer
3. run bundle install
4. run local server
5. enjoy!
```
