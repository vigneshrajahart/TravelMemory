# 🌍 TravelMemory Deployment on AWS (with Cloudflare + SSL + Scaling)

This guide walks you through deploying the **TravelMemory MERN application** on AWS EC2, securing it using **Cloudflare + Let’s Encrypt SSL**, and scaling with multiple instances and a Load Balancer.

---

## 🔧 Tech Stack

* MERN (MongoDB, Express, React, Node.js)
* AWS EC2 (Ubuntu 22.04 LTS)
* NGINX (Reverse Proxy)
* Cloudflare (DNS + SSL)
* Certbot (HTTPS SSL)
* PM2 (Node process manager)
* Load Balancer (Application Load Balancer - optional for scaling)

---

## ✅ Prerequisites

* AWS account
* Domain name (e.g., from GoDaddy or Namecheap)
* Cloudflare account (Free tier is enough)

---

## ⚙️ Phase 1: Launch EC2 Instance

1. Go to [AWS EC2 Dashboard](https://console.aws.amazon.com/ec2/)
2. Launch instance:

   * OS: **Ubuntu 22.04 LTS**
   * Type: **t2.micro** (Free Tier)
3. Configure Security Group: Allow

   * HTTP (80)
   * HTTPS (443)
   * SSH (22)
   * Custom TCP (3000)

---

## ⚙️ Phase 2: Connect to EC2 & Install Dependencies

```bash
ssh -i /path/to/key.pem ubuntu@EC2_PUBLIC_IP
```

Update and install packages:

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs npm git nginx
sudo npm install -g pm2
```

---

## ⚙️ Phase 3: Clone Backend & Run

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
npm install
```

If `start` script is missing, add to `package.json`:

```json
"scripts": {
  "start": "node index.js"
}
```

Start backend:

```bash
pm2 start index.js --name travel-backend
```

---

## ⚙️ Phase 4: Setup Frontend

```bash
cd ../frontend
npm install
```

Update `src/urls.js`:

```js
export const backendUrl = "/api";
```

Then build the frontend:

```bash
npm run build
sudo cp -r build/* /var/www/html/
```

---

## ⚙️ Phase 5: Configure NGINX

```bash
sudo nano /etc/nginx/sites-available/travelmemory
```

Paste the config:

```nginx
server {
    listen 80;
    server_name memoriesnverfade.in www.memoriesnverfade.in;

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

Enable and test:

```bash
sudo ln -s /etc/nginx/sites-available/travelmemory /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## ⚙️ Phase 6: Cloudflare Setup

### Step 1: Add Site to Cloudflare

1. Login to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Click **Add Site** and enter `memoriesnverfade.in`
3. Choose **Free Plan** and continue

### Step 2: Update Nameservers

1. Cloudflare will provide two nameservers
2. Go to your domain registrar (e.g., GoDaddy)
3. Replace default nameservers with Cloudflare’s

### Step 3: Add DNS Records

| Type  | Name | Content         | Proxy     |
| ----- | ---- | --------------- | --------- |
| A     | @    | EC2\_PUBLIC\_IP | Proxied ✅ |
| CNAME | www  | @               | Proxied ✅ |

### Step 4: Enable SSL

1. Go to **SSL/TLS** tab in Cloudflare
2. Set SSL Mode to **Full (strict)**

---

## ⚙️ Phase 7: Setup HTTPS with Certbot (Let’s Encrypt)

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run SSL command:

```bash
sudo certbot --nginx -d memoriesnverfade.in -d www.memoriesnverfade.in
```

Test auto-renew:

```bash
sudo certbot renew --dry-run
```

---

## 📈 Phase 8: Scaling the Application

1. **Create multiple EC2 instances** using your configured AMI
2. **Use an Application Load Balancer (ALB):**

   * Add both frontend and backend EC2 instances to Target Groups
   * Configure ALB Listener Rules (port 80 → NGINX)
3. **Point Cloudflare DNS A record** to the ALB DNS name instead of single EC2 IP

---

## 🗺️ Architecture Diagram

```
Client (Browser)
       │
       ▼
Cloudflare (DNS + SSL)
       │
       ▼
 Application Load Balancer (optional)
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

```bash
https://memoriesnverfade.in
```

---

## 🧹 Troubleshooting

* NGINX config test: `sudo nginx -t`
* PM2 logs: `pm2 logs travel-backend`
* Check EC2 Security Group for open ports
* Test domain DNS: [https://dnschecker.org](https://dnschecker.org)

---

## 👏 Done!

Your MERN app is deployed, secure, and scalable!

---

> Built with ❤️ by Vignesh
