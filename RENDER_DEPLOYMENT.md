# Render Deployment Guide

This document explains how to deploy the Podcast Web App on Render using the provided `render.yaml` blueprint.

## Prerequisites

1. GitHub account with the repository pushed
2. Render account (https://render.com)
3. MongoDB Atlas account (https://www.mongodb.com/cloud/atlas) - for the database

## Step-by-Step Deployment

### 1. Connect MongoDB Atlas

If you don't have MongoDB Atlas setup:
- Go to https://www.mongodb.com/cloud/atlas
- Create a free cluster
- Get your connection string (looks like: `mongodb+srv://username:password@cluster.mongodb.net/?appName=Cluster0`)

### 2. Deploy on Render

1. Go to https://render.com/dashboard
2. Click **"New +"** → **"Blueprint"**
3. Select your GitHub repository (`podcastWebApp`)
4. Render will auto-detect the `render.yaml` file
5. Click **"Deploy"**

### 3. Configure Environment Variables

After deployment, go to your service settings and add these **secret environment variables**:

#### Required Secrets (Keep these private):

| Variable | Value | Source |
|----------|-------|--------|
| `MONGO_ATLAS_URL` | Your MongoDB connection string | MongoDB Atlas dashboard |
| `GOOGLE_CLIENT_ID` | Your Google OAuth app ID | Google Cloud Console |
| `GOOGLE_CLIENT_SECRET` | Your Google OAuth app secret | Google Cloud Console |
| `SECRET_KEY` | A secure random string (generate your own) | Create a strong random key |

#### Example Google OAuth Setup:
1. Go to https://console.cloud.google.com
2. Create a new project
3. Enable Google+ API
4. Create OAuth 2.0 credentials (Web application)
5. Add `https://your-render-service.onrender.com/api/auth/google/callback` to authorized redirect URIs
6. Copy the Client ID and Client Secret to Render environment variables

### 4. What Gets Deployed

The `render.yaml` automatically:

✅ **Frontend (React)**
- Runs `npm install` and `npm run build`
- Builds React app in production mode
- Served by FastAPI backend

✅ **Backend (FastAPI)**
- Installs Python dependencies from `requirements.txt`
- Runs uvicorn server
- Handles both API requests and frontend static files
- Connects to MongoDB Atlas

## Environment Variables Reference

### Frontend Build Variables (Build Time)
- `REACT_APP_BACKEND_URL`: Backend API endpoint
- `REACT_APP_ENABLE_VISUAL_EDITS`: Visual editing feature flag
- `ENABLE_HEALTH_CHECK`: Health check feature flag

### Backend Runtime Variables (Runtime)
- `PYTHONUNBUFFERED`: Python logging
- `PORT`: Server port (automatically set to 10000)
- `MONGO_ATLAS_URL`: MongoDB connection string (SECRET)
- `DB_NAME`: Database name (default: podcast_network)
- `CORS_ORIGINS`: Allowed origins for CORS
- `FRONTEND_URL`: Frontend URL for redirects
- `GOOGLE_CLIENT_ID`: Google OAuth ID (SECRET)
- `GOOGLE_CLIENT_SECRET`: Google OAuth secret (SECRET)
- `GOOGLE_REDIRECT_URI`: Google OAuth redirect URL
- `SECRET_KEY`: JWT secret key (SECRET)

## Deployment URLs

After successful deployment, your app will be available at:
- **Frontend & API**: `https://podcast-web-app.onrender.com`
- **API Only**: `https://podcast-web-app.onrender.com/api`
- **Google OAuth Callback**: `https://podcast-web-app.onrender.com/api/auth/google/callback`

## Troubleshooting

### Build Fails
- Check that `npm run build` succeeds locally
- Ensure all frontend dependencies are in `package.json`
- Check that Python dependencies are in `requirements.txt`

### App Shows 404
- Verify the build completed successfully
- Check that `frontend/build/index.html` exists
- Review server logs in Render dashboard

### Database Connection Errors
- Verify `MONGO_ATLAS_URL` is correct
- Check MongoDB Atlas IP whitelist includes Render (or allow all IPs)
- Ensure database name matches in backend `.env`

### Google OAuth Issues
- Verify redirect URI matches exactly: `https://your-service.onrender.com/api/auth/google/callback`
- Ensure `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` are set in environment
- Check Google Cloud Console credentials are valid

## Local Development

To test locally before deploying:

```bash
# Terminal 1: Backend
cd backend
pip install -r requirements.txt
export FRONTEND_URL=http://localhost:3000
python -m uvicorn server:app --reload

# Terminal 2: Frontend
cd frontend
npm install
npm start
```

## Notes

- Free tier services on Render may spin down after 15 minutes of inactivity
- For production, consider upgrading to a paid tier
- All build logs visible in Render dashboard
- Auto-deploy enabled: pushes to `master` trigger automatic redeploys
