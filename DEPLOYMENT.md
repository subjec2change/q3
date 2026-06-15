# Quake 3:TA Boomer Break Deployment Guide

Host your own web-based Quake III Arena LAN party with HTTPS support.

---

## 📋 Prerequisites

- Docker & Docker Compose v2+
- A domain name (for public HTTPS) **OR** a LAN IP (for local HTTPS)
- Ports 80, 443, 8080, 8081 available
- Quake 3 Demo `pak0.pk3` file

---

## 🎮 Quick Start (LAN with HTTPS)

### 1. Clone & Prepare

```bash
git clone https://github.com/subjec2change/q3
cd q3
```

### 2. Get Quake 3 Demo Assets

Download the Quake 3 Arena demo and extract `pak0.pk3`:

```bash
# Option A: Use the included download script (in client/Dockerfile)
wget -O linuxq3ademo-1.11-6.x86.gz.sh \
  "http://archive.org/download/tucows_286139_Quake_III_Arena/linuxq3ademo-1.11-6.x86.gz.zip/linuxq3ademo-1.11-6.x86.gz.sh"
chmod +x linuxq3ademo-1.11-6.x86.gz.sh
./linuxq3ademo-1.11-6.x86.gz.sh --target demoq3_extract
cp demoq3_extract/demoq3/pak0.pk3 demoq3/
rm -rf demoq3_extract linuxq3ademo-1.11-6.x86.gz.sh

# Option B: If you have the full game, copy pak0.pk3 from baseq3/
cp /path/to/quake3/baseq3/pak0.pk3 demoq3/
```

### 3. Generate LAN TLS Certificates (Local HTTPS)

For LAN IP access with valid HTTPS (no browser warnings):

```bash
# Install mkcert (one-time)
# Linux: sudo apt install libnss3-tools && go install filippo.io/mkcert@latest
# macOS: brew install mkcert nss
# Windows: choco install mkcert

# Create local CA and cert for your LAN IP
mkcert -install
mkcert 192.168.1.100 localhost 127.0.0.1 ::1

# This creates: 192.168.1.100+3.pem and 192.168.1.100+3-key.pem
# Copy to Caddy config location:
mkdir -p caddy/certs
cp 192.168.1.100+3.pem caddy/certs/cert.pem
cp 192.168.1.100+3-key.pem caddy/certs/key.pem
```

### 4. Configure Caddy for LAN

Edit `Caddyfile` with your LAN IP:

```caddyfile
# Replace 192.168.1.100 with your actual LAN IP
https://192.168.1.100 {
    tls /etc/caddy/certs/cert.pem /etc/caddy/certs/key.pem
    encode zstd gzip

    handle /peer* {
        uri strip_prefix /peer
        reverse_proxy shake-server:8080
    }

    handle /lookup* {
        reverse_proxy shake-server:8080
    }

    handle {
        reverse_proxy shake-client:8081
    }
}

# Also serve on localhost for host machine
https://localhost {
    tls /etc/caddy/certs/cert.pem /etc/caddy/certs/key.pem
    encode zstd gzip

    handle /peer* {
        uri strip_prefix /peer
        reverse_proxy shake-server:8080
    }

    handle /lookup* {
        reverse_proxy shake-server:8080
    }

    handle {
        reverse_proxy shake-client:8081
    }
}
```

### 5. Update docker-compose.yml for Caddy

```yaml
version: '3.8'

services:
  shake-client:
    image: xanderstrike/shake-client:latest
    container_name: shake-client
    build:
      context: ./client
      dockerfile: Dockerfile
    volumes:
      - ./demoq3:/usr/share/nginx/html/demoq3
    expose:
      - "8081"
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

  shake-server:
    image: xanderstrike/shake-server:latest
    container_name: shake-server
    build:
      context: ./server
      dockerfile: Dockerfile
    expose:
      - "8080"
    healthcheck:
      test: ["CMD", "nc", "-z", "localhost", "8080"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

  caddy:
    image: caddy:2-alpine
    container_name: shake-caddy
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./caddy/certs:/etc/caddy/certs
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      shake-client:
        condition: service_healthy
      shake-server:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:2019/config/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

volumes:
  caddy_data:
  caddy_config:
```

