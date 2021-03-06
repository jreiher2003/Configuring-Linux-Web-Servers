# LAPP Stack (Linux-Apache-Postgres-Python)
# Linux Server Configuration  
## L - Linux Ubuntu trusty/64

### Update and Upgrade linux os 
It is always recommended to update and upgrade any linux box first and foremost.  These two commands with get you there.
* `sudo apt-get update` 
* `sudo apt-get autoremove`   
* `sudo apt-get upgrade`

### Add user to server and remove password-based logins
Every linux os sys has the notion of a root user or a user named root.  We are going to add a user and give that user superuser privileges and remove the ability to login via a password.  This will increase security of our server.  
* `sudo apt-get install finger`
* `sudo adduser student`

#### Change sudo permissions to added user
*update*  if your getting an error **unable to resolve host**  do two things  
1. read hostname file `cat /etc/hostname`  copy that in mycase it was ip-10-20-20-7
2. `sudo nano /etc/hosts/` >>> paste in top of file >> `127.0.0.1 localhost localhost.localdomain ip-10-20-20-7`
`sudo cat /etc/sudoers`  *read sudoers file*    
`sudo ls /etc/sudoers.d`  *lst sudoer users*    
1. login as root   
2. copy file and rename it student `sudo cp /etc/sudoers.d/root /etc/sudoers.d/student`  *could be called vagrant*  
3. edit file `sudo nano /etc/sudoers.d/student` change root to student  
4. On a production server you may only need to create user file as `sudo nano /etc/sudoers.d/<user>`  
5. Add #``` User privilege specification
        <user>    ALL=(ALL:ALL) NOPASSWD: ALL```   
6. check sudo access on new user  `whoami`  >>> root >> `su <user>` >> `whoami` >> <user>

#### Generate key pairs and disable password logins
* install [ssh-keygen](http://stackoverflow.com/questions/11771378/ssh-keygen-is-not-recognized-as-an-internal-or-external-command).  
Always private keys are stored locally and public keys are stored on remote server.

`ssh-keygen` (name-your-key) locally


1. login as user on remote server
2. `mkdir .ssh`
3. `touch .ssh/authorized_keys`
4. copy **your-key-name.pub** and paste in .ssh/authorized_keys
5. set file permissions `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
6. login in to user account with key ` ssh student@ip-address -p 2222 -i ~/.ssh/your-key-name`  (ip address could also be localhost 127.0.0.1, port could be 2200)
7. **Disable password logins** `sudo nano /etc/ssh/sshd_config`  *change PasswordAuthentication to* **no** 
8. *optional* while in the file change PermitRootLogin without-password to **PermitRootLogin no** to disallow root login.  
9. `sudo service ssh restart`  

### File Permissions  
```
ls -al
- = file
d = directory 
```  

```
-rw-r--r-- 1 user user date .bashrc
owner  group   everyone  owner  group
rw-    r--     r--       user   user
```    

```
octal permissions read write execute 0 if none
r = 4  w = 2  x = 1
```  

```
user/group/everyone
.bashrc octal form
644
```  
```
chmod >> change mode 
chown >> change owner
chgrp >> change group

ex. >> sudo chown <user-to-change-to> <file>
```



### Configure firewall
A server firewall is just an application that tells the server which ports to listen to on.  ubuntu pre-installed firewall is `ufw`

Protocol | Default Port
--- | ---
http | 80
https | 443
ssh | 22
ftp | 21
pop3 | 110 
smtp | 25 

`sudo ufw status`

#### incoming
we want to block all incoming traffic by default *The rule of least privilege*

`sudo ufw default deny incoming`

`sudo ufw allow ssh`

`sudo ufw allow 2222/tcp`

`sudo ufw allow www`


#### outgoing
traffic going from our server is not a big security risk.

`sudo ufw default allow outgoing`

#### enable firewall
*Never enable firewall unless you are sure your server is configured properly*

`sudo ufw enable`

`sudo ufw status` 

To | Action | From
--- | --- | ---
22 | ALLOW | Anywhere
2222/tcp | ALLOW | Anywhere
80/tcp | ALLOW | Anywhere
22 (v6) | ALLOW | Anywhere
2222/tcp (v6) | ALLOW | Anywhere
80/tcp (v6) | ALLOW | Anywhere  

