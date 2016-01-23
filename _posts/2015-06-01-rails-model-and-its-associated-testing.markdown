---
layout: post
title: "Rails Model"
date: 2015-06-01 10:39:49 +1000
comments: true
header-img: "img/post-bg-01.jpg"
categories: [rails, ruby]
reading_time: "15 mins"
---

This post contains my learning notes and some reflections towards rails model, model validation, and its
associated testing.

<!--more-->

- Model
  - Rails uses *database* for storage by default
- ActiveRecord
  - default library for interacting with the database
  - ORM
  - write model definition in ruby without using SQL
  - used for model definition
- Migration
  - a convenient way to alter your database schema
  - more like a version control of your database
  - write migration in ruby without using SQL
  - used for altering your model

Both ActiveRecord and Migration provide a high level way of interacting with database using Domain Specific Language without
touching SQL. <br>

![model](/images/rails/rails_model_migration_activerecord.png)

------------
Here is a short description of how to do the migration:
- Use rails cli to generate model
  - e.g. `rails g model User name:string email:string`
  - {% gist 94ac6c6d6a2ad511a734 generate_user_model %}
  - Explanation
    - **model names are singular** (a Users controller, but a User model), since it's a model, a template.
    - ActiveRecord creates `user.rb` for model definition, `[timestamp]_create_users` for model migration.
    - Test_unit creates `user_test.rb` for User model unit test, `users.yml` for test data preparing.

- The Migration File
  - {% gist 94ac6c6d6a2ad511a734 create_users.rb %}
  - Explanation
    - `change` method uses rails method `create_table` to create a table for users
    - table name are called `users` instead of `User`. Since a model represents a single user template, whereas a database table consists of many users
    - `create_table` accepts a block, in the block, create columns name and email for users table
    - `t.timestamps null: false`, create two columns for users table, `created_at` and `updated_at`

- Running migration
  - `bundle exec rake db:migrate`
  - {% gist 94ac6c6d6a2ad511a734 schema.rb %}
  - Explanation
    - migration alters the database schema, the `schema.rb` file
    - migration are *reversible*
      - `bundle exec rake db:rollback`
        - will execute the `drop_table`, since `change` method knows `drop_table` is the inverse of `create_table`
        - in the case of an irreversible migration, such as one to remove a database column, it is necessary to define separate `up` and `down` methods in place of the single `change` method

------------

The CRUD:

- use `rails console --sandbox` to start CRUD without side effects
- Create
  - `first_user = User.new(name: "Superman", email: "superman@gmail.com")`
  - `User.new` only creates an object in memory, `first_user.save` saves to database
  - Combine them into one step with `User.create(name: "Superman", email: "superman@gmail.com")`
- Read
  - `User.find(1)` find user with id 1
  - `User.find_by(email: "superman@gmail.com")` find user with superman@gmail.com as email
  - `User.all` return all users
  - `User.first` return first
- Update
  - `first_user.email = "siray@gmail.com"` update email attribute for first_user
  - Be sure to use `first_user.save` to save changes into databse
  - `first_user.update_attributes(name: "doom", email: "doom@gmail.com")` combine them into one step
- Delete
  - `first_user.destroy`
- Helper method
  - `first_user.validate?` run all the validations within the specified context for first_user
  - `first_user.reload` reload first_user based on the current database information

------------

The Model Validation:
- Use TDD for model validations
  - `user_test.rb` from model generation
  - {% gist 94ac6c6d6a2ad511a734 user_test.rb %}
  - Explanation
  - `setup` do test initialization, automatically gets run before each test
  - write a test to make sure the initial model object is valid. This way, when the validation tests fail we’ll know it’s  not because the initial object was invalid
- Presence, Length, Format, Uniqueness
  - the test cases
  - {% gist 94ac6c6d6a2ad511a734 user_test_v2.rb %}
  - Explanation
  - idea: set one of its attributes to something we want to be invalid, and then test that it in fact is invalid
  - `assert_not` assert something should not be true
  - `%w` convenient constructor for array of strings, like `%w` in perl
  - `.dup` make a copy of object in memory
  - the validation in model
  - {% gist 94ac6c6d6a2ad511a734 user.rb %}
  - *Be aware the uniqueness here does not guarantee uniqueness at database level*, add `index` to email attribute

-----------------

- Michael Hartl's [rails tutorial](https://www.railstutorial.org/book/modeling_users)
- Rails [API docs](http://api.rubyonrails.org/)
- Rails [Guides docs](http://edgeguides.rubyonrails.org/active_record_migrations.html)
