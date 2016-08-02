#### Set up hidden services tor on linux  

#First  set up VM and use gninx not apache 

# add tor to sources.list
deb http://deb.torproject.org/torproject.org <DISTRIBUTION> main

# add gpg keys
gpg --keyserver keys.gnupg.net --recv 886DDD89
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -

sudo apt-get update
sudo apt-get install tor

nano /etc/tor/torrc 
# add these lines
HiddenServiceDir /home/ubuntu/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80

# restart 
sudo service tor restart 

#visit hidden services link
cat /var/lib/tor/hidden_services/hostname
# http://r3jrjbwu4zl2iget.onion/

# turn off clear web 
nano /etc/apache2/ports.conf 
Listen 127.0.0.1:80
service apache2 restart 

#check 
ss -tnlp
