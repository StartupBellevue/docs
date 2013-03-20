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
The problem is, in my humble opinion, that the tresholds are low and there is no way to change them (as it can be done with FLOWPLAYER: http://flash.flowplayer.org/plugins/streaming/bwcheck.html ).

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

[i]ACLARACIÓN: voy a hacer la modificación más básica, más rústica posible; Uds. pueden hacer las mejoras que se les ocurran, sobre todo si saben AS[/i].

Prerequisitos:
[list]
[li]En mi caso, lo compilé en Xubuntu, pero se puede hacer lo mismo en Windows o Mac.[/li]
[li] ADOBE FLEX 3.4: [url=http://sourceforge.net/adobe/flexsdk/wiki/Download%20Flex%203/]http://sourceforge.net/adobe/flexsdk/wiki/Download%20Flex%203/[/url][/li]
[li]APACHE ANT: [url=http://ant.apache.org/bindownload.cgi]http://ant.apache.org/bindownload.cgi[/url][/li]
[li]Código fuente de JW. En nuestro caso, vamos a usar la versión 5.10 : [url=http://developer.longtailvideo.com/trac/browser/tags/mediaplayer-5.10]http://developer.longtailvideo.com/trac/browser/tags/mediaplayer-5.10[/url][/li]
[/list]

EDITAMOS:
[list]
[li]Vamos a buscar el archivo "src/com/longtailvideo/jwplayer/media/RTMPMediaProvider.as" y aproximadamente en la línea 214 encontraremos:
[/li][/list]
[code]
try {
  var bwd:Number = Math.round(_stream.info.maxBytesPerSecond * 8 / 1024);
	var drf:Number = _stream.info.droppedFrames;
	var stt:String = state;
  [/code]
[list]
  [li]Reemplazaremos :[/li][/list]
[code]
  var drf:Number = _stream.info.droppedFrames;
  [/code]
  por 
  [code]
  var drf:Number = 0;
  [/code]
  [i]Insisto en que esta es una solución muy rústica, porque lo que hacemos transformar esa variable en una constante. Seguramente a Uds. se les pueden ocurrir mejores maneras de lidiar con este problema[/i] :)


COMPILAMOS:
[list]
[li]Desde la consola, parados en la carpeta raíz del código, ejecutamos:
   [/li][/list][code]ant -buildfile build\build.xml[/code][list][list][/li]
[li]Si todo sale bien, [b]tendremos una carpeta "bin-release" con el archivo "player.swf"[/b][/li]
[li]El nuevo "player.swf" es el que usaremos para llamar a nuestro player FLASH.[/li]
[/list]

[b]De un lado, el player normal. Del otro, el recompilado[/b]
[IMG]http://i.imgur.com/CRMGc8I.jpg[/img]
[i](el "width" y "bitrate" de la captura no corresponde con la guía, pero me dí cuenta después de escribir la guía; sepan disculpar :P )[/i]


[b]NOTAS[/b]
[list]
[li]Es normal tener que hacer alguna modificación al código para que reconozca el path de FLEX o de ANT; varía de equipo en equipo. Si alguno le pasa, puede consultar.[/li]
[li]Si no tienen ganas de compilar este código, lo pueden descargar de acá: [URL=http://tinyurl.com/cd66yu3]http://tinyurl.com/cd66yu3[/URL][/li]
[li]Si mal no recuerdo haber leído en algún foro de Longtail, esta funcionalidad se iba a depreciar con JW6; pero no sé que pasó finalmente.[/li]
[/list]

[b]Links interesantes[/b]
[list]
[li][url=http://www.longtailvideo.com/support/jw-player/jw-player-for-flash-v5/12535/video-delivery-rtmp-streaming#dynamicstreaming]http://www.longtailvideo.com/support/jw-player/jw-player-for-flash-v5/12535/video-delivery-rtmp-streaming#dynamicstreaming[/url][/li]
[li][url=http://www.longtailvideo.com/addons/plugins/123/QualityMonitor]http://www.longtailvideo.com/addons/plugins/123/QualityMonitor[/url][/li]
[li][url=http://www.longtailvideo.com/support/addons/quality-monitor/14627/quality-monitor-plugin-users-guide]http://www.longtailvideo.com/support/addons/quality-monitor/14627/quality-monitor-plugin-users-guide[/url][/li]
[/list][/list]
