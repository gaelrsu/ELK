##   nginx/sites-available/your_domain :

```bash
          server {
              listen 80;

              server_name your_domain;                    ///////>>>>>>>>>>>>> IP

              auth_basic "Restricted Access";
              auth_basic_user_file /etc/nginx/htpasswd.users;

              location / {
                  proxy_pass http://localhost:5601;
                  proxy_http_version 1.1;
                  proxy_set_header Upgrade $http_upgrade;
                  proxy_set_header Connection 'upgrade';
                  proxy_set_header Host $host;
                  proxy_cache_bypass $http_upgrade;
              }
          }
```


Create link : 
```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/your_domain
```

### ATTENTION SI ACTIVATION FIREWALL POSSIBLE BLOCAGE ENTRE SERVEUR
firewall :
```bash
sudo ufw allow 'Nginx Full'
```

