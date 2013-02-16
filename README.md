# Flipper Redis

A [Redis](https://github.com/redis/redis-rb) adapter for [Flipper](https://github.com/jnunemaker/flipper).

## Installation

Add this line to your application's Gemfile:

    gem 'flipper-redis'

And then execute:

    $ bundle

Or install it yourself with:

    $ gem install flipper-redis

## Usage

```ruby
require 'flipper/adapters/redis'
client = Redis.new
adapter = Flipper::Adapters::Redis.new(client)
flipper = Flipper.new(adapter)
# profit...
```

## Internals

Each feature is stored in a redis hash, which means getting a feature is single query.

```ruby
require 'flipper/adapters/redis'
require 'redis/namespace'

client = Redis.new
namespaced_client = Redis::Namespace.new(:flipper, :redis => client)
adapter = Flipper::Adapters::Redis.new(namespaced_client)
flipper = Flipper.new(adapter)

# Register a few groups.
Flipper.register(:admins) { |thing| thing.admin? }
Flipper.register(:early_access) { |thing| thing.early_access? }

# Create a user class that has flipper_id instance method.
User = Struct.new(:flipper_id)

flipper[:stats].enable
flipper[:stats].enable flipper.group(:admins)
flipper[:stats].enable flipper.group(:early_access)
flipper[:stats].enable User.new('25')
flipper[:stats].enable User.new('90')
flipper[:stats].enable User.new('180')
flipper[:stats].enable flipper.random(15)
flipper[:stats].enable flipper.actors(45)

flipper[:search].enable

print 'all keys: '
pp namespaced_client.keys
# all keys: ["stats", "flipper_features", "search"]

print "known flipper features: "
pp namespaced_client.smembers("flipper_features")
# known flipper features: ["stats", "search"]

puts 'stats keys'
pp namespaced_client.hgetall('stats')
# stats keys
# {"boolean"=>"true",
#  "groups/admins"=>"1",
#  "actors/25"=>"1",
#  "percentage_of_random"=>"15",
#  "percentage_of_actors"=>"45",
#  "groups/early_access"=>"1",
#  "actors/90"=>"1",
#  "actors/180"=>"1"}

puts 'search keys'
pp namespaced_client.hgetall('search')
# search keys
# {"boolean"=>"true"}

puts 'flipper get of feature'
pp adapter.get(flipper[:stats])
# flipper get of feature
# {:boolean=>"true",
#  :groups=>#<Set: {"admins", "early_access"}>,
#  :actors=>#<Set: {"25", "90", "180"}>,
#  :percentage_of_actors=>"45",
#  :percentage_of_random=>"15"}
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
