# SSL certificates

This is a guide for installing SSL certificates with [Let's Encrypt](https://letsencrypt.readthedocs.org/en/latest/).
We're assuming that you have a working Nginx-server up and running.

## First-time server setup

* Install the Let's Encrypt client

`git clone https://github.com/letsencrypt/letsencrypt /usr/local/letsencrypt`

Note that this needs to be run as root in the future. See [the docs](https://letsencrypt.readthedocs.org/en/latest/intro.html#system-requirements).

## Adding a new ssl cert for a domain

Make sure the domain points to the server you're running this on, or
else you won't be able to auto renew.

### 1. Configure your nginx file for the domain

```
server {
   ...
   listen 443 ssl;
   server_name example.com www.example.com;
   ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
   ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

   # Handle incoming ssl verification requests
   location ^~ /.well-known/ {
      allow all;
   }
}

```

Redirect non ssl traffic to https.

```
server {
listen 80;
server_name example.com;
return 301 https://$host$request_uri;
}
```

### 2. Reload nginx

`sudo service nginx reload`

### 3. Find out your webroot path

Normally this is set in the server path in the nginx file above.
`root /var/www/myapp/current/public;`

### 4. Install the certificate

`/usr/local/letsencrypt/letsencrypt-auto certonly -a webroot --agree-tos --renew-by-default --webroot-path=/var/www/myapp/current/public -d example.com -d www.example.com`

### 4. Add to cron

```
0 0 * * 1 /usr/local/letsencrypt/letsencrypt-auto certonly -a webroot --agree-tos --renew-by-default --webroot-path=/var/www/myapp/current/public -d example.com && sudo service nginx reload
```

On most servers I'd like to collect all certificate updates in one bash script instead of writing long lines like the one above.
