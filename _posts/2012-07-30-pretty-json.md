---
layout: post
title: Pretty JSON
---

Here are some options for pretty printing your JSON:

* Copy/Paste - Without installing anything, just paste it into [JSONLint](http://jsonlint.com/)

* Ruby - Parse the JSON and then pp the resulting hash

* Browser - Install [JSON plugin for Chrome](https://chrome.google.com/webstore/detail/chklaanhfefbnpoihckbnefhakgolnmc?hl=en-US)

* Sublime - Install 'Pretty JSON' from [Package Control](http://wbond.net/sublime_packages/package_control)
([source](https://github.com/dzhibas/SublimePrettyJson))

* Curl - Use *cough* Python *cough*:

``` ruby
# in Ruby
JSON.parse(your_json)
```

``` bash
# command line
curl http://api.twitter.com/1/statuses/public_timeline.json | python -mjson.tool
```

Happy parsing!
