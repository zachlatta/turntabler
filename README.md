# turntabler [![Build Status](https://travis-ci.org/zachlatta/turntabler.png?branch=master)](https://travis-ci.org/zachlatta/turntabler) [![Dependency Status](https://gemnasium.com/obrie/turntabler.png "Dependency Status")](https://gemnasium.com/obrie/turntabler) [![Coverage Status](https://coveralls.io/repos/zachlatta/turntabler/badge.png)](https://coveralls.io/r/zachlatta/turntabler)

*turntabler* is an evented Turntable.FM API for Ruby.

## Resources

API

* http://rdoc.info/github/obrie/turntabler/master/frames

Bugs

* http://github.com/obrie/turntabler/issues

Development

* http://github.com/obrie/turntabler

Testing

* http://travis-ci.org/obrie/turntabler

Source

* git://github.com/obrie/turntabler.git

## Description

Turntabler makes it dead-simple to interact with the Turntable.FM API.  It is
designed primarily as an all-purpose library with additional thoughts given to
the use of it for bots.  It is an opinionated library that attempts to hide the
various complexities and inconsistencies with the Turntable API by providing a
clean, fresh new perspective on how data is accessed and organized.

This project was built from the ground-up by Rubyists for Rubyists.  While prior
projects in other languages were used for guidance on some of the implementation,
the design is meant to take advantage of the various features offered by Ruby 1.9+.

At a high level, this project features:

* Evented, non-blocking I/O
* Fiber-aware, untangled callbacks
* Interactive console support
* Clean, object-oriented APIs
* Detailed API documentation
* 100% complete Turntable API implementation
* Lazy-loaded attributes
* Auto-reconnects for bots
* Consistent API / attribute naming schemes
* HTTP / Web Socket interface implementations
* Room state / user list management
* DSL syntax support
* Custom events

Turntable features include management of:

* User status / profile
* Site preferences
* Avatars
* Laptops / stickers
* Playlists
* Fans
* Buddies (Twitter / Facebook)
* Blocked users
* Private messages
* Advanced room search / listings
* Room favorites
* Room profiles
* Room chat
* Moderators
* DJs
* Booted users
* Song search
* Song snags
* Song voting
* User / room reporting

Examples of the usage patterns for some of the above features are shown below.
You can find much more detailed documentation in the actual API.

## Usage

### Example

Below is an example of many of the features offered by this API, including:

