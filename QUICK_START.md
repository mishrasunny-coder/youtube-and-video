# Quick Start Guide

Get your Video Intelligence app running in 5 minutes!

---

## 📚 Documentation Overview

- **[DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)**: Complete deployment documentation
- **[ENVIRONMENT_VARIABLES.md](./ENVIRONMENT_VARIABLES.md)**: All environment variables reference
- **This file**: Quick start for impatient developers 😊

---

## 🚀 Quick Start: Local Testing

### 1. Clone & Setup (2 minutes)

```bash
# Clone repository
cd /path/to/youtube-and-video

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r src/video-intelligence-streamlit/requirements.txt
```

### 2. Configure Environment (1 minute)

```bash
# Create .env file
cat > src/video-intelligence-streamlit/.env << EOF
PROJECT_ID=video-intelligence-479903
REGION=us-east1
LOG_LEVEL=DEBUG
EOF

# Authenticate with Google Cloud
gcloud auth application-default login
gcloud auth application-default set-quota-project video-intelligence-479903
```

### 3. Run App (30 seconds)

```bash
cd src/video-intelligence-streamlit
streamlit run app.py
```

✅ **Open browser**: http://localhost:8501

---

## 🐳 Quick Start: Docker Testing

### Build & Run (2 minutes)

```bash
# Build image
docker build -t video-intelligence:local \
  -f src/video-intelligence-streamlit/Dockerfile \
  src/video-intelligence-streamlit

# Run container
docker run -p 8080:8080 \
  -e PROJECT_ID=video-intelligence-479903 \
  -e REGION=us-east1 \
  -v ~/.config/gcloud:/root/.config/gcloud \
  video-intelligence:local
```

✅ **Open browser**: http://localhost:8080

---

## ☁️ Quick Start: Deploy to Cloud Run

### Option 1: Git Push (Automatic) ⭐ Recommended

```bash
git add .
git commit -m "Deploy to Cloud Run"
git push origin main
```

✅ **Monitor**: https://console.cloud.google.com/cloud-build/builds?project=video-intelligence-479903

### Option 2: Manual Deploy

```bash
# Set variables
export PROJECT_ID=video-intelligence-479903
export REGION=us-east1
export TAG=$(git rev-parse --short HEAD)

# Build & push
docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/video-intelligence/video-intelligence:${TAG} \
  -f src/video-intelligence-streamlit/Dockerfile src/video-intelligence-streamlit

gcloud auth configure-docker ${REGION}-docker.pkg.dev

docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/video-intelligence/video-intelligence:${TAG}

# Deploy
gcloud run deploy video-intelligence \
  --image=${REGION}-docker.pkg.dev/${PROJECT_ID}/video-intelligence/video-intelligence:${TAG} \
  --platform=managed \
  --region=${REGION} \
  --no-allow-unauthenticated \
  --max-instances=1 \
  --set-env-vars=PROJECT_ID=${PROJECT_ID},REGION=${REGION},LOG_LEVEL=DEBUG
```

---

## 🌐 Access Your Deployed App

### Production URL
**https://ssms.info**

### Authentication
- Login with: **mishra.sunny@gmail.com**
- Protected by: **Google IAP**

### Direct Cloud Run URL (for testing)
```bash
# Get URL
gcloud run services describe video-intelligence --region=us-east1 --format="value(status.url)"

# Test (should return 403 - expected)
curl -I <CLOUD_RUN_URL>
```

---

## 🔍 Quick Health Checks

### Check Deployment Status
```bash
gcloud run services describe video-intelligence --region=us-east1 --format="yaml(status.conditions)"
```

### View Recent Logs
```bash
gcloud run services logs read video-intelligence --region=us-east1 --limit=20
```

### Check Cloud Build Status
```bash
gcloud builds list --limit=5
```

### Verify SSL Certificate
```bash
gcloud compute ssl-certificates describe lb-cert --global --format="value(managed.status,managed.domainStatus)"
```

---

## 🆘 Common Issues & Quick Fixes

### Issue: "ModuleNotFoundError: vertexai"
```bash
pip install google-cloud-aiplatform
```

### Issue: "Permission denied" errors
```bash
gcloud auth application-default login
gcloud auth application-default set-quota-project video-intelligence-479903
```

### Issue: Can't access https://ssms.info
1. Verify you're signed in with: **mishra.sunny@gmail.com**
2. Check IAP: https://console.cloud.google.com/security/iap?project=video-intelligence-479903
3. Ensure you're added as authorized user

### Issue: Cloud Build fails
```bash
# Check latest build logs
BUILD_ID=$(gcloud builds list --limit=1 --format="value(id)")
gcloud builds log $BUILD_ID
```

### Issue: SSL certificate not working
```bash
# Check status (should be ACTIVE)
gcloud compute ssl-certificates describe lb-cert --global

# Check DNS
dig ssms.info +short
# Should return: 136.110.239.42
```

---

## 📊 Architecture Overview

```
User Request → https://ssms.info
    ↓
Load Balancer (136.110.239.42)
    ↓
IAP Authentication (Google OAuth)
    ↓
Cloud Run: video-intelligence
    ↓
Streamlit App + Vertex AI
```

---

## 📖 Learn More

For detailed information:
- **Full deployment guide**: [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)
- **Environment variables**: [ENVIRONMENT_VARIABLES.md](./ENVIRONMENT_VARIABLES.md)
- **Troubleshooting**: See DEPLOYMENT_GUIDE.md → Troubleshooting section

---

## 🎯 Next Steps

1. ✅ Test locally with Streamlit
2. ✅ Test with Docker
3. ✅ Deploy to Cloud Run
4. ✅ Access via https://ssms.info
5. 🎉 Start analyzing videos!

---

**Need Help?**
- Check [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) for detailed troubleshooting
- Review logs: `gcloud run services logs read video-intelligence --region=us-east1`
- Cloud Console: https://console.cloud.google.com/run?project=video-intelligence-479903

---

**Last Updated**: December 7, 2025  
**Version**: 1.0




