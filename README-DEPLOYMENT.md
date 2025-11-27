# 📦 Stream Sanctuary - Deployment Package

## 🎯 What's Included

This deployment package contains everything you need to deploy Stream Sanctuary to production:

### 📄 Configuration Files

| File | Purpose | Required |
|------|---------|----------|
| `package.json` | Node.js dependencies and scripts | ✅ Yes |
| `Dockerfile` | Docker container configuration | For Docker |
| `docker-compose.yml` | Full stack orchestration | For Docker |
| `vercel.json` | Vercel deployment config | For Vercel |
| `netlify.toml` | Netlify deployment config | For Netlify |
| `kubernetes-deployment.yaml` | Kubernetes resources | For K8s |
| `.github-workflows-ci-cd.yml` | GitHub Actions CI/CD | For automation |
| `.env.example` | Environment variables template | ✅ Yes |
| `DEPLOYMENT-GUIDE.md` | Step-by-step instructions | ✅ Yes |

### 🚀 Quick Start Commands

```bash
# 1. Install dependencies
npm install

# 2. Configure environment
cp .env.example .env
nano .env  # Fill in your values

# 3. Choose your deployment method:

# Local Development
npm run dev

# Docker
docker-compose up -d

# Vercel
vercel --prod

# Netlify
netlify deploy --prod

# Kubernetes
kubectl apply -f kubernetes-deployment.yaml
```

---

## 🎨 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT OPTIONS                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Static Hosting (Vercel/Netlify)                       │
│     ✅ Easiest                                             │
│     ✅ Automatic SSL                                       │
│     ✅ Global CDN                                          │
│     ⚠️  Limited backend capabilities                       │
│                                                             │
│  2. Docker Single Container                                │
│     ✅ Quick setup                                         │
│     ✅ Portable                                            │
│     ⚠️  Manual scaling                                     │
│                                                             │
│  3. Docker Compose Stack                                   │
│     ✅ Full stack (app + databases + RTMP)                │
│     ✅ Easy orchestration                                  │
│     ✅ Production-ready                                    │
│                                                             │
│  4. Kubernetes Cluster                                     │
│     ✅ Enterprise-grade                                    │
│     ✅ Auto-scaling                                        │
│     ✅ High availability                                   │
│     ⚠️  More complex setup                                 │
│                                                             │
│  5. Traditional VPS                                        │
│     ✅ Full control                                        │
│     ✅ Cost-effective                                      │
│     ⚠️  Manual maintenance                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 NPM Scripts Reference

### Development
```bash
npm run dev          # Start Vite dev server
npm run serve        # Start production server
npm start            # Alias for npm run serve
```

### Building
```bash
npm run build        # Build production bundle
npm run preview      # Preview production build
npm run analyze      # Analyze bundle size
```

### Testing
```bash
npm test             # Run unit tests with coverage
npm run test:watch   # Run tests in watch mode
npm run test:e2e     # Run Playwright E2E tests
npm run test:a11y    # Run accessibility tests
```

### Code Quality
```bash
npm run lint         # Run ESLint
npm run lint:fix     # Fix ESLint issues
npm run format       # Format code with Prettier
```

### Deployment
```bash
npm run deploy:vercel    # Deploy to Vercel
npm run deploy:netlify   # Deploy to Netlify
npm run deploy:docker    # Deploy with Docker Compose
```

### Docker
```bash
npm run docker:build     # Build Docker image
npm run docker:run       # Run Docker container
```

### PWA
```bash
npm run pwa:generate     # Generate service worker
```

---

## 🌐 Deployment Methods Comparison

