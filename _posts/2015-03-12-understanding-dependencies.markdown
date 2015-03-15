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
We see that game knows the name of the Player class as well name of the message it intends to send to it. Therefore game is dependent on player.

## Connascence 

We know that to write Object Oriented programs there needs to be some coupling between objects. We also know that having tightly coupled objects is generally a bad thing as it affects the ability of the application to change. So how do we determine how tightly coupled our design is and how do we go about improving it?

[Jim Weirich] gave a talk about *The Grand Unified Theory* where he talks about a book called [What Every Programmer Should Know About Object-Oriented Design] by Meilir Page-Jones.
In the book, Meilir Page-Jones introduces a new metric, [Connascence] to measure the complexity of dependencies between objects. It categorises the type of dependencies and their strength of connascence. The higher the strength of connascence the more difficult and costly it is to make future changes. 

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

We can refactor to a weaker type of connascence by using **Dependency Injection**.

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

The only dependency to player is the knowledge of the name of the message that game needs to send it. This relationship is classified as a **Connascence of Name** and is the weakest type of dependency. As a result, game is now loosely coupled to player. This type of dependency is also called a dependency on **Abstraction**, in that game only cares if an object has a public interface which responds to name. This falls in line with the advice given in Sandi's Book, where she advises that an object should depend on an abstraction rather than something concrete.

Although this is just a single refactoring pattern, there are more pointed out in the talk by [Jim Weirich].

# Direction of Dependencies

When I was designing my [Battleships] game, I went about designing my domain model as follows.

{% highlight ruby %}
class Grid
end

class Player
  attr_reader :grid

  def initialize(grid)
    @grid = grid
  end
end

class Game
  attr_reader :players

  def initialize(players)
    @players = players
  end
end
{% endhighlight %}

When a player is created it will have a grid. Whenever we wanted to access a players grid we just send a grid message on the player. For example: 

{% highlight ruby %}
bob = Player.new(Grid.new)
alice = Player.new(Grid.new)

game = Game.new([bob, alice])

game.players.first.grid
{% endhighlight %}

Sebastian quickly stopped me and asked:

>What are the absolute essential classes we need to start playing? What is the absolute minimum needed?

I said: Game, Grid, Player and Ship as that is what the actual game physically had. A Game has two players and each player has a grid.

After some discussion it became obvious that we could just create our Battleships game with just two Grids and a Game object to hold them in. Now our domain looks like:

{% highlight ruby %}
class Grid
end

class Game
  attr_reader :grids

  def initialize(players)
    @grids = players.each_with_object({}) do |player, memo|
      memo[player] = Grid.new 
    end
  end
end
{% endhighlight %}

In this design we made Grid our absolute dependency of our Game. The game of battleships cannot be played if it didn't have any grids and is a central component. With this design player has become just a detail. It is not a core component of our application and can be any abstract object.

For example we can represent player as a Symbol:

{% highlight ruby %}
game = Game.new([:bob, :alice])

game.grids[:bob]
{% endhighlight %}

Or an actual Player object:

{% highlight ruby %}
class Player
  attr_reader :name
  
  def initialize(name)
    @name = name
  end
end

bob = Player.new('Bob')
alice = Player.new('Alice')

game = Game.new([bob, alice])

game.grids[bob]
{% endhighlight %}

And the application works exactly the same. By reversing our dependencies the application has become much easier to change.

## What to depend on

When I started adding new functionality to my Grid class it became apparent that Grid has now became:

* Unstable. I am constantly adding and removing or renaming my public interfaces.
* Its starting to have more dependencies. 

In POODR, Sandi mentions that when choosing the direction of dependencies we should choose to depend on objects that:

* Have the least dependencies.
* Are least likely to change.
* Depend on abstractions.

When looking at my old design:

{% highlight ruby %}
class Grid
end

class Player
  attr_reader :grid

  def initialize(grid)
    @grid = grid
  end
end

class Game
  attr_reader :players

  def initialize(players)
    @players = players
  end
end
{% endhighlight %}

We see that Player has a dependency to grid. Because grid is unstable and has other dependencies, choosing to depend on it means we will frequently have to check if our player class is affected by the frequent changes to grid.

When we reverse the dependency direction, our Grid class is now depending on an abstraction of a player. The player also is least likely to change and has no dependencies. This design makes our application easier to work with and extend.

It is important to map out and understand the dependencies within our domain model before we start programming. The direction of the dependencies play a big part in how our application responds to change in the future. If the direction is incorrect it comes increasily difficult to make any future changes as any small will affect our entire application. So it pays to spend a little time at the beginning mapping out the dependencies, their complexitities and direction.

# TL;DR

When designing Object Oriented applications which are easy to change and maintain, we should identify dependencies between objects and aim to lower their complexities. [Connascence] is a metric which measures how complex a dependency is. The weaker the type of connascence the more loosely coupled dependency between objects. Injecting dependencies weaken the type connascence between objects. 

We should identify and **only** depend on classes which are core to our domain. All other non-critical details should be abstracted away.

The direction of dependency plays an important part in our design. We should depend on objects which are less likely to change and have few dependencies.

[Sebastian Von Conrad]: https://twitter.com/vonconrad
[Jim Weirich]: https://www.youtube.com/watch?v=NLT7Qcn_PmI
[Connascence]: http://en.wikipedia.org/wiki/Connascence_(computer_programming)
[What Every Programmer Should Know About Object-Oriented Design]: http://www.amazon.com/Every-Programmer-Should-Object-Oriented-Design/dp/0932633315/ref=la_B001ITVFZQ_1_3?s=books&ie=UTF8&qid=1426399203&sr=1-3
[Battleships]: https://github.com/yoshdog/battle-ships
