---
layout: post
title: Don't confuse popular or easiest with best
---

I recently bumped into someone who proclaimed PHP as the greatest
language for web development and he had zero desire to learn another
framework, especially not Ruby on Rails.

I, for one, couldn't care less if you used Ruby or Rails, Python or
Django, C# or ASP.NET. I do care that you would a) only stick with what
you know and b) choose PHP.

For the former, it's simple - you should assume you simply don't know
everything there is to know and continually try to better yourself. If
you haven't reached that conclusion yet, the book
[The Passionate Programmer](http://pragprog.com/book/cfcar2/the-passionate-programmer)
will surely help.

For the latter, hmm, PHP eh?

I've done PHP development as recently as last year but to read into more
of a PHP'ers point of view, I recently read a blog post named
[PHP is much better than what you think](http://fabien.potencier.org/article/64/php-is-much-better-than-what-you-think).

In it, the author, Fabien Potencier, sounds just like the person I
mentioned earlier. The opening paragraphs calls PHP the easiest, most
popular and quickest language. All great reasons to pick one language
over another. So let's break this down.

#### The Language

PHP is old but instead of a language that moves forward with its syntax,
PHP chooses to build on top of the old cruft and never clears away the
cobwebs. Its syntax coherence is below most other languages. A blog titled
[PHP: a fractal of bad design](http://me.veekun.com/blog/2012/04/09/php-a-fractal-of-bad-design/)
says it better than anyone else:

> * PHP is full of surprises: `mysql_real_escape_string`, `E_ALL`
> * PHP is inconsistent: `strpos`, `str_rot13`
> * PHP requires boilerplate: error-checking around C API calls, `===`
> * PHP is flaky: `==`, `foreach ($foo as &$bar)`
> * PHP is opaque: no stack traces by default or for fatals, complex error reporting

Consistency is the key to discovering a new language, framework or API.
One of the key arguments for PHP is that it's easy. I don't see how
inconsistency is easy. From the same article:

> * Underscore versus not: `strpos`/`str_rot13`, `php_uname`/`phpversion`, `base64_encode`/`urlencode`, `gettype`/`get_class`
> * “to” versus 2: `ascii2ebcdic`, `bin2hex`, `deg2rad`, `strtolower`, `strtotime`
> * Object+verb versus verb+object: `base64_decode`, `str_shuffle`, `var_dump` versus `create_function`, `recode_string`
> * Argument order: `array_filter($input, $callback)` versus `array_map($callback, $input)`, `strpos($haystack, $needle)` versus `array_search($needle, $haystack)`
> * Prefix confusion: `usleep` versus `microtime`
> * Case insensitive functions vary on where the `i` goes in the name.
> * About half the array functions actually start with `array_`. The others do not.
> * `htmlentities` and `html_entity_decode` are *inverses* of each other, with completely different naming conventions.

Seriously read through that entire article, it's a great read.

#### The Ecosystem

The author mentions git but git is everywhere. Even Microsoft
[uses](https://github.com/WindowsAzure). [it](https://github.com/NuGet).
[now](https://github.com/MSOpenTech).

It's great to see PHP has a dependency manager but so does everyone
else. The fact that you have one and it works decently does not mean
it's better than another language or implementation. At this point, not
having a dependency manager is a problem. All modern frameworks have
them including
[RubyGems](https://rubygems.org/),
[Python PyPI](http://pypi.python.org/pypi),
[Node Package Manager (NPM)](https://npmjs.org/),
[Microsoft NuGet](http://nuget.org/), etc.

I'm not sure how the author judges that Composer is better than other
package managers. Is it because it resolves dependencies or has over
2400 packages? I don't know. The other managers do handle dependencies
as well and the package counts as of today are:

* Composer at 2,454
* RubyGems at 42,278
* PyPI at 22,860
* NuGet at 7,792

Under the original article, the author calls this community
collaboration but really he's making a point about the number of
packages available in PHP. Given the numbers, I expected a lot more from
PHP given the number of sites online and the years its been in service.

#### The Community

To me, the community is not about sharing code but more about the people
themselves. The developers make the community because they strive to
make everything better.  They don't sit in their own echo chambers and
pat each other on the back, talking about how great this is about their
preferred language or that is about their favorite framework. They see
the shortcomings and they strive to address them while acknowledging
their strengths and weaknesses.

The main issue I see with PHP but also with some other communities is
the lack of sharing. Not just code but technical advise, support and
overall camaraderie. Some communities are getting better at it. Having
spent years in the Microsoft and Java ecosystems, I've seen it shape up
better in Microsoft but that's because they have evangelists running
around doing their job. Java's "ok" but Oracle hasn't helped at all,
especially given recent lawsuits and Hudson vs Jenkins.

The JavaScript community is an interesting one and the developers within
it range from copy/paste scripters to those that thrive on functional
programming who have dived deep into [CoffeeScript](http://coffeescript.org/)
or [Dart](http://www.dartlang.org/) to know which OOP parts suck
compared to, say, [LiveScript](http://gkz.github.com/LiveScript/).

That wide variety of developers makes it harder to polarize the
community as a whole and makes it difficult for one great programmer to
recognize another. It's similar to PHP.

#### Conclusion

The language is not easy to learn if it's inconsistent. The community
around your language and framework really matters; it allows you to
learn and grow to become a passionate programmer.

Ultimately, I suggest that all developers should learn more than one
language and framework. Get out your bubble and expand your horizon.

You can then choose the right language and the right platform for the
right problem. Just as long as it's not PHP.

