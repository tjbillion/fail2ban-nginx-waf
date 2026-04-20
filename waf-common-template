# 1. nginx sites-available specific file
```bash
server {
    server_name yourwebsite.com
    root /var/www/yourwebsite.com;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    index index.php index.html;
    client_max_body_size 64M;
    
    # Only allow Cloudflare IPs
    if ($http_cf_connecting_ip = "") {
        return 403;
    }

    # Allow robots.txt
    location = /robots.txt { allow all; }
    
    # If you want to block specific user agent
    if ($http_user_agent ~* "User Agent String As You Need") {
        return 403;
    }

    # Block access to sensitive text files
    location ~* \.(txt|md|pot|log|sql|bak|gz|zip|tar)$ {
        return 403;
    }
    
    # Deny file starting with .ht
    location ~ /\.ht {
        return 403;
    }
    
	# Rate limit login - adjust burst rate number to fit your traffic
    location = /admin-login {
	    try_files $uri =404;
        limit_req zone=login burst=10 nodelay;
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

	# Handle php files and try_files here is very important, check if the files really exist, prevent RCE misconfiguration    
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Add more rules as needed 
    # According to your needs
    
    # finally, setup the SSL
    listen 443 ssl;
    ssl_certificate YOUR_CERT_PEM_FILE;
    ssl_certificateE_key YOUR_CERT_PRIVATE_KEY_FILE;
    include YOUR_SSL_OPTIONS_FILE;
    ssl_dhparam YOUR_DHPARAM_FILE;
}

server {
    listen 80;
    server_name yourwebsite.com;

    # Only allow Cloudflare IPs
    if ($http_cf_connecting_ip = "") {
        return 403;
    }

	# Redirect legitimate traffic
    if ($host = yourwebsite.com) {
        return 301 https://$host$request_uri;
    } 

	# Deny file starting with .ht
    location ~ /\.ht {
        deny all;
    }
    
    # Add more rules as needed
    # According to your needs
    
    # Final condition, default 403 to let fail2ban catch it
    return 403;
}
```

# 2. nginx.conf file
```
cat /etc/nginx/nginx.conf

cat /etc/nginx/nginx.conf

http {

# 1. disable advertising version
server_tokens off;

# 2. set the real IP of the visitors, not Cloudflare IPs
        set_real_ip_from 103.21.244.0/22;
        set_real_ip_from 103.22.200.0/22;
        set_real_ip_from 103.31.4.0/22;
        set_real_ip_from 141.101.64.0/18;
        set_real_ip_from 108.162.192.0/18;
        set_real_ip_from 190.93.240.0/20;
        set_real_ip_from 188.114.96.0/20;
        set_real_ip_from 197.234.240.0/22;
        set_real_ip_from 198.41.128.0/17;
        set_real_ip_from 162.158.0.0/15;
        set_real_ip_from 104.16.0.0/13;
        set_real_ip_from 104.24.0.0/14;
        set_real_ip_from 172.64.0.0/13;
        set_real_ip_from 131.0.72.0/22;
        set_real_ip_from 2400:cb00::/32;
        set_real_ip_from 2606:4700::/32;
        set_real_ip_from 2803:f800::/32;
        set_real_ip_from 2405:b500::/32;
        set_real_ip_from 2405:8100::/32;
        set_real_ip_from 2a06:98c0::/29;
        set_real_ip_from 2c0f:f248::/32;
        set_real_ip_from 2404:c0::/32;
        real_ip_header CF-Connecting-IP;
        

# 3. Basic Security Headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;
        add_header Content-Security-Policy "frame-ancestors 'self'" always;
        add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# 4. Default file upload buffer template
        # Request buffer settings
        # Adjust based on your expected request sizes and traffic pattern
        client_body_buffer_size 16k;
        client_header_buffer_size 1k;
        large_client_header_buffers 4 8k;
}
```

# 3. fail2ban filter.d setup example
```
cat /etc/fail2ban/filter.d/admin-login.conf

[Definition]
failregex = ^<HOST> .* "POST /admin-login.php
            ^<HOST> .* "POST /admin.php
			//add more to fit your web app
```

# 4. setup fail2ban jail file
```
cat /etc/fail2ban/jail.local
[DEFAULT]
ignoreip = 127.0.0.1/8
backend = auto

# Your custom WAF setup to Cloudflare
[admin-login]
enabled = true
port = http,https
# this is the .conf file inside /etc/fail2ban/filter.d
filter = admin-login
# the log to read
logpath = /var/log/nginx/access.log
maxretry = # how many tries before you want to trigger the action
bantime = # how long you want to ban or action applied (in seconds)
findtime = # set the "within" time
backend = auto # keep this auto is fine most of the time
action = cloudflare-token[cftoken="xxxxxxxxxxxxxxxxx", cfzone="xxxxxxxxxxxxxxxx", cfmode="xxxxxxxxx"] # cfmode here is where what action you want to set, e.g. challenge / ban IP

# SSH brute force
[sshd]
enabled = true
port = ssh
maxretry = # how many "after retries" then you want to block the IP
bantime = # how long the IP will be block, or block it forever?
```
