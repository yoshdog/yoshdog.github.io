---
layout: post
title: "Knowing when to use testing doubles"
modified: 2014-08-16 20:24:31 +0100
tags: [testing, rspec, doubles, tdd, london style, mocks]
image:
  feature: DarthTesting.png
  credit: 
  creditlink: 
comments: true
share: true
---
When I was first learning the London School of Test Driven Development I had no clue what the real purpose of testing doubles and mocks were. All I knew was that if I wanted to write my tests in isolation I had to use a testing double. This incorrect understanding led me to write test where I mocked everything but the method I was testing for. Confused at why all my tests seemed so complicated, I asked my friend look at my tests and offer me some advice. The email reply he sent back was, just watch this video. This was a light bulb moment for me and testing with doubles and mocks started making sense.

[The Magic Tricks of Testing by Sandi Metz](https://www.youtube.com/watch?v=URSWYvyc42M)

In this post I will try to explain what changes I had to make in my understanding of Object Oriented Programming as well as the points made by Sandi Metz. I will describe my thought process on identifying the type of messages Im testing and then type of test I will write.

## Understanding Object Oriented Programming
My original understanding of classes was that it was a blueprint for common methods and variables of a particular object. For example, a car has four wheels and a method to drive foward. Then when wanted to create an actual car object I just had to make an instance of the Car class and happily use the methods and variables it came with.

### Messages

This all changed when I read this gem of a quote:

> Message and method are metaphors that we use somewhat interchangeably, but they are
subtly different. In Object Oriented Programming, objects communicate by sending
messages to one another. When an object receives a message, it invokes a method with the
same name as the message.
[Rspec mocks](https://www.relishapp.com/rspec/rspec-mocks/v/3-0/docs)

This quickly changed my perception on how objects worked. Objects send messages to each other and itself. If an object understands a particular message received it will invoke an action (run the method). If the object does not understand the message received it will then say it has no idea what your talking about (NameError). This messsage passing between objects abstraction is further highlighted when reading over the naming of core ruby and rspec methods. For example: 
{% highlight ruby %}
expect(car).to receive(:drive).and_return("Im moving forward")
{% endhighlight %}

Above is a common rspec test you will see alot. Using this new abstraction, the test is saying: I expect the car object to receive a message called drive and reply back "Im moving forward".

Here are two common ruby methods:
{% highlight ruby %}
car.respond_to? :drive

car.send(:drive)
{% endhighlight %}

The respond_to? method just checks if our car object responds to a message called drive. If the object does, it will reply back with true, if it doesn't it will reply back with false.

The send method is a ruby method for objects to invoke an action. It is basically telling the car object to run the action drive.

### States
So now we know that objects receives, responds to and sends messages to each other but objects also have instance variables.  Instance variables are used to store the current state of the object. For example:

{% highlight ruby %}
class Car
  def initialize(model)
    @moving = false
    @broken = false
    @model = model
  end
end
{% endhighlight %}
Here my car class has three instance variables. These instance variables (moving, broken and model) describe the current state of the car. The outside world can change or ask for the current state of the car object by sending a message to the car. If the car object understands the message it will respond back with the current state asked.

## Types of messages
In Sandi Metz's video, she outlines that there are three types of messages in OOP:

* Query messages
* Command messages
* Messages which does both

The type of tests writtern are different for each type of message, so it is important to identify and understand the purpose of each.

### Query messages
A message is a Query Message if it returns something but changes nothing to the object we sent the message to. For example:

{% highlight ruby %}
class Car
  def initialize
    @broken = false
  end

  def broken?
    @broken
  end
end
{% endhighlight %}

If we want to see if the car is broken, we will send a message called: broken? to the car and it will return a message back telling us if its broken (true) or not broken (false). It however does not change and state of the car object. Another example of a query message is sending a GET request to an API or a Select request to a database for data. 

### Command messages
A message is a Command Message if it returns nothing but changes the receiver object's state. For example:

{% highlight ruby %}
class Car
  def initialize
    @broken = false
  end

  def crash!
    @broken = true
    self
  end
end
{% endhighlight %}

If we send a message called: crash! to the car object. We are commanding the car to change its current state to be broken. We are not expecting any message to be returned so we will return the car object back. Another example of a command message is sending an INSERT, UPDATE request to a database or a PUT request to an API. 

### Messages which does both
Dangerously there are messages which changes the state of an object and returns data. We must be aware of these messages and if possible, refactor it into a seperate query and command message. If not, we need to make special considerations on them and write seperate command and query style tests on them. An example of a message which does both is when we call pop on an array. We are changing the state of the array object (it is now missing the last element) and it is also sending the last element back to us.

## Testing an object in Isolation
So now we know how to identify each type of message, we can start using the London School of TDD to isolate our object we want to test. We are only isolating a single object during our tests, not each method. Just the object, in this case just the Car.

<figure>
  <img src="/images/object_in_isolation.png" alt="">
</figure>

In the above picture, if we put a cone on top of the object we are testing in isolation there will three sources of messages for the object. The object will either receive messages from external sources. It will receive and send messages to itself, or it will send message to external objects. We also know that there are two types of messages (Query and Command) so there are in total six types of messages that need to be accounted for when testing in isolation. 

<figure>
  <img src="/images/what_to_test.png" alt="">
</figure>

The above table outlines what types of messages we need to write tests for. I will expand on these individually.


## London Style TDD
The steps I use to determine on what type of tests to write are:

* Is the object receiving or sending a message?
* Is the destination of the message internal or external?
* Is it a query or command message?

### Internal Query and Command Messages
The great news is that we do not have to test for any messages sent within the object. These are usually private messages used within the object or its sending a message internally as part of a method. For example:

{% highlight ruby %}
class Car
  def initialize
    @broken = false
  end

  def broken?
    @broken
  end

  def drive
   go_forward unless broken?
  end
end
{% endhighlight %}

When writing a test for the drive method, we do not need to expect the car to send or receive a message of broken? or go_forward. This is because we will be testing these methods as incoming messages.

### Incoming Messages
An incoming message is any public message that your object responds to. For Example:

{% highlight ruby %}
class Car
  def parked?
    @parked
  end
end
{% endhighlight %}

When an external object wants to know if the car is parked it will send the message (parked?) to the car object. Incoming messages are the most common message you will need to test for. Rule of thumb, these will be all your public methods in your class.

#### Query Messages
To test for an incoming query message, we will just need to test for the result.

{% highlight ruby %}
describe Car do
  context 'initialised' do
    it 'should not be parked when created' do
      car = Car.new
      expect(car).not_to be_parked
    end
  end
end
{% endhighlight %}
Here I am testing if the car is not parked when its orginally created. This will force me to write the following code:

{% highlight ruby %}
class Car
  def initialize
    @parked = false
  end

  def parked?
    @parked
  end
end
{% endhighlight %}

As such, I have now created a public incoming query message (parked?) which external objects can query.

#### Command Messages
To test for an incoming command message, we will need to test if there has been a changed in an objects state when the method has been invoked.

{% highlight ruby %}
describe Car do
  it 'can park itself' do
    car = Car.new
    car.park!
    expect(car).to be_parked
  end
end
{% endhighlight %}

Here in my test I called my command message (car.park!), and then I need to test for the outcome. In this case I am testing if the car is now parked.

This will lead me to write the following lines of code.

{% highlight ruby %}
class Car
  def initialize
    @parked = false
  end

  def parked?
    @parked
  end

  def park!
    @parked = true
    self
  end
end
{% endhighlight %}

As such I have writtern a command message (park!) which external objects can use to change the state of the car to be parked.

### Outgoing Messages
Outgoing message is one where my object sends a message to an external object. This is where we will start mocking the behavour of the external object. 

#### Query Messages
When testing for an outgoing query message we will need to test it in two parts. 

* First we need to test if the message has been sent. 
* Then we will test what happens when we get a response from the external object.

For example, I can park my car in a parking lot unless it is full. First I need to test if a message is sent to the parking lot.

{% highlight ruby %}
describe Car do
  context 'parking at a parking lot'
    it 'sends a message to parking lot' do
      car = Car.new
      parking_lot = double :parkinglot
      expect(parking_lot).to receive(:full?)
      car.park_at parking_lot
    end
  end
end
{% endhighlight %}

First we will need to create a testing double of the parking lot. Then we will need to set an expection that the parking lot will receive a message full? from the car object. After this, we write the method we want to create which sends this external message to the parking lot.

This will lead me to write the following code to make the test past:
{% highlight ruby %}
class Car
  def park_at parking_lot
    parking_lot.full?
  end
end
{% endhighlight %}

Now that we tested that the message has been sent to the parking_lot double, we will need to mock a response and then test the outcome of our car object. Here I will mock and test the outcome if the parking_lot is not full.

{% highlight ruby %}
describe Car do
  context 'parking at a parking lot'
    it 'is parked if parking lot is not full' do
      car = Car.new
      parking_lot = double :parkinglot
      allow(parking_lot).to receive(:full?).and_return(false)
      car.park_at parking_lot
      expect(car).to be_parked
    end
  end
end
{% endhighlight %}

Here I am mocking that the parking_lot double is sending a response of false when its asked, are you full?
I can save time and write the mocked response when we create the double:

{% highlight ruby %}
describe Car do
  context 'parking at a parking lot'
    it 'is parked if parking lot is not full' do
      car = Car.new
      parking_lot = double :parkinglot, full?: false
      car.park_at parking_lot
      expect(car).to be_parked
    end
  end
end
{% endhighlight %}

Now when I am parking my car using the car.park_at parking_lot method and if the parking lot is not full, my expectation is that the car is now parked.

This will lead me to write the following code to make the test past.

{% highlight ruby %}
class Car
  def initialize
    @parked = false
  end

  def parked?
    @parked
  end

  def park!
    @parked = true
    self
  end

  def park_at parking_lot
    park! unless parking_lot.full?
  end
end
{% endhighlight %}

Now when I park my car at a parking lot it will be parked. Only if the parking lot is not full. 

#### Command Messages
We do not need to test for outgoing command messages to external objects. This is because its already tested for as an internal command message for that external object. For example if we took the last parking space in the parking lot, the parking lot status will change to full. As a car, we do not need to test for this.

## Conclusion
Objects interact with each other by passing messages to each other. Messages can either be Query messages where only data is returned or a Command message where no data is returned but an objects state is changed. By categorising the message we will know if we need to use a testing double or not.

We only need to use a testing double when we are sending an outgoing query message to an external object. We will first need to test if the message has been sent to the external object. Then we will mock the returned value of the query, and test if it changes a state in our object when calling the method we are testing for.
