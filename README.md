# 🚀 NGINX Master Cheatsheet

A quick reference guide for Nginx commands, configurations, and core concepts.

---

## 🛠 1. The Commands (Terminal)

| Action | Command | Note |
| :--- | :--- | :--- |
| **Check Errors** | `nginx -t` | **ALWAYS** run this first. |
| **Reload** | `nginx -s reload` | Reloads config without stopping server (Safe). |
| **Test Local** | `curl 127.0.0.1` | Test your local server from CLI. |
| **Metrics** | `curl 127.0.0.1/stub_status` | View active connections (requires stub setup). |
| **Process View** | `ps aux | grep nginx` | View Master vs Worker processes. |

---

## 💻 2. The Hosts Hack (`/etc/hosts`)

Trick your computer into thinking domains are local (`127.0.0.1`).

```bash
# File: /etc/hosts
127.0.0.1    localhost [www.example.com](https://www.example.com) cars.example.com cafe.example.com
```

---

## 🧠 3. The Configuration (The "Brain")

**File:** `/etc/nginx/conf.d/mysite.conf`

```nginx
server {
    # -- 1. ENTRY POINT --
    listen 80 default_server;       # Port 80. 'default' catches unmatched requests.
    server_name [www.example.com](https://www.example.com);    # The domain this block controls.

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
    # User types: /gtr -> Nginx looks for: /gtr.html
    location /gtr {
        root /var/www/html;
        try_files $uri $uri.html;
    }

    # -- 6. ROOT vs ALIAS (The Tricky Part) --
    
    # CASE A: ROOT (Math: Path + URL)
    # URL: /img/cat.jpg  ->  Path: /var/www/html/img/cat.jpg
    location /img {
        root /var/www/html;
    }

    # CASE B: ALIAS (Math: Replaces URL)
    # URL: /browse/cat.jpg  ->  Path: /var/www/html/cat.jpg
    # (Notice /browse is gone)
    location /browse {
        alias /var/www/html;
        index index.html;       # If folder requested, show this file.
        autoindex off;          # Security: Hide file list if index is missing.
    }
}
```

---

## 📚 4. Core Concepts (Dictionary)

| Concept | Definition |
| :--- | :--- |
| **127.0.0.1** | "Home" (Loopback). Your computer talking to itself. Works offline. Also called `localhost`. |
| **Port 80** | The door Nginx opens to **listen** (Destination). |
| **Ephemeral Port**| The random door `curl` opens to **get the reply** (Source). |
| **Daemon** | A program that runs in the background (detached from terminal). |
| **Master Process**| **The Boss.** Reads config, manages workers. |
| **Worker Process**| **The Employee.** Handles actual connections using an Event Loop. (One worker handles thousands of users). |
| `/usr/share/...`| System default html folder. **Don't touch.** |
| `/var/www/` | User folder. **Put your custom websites here.** |
