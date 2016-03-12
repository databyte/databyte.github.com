---
layout: post
title: Use the debugger, not puts
---

I'm going over the backlog of [Ruby Rogues](http://rubyrogues.com)
and I came across some very interesting comments in
[Episode 7 on Debugging in Ruby](http://rubyrogues.com/debugging-in-ruby/).
There are indeed two types of people, those that use puts and those that
use the debugger.

I find it hilarious that Charles has to convince everyone else that
using the debugger isn't harder than puts. Almost everyone else uses
puts instead of debugging and I find that interesting. My personal
opinion is that most people find puts easier because they're actually
uncomfortable or unfamiliar with debugger.

#### More than puts

Debugger is just as easy to use as puts. In fact, I would say debugger
is easier because if you guessed wrong on which object to print out, you
can try another and another until you find it.  The key isn't that the
debugger is just an interactive console to constantly print out objects,
it can do so much more.

Printing out statements into a log file works well when you have a lot
of data to process. However, if you're trying to track down a specific
bug or just trying to get your code to run right, the best option has to
be the debugger. You simply don't know what to print out and you're just
guessing along the way.

#### Why you should still use puts

There is however a place where the debugger doesn't work well and you do
print out a lot of data into log files, that's with threading. Threading
is much harder to track down problems with when using a single
interactive debugger. The C/C++, .NET and Java developer in me knows how
hard it is to get the debugger working just right and especially in
threading issues, just simply print out a lot of statements can be
easier. (Though the debugger for threading in .NET is one of the best
ones out there.)

Don't make that be the reason to ignore it in Ruby. Treat the debugger
more like how you would try to work with an assembler application.
There's less information to process out of a simpler application.

#### Let's do this!

If you're ready, we have to setup our basic environment. Don't bother
with `ruby-debug` and instead just use `debugger` which is
[available on GitHub](https://github.com/cldwalker/debugger).

    gem install debugger

    # or in your Gemfile
    gem 'debugger'

In your `~/.rdebugrc` file, you have to set:

    set autolist
    set autoeval
    set autoreload

I also have in my config:

    set forcestep

Instead of `puts @foo`, just use a debugger call. I won't go into the
details of using the debugger since
[Pivotal Labs wrote a good HOWTO](http://pivotallabs.com/users/chad/blog/articles/366-ruby-debug-in-30-seconds-we-don-t-need-no-stinkin-gui-)
and there's a [rdebug cheat sheet](http://cheat.errtheblog.com/s/rdebug/).

#### I HAZ DEBUGGER!

As with [my last blog article](/2012/07/27/testing-in-isolation-example-with-rabl.html),
I'm going to use a few examples of debugging in [RABL](https://github.com/nesquena/rabl).

In [Issue #249](https://github.com/nesquena/rabl/issues/249#issuecomment-5893385),
I explain using debugger to poke at your object:

``` ruby
glue :user do
  attributes :username => :author_name

  node :debug_me do
    debugger
  end
  child :phone_numbers => :pnumbers do
    extends "users/phone_number"
  end
end

# pops out:

(rdb:1) v l
  block => #<Proc:0x007fed82628488@/Users/databyte/projects/popular/rabl/fixtures/rails3_2/app/views/posts/show.rabl:14>
  name => :debug_me
  options => {}
  result => 1
(rdb:1) @_object
#<User id: 1924, username: "billybob", email: "billy@bob.com", location: "SF", is_admin: false, created_at: "2012-05-24 05:51:59", updated_at: "2012-05-24 05:51:59">
(rdb:1) @_object.phone_numbers
[#<PhoneNumber id: 2565, user_id: 1924, is_primary: true, area_code: "222", prefix: "000", suffix: "6666", name: "Home">, #<PhoneNumber id: 2566, user_id: 1924, is_primary: false, area_code: "222", prefix: "000", suffix: "6666", name: "Work">]
```

In [Issue #243](https://github.com/nesquena/rabl/issues/243#issuecomment-5943163),
I explain using `source_location` to find where a method is being
defined but first we have to debug directly into the RABL gem.

``` ruby
subl `bundle show rabl`
# or whatever editor you use
mvim `bundle show rabl`
mate `bundle show rabl`

# or if your editor is set right:
bundle open rabl
```

And once you have RABL open - put a debugger statement in and restart
your application since external libraries are not reloaded. Using the
debugger here is a loads more efficient than placing puts everywhere
and constantly restarting your application.

``` ruby
[193, 202] in /Users/databyte/projects/popular/rabl/lib/rabl/engine.rb
   193      # Returns a guess at the default object for this template
   194      # default_object => @user
   195      def default_object
   196        if context_scope.respond_to?(:controller)
   197          debugger
=> 198          controller_name = context_scope.controller.controller_name
   199          stripped_name = controller_name.split(%r{::|\/}).last
   200          instance_variable_get("@#{stripped_name}")
   201        end
   202      end
/Users/databyte/projects/popular/rabl/lib/rabl/engine.rb:198
controller_name = context_scope.controller.controller_name

(rdb:1) context_scope.controller.method(:controller_name).source_location
["/Users/databyte/.rvm/gems/ruby-1.9.3-p194-perf/gems/actionpack-3.2.3/lib/action_controller/metal.rb", 121]
```

Use your debugger and get your Ruby-fu in order then abuse what Ruby gives
you.
