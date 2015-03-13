---
layout: post
title: "Understanding Dependencies"
header-img: "img/packages.jpg"
author: Toan Nguyen
comments: true
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

In this example, when a new Game object is created, it will also create and hold a reference to a Player object. When ever the game wants to know the name of the player we just write: 
{% highlight ruby %}
@player.name 
{% endhighlight %}
Here the sender (game) sends a message (name), to the receiver (player).

# Recognising Dependencies

In Sandi Metz's book (POODR) she explains that an object has a dependency when it knows:

* The name of another class.
* The name of a message that it intends to send to someone else.

Looking at our previous example, we see that game knows the name of the Player class as well name of the message it intends to send to it. Therefore game is dependent on player. 

When an object knows both the name of the class and the type of message it needs to send, it has a **Concrete Dependency**. Game is now highly coupled to the Player class and any changes made to its behaviour will have an affect on the Game.

A better way to handle dependencies is to use **Dependency Injection**.

{% highlight ruby %}
class Game
  def initialize(player)
    @player = player
  end

  def player_name
    @player.name
  end
end
{% endhighlight %}



# Direction of Dependencies

# TL;DR

## Direction of Dependencies
### Absolute Dependencies
* What your App or Object cannot live without


### What should we depend on
* Abstractions
* Objects that have the least dependencies
* Objects which are least likely to change
