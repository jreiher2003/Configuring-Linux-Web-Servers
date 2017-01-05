# Ubuntu Email Server Configs
## Postfix 

1. production mail server
2. own domain name
3. set up mx records and a records to pointer to ip with regester  
4. local dns bind configure mx records a pointers text spf configure to secure mail server
5. emailsecuritygrader.com | mxtoolbox.com  
`hostname -f`  
`sudo nano /etc/hosts`   
```
127.0.0.1 localhost
127.0.0.1 mail.mydomain.com mail 
```  
`sudo nano /etc/hostname`  
`mail`  
`sudo reboot`  
`sudo apt-get install -y postfix`  
```
Postfix configuration -  
Internet Site  
System mail name: mail.mydomain.com  
```  
`sudo dpkg-reconfigure postfix`  
get you back to same Postfix configuration page  
```
Internet Site
change System mail name: fqdn ie. mydomain.com
Root and postmaster mail recipent:  use a user with root access / not root.  
Other destinations to accept mail - mail.mydomain.com localhost.mydomain.com, localhost, mydomain.com  
Force synchronus updates on mail queue - <NO>  
ifconfig - find local network 10.20.0.0/18  
Mailbox size limit 0 == no limit  
Local address extension character: +
Internet protocols = all 
```  
`sudo nano /etc/postfix/main.cf`  

```
# See /usr/share/postfix/main.cf.dist for a commented, more complete version


# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# TLS parameters
smtpd_tls_cert_file = /etc/ssl/certs/server.crt
smtpd_tls_key_file = /etc/ssl/private/server.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = mail.mydomain.com
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = mail.mydomain.com, localhost.mydomain.com, localhost, mydomain.com
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 10.20.0.0/18
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

home_mailbox = Maildir/
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_local_domain = mydomain.com
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
smtpd_client_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unknown_client_hostname
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_loglevel = 4
smtpd_tls_received_header = yes
smtpd_helo_required = yes
smtpd_helo_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_non_fqdn_hostname, reject_invalid_hostname
```
#### Next Generate certificates for tls
```
openssl genrsa -des3 -out server.key 4096  -enter passphrase  
openssl rsa -in server.key -out server.key.insecure  ^passphrase  
mv server.key server.key.secure  
mv server.key.insecure server.key
openssl req -new -key server.key -out server.csr`  Common Name: mydomain.com
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
-->Signature ok  
sudo cp server.crt /etc/ssl/certs 
sudo cp server.key /etc/ssl/private
sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/private/server.key'  
sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/server.crt'
```  
#### Config master.cf 
```
sudo nano /etc/postfix/master.cf  

#
# Postfix master process configuration file.  For details on the format
# of the file, see the master(5) manual page (command: "man 5 master" or
# on-line: http://www.postfix.org/master.5.html).
#
# Do not forget to execute "postfix reload" after editing this file.
#
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (yes)   (never) (100)
# ==========================================================================
smtp      inet  n       -       -       -       -       smtpd
#smtp      inet  n       -       -       -       1       postscreen
#smtpd     pass  -       -       -       -       -       smtpd
#dnsblog   unix  -       -       -       -       0       dnsblog
#tlsproxy  unix  -       -       -       -       0       tlsproxy
submission inet n       -       -       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
#  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
#  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       -       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
#  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
#  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
#628       inet  n       -       -       -       -       qmqpd
pickup    unix  n       -       -       60      1       pickup
cleanup   unix  n       -       -       -       0       cleanup
qmgr      unix  n       -       n       300     1       qmgr
#qmgr     unix  n       -       n       300     1       oqmgr
tlsmgr    unix  -       -       -       1000?   1       tlsmgr
rewrite   unix  -       -       -       -       -       trivial-rewrite
bounce    unix  -       -       -       -       0       bounce
defer     unix  -       -       -       -       0       bounce
trace     unix  -       -       -       -       0       bounce
verify    unix  -       -       -       -       1       verify
flush     unix  n       -       -       1000?   0       flush
proxymap  unix  -       -       n       -       -       proxymap
proxywrite unix -       -       n       -       1       proxymap
smtp      unix  -       -       -       -       -       smtp
relay     unix  -       -       -       -       -       smtp
#       -o smtp_helo_timeout=5 -o smtp_connect_timeout=5
showq     unix  n       -       -       -       -       showq
error     unix  -       -       -       -       -       error
retry     unix  -       -       -       -       -       error
discard   unix  -       -       -       -       -       discard
local     unix  -       n       n       -       -       local
virtual   unix  -       n       n       -       -       virtual
lmtp      unix  -       -       -       -       -       lmtp
anvil     unix  -       -       -       -       1       anvil
scache    unix  -       -       -       -       1       scache
#
# ====================================================================
# Interfaces to non-Postfix software. Be sure to examine the manual
# pages of the non-Postfix software to find out what options it wants.
#
# Many of the following services use the Postfix pipe(8) delivery
# agent.  See the pipe(8) man page for information about ${recipient}
# and other message envelope options.
# ====================================================================
#
# maildrop. See the Postfix MAILDROP_README file for details.
# Also specify in main.cf: maildrop_destination_recipient_limit=1
#
maildrop  unix  -       n       n       -       -       pipe
  flags=DRhu user=vmail argv=/usr/bin/maildrop -d ${recipient}
