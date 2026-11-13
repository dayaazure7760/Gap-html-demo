# Deploy a Static Website to Google Cloud Run

This repository deploys a static HTML website to **Google Cloud Run**. The site is served by Nginx in a Docker container. **GitHub Actions** builds and scans the image, pushes it to **Google Artifact Registry**, and uses **Terraform** to deploy the Cloud Run service.

## Project structure

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml
├── terraform/
│   ├── main.tf
│   ├── output.tf
│   ├── provider.tf
│   ├── variable.tf
│   └── terraform.tfvars
├── Dockerfile
└── GAP-Nexus_prototype_v2.8.6.1.html
```

The Dockerfile copies `GAP-Nexus_prototype_v2.8.6.1.html` into the Nginx web root as `index.html`. Nginx listens on port **80**, which is also the container port configured for Cloud Run in Terraform.

## Deployment prerequisites

The workflow currently uses these Google Cloud settings, defined in `.github/workflows/deploy.yml`:

- Project: `gap-html`
- Region: `us-central1`
- Artifact Registry repository: `gap-html-repo`
- Image: `gap-html-image`
- Cloud Run service: `html-service`

Before deploying, make sure:

1. The Google Cloud project exists and billing is enabled.
2. The Artifact Registry Docker repository `gap-html-repo` exists in `us-central1`.
3. The Google Cloud Storage bucket `test-terraform001-html-bucket` exists for Terraform's remote state. Its state prefix is `terraform/state`.
4. The GitHub Actions service account has permissions to push to Artifact Registry, deploy Cloud Run services, enable the APIs used by the workflow, and grant the Cloud Run invoker role.
5. Terraform 1.9.5 or later is installed if running Terraform locally.

The workflow enables the Cloud Run, Artifact Registry, Cloud Build, and IAM APIs.

## GitHub secret

In the repository, open **Settings → Secrets and variables → Actions** and add:

- `GOOGLE_APPLICATION_CREDENTIALS`: the service account key JSON contents used by `google-github-actions/auth`.

Do not commit the service account key to the repository.

## GitHub Actions deployment

The workflow in `.github/workflows/deploy.yml` runs for pushes to `main`, pull requests targeting `main`, and manual dispatch. It:

1. Checks out the repository and scans the source with Trivy.
2. Builds the Docker image and pushes commit-tagged and `latest` images to Artifact Registry.
3. Scans the built image with Trivy.
4. Initializes Terraform, previews the changes, and applies them to Cloud Run.

The Cloud Run service is configured to allow unauthenticated access. The workflow deploys on pull requests as currently configured; pull requests from forks do not receive repository secrets.

After a successful deployment, Terraform outputs the Cloud Run service URL as `cloud_run_url`.

## Custom domain (optional)

Custom domain mapping is not created by this repository's Terraform configuration. To use a domain, first deploy the service, then create a domain mapping in Google Cloud Console and add the DNS verification and routing records it provides at your DNS host. Google provisions HTTPS for the mapping after DNS is configured; propagation and certificate provisioning can take time.