| Feature | Vercel | Netlify | Docker | Kubernetes | VPS |
|---------|--------|---------|--------|------------|-----|
| **Difficulty** | ⭐ Easy | ⭐ Easy | ⭐⭐ Medium | ⭐⭐⭐ Hard | ⭐⭐ Medium |
| **Setup Time** | 5 min | 5 min | 15 min | 1 hour | 30 min |
| **Auto SSL** | ✅ Yes | ✅ Yes | ❌ Manual | ✅ Yes* | ❌ Manual |
| **Auto Scale** | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes | ❌ No |
| **CDN** | ✅ Global | ✅ Global | ❌ No | ⚠️ Optional | ❌ No |
| **Databases** | ⚠️ External | ⚠️ External | ✅ Included | ✅ Included | ✅ Self-host |
| **RTMP Server** | ❌ No | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| **Cost (Small)** | $20/mo | $19/mo | $5/mo | $30/mo | $10/mo |
| **Cost (Medium)** | $100/mo | $99/mo | $20/mo | $100/mo | $40/mo |
| **Cost (Large)** | $400/mo | $499/mo | $100/mo | $500/mo | $200/mo |

\*With cert-manager installed

---

## 📊 System Requirements

### Minimum (Development)
- **CPU:** 2 cores
- **RAM:** 4 GB
- **Storage:** 10 GB
- **Network:** 10 Mbps

### Recommended (Production - Small)
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Storage:** 50 GB SSD
- **Network:** 100 Mbps
- **Concurrent Users:** 100-500

### Recommended (Production - Medium)
- **CPU:** 8 cores
- **RAM:** 16 GB
- **Storage:** 200 GB SSD
- **Network:** 1 Gbps
- **Concurrent Users:** 500-2,000

### Recommended (Production - Large)
- **CPU:** 16+ cores
- **RAM:** 32+ GB
- **Storage:** 500+ GB SSD
- **Network:** 10 Gbps
- **Concurrent Users:** 2,000+

---

## 🔐 Security Checklist

Before deploying to production:

