---
categories: maintainability essay
comments: true
date: "2014-01-26T00:00:00Z"
published: false
title: 'Maintaining a Large Code Base, Part 4: Limiting Shared State'
---


### Namespace pollution / Unscoped monkey patching
If you've used Java or C#, you're probably familiar with `import` statements.
If you've used also used browser JavaScript or Ruby, you're probably aware that these languages
don't have an import system like the previously mentioned languages, and you're probably
already aware with where I'm going with this. 

Here's some code in Ruby that extends the language to


## Avoid modifying shared state
This is something I picked up after learning functional programming. In functional
programming languages, code is generally written *without side effects*. Variables
in functional programming languages can't be modified after being set, and
functions will always return the same value given a specific set of arguments.
For example, basic arithmetic has no side effects -- "1 + 1" will always equal 2.

The following example of variable modification would be not possible in a pure functional programming
language:
```ruby
this_is_a_string = nil             # value is assigned to nil
this_is_a_string = "another value" # not possible, already assigned
``` +

This example of instance variable modification would also not be allowed:
```ruby
class Person
  def initialize()
    @name = "Louis"
  end

  # This method is not allowed -- it alters state that is outside
  # the scope of this method
  def change_name(new_name)
    @name = new_name 
  end
  
end
```

So what's the purpose of this? Seems pretty limiting, right? Let's
look at another example, where mutating variables is commonplace.

```ruby
  
  Person.new
  person.name == "Louis" # true

  # ... another 1000 lines of who knows what. There's only
  # enough time to skim this code because there's five billion files
  # of code you have to search through to find that critical production
  # error, and it's 2 AM in the morning on a Sunday.

  get_name == "Louis" # ???
```

Okay, maybe that example's a bit contrived. Let's look at another example that involves monkey patching. 
For those who aren't familiar with Ruby, there is pretty much nothing that is final or
unmodifiable in Ruby -- you can overwrite any variable or method on any object or class at any time,
which includes built in Ruby library code. 

```
hash_map = Hash.new  # => {}

Hash = nil # this makes it impossible to call Hash.new

Hash.new # => NoMethodError: undefined method `new' for nil:NilClass

```
Now in that example, the state modification is right next to usage, so it's
completely obvious what's happening. Imagine that the first line was stuffed
in some library code somewhere instead. 

All of a sudden it becomes a nightmare to look for the line of code that is 
making it impossible to declare new hashes. It's also not possible to simply
not include / import / require the library either, since Ruby packages aren't
scoped on a per file basis like Java's import package system is.

So those examples were pretty specific to Ruby, but they were used
to illustrate the point of shared state modification leading to maintenance
headaches. 

Sometimes modifying shared state is necessary. It's best to only do it
when you need it, though, and to document the mutable state behavior when it is
done.

## Fix Broken Windows


## TL;DR
* Write tests
* Document your code
* Do design reviews and code reviews
* Split your larger code bases into smaller ones, and don't put too many
  developers on a single team
* Use standard communication protocol across services (HTTP, RPC, etc).
* Use a good programming language and be mindful of your programming
  language's weaknesses
* Don't let your code base deteriorate, fix your broken windows