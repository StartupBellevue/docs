Realtime analitycs using Google Analitycs
=========================================

<!-- Set this in the bottom of your page - below JW code -->
<!-- 
	Use the default GA script (you can skip this, if you already are using
	GA)
-->
<script type="text/javascript">

  var _gaq = _gaq || [];
  // Set your UA number	
  _gaq.push(['_setAccount', 'UA-******-*']);
  // Disable the default pageview tracking
  //_gaq.push(['_trackPageview']);

  (function() {
    var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
    ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
  })();

</script>

<!-- 
	This is our code, where we set an interval of 60 seconds for a loop
     that checks if JW Player is in "PLAYING" state. If so, a event and
     a pageview are pushed to GA (you can customize this in any way you
     like
-->
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
