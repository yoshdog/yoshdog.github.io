---
layout: post
title: "Understanding Dependencies"
header-img: "img/packages.jpg"
author: Toan Nguyen
comments: true
date: 2015-03-12T21:45:47+11:00
---
I have been very fortunate to find a great mentor in [Sebastian Von Conrad] and during one of our sessions we talked about dependencies.

# Recognising Dependencies

When designing and writing object oriented applications, objects will need to be able to talk to one another. When an object wants to collaborate with another object it will send it a message. This knowledge of one another is the basis of dependencies.

In Sandi Metz's book (POODR) she explains that an object has a dependency when it knows:

* The name of another class.
* The name of a message that it intends to send to someone else.

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
We see that game knows the name of the Player class as well name of the message it intends to send to it. Therefore game is dependent on player, also game is now coupled to player.

## Connascence 

We know that to write Object Oriented programs there needs to be some coupling between objects. We also know that having tightly coupled objects is generally a bad thing as it affects the ability of the application to change. So how do we determine how tightly coupled our design is and how do we go about improving our design?

[Jim Weirich] gave a talk about *The Grand Unified Theory* where he talks about a book called [What Every Programmer Should Know About Object-Oriented Design] by Meilir Page-Jones.
In the book, Meilir Page-Jones introduces a new metric, [Connascence] to measure the complexity of dependencies between two objects. The type of dependencies are categorised by their strength of connascence. The higher the strength of connascence the more difficult and costly it is to change the objects in the relationship.

Below are the types of Connascence ranging from weak (good) to strong (bad):

* Connascence of Name (CoN)
* Connascence of Type (CoT)
* Connascence of Meaning (CoM)
* Connascence of Position (CoP)
* Connascence of Algorithm (CoA)
* Connascence of Execution (CoE)
* Connascence of Timing (CoT)
* Connascence of Values (CoV)
* Connascence of Identity (CoI)

### Refactoring Example

{% highlight ruby %}
class Game
  def initialize(player_name)
    @player = Player.new(player_name)
  end

  def player_name
    @player.name
  end
end
{% endhighlight %}

Looking at our previous example, we see that Game knows the name of the Player class as well as the name of the message it needs to send. This relationship can be classified as a **Connascence of Identity (CoI)** and is the strongest type of dependency.

We can refactor to lessen the strength of the dependency by using **Dependency Injection**.

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

Now game is initialized with a player object and a reference is saved in a @player instance variable. Whenever we want to know the name, we just send a name message to the saved reference of the player object. 

Now the only dependency to player is just the name of the message game needs to send it. This relationship is classified as a **Connascence of Name** and is the weakest type of dependency. As a result, game is now loosley coupled to player. This type of dependency is also called a dependency on **Abstraction**, in that game only cares if the object has a public interface which responds to name. This falls in line with the advice given in Sandi's Book, in that an object should depend on an abstraction rather than something concrete.

# Direction of Dependencies

# TL;DR

## Direction of Dependencies
### Absolute Dependencies
* What your App or Object cannot live without


### What should we depend on
* Abstractions
* Objects that have the least dependencies
* Objects which are least likely to change

[Sebastian Von Conrad]: (https://twitter.com/vonconrad)
[Jim Weirich]: (https://www.youtube.com/watch?v=NLT7Qcn_PmI)
[Connascence]: (http://en.wikipedia.org/wiki/Connascence_(computer_programming))
[What Every Programmer Should Know About Object-Oriented Design]: (http://www.amazon.com/Every-Programmer-Should-Object-Oriented-Design/dp/0932633315/ref=la_B001ITVFZQ_1_3?s=books&ie=UTF8&qid=1426399203&sr=1-3)
