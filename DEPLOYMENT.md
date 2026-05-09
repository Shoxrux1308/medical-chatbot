# 🚀 DEPLOYMENT GUIDE

## Quick Deployment Options

### 1. **Render.com** (Recommended - Free Tier Available)

```bash
# 1. Create account at render.com
# 2. Create new Web Service
# 3. Connect your GitHub repository
# 4. Set Environment Variables:
#    - FLASK_ENV=production
#    - SECRET_KEY=<generate-random-key>
#    - DATABASE_URL=<postgresql-url>
# 5. Deploy automatically on push
```

### 2. **Heroku** (Requires Credit Card)

```bash
# Install Heroku CLI
curl https://cli-assets.heroku.com/install.sh | sh

# Login
heroku login

# Create app
heroku create medsync-app

# Set config vars
heroku config:set FLASK_ENV=production SECRET_KEY=<key> -a medsync-app

# Deploy
git push heroku main

# View logs
heroku logs --tail -a medsync-app
```

### 3. **Railway.app** (Modern Alternative)

```bash
# 1. Login at railway.app
# 2. Create new project
# 3. Connect GitHub
# 4. Auto-detect Flask
# 5. Add PostgreSQL plugin
# 6. Deploy
```

### 4. **DigitalOcean App Platform**

```bash
# 1. Create account at digitalocean.com
# 2. Create new App
# 3. Connect GitHub repository
# 4. Configure as Python/Flask app
# 5. Add PostgreSQL database
# 6. Deploy
```

### 5. **Self-Hosted VPS** (Full Control)

**Option A: Using systemd + Gunicorn + Nginx**

```bash
# SSH into VPS
ssh root@your-vps-ip

# Update system
apt update && apt upgrade -y

# Install dependencies
apt install -y python3 python3-pip python3-venv nginx curl supervisor

# Clone repository
cd /var/www
git clone <your-repo-url> medical-chatbot
cd medical-chatbot

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Python packages
pip install -r requirements.txt
pip install gunicorn

# Create .env file
cat > .env << EOF
FLASK_ENV=production
SECRET_KEY=$(python3 -c 'import secrets; print(secrets.token_hex(32))')
DATABASE_URL=sqlite:///medsync.db
EOF

# Test Gunicorn
gunicorn -w 4 -b 127.0.0.1:5000 app:app

# Create systemd service
sudo cat > /etc/systemd/system/medsync.service << EOF
[Unit]
Description=MedSync Flask Application
After=network.target

[Service]
User=www-data
WorkingDirectory=/var/www/medical-chatbot
Environment="PATH=/var/www/medical-chatbot/venv/bin"
ExecStart=/var/www/medical-chatbot/venv/bin/gunicorn -w 4 -b 127.0.0.1:5000 app:app

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable medsync
sudo systemctl start medsync

# Configure Nginx as reverse proxy
sudo cat > /etc/nginx/sites-available/medsync << EOF
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
    }
}
EOF

# Enable Nginx site
sudo ln -s /etc/nginx/sites-available/medsync /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Setup SSL with Let's Encrypt
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

**Option B: Using Docker**

```bash
# Create Dockerfile
cat > Dockerfile << EOF
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

ENV FLASK_APP=app.py
ENV FLASK_ENV=production

EXPOSE 5000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
EOF

# Create docker-compose.yml
cat > docker-compose.yml << EOF
version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=production
      - SECRET_KEY=your-secret-key
    volumes:
      - ./medsync.db:/app/medsync.db
EOF

# Run
docker-compose up -d
```

## Environment Variables Setup

Create `.env` file in project root:

```env
# Flask Configuration
FLASK_ENV=production
FLASK_DEBUG=False
SECRET_KEY=generate-with-python3-c-'import-secrets;-print(secrets.token_hex(32))'

# Database
DATABASE_URL=sqlite:///medsync.db
# For PostgreSQL: postgresql://user:password@localhost/medsync

# Session
SESSION_TYPE=filesystem
PERMANENT_SESSION_LIFETIME=2592000

# For production, use PostgreSQL instead of SQLite
```

## Database Upgrade (SQLite → PostgreSQL)

```bash
# Install PostgreSQL
pip install psycopg2-binary

# Update .env
DATABASE_URL=postgresql://user:password@host:5432/medsync_db

# Migrate data
python3 << EOF
from app import app, db
from flask_migrate import init, migrate, upgrade

with app.app_context():
    init()
    migrate(message='Initial migration')
    upgrade()
EOF
```

## SSL/HTTPS Setup

### Using Let's Encrypt (Free)

```bash
# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx

# Generate certificate
sudo certbot certonly --nginx -d yourdomain.com

# Auto-renewal
sudo certbot renew --dry-run
```

### Manual renewal
```bash
sudo certbot renew
```

## Monitoring & Maintenance

### Check service status
```bash
systemctl status medsync
```

### View logs
```bash
journalctl -u medsync -f
```

### Restart service
```bash
systemctl restart medsync
```

### Update application
```bash
cd /var/www/medical-chatbot
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
systemctl restart medsync
```

## Performance Optimization

### 1. Enable Caching
```python
# In app.py
from flask_caching import Cache
cache = Cache(app, config={'CACHE_TYPE': 'redis'})
```

### 2. Use CDN for Static Files
```
# Configure Nginx to serve static files
location /static {
    alias /var/www/medical-chatbot/static;
    expires 30d;
}
```

### 3. Database Optimization
```bash
# For PostgreSQL
VACUUM ANALYZE;
```

## Security Checklist

- [ ] Set `SECRET_KEY` to random value
- [ ] Set `FLASK_ENV=production`
- [ ] Use HTTPS/SSL
- [ ] Enable CORS properly
- [ ] Validate all input
- [ ] Use strong passwords
- [ ] Backup database regularly
- [ ] Monitor logs for errors
- [ ] Keep dependencies updated

## Backup Strategy

```bash
# Daily backup script
#!/bin/bash
BACKUP_DIR="/backups/medsync"
DB_FILE="/var/www/medical-chatbot/medsync.db"

mkdir -p $BACKUP_DIR
cp $DB_FILE $BACKUP_DIR/medsync-$(date +%Y%m%d-%H%M%S).db

# Keep only last 30 days
find $BACKUP_DIR -name "*.db" -mtime +30 -delete
```

## Troubleshooting Deployment

**502 Bad Gateway**
```bash
# Check if Gunicorn is running
systemctl status medsync
journalctl -u medsync -f
```

**Port already in use**
```bash
sudo lsof -i :80
sudo kill -9 <PID>
```

**Database locked**
```bash
# Restart service
systemctl restart medsync
```

**Out of memory**
```bash
# Reduce Gunicorn workers
# Edit systemd service, change: -w 4 to -w 2
systemctl daemon-reload
systemctl restart medsync
```

---

**Need Help?** Check README.md or open an issue on GitHub
