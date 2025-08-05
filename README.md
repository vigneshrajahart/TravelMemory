# 🌍 TravelMemory Deployment on AWS (with Cloudflare + SSL)

This guide will walk you through deploying the **TravelMemory MERN application** on AWS EC2, securing it using **Cloudflare** and **Let’s Encrypt SSL (Certbot)**.

---

## 🔧 Tech Stack

- MERN (MongoDB, Express, React, Node.js)
- AWS EC2 (Ubuntu 22.04 LTS)
- NGINX (Reverse Proxy)
- Cloudflare (DNS + SSL)
- Certbot (HTTPS SSL)

---

## ✅ Prerequisites

- AWS account
- Domain name (from GoDaddy, Namecheap, etc.)
- Cloudflare account (Free plan is fine)

---

## ⚙️ Phase 1: Launch EC2 Instance

1. Go to [AWS EC2 Dashboard](https://console.aws.amazon.com/ec2/)
2. Launch instance:
   - OS: **Ubuntu 22.04 LTS**
   - Type: **t2.micro** (Free Tier)
3. Security Group: Allow
   - HTTP (80)
   - HTTPS (443)
   - SSH (22)
   - Custom TCP (3000)

---

## ⚙️ Phase 2: Connect to EC2

```bash
ssh -i /path/to/key.pem ubuntu@EC2_PUBLIC_IP
```

Update and install Node.js:

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs npm
```

Install Git and PM2:

```bash
sudo apt install git -y
sudo npm install -g pm2
```

---

## ⚙️ Phase 3: Clone Backend and Run

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
npm install
```

Add a start script in `package.json` if not present:

```json
"scripts": {
  "start": "node index.js"
}
```

Start server:

```bash
pm start
```

Or use PM2:

```bash
pm install -g pm2
pm2 start index.js --name travel-backend
```

---

## ⚙️ Phase 4: Frontend Build

```bash
cd ../frontend
npm install
```

Update `src/urls.js` to:

```javascript
export const backendUrl = "/api";
```

Then build and deploy:

```bash
npm run build
sudo cp -r build/* /var/www/html/
```

---

## ⚙️ Phase 5: NGINX Configuration

Install NGINX:

```bash
sudo apt install nginx -y
```

Create a config file:

```bash
sudo nano /etc/nginx/sites-available/travelmemory-backend
```

Paste:

```nginx
server {
    listen 80;
    server_name travelmemory.in www.travelmemory.in;

    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ =404;
    }
}
```

Enable and restart:

```bash
sudo ln -s /etc/nginx/sites-available/travelmemory-backend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## ⚙️ Phase 6: Cloudflare + Domain Setup

### Step 1: Add Site in Cloudflare

1. Login to [Cloudflare](https://dash.cloudflare.com)
2. Click **Add Site**
3. Enter your domain (e.g., `travelmemory.in`)
4. Choose **Free Plan**

### Step 2: Update Nameservers

1. Cloudflare will show 2 nameservers
2. Go to your domain registrar (e.g., GoDaddy)
3. Replace default nameservers with Cloudflare's

### Step 3: DNS Configuration

In Cloudflare DNS tab:

| Type  | Name | Content         | Proxy     |
| ----- | ---- | --------------- | --------- |
| A     | @    | `EC2 Public IP` | Proxied ✅ |
| CNAME | www  | @               | Proxied ✅ |

### Step 4: SSL Setup in Cloudflare

- Go to SSL/TLS tab
- Select **Full (strict)** mode

---

## ⚙️ Phase 7: Enable HTTPS with Certbot (Let’s Encrypt)

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run SSL setup:

```bash
sudo certbot --nginx -d travelmemory.in -d www.travelmemory.in
```

Test auto-renew:

```bash
sudo certbot renew --dry-run
```

---

## 🗺️ Architecture Diagram

```
Client (Browser)
       │
       ▼
Cloudflare (DNS + SSL)
       │
       ▼
   AWS EC2 (Ubuntu)
       │
  ┌────┴────┐
  ▼         ▼
NGINX   Node.js (3000)
  │
  ▼
React Build (/var/www/html)
```

---

## ✅ Live App

Visit your domain:

```
https://travelmemory.in
```

---

## 🧹 Troubleshooting Tips

- NGINX config test: `sudo nginx -t`
- PM2 logs: `pm2 logs travel-backend`
- SSL test: [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)
- Port access: check EC2 security group

---

## 👏 Done!

Your MERN app is now fully deployed, secure, and production-ready on AWS with Cloudflare!

---

> Built with ❤️ by Vignesh

