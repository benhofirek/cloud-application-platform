## 1. Repository and AWS foundations
1. Create the GitHub repository and protect the default branch.
2. Create the repository structure: terraform/, ansible/, kubernetes/, helm/, argocd/, docs/.
3. Add architecture diagrams, README and this project checklist.
4. Configure AWS Organizations and the required OUs.
5. Create the Management account and configure billing and IAM Identity Center.
6. Create the Log Archive account for centralized CloudTrail and AWS Config logs.
7. Create the Audit account for Control Tower governance.
8. Create the Terraform State account.
9. Create the Non-Production account.
10. Create the Production account.
11. Enable Control Tower governance and baseline controls.
12. Configure MFA, administrator roles, break-glass access and least-privilege account access.
13. Configure SCP guardrails, organization CloudTrail, AWS Config and centralized log delivery.
14. Configure AWS budgets, billing alerts and initial service quotas in Frankfurt and Ireland.

## 2. Terraform delivery foundation
15. Create the controlled bootstrap process for the Terraform State account and initial GitHub OIDC trust.
16. Create the Terraform state S3 bucket with versioning, SSE-KMS, public-access blocking and native S3 lockfiles.
17. Configure cross-region replication of Terraform state to Ireland.
18. Configure Terraform backend access from GitHub Actions only.
19. Create separate Terraform roles for Non-Production and Production, with narrowly scoped cross-account trust.
20. Create GitHub Actions workflows for fmt, validate, plan and approved apply.
21. Enforce separate plan/apply permissions and Production environment approval.
22. Split Terraform into reusable modules and independent state files for accounts, networking, compute and data services.

## 3. Non-Production AWS infrastructure
23. Define the Non-Production VPC CIDR: 10.10.0.0/16.
24. Provision the Frankfurt VPC with two public, two private application and two isolated database subnets across two AZs.
25. Provision one NAT Gateway and configure private subnet outbound routing.
26. Configure route tables, subnet associations, VPC DNS and security groups.
27. Add the S3 Gateway endpoint and required interface VPC endpoints.
28. Provision the Non-Production EC2 node security groups, restricting Kubernetes API and node traffic to approved sources.
29. Provision the Non-Production KMS keys and policies.
30. Provision private ECR repositories and configure image scanning and lifecycle policies.

## 4. Non-Production Kubernetes cluster
31. Select the Linux distribution and ARM64 EC2 instance family; validate m7g.large compatibility.
32. Create Terraform modules for EC2 instances, EBS volumes, instance profiles and node security groups.
33. Provision one private control-plane EC2 instance and two private worker EC2 instances.
34. Configure SSM Session Manager for node administration; do not expose SSH publicly.
35. Create Ansible inventory from Terraform outputs or AWS dynamic inventory.
36. Create Ansible roles for OS updates, hostname, time synchronization, users, kernel modules and sysctl settings.
37. Configure containerd and Kubernetes prerequisites through Ansible.
38. Install and pin kubeadm, kubelet and kubectl versions.
39. Automate kubeadm init and configure a stable private Kubernetes API endpoint.
40. Install the selected CNI and verify pod-to-pod networking, DNS and NetworkPolicy enforcement.
41. Automate worker-node joining through Ansible.
42. Configure Kubernetes API access for administrators using the chosen OIDC authentication mechanism and RBAC.
43. Configure namespaces: development, staging and preview.
44. Apply default resource quotas, LimitRanges, Pod Security Standards and baseline NetworkPolicies.
45. Deploy a test application and verify service discovery, internal access, node failure recovery and cluster recreation.

