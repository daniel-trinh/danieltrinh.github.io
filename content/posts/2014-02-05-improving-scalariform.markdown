---
categories: scala project
comments: true
date: "2014-02-05T00:00:00Z"
published: true
title: Improving Scalariform, The Scala Source Formatter
---

Scalariform is a Scala source code formatter, originally written by [Matt Russell](http://github.com/mdr) (big thanks to him for writing it).

It's much easier to show you what this does than it is to try and explain it, so that's what I'll do.

This is some poorly formatted code before running Scalariform:

```scala
class Coffee {
val sugarCubes = 20
val isCaffeinated = false

def energyBoost = {
if (caffeinated) 
100 * sugarCubes
else
0
}
}
```

After running Scalariform:
```scala
class Coffee {
  val sugarCubes = 20
  val isCaffeinated = false

  def energyBoost = {
    if (caffeinated) 
      100 * sugarCubes
    else
      0
  }
}
```

Pretty cool, right? Unfortunately, there haven't been very many updates to the official version of this super awesome project lately,
so I decided to fork the project and start improving upon it myself. Since I use Scala quite often, I've got some pretty strong motivation to work on improving it.

Here's a quick summary of what I've added to Scalariform _so far_:

```scala Previous Scalariform Formatting
def showInput[A](
  parent: Component = null,
  message: Any,
  title: String = uiString("OptionPane.inputDialogTitle"),
  messageType: Message.Value = Message.Question,
  icon: Icon = EmptyIcon,
  entries: Seq[A] = Nil,
  initial: A
): Option[A]

case class Cake(
  icingFlavor: Flavor = Vanilla,
  cakeFlavor: Flavor = Chocolate,
  
  candles: Int = 1,
  layers: Int = 3,
  iceCream: Boolean = False
)

o.manyArguments(abc = 0,
  abcOne = 1,
  abcTwo,
  abcThree = 3,
  abcFour = 4,
  abcFive = 3
)
```

```scala Scalariform Formatting with My Changes

// Parameter names, types, and defaults are aligned into three separate columns
def showInput[A](
  parent:      Component     = null,
  message:     Any,
  title:       String        = uiString("OptionPane.inputDialogTitle"),
  messageType: Message.Value = Message.Question,
  icon:        Icon          = EmptyIcon,
  entries:     Seq[A]        = Nil,
  initial:     A
): Option[A]

// Two newlines will result in separate alignment groups 
case class Cake(
  icingFlavor: Flavor = Vanilla,
  cakeFlavor:  Flavor = Chocolate,
  
  candles:  Int     = 1,
  layers:   Int     = 3,
  iceCream: Boolean = False
)

// Same feature working with method calls
o.manyArguments(
  abc    = 0,
  abcOne = 1,
  abcTwo,
  abcThree = 3,
  abcFour  = 4,
  abcFive  = 3
)
```

And here's how to use my version:

```
// Add this to .../project/plugins.sbt
resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

addSbtPlugin("com.danieltrinh" % "sbt-scalariform" % "1.3.0-SNAPSHOT")
```

See the [plugin](https://github.com/daniel-trinh/sbt-scalariform) for how to configure formatting options, and the [Scalariform readme](https://github.com/daniel-trinh/scalariform)
for available formatting options.

Since this is an ongoing project, there will be more updates to come.

<!-- 
The rest of this article goes into great detail about my motivation to work on Scalariform,
the importance of auto-formatters in general, and some of the design dilemmas of working on Scalariform.

Before I get into what I've improved, I'll go over some reasons why Scalariform
is important.

## The Importance of Auto-Formatting
[Many](https://code.google.com/p/google-styleguide/ "Google's style guides for various languages") [companies](http://twitter.github.io/effectivescala/ "Twitter's style guide for Scala") [have](https://github.com/styleguide/ruby "Github's style guide for Ruby") coding style guidelines for a reason. [I wrote about this](/blog/2014/01/26/maintaining-a-large-code-base-part-3/) in great detail in my article about programming languages and code bases. My general conclusion is that having many ways of visualizing and formatting source code
is not a good thing -- the more ways to format the code, the more ways a reader of the
code will have to learn to read the code to understand it. 

The following code example is perfectly valid scala code... but it's not at all readable.

```scala
class Coffee {
  val sugarCubes = 20
  val isCaffeinated = false

  def energyBoost = {
    if (caffeinated) 
      100 * sugarCubes
    else
      0
  }
}
```

The output from running Scalariform for this result is the same as the previous example:
```
class Coffee {
  val sugarCubes = 20
  val isCaffeinated = false

  def energyBoost = {
    if (caffeinated) 
      100 * sugarCubes
    else
      0
  }
}
```

Having something that will take care of the formatting for me is great -- spending time thinking about formatting, manually formatting code, and spending time
on code reviews about formatting is not a very productive use of time when there are actual business product related problems to solve, and especially since it's a problem that can be solved forever by using an auto-formatter along the lines of Scalariform or gofmt.

#### Formatting Ubiquity

Having an auto formatter to enforce a consistent style in a code base is important, because
once the maintainers of that code base have become accustomed to that style, they won't have to "readjust" their eyes to get used to. If the auto formatter for a programming language is nearly ubiquitous amongst those who use it, such as with Golang's [gofmt](http://research.swtch.com/gofmt), then even better -- users across the language will be able to read anyone's elses code that also uses the formatter.

While auto formatters such as Scalariform can't check everything that is in a styleguide
(it won't automatically reformat your imperative, state-mutating code into a functional style automatically), it will take care of the visual aesthetics formatting problem for you.


#### Scalariform Dillemas 
Scalariform has many configuration options, which goes against the design philosophy
of having one canonical representation so that everyone can quickly get accustomed to the same formatting. It also happens to significantly complicate the actual Scalariform formatting code. With Scalariform, there is one
global shared state -- the output source code. More configuration options means
more potentials to clash, and more special cases.

Unfortunately, Scala programmers come from many different backgrounds, and there isn't
a global consensus on what formatting styles programmers prefer. 

When I was gathering feedback for new Scalariform features, I posted an informal poll on Reddit, to gather feedback about 3 different formatting styles:

```scala Style One
class definition(
  a: Int = 1
  bb: String = ""
  ccc: Boolean = true
)
```

```scala Style Two
class definition(
  a:   Int     = 1
  bb:  String  = ""
  ccc: Boolean = true
)
```

```scala Style Three
class definition(
    a: Int     = 1
   bb: String  = ""
  ccc: Boolean = true
)
```

I was half-expecting everyone to say they prefered `Style Two`, but of course that wasn't the case. `Style One` was the most popular choice. `Style Two` was the second most popular,
and `Style Three` was by far the least -- not a single person said they preferred this one (although a few said they didn't mind any of the options).

I also concluded that a strong contributing factor on which style people prefer is
strongly related to just what they're used to reading -- Golang's gofmt happens to format
code in something similar to `Style Two`, and happens to be used by roughly 70% of all golang projects. If this really is the main factor, then it doesn't really make sense
to  -->