```ruby
require 'turntabler'

EMAIL = ENV['EMAIL']
PASSWORD = ENV['PASSWORD']

Turntabler.run do
  client = Turntabler::Client.new(EMAIL, PASSWORD)
  
  # Events
  client.on :user_entered do |user|
    puts "#{user.name} entered the room"
    user.become_fan   # => true
  end

  client.on :user_left do |user|
    puts "#{user.name} left the room"
  end

  client.on :user_spoke do |message|
    if message.content =~ /bop/
      client.room.current_song.vote   # => true
    end
  end

  client.on :dj_added do |user|
    puts "#{user.name} started DJing"
  end

  client.on :dj_removed do |user|
    puts "#{user.name} stopped DJing"
  end

  # Authorized user interactions
  user = client.user                                # => #<Turntabler::AuthorizedUser @email="ben.zelano@gmail.com" ...>
  user.fan_of                                       # => [#<Turntabler::User @id="d5616b31654e8b22a7a1eef0">, ...]
  user.fans                                         # => [#<Turntabler::User @id="d5616b31654e8b22a7a1eef0">, ...]
  user.playlist.songs                               # => [#<Turntabler::Song @album="Abbey Road" ...>, ...]
  user.blocks                                       # => [#<Turntabler::User @id="19125d4da3b09562b2cf68b6">, ...]
  user.buddies                                      # => [#<Turntabler::User @id="efff38aeb7b9334164c1b630">, ...]
  
  # Room Directory
  client.rooms.all(:favorites => true)              # => []
  client.rooms.all(:genre => :rock)                 # => [#<Turntabler::Room @id="4e4986bb14169c5f241318a6", ...>, ...]
  client.rooms.all(:genre => :rock, :available_djs => true, :minimum_listeners => 5)
  # => [#<Turntabler::Room @id="4f4a5874a3f75128aa006c17", ...>, ...]
  client.rooms.with_friends                         # => [#<Turntabler::Room @id="50b4c1e2df5bcf4af666f876", ...>, ...]
  client.room('4dff1eac14169c565800892e').listeners # => #<Set: {#<Turntabler::User @id="4e1341e2a3f75114d003c591" ...>, ...}>
  
  # Room interaction
  client.rooms.create("My Test Room #{rand}").enter # => true
  room = client.room                                # => #<Turntabler::Room @name="My Test Room 0.24300857307298018" ...>
  room.add_as_favorite                              # => true
  room.become_dj                                    # => true
  room.say "Hey guys!"                              # => true
  
  # User interaction
  listeners = room.listeners                        # => #<Set: {#<Turntabler::User @id="309ba75b6385b83e110923bd" ..., ...}>
  listeners.each do |listener|
    listener.messages                               # => [#<Turntabler::Message @content="Hey man!" ...>, ...]
    listener.website                                # => "http://mypersonalwebsite.com"
    listener.facebook_url                           # => "https://www.facebook.com/firstname.lastname"
    listener.sticker_placements                     # => [#<Turntabler::StickerPlacement @angle=0 ...>, ...]
    listener.say "Welcome to the room!"             # => true
  end
  
  # Playlist interaction
  client.user.playlists.all                         # => [#<Turntabler::Playlist @id="default" ...>, ...]
  client.user.playlists.create("rock")              # => #<Turntabler::Playlist @id="rock" ...>
  client.user.playlist                              # => #<Turntabler::Playlist @id="default" ...>
  client.user.playlist("rock").activate             # => true
  client.user.playlist("rock").songs                # => []
  
  # Songs
  songs = client.search_song('Rolling Stones')      # => [#<Turntabler::Song @album="Tattoo You (2009 Remaster)" ...>, ...]
  songs.each do
    song.add                                        # => true
  end
end
```

The example above is just a very, very small subset of the possible things you
can do with turntabler.  For a *complete* list, see the API documentation, especially:

