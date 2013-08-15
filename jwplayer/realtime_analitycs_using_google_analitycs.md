Realtime analitycs using Google Analitycs
=========================================

JW has an API that allows you to have many actions in response to the player events. Google analytics also provides a great API, is probably the best and least expensive (it's free!) of tracking this events.

This is a very simple example on how to analyze which users are "playing" videos (or a live stream).

The GA code
-----------

* Get an GA account and its code (you can skip this, if you already are using it)



	<script type="text/javascript">

	  var _gaq = _gaq || [];
	  // Set your UA number	
	  _gaq.push(['_setAccount', 'UA-******-*']);
	  // Disable the default pageview tracking - OPTIONAL
	  //_gaq.push(['_trackPageview']);

	  (function() {
		var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
		ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
		var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
	  })();

	</script>

Using the JW API and pushing to GA
----------------------------------

This is our code, where we set an interval of 60 seconds for a loop hat checks if JW Player is in "PLAYING" state. If so, a event and a pageview are pushed to GA (you can customize this in any way you like

	<script type="text/javascript">
		setInterval(
			function(){
				if (jwplayer().getState()=="PLAYING"){
					_gaq.push(['_trackEvent', 'Live streaming', 'Concurrente', 'Live streaming']);
					_gaq.push(['_trackPageview', 'concurrente_supermasita']);
				}
			}
		,60000);

	</script>
