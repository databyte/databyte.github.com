---
layout: post
title: Splitting out your marketing pages from your main web application
---

I read an article last month on
[Adding static pages to your Rails app](https://christoph.luppri.ch/articles/2016/09/25/adding-static-pages-to-your-rails-app/)
and I've been meaning to write up how we do the same thing at
[Clockwise.MD](https://www.clockwisemd.com). In fact, it's been a very long time
since I've written on my blog.

It's funny that I'm finding time to write in the middle of a hackathon at
[48-in-48](https://atlanta.48in48.org/). After joining three teams, it seems I've
completed everything on my list and I'm waiting on Stephanie to wrap up. So that
must mean it's a great time to start writing again and create a blog entry!
Hopefully I can write up something about this experience when I'm done.

Back to static pages for your web application.

The benefit of splitting out your main app from your static marketing pages is
creating an enviornment the best suits your authors/editors, designers and engineers.
Take Clockwise.MD for instance, we have a well defined SDLC process for making
changes to the web application. It includes testing, peer review, approvals and
carefully selecting which memoral quote, interesting fact or animated GIF to add
to your PR approval. The flip side is our head of marketing shouldn't need an
extensive PR process to add a new [pet to our Team page](https://www.clockwisemd.com/about).

Similar to the earlier article I mentioned, we needed to load static pages up
for our app but we don't want to host those files on our actual web server.
Instead, we created our web site using [Jekyll](https://jekyllrb.com/). The
resulting static site is uploaded to S3 with a CDN in front to make the site
load fast. Then any files we don't want to serve up from our web application,
we proxy down from our CDN and send it to the requestor. We also cache it so
that it's super fast.

```ruby
class PagesController < ApplicationController
  def www_base
    expires_in 1.hour, public: true
    response = s3_pages(request.path)
    render status: response.code, inline: response.body
  end

  private

  def s3_pages(path)
    uri = URI('https://your-s3-or-cdn-site.com' + adjusted_path(path))
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    http.ciphers = 'TLSv1.2:!aNULL:!eNULL'
    http.ssl_version = :TLSv1_2
    http.verify_mode = OpenSSL::SSL::VERIFY_PEER
    request = Net::HTTP::Get.new(uri.request_uri)
    http.request(request)
  end
end
```

Then at the bottom of your `routes.rb` file, add:

```ruby
  # external pages
  root to: 'pages#www_base'
  get '*unmatched_route', to: 'pages#www_base'
```

Now your home page will hit `pages#www_base`. Additionally, any page that's
not already specified earlier on your web application's routes will be routed
to your static site. If a user hits `/products`, then that same request will
get served up from your static site. If a user hits `/foo/bar` and that path
doesn't exist, then they'll even get your static site's 404 page and `www_base`
will send back the correct 404 status code as well.

To make things easier for your marketing folks, let them edit pages using
Github's online editors then setup a CI process to rebuild and redeploy the
static site on every commit. Instant updates without having to the install
the Wordpress malware.

