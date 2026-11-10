# ☁️ Cloud-Native Fullstack Application Platform

## Overview

This project builds and operates a production-grade, cloud-native fullstack application platform on AWS.

The main focus isn't just the application—it's establishing the **infrastructure, automation, and engineering practices** required to operate a modern production platform.

The platform is built around Infrastructure as Code, self-managed Kubernetes, GitOps, and reproducible development environments. It automates the full lifecycle, from AWS infrastructure provisioning and Kubernetes cluster bootstrap to continuous deployment across isolated environments.

Kubernetes runs directly on **AWS EC2**, without Amazon EKS. Cluster provisioning and lifecycle management are handled using Terraform, Ansible, and kubeadm.

---

## 🎯 Key Goals

### 🏗️ Infrastructure & Automation

* **AWS Organizations:** Multi-account architecture with separate Production and Non-Production workloads.
* **Terraform:** Modular infrastructure provisioning across AWS accounts and regions.
* **Ansible & cloud-init:** Automated EC2 configuration and Kubernetes node preparation.
* **Kubernetes:** Self-managed Kubernetes clusters running on AWS EC2, bootstrapped using kubeadm.
* **Networking:** AWS VPC, private subnets, security groups, internal Network Load Balancer for the Kubernetes API, and Kubernetes CNI networking.
* **DNS & Routing:** Managed through Cloudflare and AWS Application Load Balancers.

### 💻 Application Architecture

A simple fullstack application consisting of:

* **Backend:** Go (also serves the web frontend).
* **Database:** PostgreSQL hosted on Amazon RDS.
* **Storage:** Amazon S3 for image storage.

**Workflow:** User logs in with a username ➔ views associated images ➔ uploads new images ➔ downloads existing images.

*(Authentication and authorization are intentionally outside the initial platform scope.)*

### ☸️ Kubernetes Platform

The platform operates two independent Kubernetes clusters:

* **Non-Production:** Development, staging, and ephemeral preview environments.
* **Production:** Dedicated production cluster with a highly available control plane and distributed worker nodes.

Core platform components:

* **Cluster Bootstrap:** kubeadm.
* **Container Runtime:** containerd.
* **Control Plane:** Self-managed Kubernetes API servers and etcd running on EC2.
* **Networking:** Cilium or Calico for pod networking and NetworkPolicy enforcement.
* **Persistent Storage:** AWS EBS CSI Driver.
* **Ingress:** AWS Load Balancer Controller.
* **Workload Identity:** Kubernetes OIDC federation with AWS IAM and STS.
* **Cluster Management:** Terraform, Ansible, and automated node lifecycle procedures.

Production uses three control-plane nodes distributed across three Availability Zones, maintaining an etcd quorum. Non-Production initially uses a single control-plane node as a cost-conscious trade-off.

### 🚀 CI/CD & GitOps

* **Builds & Scanning:** Nix for reproducible container builds; automated linting, testing, dependency checks, and security scanning.
* **Container Registry:** Amazon ECR for versioned container images.
* **Infrastructure:** Terraform executed through GitHub Actions using GitHub OIDC and temporary AWS credentials.
* **Cluster Bootstrap:** Automated EC2 configuration and Kubernetes installation using Ansible, cloud-init, and kubeadm.
* **Deployment:** Argo CD for Kubernetes reconciliation and Helm for application packaging.
* **Environments:** Version-controlled configurations with automated deployment of feature branches into ephemeral preview environments.
* **Production Promotion:** Explicit approval workflow and immutable container image digests.

---

## 🌍 Environments

* 🧪 **Development:** Persistent development environment with shared PostgreSQL and S3 resources. Lower availability requirements.
* 🌿 **Staging / Preview:** Staging and ephemeral environments deployed into the Non-Production cluster. Preview environments receive unique URLs and are cleaned up automatically when their associated pull requests are closed.
* 🏭 **Production:** Dedicated AWS account and Kubernetes cluster. Highly available control plane, multi-replica application deployment, isolated infrastructure and data, automated health checks, database backups, and alerting.

