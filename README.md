# Webmin Project Setup



## Prerequisites

* Ubuntu 20.04+ or compatible Linux
* Node.js v14+ and npm
* PM2 (`npm install -g pm2`)
* Nginx
* Certbot for Let's Encrypt SSL
* cPanel or DNS management access

## Installation

1. SSH into your server:

   ```bash
   ssh root@YOUR_SERVER_IP
   ```
2. Clone the repository:

   ```bash
   git clone https://github.com/yourusername/ytdownloader.git /var/www/ytdownloader
   ```
3. Install dependencies:

   ```bash
   cd /var/www/ytdownloader
   npm install
   ```

## Configuration

### CORS and Domain

* Open `app.js` (or `server.js`) and set your domain:

  ```js
  const corsOptions = {
    origin: ['https://script.formhistory.com']
  };
  app.use(cors(corsOptions));
  ```
* Save and exit.

### DNS Setup (cPanel)

1. Login to cPanel → **Zone Editor** → Manage.
2. Add an A Record:

   * **Name**: `script`
   * **Type**: A
   * **Value**: Your server's public IP
3. Save and wait for DNS propagation (up to 24 hours, typically 1-2 hours).

### Nginx Configuration

Create or update `/etc/nginx/sites-available/default`:

```nginx
server {
    listen 80;
    server_name domian name;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl;
    server_name script.formhistory.com;

    ssl_certificate /etc/letsencrypt/live/domian name/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domian name/privkey.pem;

    root /var/www/projct name;
    index index.html;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 300s;
        proxy_read_timeout 300s;
    }

    location /uploads/ {
        alias /var/www/projct name/uploads/;
        autoindex on;
        add_header Access-Control-Allow-Origin *;
    }
}
```

Reload:

```bash
nginx -t
nginx -s reload
```

### SSL Setup

```bash
apt update
apt install certbot python3-certbot-nginx -y
certbot --nginx -d script.formhistory.com
```

## Running the Application

Start with PM2:

```bash
pm2 start app.js --name projct name
pm2 save
```

Check status:

```bash
pm2 list
pm2 logs projct name
```

## Local DNS Flush (Windows)

If local DNS caching affects testing:

```bash
ipconfig /flushdns
```

## Testing

Visit `https://example.com` to verify the application is live and SSL is working.

## Flow Chart

```
Domain → DNS Setup → Server Config → Nginx Setup → App Restart → SSL Setup → Browser Test
```

## License

This project is licensed under the MIT License.
