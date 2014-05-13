---
published: false
title: Destructuring with Ruby
layout: post
tags: [ruby]
categories: [programming]
---
The Basics
So what is destructuring? The most concise definition I found is from Common Lisp the Language. Destructuring allows you to bind a set of variables to a corresponding set of values anywhere that you can normally bind a value to a single variable. It is a powerful feature of Clojure that lets you write some very elegant code. For more information about Clojure's features, I recommend you check out Jay Field's blog post on the subject. While destructuring in Ruby is not quite as powerful as Clojure, you can still do some cool stuff. Let's kick this off with an example.
``ruby
x, y = [1, 2]
x # => 1
``
If you have done Ruby for a while, you have probably seen some code like that before. One place where it can be quite useful is to return multiple values from a function like so: (apologies for the contrived example)

def min_max(array)
  [array.min, array.max]
end
min, max = min_max([3,5,2])
min # => 2
max # => 5
The Splat Operator
Ruby has some more tricks up its sleeve. One of these is the splat (*) operator. Splat will perform two different operations depending on which side of the assignment it is used. The operation you've most likely run into is called slurp or collect. This takes a variable number of arguments and collects it into an array. Its often used to define methods that can take a variable number of arguments. One caveat is that you can only splat the last parameter to a method. Let's refactor our last example to use splat.

def min_max(*values)
  [values.min, values.max]
end
min, max = min_max(3,5,2)
min # => 2
max # => 5
Splat's other mode of operation is called split. This will convert an array into a list of arguments. This is often used when you have an array and you need to call a method with the arguments of the array. Here is an example.

def divide(numerator, denominator)
  numerator / denominator
end
divide(*[4,2]) # => 2
Let's go back to our first example. What happens if our array has more elements than the number of variables we supplied?

x, y = [1, 2, 3]
x # => 1
y # => 2
Oh snap, we lost 3! Not to worry, we can use the splat operator to slurp up the rest of the array elements.

first, *rest = [1, 2, 3]
first # => 1
rest # => [2, 3]
Hey, now we are on to something. If you have looked into Clojure, Scheme, or another Lisp variant that pattern should look familiar. There is a bevy of algorithms that take very complicated problems and break them down into much simpler problems using recursion and a destructuring similar to the one that we have above.

Destructuring Block Arguments
One of my favorite uses of destructuring in Ruby is inside of a block argument. The syntax for this is that anything within parenthesis will be destructured using the rules we've outlined above. Again, here are some examples.

triples = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

triples.each { |(first, second, third)| puts second }
# 2
# 5
# 8

triples.map { |(first, *rest)| rest.join(' ') } # => ["2 3", "5 6", "8 9"]
Destructuring Hashes
Thus far, we have only talked about destructuring arrays. We can also destructure hashes in a very interesting way. A Ruby Hash has a method called values_at that takes a list of keys and returns the values for those keys in th order they were given. Check it out.

hash = {:a => 1, :b => 2, :c => 3}
a, b = hash.values_at(:a, :b)
a # => 1
b # => 2
We can also do block-based destructuring with hashes. One very common use case is using inject. Let's say we wanted to write our own implementation of Ruby's Hash#invert method. We could do that using inject with some destructuring.

hash = {:a => 1, :b => 2}
hash.inject({}) { |new_hash, (key, value)| new_hash.merge(value => key) }
# => {1 => :a, 2 => :b}
In the words of Billy Mays, who may rest in peace, "but wait, there's more". We can destructure our destructuring.

hash = {:a => [1, 2, 3], :b => [4, 5, 6]}
hash.inject({}) { |new_hash, (key, (first, second, third))| new_hash.merge(second => key) }
# => {2 => :a, 5 => :b}
I know, that's nuts. I'm not sure if that would ever be useful, but it's a nice trick to have up your sleeve. Let take it one step further, will splat work in there?

hash = {:a => [1, 2, 3], :b => [4, 5, 6]}
hash.inject({}) { |new_hash, (key, (first, *rest))| new_hash.merge(rest => key) }
=> {[2, 3]=>:a, [5, 6] => :b}
Hells yes it does.


Destructuring Objects
As far as I know, Ruby does not have any useful native facility to perform destructuring on an arbitrary object. One of the beautiful things about the language is we can add features we think are missing. Let's give it a shot. We will use the Hash#values_at method as inspiration.

class Object
  def values_at(*attributes)
    attributes.map { |attribute| send(attribute) }
  end
end

class Dog
  attr_reader :color, :breed, :weight

  def initialize(color, breed, weight)
    @color, @breed, @weight = color, breed, weight
  end
end

dog = Dog.new("black", "lab", "heavy")
color, breed = dog.values_at(:color, :breed)
color # => "black"
breed # => "lab"
Ignore the naivety of the implementation, I know there are better ways to monkey patch. That aside, its a fairly cool concept. This would also allow you to use Objects and Hashes interchangeably when destructuring.