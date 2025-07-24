
# n8n Installation with Docker, Nginx, and HTTPS (Let's Encrypt)

## 1. Update System
```sh
sudo apt update && sudo apt upgrade -y
```

## 2. Install Docker and Docker Compose
```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
# Log out and back in to apply Docker group changes
sudo apt install -y docker-compose
```

## 3. Set Up n8n Project Directory
```sh
mkdir -p ~/n8n
cd ~/n8n
```

## 4. Create docker-compose.yml
Paste this into `~/n8n/docker-compose.yml` (update credentials as needed):
```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=admin
      - N8N_HOST=your.domain.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - N8N_EDITOR_BASE_URL=https://your.domain.com/
      - WEBHOOK_URL=https://your.domain.com/
      - N8N_SECURE_COOKIE=true
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

## 5. Start n8n
```sh
sudo docker compose up -d
```

## 6. Set Up DNS for Your Domain
- Point `your.domain.com` to your VM's public IP address (A record).

## 7. Install Nginx and Certbot
```sh
sudo apt install -y nginx certbot python3-certbot-nginx
```

## 8. Configure Nginx as a Reverse Proxy
Create the config file:
```sh
sudo nano /etc/nginx/sites-available/n8n
```
Paste this config (replace domain if needed):
```
server {
    listen 80;
    server_name your.domain.com;

    location / {
        proxy_pass http://localhost:5678/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Enable the config and reload Nginx:
```sh
sudo ln -sf /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

## 9. Obtain SSL Certificate with Let's Encrypt
```sh
sudo certbot --nginx -d your.domain.com
```
Follow the prompts to enable HTTPS.

## 10. Access n8n via Your Custom Domain
- Open your browser and go to: https://your.domain.com
- Log in with your credentials.

---

## Troubleshooting

### If you see the nginx default page instead of n8n:
1. Double-check `/etc/nginx/sites-available/n8n` for correct proxy settings.
2. Ensure the config is enabled:
   ```sh
   sudo ln -sf /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```
3. Remove the default config if needed:
   ```sh
   sudo rm -f /etc/nginx/sites-enabled/default
   sudo systemctl reload nginx
   ```
4. Restart nginx:
   ```sh
   sudo systemctl restart nginx
   ```
5. Make sure n8n is running:
   ```sh
   sudo docker compose ps
   ```
6. Make sure DNS is correct and ports 80/443 are open (check firewall/UFW).

### If you see a secure cookie error:
- Make sure you are accessing n8n via HTTPS and not HTTP.
- Ensure the environment variables in `docker-compose.yml` use your domain, not localhost.

---
