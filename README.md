# Learn Rails 

Some terms to look at before we start: 
[betterexplained.com](http://betterexplained.com/articles/starting-ruby-on-rails-what-i-wish-i-knew/)

## Set up the environment: 

Install the latest version of Rails. 

```
gem install rails
```

## Create a new app

```
rails new twitter
```

This installs all the necessary files for an out-of-the-box rails app. Lets see what's inside.

```
cd twitter
```
#### Directory structure

`ls` will list all files in a folder. 
[This](http://www.tutorialspoint.com/ruby-on-rails/rails-directory-structure.htm) will give you a good overview of the directory structure. 

#### Running the server

To run the server, run `rails server` (or `rails s` for short)

Visit the app on your web browser at [localhost:3000](http://localhost:3000)

## Adding Posts 

Let's use the generate a `scaffold` for the `Post` object. This will create a Model, View and Controller for our `Post`.

``` 
rails generate scaffold Post content:string
```

The `content:string` tells rails that our `Post` object will have an attribute called `content`. We need to specify the type (e.g. `string`) for the database. 

The scaffold also created a database migration, which needs to be migrated. 

```
rake db:migrate
```

Now let's visit [localhost:3000/posts](http://localhost:3000/posts). This shows all our posts in our database. 

#### CRUD:

CRUD stands for create, update, read, destroy. These are the operations our posts can do implemented by the Rails scaffold. Try it out by pressing `New Post`. 

#### The Rails console.

The Rails console loads your rails app into an IRB console, allowing you to manipulate your posts for example. 

```
rails console
```

We can also see what posts we have in the database via the rails console.

```ruby 
Post.all
```

In order to create a post.

```ruby
Post.create(content: "hello i am a post")
```

Now if we run `Post.all` we will see our post. 

Let's destroy our `Post`: `Post.first.destroy`.

#### MVC and Rails 

MVC stands for Model View Controller. Here's a great overview of MVC in context of Rails: [betterexplained.com](http://betterexplained.com/articles/intermediate-rails-understanding-models-views-and-controller)

## Bootstrap Rails 

Gems are libraries for Ruby. We're going to use `twitter-bootstrap-rails` to add Bootstrap, a popular js and css framework, to our app. 

In the `Gemfile`, add the line `gem "twitter-bootstrap-rails"`

Now update your bundle – the list of gems installed in your app. 

```
bundle install
```

And the Bootstrap installation requires one more step:

```
rails generate bootstrap:install static
```

To quickly add a Bootstrap layout to our `Posts` view: 

```
rails g bootstrap:themed Posts
```

Check out the changes with `rails s`

## Adding user auth with devise

Posts belong to users, and there's a great gem for user authentication called `devise`. 

Add `devise` to your bundle by adding the following line to your `Gemfile`.

```
gem 'devise'
```

Then run `bundle` again. 

Like Bootstrap we need to run a generator to integrate into our app.

```
rails generate devise:install
```

We also need to create a `User` model. We do this using another devise generator:

```
rails generate devise User
```

And we need to migrate the database again: 

```
rake db:migrate
```

Let's check out our new routes devise generated. Visit [localhost:3000/users/sign_in](http://localhost:3000/users/sign_in) and [localhost:3000/users/sign_up](http://localhost:3000/users/sign_up).

#### Adding a landing page

If we visit localhost:3000/, it still shows our template page. Let's build a mini-landing-page. We will generate a Home Controller and build a method to render our landing page. 

```
rails generate controller home index
```

The `index` will add a view in `app/views/home/`. In `home_controller.rb` add the following code:

```ruby
class HomeController < ApplicationController
  def index
    render "index"
  end
end
```

This will render our view named `index`. 

We need to tell Rails to call our `index` method when we hit `localhost:3000/`. Add the following code to `routes.rb`. 

```ruby
root 'home#index'
```

If we visit [localhost:3000/](http://localhost:3000/) we see our new view.

Let's add some authentication information and links to our index page. Add this code to `index.html.erb`

```erb
<% if user_signed_in? %>
  Logged in as <strong><%= current_user.email %></strong>.
  <%= link_to 'Edit profile', edit_user_registration_path, :class => 'navbar-link' %> |
  <%= link_to "Logout", destroy_user_session_path, method: :delete, :class => 'navbar-link'  %>
<% else %>
  <%= link_to "Sign up", new_user_registration_path, :class => 'navbar-link'  %> |
  <%= link_to "Login", new_user_session_path, :class => 'navbar-link'  %>
<% end %>
```

### Posts and Authentication   

Users can have many posts, and each post belongs to a user. Rails makes this really easy. Let's generate a new migration to establish this relationship. 

```
rails g migration AddReferenceToPost user:belongs_to
```

In `user.rb` add the line `has_many :posts`, and in `post.rb` add the line `belongs_to :user`

Now when we run the rails console, we can easily access and manipulate each user's posts. 

```ruby
user = User.first # get the first user
user.posts 
# will return an array of posts
```

Let's require authentication to edit posts. Add the following line to `posts_controller.rb`

```ruby
before_action :authenticate_user!, only: [:create, :edit, :update, :destroy]
```

When you try to update a post when not logged in, you will be redirected to the login view. Login, and it'll return you to the update view. 


In the index view, we should show which user shared the post. Let's update the `views/posts/index.html.erb`.

```erb
...
<th><%= model_class.human_attribute_name(:user) %></th>
...
<td><%= post.user.email if post.user %></td>
...
```

And when you share a new post, it should automatically build the relationship between it and the logged in user. Let's update the `create` method in `posts_controller.rb`.

```ruby
@post = current_user.posts.new(post_params)
```

Try it out. It works. 

<i>Bonus points</i>: Let's also add a link to the posts view in our `index.html.erb` 

```erb
<%= link_to "posts", posts_path %>
```

## Paginating posts

#### Adding fake data

Before we paginate our posts, let's create <b>100</b> fake posts. In the rails console, run the following code:

```ruby
100.times do |i|
  User.first.posts.create(content: "this is a post lol #{i}")
end
```

Loading our posts page is slower, and it's way too long. This is where pagination comes in.

#### Kaminari

There's an amazing gem called kaminari for pagination. Let's add it to our `Gemfile`

```ruby 
gem 'kaminari'
```

And then run `bundle` to install the gem. 

Like always, we're going to run the kaminari generator: 

```
rails g kaminari:config
```

In the index method of `posts_controller.rb`, update the code:

```ruby
@posts = Post.all.page params[:page]
```

And in `index.html.erb`, add the following line:

```erb
<%= paginate @posts %>
```

Visit localhost:3000/posts. Boom. You're already done with pagination! 

## Adding image upload with Paperclip

Note: On mac, you'll need to install ImageMagick: `brew install ImageMagick`

There's a great gem called Paperclip for image upload. Let's add this gem to our Gemfile.

```ruby
gem "paperclip", "~> 5.0.0.beta1"
```

As always, run `bundle`.

Now each post will have an optional image attached. Let's update the `Post` model.
```ruby
has_attached_file :image, styles: { medium: "300x300>", thumb: "100x100>" }, default_url: "/images/:style/missing.png"
validates_attachment_content_type :image, content_type: /\Aimage\/.*\Z/
```

And let's generate a migration to add an image to posts.

```
rails generate paperclip post image
```

Now we need to update the views so you can actually upload an image.

In `form.html.erb` add the following line:

```erb
<%= f.file_field :image %>
```

And in the `PostController` we need to permit this new parameter. 

```ruby
params.require(:post).permit(:content, :image)
```

Let's show the image to the user. Add the following code to `index.html.erb`

```erb
...
<th><%= model_class.human_attribute_name(:image) %></th>
...
<td><%= image_tag post.image.url(:thumb) %></td>
...
```

Finally, migrate the database and run the server. 

```
rake db:migrate
rails s
```
#### if we have more time we'll do this: 
- upvoting posts using [acts_as_votable](https://github.com/ryanto/acts_as_votable)
- list users, profile pages show their posts. 
- deploy to heroku 
- write a test [or many tests]! hehehe. 

Check out some cool resources: 
- [rspec](https://github.com/rspec/rspec-rails) for testing.
- [Omniauth](https://github.com/intridea/omniauth) for social login.

Sources:
- http://betterexplained.com/articles/intermediate-rails-understanding-models-views-and-controllers/
- https://www.railstutorial.org/book/_single-page#sec-planning_the_application
- https://docs.google.com/document/d/1HhWzMUwaoxYuAQiroAJkabTfaqaXMb4JA5c9bqLrUdI/edit?usp=sharing
