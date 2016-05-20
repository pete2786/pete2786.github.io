---
layout: post
title:  Ruby Method Class, Get Addicted - Ruby Junior Senior
tags: [professional rails ruby]
---

As you begin to dive in to the more dynamic nature of Ruby development, it becomes pertinent to start examining precisely what your are working with before calling methods. Ruby methods tend to be composed using three common patterns.

* Required arguments (req): `def asdf(a, s, d, f); end`. This method has four arguments and all four are required. It is called like `asdf(1, 2, 3, 4)`
* Splatted arguments (rest) `def asdf(*a); end`. This method can take any number of arguments and will treat them as an array of length n. It is called like `asdf(1, 2, 3, 4), asdf(1, 2), ... etc`.
* Block arguments: `def asdf(&a); end`. This method takes a block of code. It is called like `asdf{ }`

These can be mixed an matched to some extent, however a splat and a block can only be used once in a method signature. For example: `asdf(a, s, *d, &f)` requires variables `a` and `s`, will jam any remaining variables into the array `d` and will take a block `f`.

Another neat trick `*`ing an Array allows you to call a method with signature `asdf(a,s,d,f)` cleverly. With an array of length four `asdf([1,2,3,4])` will cause an `ArgumentError` while `asdf(*[1,2,3,4])` will succeed.

## Method class

The Ruby `Method` class (http://ruby-doc.org/core-2.2.0/Method.html) provides a few useful tools to help you determine exactly what type of signature you are working with. This can help avoid the dreaded `ArgumentError` when mismatching method signatures.

## Parameters
Any Ruby object will respond to the `method` method, which returns a Method object. For example `method(:to_s)`. It is useful to check `respond_to?(:method_name)` before calling `method`, otherwise `method` will raise a `NameError`. `method(:method_name).parameters` provides us an array a few useful things: The name of arguments, the type (required (:req), splat (:rest)) and the number of arguments.

For example: `def a(a,b); end` has two required parameters. `method(:a).parameters` returns `[[:req, :a], [:req, :b]]`. For a splatted signature `def a(a,*b); end` `method(:a).parameters` returns `[[:req, :a], [:rest, :b]]`.

## Arity
The arity of a method is the number of arguments in it's signature. With Ruby, you get a number back, which tells the number of required arguments and the case tells you if the last argument is a splat.

For example: `def a(a,b); end` has two required parameters. `method(:a).arity` returns `2`. For a splatted signature `def a(a,*b); end` `method(:a).arity` returns `-2`.
##

## Real world example
Much like Calculus, these are neat tricks but how can they apply in the real world? A use case I came across recently was class which was simply provided a Rails Active Record scope name and an array of parameters to pass. I was able to leverage the `arity` method to make my class more flexible. Here is a watered down example of my solution:

```
class ScopeCaller
  attr_accessor :resource, :scope_name, :parameters

  initialize(resource, scope_name, parameters)
    self.resource = resource
    self.scope_name = scope_name
    self.parameters = Array(parameters)
  end

  def call
    if resource.method(scope_name).arity == 1
      resource.send(scope_name, parameters)
    elsif resource.method(scope_name).arity == parameters.length || resource.method(scope_name).arity == - 1
      resource.send(scope_name, *parameters)
    end
  end
end

```

It is a naive example, but you can see the usefulness in examining the arity of your method with an unknown set of parameters. Open up `irb` and play around with the `method` method. What arguments does, `method(:puts).parameters` reveal? How about the `arity`?
