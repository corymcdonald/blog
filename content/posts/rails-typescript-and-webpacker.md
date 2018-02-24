---
title: "Rails, Typescript, and Webpacker"
date: 2018-02-24T11:37:08-06:00
---

I'm a big proponent of Typescript. It has some great features like [type saftey](https://www.typescriptlang.org/docs/handbook/basic-types.html), [classes](https://www.typescriptlang.org/docs/handbook/classes.html), [decorators](https://www.typescriptlang.org/docs/handbook/decorators.html), and [sophisticated editor integration](https://code.visualstudio.com/docs/languages/typescript). With the webpacker gem adding typescript to your project is extremely simple.

## Installing Webpacker 3.2

You'll find the most [up to date instructions on the repository README](https://github.com/rails/webpacker#installation), however I'll walk you through the steps as they exist at the time of writing.

The easiest way is to start a new project.

`rails new my_application --webpack`

However, since that's not always feasible **you can add it to an existing project**.

```ruby
# Gemfile
gem 'webpacker', '~> 3.2'
```

and then run

```bash
bundle
bundle exec rails webpacker:install

# OR (on rails version < 5.0)
bundle exec rake webpacker:install
```


## Installing typescript
You've got the gem installed so now let's install typescript


```bash
bundle exec rails webpacker:install:typescript

# OR (on rails version < 5.0)
bundle exec rake webpacker:install::typescript
```

## Using it
Once you've run the installation command you won't be able to actually use typescript until you include the webpacker helper tags in your project.

```
<%= javascript_pack_tag 'application' %>
<%= stylesheet_pack_tag 'application' %>
```

It's important to note that `application` is referring to the file in the `app/javascript/packs` folder.

In the case of typescript there will be a `hello_typescript.ts` file that is generated that you can refer to that to ensure your project works.

Another thing that might throw you off is that if you don't have a controller or view and simply run the server it the "Welcome to Rails" default landing page doesn't load the `application.html.erb` page.

If you stick the pack tags in `application.html.erb` page and are simply using the landing page you will not see any console output.

1.
For **new applications** this just means you need to generate a dummy controller in order to see the result.

`rails g controller home`

2.
Modify the newly created controller

```ruby
class HomeController < ApplicationController
  def index
  end
end
```

3.
 Modify the routes

```ruby
#config/routes.rb
Rails.application.routes.draw do
  get 'home/index'
end
```

4.
Create an index page - `app/views/home/index.html.erb`

```
# app/views/home/index.html.erb
<%= javascript_pack_tag 'hello_typescript' %>
<%= stylesheet_pack_tag 'hello_typescript' %>
```

5.
Load the <a href="localhost:3000/home/index">localhost:3000/home/index</a> page.


## React, Vue, and Angular

Webpacker has a great guide on how to use typescript with your [UI framework of choice](https://github.com/rails/webpacker/blob/master/docs/typescript.md). There's detailed steps for React, Vue, and Angular.
