# 🚀 Multi-Stage GitOps Pipeline with Continuous Promotion using Kargo by Akuity! 

This project demonstrates an end-to-end DevOps pipeline that automates the process of building, promoting, and deploying containerized applications to a **Kubernetes** cluster across **development**, **staging**, and **production** environments. It leverages modern tools and methodologies, including:

- **GitHub Actions** for Continuous Integration (CI)
- **Kargo by Akuity** for orchestrating Continuous Promotion
- **Argo CD** for GitOps-based Continuous Delivery
- **Kustomize** for managing environment-specific Kubernetes configurations
- **Kubernetes** as the underlying container orchestration platform
- **GitOps** as the methodology to drive deployments declaratively via Git  

The entire pipeline is structured to ensure that every code change is built, scanned for vulnerabilities, promoted through validation stages, and finally deployed to a Kubernetes cluster—without manual intervention.

### Key Flow Summary:

1. A developer commits code to a GitHub repository.
2. **GitHub Actions** is triggered and performs the following:
   - Builds a Docker image from the code.
   - Scans the image with **Trivy** for vulnerabilities.
   - Pushes the verified image to **Docker Hub**.
3. **Kargo** detects the new image in Docker Hub via a warehouse subscription and adds it to freight.
4. The freight becomes eligible for promotion through the **dev**, **staging**, and **prod** environments, validated using HTTP checks defined in **analysis templates**.
5. On promotion, **Kargo** updates the manifests in the GitOps repository using **Kustomize** overlays.
6. **Argo CD** detects changes in the GitOps repository and syncs the updated manifests with the appropriate **Kubernetes** namespace.

This seamless integration of tools ensures fast, secure, and reliable delivery of applications on Kubernetes.

![Pipeline Diagram](/images/kargo-diagram.png)

## Table of Contents
1. [Continuous Integration (CI)](#continuous-integration-ci)
2. [Continuous Promotion](#continuous-promotion)
3. [Continuous Delivery (CD)](#continuous-delivery-cd)
4. [Links to Official Documentation](#links-to-official-documentation)

---

## Continuous Integration (CI)

**Continuous Integration (CI)** is the first step in automating the deployment pipeline. In this step, the code is built and tested automatically every time a developer pushes changes to the repository.

1. **GitHub Actions** triggers a build pipeline when the developer commits code to the repository.
2. **Docker Image Build**: The project is packaged into a Docker image using a Dockerfile.
3. **Trivy Scan**: The image is scanned for vulnerabilities using **Trivy**.
4. **Push to Docker Hub**: After a successful build and scan, the Docker image is pushed to **Docker Hub** for storage and distribution.

---

## Continuous Promotion

**Continuous Promotion** refers to the automated process of promoting Docker images through multiple stages (development, staging, and production) based on validation. This ensures that images are thoroughly tested and validated before they reach the production environment.

### Key Concepts in Continuous Promotion

1. **Warehouse**:  
   The **Kargo** tool by Akuity manages the promotion of Docker images using a **warehouse** system. A **warehouse** subscribes to the Docker image registry (Docker Hub in this case) to check for new image tags. Once a new image is detected, it is added to **freight**, which represents the image that will be promoted across different environments.

2. **Freight Eligibility**:  
   When an image is tagged and added to **freight**, it doesn't move directly to the next environment. Instead, it becomes **eligible** for promotion to the next environment (dev → staging → production). This eligibility is determined after successful validation in the previous environment.

3. **Promotion Process**:  
   The image is **promoted** through the following stages:
   - **Development**: The freight is tested in the development environment. Once it passes testing, it becomes eligible for promotion to staging.
   - **Staging**: The freight is tested in staging after successful validation in the dev environment. If it passes staging validation, it becomes eligible for promotion to production.
   - **Production**: The freight is deployed to production after successful verification in the staging environment.

   Only validated freight progresses from one stage to the next, ensuring a safe and consistent promotion process.

4. **Promotion Steps**:  
   The promotion process involves a series of steps that update the freight across environments. Here’s the breakdown:

    1. **Git Clone**:  
    The GitOps repo is cloned with two branches: one for the source (`main`) and one temporary branch to update the overlay.

    2. **Update Image with Kustomize**:  
    The Docker image tag is updated in the appropriate `kustomization.yaml` overlay (e.g., for staging).

    3. **Commit & Push Changes**:  
    The update is committed and pushed to the temporary branch.

    4. **Open Pull Request**:  
    A PR is created from the temporary branch to `main` so changes can be reviewed.

    5. **Wait for PR Merge**:  
    Kargo waits for the PR to be approved and merged.

    6. **Trigger Argo CD Sync**:  
    Once merged, Argo CD syncs the Kubernetes manifests to apply the new image in the cluster.

    7. **Run Analysis Template**:  
    After deployment, Kargo runs an analysis template that performs an HTTP check to validate the health and availability of the application. Only if this check succeeds does the freight become eligible for promotion to the next stage.

   ![Kargo Dashboard](/images/kargo-dashboard.png)

---

## Continuous Delivery (CD)

**Continuous Delivery (CD)** ensures that code changes are automatically deployed to the appropriate environment (development, staging, or production) after passing all promotion steps.

This project embraces the **GitOps** methodology, where the desired state of the application is defined and stored in a Git repository. All changes to infrastructure or application deployment are made via pull requests and then automatically synchronized with the running Kubernetes cluster.

### Argo CD for GitOps
- **Argo CD** watches the GitOps repository for changes.
- When a new commit is pushed (e.g., with an updated image tag), Argo CD detects it and automatically applies it to the cluster.
- This ensures the actual state in Kubernetes always matches the desired state defined in Git.

### Kustomize for Overlay Management
- This project uses **Kustomize** to manage Kubernetes configurations across multiple environments (dev, staging, prod).
- Each environment has its own overlay, which allows custom configuration (like resource limits, image tags, environment variables) per environment without duplicating the entire manifest.
- Kargo updates the image tag only in the environment-specific overlay (e.g., `overlays/staging/kustomization.yaml`).

### App of Apps Pattern
- The Argo CD instance in this project follows the **App of Apps** pattern.
- A single root Argo CD application manages and bootstraps child applications.
- This approach simplifies the management of multiple microservices or environments by organizing them under one parent application.

   ![ArgoCD Dashboard](/images/argo-dashboard.png)

## Links to Official Documentation

- [GitHub Actions](https://docs.github.com/en/actions)
- [Trivy](https://trivy.dev/v0.55/)
- [Kargo by Akuity](https://docs.kargo.io/)
- [Kustomize](https://kustomize.io/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)
- [Kubernetes](https://kubernetes.io/docs/home/)