Production and Non-Production are isolated at the AWS account, VPC, Kubernetes cluster, database, and IAM levels.

---

## 🔭 Observability

End-to-end visibility across application metrics, Kubernetes workloads, self-managed control-plane components, and AWS infrastructure.

* 📈 **Metrics:** Prometheus & Grafana.
* 🚨 **Alerting:** Alertmanager.
* 🪵 **Logs:** Loki.
* ⏱️ **Tracing:** OpenTelemetry & Tempo.
* 🖥️ **Infrastructure Monitoring:** AWS CloudWatch, EC2 metrics, EBS metrics, and load balancer monitoring.
* ☸️ **Kubernetes Monitoring:** kube-state-metrics, node metrics, API server metrics, and etcd health monitoring.

Grafana is accessible through private network access rather than public internet exposure.

---

## 💻 Developer Experience

* ❄️ **Nix Flakes & direnv:** Declarative dependencies and automatic development environment loading.
* 🐳 **Docker Compose:** Local development with connectivity to AWS development resources.
* 🔄 **Build Parity:** Identical container definitions used locally and in CI.
* 🧰 **Infrastructure Automation:** Reproducible Terraform and Ansible workflows.
* 📦 **GitOps:** Kubernetes application and platform configuration maintained in Git.

---

## 🛡️ Security

* **Access Control:** Short-lived AWS credentials through GitHub OIDC and IAM roles. Kubernetes access controlled through authentication, RBAC, and private API endpoints.
* **Workload Identity:** Kubernetes OIDC federation and AWS STS `AssumeRoleWithWebIdentity` for pod-level AWS permissions.
* **Isolation:** Production and Non-Production separated across distinct AWS accounts, VPCs, and Kubernetes clusters.
* **Network Security:** Private EC2 instances, restricted security groups, Kubernetes NetworkPolicies, and private database subnets.
* **Data Protection:** Encryption at rest and in transit using AWS KMS and TLS.
* **Secrets Management:** AWS Secrets Manager and External Secrets Operator.
* **Cluster Security:** Hardened EC2 nodes, restricted etcd access, Kubernetes audit logging, and controlled certificate rotation.
* **Supply Chain Security:** Continuous vulnerability scanning, SBOM generation, image signing, and provenance verification.

---

## 🔄 Backup & Disaster Recovery

* **PostgreSQL:** Amazon RDS Multi-AZ, automated backups, point-in-time recovery, and cross-region backup replication.
* **S3:** Versioning and cross-region replication for application data.
* **Container Images:** Amazon ECR cross-region replication.
* **Terraform State:** Encrypted, versioned S3 backend with cross-region replication.
* **Kubernetes:** etcd snapshots, version-controlled cluster configuration, and repeatable kubeadm-based cluster reconstruction.
* **Regional Recovery:** Infrastructure and Kubernetes clusters rebuilt in AWS Ireland (`eu-west-1`) using Terraform and automated bootstrap workflows.

The primary deployment region is Frankfurt (`eu-central-1`), with Ireland (`eu-west-1`) serving as the disaster recovery region.

---

## 🧭 Architecture Principles

1. Infrastructure as Code wherever possible.
2. Production and Non-Production isolation.
3. Self-managed Kubernetes on AWS EC2, without EKS.
4. Git as the source of truth for Kubernetes desired state.
5. Temporary credentials and least-privilege IAM permissions.
6. Stateless application workloads wherever possible.
7. Automated and repeatable infrastructure and cluster provisioning.
8. Tested backup, recovery, and disaster recovery procedures.
9. Prefer simplicity over premature distributed-system complexity.
10. Measure actual resource utilization and costs before scaling.

**The goal:** Build a production-grade AWS platform while gaining hands-on experience operating Kubernetes, infrastructure automation, networking, security, observability, and GitOps.
