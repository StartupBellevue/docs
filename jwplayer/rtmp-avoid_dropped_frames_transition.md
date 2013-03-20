JW Player 5.10 - RTMP - Avoid dropped frames transition
================================

This player uses a method to deliver the best video quality for your computer (or device) that may sometimes be counterproductive.

When delivering VOD or LIVE over RTMP, the player uses three variables in order to switch between the different configured streams:
* Player width
* Bandwidth
* Dropped frames (due to CPU spikes)

Let's assume we have a LIVE broadcast with three differents qualities (streams):
* HI (960px, 800Kbps)
* STANDARD (680px, 400Kbps); default.
* LOW (680px, 200Kbps)


Basic transition logic
-------------------------
Supposing the player has a width of 680px, this should happen:
* Our bandwith is enough for the STANDARD quality (default; 400Kbps) and that's what we'll get.
* Our bandwith is not enough for the STANDARD quality (default; 400Kbps) and we'll get the LOW quality (200kbps).
* (HI quality is not taken into account as it is wider than the player)

Supposing you use fullscreen (and your screen has more than 960px wide), this should happen:
* Our bandwith is enough for the HI quality (800Kbps) and that's what we'll get.
* Our bandwith is not enough for the HI (or even STANDARD) quality and we'll get a lower quality.


How are dropped frames taken into account?
-------------------------
Added to the transition logic described before, if in any moment more than 10fps (loose) are dropped, JW will render that quality unusable ("Blacklisted level 'n'") and will not try to use it again for the next 20 seconds. 


Why is the player taking dropped frames into account?
-------------------------
The idea is to avoid presenting a video that cannot be supported the computer (or device) as it's does not have enough CPU performance to render it properly.


What's wrong with this?
-------------------------
The problem is, in my humble opinion, that the tresholds are low and there is no way to change them (as it can be done with [FLOWPLAYER](http://flash.flowplayer.org/plugins/streaming/bwcheck.html ) .

Some things that can get you to lose 10fps:
* Popups
* A new browser tab being opened
* Flash Ads
* MSN/GTalks popup windows
* etc.

Any of those things would get a degraded video experience for at least 20 seconds.

Solving this
-------------------------
As there is no easy way to modify the treshold, we will need to modify the source code. With our modifications, JW would never be aware of dropped frames (caution: this could cause new problems, as the feature would be lost).

> DISCLAIMER: This is the simplest and most horrible way to solve this; you shoud be able to think of a better one :)

**Prerequisites** (Xubuntu):
* [ADOBE FLEX 3.4](http://sourceforge.net/adobe/flexsdk/wiki/Download%20Flex%203/)
* [APACHE ANT](http://ant.apache.org/bindownload.cgi)
* [JW Player 5.10 Source Code](http://developer.longtailvideo.com/trac/browser/tags/mediaplayer-5.10)


### Editing :

In the file "src/com/longtailvideo/jwplayer/media/RTMPMediaProvider.as", around line 214, we should find:

    try {
    var bwd:Number = Math.round(_stream.info.maxBytesPerSecond * 8 / 1024);
    var drf:Number = _stream.info.droppedFrames;
    var stt:String = state;

Replace this:

    var drf:Number = _stream.info.droppedFrames;
   
For this:

    var drf:Number = 0;

### Compiling :

From the root folder of the source code:

    ant -buildfile build\build.xml

If the compiling is succesful you should have a directory named "bin-release" with the file in it "player.swf".




Side by side comparison (L: Old; R: New)
------------------------
![side by side comparison](http://i.imgur.com/CRMGc8I.jpg)
*the width and bitrate are the exact same as in the guide, but the screenshot was already taken*


Notes
-----------------------------
* It is common to have to make some changes in the code for the FLEX or ANT path; it depends on your system.
* If you do not wan't to compile the code, you can download the SWF [here](http://tinyurl.com/cd66yu3)
* I remember having read somwhere that this "dropped frame" transition logic would be deprecated in JW6, but im not sure.

Useful links
--------------
* http://www.longtailvideo.com/support/jw-player/jw-player-for-flash-v5/12535/video-delivery-rtmp-streaming#dynamicstreaming
* http://www.longtailvideo.com/addons/plugins/123/QualityMonitor
* http://www.longtailvideo.com/support/addons/quality-monitor/14627/quality-monitor-plugin-users-guide
