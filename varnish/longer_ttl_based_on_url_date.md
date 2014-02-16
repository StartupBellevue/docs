C{
#include <stdio.h>
#include <time.h>
}C

sub vcl_recv
{

        # Given  URL  like "http://example.com/archive/2014/01/31"
        if (req.url ~ "^/archive" ){

                # Create the "X-Today" header with the date format neeeded
                C{
                        time_t timer;
                        char buffer[25];
                        struct tm* tm_info;

                        time(&timer);
                        tm_info = localtime(&timer);

                        strftime(buffer, 25, "%Y/%m/%d", tm_info);
                        VRT_SetHdr(sp, HDR_REQ, "\010X-Today:", buffer, vrt_magic_string_end);

                }C

                
                # Create the "X-DateURL" header by striping "archive" and unwanted parameters from the req.url
                set req.http.X-DateURL = regsub(req.url, "^/archive/", "");
                set req.http.X-DateURL = regsub(req.http.X-DateURL, "\?.*$", "");

                # Compare the headers and set the TTL
                if (req.http.X-DateURL != req.http.X-Today){

                        set beresp.ttl = 365d;
                        set beresp.http.X-Cacheable-TTL = beresp.ttl;

                }
        }


}
