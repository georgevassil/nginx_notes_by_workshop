# NGINX MASTER CHEATSHEET

## 1. THE COMMANDS (Terminal)
# ---------------------------------------------------------
# ALWAYS run this first to check for errors
nginx -t

# Reloads config without stopping the server (Safe)
nginx -s reload

# Test your local server from command line
curl 127.0.0.1
curl 127.0.0.1/stub_status

# View the active processes (Master vs Workers)
ps aux | grep nginx


## 2. THE HOSTS HACK (/etc/hosts)
# ---------------------------------------------------------
# Trick your computer into thinking these domains are local (127.0.0.1)
# File: /etc/hosts
127.0.0.1   localhost www.example.com cars.example.com cafe.example.com


## 3. THE CONFIGURATION (The "Brain")
# ---------------------------------------------------------
# File: /etc/nginx/conf.d/mysite.conf

server {
    # -- 1. ENTRY POINT --
    listen 80 default_server;       # Port 80. 'default' catches unmatched requests.
    server_name www.example.com;    # The domain this block controls.

    # -- 2. LOGGING --
    access_log /var/log/nginx/access.log;  # Records every visit.
    error_log /var/log/nginx/error.log;    # Records crashes/issues.

    # -- 3. BASIC RESPONSE --
    location / {
        default_type text/html;     # Treat string as HTML.
        return 200 "Hello World";   # Send text immediately. Stop.
    }

    # -- 4. METRICS (Stub Status) --
    location /stub_status {
        stub_status;                # Shows active connections/requests.
    }

    # -- 5. CLEAN URLs (try_files) --
    # User types: /gtr
    # Nginx looks for: /gtr.html
    location /gtr {
        root /var/www/html;
        try_files $uri $uri.html;
    }

    # -- 6. ROOT vs ALIAS (The tricky part) --
    
    # CASE A: ROOT (Math: Path + URL)
    # URL: /img/cat.jpg
    # Path: /var/www/html/img/cat.jpg
    location /img {
        root /var/www/html;
    }

    # CASE B: ALIAS (Math: Replaces URL)
    # URL: /browse/cat.jpg
    # Path: /var/www/html/cat.jpg  (Notice /browse is gone)
    location /browse {
        alias /var/www/html;
        index index.html;       # If folder requested, show this file.
        autoindex off;          # Security: Hide file list if index is missing.
    }
}


## 4. CORE CONCEPTS (Dictionary)
# ---------------------------------------------------------
# 127.0.0.1 (Loopback):
#   "Home". Your computer talking to itself. Works offline.
#   Also called "localhost".

# Ports:
#   Port 80: The door Nginx opens to listen (Destination).
#   Ephemeral Port: Random door 'curl' opens to get the reply (Source).

# Daemon:
#   A program that runs in background (detached from terminal).

# Master Process:
#   The Boss. Reads config, manages workers.

# Worker Process:
#   The Employee. Handles the actual connections using an Event Loop.
#   (One worker handles thousands of users).

# /usr/share/nginx/html:
#   System default folder. Don't touch.

# /var/www/:
#   User folder. Put your custom websites here.
