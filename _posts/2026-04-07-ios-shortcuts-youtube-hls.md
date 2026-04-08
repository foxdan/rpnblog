---
layout: post
title: Using iOS Shortcuts for ad-free AirPlayable YouTube
---
Two years ago while walking from a plane to the terminal I dropped my "rugged"
Android device less than a metre onto the tarmac and it smashed. I took a punt
on moving to an iPhone. I was worried about the walled garden I'd heard so much
about and believed, I plan to write about that experience more generally in a
future post (spoiler: it's mostly unfounded).

This post is going to be about my dislike of the YouTube experience out of the
box and how I was able to solve it with a solution that doesn't require any
third party apps, dodgyness or developer options.

Shortcuts
---------
Shortcuts is a native built-in macro/scripting tool for iOS (and macOS) and it
can be utilised completely on device. It has been written about a lot I'm sure,
but when tinkering with my new phone I remember being taken aback by a few small
but clearly incredibly powerful functions it has in it's library:

 * Write arbitrary JavaScript to be run on a page within Safari
 * Make HTTP requests
 * Parse and manipulate JSON
 * Run a remote SSH command passing input and receive the output
 * Base64 encode/decode
 * Hash functions

"Shoot, a fella could have a pretty good weekend in Vegas with all that stuff"

YouTube on iOS
--------------
While I have a YouTube Premium subscription and get on just fine with using it
on desktop in a browser I really disliked the native app experience on iOS. So
I migrated to using it within Safari, better experience but still with it's
issues for me. I'm very much a vanilla native experience enjoyer, that means I
want a no-frills HTML5 video player experience.

When you go full-screen it does transition to the native player (which supports
picture in picture). Funnily if you're not logged into a premium account YouTube
disables this functionality with a [declaration user side][1]. Shortcuts to flip
this flag are common and widespread, neat to see. Anyway I was happy enough with
just using YouTube in Safari for now.

The AirPlay Problem
-------------------
Now if I thought I disliked my experience on iOS it was nothing compared to the
awfulness of YouTube on Apple TV. Not to worry, I can simply use Safari on my
phone and use AirPlay to get what I want on the big screen.

That's when you discover YouTube gimps the quality of playback using AirPlay to
720p. What gives, this is clearly a synthetic restriction enforced with
user-side JavaScript.

Very frustrating, but there was an off the shelf solution available, a neat
Safari extension named [Vinegar][2]. It does everything I wanted and it was
cheap at €2. Problem solved!...

Opacity
-------
Anyone who knows me knows I loathe a black box. I had to know how it worked,
it's my machine and this is client side stuff. The next problem is how do we
get developer debugging tools going on Safari on an iPhone, turns out it's
amazingly simple; plug the phone in over USB and click the develop menu on
desktop Safari!

The meat of it is a single API request to a sparsely documented endpoint:
`https://www.youtube.com/youtubei/v1/player`.

Researching this a bit further it's the endpoint leveraged by the likes of
[yt-dlp][3]. You can spend hours playing with this endpoint and I'd recommend
it, it's fun. But in the end for our purposes it can all be boiled down to this:

```
$ curl 'https://www.youtube.com/youtubei/v1/player' \
-X 'POST' \
-H 'Content-Type: application/json' \
--data-binary '{"context":{"client":{"clientName":"IOS","clientVersion":"20.03.02"}},"videoId":"T4Upf_B9RLQ"}' 2>/dev/null |jq '.streamingData.hlsManifestUrl'
"https://manifest.googlevideo.com/api/manifest/hls_variant/expire/.../file/index.m3u8"
```

That's it, that's all we need. The returned URL can be opened in Safari and the
video will play without ads, without PiP restrictions, without AirPlay
restrictions. Enabling the media stats on my Apple TV I can see we're getting up
to 4k video playing directly in the native player and being a HLS stream it's
playing from the Apple TV without proxying through the phone!

*Note on the `clientVersion`: When I first built this over a year ago a
different version was used, then of course one day it stopped working, luckily
all that was required was bumping this to what I found currently being used by
yt-dlp*

KISS
----
Normally I'd probably be done here I know how it works, I can sleep well. But
knowing what's available in Shortcuts toolbox and how simple this really is
there was an opportunity for some more fun, we can rebuild all of this ourselves
directly on the iPhone.

Unfortunately there's no nice way of sharing the "source" of a shortcut.

[You can download the Shortcut directly here.][4]

And here's an ASCII view of what you get in the Shortcuts UI:
```
  ┌─────────────────────────────────────────────────┐
  │  Receive  URLs  from  Share Sheet               │
  │  If there's no input:                           │
  │  Stop and Respond   Gobshite                    │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get  Query  from  Shortcut Input               │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get dictionary from  Component of URL          │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get  Value  for  v  in  Dictionary             │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get contents of                                │
  │  https://www.youtube.com/youtubei/v1/player     │
  │                                                 │
  │  Method: POST                                   │
  │  Request Body: JSON                             │
  │                                                 │
  │  ┌───────────────────────────────────────────┐  │
  │  │ Key              Type         Value       │  │
  │  ├───────────────────────────────────────────┤  │
  │  │ context          Dictionary   (1 item)    │  │
  │  │   client         Dictionary   (2 items)   │  │
  │  │     clientVersion  Text       20.03.02    │  │
  │  │     clientName     Text       IOS         │  │
  │  │ videoId          Text         [vid_id]    │  │
  │  └───────────────────────────────────────────┘  │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get dictionary from  Contents of URL           │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get  Value  for  streamingData  in  Dictionary │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get dictionary from  Dictionary Value          │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get  Value  for  hlsManifestUrl  in Dictionary │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Get URLs from  Dictionary Value                │
  └───────────────────────┬─────────────────────────┘
                          │
  ┌───────────────────────▼─────────────────────────┐
  │  Open  URLs                                     │
  └─────────────────────────────────────────────────┘
```

*Et voilà!*

Final thoughts
--------------
This is a long post about what's essentially a single POST request at the end of
the day. Also amusing in the face of the ad-blocking arms race, though that was
never the intention here. I still have and will keep my YouTube Premium
account. Also I feel like Vinegar should have made this post about what it does
rather than charge €2 for a black box to bypass ads, it leaving a sour taste in
your mouth writes itself.


[1]: https://developer.apple.com/documentation/webkitjs/adding_picture_in_picture_to_your_safari_media_controls#2940032
[2]: https://apps.apple.com/ie/app/vinegar-tube-cleaner/id1591303229
[3]: https://github.com/yt-dlp/yt-dlp/tree/master
[4]: https://www.icloud.com/shortcuts/098bfc041abf423880ddbe00a4690463
