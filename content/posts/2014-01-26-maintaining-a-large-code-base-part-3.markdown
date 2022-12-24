---
categories: maintainability scala ruby java golang
comments: true
date: "2014-01-26T00:00:00Z"
title: 'Maintaining a Large Code Base, Part 3: Programming Languages'
---

[_Previous Part: Service Oriented Architecture_](/blog/2013/11/09/maintaining-a-large-code-base-part-2/)

If you are lucky enough to be able to choose the programming language for a new project,
this section might provide some insight on how it might impact the future of your code base.

## Choose Your Programming Languages Wisely

This article focuses only on the technical details of programming languages and their
effects on maintainability. It won't be covering topics such as a language's popularity, tooling,
or library support. While those things are undoubtedly important, those things aren't necessarily intrinsic to a programming language, and they tend to change much more often than the topics I'll
be discussing here.

When I first started writing this section, it was titled "Use a statically typed language."
I then thought of Java with all of its verbosity, realized it's not quite that simple, and it would
make more sense to just outline how certain language features impact code maintainability,
and let you conclude yourself on what kind of language to pursue. 

Having said that, personally I prefer languages with static type systems over dynamic ones from a pure maintainability point of view, except perhaps in the case of languages like Java.

Before I get started, let's define a few things.

> Compiled language - purely statically typed, type annotations are required, 
> or must be able to be inferred at compile time 
> (Scala, Java, Haskell, C++)
>
> Dynamic language - interpreted, no type annotations or they are optional 
> (Clojure, Typed Clojure, Ruby, Groovy, Groovy 2.0, Python, JavaScript)
>
> Statically typed language - same as compiled language

Typed Clojure and Groovy 2.0 both have optional type annotations for partial compile time
type checking and performance improvements, but I am grouping them in the dynamic
language group because they are optional, and they do not provide the same static
analysis guarantees as pure statically typed languages, and hence are not as helpful in
terms of maintainability (more on static analysis later). They are certainly easier
to pick up than a full blown compiled languages with generics, though (more on this in the language complexity section).

### Conciseness
By conciseness I mean this: How much code is necessary to express a particular
piece of business logic? How much of that code is intrinsic to
describing that logic, and how much of it is necessary because of limitations
in the language? Conciseness is important for maintainability because it means there's less
code to read, and less code to refactor when it's necessary to do so.

Here's an example comparing a basic class in Java with the same functionality (both in behavior and runtime performance) implemented in Scala that demonstrates what I mean:

```java Java class
public class Coffee {
    private final boolean caffeinated;
    private final int sugarCubes;

    public Coffee(boolean caffeinated, int sugarCubes) {
        this.caffeinated = caffeinated;
        this.sugarCubes = sugarCubes;
    }
    public boolean getCaffeinated() {
        return caffeinated;
    }
    public int getSugarCubes() {
        return sugarCubes;
    }
}
```

```scala Scala class
class Coffee(val caffeinated: Boolean, val sugarCubes: Int)
```

For comparison's sake, here's a Ruby example of something similar to the two examples
above:

```ruby Ruby
class Coffee
  attr_reader :caffeinated, :sugar_cubes
  
  def initialize(caffeinated, sugar_cubes)
    @caffeinated = caffeinated
    @sugar_cubes = sugar_cubes
  end
end
```

Dynamic languages tend to win in terms of conciseness, since there's no extra
code necessary for specifying type information, although this isn't always the case.

### Language Complexity
It doesn't really matter if code takes fewer lines of code if it's impossible to figure out what it does. Unlike conciseness, language complexity isn't quite as clear cut to define. So
instead of trying to define it in an abstract sense, I'll just give you some examples.

