---
layout: post
title: Testing in isolation, example in RABL
---

SOLID, Depending Inversion principle, testing doubles, presenters,
decorators, composites, oh my!

Now that I have your attention with a bunch of fancy words, let's apply
them to a real problem. My specific problem being that most of the
recent issues for [RABL](https://github.com/nesquena/rabl) have been
"limitations" on how RABL supports syntax for multiple variables and
conditions. The problem basically boils down to ignoring the SOLID
principles.

#### RABL

If you're not familiar with RABL, it's a template rendering engine
similar to ERB but tuned for structured outputs like JSON and XML.

RABL allows you to have a main object that everything is built around.
However, there are pull requests for [passing](https://github.com/nesquena/rabl/pull/269)
[locals](https://github.com/nesquena/rabl/pull/297) and passing objects to
[allow conditionals within blocks](https://github.com/nesquena/rabl/pull/300).
There's also a [slew of issues](https://github.com/nesquena/rabl/issues/search?q=presentation+or+decorator)
that mostly deal with syntax problems for making RABL do things you
shouldn't be allowing your views to do in the first place.

Here's an example:

{% highlight ruby %}
# post_controller.rb
def show
  @post = Post.find(params[:id])
end

# show.rabl
object @post

attributes :title, :body

child :author do
  # root_object is a hack for accessing @post
  attribute :name => :author_name unless root_object.author == current_user
end

node :publication_date do |post|
  if post.status == 'published'
    post.published_date
  end
end
{% endhighlight %}

#### Presenters, Decorators and Composites

If you're not sure what the differences between the wrappers are, I highly
suggest reading an article by Thoughtbot that covers
[Decorators compared to Strategies, Composites, and Presenters](http://robots.thoughtbot.com/post/20964851591/decorators-compared-to-strategies-composites-and).
I'm going to combine two of those to make a presentable decorator:

{% highlight ruby %}
# post_presenter.rb
class PostPresenter < Draper::Base
  decorates :post
  attr_reader :current_user

  def initialize(post, current_user)
    super(post)
    @current_user = current_user
  end

  def author_name
    author.name unless author == @current_user
  end

  def publication_date
    publication_date if status == 'published'
  end
end

# post_controller.rb
def show
  @post = PostPresenter.new(Post.find(params[:id]), current_user)
end

# show.rabl
object @post

attributes :title, :body, :author_name, :publication_date
{% endhighlight %}

Oh my, look at how easy that view is now!

If the attributes are nil, RABL will simply skip over them.
It's like your own free Null Object.
[Easy explanation of Null Object](http://devblog.avdi.org/2011/05/30/null-objects-and-falsiness/)
and a [couple](http://robots.thoughtbot.com/post/20907555103/rails-refactoring-example-introduce-null-object)
[others](http://robots.thoughtbot.com/post/12179019201/design-patterns-in-the-wild-null-object) just in case.)

If you don't want to use Draper, you can just break the
[Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter) a little
and go with a plain object that requires accessing the instance
variables through the presenter:

{% highlight ruby %}
class PostPresenter
  attr_reader :post, :current_user

  def initialize(post, current_user)
    @post = post
    @current_user = current_user
  end
end
{% endhighlight %}

Or you can use [four other methods of decorating in Ruby](http://robots.thoughtbot.com/post/14825364877/evaluating-alternative-decorator-implementations-in).

#### Dependency Inversion (the D in SOLID) used for testing

Now that I don't have logic embedded within my view, I actually don't
even have to unit test them. I should still have my controller verify
that the proper template was rendered and I need some basic level of
integration testing but overall, I can skip view rendering.

To test my "RABL", I actually test my presenter. To test my presenter, I
use Dependency Inversion to supply my presenter with the objects I need
it to test with.

Gregory Brown talks about Dependency Inversion in
[Issue #23: SOLID Design Principles of Ruby Best Practices](http://blog.rubybestpractices.com/posts/gregory/055-issue-23-solid-design.html).
The problem with his examples and my example above is we're actually
demonstrating Dependency Injection and not Inversion. An article on
RubySource titled [SOLID Ruby: Dependency Inversion Principle](http://rubysource.com/solid-ruby-dependency-inversion-principle/)
does a better job of explaining the differences between the two. It
basically boils down to initializing the object with your dependencies
versus calling methods with your dependencies passed in. Either way,
it's the same damn thing. You're abstracting the interface between
different implementations which makes it easier to change and test.

#### Test Doubles (Stubs and Mocks)

This is ultimately the entire point of this article. You have
successfully segregated your dependency on multiple instance variables,
random helper methods and anything else floating around in your scopes.
When a view refers to `foo` or `bar`, you can look at the presenter to know
exactly where that is coming from.

It also makes it possible to use stubs and mocks to easily swap out
parts of the presenter for testing. You don't have to actually have a
`current_user`, just `mock(:current_user)`. You can also really speed up
your tests by not using fixtures or factories to build out your user or
post. Have post be `Post.new` with a stub for the method you're about to
test.

[RABL Issue #299](https://github.com/nesquena/rabl/issues/299) asks:

    ... we want to use current_user in our renderer ...
    ... it doesn't cover having, say, a current_user method in use ...

Now that's easy. Pass in whatever current_user you want to use.

