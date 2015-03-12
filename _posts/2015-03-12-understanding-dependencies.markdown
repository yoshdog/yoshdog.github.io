---
layout: post
title: "Understanding Dependencies"
header-img: "img/packages.jpg"
author: Toan Nguyen
date: 2015-03-12T21:45:47+11:00
---
I have been very fortunate to find a great mentor in [Sebastian Von Conrad](https://twitter.com/vonconrad) and during one of our sessions we talked about dependencies.

When designing and writing object oriented applications, objects will need to be able to talk to one another. When an object wants to collaborate with another object it will send it a message.

{% highlight ruby %}
class Player
  attr_reader :name

  def initialize(name)
    @name = name
  end
end

class Game
  def initialize(player_name)
    @player = Player.new(player_name)
  end

  def player_name
    @player.name
  end
end
{% endhighlight %}

In this example, we have a Game class with a public instance method called player_name. When ever the Game object wants to know the player's name it will send the player object

# Topics to cover

## Absolute Dependencies
* What your App or Object cannot live without

## Direction of Dependencies

## What should we depend on
* Abstractions
* Objects that have the least dependencies
* Objects which are least likely to change
