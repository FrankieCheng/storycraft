# StoryCraft Deployment Guide (gcloud)

This guide provides instructions for deploying the StoryCraft application to Google Cloud Platform using the `gcloud` command-line tool.

## Prerequisites

1.  **Google Cloud SDK:** Make sure you have the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) installed and initialized.
2.  **gcloud Authentication:** Authenticate with gcloud:
    ```bash
    gcloud auth login
    ```
3.  **Project ID:** Have your Google Cloud Project ID ready.
4. **gcloud commands permission**: Make sure your service account used for gcloud build has the permissions to deploy the application to CloudRun. You can add the Role 'Cloud Run Service Agent' to service account {project_number}@cloudbuild.gserviceaccount.com 

5. **gcloud commands**:Rename/Copy `gcloud-deploy.sh.template`  and `gcloud-run-build.sh.template` to `gcloud-deploy.sh`  and `gcloud-run-build.sh`.

## Deployment Steps

The deployment process is divided into two main parts:

1.  **Infrastructure Setup:** Provisioning the necessary Google Cloud resources.
2.  **Application Deployment:** Building and deploying the StoryCraft application to Cloud Run.

### 1. Infrastructure Setup

The `gcloud-deploy.sh` script automates the setup of the required infrastructure. This includes:

*   Enabling necessary Google Cloud APIs.
*   Creating a service account for the application.
*   Granting IAM roles to the service account.
*   Creating a Cloud Storage bucket.
*   Creating a Firestore database and index.
*   Creating an Artifact Registry repository.

**To run the script:**

1.  **Open the script:**
    ```bash
    nano gcloud-deploy.sh
    ```
2.  **Configure the variables:**
    Update the following variables at the top of the script with your project details:
    *   `PROJECT_ID`
    *   `REGION`
    *   `FIRESTORE_LOCATION`

3.  **Make the script executable:**
    ```bash
    chmod +x gcloud-deploy.sh
    ```

4.  **Run the script:**
    ```bash
    ./gcloud-deploy.sh
    ```

### 2. Application Deployment

The `gcloud-run-build.sh` script handles building the Docker image with Cloud Build and deploying the application to Cloud Run.

**To use the script:**

The script provides several options for different actions. Here are the most common use cases:

*  **OAuth 2.0 Client ID:** Create an OAuth 2.0 Client ID in the Google Cloud Console.
    *   Go to the [Credentials page](https://console.cloud.google.com/apis/credentials).
    *   Click "Create Credentials" and select "OAuth client ID".
    *   Choose "Web application" as the application type.
    *   Add an authorized JavaScript origin. This will be the URL of your deployed Cloud Run service. The format is `https://{service-name}-{project-number}.{location}.run.app`.
    *   Add an authorized redirect URI. This will be the URL of your deployed Cloud Run service, with the path `/api/auth/callback/google`. The format is `https://{service-name}-{project-number}.{location}.run.app/api/auth/callback/google`.
    *   After creation, note the **Client ID** and **Client secret** in the download json file. These will be used in the application deployment.

*  **Configure the variables:**
    Update the following variables at the top of the script with your project details:
    *   `AUTH_GOOGLE_ID` (AUTH_GOOGLE_ID): used the **Client ID** created in the step **OAuth 2.0 Client ID:**
    *   `AUTH_GOOGLE_SECRET` (AUTH_GOOGLE_SECRET): used the **Client secret** created in the step **OAuth 2.0 Client ID:**
    *   `NEXTAUTH_SECRET` (generate a secure secret using `openssl rand -base64 32`)

*   **Build and Deploy:**
    This command will trigger a Cloud Build to build the Docker image and then deploy it to Cloud Run.
    ```bash
    ./gcloud-run-build.sh --build --deploy
    ```

*   **Build Only:**
    If you only want to build the Docker image without deploying, use:
    ```bash
    ./gcloud-run-build.sh --build
    ```

*   **List Images:**
    To see a list of the images you have built, use:
    ```bash
    ./gcloud-run-build.sh --list
    ```

**Important:**

*   The `gcloud-run-build.sh` script will use the currently configured `gcloud` project. You can override this with the `-p` or `--project` flag.
*   The script has default values for the region, repository name, image name, and service name. You can override these using the corresponding flags (`-l`, `-r`, `-i`, `-s`). For more details, run `./gcloud-run-build.sh --help`.

## What Gets Created

This deployment process will create the following resources in your Google Cloud project:

*   **Cloud Run Service:** A fully managed service running the StoryCraft application.
*   **Cloud Build:** Used to build the Docker image.
*   **Artifact Registry:** A repository to store the Docker images.
*   **Cloud Storage Bucket:** For storing application assets.
*   **Firestore Database:** A NoSQL database for application data.
*   **Service Account:** An identity for the application with specific IAM roles.

## Cleanup

To remove the resources created by this deployment, you can delete the Google Cloud project or manually delete the individual resources.
