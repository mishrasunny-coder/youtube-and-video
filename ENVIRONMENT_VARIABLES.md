# Environment Variables Quick Reference

## Local Development (.env file)

Create this file: `src/video-intelligence-streamlit/.env`

```bash
PROJECT_ID=video-intelligence-479903
REGION=us-east1
LOG_LEVEL=DEBUG
```

---

## Shell Environment Variables (for CLI operations)

```bash
# Google Cloud Configuration
export PROJECT_ID=video-intelligence-479903
export PROJECT_NUMBER=588337360510
export REGION=us-east1
export SERVICE_NAME=video-intelligence

# Service Accounts
export CICD_RUNNER_SA_EMAIL=video-intelligence-sa@video-intelligence-479903.iam.gserviceaccount.com
export IAP_SA=service-588337360510@gcp-sa-iap.iam.gserviceaccount.com

# Artifact Registry
export AR_HOSTNAME=us-east1-docker.pkg.dev
export GAR_REPO=video-intelligence

# Load Balancer
export LB_IP=136.110.239.42
export DOMAIN=ssms.info
```

---

## Cloud Run Environment Variables

Set these in Cloud Run deployment:

```bash
PROJECT_ID=video-intelligence-479903
REGION=us-east1
LOG_LEVEL=DEBUG
```

**Command to set**:
```bash
gcloud run services update video-intelligence \
  --region=us-east1 \
  --set-env-vars=PROJECT_ID=video-intelligence-479903,REGION=us-east1,LOG_LEVEL=DEBUG
```

---

## Cloud Build Substitutions

In `deploy/cloudbuild.yaml`:

```yaml
substitutions:
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

---

## IAP Configuration Variables

```bash
# OAuth 2.0
# ⚠️ SECURITY: Replace with your actual OAuth credentials from Google Cloud Console
# Get these from: https://console.cloud.google.com/apis/credentials
OAUTH_CLIENT_ID=YOUR_OAUTH_CLIENT_ID.apps.googleusercontent.com
OAUTH_CLIENT_SECRET=YOUR_OAUTH_CLIENT_SECRET
```

⚠️ **Security Note**: Keep these secrets secure. Do not commit to public repositories.

---

## Application Configuration

In `src/video-intelligence-streamlit/app.py`:

```python
MODEL_NAME = "gemini-2.5-flash"
```

---

## Quick Setup Script

Save this as `setup_env.sh`:

```bash
#!/bin/bash

# Google Cloud Configuration
export PROJECT_ID=video-intelligence-479903
export PROJECT_NUMBER=588337360510
export REGION=us-east1
export SERVICE_NAME=video-intelligence

# Authenticate
echo "Authenticating with Google Cloud..."
gcloud config set project $PROJECT_ID
gcloud auth application-default login
gcloud auth application-default set-quota-project $PROJECT_ID

echo "Environment configured for project: $PROJECT_ID"
echo "Region: $REGION"
echo "Service: $SERVICE_NAME"
```

**Usage**:
```bash
chmod +x setup_env.sh
source setup_env.sh
```

---

**Last Updated**: December 7, 2025




