# LAPP Stack (Linux-Apache-Postgres-Python)
# Linux Server Configuration  
## L - Linux

### Update and Upgrade linux os 
It is always recommended to update and upgrade any linux box first and foremost.  These two commands with get you there.
* `sudo apt-get update`  
* `sudo apt-get upgrade`

### Add user to server and remove password-based logins
Every linux os sys has the notion of a root user or a user named root.  We are going to add a user and give that user superuser privileges and remove the ability to login via a password.  This will increase security of our server.  
* `sudo apt-get install finger`
* `sudo adduser student`

#### Change sudo permissions to added user
1. login as root
2. `cd /etc/sudoers.d`
3. copy file and rename it student `sudo cp /etc/sudoers.d/root /etc/sudoers.d/student`
4. edit file `sudo nano /etc/sudoers.d/student` change root to student

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
8. *optional* while in the file change PermitRootLogin without-password to PermitRootLogin no to disallow root login.  
9. `sudo service ssh restart`


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

#### Install Git google this  
`sudo apt-get install git`  
`git config --global user.name "YOURNAME"`  
`git config --global user.email "YOU@DOMAIN.com"`   


## A - Apache2 HTTP Server 
[Here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) is some great docs using Flask  

####  Install web server  

`sudo apt-get install apache2`  
`sudo apt-get install libapache2-mod-wsgi python-dev`  
enable wsgi to serve app  
`sudo a2enmod wsgi`  
disable default placeholder site  
`sudo a2dissite 000-default`  

####  Create Flask App  
-`cd /var/www`  
-`sudo mkdir FlaskApp`  
-`cd FlaskApp`  
-sudo mkdir FlaskApp`  
-`cd FlaskApp`  
`sudo mkdir static templates `  


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
`source venv/bin/activate`
`pip install Flask`  
`python __init__.py`  

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

#### Clone Github Repo  
`sudo git clone https://github.com/jreiher2003/menu.git`  
make sure to get hidden files in move `shopt -s dotglob`  
Move files from clone dir to FlaskApp  
`mv /var/www/FlaskApp/menu/* /var/www/FlaskApp/FlaskApp/  
remove clone dir `sudo rm -r menu`  
#### Make .git inaccessible  
`cd /var/www/FlaskApp/` create .htaccess file  `sudo nano .htaccess`  
paste in `RedirectMatch 404/\.git`  

## P - PostgresSQL  










