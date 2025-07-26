# 🚀 Platform Engineer / SRE Interview Prep Guide

## 📚 Searchable Topics (Use CTRL+F to Jump to Sections)

* Kubernetes Cheat Sheet for Technical Interviews
* Example Kubernetes Troubleshooting: 60/100 Pods in Error State
* Kubernetes Deployment Failing: Troubleshooting Guide
* Terraform Cheat Sheet for SRE/Platform Interviews
* GitLab CI/CD Cheat Sheet for SRE/Platform Interviews
* Monitoring & Logging Cheat Sheet (Prometheus, Grafana, ELK)
* Scripting Cheat Sheet (Python, Bash, PowerShell)
* Security & Networking Cheat Sheet (TLS, IAM, Secrets, Subnetting, Firewalls)
* Deployment Strategies Cheat Sheet (Blue/Green, Canary & More)

## 🧠 Kubernetes Cheat Sheet for Technical Interviews

Kubernetes is a container orchestration platform. Core components:

* **API Server**: Central access point (kubectl, clients)
* **etcd**: Stores all cluster state
* **Controller Manager**: Ensures current state = desired state
* **Scheduler**: Places pods on nodes
* **Kubelet**: Node agent that manages pods
* **Kube Proxy**: Handles virtual IPs and service load balancing

Advanced concepts:

* **Operators**: Extend K8s logic using CRDs + controllers
* **CRDs**: Custom resources (e.g., `KafkaCluster`)
* **Ingress Controllers**: Manage external traffic routing
* **Deployments**: Abstract pod updates, rolling changes

## 🚨 Example Kubernetes Troubleshooting: 60/100 Pods in Error State

### Step 1: Overview

```bash
kubectl get pods -A
kubectl get events -A
```

Look for:

* CrashLoopBackOff, OOMKilled, image issues
* Node pressure warnings

### Step 2: Logs & Status

```bash
kubectl describe pod <name>
kubectl logs <name>
```

Pattern match across namespaces, deployments.

### Step 3: Classify the Error

* **CrashLoopBackOff**: Crashing repeatedly → check readiness/liveness
* **ImagePullBackOff**: Bad tag, repo auth issue
* **OOMKilled**: Increase memory limits
* **Pending**: Not enough resources, taints

### Step 4: Node/Network

```bash
kubectl describe node <node>
kubectl exec -it <pod> -- curl <svc>
```

## 🧨 Kubernetes Deployment Failing: Troubleshooting Guide

### Checklist

* `kubectl describe deploy <name>`
* Check pod events + logs
* Validate ConfigMaps/Secrets
* DNS issues (check CoreDNS, Service endpoints)
* Rollback:

```bash
kubectl rollout undo deploy <name>
```

## 🧱 Terraform Cheat Sheet for SRE/Platform Interviews

Terraform is a powerful open-source tool used to **provision and manage infrastructure** declaratively across cloud providers. For SRE and Platform roles, it's often used to define infrastructure-as-code (IaC), manage multi-cloud deployments, and enforce consistency via automation.

### 📘 Key Concepts

| Concept     | Description                                          |
| ----------- | ---------------------------------------------------- |
| `resource`  | Defines a specific cloud resource, e.g. EC2, IAM, S3 |
| `variable`  | Parameterise configurations for reusability          |
| `output`    | Export values (e.g., public IP, KMS ARN)             |
| `module`    | Group related resources, reuse across projects       |
| `provider`  | Connects Terraform to AWS, Azure, GCP, etc.          |
| `backend`   | Where Terraform stores state (e.g., S3 + DynamoDB)   |
| `workspace` | Used to separate environments (e.g., dev, prod)      |
| `locals`    | Define calculated values for reuse inside configs    |
| `data`      | Fetch external info (e.g., existing VPCs, AMIs)      |

### 📦 Terraform Folder Structure (Recommended)

```
.
├── main.tf
├── variables.tf
├── outputs.tf
├── backend.tf
└── modules/
    ├── network/
    ├── iam/
    └── eks/
```

Use `terragrunt` to wrap this structure for:

* Multi-environment management
* Centralised backends
* DRY (`Don't Repeat Yourself`) configurations

### 🛠️ Common Terraform Commands

```bash
terraform init               # Initialise the working directory
terraform validate           # Syntax check
terraform plan -out=plan.tfplan   # Preview changes
terraform apply plan.tfplan       # Apply with pre-generated plan
terraform destroy            # Tear everything down
terraform fmt                # Format configs
terraform output             # View output values
terraform show               # View full state
```

