# AWS CI/CD Pipeline with GitHub Actions, Docker, Terraform, and Amazon ECR

## Project Overview

This project demonstrates an automated CI/CD pipeline for a Python Flask application. GitHub Actions runs automated tests, validates Terraform configuration, builds a Docker image, and publishes the image to Amazon Elastic Container Registry (ECR).

GitHub Actions uses OpenID Connect (OIDC) to authenticate to AWS without storing long-lived AWS access keys in GitHub repository secrets.

## Architecture

```text
Developer pushes code to GitHub
             |
             v
      GitHub Actions
             |
     +-------+--------+
     |                |
     v                v
 Python tests    Terraform validation
     |                |
     +-------+--------+
             |
             v
      Build Docker image
             |
             v
   GitHub OIDC authentication
             |
             v
         Amazon ECR
             |
             v
       Publish image
       - latest tag
       - commit SHA tag
```

## Technologies Used

- **Python 3.13** — application runtime
- **Flask** — web application framework
- **pytest** — automated testing
- **Docker** — application containerization
- **GitHub Actions** — CI/CD automation
- **Terraform** — infrastructure as code and configuration validation
- **AWS IAM and OIDC** — short-lived authentication for GitHub Actions
- **Amazon ECR** — private container image registry
- **AWS CLI** — AWS resource inspection
- **Git and GitHub** — source control and collaboration

## Repository Structure

```text
cicd-pipeline-aws/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── terraform.yml
├── app/
│   └── main.py
├── infra/
│   ├── main.tf
│   ├── github-actions-trust-policy.json
│   └── github-actions-ecr-policy.json
├── tests/
│   └── test_app.py
├── screenshots/
├── .gitignore
├── Dockerfile
├── requirements.txt
└── README.md
```

The `ci.yml` workflow tests the application and builds and publishes the Docker image. The `terraform.yml` workflow validates the Terraform configuration. The `infra/` directory contains the ECR infrastructure configuration and IAM policy documents.

## CI/CD Pipeline

The pipeline runs on pushes to the `main` branch and pull requests targeting `main`.

### 1. Automated Testing

- Sets up Python 3.13.
- Installs application dependencies and pytest.
- Runs the test suite using `python -m pytest -v`.

### 2. Terraform Validation

- Sets up Terraform.
- Checks Terraform formatting.
- Initializes Terraform without configuring a remote backend.
- Validates the infrastructure configuration.

### 3. Docker Image Publishing

After the required checks succeed on a push to `main`, the pipeline:

- Builds the Docker image.
- Authenticates to AWS using GitHub Actions OIDC.
- Logs in to Amazon ECR.
- Publishes the image to the ECR repository.
- Tags the image with `latest` and the commit SHA.

## AWS Configuration

- **Region:** `us-east-1`
- **ECR repository:** `cicd-pipeline-aws`
- **Image URI:** `823963318359.dkr.ecr.us-east-1.amazonaws.com/cicd-pipeline-aws`
- **Authentication:** GitHub Actions OIDC with an IAM role

The ECR repository uses mutable tags, image scanning on push, and AES-256 encryption.

## Security Considerations

- Uses OIDC instead of storing long-lived AWS access keys in GitHub.
- Restricts the workflow's IAM trust relationship to the intended GitHub repository and branch identity.
- Scopes ECR image operations to the project repository where applicable.
- Excludes Terraform state files and variable files through `.gitignore`.
- Keeps AWS credentials and other secrets out of source control.
- Uses an Alpine-based Python image and verifies the published image with Amazon ECR vulnerability scanning. The scan completed on October 14, 2026, and reported zero Critical, High, Medium, and Low findings.

The IAM trust policy and permissions should be reviewed periodically as the project evolves. Future scans may identify newly disclosed vulnerabilities.

## Verification Results

The following checks have passed during project development:

- Python automated tests passed in GitHub Actions.
- Terraform formatting and validation passed.
- Docker image built successfully and its health endpoint returned a healthy status.
- The main application endpoint returned the expected application response.
- GitHub Actions successfully published the image to Amazon ECR.
- The `latest` tag and commit-specific tag were verified against the same image digest.
- The latest recorded ECR vulnerability scan reported zero findings across Critical, High, Medium, and Low categories.
- The GitHub repository was synchronized with the local working tree.

## Scope and Future Improvements

This project demonstrates continuous integration, infrastructure validation, containerization, security scanning, and container image publishing. It does not currently deploy the application to a running production environment.

Potential improvements include:

- Deploying the container to Amazon ECS with AWS Fargate.
- Adding deployment approval gates and rollback procedures.
- Adding an image retention policy to manage ECR storage costs.
- Adding monitoring and operational alerts for a deployed application.

## Learning Outcomes

This project provided hands-on experience with Python testing, Docker containerization, Terraform validation, GitHub Actions workflows, AWS IAM, OIDC federation, Amazon ECR, vulnerability scanning, and troubleshooting cloud automation.

## Screenshots

### GitHub Actions Workflow

The CI/CD pipeline successfully runs its configured checks and publishes the Docker image.

![Successful GitHub Actions workflow](screenshots/cicd-test-successful.png)

### Automated Python Tests

The Python application tests pass successfully.

![Successful automated Python tests](screenshots/pytest-success.png)

---

**Author:** John White

**GitHub Repository:** [cicd-pipeline-aws](https://github.com/Johnwhite1212/cicd-pipeline-aws)
