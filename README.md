# DevOps Deployment Automation — Terraform, AWS, Docker, and ECS

This project demonstrates an end-to-end DevOps deployment workflow using Terraform, AWS, Docker, and CI/CD automation. It provisions cloud infrastructure using Infrastructure-as-Code, containerizes a Django REST API, and deploys it to AWS container services through automated workflows.

The system is designed to showcase repeatable infrastructure provisioning, containerized application delivery, and deployment automation practices aligned with DevOps and SRE principles.

Originally built through guided DevOps training and extended into a portfolio project with additional documentation, architecture explanation, and operational run procedures.

---

## What This Project Demonstrates

- Infrastructure provisioning with Terraform
- Modular Infrastructure-as-Code structure
- Containerized Django REST API deployment
- Docker-based local development workflow
- AWS container service deployment (ECS)
- Container registry integration (ECR)
- CI/CD pipeline automation
- Remote Terraform execution via containers
- Secure AWS CLI authentication using aws-vault
- Reproducible environment setup

---

## Technology Stack

**Infrastructure & Cloud**
- Terraform
- AWS (ECS, ECR, IAM, Networking, RDS)

**Containers**
- Docker
- Docker Compose

**Application**
- Python
- Django REST Framework

**CI/CD**
- GitHub Actions

**Tooling**
- AWS CLI
- aws-vault

---

## Local Development

The application runs locally using Docker and Docker Compose for consistent cross-platform development.

### Requirements

- Docker Desktop

### Run Project Locally

Clone the repository and run:

```bash
docker compose up
```

Health endpoint:

```
http://127.0.0.1:8000/api/health-check/
```

---

## Create Django Superuser

To access the Django admin interface:

```bash
docker compose run --rm app sh -c "python manage.py createsuperuser"
```

Admin URL:

```
http://127.0.0.1:8000/admin
```

---

## Reset Local Storage

To remove all volumes and start fresh:

```bash
docker compose down --volumes
docker compose up
```

---

## AWS CLI Authentication

AWS CLI authentication is performed using **aws-vault** for improved credential security.

Authenticate:

```bash
aws-vault exec PROFILE --duration=8h
```

List available profiles:

```bash
aws-vault list
```
List configured AWS Vault profiles:

```bash
aws-vault list
```

---

## ECS Exec — Container Shell Access

ECS Exec can be used to run commands directly inside running containers for debugging and operational inspection.

Example shell access command:

```bash
aws ecs execute-command \
  --region REGION \
  --cluster CLUSTER_NAME \
  --task TASK_ID \
  --container CONTAINER_NAME \
  --interactive \
  --command "/bin/sh"
```

Replace:

- `REGION` — AWS region where the ECS cluster is deployed
- `CLUSTER_NAME` — ECS cluster name
- `TASK_ID` — running task ID
- `CONTAINER_NAME` — container name inside the task

This is useful for live troubleshooting and runtime inspection.

---

## Terraform Commands (Containerized Execution)

Terraform is executed through Docker to ensure consistent tooling versions and avoid local dependency drift.

**Run commands from the `infra/` directory after authenticating with aws-vault.**

General syntax:

```bash
docker compose run --rm terraform -chdir=TF_DIR COMMAND
```

Where:

- `TF_DIR` = Terraform directory (`setup` or `deploy`)
- `COMMAND` = Terraform command (`init`, `plan`, `apply`, etc.)

Example — run plan:

```bash
docker compose run --rm terraform -chdir=deploy plan
```

### Get Terraform Outputs

```bash
docker compose run --rm terraform -chdir=setup output
```

If output is marked sensitive:

```bash
docker compose run --rm terraform -chdir=setup output OUTPUT_NAME
```

---

## CI/CD Pipeline Variables — GitHub Actions

The CI/CD pipeline requires repository variables and secrets to enable automated builds and deployments.

Variables (non-secret):

