---
layout: post
title: How to Set Up a New Rails Project with Minitest and Capybara for Behaviour Driven Development
published: true
---

## Getting Started

Here are step-by-step instructions on setting up a new rails app with Minitest and Capybara. We'll be using a rails 4.1+ in this example. Lets get started!

First, create a new rails application, a git repository, and the first commit to the repo.

{% highlight bash %}
rails new awesomeapp
cd awesomeapp
git init
git add -A
git commit -m "Initial Commit"
{% endhighlight %}

Next, open the `Gemfile` and add the [minitest-rails](https://github.com/blowmage/minitest-rails) and [minitest-rails-capybara](https://github.com/blowmage/minitest-rails-capybara) gems. Note the `minitest-rails-capybara` needs to be grouped under `test`, while `minitest-rails` does not.

{% highlight ruby %}
gem "minitest-rails"

group :test do
  gem "minitest-rails-capybara"
end
{% endhighlight %}

Then you're going to bundle your `Gemfile` and run the minitest-rails installation generator. This will overwrite the test_helper.rb that is in the `test` directory.  Press `y` when prompted to overwrite this file.

{% highlight bash %}
bundle install
rails generate minitest:install
{% endhighlight %}

## Configuration

Open up `test_helper.rb` and uncomment `require "mintitest/rails/capybara"` and optionally `minitest/pride`. The first required statement adds Capybara for our feature tests and the second adds awesome colors to the test suite (always a bonus!). At this point you can remove the comments, as they are no longer needed. Your `test_helper.rb` should look similar to the one below.

{% highlight ruby %}
ENV["RAILS_ENV"] = "test"
require File.expand_path("../../config/environment", __FILE__)
require "rails/test_help"
require "minitest/rails"
require "minitest/rails/capybara"
require "minitest/pride"

class ActiveSupport::TestCase
  ActiveRecord::Migration.check_pending!
  fixtures :all
end
{% endhighlight %}

There are a few ways to generate tests. You can choose either TestCase or Spec DSL to write the tests. [This article](http://www.mattsears.com/articles/2011/12/10/minitest-quick-reference) by Matt Sears is a great quick reference to both of these DSLs. While it is possible to remember to put the `--spec` flag on the generator commands, it is also possible to set these defaults in the `config/application.rb`.

{% highlight ruby %}
config.generators do |g|
  g.test_framework :minitest, spec: true, fixture: true
end
{% endhighlight %}

## Adding a runner

Now add a `features` rake task (since we'll be writing feature specs) as the default test task to run. Create a new file named `features.rake` in the `lib/tasks/` folder and add the following:

{% highlight ruby %}
Rails::TestTask.new("test:features" => "test:prepare") do |t|
  t.pattern = "test/features/**/*_test.rb"
end

Rake::Task["test:run"].enhance ["test:features"]
{% endhighlight %}

## Writing our first feature spec

Now that Minitest and Capybara are setup, lets commit the changes to Git and then create a new `feature` spec.

{% highlight bash %}
git add -A
git commit -m "Add Minitest-Rails and Minitest-Rails-Capybara gems and configure for Spec DSL"
rails generate minitest:feature HelloWorldDisplayedOnHomePage
{% endhighlight %}

Inspecting the `test/features` directory, you'll find that a new file has been created called `hello_world_displayed_on_home_page_test.rb`. Open this and take a look at the generated boilerplate spec that has been created.

{% highlight ruby %}
require "test_helper"

feature "HelloWorldDisplayedOnHomePage" do
  scenario "the test is sound" do
    visit root_path
    page.must_have_content "Hello World"
    page.wont_have_content "Goobye All!"
  end
end
{% endhighlight %}

To run the newly-created test, simply run `rake` in the terminal and watch the newly-generated test fail with an error.

{% highlight ruby %}
Run options: --seed 41493 
# Running:

E

Fabulous run in 0.011996s, 83.3611 runs/s, 0.0000 assertions/s.

1) Error:
HelloWorldDisplayedOnHomePage Feature Test#test_0001_the test is sound:
NameError: undefined local variable or method `root_path' for #<#<Class:0x007fbc7e44bf00>:0x007fbc7e402300>
test/features/hello_world_home_page_test.rb:5:in `block (2 levels) in <top (required)>'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
{% endhighlight %}
Thats it! We now have Rails, Minitest, and Capybara set up for BDDing your new app.  Happy red-green refactoring!!!'`