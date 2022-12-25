---
categories: maintainability
comments: true
date: '2013-11-09T00:00:00Z'
title: 'Maintaining a Large Code Base, Part 2: Service Oriented Architecture'
---

[_Previous Part: Backstory and the Basics_](/blog/2013/11/05/maintaining-a-large-code-base-part-1/)

How do you deal with a code base that's too large to handle? Try making it smaller --
split it up into multiple smaller, separate code bases that communicate with
each other through well defined interfaces. When done with services
at the server level, this is typically known as Service Oriented Architecture (SOA). SOA is just about applying the practices of code decoupling, clear interfaces, and code reuse at the scale of servers.

#### Service Oriented Architecture, AKA Don't Repeat Yourself for Servers

Twitter, Netflix, and Amazon all started out with monolithic, tightly coupled
architectures in their infant years, and have all adopted the SOA approach as
they've grown.

In Twitter's case, they started out with stuffing all functionality into single Ruby
on Rails application, and later moved to the JVM, using Scala and Java, splitting
their product into smaller services along the way.

From an [interview with Alex Payne, a former Twitter engineer:](http://blog.redfin.com/devblog/2010/05/how_and_why_twitter_uses_scala.html)

> In the enterprise world, a service-oriented architecture is not new, but in Web 2.0
> it is crazy new science. With PHP or Ruby on Rails, when you need more functionality,
> you just include more plugins and libraries, shoving them all in to the server.
> The result is a giant ball of mud.
>
> So _anything that has to do heavy lifting in our stack is going to be an independent service._

They split one code base that handled the entirety of Twitter's functionality into several
services, including a queuing service, a social graph store, a people search service,
and a tweet streaming service, using Thrift (common RPC network protocol library) to tie their
services together. Their system has become much more reliable [ever](http://www.whatisfailwhale.info/)
[since](https://blog.twitter.com/2013/new-tweets-per-second-record-and-how).

Netflix [started off as a monolithic](http://techblog.netflix.com/2012/06/netflix-operations-part-i-going.html)
Java application, and split their code base off into smaller services. The change
also allowed them to split their engineering team into smaller teams on a per
service basis. Engineers wanting to integrate their service with another service no longer had to search
through mud to integrate with other features -- they only had to be concerned
about the interfaces.

Amazon started doing SOA as early back as 2002. According to a
[well known leaked rant by Steve Yegge](https://plus.google.com/+RipRowan/posts/eVeouesvaVX),
Jeff Bezos (CEO of Amazon) sent out a mandate to all engineering teams, requiring all
data and functionality to be exposed through services, with no hooks or backdoors for communicating
between services. Everything was to be a stand-alone service with a well defined API, and every engineer would have to abide by this new rule, unless they wanted to be fired.

Steve goes on to make a point that Bezos' reasoning for this was to be able
to sell Amazon's internal platform for managing servers (hence the no hooks thing),
now known to us today as Amazon Web Services (AWS), but I'm sure he was aware of the code maintainability
benefits of splitting products up, and how it leads to [smaller self contained teams](http://zurb.com/word/two-pizza-team).

If you are still wondering why this works so well... Well, it's almost impossible for one engineer to
understand every detail in a giant monolithic application. If an engineer
doesn't know how some code works, the chances of them being able to reliably
modify it are slim. By splitting up responsibilities into services, engineers
can be assigned to work on specific services, limiting how much they need to know. So it helps in the division of labor in a larger software organization.

As Steve mentions in his rant, it's also a good way of dogfooding services, which happens
when the team behind one service has to integrate with the interface of another
team's interface.

The other obvious benefit of splitting things into strict independent services is
the potential for open sourcing the services, just as [Twitter](http://twitter.github.io/) and [Netflix](http://netflix.github.io/#repo)
have done. Open source means more contributors (if it's good enough for others to use) and
more dogfooding. Or, if it's _really_ good, you could sell the service, like Amazon has done
with AWS.

#### Some Basic Service to Service Examples

An example of SOA that's perhaps more well known is the movement of
generating GUIs from the server to the client, and having the server side code serve pure data
over HTTP, websockets, or some other protocol. This is the idea behind
the Javascript GUI frameworks such as Backbone.js and Angular.js. By building
UI on top of an API that servers data, the API gets used and tested in the process
of building the API. If the API is good enough, it can be opened to the public!
Now if users don't like your GUI, they can build their own.

That same API could also be [split into a separate](http://spray.io/wjax/#/41) service that talks to other services that handle the actual business logic. To directly paraphrase from the link, the API layer would only handle authentication, request routing, serialization and deserialization of objects, and request caching.

So what if you aren't building servers or applications that have to talk over the network? The same principle of splitting things into separate code bases and communicating through a common
protocol can be applied to smaller scale programs as well.

#### The Future of IDEs

Ever used an IDE such as Eclipse, Intellij Idea, or Visual Studio? If you've ever thought to yourself,
_"man, I really like the IDE auto completion, incremental compilation, and refactoring tools,
but I wish I could be using Emacs / Vim / Sublime Text / Microsoft Word for the text editing instead."_, it's technically possible to write an IDE that would let you do this, even if the aforementioned IDEs can't.

[Gocode](https://github.com/nsf/gocode),
[Ensime](https://github.com/aemoncannon/ensime), and
[Slime](http://common-lisp.net/project/slime/) are IDE daemons for the Golang, Scala, and Common Lisp programming languages, respectively. They communicate
over a protocol to any capable text editor -- the daemon receives code from the editor and some commands, executes the commands on the specified code, and returns any necessary text deltas back to the text editor.

Speaking of programming languages...

[_Next Part: Programming Languages_](/blog/2014/01/26/maintaining-a-large-code-base-part-3/)
