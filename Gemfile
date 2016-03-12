source 'https://rubygems.org'

# core
# gem 'jekyll'
require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)
gem 'github-pages', versions['github-pages']

# addons
gem 'jekyll-paginate'

# development
gem 'foreman'

gem 'ruby_gntp' # notification
gem 'guard'
gem 'guard-jekyll'