- [ ] **Change all default passwords**
- [ ] **Generate strong JWT secret** (32+ characters)
- [ ] **Configure HTTPS/SSL** (Let's Encrypt or Cloudflare)
- [ ] **Set secure CORS origins** (not wildcard)
- [ ] **Enable rate limiting**
- [ ] **Configure CSP headers**
- [ ] **Set up firewall rules**
- [ ] **Enable 2FA for admin accounts**
- [ ] **Configure backup strategy**
- [ ] **Set up monitoring/alerts**
- [ ] **Review all environment variables**
- [ ] **Audit npm dependencies** (`npm audit`)
- [ ] **Configure DDoS protection**
- [ ] **Set up log rotation**
- [ ] **Test disaster recovery**

---

## 🚨 Environment Variables Priority

### Critical (Must Configure)
1. `JWT_SECRET` - Authentication security
2. `POSTGRES_PASSWORD` - Database security
3. `REDIS_PASSWORD` - Cache security
4. `MONGO_PASSWORD` - Analytics security
5. `BASE_URL` - Application URL

### High Priority (Strongly Recommended)
6. `SENTRY_DSN` - Error tracking
7. `SMTP_*` - Email notifications
8. `CLOUDFLARE_*` - CDN/DDoS protection

### Medium Priority (Feature-Dependent)
9. `ETH_RPC_URL` - Web3 features
10. `OPENAI_API_KEY` - AI features
11. `IPFS_*` - Decentralized storage
12. `TWITCH_*` / `YOUTUBE_*` - Platform integrations

### Low Priority (Optional)
13. `GA_TRACKING_ID` - Analytics
14. `MIXPANEL_TOKEN` - User analytics
15. `SLACK_WEBHOOK_URL` - Team notifications

---

## 📈 Monitoring Endpoints

Once deployed, monitor these endpoints:

```bash
# Health Check
curl https://your-domain.com/health

# Metrics (if Prometheus enabled)
curl https://your-domain.com/metrics

# Version Info
curl https://your-domain.com/api/version
```

---

## 🔄 Update Strategy

### Rolling Update (Zero Downtime)

**Docker Compose:**
```bash
# Pull new image
docker-compose pull

# Update services one by one
docker-compose up -d --no-deps --build app

# Verify health
curl https://your-domain.com/health
```

**Kubernetes:**
```bash
# Update image
kubectl set image deployment/stream-sanctuary-app \
  app=stream-sanctuary:v2.0.0 \
  -n stream-sanctuary

# Watch rollout
kubectl rollout status deployment/stream-sanctuary-app -n stream-sanctuary

# Rollback if needed
kubectl rollout undo deployment/stream-sanctuary-app -n stream-sanctuary
```

**PM2:**
```bash
# Pull changes
git pull origin main

# Install dependencies
npm install

# Build
npm run build

# Reload without downtime
pm2 reload stream-sanctuary
```

---

## 📞 Support & Resources

### Documentation
- **Full Docs:** [DEPLOYMENT-GUIDE.md](./DEPLOYMENT-GUIDE.md)
- **API Docs:** [API.md](./API.md)
- **Architecture:** [multi-source-architecture.md](./multi-source-architecture.md)

### Community
- **Discord:** https://discord.gg/stream-sanctuary
- **GitHub Discussions:** https://github.com/stream-sanctuary/platform/discussions
- **Stack Overflow:** Tag `stream-sanctuary`

### Professional Support
- **Email:** support@stream-sanctuary.app
- **Enterprise:** enterprise@stream-sanctuary.app

---

## 🎯 Common Deployment Scenarios

### Scenario 1: Solo Streamer (Hobby)
**Recommendation:** Vercel or Netlify
**Cost:** $0-20/month
**Setup Time:** 10 minutes
```bash
vercel --prod
```

### Scenario 2: Small Streamer (Semi-Pro)
**Recommendation:** VPS with Docker Compose
**Cost:** $10-40/month
**Setup Time:** 1 hour
```bash
docker-compose up -d
```

### Scenario 3: Team/Agency (Multiple Streamers)
**Recommendation:** Kubernetes on managed cluster
**Cost:** $100-500/month
**Setup Time:** 4 hours
```bash
kubectl apply -f kubernetes-deployment.yaml
```

### Scenario 4: Enterprise/Platform
**Recommendation:** Multi-region Kubernetes
**Cost:** $1,000+/month
**Setup Time:** 1-2 days
- Custom architecture design
- Professional support recommended

---

## 🧪 Testing Deployment

### Local Testing
```bash
# Build production bundle
npm run build

# Test production build locally
npm run preview

# Run all tests
npm test
npm run test:e2e
npm run test:a11y
```

### Pre-Production Testing
```bash
# Deploy to staging
vercel --prod --env=staging

# Run smoke tests
curl https://staging.your-domain.com/health

# Load test
artillery quick --count 100 -n 20 https://staging.your-domain.com
```

---

## ✅ Deployment Verification

After deployment, verify:

1. **Application Loads:** Visit your domain
2. **Health Check:** `curl https://your-domain.com/health`
3. **SSL Valid:** Check certificate in browser
4. **Performance:** Lighthouse score >90
5. **Accessibility:** WCAG AA minimum
6. **Database Connected:** Check logs
7. **Redis Connected:** Check logs
8. **RTMP Working:** Test stream
9. **Monitoring Active:** Check dashboard
10. **Backups Scheduled:** Verify cron

---

## 🎓 Learning Resources

### Video Tutorials
- **Full Deployment Guide:** [YouTube Playlist]
- **Docker Basics:** [Docker Official Docs]
- **Kubernetes Crash Course:** [Kubernetes.io]

### Reading
- **12-Factor App:** https://12factor.net
- **Web Performance:** https://web.dev/performance
- **Security Best Practices:** https://cheatsheetseries.owasp.org

---

## 📝 License

MIT License with Non-Weaponization Clause

Copyright (c) 2025 Stream Sanctuary

See [LICENSE](./LICENSE) for full terms.

---

## 🙏 Acknowledgments

Built with:
- Node.js, Express, WebRTC
- PostgreSQL, Redis, MongoDB
- Docker, Kubernetes
- Vite, Playwright, Jest
- And many more amazing open-source projects

---

**Ready to deploy? Choose your method above and follow the guide!** 🚀

For detailed instructions, see [DEPLOYMENT-GUIDE.md](./DEPLOYMENT-GUIDE.md)