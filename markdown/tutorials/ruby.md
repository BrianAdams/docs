 {
  title: "Ruby",
  description: "How to run Selenium tests on Sauce Labs using Ruby",
  category: 'Tutorials',
  index: 1,
  image: "/images/tutorials/ruby.png"
}

The [sauce gem](https://github.com/saucelabs/sauce_ruby) makes it easy to run Selenium or Capybara tests against a wide range of browsers on Windows (XP, 7, 8), OS X, Linux, iOS and Android.

This example uses [Capybara](http://jnicklas.github.com/capybara/) and RSpec with Rails 3 and Ruby 1.9, but Sauce Labs also works great against any Ruby web stack, and with [Test::Unit](https://saucelabs.com/docs/ondemand/getting-started/env/ruby/se2/mac), [Cucumber](https://github.com/sauce-labs/sauce_ruby/wiki/Cucumber-and-Capybara), and most other testing frameworks... right down to vanilla [WebDriver](http://code.google.com/p/selenium/wiki/RubyBindings).

Your tests are run in real browsers on a real operating system, in a
dedicated, single-use VM.  Once they're complete, screenshots, video,
Selenium log and a log of passes and failures can be seen and shared.


What You'll Need
----------------

In your Gemfile:

```ruby
group :test, :development do
  # These are the target gems of this tutorial
  gem 'rspec-rails', '~> 2.12'
  gem 'sauce', '~> 3.1.1'
  gem 'sauce-connect'
  gem 'capybara', '~> 2.0.3'
  gem 'parallel_tests'
end
```

Setting up RSpec
-----------

From your `RAILS_ROOT`, generate a ./spec directory, a ./spec/spec_helper.rb file, and a warm, fuzzy feeling of productivity by executing:

```bash
rails generate rspec:install
```

Set up your `spec_helper` and create a template `sauce_helper`, by running:

```bash
rake sauce:install:spec
```

Now, open the new spec/sauce_helper.rb file, and just under the `require` statements, add requires for `capybara/rails` and `capybara/rspec`. Finally, set the default Capybara driver to be Sauce:

```ruby
Capybara.default_driver = :sauce
```

Once this is done, you can configure your desired test platforms. Here's an example:

```ruby
# Use Capybara integration
require "sauce"
require "sauce/capybara"
require "capybara/rails"
require "capybara/rspec"

Capybara.default_driver = :sauce

# Set up configuration
Sauce.config do |c|
  c[:browsers] = [
    ["Windows 8", "Internet Explorer", "10"],
    ["Windows 7", "Firefox", "20"],
    ["OS X 10.8", "Safari", "6"],
    ["Linux", "Chrome", nil]
  ]
end
```

Check out [our platforms page](http://saucelabs.com/docs/platforms) for available platforms (130+ and counting!).


Setting up the Sauce Gem
-------------------------

Keep your Sauce Labs credentials out of your repositories and available to all your Sauce Labs tools using projects by adding them as environment variables.

Open `~/.bash_profile` and add the following lines:

```bash
export SAUCE_USERNAME=sauceUsername
export SAUCE_ACCESS_KEY=sauceAccessKey
```

You'll then need to re-load that profile with `source ~/.bash_profile`

Open your environment variables settings window (Instructions [here](http://www.itechtalk.com/thread3595.html)) and set the following variables:

    Name: SAUCE_USERNAME
    Value: sauceUsername

    Name: SAUCE_ACCESS_KEY
    Value: sauceAccessKey

Writing your tests
-----------------

Phew!  That's all your setup done.  You're ready to write your tests.

Because we want the Capybara DSL included, we're going to put our tests in
the spec/features directory.

Turn on the Sauce voodoo by tagging each describe block ('example group' in RSpec-lish)  with `:sauce => true` like this:

    $ mkdir ./spec/features
    $ vim ./spec/features/ramen_spec.rb

```ruby
require "spec_helper"

describe "Wikipedia's Ramen Page", :sauce => true do
  it "Should mention the inventor of instant Ramen" do
    visit "http://en.wikipedia.org/"
    fill_in 'search', :with => "Ramen"
    click_button "searchButton"
    page.should have_content "Momofuku Ando"
  end
end
```
That's one spec, how about another?

    $ vim ./spec/features/miso_spec.rb

```ruby
require "spec_helper"

describe "Wikipedia's Miso Page", :sauce => true do
  it "Should mention a favorite type of Miso" do
    visit "http://en.wikipedia.org/"
    fill_in 'search', :with => "Miso"
    click_button "searchButton"
    page.should have_content "Akamiso"
  end
end
```

And that's everything!

Running your tests
------------------

```bash
rake sauce:spec
```

It's that simple (Thanks in part to the excellent [parallel_tests](https://github.com/grosser/parallel_tests) gem.)
Your tests will run once for every platform, taking advantage of Sauce Labs to run at the maximum concurrency for your
account. The sauce:spec rake command takes two optional parameters to let you control the directory you keep your specs
in and the level of concurrency at which you run them. For example, to run specs from the "spec" directory one at a time,
you'd run the command like so: 

```bash
rake sauce:spec[spec,1]
```

You should see output much like the following:

```
20 processes for 8 specs, ~ 0 specs per process
[Connecting to Sauce Labs...]
Sauce Connect 3.0-r25, build 38

[snip]A LOT OF TEST DATA[/snip]

8 examples, 0 failures

Took 196.657036 seconds
```

The `8 examples, 0 failures` line means your tests are running against each browser, and passing. Congratulations!

Running in parallel makes your build faster, so you can run more tests in more browsers in less total time.

Check out the results, including a command log, screenshots, and video of the browser executing the test, on your [account page](https://saucelabs.com/account).

Help, it didn't work!
---------------------

Make sure your example groups are tagged with `:sauce => true`.  Without this tag, the RSpec integration won't be used, and your tests will only run with the default Sauce configuration.

If you still can't get things going, drop us a line at help@saucelabs.com and we're happy to assist you.  Don't forget to include your Gemfile.lock!

What's Next?
------------
**Capybara Resources**

Now that you have an example to work with, it's time to write a test for your web app! For more info on how to write Capybara tests, we recommend the excellent [Capybara README](https://github.com/jnicklas/capybara).

**Testing against local servers with Sauce Connect**

If you need to test a staged site behind your firewall, that's no problem: you're already set up to use [Sauce Connect](http://saucelabs.com/docs/connect), which means local servers are available to your Sauce Labs tests.

If you don't need access to local servers, you can turn Sauce Connect off by adding this to your Sauce.config block:

```ruby
  config[:start_tunnel] = false
```

**parallel_tests considerations**

If you find that you run into problems with all your tests using the same db at once, or otherwise need some careful setup, the
[parallel_tests instructions](https://github.com/grosser/parallel_tests)
have useful info.

**The Sauce Gem**

The Sauce gem has its own [github repo](https://github.com/saucelabs/sauce_ruby) and a constantly evolving [wiki](https://github.com/saucelabs/sauce_ruby/wiki/_pages) you should check out!
