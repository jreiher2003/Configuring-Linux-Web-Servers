# Linux Server Configuration LAPP Stack (Linux-Apache-Postgres-Python)
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
8. `sudo service ssh restart`


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


## A - Apache2 HTTP Server 
[Here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) is some great docs using Flask
1.  Install web server
`sudo apt-get install apache2`

`sudo apt-get install libapache2-mod-wsgi python-dev`

enable wsgi to serve app

`sudo a2enmod wsgi`

2.  Create Flask App
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
3. Install Flask  
`sudo apt-get install python-pip`  
`sudo pip install virtualenv`  
`sudo virtualenv venv --always-copy` *if using vagrant*  
`source venv/bin/activate`
`pip install Flask`  
`python __init__.py`  





