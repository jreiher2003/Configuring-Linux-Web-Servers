# Linux Server Configuration 

### Update and Upgrade linux os 
It is always recommended to update and upgrade any linux box first and formost.  These two commands with get you there.
* `sudo apt-get update`  
* `sudo apt-get upgrade`

### Add users to server and remove root user
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
7. **Disable password logins** `sudo nano /etc/ssh/sshd_config`  *change PasswordAuthentication to **no***
8. `sudo service ssh restart`

