# 🚀 Stream Sanctuary - Complete Deployment Guide

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start (Local Development)](#quick-start-local-development)
3. [Static Hosting (Vercel/Netlify)](#static-hosting)
4. [Docker Deployment](#docker-deployment)
5. [Kubernetes Deployment](#kubernetes-deployment)
6. [Traditional VPS](#traditional-vps)
7. [Environment Variables](#environment-variables)
8. [SSL/TLS Configuration](#ssltls-configuration)
9. [Monitoring & Maintenance](#monitoring--maintenance)
10. [Troubleshooting](#troubleshooting)

---

## 🔧 Prerequisites

### Required Software
- **Node.js** 18+ and npm 9+
- **Git** (for version control)
- **Docker** (for containerized deployment)
- **kubectl** (for Kubernetes deployment)

### Required Accounts (choose based on deployment method)
- GitHub account (for CI/CD)
- Vercel/Netlify account (for static hosting)
- Cloud provider account (AWS/GCP/Azure/DigitalOcean)
- Domain name (optional but recommended)

### Recommended Tools
- **VS Code** with extensions: ESLint, Prettier
- **Postman** or **curl** for API testing
- **Docker Desktop** for local testing
- **k9s** for Kubernetes cluster management

---

## 🏃 Quick Start (Local Development)

### Step 1: Clone Repository
```bash
git clone https://github.com/your-org/stream-sanctuary.git
cd stream-sanctuary
```

### Step 2: Install Dependencies
```bash
npm install
```

### Step 3: Configure Environment
```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your values
nano .env  # or use your preferred editor
```

### Step 4: Start Development Server
```bash
npm run dev
```

The application will be available at `http://localhost:3000`

### Step 5: Run Tests
```bash
# Unit tests
npm test

# E2E tests
npm run test:e2e

# Accessibility tests
npm run test:a11y
```

---

## 🌐 Static Hosting

### Option 1: Vercel (Recommended for simplicity)

#### Prerequisites
- Vercel account
- GitHub repository linked to Vercel

#### Deployment Steps

**1. Install Vercel CLI**
```bash
npm install -g vercel
```

**2. Login to Vercel**
```bash
vercel login
```

**3. Configure Project**
```bash
# Create vercel.json if not exists (already provided)
# Set environment variables in Vercel dashboard
```

**4. Deploy**
```bash
# Preview deployment
vercel

# Production deployment
vercel --prod
```

**5. Configure Environment Variables**
Go to Vercel Dashboard → Your Project → Settings → Environment Variables

Add all variables from `.env.example`

**6. Configure Custom Domain**
Vercel Dashboard → Your Project → Settings → Domains
Add your custom domain and configure DNS

**Automatic Deployments**
- Push to `main` branch → Automatic production deployment
- Push to other branches → Preview deployments
- Pull requests → Preview deployments with unique URLs

---

### Option 2: Netlify

#### Deployment Steps

**1. Install Netlify CLI**
```bash
npm install -g netlify-cli
```

**2. Login to Netlify**
```bash
netlify login
```

**3. Initialize Project**
```bash
netlify init
```

**4. Configure Build Settings**
```bash
# netlify.toml is already configured
# Verify build command: npm run build
# Verify publish directory: dist
```

**5. Deploy**
```bash
# Draft deployment
netlify deploy

# Production deployment
netlify deploy --prod
```

**6. Configure Environment Variables**
Netlify Dashboard → Site Settings → Build & Deploy → Environment

**Continuous Deployment**
- Connect GitHub repository in Netlify
- Automatic deployments on push to main

---

## 🐳 Docker Deployment

### Single Container (Development)

**1. Build Image**
```bash
docker build -t stream-sanctuary:latest .
```

**2. Run Container**
```bash
docker run -d \
  -p 3000:3000 \
  --name stream-sanctuary \
  --env-file .env \
  stream-sanctuary:latest
```

**3. View Logs**
```bash
docker logs -f stream-sanctuary
```

**4. Stop Container**
```bash
docker stop stream-sanctuary
docker rm stream-sanctuary
```

---

### Docker Compose (Production Stack)

**1. Configure Environment**
```bash
# Create .env file with all variables
cp .env.example .env
nano .env
```

**2. Start All Services**
```bash
docker-compose up -d
```

This starts:
- Application (3 replicas with load balancing)
- PostgreSQL database
- Redis cache
- MongoDB analytics
- RTMP media server
- Nginx reverse proxy

**3. Check Status**
```bash
docker-compose ps
```

**4. View Logs**
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f app
```

**5. Scale Services**
```bash
# Scale app to 5 replicas
docker-compose up -d --scale app=5
```

**6. Stop All Services**
```bash
docker-compose down
```

**7. Stop and Remove Volumes**
```bash
docker-compose down -v
```

---

## ☸️ Kubernetes Deployment

### Prerequisites
- Kubernetes cluster (EKS, GKE, AKS, or self-hosted)
- kubectl configured
- Docker image pushed to registry

### Step 1: Build and Push Image

```bash
# Build for multiple architectures
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/your-org/stream-sanctuary:latest \
  --push .
```

### Step 2: Create Secrets

```bash
# Create namespace
kubectl create namespace stream-sanctuary

# Create secrets from .env file
kubectl create secret generic stream-sanctuary-secrets \
  --from-env-file=.env \
  --namespace=stream-sanctuary
```

### Step 3: Deploy Application

```bash
# Apply all resources
kubectl apply -f kubernetes-deployment.yaml

# Verify deployment
kubectl get pods -n stream-sanctuary
kubectl get services -n stream-sanctuary
```

### Step 4: Configure Ingress

**Install Ingress Controller (if not already installed)**
```bash
# Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

**Install Cert-Manager (for SSL)**
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

**Configure Let's Encrypt**
```yaml
# letsencrypt-prod.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@stream-sanctuary.app
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

```bash
kubectl apply -f letsencrypt-prod.yaml
```

### Step 5: Monitor Deployment

```bash
# Watch deployment rollout
kubectl rollout status deployment/stream-sanctuary-app -n stream-sanctuary

# Check logs
kubectl logs -f deployment/stream-sanctuary-app -n stream-sanctuary

# Get service external IP
kubectl get service stream-sanctuary-app -n stream-sanctuary
```

### Step 6: Configure DNS

Point your domain to the external IP from the LoadBalancer service:
```bash
A    stream-sanctuary.app    → 203.0.113.1
CNAME www.stream-sanctuary.app → stream-sanctuary.app
```

### Step 7: Scale Deployment

```bash
# Manual scaling
kubectl scale deployment stream-sanctuary-app --replicas=10 -n stream-sanctuary

# Autoscaling is configured in kubernetes-deployment.yaml
# HPA will automatically scale based on CPU/memory
```

---

## 🖥️ Traditional VPS

### Option 1: Ubuntu 22.04 LTS

**1. Update System**
```bash
sudo apt update && sudo apt upgrade -y
```

**2. Install Node.js**
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

**3. Install PM2 Process Manager**
```bash
sudo npm install -g pm2
```

**4. Clone Application**
```bash
cd /var/www
sudo git clone https://github.com/your-org/stream-sanctuary.git
cd stream-sanctuary
```

**5. Install Dependencies**
```bash
sudo npm install --production
```

**6. Configure Environment**
```bash
sudo cp .env.example .env
sudo nano .env
```

**7. Build Application**
```bash
sudo npm run build
```

**8. Start with PM2**
```bash
pm2 start src/server/index.js --name stream-sanctuary
pm2 startup
pm2 save
```

**9. Configure Nginx Reverse Proxy**
```bash
sudo apt install nginx

sudo nano /etc/nginx/sites-available/stream-sanctuary
```

Add configuration:
```nginx
server {
    listen 80;
    server_name stream-sanctuary.app www.stream-sanctuary.app;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable site:
```bash
sudo ln -s /etc/nginx/sites-available/stream-sanctuary /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**10. Install SSL Certificate**
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d stream-sanctuary.app -d www.stream-sanctuary.app
```

**11. Monitor Application**
```bash
# View logs
pm2 logs stream-sanctuary

# Monitor resources
pm2 monit

# Restart application
pm2 restart stream-sanctuary
```

---

## 🔐 Environment Variables

### Critical Variables (Must Configure)

```bash
# Application
NODE_ENV=production
PORT=3000
BASE_URL=https://your-domain.com

# Database
POSTGRES_PASSWORD=strong_random_password
REDIS_PASSWORD=strong_random_password
MONGO_PASSWORD=strong_random_password

# Security
JWT_SECRET=minimum_32_character_random_string
SESSION_SECRET=another_random_string
ENCRYPTION_KEY=32_character_key_for_e2e_encryption
```

### Optional Variables (Feature-Dependent)

```bash
# Web3 (if using NFT/crypto features)
ETH_RPC_URL=https://mainnet.infura.io/v3/YOUR_KEY
IPFS_PROJECT_ID=your_infura_ipfs_id
IPFS_PROJECT_SECRET=your_secret

# AI (if using transcription/analysis)
OPENAI_API_KEY=sk-your-key

# Analytics
SENTRY_DSN=https://your-sentry-dsn
GA_TRACKING_ID=UA-XXXXXXX-X
```

### Generating Secure Random Values

```bash
# Generate random secrets
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Or use openssl
openssl rand -base64 32
```

---

## 🔒 SSL/TLS Configuration

### Option 1: Let's Encrypt (Recommended - Free)

**For Nginx:**
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

**For Apache:**
```bash
sudo certbot --apache -d yourdomain.com -d www.yourdomain.com
```

**Auto-renewal:**
```bash
# Test renewal
sudo certbot renew --dry-run

# Cron job (already configured by certbot)
sudo certbot renew --quiet
```

### Option 2: Cloudflare (Free SSL + CDN)

1. Add domain to Cloudflare
2. Update nameservers at your registrar
3. Enable "Full (strict)" SSL mode
4. Origin certificate: Cloudflare dashboard → SSL/TLS → Origin Server

---

## 📊 Monitoring & Maintenance

### Health Checks

**Application Health Endpoint:**
```bash
curl https://your-domain.com/health
```

Expected response:
```json
{
  "status": "ok",
  "uptime": 123456,
  "database": "connected",
  "redis": "connected"
}
```

### Logging

**PM2 Logs:**
```bash
pm2 logs stream-sanctuary
```

**Docker Logs:**
```bash
docker logs -f stream-sanctuary
```

**Kubernetes Logs:**
```bash
kubectl logs -f deployment/stream-sanctuary-app -n stream-sanctuary
```

### Monitoring Tools

**Install Monitoring Stack:**
```bash
# Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

**Access Grafana:**
```bash
kubectl port-forward svc/prometheus-grafana 3001:80 -n monitoring
# Visit http://localhost:3001
# Default credentials: admin / prom-operator
```

### Backup Strategy

**Database Backups:**
```bash
# PostgreSQL
pg_dump -U stream_user stream_sanctuary > backup_$(date +%Y%m%d).sql

# MongoDB
mongodump --db stream_sanctuary --out /backups/$(date +%Y%m%d)

# Redis
redis-cli --rdb /backups/dump_$(date +%Y%m%d).rdb
```

**Automated Backups (Cron):**
```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /path/to/backup-script.sh
```

---

## 🔧 Troubleshooting

### Common Issues

**1. Port Already in Use**
```bash
# Find process using port 3000
sudo lsof -i :3000

# Kill process
sudo kill -9 <PID>
```

**2. Permission Denied**
```bash
# Fix file permissions
sudo chown -R $USER:$USER /var/www/stream-sanctuary
```

**3. Out of Memory**
```bash
# Check memory usage
free -h

# Increase Node.js memory limit
node --max-old-space-size=4096 src/server/index.js
```

**4. Database Connection Failed**
```bash
# Check database status
sudo systemctl status postgresql

# Restart database
sudo systemctl restart postgresql

# Check connection
psql -U stream_user -d stream_sanctuary -h localhost
```

**5. SSL Certificate Issues**
```bash
# Renew certificate
sudo certbot renew --force-renewal

# Check certificate expiry
sudo certbot certificates
```

**6. High CPU Usage**
```bash
# Check process CPU
top
# or
htop

# Restart application
pm2 restart stream-sanctuary
```

### Performance Optimization

**1. Enable Compression**
Already configured in nginx/express

**2. Enable HTTP/2**
```nginx
# In nginx config
listen 443 ssl http2;
```

**3. Enable Brotli Compression**
```bash
# Install nginx brotli module
sudo apt install libnginx-mod-http-brotli
```

**4. Optimize Database**
```sql
-- PostgreSQL
VACUUM ANALYZE;
REINDEX DATABASE stream_sanctuary;

-- Create indexes
CREATE INDEX idx_streams_user ON streams(user_id);
CREATE INDEX idx_streams_created ON streams(created_at);
```

**5. Redis Memory Optimization**
```bash
# redis.conf
maxmemory 256mb
maxmemory-policy allkeys-lru
```

---

## 📈 Scaling Strategies

### Horizontal Scaling (Multiple Instances)

**Docker Compose:**
```bash
docker-compose up -d --scale app=5
```

**Kubernetes:**
```bash
kubectl scale deployment stream-sanctuary-app --replicas=10
```

**PM2 Cluster Mode:**
```bash
pm2 start src/server/index.js -i max
```

### Load Balancing

**Nginx Load Balancer:**
```nginx
upstream stream_sanctuary {
    least_conn;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    location / {
        proxy_pass http://stream_sanctuary;
    }
}
```

### CDN Integration

**Cloudflare:**
1. Add domain to Cloudflare
2. Enable caching
3. Configure cache rules
4. Enable minification

**AWS CloudFront:**
```bash
aws cloudfront create-distribution \
  --origin-domain-name stream-sanctuary.app \
  --default-root-object index.html
```

---

## ✅ Post-Deployment Checklist

- [ ] SSL certificate installed and auto-renewal configured
- [ ] Environment variables set correctly
- [ ] Database backups configured
- [ ] Monitoring and alerts set up
- [ ] DNS records properly configured
- [ ] Firewall rules configured
- [ ] Health checks passing
- [ ] Load testing completed
- [ ] Security audit performed
- [ ] Documentation updated
- [ ] Team trained on deployment process

---

## 📞 Support

- **Documentation:** https://docs.stream-sanctuary.app
- **Discord:** https://discord.gg/stream-sanctuary
- **GitHub Issues:** https://github.com/stream-sanctuary/platform/issues
- **Email:** support@stream-sanctuary.app

---

**Last Updated:** November 2025
**Version:** 2.0.0