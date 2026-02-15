# Deployment Guide

PingMyDb is a monorepo containing a React frontend and a Node.js backend.

## 1. Backend Deployment (DigitalOcean / Heroku / Railway)
1. **Database**: Provision a PostgreSQL instance and run the migrations found in `backend/migrations/`.
2. **Redis**: Deploy a Redis instance for BullMQ.
3. **Environment**: Set the following variables on your hosting provider:
   - `DATABASE_URL`, `REDIS_URL`
   - `JWT_SECRET`, `ENCRYPTION_KEY`
   - `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`
4. **Command**:
   ```bash
   cd backend
   npm install
   npm start
   ```

## 2. Frontend Deployment (Vercel / Netlify)
1. **Build Settings**:
   - Framework Preset: Vite
   - Root Directory: `frontend`
   - Build Command: `npm run build`
   - Output Directory: `dist`
2. **Environment Variables**:
   - `VITE_API_URL`: Your deployed backend URL.
   - `VITE_RAZORPAY_KEY_ID`: Your public Razorpay key ID.

## 3. Worker Scaling
For high-traffic production environments, consider running the monitoring worker as a separate process from the web server. You can achieve this by setting a different entry point for workers that only initializes the BullMQ `Worker`.

## 4. Security Checklist
- Ensure `ENCRYPTION_KEY` is exactly 32 bytes (64 hex characters).
- Rotate `JWT_SECRET` periodically.
- Use HTTPS for all production traffic to protect tokens and credentials in transit.
