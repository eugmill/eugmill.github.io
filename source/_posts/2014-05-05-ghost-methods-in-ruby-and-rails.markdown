---
layout: post
title: "Dynamic finders and Ghost methods in Ruby and Rails"
date: 2014-05-05 15:37:29 -0400
comments: true
categories: [ruby,rails]
---
Rails does a lot for you, including dynamically generating methods like `find_by_[attribute]`, for every attribute your model has. I had always assumed that rails generated the methods when you start it up. While it's true that rails generates the methods for you, it's the where and the how that's interesting.

##`method_missing`

When you send an object a method in Ruby, it first looks for that method in the object itself, then it looks up the inheritance chain. What happens if it reaches the `Object` class and still can't find the method? It calls `method_missing`, which is defined on `Object`. 

```ruby
class Animal
  attr_accessor :sound

  EMOTIONS = {confused:'?',
              surprised:'!'}

  def initialize(sound)
    self.sound = sound
  end

  def confused_sound
    sound + EMOTIONS[:confused]
  end

end

whiskers = Animal.new("meow")

puts whiskers.sound #=> meow
puts whiskers.confused_sound #=> meow?
puts whiskers.surprised_sound #=> NoMethodError
```
So how can we take advantage of this? By overriding `method_missing`, we can actually create dynamic methods on the fly. So before, we were adding the `[emotion]_sound` manually, now we can add them dynamically.

```ruby
class Animal
  attr_accessor :sound

  EMOTIONS = {confused:'?',
              surprised:'!'}

  def initialize(sound)
    self.sound = sound
  end

  def method_missing(name, *args)
    if match = EMOTIONS.keys.find{|emotion| name.match(/#{emotion}_sound/)}
      sound + EMOTIONS[match]
    else
      super
    end
  end

end

whiskers = Animal.new("meow")

puts whiskers.sound #=> meow
puts whiskers.confused_sound #=> meow?
puts whiskers.surprised_sound #=> meow!
puts whiskers.angry_sound #=> NoMethodError
```

Now, as long as our Animal class has a particular emotion, we can call `[emotion]_sound` without defining it ourselves. 

## `respond_to?` and `respond_to_missing?`

Now we need to be able to tell that our Animal class has a method called `confused_sound` but not `angry_sound`. The way we do that is by overriding the `respond_to?` or `respond_to_missing?` methods. `respond_to_missing?` has the advantage in that it allows us to use `whiskers.method` , such as `whiskers.method(confused_sound)`.

```ruby
class Animal
  ...
  def respond_to_missing?(name, include_private=false)
    EMOTIONS.keys.any?{|emotion| name.match(/#{emotion}_sound/)} || super
  end
end

whiskers.respond_to?("sound") #=> true
whiskers.respond_to?("confused_sound") #=> true
whiskers.respond_to?("angry_sound") #=> false
```
These are called _ghost methods_ because if we call `whiskers.methods`, `confused_sound` won't show up. 

## Rails and Ghost methods

Coming back to Rails, we can see how `method_missing` can be used to dynamically define finders such as `find_by_name`, `find_by_age`, etc. However, going through `method_missing` every time can be a little slow. 

As a result, Rails gets even more meta and actually defines the finders as methods on the Model the first time they are called. This way, they are essentially cached, and the next call will no longer go through `method_missing`. 

Stay tuned for a  future post on other details of metaprogramming in Rails. 