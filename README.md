Python django deployment from GitHub to aws ec2 (Elastic Compute Cloud)

cd Downloads/
mv zillows.pem ~/Desktop/
cd ..
cd desktop

chmod

ssh

yes

sudo apt-get update
sudo apt-get install python-pip python-dev nginx git

Y

sudo apt-get update
sudo pip install virtualenv
git clone https://github.com/mruanova/zillow.git
cd zillow
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
pip install django bcrypt django-extensions
pip install gunicorn
cd zillow
sudo vim settings.py


# Inside settings.py modify these lines allowed host public IP address I for INSERT

i


ALLOWED_HOSTS = ['13.59.206.93']

# add the line below to the bottom of the file

STATIC_ROOT = os.path.join(BASE_DIR, "static/")

Save your changes and quit. ESC :wq

cd .. 
python manage.py collectstatic
gunicorn --bind 0.0.0.0:8000 zillow.wsgi:application

ctrl+c

sudo vim /etc/systemd/system/gunicorn.service

i

[Unit]
Description=gunicorn daemon
After=network.target
[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/zillow
ExecStart=/home/ubuntu/zillow/venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/zillow/zillow.sock zillow.wsgi:application
[Install]
WantedBy=multi-user.target

ESC :wq

sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo vim /etc/nginx/sites-available/zillow

i

server {
  listen 80;
  server_name 13.59.206.93;
  location = /favicon.ico { access_log off; log_not_found off; }
  location /static/ {
      root /home/ubuntu/zillow;
  }
  location / {
      include proxy_params;
      proxy_pass http://unix:/home/ubuntu/zillow/zillow.sock;
  }
}

ESC :wq

sudo ln -s /etc/nginx/sites-available/zillow /etc/nginx/sites-enabled
sudo nginx -t
sudo rm /etc/nginx/sites-enabled/default
sudo service nginx restart

http://13.59.206.93
