# Publish .NET-10 Web API/App on Ubuntu with Nginx

Step 1: Install .NET 10 Runtime

        > sudo apt-get update && \ sudo apt-get install -y aspnetcore-runtime-10.0
        
        Verify installation:
        > dotnet --list-runtimes
        
Step 2: Install Nginx

        > sudo apt update -y && sudo apt install nginx
        
        Verify installation:
        > sudo systemctl status nginx
        
        Health Checks:
        > sudo service nginx status
        > sudo service nginx stop
        > sudo service nginx restart
        > sudo nginx -s reload
        > sudo nginx -t
        
Step 3: Copy Publish Files

        Create publish folder
        > sudo mkdir -p /var/www/myapp
        
        Transfer files in your ubuntu server directory using WinSCP/FileZilla
        
Step 4: Create systemd Service for .NET App
       > sudo nano /etc/systemd/system/myapp.service
       
       Copy-paste Service Configuration and replace according to your own:
       
       [Unit]
       Description=Your message/title
       After=network.target
       
       [Service]
       WorkingDirectory=/var/www/myapp
       ExecStart=/usr/bin/dotnet /var/www/myapp/myapp.dll
       Restart=always
       RestartSec=10
       KillSignal=SIGINT
       SyslogIdentifier=dotnet-example
       User=www-data
       Environment=ASPNETCORE_ENVIRONMENT=Production
       Environment=DOTNET_NOLOGO=true

       [Install]
       WantedBy=multi-user.target

       Save and exit: 
       Ctrl+S, Ctrl+X
       
       Health Checks:
       > sudo systemctl status myapp.service
       > sudo systemctl enable myapp.service
       > sudo systemctl start myapp.service

Step 5: Configure Nginx Reverse Proxy
        > sudo nano /etc/nginx/sites-available/myapp

        Copy-paste Nginx Configuration and replace according to your own:
        
        map $http_connection $connection_upgrade {
          "Upgrade" $http_connection;
          default keep-alive;
        }

        server {
          listen 80;
          server_name myapp.com;

          location / {
            proxy_pass http://127.0.0.1:5000/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
          }
        }

      server {
        listen 80;
        server_name www.myapp.com;

        location / {
          proxy_pass http://127.0.0.1:5000/;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $connection_upgrade;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
        }
      }
      
Step 6: Enable Site & Restart Nginx

        Create Symlink
        > sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
        
        Test Nginx configuration:
        > sudo nginx -t

        Health Checks:
        > sudo service nginx status
        > sudo service nginx stop
        > sudo service nginx restart
        > sudo nginx -s reload
        > sudo service nginx status
        
Step 7: Install SSL (HTTPS) with Let's Encrypt

        Install Certbot
        > sudo apt install certbot python3-certbot-nginx

        Generate and apply SSL certificates:
        > sudo certbot --nginx -d myapp.com
        > sudo certbot --nginx -d www.myapp.com

Step 8: Configure Firewall for public access (UFW – Uncomplicated Firewall)

        Install UFW:
        > sudo apt update
        > sudo apt install ufw

        Allow SSH Access (CRITICAL) 
        > sudo ufw allow OpenSSH
        
        Allow Web Traffic (HTTP) 
        > sudo ufw allow 80
        
        Allow Web Traffic (HTTPS) 
        > sudo ufw allow 443
        
        Nginx profiles (recommended) 
        > sudo ufw allow 'Nginx Full'

*** After any new site/api deployment/publish files change:
        > sudo systemctl restart myapp.service
*** After any change in nginx:
        > sudo nginx -s reload

> DONE ! Now check your site using mobile data or from a different network from your premises