- `AWS_ACCESS_KEY_ID` — CI deployment IAM user access key (Terraform output)
- `AWS_ACCOUNT_ID` — AWS account ID
- `DOCKERHUB_USER` — Docker Hub username
- `ECR_REPO_APP` — Application image repository URL
- `ECR_REPO_PROXY` — Proxy image repository URL

Secrets (masked):

- `AWS_SECRET_ACCESS_KEY` — CI deployment IAM secret key
- `DOCKERHUB_TOKEN` — Docker Hub access token
- `TF_VAR_DB_PASSWORD` — Database password variable
- `TF_VAR_DJANGO_SECRET_KEY` — Django secret key variable

These values are configured in the repository CI/CD settings.

---

## CI/CD Pipeline Variables — GitLab (Optional)

If using GitLab CI/CD instead of GitHub Actions, equivalent variables must be configured in GitLab project settings.

Variables can be marked as:

- Masked — hidden in logs
- Protected — restricted to protected branches

Only required when running the GitLab pipeline variant.
Repository CI/CD variables must be configured with appropriate masking and protection settings depending on platform.

Required variables typically include:

- `AWS_ACCESS_KEY_ID` — CI deployment IAM access key (Terraform output)
- `AWS_ACCOUNT_ID` — AWS account identifier
- `DOCKERHUB_USER` — Docker Hub username
- `ECR_REPO_APP` — Application image repository URL
- `ECR_REPO_PROXY` — Proxy image repository URL
- `AWS_SECRET_ACCESS_KEY` (masked) — CI deployment IAM secret key
- `DOCKERHUB_TOKEN` (masked) — Docker Hub access token
- `TF_VAR_db_password` (masked) — Database password variable
- `TF_VAR_django_secret_key` (masked/protected) — Django secret key variable

Always store sensitive values as masked secrets in CI/CD settings.

---

## Environment & Tooling Verification

Use the following commands to verify required tooling is installed and available:

Check Docker:

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose --version
```

Check aws-vault:

```bash
aws-vault --version
```

Check AWS CLI:

```bash
aws --version
```

Check Session Manager plugin:

```bash
session-manager-plugin
```

---

## Git Configuration (Optional)

If setting up a new development environment:

```bash
git config --global user.email "your-email@example.com"
git config --global user.name "Your Name"
git config --global push.autoSetupRemote true
```

---

## Cost & Deployment Notes

This infrastructure can exceed AWS free-tier limits depending on configuration choices.

For cost-controlled testing:

- use smaller instance sizes
- reduce replica counts
- disable optional components
- destroy environments after validation using Terraform

Infrastructure is designed to be reproducible and safely recreated.

---

## Future Improvements

Planned or possible enhancements:

- Autoscaling policies
- Monitoring dashboards and alerts
- Blue/green or canary deployments
- Secrets Manager integration
- Automated rollback stages
- Expanded observability stack

---
## Troubleshooting & Incident Notes

During CI/CD deployment, Terraform backend initialization failed with an S3 `HeadObject 403` error when reading remote state.

Root cause:
The IAM policy for the CI/CD user allowed access only to prefixed object paths:

- `tf-state-deploy/*`

But the Terraform backend key used the exact object name:

- `tf-state-deploy`

Because S3 object permissions are exact-match, this resulted in access being denied during Terraform state refresh in the pipeline.

Resolution:
- Verified CI identity using `aws sts get-caller-identity` inside GitHub Actions
- Reproduced failure using `aws s3api head-object`
- Inspected attached IAM policy versions via AWS CLI
- Updated IAM policy to allow both:
  - exact object key access
  - prefixed path access
- Created new policy version and set as default
- Re-ran pipeline successfully

This highlights the importance of exact S3 object ARN matching in Terraform backend IAM policies.
---
## Portfolio Context

This repository is maintained as a DevOps portfolio project demonstrating Infrastructure-as-Code, containerized application deployment, CI/CD automation, and cloud environment reproducibility practices.
