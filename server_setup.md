# The Ultimate Guide for the Workflow 💯
# Development 🛠️ > Home Lab 🔐 > Production Server 🛡️

*This file is about automating the workflow of a Full-Stack Developer. In some cases, it will also help the Dev-Ops engineers. You don't need to follow every step of it (Unless you are new to deployment). You are also welcome to contribute to this setup guide so that it become one of the finest setup guide for django-developers and Dev-Ops Engineers.*

---

## 🛠️ Development Machine Setup

*Assuming that you are using laptop or desktop*

### 📍 Confirming system is up-to-date and has all the dependencies

- **⚙️ Prerequisites**

Linux (Debian Based)

```bash
sudo apt update && upgrade -y
sudo apt install python3 python3-venv python3-pip python3-dev libpq-dev curl git gunicorn
python3 --version
# Your should see python version 3.X.X
pip3 --verison
# You should see pip version X.X.X
```

- **⚙️ Setting up Git**

*Those who wants version controling via [Github Website](https://github.com/), follow below steps:*

```bash
git config --global user.name "Your Github Username"
git config --global user.email "Your Email used in Github"
# Check the settings
git config --global --list
# You should see username and email listed there

# Generate a ssh key to authenticate without password
# Using ed25519
ssh-keygen -t ed25519 -C "Your Email Address"
# Or you can use RSA
ssh-keygen -t rsa -b 4096 -C "Your Email Address"
# When asked for file location, press enter. Default is fine
# When asked for a passphrase, you can set one or leave it blank, press enter.

# Add ssh key to ssh agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
# Or if you used RSA
ssh-add ~/.ssh/id_rsa

# Add ssh-key to github
cat ~/.ssh/id_ed25519.pub
# Or if you used RSA
cat ~/.ssh/id_rsa.pub
# Copy the key
```

📎 Go to `Github > Settings > SSH and GPG keys > New SSH key`
Paste the copied key there.

- **⚙️ Test the connection**

```bash
ssh -T git@github.com
# You should see success response.
```

### 📍 Setting up django project settings for production-ready development workspace

- **⚙️ Initialize**

*Now that we have installed all the dependencies we need, we can start our first app. Follow below steps:*

```bash
mkdir -p project
cd project
python3 -m venv venv
source venv/bin/activate
pip install django djangorestframework python-decouple gunicorn psycopg2-binary
django-admin startproject project .
```
This will install least python modules required to work with django.

*N.B: You can use `pyhton-dotenv` instead of `python-decouple` for handling `.env` files. Skip `psycopg2-binary` module if you don't use PostgreSQL database.*

**Create a project**
```bash
django-admin startproject project1 .
python3 manage.py startapp core
touch .env README.md LICENSE .gitignore requirements.txt core/urls.py
mkdir -p static static/images static/icons static/css static/js static/docs templates templates/core
```

- **⚙️ Update your `project/settings.py` file:**

```python
from decouple import config

# ... existing codes ...

SECRET_KEY = config('SECRET_KEY')
DEBUG = True if config('DEBUG') == 'True' else False
ALLOWED_HOSTS = config('ALLOWED_HOSTS').split(',')

INSTALLED_APPS += ['rest_framework','corsheaders','core',] # add other apps as needed
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'config.wsgi.application'
ASGI_APPLICATION = 'config.asgi.application'

if DEBUG:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST'),
            'PORT': '6432',  # PgBouncer port
        }
    }

SITE_ID = 1

AUTHENTICATION_BACKENDS = [
    "django.contrib.auth.backends.ModelBackend",
]

from django.contrib.messages import constants as messages
MESSAGE_TAGS = {
    messages.DEBUG: 'debug',
    messages.INFO: 'info',
    messages.SUCCESS: 'success',
    messages.WARNING: 'warning',
    messages.ERROR: 'error',
}

STATIC_URL = 'static/'
if DEBUG:
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static')
    ]
else:
    STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

MEDIA_URL = '/media/'

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

- **⚙️ Update `project/urls.py` file:**

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('core.urls')),
]
```

- **⚙️ Update `.env` file:**

```bash
nano .env
```

Edit variables:

```bash
DEBUG="False"
SECRET_KEY="your-secret-key"
ALLOWED_HOSTS="127.0.0.1,localhost,192.168.0.X"
DB_HOST="localhost"
DB_NAME="db_name"
DB_USER="db_user"
DB_PASSWORD="db_password"
```

- **⚙️ Update `core/urls.py` file:**

```python
# ... existing codes ...
from django.urls import re_path
from django.conf import settings
from django.views.static import serve

urlpatterns += [
    re_path(r'^media/(?P<path>.*)$', serve , {'document_root': settings.MEDIA_ROOT}),
    re_path(r'^static/(?P<path>.*)$', serve , {'document_root': settings.STATIC_ROOT}),
]
```

Assuming that you have created models in `core/models.py` file, adjusted view functions in `core/views.py` file and routed urls in `core/urls.py` Now we will move to next step.

- **⚙️ Running migrations and testing in localserver**

```bash
python3 manage.py collectstatic
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py runserver
# This will start the server at http://127.0.0.1:8000 by default
# Use 'python3 manage.py runserver 0.0.0.0:8000' to access the web application from local network. It will then accessible via 'http://192.168.X.X:8000' from any device in the network.
```

### 📍 Preparing development for homelab/production

To this far we have created a satisfactory web application, with beautiful features. We will use git for version controlling so that we don't lose any of our developments.

- **⚙️ Initialize git repository:**

```bash
cd project
git init
# This will create a .git folder in the project folder
```

We will add unwanted files to `.gitignore` file, so that git doesn't track them.
Basic files and folders to be put in `.gitignore` file:

- **⚙️ Update `.gitignore` file:**

```bash
# ---- Python ----
__pycache__/
*.py[cod]
*$py.class
**/migrations/__pycache__/

# ---- Virtual environment ----
venv/
env/
.venv/

# ---- IDE / Editor files ----
.vscode/
.idea/
*.sublime-workspace
*.sublime-project

# ---- OS files ----
.DS_Store
Thumbs.db

# ---- Environment & Secrets ----
.env
*.env
*.key
*.pem
*.cert
secrets.json

# ---- Django ----
*.log
media/
staticfiles/
db.sqlite3
local_settings.py
*.sock
*.service

# ---- Build / Compiled files ----
dist/
build/

# ---- Backups ----
*.bak
*.swp
*.tmp

# ---- Coverage / Tests ----
.coverage
htmlcov/
pytest_cache/

# ---- Cache ----
__pycache__/
*.cache/
.cache/
```

You can add/delete necessary/unnecessary tracking.

- **⚙️ Add and Commit Changes:**

```bash
pip freeze > requirements.txt
git add .
git commit -m "Commit mesage goes here"
git branch -M main
```

- **⚙️ Set up remote git to push**

Here is a trick. 
> We can push codes to `https://github.com/`. Then use `git clone` and `git pull` to get updates from `Github`

There is another trick

> We can directly push updates to homelab server or production server.

We will see both:

- **Use Github website:**
    - Log in to github.com
    - Create an empty git repository

Then in your terminal:

```bash
cd project
git add remote origin "https://github.com/username/repo.git"
```

- **Use remote git:**
    - Log in to your homelab/production server
    - Create a bare git repository

```bash
sudo mkdir -p /opt/git
cd /opt/git
sudo git init --bare repo.git
sudo chown -R user:user /opt/git
```

This will create a bare git repository in the server as we did in the github.
Then in our development machine terminal:

```bash
cd project

# For lab server
git remote add lab ssh://user@your-lab-server-ip/opt/git/repo.git

# For production server
git remote add production ssh://user@your-production-server-ip/opt/git/repo.git
```

Finally, we can push our codes to remote repository.

```bash
# Github website
git push -u origin main

# Git repo - lab
git push lab main

# Git repo production
git push production main
```

This is how we can `push/pull` between local and github/lab/production

> Personal note:
>> I don't use github website for tracking. Instead, I use lab and production remote both.

---

## 🗄️ Homelab or Production Server Set-Up

**Prerequisites**

- You have a Virtual Private Server or Dedicated Server or Raspbery Pi or Any Other Single Board Computer running.
- You have access to server
- Server has a static public IP address.
- You are logged in as user, not root.

### 📍 Set Up Git & Auto-deploy On Push From Local

Install below dependencies:

```bash
sudo apt update && upgrade -y
sudo apt install python3 python3-venv python3-pip python3-dev libpq-dev curl git gunicorn postgresql postgresql-contrib supervisor pgbouncer
python3 --version
# Your should see python version 3.X.X
pip3 --verison
# You should see pip version X.X.X
git --version
# You should see git version X.X.X
psql --version
# You should see postgresql version X.X
```

#### ⚙️ Fetch codes from bare git

Recall the bare git repository creationin the server? We will now use it.

```bash
cd ~
git clone /opt/git/repo.git repo
# This will create a folder repo and collect all the files same as development hierarchy

cd repo
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
touch .env
nano .env
# Copy & Paste all key-value environment variables

python3 manage.py collectstatic
```

#### ⚙️ Automatically updates codes from bare git whenever local pushes

On server:

```bash
cd /opt/git/repo.git/hooks
nano automatic-receive
```

Use the below script:

```bash
#!/bin/bash
DEPOLY_DIR=/home/user/repo
GIT_DIR=/opt/git/repo.git

echo "Deploying latest code to $DEPLOY_DIR"

if [ ! -d "$DEPLOY_DIR" ]; then
    git clone $GIT_DIR $DEPLOY_DIR
else
    cd $DEPLOY_DIR
    git pull origin main
fi

source venv/bin/activate
pip install -r requirements.txt
python3 manage.py collectstatic --noinput
python3 manage.py makemigrations --noinput
python3 manage.py migrate --noinput
systemctl restart repo # We will see this later
```

Make this script executable:

```bash
chmod +x automatic-receive
```

This hook will automatically fetch-update-migrate-restart new changes on every push.

### 📍 Set Up Database and Configure PG-Bouncer

We will now set up a postgresql database and pgbouncer to handle multiple databases. Some pgbouncer configuration might need to chnage if you are handling large-scale database. This configuration is good for managing multiple small databases.

Let's confirm our postgresql and pgbouncer installation:

```bash
psql --version # postgresql version checking
pgbouncer --version # pgbouncer version checking
```

#### ⚙️ Update postgresql configuration

Use `sudo nano /etc/postgresql/17/main/postgresql.conf` to edit:  

```bash
max_connections = 1000
shared_buffers = 2GB
work_mem = 16MB
maintenance_work_mem = 512MB
effective_cache_size = 4GB
wal_level = replica
synchronous_commit = off
checkpoint_timeout = 10min
checkpoint_completion_target = 0.9
```

Restart the postgresql:

```bash
sudo systemctl restart postgresql
```

#### ⚙️ Create a database

Use `sudo -u postgres psql` to start postgres CLI:

```bash
CREATE DATABASE db_name;
CREATE USER db_user WITH PASSWORD 'db_password';
GRANT ALL PRIVILEGES ON DATABASE db_name TO db_user;
ALTER DATABASE db_name OWNER TO db_user;
\q
```

Do this to other databases also.

#### ⚙️ Update pgbouncer configuration

Use `sudo nano /etc/pgbouncer/pgbouncer.ini` to edit:

```bash
# ... existing codes ...

[databases]
project_db = host=127.0.0.1 port=5432 user=db_user password=db_password
# another_project_db = host=127.0.0.1 port=5432 user=another_db_user password=another_db_password

# ... existing codes ...
[pgbouncer]
listen_port = 6432
listen_addr = 127.0.0.1
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 2000
default_pool_size = 100
```

I use this configuration for my 8 core - 8GB RAM server. Adjust numbers based on your server capacity.  

Use `sudo nano /etc/pgbouncer/userlist.txt` :

```bash
"db_user" "db_password"
# "another_db_user" "another_db_password"
```

Then, restart pgbouncer to load changes.

```bash
sudo systemctl restart pgbouncer
```

### 📍 Connect Gunicorn to django web app

**Prerequisites:** We have already installed gunicorn, locally and in venv too. Let's test that gunicorn is running well or not:

```bash
cd project
source venv/bin/activate
gunicorn --bind 0.0.0.0:8000 project.wsgi
```

You should see gunicorn activated the server. Go to browser and check:

```bash
http://server-ip:8000
```

If you face troble starting gunicorn, this might be the problem of the `ufw` To fix:

```bash
sudo ufw allow 8000
```

#### ⚙️ Create a service file to automate this process

First, we will update permission classes for project directory:

```bash
sudo chown -R uch:www-data /home/user
sudo chown -R uch:www-data /home/user/project
sudo chmod 755 /home/user/project
```

Then, use `sudo nano /etc/systemd/system/project.service` to create a service file:

```bash
[Unit]
Description=Gunicorn daemon for project
After=network.target

[Service]
User=uch
Group=www-data
WorkingDirectory=/home/user/project
RuntimeDirectory=project
ExecStart=/home/user/project/venv/bin/gunicorn --access-logfile /home/user/project/logs/gunicorn-access.log --error-logfile /home/user/project/logs/gunicorn-error.log --workers 4 --threads 2 --timeout 60 --bind unix:/home/username/project/project.sock project.wsgi:application
Restart=always
Environment="DJANGO_SETTINGS_MODULE=project.settings"
Environment="PYTHONUNBUFFERED=1"

[Install]
WantedBy=multi-user.target
```

Service file created. Now start this:

```bash
sudo systemctl start project
# or you can use 'sudo systemctl start project.service'

sudo systemctl enable project

sudo systemctl status project
```

You should see active-running in the terminal.

If you see any error, most probable cause is that socket file is not created for `Permission Denied` Check if file exists or not:

```bash
file /home/user/project/project.sock
```
If file is not there, check for permission classes of folders again:

```bash
ls -la /home/user
# It will show owner-group information of every folder. Look for project folder permissions.

ls -la /home/user/project
# It will show owner-group information of every folder. Look for project folder permissions.
```

Or, you can use `journalctl` to see error codes, it will briefly describe the cause of error.

```bash
sudo journalctl -u project.service
# It displays all logs related to this service

sudo journalctl -u project.service -n 100
# It displays the last 100 log lines related to this service

sudo journalctl -u project.service -f
# It displays real-time logs of this service
```

### 📍 Update NGINX configuration file to connect gunicorn to DNS

We have `nginx` installed in previous commands.

First, we update some configuration `sudo nano /etc/nginx/nginx.conf` :

```bash
worker_processes 4;
worker_rlimit_nofile 100000;

events {
    worker_connections 10000;
    multi_accept on;
}

http {
    keepalive_timeout 30;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    client_max_body_size 50M; # adjust this to set limit for max file upload size from clients
    proxy_read_timeout 120s;
    proxy_connect_timeout 120s;
    proxy_send_timeout 120s;
}
```

Restart nginx to apply changes : `sudo systemctl restart nginx`

Now, use `sudo nano /etc/nginx/sites-available/project` to create:

```bash
upstream project{
    server unix:/home/user/project/project.sock;
}
server{
    server_name example.com www.example.com;
    access_log /var/log/nginx/project.access.log;
    error_log /var/log/nginx/project.error.log;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        alias /home/user/project/static/;
    }
    
    location /media/ {
        alias /home/user/project/media/;
        }

    location / {
        proxy_pass http://project;
        include proxy_params;
    }
}
```

Save this and enable this configuration:

```bash
sudo ln -s /etc/nginx/sites-available/project /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl reload nginx
```

Configure your DNS A record to server IP from your domain doashboard (typically provided by the service provider) visit http://example.com from your browser.

### 📍 Using certbot to have free ssl certificate

SSL Certificate is needed to establish https connection. Lets install some prerequisites:

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
sudo systemctl reload nginx
```

You can now visit example.com from your browser and see there is a https connection.

Test `dry-run` to check auto-renewal working or not:

```bash
sudo certbot renew --dry-run
```

That is all. I hope this helps you a lot.

---

# 🚩 Conclusion

I have shared everything I learned through my entire career. I shall contribute to this setup configuration if anything feels necessary or informative. You should also contribute to this repo. Here is how you can do this:

#### ⚙️ Fork the repository

Fork this repository to your github account by clicking the "Fork" button on the top right.  

#### ⚙️ Clone your fork locally

```bash
git clone https://github.com/your-username/repository-name.git
```

#### ⚙️ Create a new branch

```bash
git checkout -b new-feature-branch
```

#### ⚙️ Make your changes

Work on your changes in this branch. You can modify this script, add new features or fix scripts.

#### ⚙️ Commit your changes

```bash
git commit -m "Added this feature or fixed bug"
```

#### ⚙️ Push changes to your fork

```bash
git push origin new-feature-branch
```

#### ⚙️ Create a PR (Pull Request)

Open a pull request to the `main` branch off the original repository. Provide a clear description of the changes you have made.

---

*2025, Tanvir Sakaln @uch105*