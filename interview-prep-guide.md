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
* Deployment Strategies: Blue/Green & Canary

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

Terraform is a tool for declaratively managing infrastructure.

### Key Concepts

* Resources (`resource "aws_instance"`)
* Variables (`variable "region"`)
* Outputs
* Modules for reuse
* Remote state (e.g., S3 + DynamoDB)
* Workspaces (env separation)
* `backend`, `provider`, `locals`

### Common Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

### Troubleshooting Scenarios

| Problem         | Fix                                        |
| --------------- | ------------------------------------------ |
| Locked state    | `terraform force-unlock`                   |
| Drift           | `terraform refresh`                        |
| Plan fails      | Validate variable values, resource changes |
| S3 state issues | Check backend config, bucket policy        |

### Tips

* Use `terraform fmt` and `terraform-docs`
* Split modules by domain (e.g., `network`, `iam`, `eks`)
* Use `terragrunt` for multi-envs and DRY structure

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

#### 🛠️ Interview Tip:

> "We used IRSA to give our Kubernetes pods temporary access to S3 buckets using scoped IAM policies, reducing the need for static secrets."

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

## 🚦 Deployment Strategies: Blue/Green & Canary

### Blue/Green

* Two environments (blue = live, green = new)
* Swap traffic via DNS, load balancer, or Ingress
* Pros: Zero-downtime
* Cons: Higher infra cost

Kubernetes example:

```yaml
- name: green-deploy
  selector:
    matchLabels:
      version: green
```

Switch traffic with Ingress or service selector.

### Canary

* Deploy to a small % of users
* Gradually increase replica weight
* Use metrics/alerts to halt if issues

Kubernetes with Argo Rollouts or Flagger:

```yaml
spec:
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: { duration: 10m }
```

CI/CD Tools Support:

* **GitLab**: Canary stages + manual approvals
* **Argo Rollouts**: Native CRD support for progressive delivery
