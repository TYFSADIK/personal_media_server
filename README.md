# 📸 Self-Hosted Immich Server with Docker and Cloudflare Tunnel

This project allows you to host your own **photo and video gallery** using [Immich](https://github.com/immich-app/immich), running on your own server — **secure**, **free**, and **private**.

No need to pay for Google Photos, iCloud, or other platforms to store your personal memories. This setup uses **Docker**, **Cloudflare Tunnel**, and any machine with some storage (like an old laptop or desktop).

---

## 🔒 Why?

- I value my **privacy**
- I don't want to pay for my **own pictures**
- I enjoy building my **homelab**
- I had an old **laptop with 1TB** storage sitting around

---

## 💻 What You'll Need

- A Linux machine (like Ubuntu Server) or even an old laptop
- Docker + Docker Compose installed
- Cloudflare account (free)
- Optional: your own domain (e.g. `photos.yourdomain.com`)

---

## 🚀 Step-by-Step Installation Guide

### ✅ Step 1: Install Docker and Docker Compose

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo usermod -aG docker $USER
🔁 Log out and back in (or run newgrp docker) to activate Docker group permissions.

✅ Step 2: Clone This Repo
bash
Copy
Edit
git clone https://github.com/YOUR_USERNAME/immich-selfhost.git
cd immich-selfhost
❗ Replace YOUR_USERNAME with your GitHub username.

✅ Step 3: Create Docker Compose File
Create a file named docker-compose.yml:

bash
Copy
Edit
nano docker-compose.yml
Paste this configuration:

yaml
Copy
Edit
version: "3.9"
services:
  immich:
    image: ghcr.io/imagegenius/immich:latest
    container_name: immich
    ports:
      - "2283:2283"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - DB_HOSTNAME=postgres14
      - DB_USERNAME=postgres
      - DB_PASSWORD=postgres
      - DB_DATABASE_NAME=immich
      - REDIS_HOSTNAME=valkey
      - SERVER_PORT=2283
    volumes:
      - ./immich-data:/usr/src/app/upload
    depends_on:
      - postgres14
      - valkey
    restart: unless-stopped

  postgres14:
    image: postgres:14
    container_name: postgres14
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=immich
    volumes:
      - ./pgdata:/var/lib/postgresql/data
    restart: unless-stopped

  valkey:
    image: valkey/valkey
    container_name: valkey
    restart: unless-stopped
✅ Step 4: Start the Server
bash
Copy
Edit
docker-compose up -d
Wait a minute, then visit:
🔗 http://localhost:2283

✅ Step 5: Set Up Cloudflare Tunnel
Install cloudflared:

bash
Copy
Edit
wget -O cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
Login to Cloudflare:

bash
Copy
Edit
cloudflared tunnel login
Create a Tunnel:

bash
Copy
Edit
cloudflared tunnel create immich-tunnel
Create the config file:

bash
Copy
Edit
sudo mkdir -p /etc/cloudflared
sudo nano /etc/cloudflared/config.yml
Paste and edit this config:

yaml
Copy
Edit
tunnel: YOUR_TUNNEL_ID_HERE
credentials-file: /etc/cloudflared/YOUR_TUNNEL_CREDENTIALS.json

ingress:
  - hostname: photos.YOURDOMAIN.com
    service: http://localhost:2283
  - service: http_status:404
🟡 Replace:

YOUR_TUNNEL_ID_HERE — copy from cloudflared tunnel list

YOUR_TUNNEL_CREDENTIALS.json — path given after tunnel create

photos.YOURDOMAIN.com — your subdomain from Cloudflare

✅ Step 6: Run the Tunnel as a Service
bash
Copy
Edit
cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
📸 Access Your Private Immich
Now go to:
🔗 https://photos.YOURDOMAIN.com

✅ Immich is running securely and remotely!

📦 Folder Structure
bash
Copy
Edit
immich-selfhost/
├── docker-compose.yml
├── cloudflared/
│   └── config.yml
├── immich-data/         # media storage
├── pgdata/              # postgres data
├── screenshots/
│   └── web-ui.png
└── README.md
🛠️ Tips
Mobile upload? Install Immich app (Android/iOS) and use your domain.

Want to expose only via tunnel? Remove ports: from docker-compose.yml.

Backups? Mount an external drive to ./immich-data.

📌 Optional Improvements
Add basic auth or SSO

Enable object detection in Immich

Use a reverse proxy like Nginx (optional if not using Cloudflare Tunnel)

🙋‍♂️ About This Project
This is a personal project that’s part of my homelab journey. I value data privacy and wanted to stop relying on paid third-party platforms for my photos.

💬 Feel free to fork, star, or open issues if you try this too.

⭐ Like This Project?
Give it a 🌟 star on GitHub if it helped you!
