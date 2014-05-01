Wowza + Varnish: Blocking HLS for non mobile devices
----------------------------------------------------

I had a situation where some sites stole a live streaming by using HLS using a Flash video player (like JW6) published in other sites. That live stream was served to desktop players with RTMP and HLS was only used by IOS. RTMP couldn't be stole "Domain Lock" control was implemented.

After contacting Wowza support, the only solution we were offered was that we develop a service with a token, in order to validate every player. The solution was pretty standard, but there was no chance to develop this and we couldn't disable IOS support.


My workaround: installing Varnish as a proxy for port 80 and HLS (RTMP and RTSP wouldn't be affected as they use other ports).

1. Install Varnish and set it to use port 80.

2. Enable Wowza HLS over port 8080 (or other) in "conf/VHost.xml":
```xml
	<Root>
		<VHost>
	                <HostPortList>
	                        <HostPort>
	                                <ProcessorCount>16</ProcessorCount>
	                                <IpAddress>*</IpAddress>
	                                <!-- Separate multiple ports with commas -->
	                                <!-- 80: HTTP, RTMPT -->
	                                <!-- 554: RTSP -->
	                                <!--<Port>80,554,1935</Port>-->
	                                <Port>8080,554,1935</Port>
```

3. Create a VCL with rules similar to this (adjust according to your configuration):
```javascript
	backend default {
	  .host = "127.0.0.1";
	  .port = "8080";
	}
	acl mynetworks
	{
	        "localhost";
		"xxx.xxx.xxx.xxxx"/xx; 
	}
	sub vcl_recv {
	
		if (
			# Is this request coming from your networks?                
			!client.ip ~ mynetworks
	
			# Is it a mobile user agent? (extended as needed)
	
	                && !req.http.User-Agent ~ "(?i)(iphone|ipad)"
	
	                # POST needed by RTMPT - This is handled afterwards by Wowza's "Domain Lock"
	                && req.request != "POST"
		)
		{
			error 403 "Forbidden";
		}
		else {
			if ( 
				# Is this coming from one my sites?
				req.http.referer ~ "(mysite1.com|mysite2.com)"		
	
				# Does it have an empty referer? This could be to an IOS player doing a GET over the stream URL, and that's fine for us.
				|| req.http.referer ~ "^$"
			)
			{
				# If connection is legit, then we use PIPE in order to establish a TCP proxy
				return (pipe);
			}
			else {
				error 403 "Forbidden";
			}
	
		}
		
		error 403 "Forbidden";
	
	}
```