## 5. Non-Production AWS integrations and application data
46. Provision the Non-Production RDS PostgreSQL instance in isolated database subnets.
47. Configure RDS encryption, backups, parameter settings, security groups and Secrets Manager credentials.
48. Provision separate private S3 buckets for application images and observability data.
49. Configure S3 Block Public Access, encryption, versioning, lifecycle policies, TLS-only access and ownership controls.
50. Install the AWS EBS CSI Driver and create an encrypted EBS StorageClass; verify PVC provisioning and AZ-aware volume attachment.
51. Install the AWS Load Balancer Controller and configure its IAM permissions.
52. Deploy the ingress path and verify the flow from an internet-facing ALB to Kubernetes ingress and the application.
53. Configure ACM certificates and Cloudflare DNS/proxying for the Non-Production application.
54. Restrict ALB origin access to Cloudflare IP ranges where proxying is enabled.
55. Configure the cluster's Kubernetes OIDC issuer and register it as an AWS IAM OIDC provider.
56. Give the application a dedicated Kubernetes service account and narrowly scoped IAM role using AssumeRoleWithWebIdentity.
57. Install External Secrets Operator and retrieve application credentials from Secrets Manager.
58. Deploy the Go application with access only to its environment's RDS and S3 resources; verify image upload and presigned image URLs.

## 6. Non-Production delivery and GitOps
59. Create the GitHub Actions application workflow for linting, tests, dependency checks and security scanning.
60. Configure Nix-based ARM64 image builds.
61. Configure GitHub OIDC authentication for image publishing; do not use long-lived AWS keys.
62. Push versioned images to Non-Production ECR and use immutable image digests.
63. Generate an SBOM, sign images and publish build provenance.
64. Create a Helm chart for the application, including probes, resource requests/limits, graceful shutdown and configuration.
65. Install Argo CD in the Non-Production cluster.
66. Connect Argo CD to the GitOps repository and deploy the application through Helm.
67. Configure automated sync, drift detection and documented rollback.
68. Configure ApplicationSets to create and remove preview-pr-<number> environments.
69. Provision unique preview database credentials and implement preview database cleanup on PR closure plus TTL cleanup.
70. Verify the complete PR-to-deployment flow without using CI to run kubectl against the cluster.

## 7. Non-Production observability and operational validation
71. Install Prometheus, Grafana and Alertmanager.
72. Install Loki and Tempo, using private S3 buckets for their object storage.
73. Install OpenTelemetry Collectors: a node-level DaemonSet and a gateway Deployment.
74. Instrument the Go application with OpenTelemetry and structured JSON logs containing trace identifiers.
75. Add kube-state-metrics, node metrics and PostgreSQL/connection-pool monitoring.
76. Configure EBS-backed persistent storage for Prometheus and Grafana.
77. Configure dashboards and alerts for application health, nodes, Kubernetes, database and telemetry pipeline failures.
78. Set Non-Production retention targets: metrics 7 days, logs 3–7 days and traces 3 days.
79. Restrict Grafana access to VPN users; configure CloudWatch infrastructure monitoring, audit logs and external HTTP uptime checks.
80. Run application and observability load tests; validate node sizing, HPA behaviour, resource headroom and monthly cost.

## 8. Production AWS infrastructure and cluster
81. Create the Production VPC with CIDR 10.20.0.0/16 in Frankfurt.
82. Provision three public, three private application and three isolated database subnets across three AZs.
83. Provision one NAT Gateway per AZ and configure AZ-local private routing.
84. Configure Production VPC endpoints, route tables, security groups, KMS keys and private DNS.
85. Provision Production ECR repositories and configure cross-region image replication to Ireland.
86. Provision the Production EC2 control-plane and worker nodes: three control-plane nodes and at least three workers, distributed across three AZs.
87. Provision an internal Network Load Balancer for the Kubernetes API on TCP/6443; configure health checks and the stable kubeadm control-plane endpoint/SANs.
88. Configure Production nodes through the same Ansible roles validated in Non-Production, with Production-specific variables and hardening.
89. Bootstrap the three-member etcd cluster and verify quorum, membership, peer connectivity and API availability.
90. Join the worker nodes and verify scheduling, pod networking, DNS and node/AZ failure behaviour.
91. Configure Production administrator authentication, restricted operator RBAC, break-glass access and audit logging.
92. Configure Production namespaces, resource quotas, NetworkPolicies, Pod Security Standards and workload isolation.
93. Configure the Production EBS CSI Driver, encrypted StorageClass and persistent-volume recovery tests.
94. Install the Production AWS Load Balancer Controller and verify ALB provisioning and target health.
95. Deploy the application with three replicas, topology spread across three AZs, readiness/liveness probes, graceful shutdown and a PDB with minAvailable: 2.