* [Turntabler::AuthorizedUser](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/AuthorizedUser)
* [Turntabler::Client](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/Client)
* [Turntabler::Playlist](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/Playlist)
* [Turntabler::Room](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/Room)
* [Turntabler::RoomDirectory](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/RoomDirectory)
* [Turntabler::Song](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/Song)
* [Turntabler::User](http://rdoc.info/github/obrie/turntabler/master/frames/Turntabler/User)

For additional examples, see the [examples](https://github.com/obrie/turntabler/tree/master/examples)
directory in the repository.

### Custom events

In addition to the default Turntable events supported out of the box, turntabler
also allows you to define your own events.  This is particularly useful in cases
where you may want to provide extensions on top of the turntabler library for
others to use.  These extensions may be higher-order events, such as a user
reaching their maximum play count for a turn or a user timing out.

For example:

```ruby
require 'turntabler'

EMAIL = ENV['EMAIL']
PASSWORD = ENV['PASSWORD']

# Register custom events
Turntabler.events :user_greeted

Turntabler.run do
  client = Turntabler::Client.new(EMAIL, PASSWORD)
  
  # Events
  client.on :user_spoke do |message|
    if (message.content =~ /^\/hello$/)
      # Trigger the custom event
      client.trigger(:user_greeted, message.sender)
    end
  end

  # Handle custom event
  client.on :user_greeted do |user|
    client.room.say "Hey! How are you #{message.sender.name}?"
  end
end
```

## Additional Topics

### Differences with existing libraries

So you may be asking "Why another Turntable.FM API library?" or "Why re-build
this in Ruby when you have a stable Javascript project?"  Simply put, I felt that
many of the high-level details highlighted in the Description section of this
document were missing in existing libraries.

Some of those details include evented I/O, untangled callbacks, object-oriented
APIs, external API consistency, internal state management, auto lazy-loading, etc.
This library also strives to be a complete implementation, easy to use / play
around with, and generally just put together a more organized fashion.

By no means does this discredit the significance of the efforts put forth by the
authors of these existing libraries -- all of their work provided the foundation
necessary to build out this project.

### Authentication

By default, turntabler authenticates users with the e-mail address and password
associated with their account.  For example:

```ruby
TT.run do
  client = TT::Client.new(EMAIL, PASSWORD)
  # ...
end
```

However, older Turntable accounts do not have e-mail addresses or passwords
associated with them since they were initially created through a third-party
social network like Facebook or Twitter.  If you're using an account like that,
you can do one of two things: (1) Add an e-mail / password through the
"Manage Accounts" link on Turntable or (2) configure the user's id / auth token
manually.

If you do not wish to set up an e-mail / password, you can generate your user id
and auth token via the directions @ http://alaingilbert.github.com/Turntable-API/bookmarklet.html.
Once generated, you can use them in turntabler like so:

```ruby
TT.run do
  client = TT::Client.new(EMAIL, PASSWORD, :user_id => USER_ID, :auth => AUTH)
  # ...
end
```

In this example, `EMAIL` and `PASSWORD` can be any value since they won't need
to be used to generate the user id / auth token.

### Shortcuts

`Turntabler` is a long name and sometimes it's easier to just have a more brief
name available in the same way that `EventMachine` can also be referenced as `EM`.
To help you type a little bit fast, `Turntabler` is also aliased as `TT`.  As a
result, you can interact with the API like so:

```ruby
TT.run do
  client = TT::Client.new(EMAIL, PASSWORD, :room => ROOM)
  client.room.become_dj
  # ...
end
```

Note, however, that if `TT` is already defined when this library is loaded, then
the constant won't get redefined.

### Interactive Console

Typically it's difficult to debug or run simple tests within IRB when using
[EventMachine](http://rubyeventmachine.com/).  However, turntabler provides a
few simple ways to do this so that you can play around with the API interactively.

For example:

```ruby
1.9.3-p286 :001 > require 'turntabler'
=> true
1.9.3-p286 :002 > Turntabler.interactive
=> true
1.9.3-p286 :003 > client = nil
=> nil
1.9.3-p286 :004 > Turntabler.run do
1.9.3-p286 :005 >   client = Turntabler::Client.new(EMAIL, PASSWORD)
1.9.3-p286 :006 > end
=> nil
D, [2012-11-20T08:36:08.025015 #21419] DEBUG -- : Socket opened
D, [2012-11-20T08:36:08.045872 #21419] DEBUG -- : Message received: {"command"=>"no_session"}
D, [2012-11-20T08:36:08.046437 #21419] DEBUG -- : Message sent: {:api=>"user.authenticate", ...}
D, [2012-11-20T08:36:08.119629 #21419] DEBUG -- : Message received: {"msgid"=>1, "success"=>true, ...}
D, [2012-11-20T08:36:08.120213 #21419] DEBUG -- : Message sent: {:api=>"user.get_fan_of", ...}
D, [2012-11-20T08:36:08.188266 #21419] DEBUG -- : Message received: {"msgid"=>2, "success"=>true, ...}
D, [2012-11-20T08:36:08.189158 #21419] DEBUG -- : Message sent: {:api=>"presence.update", ...}
D, [2012-11-20T08:36:08.266749 #21419] DEBUG -- : Message received: {"msgid"=>3, "success"=>true, ...}

# later on...
1.9.3-p286 :008 > Turntabler.run { puts client.user.fan_of.inspect }
=> nil
D, [2012-11-20T08:39:41.084693 #21419] DEBUG -- : Message sent: {:api=>"user.get_fan_of", ...}
D, [2012-11-20T08:39:41.159466 #21419] DEBUG -- : Message received: {"msgid"=>25, "success"=>true, ...}
[#<Turntabler::User:0xa0c7da8 @id="...">, #<Turntabler::User:0xa0c7bf0 @id="...">]
```

In this example, an instance of `Turntabler::Client` is created and tracked in
the console.  Later on, we can then run a command on that client by evaluating
it within a `Turntabler.run` block.  Note that additional debugging output is
displayed -- this is for demonstration purposes only and can be turned off
simply by changing the logging level of `Turntabler.logger`.

### DSL syntax

turntabler has basic support for a DSL language in order to simplify some of the
scripts you may be writing.  The DSL is essentially made available by executing
blocks within the context of a `Turntabler::Client` instance.

There are two ways to do this:

```ruby
# Using the Turntabler.run shortcut:

Turntabler.run(EMAIL, PASSWORD, :room => ROOM) do
  room.dj
  on :user_entered do
    # ...
  end
end

# Passing a block into Turntabler::Client:

Turntabler.run do
  Turntabler::Client.new(EMAIL, PASSWORD, :room => ROOM) do
    room.dj
    on :user_entered do
      # ...
    end
  end
end
```

*Note* that you will likely not want to use the first example (using the
`Turntabler.run` shortcut) when running in the context of a web request in a
web server, simply because it will start a new Fiber.

The equivalent, non-DSL example looks like so:

```ruby
Turntabler.run do
  client = Turntabler::Client.new(EMAIL, PASSWORD, :room => ROOM)
  client.room.dj
  client.on :user_entered do
    # ...
  end
end
```

Notice that in this example the syntax is essentially the same except that we're
one level out and need to interact directly with the `Turntabler::Client`
instance itself.

## Deployment

### Web Server Usage

You'll notice that in many places in the documentation, `Turntabler.run` or `TT.run`
is used to start running a block of code for interacting with the API.  This is
done in order to ensure that the block of code is being run with a running
EventMachine and within a non-root Fiber.

When turntabler is being used as part of a web server or anything else that's
already running EventMachine and already executing code within a non-root Fiber
(such as the rainbows web server) you *should not* using the `run` API.  Instead
you can just run your block like normal:

```ruby
client = Turntabler::Client.new(EMAIL, PASSWORD, :room => ROOM)
songs = client.user.playlist.songs
# ...
```

### Bot Usage

If you're using turntabler in order to build a bot, the primary thing to keep
in mind is how to handle connection loss.  This can occur as a result of a lost
internet connection or even just Turntable forcefully closing a socket for unknown
reasons.  To protect against this, you can configure turntabler to automatically
keep attempting to re-open a connection when it's been closed.

For example:

```ruby
Turntabler.run(EMAIL, PASSWORD, :room => ROOM, :reconnect => true, :reconnect_wait => 60) do
  # ...
end
```

In this example, turntabler will automatically attempt to reconnect if the socket
is ever closed by reasons other than you closing it yourself.  However, rather
than constantly trying to hit Turntable's servers you can configure a reconnect
wait timeout that will cause turntabler to wait a certain number of seconds before
attempting to open a connection.  This will continue to happen until the connection
is successful.  If you were previously in a room, this will also automatically
enter you into the room.  However, it will *not* put you back into the DJ spot.

## Testing

To run the core test suite:

```bash
bundle install
bundle exec rspec
```

## Caveats

The following caveats should be noted when using turntabler:

* Since this library uses EventMachine / Fibers it will only be compatible with
  web servers that support those technologies.  Examples of such web servers include:
  * [Thin](http://code.macournoyer.com/thin/)
  * [Rainbows](http://rainbows.rubyforge.org/)
  * [Goliath](http://postrank-labs.github.com/goliath/)
* This is *not* an official library and so Turntable may make changes to its API
  that causes this to break.  Hopefully we can build a community that can quickly
  react and provide fixes to those changes.

## Things to do

* Add test coverage

## Contributions

The largest contribution for this library is the reference material provided by
Alain Gilbert's [Turntable-API](https://github.com/alaingilbert/Turntable-API)
library.  He provided much of the legwork to get understand how Turntable.FM's
API works and made it much easier to bring a Ruby persperctive to it.

## Dependencies

* Ruby 1.9.3 or later
* [faye-websocket-ruby](https://github.com/faye/faye-websocket-ruby)
* [em-http-request](https://github.com/igrigorik/em-http-request)
* [em-synchrony](https://github.com/igrigorik/em-synchrony)
