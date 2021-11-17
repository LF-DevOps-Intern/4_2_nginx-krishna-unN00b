# Solution

1. Install nginx and host a simple index.html with message “hello nginx”

```console
sudo apt install nginx
```

##### index.html
```html
<h1>hello nginx</h1>
```

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

##### test.conf
```console
server {
	listen 80 default_server;
	listen [::]:82 default_server;

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