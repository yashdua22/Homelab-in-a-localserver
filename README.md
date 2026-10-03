# Homelab-in-a-localserver
# 🏠 Homelab-in-a-Box

> Turn a single laptop into a personal server running four self-hosted services — all defined in one `docker-compose.yml` and launched with one command.


## 📖 Overview

Instead of renting SaaS for everything, this project makes you your own SaaS
provider for an audience of one. A single Docker host runs four containers that
together give you a private cloud, network-wide ad blocking, a management panel,
and uptime monitoring — the foundation of any homelab.

Everything is declarative: the entire stack lives in one Compose file, so it's
reproducible, version-controlled, and tears down cleanly.

## 🧩 What's in the box

| Service | Role | Port |
|---------|------|------|
| **Portainer** | Web UI to manage all Docker containers | `9000` |
| **Pi-hole** | Network-wide DNS ad blocker | `53` (DNS), `8081` (admin) |
| **Nextcloud** | Self-hosted file storage ("private Google Drive") | `8080` |
| **Uptime Kuma** | Status dashboard that pings services and alerts on failure | `3001` |

## 🏗️ Architecture

```
                 ┌──────────────────────────────────────────┐
   Home devices  │            Your laptop (Docker host)       │
   phone / TV ───┼──► Pi-hole (DNS + ad block)                │
   over WiFi     │    Nextcloud (files)                       │
                 │    Portainer (controls all containers) ◄───┤
                 │    Uptime Kuma (watches all containers) ◄──┤
                 └──────────────────────────────────────────┘
```

Devices on the LAN point their DNS at the laptop. Pi-hole answers every lookup
and silently drops ad/tracker domains. Nextcloud serves files to the same
devices. Portainer and Uptime Kuma are the behind-the-scenes crew — one manages
the containers, the other watches them.

## 🚀 Quick start

```bash
# 1. Clone and enter
git clone https://github.com/<your-username>/homelab.git
cd homelab

# 2. Create your .env from the example and set strong passwords
cp .env.example .env
nano .env

# 3. (Linux) free up port 53 for Pi-hole
sudo sed -i 's/#DNSStubListener=yes/DNSStubListener=no/' /etc/systemd/resolved.conf
sudo systemctl restart systemd-resolved

# 4. Launch the whole stack
docker compose up -d

# 5. Confirm all four are running
docker compose ps
```

Then open each service in your browser at `http://<laptop-ip>:<port>` using the
ports in the table above.

## 📂 Project structure

```
homelab/
├── docker-compose.yml   # the recipe: defines all 4 services
├── .env                 # your secrets (gitignored)
├── .env.example         # template to copy from
├── .gitignore
└── README.md
```

## ⚙️ Configuration

All secrets and host-specific values live in `.env`:

```env
PIHOLE_PASSWORD=change_me
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=change_me
HOST_IP=192.168.x.x        # your laptop's LAN IP (run: hostname -I)
```

Data persists in Docker named volumes, so containers can be recreated or
upgraded without losing Nextcloud files or Pi-hole blocklists.

## 🧪 Verify it works

- **Ad blocking:** point a device's DNS at the laptop, then visit the
  [ad-block tester](https://d3ward.github.io/toolz/adblock) — you should see a
  high block rate, and Pi-hole's dashboard query count climbing.
- **Monitoring:** Uptime Kuma should show all monitors green. (Nextcloud is best
  monitored with a **TCP Port** check on `8080`, since it rejects HTTP requests
  from untrusted domains with a `400`.)

## 🧰 Common commands

```bash
docker compose up -d            # start / apply changes
docker compose ps               # status
docker compose logs -f <app>    # live logs for one service
docker compose pull && docker compose up -d   # upgrade all images
docker compose down             # stop & remove containers (volumes kept)
```

## 📸 Screenshots

> _Add yours here._

| Portainer | Pi-hole | Uptime Kuma |
|-----------|---------|-------------|
| _container list_ | _blocked queries_ | _all green_ |

## 📚 What I learned

- Writing a multi-service `docker-compose.yml` with named volumes for persistence
- How DNS-based ad blocking actually works (and why YouTube ads survive it)
- Port mapping and avoiding host port clashes across services
- Why port 53 conflicts with `systemd-resolved`, and how to resolve it
- Monitoring self-hosted apps correctly (TCP vs HTTP checks, trusted domains)

## 🔮 Possible next steps

- Add a reverse proxy (Nginx Proxy Manager / Traefik) for clean hostnames + HTTPS
- Swap Nextcloud's built-in DB for MariaDB + Redis for production-grade performance
- Set Pi-hole as the router's DHCP DNS so every device is covered automatically
- Add Grafana + Prometheus for metrics

## 📄 License

MIT — use it, fork it, break it, learn from it.
