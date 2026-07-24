# Coworking Space Analytics Service

## Overview
This service provides business analysts with usage reports (daily check-ins and per-user visit counts) for the coworking space application, backed by a PostgreSQL database.

## Architecture
The analytics API is a Flask application containerized with Docker and deployed to AWS EKS. PostgreSQL runs as a separate Kubernetes deployment with a PersistentVolume for data durability. Configuration is split between a ConfigMap (non-sensitive values like DB host, port, and username) and a Secret (the database password), both referenced by the application's Deployment manifest.

## Build and Deployment Process
Docker images are built and pushed to Amazon ECR via an AWS CodeBuild pipeline defined in `buildspec.yml`, triggered automatically on changes to the connected GitHub repository. Each build tags the image using semantic versioning (e.g. `1.0.0`), which should be incremented for every release to keep image history traceable. The Kubernetes manifests in `deployments/` reference the ECR image URI directly, so deploying a new version means updating that URI's tag and reapplying the deployment file with `kubectl apply`.

## Releasing New Builds
To ship a new version: bump the semantic version in `buildspec.yml`, push the change to trigger CodeBuild, then update the image tag in `deployments/coworking.yaml` to match and run `kubectl apply -f deployments/coworking.yaml`. Kubernetes will perform a rolling update, using the liveness and readiness probes already defined to confirm the new pod is healthy before fully cutting over.

## Monitoring
CloudWatch Container Insights is enabled on the EKS cluster, capturing application logs and periodic health check output for ongoing operational visibility.

## Stand-Out Suggestions

**Resource allocation:** The deployment currently has no explicit CPU/memory requests or limits set; adding conservative values (e.g. 100m CPU / 128Mi memory requests) would let Kubernetes schedule pods more predictably and prevent one service from starving others on the node.

**Instance type:** A `t3.small` (or `t3.medium` under heavier load) burstable instance fits this workload well, since analytics queries and the health-check scheduler are lightweight and intermittent rather than sustained. Burstable instances are also notably cheaper than compute-optimized types for this access pattern.

**Cost savings:** Using Spot instances for the node group instead of On-Demand would substantially cut compute costs, since this analytics service can tolerate brief interruptions given its stateless, restartable pods. Scaling the node group down outside business hours, and deleting the EKS cluster entirely when not actively in use during development, further reduces spend.
