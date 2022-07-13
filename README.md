# Demo Rails app

Demonstration of:

* [Ruby on Rails](https://rubyonrails.org) web framework

* [Forest Admin](https://forest_admin.com) web service

* [Postgres](https://www.postgresql.org) relational database

* [SQLite](https://www.sqlite.org/index.html) lightweight database


## Prepare

If you use the command `asdf` to version:

```sh
asdf global ruby latest
asdf global postgres latest
```

Have Ruby 3?

```sh
ruby --version
```

Have Rails 7?

```sh
gem list rails
```

Have Postgres 14?

```sh
postgres --version
```

Have SQLite 3?

```sh
sqlite3 --version
```


## New demo

Create a new demo:

```sh
rails new demo_rails_forest_admin --database=postgresql
cd demo_rails_forest_admin
git add -A && git commit -am "Run rails new demo_rails_forest_admin --database=postgresql"
```

If you use `asdf` then set Ruby and Postgres

```sh
asdf local ruby latest
asdf local postgres latest
asdf install
git add -A && git commit -am "Add asdf with local ruby latest and local postgres latest"
```

Verify you can use the Postgres control command `pg_ctl`:

```
pg_ctl stop
pg_ctl start
```


## Add gem dotenv-rails

Add to `Gemfile`:

```ruby
# Environment variables
gem "dotenv-rails", groups: [:development, :test]
```

Run:

```sh
git add -A && git commit -am "Add gem dotenv-rails"

bundle
git add -A && git commit -am "Run bundle"
```

Append to file `.gitgnore`:

```gitignore
# Ignore dotenv environment variable file.
/.env
```

Run:

```sh
git add -A && git commit -am "Add .gitignore rule for dotenv file /.env"
touch .env
```


## Create the database access

For a demo app, we prefer to set database access via environment variables.

For a production app, we prefer to set database access via ephemeral controls such as via Hashicorp Vault.

For a naming convention, we prefer explicit database access environment variables:

* The database server name: POSTGRES

* The database server object: USER

* The role identifier: DEMO_RAILS_FOREST_ADMIN

* The role attribute: USERNAME or PASSWORD

* The word FOR and the runtime environment: DEVELOPMENT or TEST or PRODUCTION

For our password, we prefer to generate a strong random password, such as a strong random 32-digit number:

```sh
printf "%s\n" $(LC_ALL=C < /dev/urandom tr -dc '[:digit:]' | head -c32)
```

Edit file `.env` to set the database access enviornment variables and the generated passwords:

```sh
POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_DEVELOPMENT=demo_rails_forest_admin
POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_PASSWORD_FOR_DEVELOPMENT=11053635016261456248726591454979

POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_TEST=demo_rails_forest_admin
POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_PASSWORD_FOR_TEST=82138770260839801404701988766919

POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_PRODUCTION=demo_rails_forest_admin
POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_PRODUCTION=59637922222719767183438989864425
```

Create the database role and enter the password for development:

```sh
createuser --username=postgres --pwprompt --createdb demo_rails_forest_admin
```


## Configure the database access

Edit file `config/database.yml` to add each section's username and password:

```yaml
development:
  <<: *default
  database: demo_rails_forest_admin_development
  username: <%= ENV["POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_DEVELOPMENT"] %>
  password: <%= ENV["POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_DEVELOPMENT"] %>
  …

test:
  <<: *default
  database: demo_rails_forest_admin_test
  username: <%= ENV["POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_TEST"] %>
  password: <%= ENV["POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_TEST"] %>
  …

production:
  <<: *default
  database: demo_rails_forest_admin_production
  username: <%= ENV["POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_PRODUCTION"] %>
  password: <%= ENV["POSTGRES_USER_DEMO_RAILS_FOREST_ADMIN_USERNAME_FOR_PRODUCTION"] %>
  …
```

RUn:

```sh
git add -A && git commit -am "Add database access usernames and passwords"
```
Run:


## Pepare the database

```sh
bin/rails db:prepare
```

Output should include:

```sh
Created database 'demo_rails_forest_admin_development'
Created database 'demo_rails_forest_admin_test'
```



## Launch the server

Launch the Rails Puma server:

```sh
bin/rails server
```

Output should include:

```sh
Puma starting in single mode...
* Listening on http://127.0.0.1:3000
```

Browse <http://127.0.0.1:3000> and you should see the Rails welcome page.


## Enable Postgres UUID primary keys

Enable Postgres UUID primary keys and their functions via pgcrypto:

```sh
bin/rails generate migration EnableExtensionPgcrypto
```

Edit file `db/migration/*_enable_extension_pgcrypto`:

```ruby
class EnableExtensionPgcrypto < ActiveRecord::Migration[7.0]
  def change
    enable_extension 'pgcrypto'
  end
end
```

```sh
git add -A && git commit -am "Add db migration to enable extension pgcrypto, then migrate"
bin/rails db:migrate
git add -A && git commit -am "Run rails db:migrate"
```

Create file `config/initializers/generators.rb`:

```ruby
Rails.application.config.generators do |g|
  g.orm :active_record, primary_key_type: :uuid
end
```

```sh
git add -A && git commit -am "Add generators ORM primary key type UUID"
```


## Install ActiveStorage

Add to `Gemfile`:

```ruby
gem "image_processing", ">= 1.2"
```

Run:

```sh
git add -A && git commit -am "Add gem image_processing"

bundle
git add -A && git commit -am "Run bundle"

bin/rails active_storage:install
git add -A && git commit -am "Run rails active_storage:install"

bin/rails db:migrate
git add -A && git commit -am "Run rails db:migrate"
```

The project now has a new database migration file `db/migrate/*_create_active_storage_tables.active_storage.rb`.


## Add scaffolds for users, groups, memberships

Generate scaffolds for our demo resources, which are users, groups, memberships.

```sh
bin/rails generate scaffold User name:string description:string
bin/rails db:migrate
git add -A && git commit -am "Run rails generate scaffold User, then db:migrate"

bin/rails generate scaffold Group name:string description:string
bin/rails db:migrate
git add -A && git commit -am "Run rails generate scaffold Group, then db:migrate"

bin/rails generate scaffold Membership group:references user:references
bin/rails db:migrate
git add -A && git commit -am "Run rails generate scaffold Membership, then db:migrate"
```

Edit file `app/models/user.rb`

```ruby
class User < ApplicationRecord
  has_many :memberships
  has_many :groups, through: :memberships
end
```

```sh
git add -A && git commit -am "Add user associations"
```

Edit file `app/models/group.rb`

```ruby
class Group < ApplicationRecord
  has_many :memberships
  has_many :users, through: :memberships
end
```

```sh
git add -A && git commit -am "Add group associations"
```


## Add root route

Edit file `config/routes`:

```ruby
root "users#index"
```

```sh
git add -A && git commit -am "Add root route"
```


## Add Devise authentication

Add to `Gemfile`:

```ruby
# Devise authentication
gem "devise"
```

```sh
git add -A && git commit -am "Add gem devise"

bundle
git add -A && git commit -am "Run bundle"

rails generate devise:install
git add -A && git commit -am "Run rails generate devise:install"

rails generate devise User
git add -A && git commit -am "Run rails generate generate devise User"

rails db:migrate
git add -A && git commit -am "Run rails db:migrate"
```