## 9. Production data, identity and traffic
96. Provision a separate Production RDS PostgreSQL Multi-AZ instance in isolated database subnets, with KMS encryption, deletion protection and 35-day backup retention.
97. Enable RDS point-in-time recovery, Secrets Manager credentials and cross-region automated backup replication to Ireland.
98. Provision separate private Production S3 buckets for application images and observability; configure encryption, versioning, lifecycle policies and cross-region replication with a destination-region KMS key.
99. Configure Production workload identity through Kubernetes OIDC and dedicated IAM roles; restrict the application to its own Production S3 resources and explicitly required AWS permissions.
100. Configure External Secrets Operator, Cloudflare DNS/proxying, ACM certificates and the Production ALB; restrict the ALB origin to Cloudflare where proxying is enabled.

## 10. Production delivery, observability and security
101. Create a protected GitHub Production environment with required human approval and separate Production deployment permissions.
102. Implement explicit Non-Production-to-Production promotion of the already-tested, signed image digest; do not rebuild a different image for Production.
103. Configure Production Argo CD and Helm values; ensure CI updates Git rather than deploying directly with kubectl.
104. Configure Production HPA, three-replica availability, topology spread, PDBs and application resource limits; validate behaviour under load.
105. Deploy the Production Prometheus, Grafana, Loki, Tempo, OpenTelemetry and Alertmanager stack.
106. Configure Production retention targets: metrics 15 days, logs 14 days and traces 7 days; store Loki and Tempo data in private S3.
107. Configure node-level OTel collection, gateway collection, application trace correlation, PostgreSQL metrics and connection-pool monitoring.
108. Configure CloudWatch infrastructure monitoring, Kubernetes API audit logs, external HTTP uptime checks and alerts with named owners and escalation procedures.
109. Apply Production security controls: SCPs, IAM least privilege, restricted Production access, KMS separation, secret isolation, image scanning, SBOM, signing, provenance and dependency/secret scanning.
110. Validate the Production application path, database connectivity, image uploads, presigned URLs, Cloudflare origin protection and failure behaviour across AZs.

## 11. Backup, disaster recovery and operations
111. Automate encrypted etcd snapshots and store them in a protected backup location; document snapshot retention and access.
112. Test restoring etcd from a snapshot and recovering the control plane.
113. Configure Terraform state recovery from the replicated Ireland backend, including a documented and controlled backend-promotion procedure.
114. Verify RDS backup replication, restore a database in Frankfurt and test the application against the restored database.
115. Perform a database restore test monthly and a DR-region database restore test quarterly.
116. Verify S3 cross-region replication, destination KMS access and replication-failure monitoring.
117. Provision Ireland DR networking and recovery prerequisites through Terraform; do not run a permanent duplicate Production cluster.
118. Implement the Ireland recovery automation: provision EC2, configure nodes, bootstrap Kubernetes with kubeadm, install platform components and deploy workloads through Argo CD.
119. Test restoration of RDS, S3 and ECR dependencies in Ireland; validate the recovered application before changing DNS.
120. Document and test manual Cloudflare failover to the recovered regional ALB, targeting a 60–120 second DNS TTL where supported.
121. Run quarterly failure/recovery tests and a full regional DR exercise every six months; measure actual RTO/RPO against the initial regional targets of 4–8 hours and 30 minutes.
122. Create operational runbooks for node replacement, etcd recovery, certificate renewal, database restoration, application rollback and regional recovery.
123. Establish a quarterly Kubernetes/add-on upgrade process: test in Non-Production first, pin versions, prepare rollback/roll-forward plans and roll nodes rather than modifying them in place.
124. Rehearse PostgreSQL major-version upgrades and database migration/rollback procedures separately.
125. Complete a Production Operational Readiness Review covering security, capacity, cost, backups, monitoring, incident ownership, upgrade procedures and DR evidence.