Scala is quite conciseness, but the type system is anything but simple. 
The type system is so involved that it is Turing Complete -- it is so complex that
it is possible to write algorithms such as [infinite loops](http://apocalisp.wordpress.com/2010/06/08/type-level-programming-in-scala/) that only utilize the type system features; no for loops, while loops,
or function recursion. The type system is so complex to the point that a simpler programming
language to possibly become the next Scala is [in the works](http://www.infoq.com/presentations/data-types-issues) by Scala's original designer.

Here's one of the crazier function signatures in Scala, from the Scalaz library:

```scala
implicit def CokleisliMAB[M, A, B](k: Cokleisli[M[_][_], A, B]): MAB[Cokleisli[M[_][_], A, B][A, B], A, B]
``` 

It probably doesn't seem fair to show that signature for those unfamiliar with Scala, but I assure
you, that kind of function signature is still daunting for seasoned Scala developers. 

Although without those complicated type features, code like this might not be possible in Scala:

```scala
// Just Ints
List(1, 2, 3, 4, 5) map { x => x * x } sum 
// => Int = 55

// Ints and Doubles
Array(1, 2 ,3.0, 4.5, 5.55) map { x => x * x } sum
// => Double = 65.0525 

// Just Strings
Vector("a", "b", "c", "d", "e") map { x => x * 2 }
// =>  Vector[String] = Vector(aa, bb, cc, dd, ee)
```

The implementation of the `map` function in Scala is far from simple, though. Writing
this sort of code is not something the average Scala developer can accomplish. In fact,
I'm not sure there's more than a handful of people who could write a collections library
as powerful as the one in Scala -- it took the creator of the language and a few helpers an entire year to get it to the point it is currently at. 

Ruby, on the other hand, has an incredibly simple implementation of the `map` method.
It's just a few lines of code, within the [Enumerable](http://ruby-doc.org/core-2.1.0/Enumerable.html) module. To use it, all that is necessary is an implementation of the `each` function on the inheriting member, and that's it. Because of duck typing, there isn't much else that's needed.

In contrast to both Scala and Ruby, [Golang](http://golang.org/) is far from being concise, but it is well reported for being relatively quick to pick up and start coding, likely in large part due to its incredibly minimal type system. It has no form of generics, meaning it is impossible to write type safe collection methods that work with all types using just the language. Golang is packaged with it's own
basic set of generic data structures (slices, maps, channels), but if you want to write
your own priority queue that works with all types, you are out of luck. This means
the language is easier to learn than a language like Scala, but also results in more code and code duplication.

### Syntactic Flexibility

There's a reason why [Google](https://code.google.com/p/google-styleguide/ "Google's style guides for various languages"),
[Twitter](http://twitter.github.io/effectivescala/ "Twitter's style guide for Scala"),
and [Github](https://github.com/styleguide/ruby "Github's style guide for Ruby") have coding style guidelines.

Can you imagine if each letter of the English alphabet was in a different Unicode character? 
Here's some English that has had each letter of the alphabet mapped to a different character
in Unicode:

    Original text:
    I am Heavy Weapons Guy. And this... [grips Sasha] is my weapon. She weighs 150
    kilograms and fires $200 custom-tooled cartridges at 10,000 rounds per minute. 
    [leans in] It cost $400,000 to fire this weapon...for 12 seconds.
    
    Altered text:
    "ᚋ ᣀᣌ ᚊᣄᣀᨇᨊ ᚙᣄᣀᣏᣎᣍᣒ ᚉᨐᨊ. ᚃᣍᣃ ᣓᣇᣈᣒ... [ᣆᣑᣈᣏᣒ ᚕᣀᣒᣇᣀ] ᣈᣒ ᣌᨊ ᨈᣄᣀᣏᣎᣍ. ᚕᣇᣄ ᨈᣄᣈᣆᣇᣒ 150
    ᣊᣈᣋᣎᣆᣑᣀᣌᣒ ᣀᣍᣃ ᣅᣈᣑᣄᣒ $200 ᣂᨐᣒᣓᣎᣌ-ᣓᣎᣎᣋᣄᣃ ᣂᣀᣑᣓᣑᣈᣃᣆᣄᣒ ᣀᣓ 10,000 ᣑᣎᨐᣍᣃᣒ ᣏᣄᣑ ᣌᣈᣍᨐᣓᣄ. 
    [ᣋᣄᣀᣍᣒ ᣈᣍ] ᚋᣓ ᣂᣎᣒᣓ $400,000 ᣓᣎ ᣅᣈᣑᣄ ᣓᣇᣈᣒ ᨈᣄᣀᣏᣎᣍ...ᣅᣎᣑ 12 ᣒᣄᣂᣎᣍᣃᣒ.

    Code for converting: https://gist.github.com/daniel-trinh/2b6d4b9c38e713148db4

While that is a bit of a contrived example, it's a taste of what programmers have to deal with
in programming languages that have flexible syntax. Out of the times I've talked about the new languages I've been exploring (Clojure, Go, Scala), syntax is almost always the first thing my coworkers notice
and talk about -- this is because understanding syntax is the first step to being able to read the language. If it's different from what they're familiar with, it's just another barrier to learning it.

Languages that are designed for building domain-specific languages tend to have more
syntactic flexibility and lexical complexity. Languages like Ruby and Scala were designed
in mind of supporting DSLs. Unfortunately, flexible syntax makes it harder for users to read code. For every syntax permutation that people use in a language, everyone who reads the language is going
to have to be able to read those permutations. In terms of language design, I don't hear this topic talked about as often as some of the other topics in this article, but nevertheless I still see it as worth discussing. 

Here are some real examples demonstrating the flexibility of Ruby's and Scala's syntax.

```ruby Ruby

# ( ͡° ͜ʖ ͡°) do..end vs curly braces

#  Curly Braces
[1,2,3,4,5].map { |x| x * x }

# do..end
[1,2,3,4,5].map do |x| 
  x * x
end

# ( ͡° ͜ʖ ͡°) Optional periods

# Method calls without periods
[1,2,3,4,5].map { |x| x*x } reduce (:+)

# Method calls with periods
[1,2,3,4,5].map { |x| x*x }.reduce (:+)

# ( ͡° ͜ʖ ͡°) Different ways of defining functions

# Method method
def square(x) 
 x * x
end

# Lambda method
square = lambda { |n| n * n }

# Proc method
square = Proc.new { |x| x * x }

# ( ͡° ͜ʖ ͡°) Flexible method names

# Non Alpha-numeric method names
def +=(new_value)
  @value = @value + new_value
end
```

```scala Scala

/* ( ͡° ͜ʖ ͡°) Curly braces vs parenthesis */

// Curly Braces
List(1,2,3,4,5,6).map { x => x * x }

// Parenthesis
List(1,2,3,4,5,6).map( x => x * x )

// Unnecessary Curly Braces .. and Parenthesis
List(1,2,3,4,5,6).map(
  {
    {
      ( { x => x * x } )
    }
  }
)

/* ( ͡° ͜ʖ ͡°) Optional periods */

// Method calls without periods
List(1,2,3,4,5) map ( x => x * x ) sum

// Method calls with periods
List(1,2,3,4,5).map( x => x * x ) sum

/* ( ͡° ͜ʖ ͡°) Different ways of defining "functions" */

// Method
def square(number: Int): Int = number * number

// Function
val square: Int => Int = { x => x * x }

/* ( ͡° ͜ʖ ͡°) Flexible parameter newline formatting */

// One line
def manyParams(a: Int, b: Int, c: Int): Int

// Several lines
def manyParams(
  a: Int,
  b: Int,
  c: Int
): Int

/* ( ͡° ͜ʖ ͡°) Flexible naming */

// Non Alpha-numeric method names
def +=[T](newValue: T): List[T] = {
  newValue :: this.list
}

// Unicode method names 
def `(╯°□°）╯︵ ┻━┻`: Unit = {
  sys.exit(1)
}
```

And for comparison's sake, here's some Golang:

```go Golang
// ( ͡° ͜ʖ ͡°) Curly braces or parenthesis, no mix and matching
func main() {
  fmt.Println("Hello, 世界")
}

// ( ͡° ͜ʖ ͡°) No optional periods
func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// ( ͡° ͜ʖ ͡°) Unicode naming, but no "+=" stuff
func 世界() {
  fmt.Println("Hello, 世界")
}
```

Those examples are just the tip of the iceberg, but they should give you a good idea
of the kinds of syntax quirks I'm talking about.

Personally, I'm not a fan of the Golang syntax, but there is something to be said about
it's uniformity -- once I've learned how to read one person's Golang code, I can read pretty much all Golang code. This isn't necessarily true of Ruby or Scala.

In some cases, newer languages have relied on the syntax of previously famous languages
to gain popularity -- Java's syntax similarity to C++ was no accident, it was an intentional
design decision by James Gosling to lure programmers away from their familiar C++ homes. JavaScript 
[wasn't always named JavaScript](http://en.wikipedia.org/wiki/ECMAScript#History), but Brendan Eich
decided it would help with gaining popularity. The syntax is also reminiscent of Java -- it uses
curly braces for scopes as well was semicolons for terminating sequences.

The bottom line is that programmers don't want to learn a million different ways of reading
the same code. If you're using a language with an auto formatter such as [Scala](https://www.github.com/mdr/scalariform) or [Golang](http://golang.org/cmd/gofmt/), you're probably in the best boat -- these formatters will format your code for you, enforcing a consistent style without having to spend time and energy trying to manually modify your code to be more readable (no more time wasted on syntax during code reviews). Ruby doesn't have a full fledged auto formatter, but it does have [Rubocop](https://github.com/bbatsov/rubocop) for telling you when your code is breaking style conventions.

### Static Analysis
If there's anything that kills the maintainability of dynamic languages, it's the
lack of type safety. I've used Ruby to death, and while I love using it for small applications
or building prototypes, it's not something I would choose if I had to work on a project
with more than a few engineers or one that was more than a few thousand lines of code (or one that demanded performance, but that's another story).

While I was working at RightScale, we had a massive 900,000 SLOC repository that contained way more
business logic than it should have. The code base was a nightmare to maintain, and it had been that way for years. This was partially from the lack of time given to fix the problem, partially from having fifty engineers modifying the same code base, but also partly due to the nature of Ruby itself.

At one point, we really needed to start deprecating old code to get a sense of what was still
in use and what wasn't. In order to do this, one of our software architects proposed this solution:
add a snippet (that I'll reference as) `dead_code` to any file that was thought to
no longer be in use. The method `dead_code` was a monkey patch on the `Object` namespace that
would log / email / sound the alarms whenever the code was utilized at runtime (since everything
is an object in Ruby due to its Smalltalk influences, this works -- don't try this in other languages).
The idea was that if we ever got production error logs from the `dead_code` snippet, it meant that
the piece of code we thought we could remove was in fact not removable.

In another case, our infrastructure team was prototyping an idea of a "code fence", which
would log / email / sound the alarms whenever a set of files tagged with a certain method was called at runtime (in production). I'll reference this method as `code_fence(some_fence_group)`. We needed this in order to extract business logics into separate services for SOA services

And here's what drives me crazy about the two solutions I outlined above -- neither of those slow iterating approaches would have been necessary in a strongly typed static programming language. While they were clever and much easier than adding unit tests to every piece of code we could find after years of neglecting writing unit tests, it wouldn't have been such a problem if we had used a compiled language (we may have never even had enough of a product to get to this point if we used a compiled language, but let's leave that discussion for another time).

Here is how the `dead_code` situation would have been solved in a strongly typed static programming language:

    1. Remove the file or code you want to check from the repo, and compile.
    2. If there are any compile errors, it's being used. Stop using it. If it isn't, you're good.

    1a. Alternatively, use your IDE to tell you if its used anywhere.
    2a. If it is, stop using it. If it isn't, remove it.

That's all there is to it... unless your code to remove is API code that is called from a separate service, such as a Rails controller for a RESTful HTTP API. Then some logging is required in the API code layer, but at least in this case it's only the API code that needs logging, and not every possible file in your entire code base that you want to get rid of.

For the `code_fence(some_fence_group)` situation, solving this is even easier than with the `dead_code`
case:

    1. Remove the set of code you want refactored into a separate service or library repository.
    2. Compile your newly divided two sets of code. 
    3. If it compiles, you're good. If it doesn't, fix the interfaces and GOTO #2.

You might be thinking at this point, "dude.. unit tests? WTF!?", but I offer you this counter point: static analysis in a compiled language is tantamount to a proof of correctness in terms of interfaces. Unit tests are not proofs, they do not guarantee your code will work for every possible permutation. You literally get interface checking for free by just using a compiled language -- in a language such as Ruby, to merely get a poor mans version of the same test coverage, you'd have to write a unit test for every single new method written to get the level of fine grained error reporting that a compiler would give you.

To drive my point home, that 900,000 SLOC Ruby application I mentioned was a Rails 2.3 application, running on a version of Ruby 1.8.7. Rails 2.3 was released in _2009_. It's still running on Rails 2.3 and Ruby 1.8.7, and it's _2014_. Ruby is up to version 2.1, and Rails is up to 4.0.2 as of me writing this. To be fair, some of the reasons for this is not purely related to the language, but it certainly would have helped with upgrading libraries and Ruby versions if the language was a compiled one.

I should note that there are a few exceptions to this rule in the land of static typing, most notably
pointers in C / C++ and other low level languages, and in Scala there is the [dynamic](http://www.scala-lang.org/api/current/index.html#scala.Dynamic) feature. The important thing here is that these are exceptions, and typical code in these languages will find errors more often than not.


### Compile Times and Unit Test Iteration Times
Slow compile times and slow unit tests slow down code iteration. Dynamic languages
don't have a compile step, but they are not immune to this issue as they do not
have a compile time static type checker, lexer, or parser to find these bugs. Unit tests are vital to fill in this gap... and they can be slow.

That 900,000 line Ruby application I mentioned earlier took _five_ minutes to run a single unit test, mostly due to having to load and initialize way too many gems and libraries. Needless to say, it was impossible to iterate quickly. Small bugs such as interface errors between [strings and symbols](http://www.ruby-doc.org/core-2.1.0/Symbol.html) became more troublesome to debug than they should have been -- for every mismatched `def .. end`, typo, type mismatch, or invalid argument bug I had in my code, it added five minutes to the development time of what I was working on. Unlike with static typing, dynamic languages will typically only find one of these bugs at a time due to their interpreted "run one line at a time" nature. If it was up to me, fixing the time to run unit tests would have been more important than anything else (except for production bugs that needed fixing).

Conversely, on the other side of the language typing fence, I've heard of 45 minute C++ build times -- I can't possibly imagine trying to modify a code base that takes that long to compile. Luckily, I don't think it gets worse than C++, and most statically typed languages have much better compilation times than C++.

Languages like Java, Scala or Golang don't quite have the compile time problem to the extent C++ does.
They all have incremental compilers, which will only recompile code that has been changed
(and any code that was using the changed code). Golang's compiler is likely one of the
fastest for a statically typed language, and Scala is bordering on being as slow as C++
without incremental compilation. One caveat with the JVM -- it takes time to "warm up";
anything that requires restarting the JVM is going to add several seconds to the build process.

If you haven't seen it, [Bret Victor's talk](http://vimeo.com/36579366) makes the case for immediate feedback in much greater detail than I can in this article. Let's just say if it took a musician 5 minutes to hear a note once it's been played, there would be much fewer musicians in the world.

### Conclusion and Next Up
It used to be the case that it was common knowledge that statically typed languages were much more verbose than dynamically typed languages were. This hasn't always been true -- concise compiled languages such as Haskell and ML have been around for a while now -- but concise, strongly and statically typed languages such as Scala are only now starting to gain traction in the industry. 

So now our current options now seem to be these: 

> Simple to learn, concise languages, but terrible for long term large scale 
>    application maintainability due to the lack of compile time static analysis 
>    (Ruby, Python, JavaScript) 
> 
> Simple to learn, verbose languages with okay long term maintainability, but might
> be a pain to refactor because of code duplication
>    (Go, Java 7 and lower)
> 
> Difficult to learn, but concise languages with good long term maintainability...
> as long as you can figure out what your code is doing.
>    (Scala, Haskell)

It remains to be seen if it's possible for a language to have all three qualities -- simple to learn, concise, and good for long term maintainability. 

The next article in this series is still related to programming languages, but it's important enough
to warrant it's own article.

_Coming Next: Limiting Shared Mutable State, Or Why You Should Learn Functional Programming_