### 🧪 Troubleshooting Scenarios

| Problem                    | What to Check / Fix                                             |
| -------------------------- | --------------------------------------------------------------- |
| 🔒 Locked state file       | `terraform force-unlock <lock-id>`                              |
| 🌀 Resource drift          | `terraform refresh`, compare against real infra                 |
| ❌ Plan fails               | Missing variables? Wrong provider version? Changed resources?   |
| ⚠️ State backend error     | Check S3 bucket policy, IAM role access, DynamoDB locking table |
| 🔁 Cyclical dependencies   | Use `depends_on` explicitly                                     |
| ☁️ Provider version issues | Pin versions in `required_providers` block                      |

### 🧰 Remote State Best Practices

* Store state in **S3**, lock with **DynamoDB**
* Encrypt state at rest (`bucket` with SSE or KMS)
* Use **workspaces** sparingly — prefer directory structure for envs
* Enable **versioning** on state bucket to recover accidental applies

### ✅ Terraform Best Practices

* Use **`terraform-docs`** to auto-generate module documentation
* Run `terraform plan` in CI/CD before every apply (e.g., via GitLab or GitHub Actions)
* Protect production applies behind approvals or pipelines
* Tag all resources with `env`, `team`, `owner`, `cost-center`
* Use `pre-commit` hooks with `tflint`, `tfsec`, and `checkov` for security scanning

### 🔁 Canary / Blue-Green with Terraform?

While Terraform is typically used for infra, you can:

* Use **feature flags** in variables to deploy different versions
* Combine with CI/CD (GitLab) to deploy green infra, test, then cut over
* Manage DNS, Load Balancer weights via Terraform for phased rollout

## 🚀 GitLab CI/CD Cheat Sheet for SRE/Platform Interviews

GitLab CI/CD is used to automate build/test/deploy using a `.gitlab-ci.yml`.

### Core Concepts

* **Stages**: Sequence (build → test → deploy)
* **Jobs**: Units of work (per stage)
* **Runners**: Where jobs execute (shared or custom)

```yaml
stages:
  - validate
  - plan
  - apply

terraform-plan:
  stage: plan
  script:
    - terraform init
    - terraform plan
  only:
    - merge_requests
```

### Features

* **Artifacts**: Persist files between stages
* **Environments**: Track deployments
* **Secrets/Vars**: Use CI/CD variables or vault integrations
* **Includes**: Reuse templates across projects

### Troubleshooting

| Problem           | Fix                                |
| ----------------- | ---------------------------------- |
| Job stuck         | Runner may be missing tags         |
| Secret missing    | Check CI/CD Variable scope         |
| Cache not working | Validate `paths:` and `key:` usage |

## 📊 Monitoring & Logging Cheat Sheet (Prometheus, Grafana, ELK)

Effective monitoring and observability help you detect, investigate, and fix problems before users are affected. You’ll often be expected to **instrument**, **visualise**, and **alert** on metrics/logs — especially in Kubernetes or multi-cloud environments.

### 🔭 Prometheus

#### ✅ What it is:

A pull-based, time-series metrics collector often used in Kubernetes and SRE platforms.

#### 🔧 Core Concepts:

* **Exporters** – expose metrics in Prometheus format (e.g., `node_exporter`, `kube-state-metrics`)
* **TSDB** – built-in storage engine
* **Pull model** – Prometheus scrapes endpoints on a schedule
* **PromQL** – query language for metrics and alerting

#### 🧪 Sample PromQL Queries:

```promql
# CPU usage per pod
rate(container_cpu_usage_seconds_total[5m])

# Request error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)

# Pod restart spike
increase(kube_pod_container_status_restarts_total[15m]) > 3
```

#### 📦 Kubernetes Integration:

* Helm charts deploy Prometheus + Alertmanager
* Use `ServiceMonitor` and `PodMonitor` CRDs (via Prometheus Operator)

#### 🚨 Alerting:

Set up **Alertmanager** for:

* CPU/memory saturation
* Pod crash loops
* Slow API responses
* Disk usage alerts

Send to: Slack, PagerDuty, Email

#### 🛠️ Common Troubleshooting:

| Problem          | Reason              | Fix                                     |
| ---------------- | ------------------- | --------------------------------------- |
| Metrics missing  | Target not scraping | Check ServiceMonitor or relabel configs |
| Gaps in data     | Pod/node restarts   | Increase retention, enable WAL replay   |
| Alert not firing | Query logic wrong   | Test with `PromQL` in console           |

