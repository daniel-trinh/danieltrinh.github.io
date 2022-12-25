---
categories: golang project
comments: true
date: '2016-07-12T00:00:00Z'
title: Filtering Duplicate Messages in Twitch Chat
---

I recently built this toy command line interface (CLI) app for fun. You can see a higher resolution video of it in action by clicking the image below.

[![Twitch Chat Filterer](http://i.imgur.com/m50Kii1.gif)](https://www.youtube.com/watch?v=i8sRO7_qvOY 'Twitch Chat Filterer')

The video above is a snapshot of the CLI in action during the middle of a speed run of the Nintendo 64 game The Legend of Zelda: Ocarina of Time, hosted on [Twitch.tv](https://www.twitch.tv/). It shows the filtering in action, in the middle of a burst of the `WutFace` spam, which you can see in the lower left of the above video.

#### Twitch.tv Overview

For the uninitiated, [Twitch.tv](https://www.twitch.tv/) is a streaming video website that allows anyone to share live video with viewers over the internet, with twitch. The most popular use case by a wide margin for streaming players playing video games, although it is also sometimes used for sharing live music, where the video streamer might be playing an instrument (or instruments, if it's a band.

Several thousand people can be watching the same video stream at the same time -- it's not uncommon to see streams that have 100,000 simultaneous viewers. Since viewers who watch the same video stream are also put into the same chat room (there's no chat room splitting that happens automatically), the chat room messages can become incredibly difficult to parse.

To (sort of) solve this problem, I wrote a simple CLI Twitch/IRC chat client that supports the ability to filter out duplicate messages within a sliding time window in realtime.

#### Implementation Details

Although this was built specifically for Twitch, it works with any chat that supports IRC as a protocol.

The fully functional IRC client was written in Golang. The choice of Golang was purely out of my desire to learn the language. If I was to build something beyond for toy usage, I'd probably write it as a Chrome extension, since native Twitch chat is in the browser, and it's where users are accustomed to chatting.

I used [termui](https://github.com/gizak/termui) for the UI. The usecase for termui is pretty niche -- I'd say it's useful for interfaces when a browser is out of the question, such as while SSHed onto a remote server. Unless performance is a problem or golang is absolutely necessary, [blessed-contrib](https://github.com/yaronn/blessed-contrib) using JavaScript looks to be a better option for building CLI UIs. I'd avoid building CLI UIs over browser based ones in general though -- these frameworks for building CLI UIs is not like working with the DOM in the browser, and the kinds of UIs you can build using `termui` or `blessed-contrib` are limited in scope.

There are the following widgets in this client:

1. Message Rate Monitor (Rate of messages over sliding window)

2. Message Stats Monitor (Min, Max, Avg messages over sliding window)

3. Duplicate Message Aggregator (List of messages sorted by number of occurrences over sliding window)

The trickiest implementation detail was the duplicate message aggregator. It involved maintaining a sorted list of `(message, count)` pairs in realtime as messages arrived, and then decrementing / removing the message from the sorted list of (message, count) pairs, while also preventing concurrent updates from malforming the sorted list counts.

Despite the complexity, Golang made this process quite easy. Concurrent updates were handled by serializing all updates through a queueing channel, and goroutines with `time.Sleep(duration)` were used to decrement counts.

#### Further Thoughts on Golang

Golang was a bit of a pain at times -- the lack of generics in the language shows. I had to duplicate methods like `math.Max` for `int` types, since the builtin `math.Max` only works with `float64`. Also, the extra overhead of thinking about when to pass a pointer to a struct vs just the struct got in the way of thinking about the higher level functionality of the application.

On the flip side, I spent almost no time learning Golang and was able to build something functional almost immediately. This hasn't been the case with other languages I've learned, such as `TypeScript` or `Scala`. The expediency experienced with Golang comes from two things -- firstly, the amazing tooling that has gone into the language ecosystem, and secondly, the sheer simplicity of the language and lack of language features.

I wouldn't use Golang as my first language choice for prototyping a product, but for system level programming, network intensive applications, or performance critical backends, Golang seems like the perfect choice for development. I'll stick to Python, Scala, or TypeScript for prototyping products quickly, and use Golang when the extra performance is necessary.

#### Using the Client

The source code and instructions for usage of the project described earlier can be found at my github page [here](https://github.com/daniel-trinh/twitch_chat_filter). All that is needed is a golang installation and a Twitch account.
