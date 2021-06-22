# Install Let's Encrypt

Steps to install and generate Let's Encrypt certificates.

First add the certbot's repository and install.

```
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install python-certbot-nginx
```

Then install the certificate to your domain. Input all the information required.

```
sudo certbot --nginx -d mysite.example.com.br
sudo certbot renew --dry-run
```

Add the certificates to your's site nginx config file `/etc/nginx/sites-available/mysite`
```
server {
    listen 443 ssl;
    
    # SSL via LetsEncrypt
    ssl_certificate /etc/letsencrypt/live/mysite.example.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mysite.example.com.br/privkey.pem;

    # Strengthening security
    ssl_session_cache shared:SSL:20m;
    ssl_session_timeout 60m;
    ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    
    #your other configurations
    #...
    #...
    #...
}
```

Restart Nginx.
```
sudo service restart nginx
```

That's it!