### 📈 Grafana

#### ✅ What it is:

A dashboarding tool for visualising metrics, logs, traces (a full observability UI).

#### 💡 Features:

* Datasources: Prometheus, Loki, Elasticsearch, PostgreSQL
* Templated dashboards: dynamic filters
* Alerting (legacy and unified alerting)

#### 🧰 Use Cases:

* Live dashboard: CPU, memory, request rate
* On-call SRE: check latency, error rates, and saturation
* Dev teams: application-specific dashboards

#### 🚨 Troubleshooting:

| Problem               | Fix                                       |
| --------------------- | ----------------------------------------- |
| No data in panel      | Check query, datasource connection        |
| High panel load times | Use `rate()` or downsampled metrics       |
| Alert not triggering  | Review thresholds and alert state history |

### 📚 ELK Stack (Elasticsearch, Logstash, Kibana)

#### ✅ What it is:

A popular stack for **log aggregation, analysis, and search**.

* **Elasticsearch** – full-text index & storage
* **Logstash** – log pipeline (input, filter, output)
* **Kibana** – visualisation layer

📝 Often used via **Beats** (e.g., Filebeat, Metricbeat) to ship logs from Kubernetes nodes.

#### 🧰 Common Use Cases:

* View `stderr/stdout` logs from pods
* Filter logs by container name, namespace, correlation ID
* Detect error spikes, parse app logs (e.g. Java stack traces)

#### ⚙️ Indexing & Storage Tips:

* Use ILM (Index Lifecycle Management)
* Roll over indexes daily or based on size
* Archive to S3-compatible storage (if needed)

#### 🛠️ Troubleshooting:

| Symptom        | Cause                                    | Fix                                      |
| -------------- | ---------------------------------------- | ---------------------------------------- |
| Delayed logs   | Logstash pipeline or queue backpressure  | Check filter plugins and output health   |
| Index bloating | Too many fields                          | Enable field filtering in Beats          |
| Search slow    | Unoptimised queries or large time ranges | Use filters instead of full-text queries |

### 🔀 Putting It All Together

| Tool       | Data Type       | Strength                                    |
| ---------- | --------------- | ------------------------------------------- |
| Prometheus | Metrics         | Best for numerical alerts & autoscaling     |
| Grafana    | Metrics + UI    | Visualisation + Alerting + Dashboards       |
| ELK Stack  | Logs            | Log analytics, debugging complex issues     |
| Loki       | Logs (like ELK) | Fast, Prometheus-style logs (less overhead) |

## 🧾 Scripting Cheat Sheet (Python, Bash, PowerShell)

Scripting is key in SRE/Platform roles for **automation**, **integration**, and **troubleshooting**. Expect to use it for provisioning, monitoring, CI/CD tasks, or data wrangling.

### 🐍 Python

**Why use it:**

* Great for REST APIs, cloud SDKs, JSON parsing, and CLI tools
* Works well with libraries like `boto3` (AWS), `google-cloud`, or `requests`

**Key Libraries:**

* `requests` → HTTP APIs
* `subprocess` → Shell commands
* `boto3` → AWS automation
* `argparse` → CLI interface
* `json/yaml` → Config processing

**Example: Call a REST API and parse response**

```python
import requests
response = requests.get("https://api.example.com/health")
if response.status_code == 200:
    print("API is healthy:", response.json())
else:
    print("Error:", response.status_code)
```

**Common tasks:**

* Call Terraform Cloud API
* Process Prometheus/Grafana metrics
* Automate GitLab MR comments / labels

### 🖥️ Bash

**Why use it:**

* Perfect for glue code in Linux systems, CI pipelines, or cluster automation
* Lightweight and runs everywhere

**Common Patterns:**

```bash
# Loop over pods and check logs
for pod in $(kubectl get pods -o name); do
  echo "Logs for $pod"
  kubectl logs $pod
done
```

**Handy tools to know:**

* `jq` – JSON parsing
* `awk`, `cut`, `sed` – Text processing
* `curl` – API testing
* `xargs` – Input pipelining
* `grep` – Log filtering

**Example: Check pods for CrashLoopBackOff**

```bash
kubectl get pods -A | grep CrashLoopBackOff
```

### 🪟 PowerShell

**Why use it:**

* Ideal for Azure + Windows VMs, or when working in hybrid environments
* Used in Azure Cloud Shell and scripts like `AzCopy` or `Get-AzResource`

