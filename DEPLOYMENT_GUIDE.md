# Video Intelligence Streamlit App - Complete Deployment Guide

## Table of Contents
1. [Project Overview](#project-overview)
2. [Environment Variables](#environment-variables)
3. [Local Development & Testing](#local-development--testing)
4. [Docker Testing](#docker-testing)
5. [Cloud Run Deployment](#cloud-run-deployment)
6. [Infrastructure Setup](#infrastructure-setup)
7. [Troubleshooting](#troubleshooting)

---

## Project Overview

**Application**: Video Intelligence Analysis with Google Gemini AI  
**Technology Stack**: Streamlit, Google Cloud Run, Vertex AI, Docker  
**Domain**: https://ssms.info  
**Authentication**: Identity-Aware Proxy (IAP) with Google OAuth

---

## Environment Variables

### 1. Google Cloud Project Settings

```bash
# Core Configuration
PROJECT_ID=video-intelligence-479903
PROJECT_NUMBER=588337360510
REGION=us-east1
```

### 2. Cloud Run Service

```bash
# Service Configuration
SERVICE_NAME=video-intelligence
CLOUD_RUN_URL=https://video-intelligence-mn55ncag3q-ue.a.run.app
```

**Environment Variables Set on Cloud Run**:
```bash
PROJECT_ID=video-intelligence-479903
REGION=us-east1
LOG_LEVEL=DEBUG
```

### 3. Service Accounts

```bash
# CI/CD Service Account
CICD_RUNNER_SA_EMAIL=video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com

# IAP Service Account (auto-created by Google)
IAP_SA=service-588337360510@gcp-sa-iap.iam.gserviceaccount.com
```

**Required IAM Roles** for `video-intelligence-sa`:
- `roles/cloudbuild.builds.builder`
- `roles/artifactregistry.writer`
- `roles/run.admin`
- `roles/iam.serviceAccountUser`
- `roles/run.invoker`

### 4. Artifact Registry

```bash
# Registry Configuration
AR_HOSTNAME=us-east1-docker.pkg.dev
GAR_REPO=video-intelligence
FULL_IMAGE_PATH=us-east1-docker.pkg.dev/video-intelligence-479903/video-intelligence/video-intelligence
```

### 5. Load Balancer & SSL

```bash
# Load Balancer Components
LB_IP=136.110.239.42
LB_CERT_NAME=lb-cert
LB_TARGET_PROXY=lb-target-proxy
LB_URL_MAP=lb-url-map
LB_BACKEND_SERVICE=video-intelligence-backend-service
LB_NEG=video-intelligence-serverless-neg
LB_FORWARDING_RULE=lb-forwarding-rule

# HTTP to HTTPS Redirect
LB_HTTP_URL_MAP=lb-url-map-http
LB_HTTP_TARGET_PROXY=lb-target-http-proxy
LB_HTTP_FORWARDING_RULE=lb-forwarding-rule-http

# Domain
DOMAIN=ssms.info
DNS_A_RECORD=136.110.239.42
DNS_TTL=300  # 5 minutes
```

### 6. IAP Configuration

```bash
# OAuth 2.0 Credentials
# ⚠️ SECURITY: Replace with your actual OAuth credentials from Google Cloud Console
# Get these from: https://console.cloud.google.com/apis/credentials
OAUTH_CLIENT_ID=YOUR_OAUTH_CLIENT_ID.apps.googleusercontent.com
OAUTH_CLIENT_SECRET=YOUR_OAUTH_CLIENT_SECRET

# IAP Settings
IAP_ENABLED=true
IAP_METHOD=IAM
```

### 7. Cloud Build Substitutions

**Variables in `deploy/cloudbuild.yaml`**:
```yaml
_DEPLOY_REGION: us-east1
_TRIGGER_ID: no-trigger
_AR_HOSTNAME: us-east1-docker.pkg.dev
_PLATFORM: managed
_SERVICE_NAME: video-intelligence
_GAR_REPO: video-intelligence
_LOG_LEVEL: DEBUG
_MAX_INSTANCES: "1"
_CICD_RUNNER_SA_EMAIL: video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com
```

### 8. Application Configuration

**Vertex AI Model**:
```bash
MODEL_NAME=gemini-2.5-flash
```

**Python Dependencies** (key packages):
- `streamlit`
- `google-cloud-aiplatform`
- `pytubefix`
- `youtube-transcript-api`
- `certifi`
- `yt-dlp`
- `python-dotenv`
- `ffmpeg-python`

---

## Local Development & Testing

### Prerequisites

1. **Python 3.9+** installed
2. **Google Cloud SDK** installed and authenticated
3. **Git** repository cloned

### Step 1: Set Up Python Virtual Environment

```bash
# Navigate to project directory
cd /path/to/youtube-and-video

# Create virtual environment
python3 -m venv .venv

# Activate virtual environment
# On macOS/Linux:
source .venv/bin/activate
# On Windows:
# .venv\Scripts\activate
```

### Step 2: Install Dependencies

```bash
# Install required packages
pip install -r src/video-intelligence-streamlit/requirements.txt
```

### Step 3: Set Up Environment Variables

Create a `.env` file in `src/video-intelligence-streamlit/`:

```bash
# src/video-intelligence-streamlit/.env
PROJECT_ID=video-intelligence-479903
REGION=us-east1
LOG_LEVEL=DEBUG
```

### Step 4: Authenticate with Google Cloud

```bash
# Login to Google Cloud
gcloud auth login

# Set default project
gcloud config set project video-intelligence-479903

# Set up Application Default Credentials (required for Vertex AI)
gcloud auth application-default login

# Set quota project
gcloud auth application-default set-quota-project video-intelligence-479903
```

### Step 5: Run Streamlit Locally

```bash
# Navigate to app directory
cd src/video-intelligence-streamlit

# Run Streamlit app
streamlit run app.py
```

**Expected Output**:
```
You can now view your Streamlit app in your browser.

Local URL: http://localhost:8501
Network URL: http://192.168.x.x:8501
```

### Step 6: Test the Application

1. **Open browser** to http://localhost:8501
2. **Upload a video** or enter a YouTube URL
3. **Verify**:
   - Video uploads successfully
   - Gemini AI analysis works
   - Transcripts are extracted
   - No errors in terminal

### Common Local Testing Issues

**Issue**: `ModuleNotFoundError: No module named 'vertexai'`
```bash
# Solution: Install google-cloud-aiplatform
pip install google-cloud-aiplatform
```

**Issue**: `SSL: CERTIFICATE_VERIFY_FAILED`
```bash
# Solution: Already handled in video_utils.py with certifi
# Verify certifi is installed:
pip install certifi
```

**Issue**: `PermissionError` when accessing Vertex AI
```bash
# Solution: Re-authenticate
gcloud auth application-default login
gcloud auth application-default set-quota-project video-intelligence-479903
```

---

## Docker Testing

### Step 1: Build Docker Image Locally

```bash
# Navigate to project root
cd /path/to/youtube-and-video

# Build Docker image
docker build -t video-intelligence:local -f src/video-intelligence-streamlit/Dockerfile src/video-intelligence-streamlit
```

### Step 2: Run Docker Container Locally

```bash
# Run container with environment variables
docker run -p 8080:8080 \
  -e PROJECT_ID=video-intelligence-479903 \
  -e REGION=us-east1 \
  -e LOG_LEVEL=DEBUG \
  -v ~/.config/gcloud:/root/.config/gcloud \
  video-intelligence:local
```

**Explanation**:
- `-p 8080:8080`: Map container port to host port
- `-e`: Set environment variables
- `-v`: Mount Google Cloud credentials (for local testing only)

### Step 3: Test Docker Container

1. **Open browser** to http://localhost:8080
2. **Verify** the app works as expected
3. **Check logs** in terminal for any errors

### Step 4: Test Without Credentials (Simulate Cloud Run)

```bash
# Run without mounted credentials (will fail auth, which is expected)
docker run -p 8080:8080 \
  -e PROJECT_ID=video-intelligence-479903 \
  -e REGION=us-east1 \
  -e LOG_LEVEL=DEBUG \
  video-intelligence:local
```

This simulates Cloud Run environment. The app should show authentication errors, which is expected locally.

### Docker Testing Checklist

- [ ] Image builds without errors
- [ ] Container starts successfully
- [ ] App is accessible on port 8080
- [ ] Static assets load correctly
- [ ] Environment variables are read correctly
- [ ] No missing dependencies

---

## Cloud Run Deployment

### Method 1: Automated Deployment via Git Push (Recommended)

#### Prerequisites
- Cloud Build trigger configured (`deploy-to-dev`)
- GitHub repository connected
- Service account permissions granted

#### Deployment Steps

```bash
# 1. Make code changes
# 2. Commit changes
git add .
git commit -m "Your commit message"

# 3. Push to main branch (triggers automatic build)
git push origin main
```

#### Monitor Deployment

```bash
# Watch Cloud Build logs
gcloud builds list --limit=5

# Get specific build logs
gcloud builds log <BUILD_ID>

# Check Cloud Run service status
gcloud run services describe video-intelligence --region=us-east1
```

### Method 2: Manual Deployment via gcloud

#### Step 1: Build and Push Docker Image

```bash
# Set variables
PROJECT_ID=video-intelligence-479903
REGION=us-east1
SERVICE_NAME=video-intelligence
IMAGE_TAG=$(git rev-parse --short HEAD)

# Build image
docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/video-intelligence/${SERVICE_NAME}:${IMAGE_TAG} \
  -f src/video-intelligence-streamlit/Dockerfile \
  src/video-intelligence-streamlit

# Authenticate Docker with Artifact Registry
gcloud auth configure-docker ${REGION}-docker.pkg.dev

# Push image
docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/video-intelligence/${SERVICE_NAME}:${IMAGE_TAG}
```

#### Step 2: Deploy to Cloud Run

```bash
gcloud run deploy video-intelligence \
  --image=${REGION}-docker.pkg.dev/${PROJECT_ID}/video-intelligence/${SERVICE_NAME}:${IMAGE_TAG} \
  --platform=managed \
  --region=${REGION} \
  --no-allow-unauthenticated \
  --max-instances=1 \
  --min-instances=0 \
  --set-env-vars=PROJECT_ID=${PROJECT_ID},REGION=${REGION},LOG_LEVEL=DEBUG
```

#### Step 3: Grant IAP Service Account Access

```bash
# Allow IAP service account to invoke Cloud Run
gcloud run services add-iam-policy-binding video-intelligence \
  --region=${REGION} \
  --member="serviceAccount:service-588337360510@gcp-sa-iap.iam.gserviceaccount.com" \
  --role="roles/run.invoker"
```

### Deployment Verification

```bash
# Check service status
gcloud run services describe video-intelligence --region=us-east1 --format="yaml(status.conditions)"

# Get service URL
gcloud run services describe video-intelligence --region=us-east1 --format="value(status.url)"

# Test direct Cloud Run URL (should get 403 if IAP is working correctly)
curl -I <CLOUD_RUN_URL>
```

### Deployment Checklist

- [ ] Docker image builds successfully
- [ ] Image pushed to Artifact Registry
- [ ] Cloud Run service deployed
- [ ] Environment variables set correctly
- [ ] Service is not publicly accessible (returns 403)
- [ ] IAP service account has invoker permission
- [ ] Load balancer routes to Cloud Run service

---

## Infrastructure Setup

### Complete Infrastructure Architecture

```
Internet
   ↓
DNS (ssms.info) → 136.110.239.42
   ↓
HTTPS Load Balancer (port 443)
   ├─ SSL Certificate (lb-cert) → ACTIVE
   ├─ Target HTTPS Proxy → lb-target-proxy
   ├─ URL Map → lb-url-map
   └─ IAP Authentication → OAuth 2.0
       ↓
Backend Service → video-intelligence-backend-service
   ↓
Network Endpoint Group → video-intelligence-serverless-neg
   ↓
Cloud Run Service → video-intelligence
   ↓
Streamlit App + Vertex AI
```

### SSL Certificate Setup

**Status**: Active  
**Domain**: ssms.info  
**Issuer**: Google Trust Services  
**Valid Until**: March 6, 2026  
**Auto-renewal**: Enabled

### IAP Configuration

**OAuth Consent Screen**:
- App name: video intelligence
- User type: External
- Publishing status: Testing
- Authorized domains: ssms.info

**Authorized Users**:
- Email: mishra.sunny@gmail.com
- Role: IAP-secured Web App User

**OAuth Redirect URIs**:
```
https://iap.googleapis.com/v1/oauth/clientIds/YOUR_OAUTH_CLIENT_ID.apps.googleusercontent.com:handleRedirect
https://ssms.info/_gcp_iap/oauth2callback
```

### DNS Configuration

**Provider**: Squarespace  
**Record Type**: A  
**Host**: @ (root domain)  
**Value**: 136.110.239.42  
**TTL**: 300 seconds (5 minutes)

### HTTP to HTTPS Redirect

**Configuration**: Enabled  
**Status Code**: 301 Permanent Redirect  
**Behavior**: All HTTP traffic → HTTPS

---

## Troubleshooting

### Issue 1: SSL Certificate Stuck in PROVISIONING

**Symptoms**: Certificate shows `FAILED_NOT_VISIBLE` or stays in `PROVISIONING` for hours

**Causes**:
- DNS hasn't propagated globally
- High TTL on DNS records
- CAA records blocking certificate issuance

**Solutions**:
```bash
# 1. Verify DNS is correct
dig ssms.info +short
# Should return: 136.110.239.42

# 2. Check certificate status
gcloud compute ssl-certificates describe lb-cert --global --format="value(managed.status,managed.domainStatus)"

# 3. Wait for DNS propagation (with 5-min TTL, should take 30-90 minutes)

# 4. Check from Google's DNS servers
dig @8.8.8.8 ssms.info +short
```

### Issue 2: "Empty Google Account OAuth client ID(s)/secret(s)"

**Symptoms**: Error when accessing site through load balancer

**Cause**: IAP not configured with OAuth credentials

**Solution**:
```bash
# Configure IAP with OAuth client
# ⚠️ SECURITY: Replace YOUR_OAUTH_CLIENT_ID and YOUR_OAUTH_CLIENT_SECRET with actual values
gcloud iap web enable --resource-type=backend-services \
    --oauth2-client-id=YOUR_OAUTH_CLIENT_ID.apps.googleusercontent.com \
    --oauth2-client-secret=YOUR_OAUTH_CLIENT_SECRET \
    --service=video-intelligence-backend-service
```

### Issue 3: Cloud Build Fails - "bad substitution"

**Symptoms**: Build fails with `$_CB_REGION-docker.pkg.dev: bad substitution`

**Cause**: Undefined variable in cloudbuild.yaml

**Solution**: Use hardcoded region instead of variables
```yaml
_AR_HOSTNAME: us-east1-docker.pkg.dev  # Not: $_CB_REGION-docker.pkg.dev
```

### Issue 4: Cloud Build Permission Denied

**Symptoms**: Build fails with permission errors

**Cause**: Service account lacks necessary IAM roles

**Solution**:
```bash
# Grant required permissions
gcloud projects add-iam-policy-binding video-intelligence-479903 \
    --member="serviceAccount:video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com" \
    --role="roles/cloudbuild.builds.builder"

gcloud projects add-iam-policy-binding video-intelligence-479903 \
    --member="serviceAccount:video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com" \
    --role="roles/artifactregistry.writer"

gcloud projects add-iam-policy-binding video-intelligence-479903 \
    --member="serviceAccount:video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com" \
    --role="roles/run.admin"

gcloud projects add-iam-policy-binding video-intelligence-479903 \
    --member="serviceAccount:video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com" \
    --role="roles/iam.serviceAccountUser"
```

### Issue 5: IAP Error - "IAP only available for organizations"

**Symptoms**: Deployment fails with organization requirement error

**Cause**: Using `--iap` flag on Cloud Run (not supported for non-org projects)

**Solution**: Remove `--iap` flag from Cloud Run deployment. IAP should only be at load balancer level.

### Issue 6: YouTube Download Fails with 403 or "Bot Detected"

**Symptoms**: YouTube videos fail to download from Cloud Run

**Cause**: YouTube blocks automated downloads from cloud IPs

**Solution**: This is expected behavior. The app includes manual upload fallback:
```python
# video_utils.py handles this with error message:
"YouTube is blocking automated downloads from this server. 
Please download the video manually and upload it using the file uploader below."
```

### Issue 7: ModuleNotFoundError: vertexai

**Symptoms**: Import error for vertexai module

**Cause**: Wrong virtual environment or missing package

**Solution**:
```bash
# Ensure you're in correct virtual environment
source .venv/bin/activate

# Install google-cloud-aiplatform (provides vertexai)
pip install google-cloud-aiplatform

# Verify installation
python -c "import vertexai; print('Success')"
```

### Issue 8: 403 Forbidden When Accessing Site

**Symptoms**: Users get 403 error at https://ssms.info

**Causes & Solutions**:

**A. User not authorized in IAP**:
```bash
# Add user to IAP
# Go to: https://console.cloud.google.com/security/iap
# Check box next to backend service
# Add user with role: IAP-secured Web App User
```

**B. OAuth consent screen not configured**:
```bash
# Configure at: https://console.cloud.google.com/apis/credentials/consent
# Fill in required fields (app name, support email, etc.)
```

**C. User not added as test user** (if in Testing mode):
```bash
# Go to OAuth consent screen > Audience
# Add user email to test users list
```

### Useful Debugging Commands

```bash
# Check Cloud Run logs
gcloud run services logs read video-intelligence --region=us-east1 --limit=50

# Check Cloud Build history
gcloud builds list --limit=10

# Verify IAP configuration
gcloud compute backend-services describe video-intelligence-backend-service --global --format="yaml(iap)"

# Check SSL certificate
gcloud compute ssl-certificates describe lb-cert --global

# Test load balancer health
curl -I https://ssms.info

# Check DNS propagation
nslookup ssms.info
dig ssms.info

# View service account permissions
gcloud projects get-iam-policy video-intelligence-479903 \
  --flatten="bindings[].members" \
  --filter="bindings.members:video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com"
```

---

## Quick Reference Commands

### Project Setup
```bash
# Set project
gcloud config set project video-intelligence-479903

# Authenticate
gcloud auth login
gcloud auth application-default login
```

### Development
```bash
# Local testing
cd src/video-intelligence-streamlit
source ../../.venv/bin/activate
streamlit run app.py

# Docker testing
docker build -t video-intelligence:local -f src/video-intelligence-streamlit/Dockerfile src/video-intelligence-streamlit
docker run -p 8080:8080 video-intelligence:local
```

### Deployment
```bash
# Deploy via Git
git add .
git commit -m "Deploy changes"
git push origin main

# Check deployment status
gcloud builds list --limit=5
gcloud run services describe video-intelligence --region=us-east1
```

### Monitoring
```bash
# View logs
gcloud run services logs read video-intelligence --region=us-east1

# Check service status
gcloud run services list --platform=managed

# View Cloud Build logs
gcloud builds log <BUILD_ID>
```

---

## Cost Optimization

### Enable Scale-to-Zero (Pay Only When Using)

By default, Cloud Run scales to zero when there's no traffic, meaning you only pay when the service is actively handling requests. If you're being charged continuously (e.g., $0.628/day), it likely means `min-instances` is set to 1 or higher.

**Immediate Fix** - Update existing service to scale to zero:

```bash
# Set min-instances to 0 to enable scale-to-zero
gcloud run services update video-intelligence \
  --region=us-east1 \
  --min-instances=0
```

**Verify the change**:

```bash
# Check current scaling configuration
gcloud run services describe video-intelligence \
  --region=us-east1 \
  --format="value(spec.template.spec.containers[0].resources.limits,spec.template.spec.containerConcurrency,spec.template.metadata.annotations)"
```

**Expected behavior after fix**:
- Service scales to zero when idle (no traffic for ~15 minutes)
- Cold start delay of ~5-10 seconds on first request after idle period
- You only pay for actual request processing time
- Estimated cost: ~$0.00/day when idle (vs $0.628/day with min-instances=1)

**Note**: The deployment configuration (`deploy/cloudbuild.yaml`) has been updated to include `--min-instances=0` by default, so future deployments will automatically scale to zero.

---

## Additional Resources

- **Cloud Console**: https://console.cloud.google.com
- **Cloud Run Service**: https://console.cloud.google.com/run?project=video-intelligence-479903
- **Cloud Build History**: https://console.cloud.google.com/cloud-build/builds?project=video-intelligence-479903
- **IAP Configuration**: https://console.cloud.google.com/security/iap?project=video-intelligence-479903
- **Artifact Registry**: https://console.cloud.google.com/artifacts?project=video-intelligence-479903

---

**Last Updated**: December 7, 2025  
**Version**: 1.0  
**Maintained By**: Sunny Mishra



