# DevOps Screening Assignment

This repository contains the completed screening assignment for the Junior DevOps Engineer position. It includes a containerized Flask application, a multi-container Docker Compose setup, a GitHub Actions CI pipeline with image push to GHCR, and Terraform infrastructure code.

## 1. Local Setup

### Without Docker
1.  **Prerequisites**: Python 3.11 installed.
2.  **Environment Setup**:
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip install -r app/requirements.txt
    ```
3.  **Run Application**:
    ```bash
    python app/app.py
    ```
    The app will be available at [http://localhost:5000](http://localhost:5000).

### With Docker Compose
1.  **Run Services**:
    ```bash
    docker-compose up --build
    ```
    This will build the Flask application and pull the Redis image.
2.  **Access App**: [http://localhost:8080](http://localhost:8080).
    *Note: The application is mapped from container port 5000 to host port 8080.*

---

## 2. CI Pipeline
The pipeline is defined in `.github/workflows/ci.yml` and is triggered on every `push` to `main` and all `pull_requests`.

### Steps & Rationale:
- **Checkout Code**: Uses `actions/checkout@v4` to bring the repository content into the runner.
- **Setup Python 3.11**: Ensures the environment matches the production/container version.
- **Install Dependencies**: Installs the exact versions specified in `requirements.txt`.
- **Run Pytest**: Validates the application logic before building the image. If tests fail, the pipeline stops to prevent broken images from being created.
- **Log in to GHCR**: (Bonus) Uses `GITHUB_TOKEN` to authenticate with the GitHub Container Registry.
- **Build and Push**: (Bonus) Builds the container image and pushes it to GHCR (only on pushes to `main`).

---

## 3. Design Decisions
- **Base Image (`python:3.11-slim`)**: I chose the `slim` version to maintain a small image foot-print (reducing storage costs and pull times) while keeping the security surface area minimal.
- **Non-Root User**: In the `Dockerfile`, a user named `appuser` is created. Running applications as non-root is a critical security best practice to prevent potential container escape vulnerabilities from gaining root access to the host.
- **Layer Optimization**: The `requirements.txt` is copied and installed *before* the rest of the application code. This leverages Docker's layer caching; dependencies are only re-installed if the `requirements.txt` file changes.
- **Healthchecks in Compose**: The `app` service uses a `service_healthy` condition on `redis`. This ensures the application layer doesn't start until the backend database (Redis) is fully responsive.

---

## 4. What I'd Improve
- **Multi-Stage Builds**: For an even smaller production image, I would use a multi-stage build to compile dependencies in one stage and copy only the final artifacts to a cleaner, separate runner stage.
- **Secrets Management**: Currently, sensitive information like the `GITHUB_TOKEN` is handled by GitHub Actions, but for the terraform/cloud setup, I would integrate AWS Secrets Manager or HashiCorp Vault.
- **Infrastructure State**: For the Terraform component, I would implement a remote backend (e.g., S3 with DynamoDB locking) to ensure state consistency in a team environment.
- **Expanded Testing**: I would add integration tests to verify the connectivity between the Flask app and Redis in the containerized environment.
