# VPS --- MERN
*-*-*-*-*-*-*-*-*-*
# Node.js Deployment

> All of the steps to deploy a Node.js app (Frontend/Backend) to any VPS using PM2, NGINX as a reverse proxy and an SSL

## 1. Create Free AWS Account
Create free AWS Account at https://aws.amazon.com/

## 2. Create and Lauch an EC2 instance and SSH into machine
I would be creating a t2.medium ubuntu machine for this demo.

## 3. Install Node and NPM
```
curl -sL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
```
git clone https://github.com/piyushgargdev-01/short-url-nodejs
```

## 5. Install dependencies and test app
```
sudo npm i pm2 -g
pm2 start index

# But for Next JS, we'll first Install sharp for Image Optimization and then build it
npm i
npm i sharp
npm run build
pm2 start npm --name "IES" -- start
(if we want to run Next JS on any other port)
export PORT=3001

# As for React App, we'll build it noramlly
npm i
npm run build
# No need to pm2, just use the index.html file in build through NGINX route

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 delete (app or all or 0/1)
pm2 save
pm2 save --force
pm2 logs (Show log stream)
pm2 flush (Clear logs)
pm2 env 0

# 0 is the ID of running app in pm2

# To make sure app starts when reboot
pm2 startup ubuntu

# Sometimes pm2 don't tell the running apps after vps reboot but the app is working totally fine (because of pm2 startup and pm2 save)
sudo su
systemctl reload-or-restart pm2-root
```

## 6. Setup Firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 7. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
Add VPS Server IP on Domain and Sub Domain (All of these should be A Tags)
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:8001; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

# Save it using
(Ctrl + C) :wq!
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo nginx -s reload
```

## 8. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
sudo certbot renew --dry-run
```
