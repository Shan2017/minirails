# Minirails

> Run example rails app from single file

Inspired by [The Smallest Rails App](http://thesmallestrailsapp.com/), I wanted to easily try various concepts inside rails apps without having to deal with all that bloat from standard rails directory structure.

I went even further, not only reducing full rails app into single file (that has already been done) but to reducing a set of rails apps into single file :sunglasses:

More seriously, you can now create whole microservice architecture within a singe file and somebody else will be able to grasp it all and execute it locally. Or you can easily make a single file app and post it to StackOverflow (and people can actually run this code)

Just think about it. Proof of Concepts. Asking questions. Explaining concepts.

## Installation

```
$ gem install minirails
```

## Basic usage

Create a file `myapp.rb` with the following content

```ruby
# myapp.rb
define do |app|
  class MainController < ActionController::Base
    def index
      render text: "Hello"
    end
  end

  app.routes.draw do
    get "/", to: "main#index"
  end
end
```

then run
```
$ minirails myapp.rb
```

and that's it - you now have fully functional rails app running on port 5000!

## Advanced usage

### Multiple apps

You can run multiple applications from singe file

```ruby
# file: multiple.rb
# usage: $ minirails multiple.rb
define "one" do |app|
  app.routes.draw do
    get "/", to: proc {|*| [200, {}, "Hello".lines] }
  end
end

define "two" do |app|
  app.routes.draw do
    get "/", to: proc {|*| [200, {}, "World".lines] }
  end
end
```

They will be run on ports 5000 and 5100 (and 5200 etc.)

### Database setup

You can also use full ActiveRecord capabilities.
The database (specified with `ENV["DATABASE_URL"]`) will be recreated (drop & create & migrate) on every run

```ruby
# file: database.rb
# usage: $ minirails database.rb
define "source" do |app|
  # Configuration
  ENV["DATABASE_URL"] = "postgresql://localhost/minirails_test"

  # Migrations
  class Migrations < ActiveRecord::Migration
    def change
      create_table :users do |t|
        t.string :name
        t.string :email
      end

      # Seeds
      User.create! [
        {name: "Jon", email: "jon@example.com"},
        {name: "Hodor", email: "hodor@example.com"}
      ]
    end
  end


  # Models
  class User < ActiveRecord::Base
  end

  # Controllers
  class UsersController < ActionController::Base
    def index
      render json: User.all
    end
  end

  # Routes
  app.routes.draw do
    resources :users
  end
end
```


### Non-rails apps

You can start any rack compatible web app inside define block
or even generic worker process (sidekiq, clock, whatever)
by specifying `:type` option.

```ruby
# file: types.rb
# usage: $ minirails types.rb

# this is rails app
define "rails" do |app|
  app.routes.draw do
    get "/", to: proc {|*| [200, {}, "Hello from Rails".lines] }
  end
end

# with type: :rack you need to return Rack-compatible app
define "rack", type: :rack do
  proc {|*| [200, {}, "Hello from Rack".lines] }
end

# with type: :blank you can run anything
define "worker", type: :blank do
  loop do
    $stderr.puts "Hello from worker"
    sleep 1
  end
end
```

### Running worker with Rails env loaded
```ruby
define "my-rails-app" do |app|
  app.routes.draw do
    get "/", to: proc {|*| [200, {}, "Hello from Rails".lines] }
  end
end

# you can use :initialize option to load rails (or some rack)
# before loading worker code (similar to `rake environment`)
define "worker", type: :blank, initialize: "my-rails-app" do
  loop do
    Rails.logger.debug "Hello from worker with Rails env"
    sleep 1
  end
end
```

## FAQ

- **How to speed up loading?**

  Run `$ minirails --fast` and then use `bin/minirails myapp.rb` instead


## Contributing

1. Fork it ( https://github.com/teamon/minirails/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
