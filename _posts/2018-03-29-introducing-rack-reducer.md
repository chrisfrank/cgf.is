---
title: "Dynamically filter data via URL params with Rack::Reducer"
published: true
description: "Rack::Reducer is a new gem for connecting URL params to functions that filter data, in any rack app."
tags: ruby, rails, rack, filtering
---

Whether you're building a tiny API or a giant monolith, your app probably needs to render a list of database records somewhere. And you probably want those records to be filterable via URL params, like so:

`GET /artists?genre=electronic&sort=name`

It'd be nice to sort or filter when the relevant params are present, and return a sensible default dataset otherwise.

```
GET /artists?name=blake` => artists named 'blake'
GET /artists?genre=electronic&sort=name => electronic artists, sorted by name
GET /artists => all artists
```

You _could_ conditionally apply filters with hand-written `if` statements, but that approach gets uglier the more filters you add.

In Rails, you could use Plataformatec's venerable [HasScope][has_scope] gem.

But what if you're working in Roda, Sinatra, or Hanami? Even in Rails, what if you'd rather write dedicated query objects than pollute your models and controllers with filtering code?

[Rack::Reducer][reducer] can help. It's a gem that maps incoming URL params to an array of filter functions you define, applies only the applicable filters, and returns your filtered data. It can make your controller logic as minimal as... 

```ruby
@artists = Artist.reduce(params)
```

...or, if magical, implicit code isn't your thing, you can call Rack::Reducer as a function to build more explicit queries -- maybe right in your controllers, maybe in dedicated [query objects][query_obj], or really anywhere you like.

```ruby
# app/controllers/artists_controller.rb
class ArtistsController < ApplicationController
  def index
    @artists = Rack::Reducer.call(params, dataset: Artist.all, filters: [
      ->(name:) { where('lower(name) like ?', "%#{name.downcase}%") },
      ->(genre:) { where(genre: genre) },
      ->(sort:) { order(sort.to_sym) },
    ])
    render json: @artists
  end
end
```

Rack::Reducer works in any Rack-compatible app, with any ORM, and has no dependencies beyond Rack itself. Full documentation is [on GitHub][reducer]. Whether you work in Rails, [Sinatra][sinatra], [Roda][roda], [Hanami][hanami], or raw Rack, I hope it can be of use to you.

[reducer]: https://github.com/chrisfrank/rack-reducer
[query_obj]: https://robots.thoughtbot.com/using-yieldself-for-composable-activerecord-relations
[has_scope]: https://github.com/plataformatec/has_scope
[rails]: http://rubyonrails.org
[sinatra]: http://sinatrarb.com
[roda]: http://roda.jeremyevans.net
[hanami]: http://hanamirb.org
