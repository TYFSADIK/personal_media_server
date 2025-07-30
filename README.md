# 📸 Self-Hosted Photo Server without Port Forwarding

This project enables you to host your own **photo and video gallery** using [Immich](https://github.com/immich-app/immich) on your personal server. It’s **secure**, **free**, and **private**, eliminating the need for paid services like Google Photos or iCloud to store your memories. The setup leverages **Docker**, **Cloudflare Tunnel**, and any machine with storage (e.g., an old laptop or desktop).

---

## 🔒 Why Use This?

- Prioritize **privacy** for your personal data
- Avoid **paying** for storing your own media
- Enjoy building your **homelab**
- Repurpose an old machine (e.g., a **laptop with 1TB** storage)

---

## 💻 Prerequisites

- A Linux machine (e.g., Ubuntu Server) or an old laptop
- **Docker** and **Docker Compose** installed
- A free **Cloudflare account**
- Optional: A custom domain (e.g., `photos.yourdomain.com`)

---

## 🚀 Installation Guide

### ✅ Step 1: Install Docker and Docker Compose

Update your system and install Docker:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

🔁 **Log out and back in** (or run `newgrp docker`) to apply Docker group permissions.

---

### ✅ Step 2: Clone the Repository

Clone this project to your machine:

```bash
git clone https://github.com/YOUR_USERNAME/immich-selfhost.git
cd immich-selfhost
```

❗ **Replace** `YOUR_USERNAME` with your GitHub username.

---

### ✅ Step 3: Set Up Docker Compose

Create a `docker-compose.yml` file:

```bash
nano docker-compose.yml
```

Paste the following configuration:

```yaml
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
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

### ✅ Step 4: Launch the Server

Start the containers in detached mode:

```bash
docker-compose up -d
```

Wait a minute, then visit:  
🔗 [http://localhost:2283](http://localhost:2283)

---

### ✅ Step 5: Configure Cloudflare Tunnel

Install the `cloudflared` tool:

```bash
wget -O cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
```

Authenticate with Cloudflare:

```bash
cloudflared tunnel login
```

Create a tunnel:

```bash
cloudflared tunnel create immich-tunnel
```

Set up the tunnel configuration:

```bash
sudo mkdir -p /etc/cloudflared
sudo nano /etc/cloudflared/config.yml
```

Paste and customize this config:

```yaml
tunnel: YOUR_TUNNEL_ID_HERE
credentials-file: /etc/cloudflared/YOUR_TUNNEL_CREDENTIALS.json

ingress:
  - hostname: photos.YOURDOMAIN.com
    service: http://localhost:2283
  - service: http_status:404
```

🟡 **Replace:**
- `YOUR_TUNNEL_ID_HERE` — Use the ID from `cloudflared tunnel list`
- `YOUR_TUNNEL_CREDENTIALS.json` — Path provided after tunnel creation
- `photos.YOURDOMAIN.com` — Your chosen subdomain in Cloudflare

Save and exit.

---

### ✅ Step 6: Run the Tunnel as a Service

Install and enable the tunnel service:

```bash
cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
```

---

## 📸 Access Your Immich Instance

Visit your gallery at:  
🔗 [https://photos.YOURDOMAIN.com](https://photos.YOURDOMAIN.com)

✅ Your Immich server is now running securely and accessible remotely!

---

## 📦 Project Folder Structure

```bash
immich-selfhost/
├── docker-compose.yml
├── cloudflared/
│   └── config.yml
├── immich-data/         # Stores your media
├── pgdata/              # PostgreSQL data
├── screenshots/
│   └── web-ui.png
└── README.md
```

---

## 🛠️ Useful Tips

- **Mobile Upload:** Download the Immich app (Android/iOS) and connect using your domain.
- **Tunnel-Only Access:** Remove the `ports:` section from `docker-compose.yml` to restrict access to the tunnel.
- **Backups:** Mount an external drive to `./immich-data` for additional storage or redundancy.

---

## 📌 Optional Enhancements

- Add **basic auth** or **SSO** for extra security
- Enable **object detection** in Immich settings
- Use a **reverse proxy** (Nginx) instead of Cloudflare Tunnel

---

## 🙋‍♂️ About This Project

This is a personal homelab project driven by a passion for **data privacy** and a desire to break free from paid photo storage platforms. I hope it inspires others to take control of their media!

💬 Feel free to fork, star, or submit issues if you give it a try.

---

## ⭐ Enjoyed This?

If this project helped you, give it a 🌟 **star** on GitHub!

---

### 📝 Additional Notes

- **Security:** Double-check your Cloudflare tunnel config to avoid exposing unintended services.
- **Storage:** Keep an eye on `./immich-data` as it grows with your uploads.
- **Updates:** Run `docker-compose pull && docker-compose up -d` to update Immich.

---

### 📸 Screenshots

![Immich Web UI])
<img width="2940" height="1912" alt="image" src="https://github.com/user-attachments/assets/fa801f5a-66c8-493e-b40c-fb423992fabf" />


---

**Happy self-hosting!** 🎉
