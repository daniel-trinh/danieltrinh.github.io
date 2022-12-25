---
categories: maintainability
comments: true
date: '2013-11-05T00:00:00Z'
title: 'Maintaining a Large Code Base, Part 1: Backstory and the Basics'
---

What's one of the most important quality of good software?

_It's maintainable._

It doesn't matter how well optimized the code is if the code base isn't maintainable.
If the code can't be refactored and improved, the software project is stuck in time --
new features are difficult to add, performance can't be improved, and bugs will be harder to pinpoint.
Upfront planning and design can greatly reduce the amount of code rewriting that is necessary,
but its near impossible to get everything right the first time code is written in a significantly
sized code base. I've yet to work on a non trivial software project that was finished and
perfect on the first iteration.

Back when I was in college, the typical coding assignment involved writing a few hundred
to a few thousand lines of code to solve some mind bending arbitrary assignment that
the professor thought was more important than the other three programming assignments
I had from my three other professors, but wouldn't explain what was so important about it.

Anyway, the assignments were mostly automatically graded, and it was quite rare
for the teaching assistant or professor to take a look at students' code to
provide coding style feedback. After the solution to an assignment was submitted,
the code for it would pretty much never be touched again.

Since the assignments were written to facilitate the learning of specific computer science concepts, the source
code for the assignment solutions were very rarely useful outside of the context of the class
that it was presented in. There was pretty much always a better implementation for whatever data structure
or algorithm that were being implemented in the assignments. The only reason to keep the
source code was to be able to look back at it in several years and could go, "Yep.. I wrote that wonderful piece of turd."

I ended up getting better and better at solving these types of self contained
assignments, but I never really learned how to write maintainable code from completing
those assignments.

