---
layout: post
title: AuthSeries Pt. 2 - Creating Your Own Authentication
categories: Development
excerpt: "Part 2 of the Rails Authentication Series - Building Your Own Authentication Logic"
comments: true
---

Welcome to **Part 2** of AuthSeries, a series dedicated to authentication in Rails. In today's post, I will go over how to go about creating your own authentication logic from scratch. Rails has a lot of useful gems that get this job done for you, but I think it's always useful to understand what is going on under the hood before using gems that abstract away these basic, but very important principles.

#### OUR GOAL TODAY:

1. Learn how to build our own authentication logic inside of the:
  + User model and controller
  + Sessions controller
  + Application controller
2. Allow users to sign up and login through forms
3. Use the `has_secure_password` method
4. Write validations

#### STEP 1: GENERATE USER RESOURCE

Let's say we are building a "Recipes" website where users can sign up, login, create, edit and view their own recipes. 

![image of recipes index page](http://i66.tinypic.com/2d960sj.jpg=100x100)

In order to keep track of our users and store their recipes in the database, we will need to create sign up and login forms on your website. But before we do that, let's begin by generating our User model and migrating it to the database.

**NOTE**: This is assuming that you have already generated your rails application and are cd'd into your application's folder. 

In terminal, begin by typing in the following:

```
rails g resource User email:string password_digest:string
```

It is important to take note of the `password_digest` column, as it is the column that will utilize the `has_secure_password` feature in Rails that will be an essential part of our authentication logic.

Running the above command  will generate the `User` model and `UsersController`.

#### STEP 2: `has_secure_password`

Now, go inside of your User model and add `has_secure_password` like so:

```ruby
class User < ActiveRecord::Base
has_many :recipes
has_secure_password

# more code to come 

end
```

The `has_secure_password` will do the following:

1. Require a `password_digest` attribute (which we added previously in our Users table)

2. Add methods to set and authenticate against a BCrypt password (a gem we will install later to help encrypt passwords)

3. Automatically add  validations:
  + Password must be present on creation
  + Password length should be less than or equal to 72 characters
  + Password confirmation (using the password_confirmation attribute)

Just by typing in one little line, Rails automagically generated validations and methods to help us in our authentication journey. Rails, you da best.

#### STEP 3: `BCrypt`

To utilize `has_secure_password`, uncomment this line in your `Gemfile`:

```ruby
# Use ActiveModel has_secure_password
gem 'bcrypt', '~> 3.1.7'
```

Then, run `bundle install` in your terminal. The `bcrypt` gem will hash a user's password and stores it securely in your database.

#### STEP 4: ADD VALIDATIONS

Now that we have successfully set up our params, let's go back inside the User model. Here, you can add validations for the attributes of your choice. This is utilized mostly in the 'Sign Up' portion of our application.

For example:

```ruby
class User < ActiveRecord::Base
has_many :recipes
has_secure_password

validates_uniqueness_of :email
validates_presence_of :email
# ... etc.

end
```

You do not need to add validations for the `password` attribute since they are automatically added by `has_secure_password`. Woo!

#### STEP 5: STRONG PARAMS

If Rails terms were people, this one would be wearing sunglasses. Besides sounding undeniably cool, strong parameters are crucial in that they give you the ability to sanitize params passed in through user forms.

What does this mean exactly?

In your `UsersController`, type in the following:

```ruby
class UsersController < ApplicationController

# we will add actions here later

private # create private method user_params

def user_params
  params.require(:user).permit(:email, :password, 
  :password_confirmation)
end
```

**Let's break down the above code:**

1. In the `params` hash, require the `user` key

2. Allow only `email`, `password` and `password_confirmation` attributes to be passed in by the user. (The permitted attributes are not necessarily required - just allowed.)

What exactly will this look like when a user submits a sign up form? The value of `params[:user]` upon form submission should look similar to the following:

```
{ "email" => "grace@test.com", "password" => "test",
 "password_confirmation" => "test }
```

This way, users cannot assign attributes to fields that you do not want them to have access to (such as the option to select whether or not they are an admin). Strong parameters are so important in fact, that Rails will raise an exception if a form is submitted and the model params are not found.

The strong params plugin not only sanitizes your params, but it allows for flexibility in mass assignment. 

For example:

```ruby
def user_params
  if current_user.admin?
    params.require(:user).permit(:email, :password, 
    :password_confirmation, :is_admin)
  else
    params.require(:user).permit(:email, :password, 
    :password_confirmation)
  end
end
```

**NOTE**: If you have googled around about how to set up authentication in Rails, you may have read about `attr_accessible`. This method has been deprecated in Rails 4 in favor of strong params as the latter is more flexible in mass assignment. 

#### STEP 6: `UsersController`

Next, we will write some actions in our `UsersController` to handle signing up a user.

In `UsersController`, type in the following:

```ruby
class UsersController < ApplicationController
  def new
    @user = User.new
  end
  
private
  def user_params
  params.require(:user).permit(:email, :password, 
  :password_confirmation)
  end

end
```

Here, we are setting up an instance variable of `@user` that our view will be able to access later on in order to build our user sign up form. 

Next, let's build our `create` action:

```ruby
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      flash[:notice] = "Thank you for signing up!"
      redirect_to root_path
    else
      render :new
    end
  end
  
  private
  def user_params
    params.require(:user).permit(:email, :password, 
    :password_confirmation)
  end
end
```

**Let's break down what is happening on within our `create` method:**

1. Set the `@user` variable to a new `User` instance with our strong parameters, `user_params`, to mass assign attributes

2. If `@user` saves successfully (meaning all of our validations have returned true), create a `session[:user_id]` equal to the current user's id, `@user.id`. This automatically logs the user in upon signing up so they do not have to login again.
  + **NOTE**: Every Rails app comes equipped with a session for each user. Read more about how Rails sessions work under the hood [here](http://guides.rubyonrails.org/action_controller_overview.html#session).

3. Redirect to `root_path` and display a flash notice thanking the user for signing up. 

4. If `@user` does not save successfully due to validation errors, re-render the sign up page with error messages.

#### STEP 7: `ROUTES`

Now that we have successfully set up our `User` actions, let's write out some routes to make sure our actions actually have paths to which they can route to. 

In `config/routes.rb`, type in the following:

```ruby
Rails.application.routes.draw do
  resources :users
  resources :recipes
end
```

The `resources` keyword will create restful routes for the specified controllers. To view these routes, type in `rake routes` or `rake route | grep users` in terminal. 

#### STEP 8: `VIEWS/USERS`

It is finally time to move on to our views! 

In `views/users/new`, type in the following:

```ruby
<h1>Sign Up</h1>

<%= form_for @user do |f| %>
  <% if @user.errors.any? %>
    <div class = "error_messages">
    <h2>Form is invalid</h2>
    <ul>
      <% @user.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
    </div>
  <% end %>
  
  <div class = "field" >
    <%= f.label :email %><br>
    <%= f.text_field :email %><br>
  </div>
  
  <div class = "field" >
    <%= f.label :password %><br>
    <%= f.text_field : password %><br>
  </div>

  <div class = "field" >
    <%= f.label : password_confirmation %><br>
    <%= f.text_field :password_confirmation %><br>
  </div>
  
  <%= f.submit%>
<% end %>
```

You can, of course, customize your form in whatever way you'd like. Another option is to write your form in a partial, such as `_form.html.erb` and simply render the partial within your `new.html.erb` view.

We're *almost* there! Next, let's create a link to our sing up page within our application layout.

In `app/views/layouts/application.html.erb`, type in the following line wherever you'd like (I personally like to put this in a `_navigation.html.erb` partial and render it in `layout.html.erb`):

```ruby
<%= link_to "Sign Up", new_users_path %>
```

Run `rails s` in your terminal to start up the rails server on `http://localhost:3000/` and you should see something similar to:

[drawing of recipes index page with sign up link]

![drawing of signup form](http://i67.tinypic.com/20auge1.jpg)

[example of login form with error]

[example of login form filled in]

[recipes index page with success message]

At this point, we have successfully managed to create our user and log them in! We. Are. Awesome. 

But wait - we still have no way to log ourselves out, or log back in! Let's build that next.

#### STEP 9: `SessionsController`

In terminal, type in the following:

```
rails g controller sessions
```

This will generate your `SessionsController`, where our authentication will actually happen when a user logs in.

In `SessionsController`, type in the following:

```ruby
class SessionsController < ApplicationController
  
  def new
    @user = User.new
  end

  def create
    @user = User.find_by_email(params[:email])
    if @user && @user.authenticate(params[:password])
      session[:user_id] = @user.id
      flash[:notice] = "Login successful!"
      redirect_to user_path(@user)
    else
      flash[:notice] = "Email or password is invalid."
      redirect_to new_users_path
    end
  end

  def destroy
    if session[:user_id]
      reset_session
    end
    flash[:notice] = "Logged out successfully."
    redirect_to '/'
  end
  
end
```

**Let's break down what is going on within our `create` action:**

1. When someone logs in, use their email to search for an existing user with that email address in the database.

2. If a user is found and authenticated (`authenticate` is a method that `has_secure_password` provides for us, which will return true or false depending on whether the password entered matches the one saved in the database), create a `session[:user_id]` equal to `@user.id` and redirect the user to their show page.

3. If a user with that email address is not found, redirect the user to the sign up page and show an error message.

The `destroy` action logs a user out by simply resetting the session if the session contains a `user_id`.

Going back into `routes.rb`, let's define a few routes to help us out.

**You can go about the next step in two ways:**

1. Simply add `resources :sessions` into your route file

2. OR create custom routes as shown below:

```ruby
Rails.application.routes.draw do
  resources :users
  resources :recipes

  root 'recipes#index'
  get '/' => 'recipes#index'

  get '/login' => 'sessions#new'
  post '/login' => 'sessions#create'

  delete '/logout' => 'sessions#destroy'

end
```

#### STEP 10: `VIEWS/SESSIONS`

Next up are our views! 

In `app/views/sessions/new.html.erb`, type in the following:

```html
<%= form_for @user, url: {action: "create"} do |f| %>
  <p>
  <%= f.label :email %><br>
  <%= f.text_field :email %>
  </p>
  
  <p>
  <%= f.label :password %><br>
  <%= f.password_field :password %>
  </p>  
  
  <%= f.submit "Log In" %>
<% end %>
```

This will render our login form when a user visits `www.your-site-name.com/login`, prompting the user to input their email and password with which to authenticate them.

Finally, we will add the following links to our `_navigation.html.erb`:

```html
<%= link_to "Login", login_path %>
<%= link_to "Logout", logout_path, :method=>'delete' %>
```

Now when we go back to `http://localhost:3000/`, you should see:

[login, sign up, logout in navbar]

Clicking on "login" will take us to `http://localhost:3000/login`:

[login form]

[filling out login form]

[if password is incorrect / display error message]

[if login is successful / login success message]

Clicking on "logout" will post to `http://localhost:3000/logout` and redirect the user to the `root_path`:

[logout success message on index page]

And there you have it - you have just built your own Rails authentication system! So proud of you.

#### CONCLUSION

In today's post, we've covered how to set up basic user authentication using sessions, validations, and `has_secure_password`. In our next post, I will go over how to build some great **helper methods** that will allow us to further tailor our authentication system, including `current_user`, `logged_in?`, and `authenticate!`.