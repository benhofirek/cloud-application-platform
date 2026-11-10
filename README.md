# ☁️ Cloud-Native Fullstack Application Platform

## Overview

This project builds and operates a production-grade, cloud-native fullstack application on AWS.

The main focus isn't just the app—it's establishing the **infrastructure, automation, and engineering practices** needed to operate a modern platform. It uses Infrastructure as Code, GitOps, Kubernetes, and reproducible dev environments to automate the full lifecycle from cloud provisioning to continuous deployment across isolated environments.

---

## 🎯 Key Goals

### 🏗️ Infrastructure & Automation

* **Terraform:** Provision and manage infrastructure modularly across a multi-account AWS architecture.
* **DNS & Routing:** Managed via Cloudflare.
* **Kubernetes:** Orchestrate workloads across independent Development, Staging, and Production environments.

### 💻 Application Architecture

A simple fullstack app consisting of:

* **Backend:** Go (also serves web frontend)
* **Database:** PostgreSQL
* **Storage:** Amazon S3 (image storage)

**Workflow:** User logs in with a username ➔ views associated images ➔ uploads new images ➔ downloads existing images. *(Auth/Authz intentionally out of initial scope).*

### 🚀 CI/CD & GitOps

* **Builds & Scanning:** Nix for container builds; automated linting, testing, and security scanning.
* **Deployment:** Argo CD for Kubernetes reconciliation and Helm for packaging.
* **Environments:** Version-controlled configs with automated deployment of feature branches into ephemeral preview environments.

---

## 🌍 Environments

* 🧪 **Development:** Persistent, shared PostgreSQL & S3 bucket, lower availability requirements.
* 🌿 **Staging / Preview:** Ephemeral environments spawned by git deployment tags, assigned unique URLs, and cleaned up automatically.
* 🏭 **Production:** High availability, multi-replica, fully isolated data/infrastructure, automated health checks, DB backups, and alerting.

---

## 🔭 Observability

End-to-end visibility across application metrics, Kubernetes workloads, and AWS services using:

* 📈 **Metrics:** Prometheus & Grafana
* 🚨 **Alerting:** Alertmanager
* 🪵 **Logs:** Loki
* ⏱️ **Tracing:** OpenTelemetry & Tempo

---

## 💻 Developer Experience

* ❄️ **Nix Flakes & direnv:** Declarative dependencies and automatic environment loading.
* 🐳 **Docker Compose:** Local development with direct connectivity to AWS dev resources.
* 🔄 **Build Parity:** Identical container definitions used locally and in CI.

---

## 🛡️ Security

* **Access Control:** Short-lived credentials via IAM roles and Kubernetes Workload Identity.
* **Isolation:** Environment separation across distinct AWS accounts; strict least-privilege policies.
* **Data Protection:** Encryption at rest and in transit; secrets managed via AWS Secrets Manager.
* **Scanning:** Continuous vulnerability scanning for containers and dependencies.
