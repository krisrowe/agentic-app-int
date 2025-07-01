# agentic-app-int
Demo of leveraging Google's Application Integration platform from Conversational Agents and Agentspace

# Purpose

This repository contains a demonstration of how to integrate Google's Application Integration (Apigee Integration) platform with conversational agents and Agentspace. The goal is to showcase how to build robust, event-driven integrations that can be triggered and managed through natural language interactions.

# Setup

## Local Environment

1.  **Clone the repository:**

    ```
    git clone https://github.com/kdrowe/agentic-app-int.git
    cd agentic-app-int
    ```

2.  **Set up a Python virtual environment and install dependencies:**

    If you are using Poetry (recommended):

    ```bash
    poetry install
    poetry shell
    ```

    If you are not using Poetry:

    ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    ```
    
## Cloud Resources


1. **Authenticate `gcloud`:**

    ```bash
    gcloud auth application-default login
    ```

2. **Configure your Google Cloud Project:**

    Set your Google Cloud project ID:

    ```bash
    PROJECT_ID=YOUR_PROJECT_ID
    gcloud config set project $PROJECT_ID
    PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")

    ```

3. Import data set (military-bases.csv) into Cloud Storage

```
# Create the bucket
gcloud storage buckets create gs://$PROJECT_ID \
    --project=$PROJECT_ID \
    --location="us-central1"
   
# Upload the file to the bucket
gsutil cp military-bases.csv gs://$PROJECT_ID/military-bases.csv
```

4. Import data set into BigQuery
```
bq mk --dataset --default_table_expiration 36000 $PROJECT_ID:dod

# this next line must specify a project

bq load --schema=bq_schema.json --source_format=CSV --project_id=$PROJECT_ID --skip_leading_rows=1 $PROJECT_ID:dod.military_bases gs://$PROJECT_ID/military-bases.csv 
```

# Prep for using the Geocoding API used in service orchestration
```
gcloud services enable geocoding-backend.googleapis.com --project=$PROJECT_ID

gcloud services api-keys create \
    --project=$PROJECT_ID \
    --display-name="Geocoding API Key" \
    --api-target=service=geocoding-backend.googleapis.com

API_KEY=(your API Key from output generated above)

# Store the API Key in Secrets Manager
echo -n $API_KEY | gcloud secrets create geocode-api-key --project=$PROJECT_ID --data-file=-
# Or, to update the existing key, run the following.
echo -n $API_KEY | gcloud secrets versions add geocode-api-key --project=$PROJECT_ID --data-file=-

# Grant access for our custom service account (TODO: create it above) to access the API key secret
gcloud secrets add-iam-policy-binding geocode-api-key --project=$PROJECT_ID --role=roles/secretmanager.secretAccessor --member=serviceAccount:integration@${PROJECT_ID}.iam.gserviceaccount.com


```