**Example: List all running Azure VMs**

```powershell
Get-AzVM | Where-Object { $_.PowerState -eq "VM running" }
```

**Other tasks:**

* Manage Azure resources via `Az` module
* Query Entra ID groups/users
* Trigger builds or actions in Azure DevOps

## 🔐 Security & Networking Cheat Sheet (TLS, IAM, Secrets, Subnetting, Firewalls)

Security and networking aren’t just checkboxes — they’re foundational to platform reliability. Whether you're deploying apps, configuring policies, or locking down internal traffic, expect to **automate**, **enforce**, and **debug** security controls daily.


### 🔐 TLS & Encryption in Transit

#### 📘 What It Is:

TLS ensures encrypted communication between clients and services. In Kubernetes, it's often used:

* At ingress layer (HTTPS via NGINX, Traefik)
* For service-to-service encryption (mTLS)

#### 🔧 Tooling:

| Tool                | Use Case                                          |
| ------------------- | ------------------------------------------------- |
| **cert-manager**    | Auto-generates TLS certs via ACME (Let's Encrypt) |
| **AWS ACM**         | TLS for ELB/ALB, managed renewal                  |
| **Azure Key Vault** | Stores & manages TLS certs securely               |
| **Istio / Linkerd** | For mutual TLS (mTLS) in service mesh             |

#### 🛠️ Troubleshooting:

* 🔒 Certificate expired? → Check renewal logs from cert-manager
* 🔐 mTLS handshake failure? → Incompatible trust roots or mesh misconfig
* 🌐 HTTP instead of HTTPS? → Ingress config missing TLS block

### 🧑‍💼 IAM / Identity & Access Management

#### 📘 What It Is:

IAM ensures users, services, and workloads can **only do what they're supposed to** — principle of least privilege (PoLP).

#### ✅ Common Mechanisms:

| Cloud | IAM Mechanism                      | Kubernetes Integration                    |
| ----- | ---------------------------------- | ----------------------------------------- |
| AWS   | IAM Roles, Policies, **IRSA**      | ServiceAccount + OIDC + Role              |
| Azure | Entra ID, RBAC, Managed Identities | Azure AD pod identity / workload identity |
| GCP   | IAM Roles, Workload Identity       | Annotate SA to bind GCP IAM role          |

### 🧪 Secrets Management

#### 📘 What It Is:

Secrets include passwords, API keys, TLS certs — anything confidential.

#### 🔧 Tools:

| Method                 | Pros                          | Cons                           |
| ---------------------- | ----------------------------- | ------------------------------ |
| **K8s Secrets**        | Native, fast                  | Base64 = not encrypted         |
| **External Secrets**   | Sync from Vault/SSM/Key Vault | More secure, needs setup       |
| **Vault by HashiCorp** | Highly secure, audited        | Requires auth and unseal logic |

#### 🔐 Patterns:

* Prefer **External Secrets Operator** for dynamic secrets
* Rotate secrets via Terraform or CI/CD hooks
* Store in secret backends, not Git!

#### 🛠️ Debugging:

* `CreateContainerConfigError` → missing secret ref
* `kubectl describe pod` → shows which secret failed to mount
* Confirm RBAC access to secrets

### 🌐 Subnetting & VPCs

#### 📘 What It Is:

Organising IP ranges for workloads, services, and endpoints — **core to routing & firewalling**.

#### Key Patterns:

* Public subnets: expose LB/Ingress
* Private subnets: pods, DBs, internal services
* NAT Gateway: outbound internet for private subnets
* Pod CIDRs must **not overlap** VPC CIDRs

#### 🛠️ Cloud-Specific Notes:

* **AWS VPC CNI plugin** requires ENIs for pod networking
* Use `ip-masq-agent` to control NAT traffic in GKE
* Azure requires `AzureCNI` or `Kubenet` for subnet mapping

#### 🛠️ Troubleshooting:

* Pod stuck in `Pending` → insufficient IPs in subnet
* Node unreachable → route table or security group missing
* DNS failures → CoreDNS can’t resolve across peered VPCs

### 🔥 Firewalls & Network Rules

#### 📘 What It Is:

Controls **who can talk to what** — important for lateral movement protection, microsegmentation, and compliance.

#### 🔧 Cloud-Specific:

| Cloud | Ingress Control         | Egress Control      |
| ----- | ----------------------- | ------------------- |
| AWS   | Security Groups, NACLs  | NACLs, Routing      |
| Azure | NSGs (per NIC/subnet)   | NSGs + Route Tables |
| GCP   | Firewall Rules (tagged) | Egress rules        |

#### 🧰 Kubernetes-Level:

* Use **Network Policies** to limit pod-to-pod comms
* Tools like **Cilium**, **Calico**, or **Weave** support them

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

#### 🛠️ Debugging Tips:

* 🔒 Service unreachable? Check NSG/SG and pod NetworkPolicy
* 🧱 Timeout but no error? Check route table or firewall silently dropping
* 🔄 ClusterIP unreachable from outside? Not exposed through LoadBalancer or ingress

## 🚦 Deployment Strategies Cheat Sheet (Blue/Green, Canary & More)

Choosing the right deployment strategy is key to reducing risk, ensuring availability, and enabling fast rollback. These strategies often involve traffic shifting, replica management, and observability hooks (metrics, alerts, feature flags).

### 🟢 Blue/Green Deployment

**What it is:**
Run two environments in parallel:

* **Blue** is the currently live environment
* **Green** is the new version you're testing and preparing to promote

**How it works:**

* Deploy the new version (green) alongside the old (blue)
* Run smoke tests, end-to-end checks
* Once verified, switch all traffic to green by updating:

  * Load Balancer target group
  * Kubernetes `Service` selector
  * DNS record (less common in K8s)

**Pros:**

* Zero downtime (if done correctly)
* Easy rollback: just switch back to blue

**Cons:**

* Duplicate resource costs
* Manual testing or gating usually needed

**Kubernetes Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    version: green  # or 'blue', swap this to cut over
```

**Pro Tip:** Use Helm or Kustomize overlays to manage separate versions declaratively.

### 🐤 Canary Deployment

**What it is:**
Roll out new changes gradually to a subset of users/pods.

**How it works:**

* Start with 5-10% of traffic or replicas
* Monitor metrics like error rate, latency, saturation
* Increase traffic in steps (e.g. 20% → 50% → 100%)
* Halt or rollback if thresholds are breached

**Kubernetes Example with Argo Rollouts:**

```yaml
spec:
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
        - setWeight: 100
```

**Tools That Support Canary:**

* **Argo Rollouts** (Kubernetes-native CRD)
* **Flagger** (canary + metrics-based automation)
* **GitLab CI/CD** (canary stages + manual gates)
* **Spinnaker**, **LaunchDarkly**, **Istio VirtualService**

**Pros:**

* Safer than blue/green for high-risk changes
* Early detection of regressions

**Cons:**

* More complex setup
* Observability + metrics gating are a must

### 🎯 Rolling Update (K8s default)

**What it is:**
Replace old pods with new ones gradually.

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

**Pros:**

* Simple, built-in to Deployments
* Minimises downtime (unless readiness/liveness probes misconfigured)

**Cons:**

* No fine-grained traffic control
* Rollback not instantaneous if many pods updated

### 🧪 A/B Testing (Feature Flag Style)

**What it is:**
Route traffic based on user attributes (e.g. headers, cookies, user cohorts). Common in frontend and API deployments.

**How it works:**

* Run both versions (A + B) simultaneously
* Use gateway or reverse proxy to split traffic
* Often coupled with **feature flags** or experimentation tools

**Tools:**

* Istio/Linkerd (for header-based routing)
* LaunchDarkly
* Unleash
* Azure App Configuration + Feature Management SDK

**Pros:**

* Flexible — can test multiple dimensions
* Real user feedback

**Cons:**

* Requires identity/context aware routing
* Higher application and observability complexity

### 🧯 Shadow/Traffic Mirroring

**What it is:**
Send a copy of live traffic to the new version — doesn’t affect end users.

**Use case:**

* Performance validation
* Load testing new logic under production-like conditions

**Tools:**

* Istio mirror traffic
* Envoy filters
* GCP/AWS/Azure API Gateway with traffic duplication


### 🛠️ When to Use What

| Strategy       | Best For                                  | Risk Level | Rollback Difficulty       |
| -------------- | ----------------------------------------- | ---------- | ------------------------- |
| Blue/Green     | Major version changes                     | Low        | Easy (switch back)        |
| Canary         | Gradual production rollout                | Medium     | Controlled                |
| Rolling Update | Minor, backwards-compatible updates       | Low        | Medium (restart deploy)   |
| A/B Testing    | UX/API changes based on user segments     | Medium     | Requires flag reversal    |
| Shadowing      | Performance/load verification pre-release | None       | N/A (no real user impact) |
