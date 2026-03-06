# PingMyDB GCP Deployment Guide (with Upstash Redis)

## 📋 Prerequisites

1. **Google Cloud Account** with billing enabled
2. **gcloud CLI** installed ✅ (Already done!)
3. **Docker** installed ✅ (Already done!)
4. **Upstash Account** (Free tier available - https://upstash.com)
5. **Project ID** from GCP Console

---

## 🚀 Quick Deployment Steps

### 1. Set Up Upstash Redis (5 minutes)

1. **Go to:** https://console.upstash.com/
2. **Sign up/Login** (free tier available)
3. **Create Redis Database:**
   - Click "Create Database"
   - Name: `pingmydb-redis`
   - Type: `Regional`
   - Region: Choose closest to `us-central1` (e.g., `us-east-1`)
   - Click "Create"
4. **Copy Connection Details:**
   - Click on your database
   - Copy `UPSTASH_REDIS_REST_URL`
   - Copy `UPSTASH_REDIS_REST_TOKEN`
   - **OR** copy the Redis URL (format: `redis://default:password@host:port`)

**Why Upstash?**
- ✅ **Free tier**: 10,000 commands/day
- ✅ **Serverless**: Pay only for what you use
- ✅ **No setup**: Instant provisioning
- ✅ **Global**: Low latency worldwide
- ✅ **Cost**: ~$0-10/month vs GCP Memorystore ~$45/month

---

### 2. Set Up GCP Project

```bash
# Login to GCP
gcloud auth login

# Set your project ID
export PROJECT_ID="your-project-id"
gcloud config set project $PROJECT_ID

# Enable required APIs
gcloud services enable \
  cloudbuild.googleapis.com \
  run.googleapis.com \
  sql-component.googleapis.com \
  secretmanager.googleapis.com
```

---

### 3. Create PostgreSQL Database (Cloud SQL)

**Option A: Using GCP Console (Recommended for first time)**

1. Go to: https://console.cloud.google.com/sql
2. Click "Create Instance" → "PostgreSQL"
3. Settings:
   - Instance ID: `pingmydb-postgres`
   - Password: (create a secure password)
   - Database version: PostgreSQL 15
   - Region: `us-central1`
   - Machine type: **Shared core** → `db-f1-micro` (cheapest)
   - Storage: 10 GB SSD
   - Backups: Disabled (for dev) or Enabled (for prod)
4. Click "Create" (takes ~5 minutes)
5. After creation:
   - Go to "Databases" tab → Create database → Name: `pingmydb`
   - Go to "Overview" tab → Copy "Connection name"

**Option B: Using Commands**

```bash
# Create PostgreSQL instance
gcloud sql instances create pingmydb-postgres \
  --database-version=POSTGRES_15 \
  --tier=db-f1-micro \
  --region=us-central1 \
  --root-password=YOUR_SECURE_PASSWORD

# Create database
gcloud sql databases create pingmydb \
  --instance=pingmydb-postgres

# Get connection name (save this!)
gcloud sql instances describe pingmydb-postgres \
  --format="value(connectionName)"
```

---

### 4. Store Secrets in Secret Manager

```bash
# JWT Secret (generate a random string)
echo -n "$(openssl rand -base64 32)" | \
  gcloud secrets create jwt-secret --data-file=-

# Encryption Key (32-byte hex for AES-256)
echo -n "$(openssl rand -hex 32)" | \
  gcloud secrets create encryption-key --data-file=-

# Database URL (replace CONNECTION_NAME and PASSWORD)
echo -n "postgresql://postgres:YOUR_PASSWORD@/pingmydb?host=/cloudsql/CONNECTION_NAME" | \
  gcloud secrets create database-url --data-file=-

# Upstash Redis URL (from Upstash console)
echo -n "redis://default:YOUR_UPSTASH_PASSWORD@YOUR_UPSTASH_HOST:PORT" | \
  gcloud secrets create redis-url --data-file=-

# SMTP Configuration (for Gmail)
echo -n "smtp.gmail.com" | gcloud secrets create smtp-host --data-file=-
echo -n "587" | gcloud secrets create smtp-port --data-file=-
echo -n "your-email@gmail.com" | gcloud secrets create smtp-user --data-file=-
echo -n "your-app-password" | gcloud secrets create smtp-pass --data-file=-
```

---

### 5. Build and Push Docker Images

```bash
# Configure Docker for GCP
gcloud auth configure-docker

# Build and push backend
cd backend
docker build -t gcr.io/$PROJECT_ID/pingmydb-backend:latest .
docker push gcr.io/$PROJECT_ID/pingmydb-backend:latest

# Build and push frontend
cd ../frontend
docker build -t gcr.io/$PROJECT_ID/pingmydb-frontend:latest .
docker push gcr.io/$PROJECT_ID/pingmydb-frontend:latest
```

**Or use the automated script:**
```bash
cd /Users/rishabhrathore/Desktop/pingmydb
./deploy-to-gcp.sh
```

---

### 6. Deploy Backend to Cloud Run

```bash
# Get your Cloud SQL connection name
export SQL_CONNECTION=$(gcloud sql instances describe pingmydb-postgres \
  --format="value(connectionName)")

# Deploy backend
gcloud run deploy pingmydb-backend \
  --image gcr.io/$PROJECT_ID/pingmydb-backend:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --add-cloudsql-instances $SQL_CONNECTION \
  --set-env-vars "PORT=8080,NODE_ENV=production" \
  --set-secrets "JWT_SECRET=jwt-secret:latest,ENCRYPTION_KEY=encryption-key:latest,DATABASE_URL=database-url:latest,REDIS_URL=redis-url:latest,SMTP_HOST=smtp-host:latest,SMTP_PORT=smtp-port:latest,SMTP_USER=smtp-user:latest,SMTP_PASS=smtp-pass:latest" \
  --memory 512Mi \
  --cpu 1 \
  --min-instances 0 \
  --max-instances 10 \
  --timeout 300

# Get backend URL (save this!)
gcloud run services describe pingmydb-backend \
  --region us-central1 \
  --format="value(status.url)"
```

---

### 7. Update Backend Code for Upstash (if needed)

Your current code should work with Upstash Redis URL. Just make sure your `backend/worker/dbMonitorWorker.js` uses the Redis URL from environment:

```javascript
// Should look like this:
const connection = process.env.REDIS_URL || { host: 'localhost', port: 6379 };
```

If you're using the REST API approach:
```javascript
const { Redis } = require('@upstash/redis');

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN,
});
```

---

### 8. Deploy Frontend to Cloud Run

First, update the frontend environment variable:

```bash
# Get your backend URL
export BACKEND_URL=$(gcloud run services describe pingmydb-backend \
  --region us-central1 \
  --format="value(status.url)")

# Create production env file
echo "VITE_API_URL=$BACKEND_URL" > frontend/.env.production

# Rebuild and push frontend
cd frontend
docker build -t gcr.io/$PROJECT_ID/pingmydb-frontend:latest .
docker push gcr.io/$PROJECT_ID/pingmydb-frontend:latest

# Deploy frontend
gcloud run deploy pingmydb-frontend \
  --image gcr.io/$PROJECT_ID/pingmydb-frontend:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --memory 256Mi \
  --cpu 1 \
  --min-instances 0 \
  --max-instances 5

# Get frontend URL
gcloud run services describe pingmydb-frontend \
  --region us-central1 \
  --format="value(status.url)"
```

---

## 🔐 Setting Up OAuth (Google & GitHub)

### Google OAuth Setup

1. **Go to:** https://console.cloud.google.com/apis/credentials
2. **Click:** "Create Credentials" → "OAuth client ID"
3. **If prompted**, configure OAuth consent screen first:
   - User Type: External
   - App name: PingMyDB
   - User support email: your-email@gmail.com
   - Developer contact: your-email@gmail.com
4. **Create OAuth Client ID:**
   - Application type: `Web application`
   - Name: `PingMyDB`
   - Authorized JavaScript origins:
     ```
     https://YOUR-FRONTEND-URL.run.app
     http://localhost:5173
     ```
   - Authorized redirect URIs:
     ```
     https://YOUR-BACKEND-URL.run.app/auth/google/callback
     http://localhost:5001/auth/google/callback
     ```
5. **Copy** Client ID and Client Secret

### GitHub OAuth Setup

1. **Go to:** https://github.com/settings/developers
2. **Click:** "New OAuth App"
3. **Fill in:**
   - Application name: `PingMyDB`
   - Homepage URL: `https://YOUR-FRONTEND-URL.run.app`
   - Authorization callback URL: `https://YOUR-BACKEND-URL.run.app/auth/github/callback`
4. **Click** "Register application"
5. **Generate** a new client secret
6. **Copy** Client ID and Client Secret

### Add OAuth Secrets to GCP

```bash
# Google OAuth
echo -n "your-google-client-id" | \
  gcloud secrets create google-client-id --data-file=-
echo -n "your-google-client-secret" | \
  gcloud secrets create google-client-secret --data-file=-

# GitHub OAuth
echo -n "your-github-client-id" | \
  gcloud secrets create github-client-id --data-file=-
echo -n "your-github-client-secret" | \
  gcloud secrets create github-client-secret --data-file=-
```

### Update Backend Deployment with OAuth

```bash
gcloud run services update pingmydb-backend \
  --region us-central1 \
  --update-secrets "GOOGLE_CLIENT_ID=google-client-id:latest,GOOGLE_CLIENT_SECRET=google-client-secret:latest,GITHUB_CLIENT_ID=github-client-id:latest,GITHUB_CLIENT_SECRET=github-client-secret:latest"
```

---

## 💰 Cost Breakdown (with Upstash)

### Monthly Costs:
- **Cloud Run Backend**: $0-5 (2M requests free, then ~$0.40/million)
- **Cloud Run Frontend**: $0-2 (mostly free tier)
- **Cloud SQL (db-f1-micro)**: ~$7/month
- **Upstash Redis**: $0-10/month (10K commands/day free)
- **Egress/Networking**: $0-5/month

**Total: ~$7-30/month** (vs ~$50-60 with GCP Memorystore)

### Cost Optimization Tips:
1. Use Cloud Run's `--min-instances 0` to scale to zero
2. Use Upstash free tier (10K commands/day)
3. Enable Cloud SQL automatic backups only if needed
4. Use Cloud Scheduler to pause SQL instance during off-hours (dev only)

---

## 🔧 Environment Variables Summary

### Backend Environment Variables:
```env
PORT=8080
NODE_ENV=production
DATABASE_URL=postgresql://...  # From Secret Manager
REDIS_URL=redis://...          # Upstash Redis URL
JWT_SECRET=...                 # From Secret Manager
ENCRYPTION_KEY=...             # From Secret Manager
GOOGLE_CLIENT_ID=...           # From Secret Manager
GOOGLE_CLIENT_SECRET=...       # From Secret Manager
GITHUB_CLIENT_ID=...           # From Secret Manager
GITHUB_CLIENT_SECRET=...       # From Secret Manager
```

### Frontend Environment Variables:
```env
VITE_API_URL=https://your-backend-url.run.app
```

---

## 📊 Monitoring & Logging

```bash
# View backend logs
gcloud run services logs read pingmydb-backend --region us-central1 --limit 50

# View frontend logs
gcloud run services logs read pingmydb-frontend --region us-central1 --limit 50

# Tail logs in real-time
gcloud run services logs tail pingmydb-backend --region us-central1
```

---

## 🔄 Updating Your Deployment

When you make code changes:

```bash
# Rebuild and redeploy backend
cd backend
docker build -t gcr.io/$PROJECT_ID/pingmydb-backend:latest .
docker push gcr.io/$PROJECT_ID/pingmydb-backend:latest
gcloud run deploy pingmydb-backend \
  --image gcr.io/$PROJECT_ID/pingmydb-backend:latest \
  --region us-central1

# Rebuild and redeploy frontend
cd ../frontend
docker build -t gcr.io/$PROJECT_ID/pingmydb-frontend:latest .
docker push gcr.io/$PROJECT_ID/pingmydb-frontend:latest
gcloud run deploy pingmydb-frontend \
  --image gcr.io/$PROJECT_ID/pingmydb-frontend:latest \
  --region us-central1
```

---

## ✅ Deployment Checklist

- [ ] Upstash Redis database created
- [ ] Redis URL and token copied
- [ ] GCP project created and billing enabled
- [ ] gcloud CLI installed and initialized
- [ ] PostgreSQL database created (Cloud SQL)
- [ ] Secrets stored in Secret Manager
- [ ] Backend Docker image built and pushed
- [ ] Frontend Docker image built and pushed
- [ ] Backend deployed to Cloud Run
- [ ] Frontend deployed to Cloud Run
- [ ] Google OAuth configured
- [ ] GitHub OAuth configured
- [ ] Environment variables set
- [ ] Test login functionality
- [ ] Test database monitoring

---

## 🆘 Troubleshooting

### Redis Connection Issues:
- Verify Upstash Redis URL format
- Check if REST API or TCP connection is being used
- Ensure REDIS_URL secret is set correctly

### Database Connection Issues:
- Ensure Cloud SQL instance is running
- Check connection name is correct
- Verify password in DATABASE_URL
- Check Cloud SQL Proxy is enabled in Cloud Run

### OAuth Issues:
- Verify redirect URIs match exactly (no trailing slashes)
- Ensure HTTPS is used in production
- Check OAuth consent screen is configured
- Verify client IDs and secrets are correct

---

## 📚 Additional Resources

- [Upstash Documentation](https://docs.upstash.com/)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Cloud SQL Documentation](https://cloud.google.com/sql/docs)
- [Secret Manager Documentation](https://cloud.google.com/secret-manager/docs)
