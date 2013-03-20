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


Supposing the player has a width of 680px, this should happen:
* Our bandwith is enough for the STANDARD quality (default; 400Kbps) and that's what we'll get.
* Our bandwith is not enough for the STANDARD quality (default; 400Kbps) and we'll get the LOW quality (200kbps).
* (HI quality is not taken into account as it is wider than the player)


Cuando usemos pantalla completa, si la misma tiene un ancho superior a 960px, cuando reproduzcas la transmisión pueden pasar lo siguiente:
[list]
[li]Que nuestro ancho de banda sea suficiente para la calidad alta (800Kbps) y se muestre la misma.[/li]
[li]Que nuestro ancho de banda no sea suficiente para la calidad alta (o incluso la media) y se degrade a una calidad inferior.[/li]
[/list]

How are dropped frames taken into account?
-------------------------
Sumada a la lógica de transición comentada antes, si en algún momento se pierden más de 10fps, JW marcará la calidad ("level") como inusable ("blacklisted level 'n'") y no volverá a intentar usarlo durante unos 20 segundos.

Why is the player taking dropped frames into account?
-------------------------
La idea es que si tu computadora (o dispositivo) no tiene capacidad de CPU para reproducir una calidad en particular, automáticamente te muestre una inferior.

What's wrong with this?
-------------------------
El problema de cómo JW maneja los "dropped frames" es que tiene umbrales de trabajo que, a mi criterio, son muy bajos y no hay manera de modificarlos (como sí se puede en [url=http://flash.flowplayer.org/plugins/streaming/bwcheck.html]FLOWPLAYER[/url]).
Algunos motivos que te pueden hacer perder 10fps fácilmente son: 
[list]
[li]Una ventana emergente.[/li]
[li]Un mensaje de MSN, GTalk, etc.[/li]
[li]Abrir una pestaña nueva en el navegador.[/li]
[/list]
Si te encontrases con alguna de las situaciones antes descriptas, tendrías mínimo 20 segundos de calidad degradada.

[b]¿Cómo solucionamos esto?[/b]
Como decía antes, en JW Player 5 no existe manera de configurar los parámetros para la transición por pérdida de frames. Por tal motivo, lo único que podemos hacer, es recompilarlo cambiando algunas cosas en código. 
En nuestro caso, vamos a "hacerle creer" al player que nunca se pierden frames. Obviamente, esto puede generar nuevos problemas; evalúen si se justifica o no el cambio.

[b]Manos a la obra[/b]
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
