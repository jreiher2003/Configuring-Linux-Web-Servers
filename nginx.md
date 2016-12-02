# Nginx server set up (work in progress)

### Add a new User 
```adduser youruser sudo```  

## Install the Requirements 
```sudo apt-get update```  
```sudo apt-get install -y python python-pip python-virtualenv nginx gunicorn```  

### Configure nginx  
```sudo /etc/init.d/nginx start```  
```
sudo rm /etc/nginx/sites-enabled/default
sudo touch /etc/nginx/sites-available/flask_project
sudo ln -s /etc/nginx/sites-available/flask_project /etc/nginx/sites-enabled/flask_project
```
#### --> /etc/nginx/sites-available/flask_configs 
```
##nginx virtual host setting


#client_max_body_size 500M; # allows file uploads up to 500 megabytes;

server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /home/finn/www/peer_flask;

        # Make site accessible from http://localhost/
        #server_name localhost;

        location /static {
                alias /home/finn/www/peer_flask/app/static;
                expires max;
        }
        location / {
                proxy_pass http://127.0.0.1:9000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

}cd
```
### Restart nginx 
``` sudo /etc/init.d/nginx restart```  
```
cd /home/www/flask_project/
gunicorn app:app -b localhost:8000
```