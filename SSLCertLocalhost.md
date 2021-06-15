# Manage SSL Certificates for localhost

Based on https://askubuntu.com/questions/73287/how-do-i-install-a-root-certificate

And https://gist.github.com/cecilemuller/9492b848eb8fe46d462abeb26656c4f8

To access development with HTTPS you need to setup a Certification Authority (think of it like our local Let's Encrypt) and generate certificates for your applications based on this authority.

First, create a folder to organize and then generate the authority RootCA.pem, RootCA.key & RootCA.crt:
```
mkdir ~/ssl
cd ~/ssl
openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout RootCA.key -out RootCA.pem -subj "/C=BR/CN=Local-Root-CA"
openssl x509 -outform pem -in RootCA.pem -out RootCA.crt
```

Now, for every domain you want to serve with HTTPS, add to the end of the new file `domains.ext`, using `nano domanins.ext`.
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
DNS.2 = mysite.local
DNS.3 = fake2.local
```

Generate localhost.key, localhost.csr, and localhost.crt:

```
openssl req -new -nodes -newkey rsa:2048 -keyout localhost.key -out localhost.csr -subj "/C=BR/ST=SC/L=Floripa/O=Local-Certificates/CN=localhost.local"
openssl x509 -req -sha256 -days 1024 -in localhost.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -extfile domains.ext -out localhost.crt
```

On Nginx (based on Laravel's default configuration), configure it so it will redirect unsecure traffic to HTTPS and setup HTTPS engine properly. On every `site-available` configure the following, using the `default` site as an example.

```
sudo nano /etc/nginx/sites-available/default
```
```
server {
    listen 80;
    server_name mysite.local;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;

    ssl_certificate     /home/myuser/ssl/localhost.crt;
    ssl_certificate_key /home/myuser/ssl/localhost.key;
    ssl_protocols       SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    server_name mysite.local;
    root /var/www/mysite/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    client_max_body_size 24M;
    client_header_buffer_size 256k;
    large_client_header_buffers 4 256k;

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
        proxy_buffering off;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #fastcgi_pass   127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME     $fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Done? Your local development is now HTTPS enabled, but your browser and system don't trust the certificates just generated. So we will add the RootCA to the Trusted Root Certification Authorities.

## Add RootCA to Windows
On Windows, open the start menu and search for `certificate` and select `Manage Computer Certificates`. Select Trusted Root Certification Authorities and import `RootCA.ctr`. Close all your browsers and access. It should be all set.

## Add RootCA to Ubuntu
On Ubuntu, copy the `RootCA.crt` to `/usr/localshare/ca-certificates`
```
sudo cp ~/ssl/RootCA.crt /usr/local/share/ca-certificates/
```
And run

```
sudo update-ca-certificates
```
Now our `RootCA` is trusted by the system and will accept certificates emitted by it.
