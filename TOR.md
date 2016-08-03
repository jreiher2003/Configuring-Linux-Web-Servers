# Set up hidden services tor on linux  

###First  set up VM and use gninx not apache 

### add tor to sources.list
`deb http://deb.torproject.org/torproject.org <DISTRIBUTION> main`  


###add gpg keys
`gpg --keyserver keys.gnupg.net --recv 886DDD89`  
`gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -`  


`sudo apt-get update`  
`sudo apt-get install tor`  

`nano /etc/tor/torrc`  
### add these lines
`HiddenServiceDir /home/ubuntu/tor/hidden_service/`  
`HiddenServicePort 80 127.0.0.1:80`  

### restart 
`sudo service tor restart `  

###visit hidden services link
`cat /var/lib/tor/hidden_services/hostname`  
#### http://r3jrjbwu4zl2iget.onion/

### turn off clear web 
`nano /etc/apache2/ports.conf `  
##### add this line
`Listen 127.0.0.1:80`  
`service apache2 restart `  

###check 
`ss -tnlp`  

### set proxychains.conf
`nano /etc/proxychains.conf`   
change *strict* to *dynamic_chain* 
add this to bottom to config socks5 also  
`socks5 127.0.0.1 9050`  


### check tor if running 
`service tor status `  
`service tor start `  

### check dns leaks
`proxychains firefox www.site.com `  
`service tor stop`  

###route all traffic through the proxychains tor
`proxychains nmap ip port <args>`  

#### set open dns server
`nano /etc/dhcp/dhclient.conf`  
use opendns.com  
`service network-manager restart`  