#### Configure the local timezone to UTC
`sudo dpkg-reconfigure tzdata` 
 select none of the above. Then select UTC

## A - Apache2 HTTP Server 
[Here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) is some great docs using Flask  
[Here](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04)
####  Install web server  

`sudo apt-get install apache2`  
`sudo apt-get install libapache2-mod-wsgi python-dev`  
enable wsgi to serve app  
`sudo a2enmod wsgi`  

check that its working:  
http://your_server_IP_address

see error logs  
`sudo cat /var/log/apache2/error.log`  
  disable default placeholder site  
`sudo a2dissite 000-default` 
`sudo a2ensite 000-default`   

####  Create Flask App  
-`cd /var/www`  
-`sudo mkdir FlaskApp`  
-`cd FlaskApp`  
-`sudo mkdir FlaskApp`  
-`cd FlaskApp`  
`sudo mkdir static templates`  


```
|----FlaskApp
|---------FlaskApp
|--------------static
|--------------templates
```

`sudo nano __init__.py`

```python 
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()
```
#### Install Flask  
`sudo apt-get install python-pip`  
`sudo pip install virtualenv`  
`sudo virtualenv venv --always-copy` *if using vagrant*  
`sudo chown -R jeff:jeff venv/`  change file permissions to venv to not have to install globally  
`source venv/bin/activate`
`pip install Flask`  
`python __init__.py`  

>> *Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)*

#### Configure and Enable a New Virtual Host 
Newer versions of Ubuntu (13.10+) require a ".conf" extension for VirtualHost files -- run the following command 
`sudo nano /etc/apache2/sites-available/FlaskApp.conf`  

Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud server's IP address:  
```
<VirtualHost *:80>
        ServerName 52.34.51.82
        ServerAdmin admin@52.34.51.82
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```  
Enable the virtual host with the following command:  
`sudo a2ensite FlaskApp`  

#### Create the .wsgi File
Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/FlaskApp directory and create a file named flaskapp.wsgi with following commands:  
`cd /var/www/FlaskApp`  
`sudo nano flaskapp.wsgi`  

```python
#!/usr/bin/python
import sys
import logging
activate_this = '/var/www/FlaskApp/FlaskApp/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```  
Now your directory structure should look like this:  
```
|----FlaskApp
|---------FlaskApp
|--------------static/
|--------------templates/
|--------------venv/
|--------------__init__.py
|---------flaskapp.wsgi
```  
#### Restart Apache  
`sudo service apache2 restart` 

#### Install Git   
`sudo apt-get install git`  
`git config --global user.name "Jeff Reiher"`  
`git config --global user.email "jreiher2003@yahoo.com"`  
`git config --global push.default upstream`  
`git config --global merge.conflictstyle diff3`  
`git config --global credential.helper 'cache --timeout=10000'`   
`wget https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash`  
`wget https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh`  
`wget https://www.udacity.com/api/nodes/3333158951/supplemental_media/bash-profile-course/download`  

`cat download >> .bashrc`  
`rm download` 

**configure git awesomely!**[here](https://www.udacity.com/course/viewer#!/c-ud775/l-2980038599/m-3333158951)


#### Clone Github Repo  
`sudo git clone https://github.com/jreiher2003/menu.git`  
make sure to get hidden files in move `shopt -s dotglob`  
Move files from clone dir to FlaskApp  
`mv /var/www/FlaskApp/menu/* /var/www/FlaskApp/FlaskApp/`  
remove clone dir `sudo rm -rf menu`  
#### Make .git inaccessible  
`cd /var/www/FlaskApp/` create .htaccess file  `sudo nano .htaccess`  
paste in `RedirectMatch 404/\.git`  

## P - PostgresSQL  
installing [PostgresSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)  
`sudo apt-get install postgresql postgresql-contrib`  

login as default user  
`sudo -i -u postgres`  

shell prompt  
`psql`  

quit  
`\q`  

#### create new role  
`createuser --interactive`  
`createuser -P <user>`  w/password enabled  
 >> name user (same as linux user) superuser yes 

`exit`   

#### login as postgres user  
`sudo -i -u <user>`  

#### create db
`createdb test1`  

#### connect to db  
`psql -d test1`  
`\conninfo`  

You are connected to database "test1" as user "jeff" via socket in "/var/run/postgresql" at port "5432".
















