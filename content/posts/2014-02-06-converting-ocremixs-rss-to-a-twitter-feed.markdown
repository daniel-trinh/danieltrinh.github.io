---
categories: scala project
comments: true
date: '2014-02-06T00:00:00Z'
title: Converting the OverClocked Remix RSS to a Twitter Feed
---

[OverClocked Remix (OCRemix)](http://ocremix.org) is a website dedicated to serving high quality remixes of music from video games. The RSS feed is occasionally updated with the latest ten new songs that are posted on the site.

This post is about a server daemon project I wrote a year ago -- it polls the RSS feed of OCRemix periodically, and converts new results to a Twitter feed. The results can be seen at [@newOCRemixes](https://twitter.com/newOCRemixes) on Twitter. It's been running on Heroku 24/7 for the past year without any problems. Source code is [here](http://github.com/daniel-trinh/ocremix).

The rest of this post goes into the design and development of this project, so if that doesn't sound interesting to you, now is a good time to hop off this train.

#### Motivation and design

I've been listening to music off of OCRemix ever since 2002. The way I used
to find new songs was to repeatedly visit their website, look for new songs,
and then click through three or four links for every new song that was posted.

I got tired of checking the website, only to end up finding out there were no new songs,
or finding out there are ten new songs, and having to click 30 times to hear all of them.

So I set out to figure out how to make it much, much easier to hear new songs and download
them if I liked them.

#### Existing OCRemix Feeds

OCRemix has it's [own Twitter feed](https://twitter.com/ocremix), which they will use
to post new remixes, but it's not exclusive to providing just new songs:

{{< figure class="center" src="/images/ocremix/official_unrelated.png" width="500" height="450" title="Official OCRemix Twitter Feed" alt="Official OCRemix Twitter Feed picture" >}}

Twitter happens to have this nice feature that they call [cards](https://dev.twitter.com/docs/cards). Cards are essentially a feature that detect certain types of URLs, and display information within the tweet that would only be accessible from visiting the URL. One of the
supported card types are Youtube links, which will embed a Youtube video into a Tweet
whenever a Youtube link is detected.

Sometimes the links to new songs include a Youtube link to the remix for easy listening...

{{< figure class="center" src="/images/ocremix/official_link.png" width="400" height="450" title="Official OCRemix Twitter Feed -- Embedded Youtube" alt="Official OCRemix Twitter Feed picture, embedded Youtube" >}}

and sometimes they don't:

{{< figure class="center" src="/images/ocremix/official_no_link.png" width="400" height="450" title="Official OCRemix Twitter Feed -- no Embedded Youtube" alt="Official OCRemix Twitter Feed picture, no Youtube" >}}

They also have their own [Youtube channel](https://www.youtube.com/channel/UC8Wh7qpPGJm3K1eZ6DHjpKA)
where new songs are posted, but similarly to their Twitter feed, it is not exclusive to new songs.

#### My OCRemix Feed

Since I was already using Twitter as a news feed, I chose that as my target platform for
receiving new song notifications. The following is a list of some of the things I wanted
out of my Twitter feed system.

> - Only new songs should be posted -- no reposts.
>
> - Only songs should be posted -- nothing else but new OCRemix songs.
>
> - Every single song must contain a link to the Youtube song link.
>
> - Show the video game, remixer, and composer the song is remixed from,
>   if it fits in 140 characters along with the Youtube link.
>
> - This system should not require any input from me (automated).
>
> - If something is broken, I should be notified of the breakage.

Initially, I wanted to implement direct download links within each Tweet in order to simplify the process of downloading the mp3 for new songs. However, because ocremix.org is non-profit and receives advertising money from the main website to pay for bandwidth, I chose to just link to their site to download out of politeness.

This is what the final version of my implementation looks like:

{{< figure class="center" src="/images/ocremix/my_version.png" width="500" height="550" title="My Unofficial Twitter Feed" alt="My Unofficial Twitter Feed picture" >}}

Assuming this information fits into the 140 character limit, each one of these song tweets is composed of the following:

    songId     - Integer, official remix number (currently at 2827 at the time of writing).
    title      - Song title, given to the song by the remixers.
    remixers   - The remix artists that created the remix.
    composers  - The original artists that created the original song.
    youtubeUrl - Link to the remix on Youtube.
    writeupUrl - Official OCRemix link with remix information and a download link.

#### Programming Details

The actual project was implemented using Scala. The main libraries used were
Dispatch for OAuth and talking with Twitter's API, and [Akka](akka.io) for the periodic
polling of the RSS feed.

##### Fitting Info into a Tweet

Trying to detect whether or not a Tweet will fit in 140 characters is tricky, especially
if there are URLs in the Tweet. Twitter will automatically shorten URLs for you -- the caveat
is that the length of the shortened URL is what counts to the 140 character limit,
and not the size of the original URL.

So how do you figure out what the shortened URL size will be?
Twitter happens to have a [configuration API](https://dev.twitter.com/docs/api/1.1/get/help/configuration) that will tell you the current length
of URLs (https links are one character longer than http). Since this number can
change, it can't be hardcoded into the system. It's also not a number that is likely
to change often.

To take this varying URL length into consideration, the previously known shortened URL length is cached in memory for quick usage. I poll the configuration API
every 24 hours to check for and update the URL length cache if necessary.

This polling is done using Akka's scheduler:

```scala
val twitterConfigUpdaterSchedule = MySystem().scheduler.schedule(
  initialDelay = 0 seconds,
  frequency    = 1 day,
  receiver     = configUpdater,
  // If I was writing this again, I probably wouldn't use a "doit" message
  // for initiating an update.
  message      = "doit"
)
```

Sometimes there's just too much information to include in 140 characters. Going back
to my design goal of making it easy to hear new songs, the Youtube link is the
most important thing to include in the Tweet.

Here's a shortened version of the code that figures out what to put into a tweet:

```scala
// The youtube URL is in every possible one of these.
lazy val v0 = Tweet(songId, title, remixers, composers, youtubeUrl, writeupUrl)
lazy val v1 = Tweet(songId, title, remixers, youtubeUrl, writeupUrl)
lazy val v2 = Tweet(songId, title, youtubeUrl, writeupUrl)
lazy val v3 = Tweet(songId, title, youtubeUrl)
lazy val v4 = Tweet(songId, youtubeUrl)

// Start with the longest possible, then slowly go to the shortest.
if (v0.isTweetable)
  v0
else if (v1.isTweetable)
  v1
else if (v2.isTweetable)
  v2
else if (v3.isTweetable)
  v3
else
  v4
```

Unless Twitter plans on upping the shortened URL length to ~135 characters,
or the ids start incrementing in several orders of magnitude per new remix,
the smallest tweet content (`songId`,`youtubeUrl`) is good enough.

_There's a reason songId is included in all of the possibilities as well, more on that later._

##### Receiving Error Notifications

In order to make sure I received a notification whenever something breaks, every single
method that could cause an error returns an `Either[_, String]`, where
the `Right` case stores the successful result, and the `Left` case stores a string explaining what went wrong. Whenever a method is completed
with the `Left` case, a direct message is sent to my own personal Twitter handle,
with information about the failure.

Here's what error messages look like when they reach my Twitter inbox:

{{< figure class="center" src="/images/ocremix/error_messages.png" width="600" height="450" title="All in one place" alt="Error Messages in my Twitter Inbox" >}}

Some examples of methods that might fail that I want to be
alerted about if they do:

```scala
  // Sends a private message to someone on Twitter
  def directMessage(user: TwitterHandle, message: String): Future[Either[String,String]]

  // Tweets a new post
  def statusesUpdate(tweet: Tweetable): Future[Either[String,String]]

  // Retrieves the last few Tweets from a user's timeline
  def userTimeline(
    userId:     String,
    screenName: String,
    count:      Int
  ): Future[Either[String,String]]

  // Retrieves configuration information about Twitter
  def helpConfiguration: Future[Either[String, TwitterConfiguration]]
```

(In retrospect, just using Future[String] would have been enough, since it
stores the exception information in case of failure... but keep in mind I _did_ write this code as a novice just as I started learning Scala).

##### Polling and Parsing the RSS Feed

There's nothing particularly fancy about this part -- it mostly consists of
periodically polling [OCRemix's RSS](http://ocremix.org/feeds/ten20/) and
parsing out new songs to post on Twitter.

Here's the code that schedules an RSS polling check every half hour:

```scala
val rssPollerSchedule = MySystem().scheduler.schedule(
  initialDelay = 0 seconds,
  frequency    = 30 minutes,
  receiver     = rssPoller,
  message      = "doit"
)
```

In order to figure out whether or not a song is new, I send a request to Twitter's API to retrieve the `@newOCRemixes` Twitter Timeline, check the last Tweet's `songId`, and make sure any new song I post happens
to have a songId greater than the one on the timeline.

##### Things to Improve

All of this _mostly_ works. There are a few problems that were more complicated
to solve than the time I wanted to spend on this.

One of these problems happens to be that the RSS feed
only stores the 10 latest new songs -- if there's ever more than 10 songs posted
within a 30 minute interval, any songs prior to the last 10 will not get posted.
This has only happened once since I've launched this project, but there's not
much that can be done about it, short of asking for the OCRemix owners to not post
so many songs at once, or up the number on their RSS feed.. or polling the website
itself, and parsing the song URLs from raw HTML.

Another problem (that hasn't actually been much trouble so far) is that the error
message reporting is dependent on Twitter's API working. If there's a problem with
the `directMessage` call to send me a message on errors, I won't be notified. I could have solved this by setting up a secondary notification system such as email, but errors happen so rarely that it wasn't worth the trouble.

The third and last problem that I've run into is actually not one I can do much about. The Twitter card feature for Youtube videos sometimes won't work -- a few Tweets just won't embed a Youtube video into the post, even if there's a valid Youtube URL in the message.

Having said that, I'm pretty happy with the results -- it's been working without a hitch for the last year since it's been deployed.