#
# ====================================================================
#
# Recent Cyrus versions can use the existing "lmtp" master.cf entry.
#
# Specify in cyrus.conf:
#   lmtp    cmd="lmtpd -a" listen="localhost:lmtp" proto=tcp4
#
# Specify in main.cf one or more of the following:
#  mailbox_transport = lmtp:inet:localhost
#  virtual_transport = lmtp:inet:localhost
#
# ====================================================================
#
# Cyrus 2.1.5 (Amos Gouaux)
# Also specify in main.cf: cyrus_destination_recipient_limit=1
#
#cyrus     unix  -       n       n       -       -       pipe
#  user=cyrus argv=/cyrus/bin/deliver -e -r ${sender} -m ${extension} ${user}
#
# ====================================================================
# Old example of delivery via Cyrus.
#
#old-cyrus unix  -       n       n       -       -       pipe
#  flags=R user=cyrus argv=/cyrus/bin/deliver -e -m ${extension} ${user}
#
# ====================================================================
#
# See the Postfix UUCP_README file for configuration details.
#
uucp      unix  -       n       n       -       -       pipe
  flags=Fqhu user=uucp argv=uux -r -n -z -a$sender - $nexthop!rmail ($recipient)
#
# Other external delivery methods.
#
ifmail    unix  -       n       n       -       -       pipe
  flags=F user=ftn argv=/usr/lib/ifmail/ifmail -r $nexthop ($recipient)
bsmtp     unix  -       n       n       -       -       pipe
  flags=Fq. user=bsmtp argv=/usr/lib/bsmtp/bsmtp -t$nexthop -f$sender $recipient
scalemail-backend unix  -       n       n       -       2       pipe
  flags=R user=scalemail argv=/usr/lib/scalemail/bin/scalemail-store ${nexthop} ${user} ${extension}
mailman   unix  -       n       n       -       -       pipe
  flags=FR user=list argv=/usr/lib/mailman/bin/postfix-to-mailman.py
  ${nexthop} ${user}
```

### Dovecot common configs
`sudo apt-get install dovecot-common -y`  
```
Create self signed cert? <Yes>   
Host name: mail.mydomain.com
_____
sudo nano /etc/dovecot/conf.d/10-master.conf

service auth {
  # auth_socket_path points to this userdb socket by default. It's typically
  # used by dovecot-lda, doveadm, possibly imap process, etc. Users that have
  # full permissions to this socket are able to get a list of all usernames and
  # get the results of everyone's userdb lookups.
  #
  # The default 0666 mode allows anyone to connect to the socket, but the
  # userdb lookups will succeed only if the userdb returns an "uid" field that
  # matches the caller process's UID. Also if caller's uid or gid matches the
  # socket's uid or gid the lookup succeeds. Anything else causes a failure.
  #
  # To give the caller full permissions to lookup all users, set the mode to
  # something else than 0666 and Dovecot lets the kernel enforce the
  # permissions (e.g. 0777 allows everyone full permissions).
  unix_listener auth-userdb {
    #mode = 0666
    #user =
    #group =
  }

  # Postfix smtp-auth
    unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }

  # Auth process is run as this user.
  #user = $default_internal_user
}

```
```
sudo nano /etc/dovecot/conf.d/10-auth.conf

##
## Authentication processes
##
auth_mechanisms = plain login

```  
`sudo service postfix restart`  
-->* Stopping Postfix Mail Transport Agent postfix                         
-->* Starting Postfix Mail Transport Agent postfix 
`sudo serivce dovecot restart`  
-->dovecot stop/waiting
-->dovecot start/running, process 3530

### Testing  
```
telnet mail.mydomain.com smtp

Trying 127.0.0.1...
Connected to mail.asciichan-tripplannr.com.
Escape character is '^]'.

ehlo mail.asciichan-tripplannr.com
250-mail.asciichan-tripplannr.com
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH PLAIN LOGIN
250-AUTH=PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
```
--> quit  
``` 
telnet mail.asciichan-tripplannr.com 587
-->ehlo mail.asciichan-tripplannr.com
250-mail.asciichan-tripplannr.com
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
``` 
### Dovecot main 
`sudo apt-get install dovecot-imapd dovecot-pop3d -y`  
#### Configure Mailbox dovecot  
```
sudo nano /etc/dovecot/conf.d/10-mail.conf

##
## Mailbox locations and namespaces
##

mail_location = maildir:~/Maildir
```  
POP3  

sudo nano /etc/dovecot/conf.d/20-pop3.conf

pop3_uidl_format = %08Xu%08Xv  

```
```
SSL configure  
sudo nano /etc/dovecot/conf.d/10-ssl.conf  
ssl = yes
```  
`sudo service dovecot restart`  

### Test pop3 imap access for receiving mail  
```
telnet mail.mydomain.com 110 or 995 993 143
Trying 127.0.0.1...
Connected to mail.asciichan-tripplannr.com.
Escape character is '^]'.
+OK Dovecot (Ubuntu) ready.
```
#### check what server ports are open
`netstat -nl4`  
### check mail log
`sudo nano /var/log/mail.log`  