### 6. Launch

```bash
docker compose up -d
```

Visit `https://192.168.1.100` on any device in your LAN.

---

## 🌐 Public Deployment (Domain + Automatic HTTPS)

### 1. DNS Setup

Point your domain's **A record** to your server's public IP:
```
A    @      YOUR_PUBLIC_IP
A    www    YOUR_PUBLIC_IP
```

### 2. Caddyfile for Public Domain

```caddyfile
shake.yourdomain.com {
    encode zstd gzip

    handle /peer* {
        uri strip_prefix /peer
        reverse_proxy shake-server:8080
    }

    handle /lookup* {
        reverse_proxy shake-server:8080
    }

    handle {
        reverse_proxy shake-client:8081
    }
}
```

**That's it!** Caddy automatically provisions Let's Encrypt certificates.

### 3. Firewall

```bash
# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 443/udp   # For HTTP/3 (optional)
sudo ufw enable
```

### 4. Launch

```bash
docker compose up -d
```

Visit `https://shake.yourdomain.com`

---

## 🔧 Configuration Options

### URL Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `?server=name` | Private lobby name (share with friends) | `?server=myparty` |
| `?lonely` | Enable bots only (no multiplayer) | `?lonely` |
| `?map=q3dm7` | Select map (see list below) | `?map=q3dm17` |

### Available Maps (Demo)

| Map | Name |
|-----|------|
| `q3dm1` | Arena Gate |
| `q3dm7` | Temple of Retribution |
| `q3dm17` | The Longest Yard |
| `q3tourney2` | The Proving Grounds |

### Custom Maps

Place additional `.pk3` files in `demoq3/` and they'll be available.

---

## 🛠️ Advanced Configuration

### Custom Game Settings

Edit `client/index.html` → `generatedArguments` to modify:

```javascript
let generatedArguments = `
+set fs_game demoq3
+set g_gametype 0        # 0=FFA, 1=Tournament, 3=CTF
+set fraglimit 20
+set timelimit 10
+set g_forcerespawn 2
+set sv_maxclients 8
`;
```

### Bot Skill Level

In `client/index.html`:
```javascript
let botSkill = 2;  // 1=I Can Win, 2=Bring It On, 3=Hurt Me Plenty, 4=Hardcore, 5=Nightmare
```

### Persistent Config

Config is stored in browser IndexedDB (`/home/web_user/.q3a/demoq3/q3config.cfg`). Survives container restarts.

---

## 🐛 Troubleshooting

### "Connection Refused" on /peer
- Check `shake-server` logs: `docker compose logs shake-server`
- Ensure Caddy routes `/peer` to `shake-server:8080` (not localhost)

### WASM Fails to Load
- Check browser console for CORS errors
- Ensure nginx serves `.wasm` with `application/wasm` MIME type (add to nginx.conf if needed)

### Certificate Warnings on LAN
- Install mkcert root CA on each device:
  - **Desktop**: `mkcert -install` then trust in browser/OS
  - **iOS**: Email/airdrop `rootCA.pem` → Settings → General → VPN & Device Management → Trust
  - **Android**: Settings → Security → Encryption & Credentials → Install Certificate

### Performance Issues
- Reduce resolution: browser zoom out or modify canvas size in index.html
- Lower bot count: edit `+addbot` lines in index.html
- Enable server-side frame sync: already implemented via `readPixels()` hack

---

## 📊 Monitoring

```bash
# View all logs
docker compose logs -f

# Check health status
docker compose ps

# Resource usage
docker stats
```

---

## 🔄 Updates

```bash
git pull
docker compose pull
docker compose up -d --build
```

---

## 📝 License

Based on [shake](https://github.com/xanderstrike/shake) by XanderStrike, which builds on:
- [ioquake3](https://github.com/jdarpinian/ioq3) by jdarpinian
- [HumbleNet](https://github.com/jdarpinian/HumbleNet) by jdarpinian