Fast forward a few years, the largest code base I've worked on has gone from
being somewhere around 10,000 source lines of code (SLOC) in Java, a naturally verbose language,
to the largest code base being a 900,000 SLOC service written in _Ruby_,
a [duck-type-able](http://stackoverflow.com/a/4205396/1093160) dynamic language that's known for its conciseness.
That line count doesn't include comments or blank spaces, as you might expect from "source lines,
and that code base also only refers to one code base -- it was one of many.
It wasn't a code base that I could submit somewhere, forget about and never see again either. This was
the real deal -- industry programming, where things get reused.

This code base was something that I had to stare at on a daily basis. Needless to say,
I've learned many lessons in having to work with a code base of that size, and I'm writing
this to help software engineers, including myself, to think about code maintainability to the same degree they might
think about code correctness or performance, if not higher.

#### The Basics

Let's get the obvious software practices that happen to help code maintainability
out of the way first. These should be familiar with anyone who's been coding for a while.

##### Don't Repeat Yourself

What do you do when you've got a function duplicated 100 times over 100 different
files, and you want to modify the behavior of that common code? Refactor the common code into one function somewhere,
and have the code in those 100 files use that common code.

This is important for maintenance so that if the behavior
of the common code needs to be changed (and the interface is the same),
it can be done in one single location, instead of once per location it is duplicated.

This might seem too obvious to mention, but the convenience of copy paste seems to
win quite often, since it's easier do than to refactor code, which might bring bugs.
Copying and pasting code guarantees your own code won't affect anyone elses, but
it causes maintenance headaches later on.

Sometimes this isn't always a good thing, especially if the code refactoring involves
metaprogramming or other language features that tend to make code less readable.

##### Eat Your Own Dog Food (Dogfooding)

If nobody has ever used your application before shipping it to customers, how can
you be sure it's any good?

Dogfooding is about using your own product whenever possible. If it's bad,
hopefully the pain from using the product will be motivating enough to improve it.

The term supposedly [originated in Microsoft](http://en.wikipedia.org/wiki/Eating_your_own_dog_food#Origin), and apparently they were trying to get the term changed to
[icecreaming](http://www.bizjournals.com/seattle/blog/techflash/2009/11/turning_dog_food_into_ice_cream_and_other_tidbits_from_microsofts_cio.html).

I'll be referring to it as dogfooding though, because that's what most people know it by,
and it doesn't really make sense to dogfood icecream, because icecream is already delicious
and is not dog food quality food, so it doesn't need to be dogfooded to be improved. Or in English,
products that are already good (icecream) don't need to be improved.

Dogfooding doesn't have to be limited to using something at the scale of products, though.
Testing could be considered a form of dogfooding...

##### Write Unit Tests

How can you refactor and change code reliably if there are no checks in place to
make sure the application is working as expected after the change?

These are necessary to quickly catch errors in the business logic of your application.
With dynamic languages, unit tests are also necessary to catch typos, missing method declarations,
and type errors that would be caught in statically typed languages. By having a
framework for quickly checking correctness, it'll be that much easier to reorganize your code, add
features, fix bugs, and possibly splitting it into reusable libraries or services.

Unit tests are really important in dynamic language code bases. Without
a compiler to perform semantic analysis, the next safety net after unit tests are
integration tests, and then it's your manual testers, then customers. Each step
along the way is typically slower than the previous, with
[having customers do your testing](http://blogs.msdn.com/cfs-filesystemfile.ashx/__key/communityserver-blogs-components-weblogfiles/00-00-01-32-02-metablogapi/7317.image_5F00_0F65063B.png just ship it) generally much slower than running a unit test.

##### Document Your Code

Sometimes it's a lot easier to explain in words what your code does than
it is to try to read the code itself, especially if your programming language is not
particularly concise (more on this in part 3). A few comments here and there
can greatly increase the understandability of your code.

##### Do Code Reviews and Design Reviews

So how do you go about figuring out what code needs to be better documented?

Ever write a piece of software, come back to it later some time later,
and have no idea how it works until you sit down and stare at it until your
eyes bleed and you want to rewrite your entire code base from scratch? Me neither.
But for those that this does happen to, it is probably time to dogfood your designs or your code to others.

The writing courses I took in college made a point of having peer reviews for essays.
Peer reviews exist because sometimes things that sound smart in your head do not
read smart when written out on paper. Your peers can offer their own views
on what you are writing that can strengthen your existing ideas. And sometimes,
peer reviews are useful simply because you were too lazy to proof-read your essay
on comparing book X about a topic only the professor cares about, to book Y about who knows what,
mainly because you were busy trying to complete the three programming assignments from your three other professors.

Code reviews were almost never part of the curriculum for any of my computer science
classes, neither by peers nor the teaching staff. Code reviews by peers weren't allowed because they
couldn't trust us not to cheat, and they couldn't trust us not to save solutions for future students during the next iteration of the course. Code reviews by teaching assistants or professors
couldn't be done for everyone because of the numbers problem -- it was too much work
for a couple of people to review the code of thirty students while also coding
automated testing systems for homework solutions, doing research,
and preparing course material, exams, and lectures -- so it was easier to just not
do it for anyone. At least it was fair.

Anyway, have someone else try to read your code. When you are programming, there
is a significant amount of context that is in your short term memory that
might not be obvious or apparent for someone else who didn't write the code.
The process of coding is like storing info about how your code works in RAM,
and for someone else reading it, they don't have the contents of your RAM -- they have
to reconstitute what was in your RAM into their own RAM by [lossily](http://en.wikipedia.org/wiki/Lossy_data_conversion)
interpreting it from your code. Good code results in equivalent RAM in both parties, and bad code is like trying to
interpret a Picasso painting, where one person thinks it's a man's face looking to
the side, and another thinks it's a woman's face looking towards the viewer, when it's really a picture of a plane.

Okay, so it's not quite like that, but what I'm trying to say is to dog-food the readability of your code, otherwise its going to be very hard for you to modify your code if you don't understand it.

[_Next Part: Service Oriented Architecture_](/blog/2013/11/09/maintaining-a-large-code-base-part-2/)
