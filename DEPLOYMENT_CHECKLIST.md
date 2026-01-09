# 🚀 Deployment Checklist

Use this checklist to deploy your Enetix Portfolio application to production.

---

## ☑️ Pre-Deployment Checklist

### 1. Get API Keys

- [ ] **Infura** - Ethereum RPC ([infura.io](https://infura.io))
- [ ] **Alchemy** - Multi-chain RPC ([alchemy.com](https://alchemy.com))
- [ ] **Helius** - Solana RPC ([helius.xyz](https://helius.xyz))
- [ ] **BlockCypher** - Bitcoin API ([blockcypher.com](https://blockcypher.com))
- [ ] **Sentry** - Error tracking ([sentry.io](https://sentry.io))
- [ ] **The Graph** - DeFi data (optional but recommended) ([thegraph.com/studio](https://thegraph.com/studio))

### 2. Create Accounts

- [ ] **Railway** - Backend hosting ([railway.app](https://railway.app))
- [ ] **Vercel** or **Netlify** - Frontend web hosting
- [ ] **Expo** - Mobile app builds ([expo.dev](https://expo.dev))

### 3. Security Tokens

Generate secure secrets:
```bash
# JWT Secret
openssl rand -base64 32

# JWT Refresh Secret
openssl rand -base64 32
```

---

## 🗄️ Step 1: Database Setup (Railway)

### A. Create PostgreSQL Database

1. Go to [railway.app](https://railway.app)
2. Click "New Project" → "Provision PostgreSQL"
3. Copy the `DATABASE_URL` from the database settings
4. Save it securely (you'll need it for the backend)

### B. Test Connection (Optional)

```bash
# Install PostgreSQL client
brew install postgresql  # macOS
# or
sudo apt install postgresql-client  # Linux

# Test connection
psql "postgresql://user:pass@host:port/dbname"
```

---

## 🖥️ Step 2: Backend Deployment (Railway)

### A. Initialize Git (if not done)

```bash
cd /Users/mattgreene/dev/enetix/repo/enetix-portfolio-backend
git init
git add .
git commit -m "Initial backend deployment"
```

### B. Push to GitHub

```bash
# Create repo on GitHub first
git remote add origin https://github.com/YOUR_USERNAME/enetix-portfolio-backend.git
git branch -M main
git push -u origin main
```

### C. Deploy on Railway

1. Go to [railway.app/new](https://railway.app/new)
2. Click "Deploy from GitHub repo"
3. Select `enetix-portfolio-backend`
4. Railway will auto-detect and deploy

### D. Set Environment Variables

In Railway dashboard, go to Variables and add:

```bash
DATABASE_URL=<from step 1 - Railway provides this automatically>
NODE_ENV=production
PORT=5000

# JWT Secrets (generate with: openssl rand -base64 32)
JWT_SECRET=<your-generated-secret>
JWT_REFRESH_SECRET=<your-generated-refresh-secret>

# Blockchain APIs
INFURA_API_KEY=<your-infura-key>
ALCHEMY_API_KEY=<your-alchemy-key>
HELIUS_API_KEY=<your-helius-key>
BLOCKCYPHER_API_KEY=<your-blockcypher-key>

# RPC URLs (optional - will use defaults if not set)
POLYGON_RPC_URL=https://polygon-rpc.com
ARBITRUM_RPC_URL=https://arb1.arbitrum.io/rpc
BASE_RPC_URL=https://mainnet.base.org
OPTIMISM_RPC_URL=https://mainnet.optimism.io
AVALANCHE_RPC_URL=https://api.avax.network/ext/bc/C/rpc
BNB_RPC_URL=https://bsc-dataseed.binance.org
FANTOM_RPC_URL=https://rpc.ftm.tools

# Monitoring
SENTRY_DSN=<your-sentry-dsn>

# Optional: The Graph API Key (recommended for production)
GRAPH_API_KEY=<your-graph-api-key>

# CORS (add your frontend domains)
ALLOWED_ORIGINS=https://your-app.vercel.app,https://your-app.netlify.app
```

### E. Run Database Migrations

In Railway dashboard:

1. Go to your backend service
2. Click "Settings" → "Deploy"
3. Add a deployment command or run manually via Railway CLI:

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Link to your project
railway link

# Run migrations
railway run npm run db:push
```

### F. Verify Deployment

```bash
# Test health endpoint
curl https://your-backend.railway.app/api/health

# Expected response:
# {"status":"ok","timestamp":1234567890}
```

---

## 🌐 Step 3: Frontend Web Deployment (Vercel)

### A. Set Up Environment Variables

Create `.env` file in `/enetix-portfolio-frontend`:

```bash
EXPO_PUBLIC_API_URL=https://your-backend.railway.app/api
EXPO_PUBLIC_SENTRY_DSN=<your-sentry-dsn>
```

### B. Build Web Version

```bash
cd /Users/mattgreene/dev/enetix/repo/enetix-portfolio-frontend
npm run build:web
```

### C. Deploy to Vercel

#### Option 1: Vercel CLI

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
cd web-build
vercel --prod
```

#### Option 2: Vercel Dashboard

1. Go to [vercel.com/new](https://vercel.com/new)
2. Import your `enetix-portfolio-frontend` repo
3. Set build command: `npm run build:web`
4. Set output directory: `web-build`
5. Add environment variables:
   - `EXPO_PUBLIC_API_URL`
   - `EXPO_PUBLIC_SENTRY_DSN`
6. Deploy

#### Option 3: Netlify

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Deploy
cd web-build
netlify deploy --prod
```

### D. Configure Custom Domain (Optional)

1. In Vercel/Netlify dashboard, go to Domains
2. Add your custom domain
3. Update DNS records as instructed
4. Enable SSL (automatic)

### E. Update Backend CORS

Add your frontend domain to Railway backend environment variables:

```bash
ALLOWED_ORIGINS=https://your-app.vercel.app
```

---

## 📱 Step 4: Mobile App Deployment (iOS & Android)

### A. Set Up EAS

```bash
cd /Users/mattgreene/dev/enetix/repo/enetix-portfolio-frontend

# Install EAS CLI globally
npm install -g eas-cli

# Login to Expo
eas login

# Configure EAS for your project
eas build:configure
```

### B. Configure app.json

Update `app.json` with production settings:

```json
{
  "expo": {
    "name": "Enetix Portfolio",
    "slug": "enetix-portfolio",
    "version": "1.0.0",
    "ios": {
      "bundleIdentifier": "com.enetix.portfolio",
      "buildNumber": "1.0.0"
    },
    "android": {
      "package": "com.enetix.portfolio",
      "versionCode": 1
    },
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

### C. Create EAS Configuration

Create `eas.json`:

```json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://your-backend.railway.app/api"
      }
    },
    "production": {
      "env": {
        "EXPO_PUBLIC_API_URL": "https://your-backend.railway.app/api",
        "EXPO_PUBLIC_SENTRY_DSN": "your-sentry-dsn"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

### D. Build for iOS

```bash
# Build iOS app
eas build --platform ios --profile production

# This will:
# 1. Create a build in the cloud
# 2. Generate an IPA file
# 3. Provide a link to download
```

### E. Build for Android

```bash
# Build Android app
eas build --platform android --profile production

# This will:
# 1. Create a build in the cloud
# 2. Generate an AAB or APK file
# 3. Provide a link to download
```

### F. Submit to App Stores

#### iOS App Store

```bash
# Submit to App Store
eas submit --platform ios --latest

# You'll need:
# - Apple Developer Account ($99/year)
# - App Store Connect account
# - Bundle ID registered
# - Certificates and provisioning profiles
```

#### Google Play Store

```bash
# Submit to Play Store
eas submit --platform android --latest

# You'll need:
# - Google Play Developer Account ($25 one-time)
# - Signed APK/AAB
# - App listing details
```

---

## ✅ Step 5: Post-Deployment Verification

### A. Test Backend Endpoints

```bash
# Health check
curl https://your-backend.railway.app/api/health

# Register user
curl -X POST https://your-backend.railway.app/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123"}'

# Login
curl -X POST https://your-backend.railway.app/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123"}'
```

### B. Test Frontend

1. Open web app in browser
2. Register a new account
3. Add a wallet address
4. Verify portfolio data loads
5. Check analytics charts
6. Test price alerts
7. Verify settings sync

### C. Test Mobile Apps

1. Install on test device
2. Complete same flow as web
3. Test offline mode
4. Test push notifications (if enabled)
5. Test app refresh
6. Verify data sync across devices

---

## 🔍 Step 6: Monitoring Setup

### A. Configure Sentry

1. Go to [sentry.io](https://sentry.io)
2. Create project for backend
3. Create project for frontend
4. Copy DSN for each
5. Add to environment variables (already done in step 2 & 3)

### B. Set Up Alerts

In Sentry dashboard:
- [ ] Set up email alerts for errors
- [ ] Configure Slack/Discord notifications
- [ ] Set error thresholds
- [ ] Create uptime monitors

### C. Database Monitoring

In Railway dashboard:
- [ ] Set up database backups (automatic daily)
- [ ] Monitor database size
- [ ] Check connection pool usage
- [ ] Set up alerts for high usage

---

## 🔒 Step 7: Security Checklist

- [ ] All API keys in environment variables (not hardcoded)
- [ ] Database credentials secure (Railway managed)
- [ ] HTTPS enabled on all endpoints (automatic)
- [ ] CORS properly configured
- [ ] Rate limiting active
- [ ] JWT secrets strong (32+ characters)
- [ ] Password hashing with bcrypt (12 rounds)
- [ ] SQL injection protection (using ORM)
- [ ] Input validation on all endpoints (Zod)
- [ ] Error messages don't expose sensitive info
- [ ] Sentry tracking enabled

---

## 📊 Step 8: Performance Optimization

### A. Backend Optimization

- [ ] Database indexes created (automatic via Drizzle)
- [ ] Caching enabled (5min portfolio, 1min prices)
- [ ] Background jobs running (hourly snapshots, 5min alerts)
- [ ] Connection pooling configured (max 20)

### B. Frontend Optimization

- [ ] Web build minified and compressed
- [ ] Images optimized
- [ ] Code splitting enabled
- [ ] Lazy loading implemented
- [ ] Service worker for offline (if using PWA)

---

## 📈 Step 9: Analytics Setup (Optional)

### Google Analytics

1. Create GA4 property
2. Add tracking ID to app
3. Track key events:
   - User registration
   - Wallet added
   - Portfolio viewed
   - Alert created
   - Tax report generated

### Mixpanel/Amplitude

Similar setup for detailed user analytics.

---

## 🎉 Step 10: Launch!

### Pre-Launch Checklist

- [ ] All tests passing
- [ ] Backend health check returns 200
- [ ] Frontend loads successfully
- [ ] Mobile apps approved by stores
- [ ] Monitoring active
- [ ] Backups configured
- [ ] Documentation complete
- [ ] Team trained (if applicable)
- [ ] Support email/chat ready

### Launch Day

1. Announce on social media
2. Send email to beta testers
3. Monitor error rates closely
4. Watch performance metrics
5. Be ready for quick fixes

### Post-Launch (Week 1)

- [ ] Monitor user signups
- [ ] Track error rates (target: <0.1%)
- [ ] Check API response times (target: <2s)
- [ ] Review user feedback
- [ ] Fix critical bugs ASAP
- [ ] Optimize based on real usage

---

## 🆘 Troubleshooting

### Backend won't start

```bash
# Check logs in Railway dashboard
railway logs

# Common issues:
# - Missing environment variables
# - Database connection failed
# - Port already in use
```

### Database migration failed

```bash
# Reset and re-run
railway run npm run db:push

# If that fails, check:
# - DATABASE_URL is correct
# - Database is running
# - Sufficient permissions
```

### Frontend can't connect to backend

```bash
# Check:
# 1. EXPO_PUBLIC_API_URL is correct
# 2. Backend is running (health check)
# 3. CORS allows your frontend domain
# 4. No typos in API calls
```

### Mobile build failed

```bash
# Check EAS build logs
eas build:list

# Common issues:
# - Bundle ID conflicts
# - Certificate issues
# - Memory limits
# - Build timeout
```

---

## 📞 Support Resources

- **Railway**: [docs.railway.app](https://docs.railway.app)
- **Expo**: [docs.expo.dev](https://docs.expo.dev)
- **Vercel**: [vercel.com/docs](https://vercel.com/docs)
- **Sentry**: [docs.sentry.io](https://docs.sentry.io)
- **Drizzle ORM**: [orm.drizzle.team](https://orm.drizzle.team)

---

## ✨ Congratulations!

Your Enetix Portfolio application is now live! 🎉

**Next steps:**
- Gather user feedback
- Monitor performance
- Plan feature updates
- Scale as needed

**Remember:**
- Keep API keys secure
- Monitor error rates
- Back up database regularly
- Update dependencies
- Stay compliant with regulations

---

**Deployment completed!** ✅

Total estimated deployment time: 2-4 hours (depending on app store approval)
