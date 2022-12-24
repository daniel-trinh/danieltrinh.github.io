---
categories: scala
comments: true
date: "2013-11-04T00:00:00Z"
published: false
title: 'One Year of Scala: Impressions From a Rubyist'
---

This is about my experiences of using the [Scala programming language](http://www.scala-lang.org/)
over the past year outside of work, compared with my experiences of using the 
[Ruby programming language](https://www.ruby-lang.org/), which I have been using for over two years at work.

Warning: This article is the story of how I ended up using Scala for a year. It's not intended to
be a direct comparison of the two languages. If you're looking for a more technical comparison and evaluation of the 
two languages, [see these slides](http://www.slideshare.net/El_Picador/scala-vs-ruby#btnNext) that directly compare the two
langauges, or [this video](http://parleys.com/play/51c178ece4b0d38b54f46217/chapter0/about) about a Ruby developer who got really into Scala.

Let's get into a little bit of my own history with programming languages.
##Getting into Ruby

I picked up Ruby towards the end of college, after getting tired of the verbosity of coding in
C, C++, and Java -- the primary programming languages taught at my college.

During my college days, there was a lot of hype around Ruby on Rails, a popular Web MVC framework
that held the promise of allowing developers to crank out CMS style web applications in very little time. 
There were prominent companies using it, some of the more notable ones being Twitter and Linkedin.

I did a bit more research, and noticed it was decreasing in popularity, having 
[peaked in google searches by 2007](http://www.google.com/trends/explore#q=ruby%20on%20rails&cmpt=q), 
but I decided to give it a try anyway. My (over simplified) line of thought more or less went something like this: 
*"If it's good enough for Twitter, it's good enough for me".* My first impressions of Ruby were that it was 

At the time, I was completely engrossed in the language -- there wasn't a single thing anyone could say 
to me to dissuade me from using the language.

Nearing graduation from college, I got an internship that turned into a full time job at [RightScale](http://www.rightscale.com/),
and have been using Ruby there ever since.

## Searching for a new programming language

I kept running into a particular 
[programming language performance benchmark website](http://benchmarksgame.alioth.debian.org/u64q/benchmark.php?test=all&lang=yarv&lang2=scala&data=u64q).
I knew that the benchmarks weren't necessarily reflective of real world applications of programming languages, 
but there I was, sitting in my chair, staring at a chart on the website that said Ruby to be about *fifty times slower than C code.*

So I decided to look for a new programming language to learn, one that might complement my existing Ruby knowledge,
one that could be considered good enough to bet building an entire company's infrastructure on.

### Other Languages

I looked at several languages, the notable ones being *C++, Java, C#, Haskell, Javascript(Node.js), Clojure, and Go.* I was looking for something
open source, that had [first class functions](http://stackoverflow.com/questions/5178068/what-is-a-first-class-citizen-function), was relatively concise, 
ran fast and supported concurrency well, and was quick to develop code with.

I tried *C++* first, despite it being verbose and not having first class functions (unless you include pointers), 
mostly due to it's pedigree as the language that can render video games with amazing visuals at sixty frames per second, 
and can help scale Google-sized data centers better than any other language (when written well). 
I eventually dropped it due to it's verbosity and language complexity 
(I just downloaded a draft of the C++ language specification from 2005, and it was 879 pages long).

*Java* was dismissed because it didn't have first class functions and was verbose, similarly to C++. 
I was pretty spoiled by the conciseness of typical Ruby code.

*C#* was dismissed because it isn't open source.

*Haskell* looked interesting, but it's been around for a couple of decades and hasn't quite gotten as much traction as 
I would hope for in a language I was about to invest a significant amount of time using.

I tried *Clojure* briefly, as it ranked surprisingly high in CPU performance on the language benchmarks website, 
despite it being a dynamic language -- about six times the performance of C code. The syntax was off putting, but
I was really put off when I ran across a forum post from Rich Hickey, the creator of Clojure, more or less pleading
to the community for financial help to continue working on Clojure. Let's get things straight -- I had no problems with the actual gesture of asking for donations. 
What I did have a problem with was the implications of him *needing* money to continue Clojure development. 
If he couldn't afford to work on Clojure, what did that mean for the future of the language?

*Node.js* was interesting for a while. It introduced me to the concept of non-blocking code, and I was already a bit
familiar with Javascript. Javascript is relatively concise, and fast for a dynamic language
(six times slower than C code on the language benchmarking website, similar to Clojure, compared to Ruby's 50x).
Most of the performance benefit comes from it being built on the V8 Javascript engine, the same highly optimized engine that
power's Google Chrome's browser. I decided against pursuing Node.js further because of call back hell, and it having a global interpreter lock.
The same thing that brought most of the performance to the platform was the same thing that was limiting it from being able to utilize
multiple cores -- V8 was designed as a single threaded, sandboxed engine to run one V8 process per browser tab.

These days, callbacks in Javascript can be avoided by using [promises](https://github.com/kriskowal/q), a way
of composing non blocking code written without having to nest functions within functions that is the norm in the callback style of programming.
I haven't gone back to learn it because I've fallen in love with static typing -- and frankly, I think Javascript is an ugly language.

*Go* is the youngest of all of the languages listed here. It's fast, statically typed, somewhat concise due to type inference,
compiles fast, doesn't have a million ways of implementing the same thing like Ruby does, is built for
concurrency from the bottom up, and has a decent amount of tooling for such a new language. The things that held me back from using it
was the lack of [generics](http://stackoverflow.com/questions/31693/what-are-the-differences-between-generics-in-c-sharp-and-java-and-templates-i?rq=1),
and that the ecosystem and community are still quite small. At the time of writing, there's no way of distributing versioned Go code --
it's currently distributed using version control branches, and version control branches are not code snapshots, they are meant
to be continuously updated with new code.

### Discovering Scala
The first time I read about Scala was on a StackOverflow page. There was a mention of it being used at Twitter, 
which piqued my curiosity for a bit. However, soon after that, I came across a comment from a user that said it was mostly 
a complicated research language, and no one really used it. Images of professors sitting in ivory towers came to my mind. 
I didn't look into it much further for some time.

## Ruby Pros
## Ruby Cons

## Scala Pros
## Scala Cons


Frankly, I could 