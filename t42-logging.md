# Google Cloud Logging Setup for Project "t42-logging"  
With 14‑day log retention and routing all logs to a custom bucket.

## 0. Variables
```bash
PROJECT_NAME="Team42 logging"
PROJECT_ID="t42-logging"
BILLING_ACCOUNT_ID="XXXXXX-YYYYYY-ZZZZZZ"
SA_NAME="t42-logging-writer"
SA_EMAIL="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
KEY_FILE="${HOME}/${PROJECT_ID}-${SA_NAME}.json"

LOCATION="europe-west1"
BUCKET_NAME="t42-logging-14d"
SINK_NAME="t42-logging-14d"
DEST="logging.googleapis.com/projects/${PROJECT_ID}/locations/${LOCATION}/buckets/${BUCKET_NAME}"
```

## 1. Create the Project
```bash
gcloud projects create "${PROJECT_ID}" --name="${PROJECT_NAME}"
gcloud config set project "${PROJECT_ID}"
```

## 2. Link Billing
```bash
gcloud beta billing projects link "${PROJECT_ID}"   --billing-account="${BILLING_ACCOUNT_ID}"
```

## 3. Enable Cloud Logging API
```bash
gcloud services enable logging.googleapis.com --project="${PROJECT_ID}"
```

## 4. Create Service Account
```bash
gcloud iam service-accounts create "${SA_NAME}"   --display-name="Logs Writer for DigitalOcean"
```

## 5. Grant Logs Writer Role
```bash
gcloud projects add-iam-policy-binding "${PROJECT_ID}"   --member="serviceAccount:${SA_EMAIL}"   --role="roles/logging.logWriter"
```

## 6. Create JSON Key
```bash
gcloud iam service-accounts keys create "${KEY_FILE}"   --iam-account="${SA_EMAIL}"
```

## 7. Create Log Bucket With 14-Day Retention
```bash
gcloud logging buckets create "${BUCKET_NAME}"   --location="${LOCATION}"   --project="${PROJECT_ID}"   --retention-days=14
```

### (If bucket already exists)
```bash
gcloud logging buckets update "${BUCKET_NAME}"   --location="${LOCATION}"   --project="${PROJECT_ID}"   --retention-days=14
```

## 8. Route **all logs** into this bucket (sink without filter)
```bash
gcloud logging sinks create "${SINK_NAME}" "${DEST}"   --project="${PROJECT_ID}"   --description="Route all logs to custom bucket with 14‑day retention"
```

## 9. Add Logs Viewer Permissions to Team42 group
```bash
gcloud projects add-iam-policy-binding "${PROJECT_ID}" \
  --member="group:team42@epages.com" \
  --role="roles/logging.viewer"
```

## 10. Verify
```bash
gcloud services list --project="${PROJECT_ID}"   --filter="NAME:logging.googleapis.com"

gcloud projects get-iam-policy "${PROJECT_ID}"   --flatten="bindings[].members"   --format="table(bindings.role, bindings.members)"
```
