# Wireshark  

## Sniffing browser packets

### Getting to the correct interface  

It would be either the WI-FI interface or the Ethernet interface.

You can also open cmd and run `ipconfig`, and look for the single interface which is configured with `Default Gateway`. 

### Start, Stop, Restart capturing 

Once entering the interface, capturing starts immediately. 

To stop it, press Ctrl+E (or, in the navigation bar: Capture > Stop).  
To restart it, press again Ctrl+E (or, in the navigation bar: Capture > Start), and then click Continue without saving".  

### Go back to the interface page

Press Ctrl+W (or, in the navigation bar: File > Close (not Quit!)).

### Filter packets

#### By domain name
 
You can only filter by domain name when browsing to `http` domains, such as [http://example.com](http://example.com).  

To do that:

`http.host contains "example"`

#### By protocol
 
Filter http packets by:
 
`tcp.port==80`

Filter https packets by:

`tcp.port==443`

#### By source ip
 
Filter source ipv4 by:

`ip.src = 8.8.8.8`

Filter ipv6 source by:

`ipv6.src == 2001:4860:4860::8888`
