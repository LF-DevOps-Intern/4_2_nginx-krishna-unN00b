# Solution

1. Install nginx and host a simple index.html with message “hello nginx”

```console
sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

##### index.html
```html
<h1>hello nginx</h1>
```

![nginx up and running](https://user-images.githubusercontent.com/23631617/142185433-971dd4a5-fe65-4b9d-9d0d-93c93d4a4984.png)

---

2. What are nginx header security and its uses. And also implement in the test.conf file.

Since nginx can work as a proxy, it can add additional headers to clients before
forwarding the response to them. The same functionality is used to append
security headers to the response. These headers can instruct the browsers to
perform certain actions such as deny framing of the resource (against
clickjacking), use TLS only (HSTS, against MITM), reject cross-origin requests
(against CSRF), Content Security Policy (against loading unallowed remote content and XSS), as well as to turn the browser XSS
filters on or off, according to the headers sent. Additionally, some new headers can tell the
browser to automatically reject camera, microphone, geolocation and other sensitive
permissions. I've also explained in brief about the use of each security headers
in the following config.

##### /etc/nginx/sites-available/test.conf
```console
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location /test {
		try_files $uri $uri/ =404;
	}

	# Enable HSTS
	# Prevents users from accessing HTTP version of the website
	add_header Strict-Transport-Security "max-age=63072000; includeSubdomains;" always;

	# Disable iframe creation
	# Prevents the framing of website from another website to prevent against clickjacking
    add_header X-Frame-Options "deny" always;

	# Enable browser's XSS protection
	# Prevents some well-known XSS payloads using browser's anti-XSS engine
    add_header X-XSS-Protection "1; mode=block" always;

	# Turn off content sniffing
	# Prevents attacks based on MIME sniffing
    add_header X-Content-Type-Options "nosniff" always;

	# Only allow resources to run from listed domains
	# Prevents from XSS and leaking of information via remote-loading of malicious contents
    add_header Content-Security-Policy "default-src 'self'" always;

	# Disable referer header when requesting a cross-origin resource
	# Prevents referral-based leaks to third parties
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

We need to enable the config, test, then restart nginx.

```console
sudo ln -s /etc/nginx/sites-available/test.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

![Append Security Headers](https://user-images.githubusercontent.com/23631617/142187615-3ef79a28-268b-478a-be50-f353fafb1caa.png)

---

3. Nginx Reverse proxy all http requests to nodes js api.

```
    location / {
            proxy_set_header X-Forwarded-For $remote_addr
            proxy_set_header Host $http_host;
            proxy_pass http://127.0.0.1:6080;
    }   
```

![NodeJS and nginx](https://user-images.githubusercontent.com/23631617/142189862-aa8a60ac-e961-4315-b951-de09c2626763.png)

![nginx as a proxy to nodeJS](https://user-images.githubusercontent.com/23631617/142189369-5d02402b-2073-4878-96fb-425618590deb.png)

---

4. Create a test2.conf and listen on port 82 and  to “ location /test/” with message “ test is successful”.

We create a `/test/index.html` file inside of our root directory then inside `test2.conf`, match the path `/test/`. And we listen to port 82.

##### test2.conf
```console
server {
        listen 82 default_server;
        listen [::]:82 default_server;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

location /test/ {
                try_files $uri/ = 404;
        }
```

![tes2 conf](https://user-images.githubusercontent.com/23631617/142190566-d41b07ea-f030-44ed-95ed-5f9061660f84.png)

---

5. Reverse proxy all http traffic of port 82 to port 85.

As in question number 3, we can create a new config file which listens on port 85, then use `proxy_pass 127.0.0.1:82`, create its symlink in sites-enabled directory, then restart nginx service.

![Reverse proxy port 82 to 85](https://user-images.githubusercontent.com/23631617/142191545-23887774-f753-4c2b-8b69-4477595a2abd.png)

---

6. Install LEMP stack (avoid installing mysql) and open info.php on port 80 and print message info.php.

```console
# Linux: already installed as VM
# Nginx: already installed
# MySQL: skipping

# PHP
sudo apt install php php-fpm
```

We need to edit `/etc/php/7.4/fpm/php.ini` and add this line in order to prevent CGI from exectuting PHP scripts
that don't exactly match the file name in the HTTP request. 
```
cgi.fix_pathinfo=0
```

Then we need to restart fpm.
```
sudo systemctl restart php7.4-fpm.service
```

We then create info.php.
```php
<?php
    phpinfo();
?>
```

Finally, we make a new Nginx config to listen on port 80 and serve info.php using fpm and deny access to `.ht`.
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index info.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

![Display phpinfo from info.php](https://user-images.githubusercontent.com/23631617/142194627-a5d440a9-6915-45c0-ab2a-6903b3c6b00a.png)
