---

# 🚀 Platform Engineer / SRE Interview Prep Guide

---

## 📚 Searchable Topics (Use CTRL+F to Jump to Sections)

* Kubernetes Cheat Sheet for Technical Interviews
* Example Kubernetes Troubleshooting: 60/100 Pods in Error State
* Kubernetes Deployment Failing: Troubleshooting Guide
* Terraform Cheat Sheet for SRE/Platform Interviews
* GitLab CI/CD Cheat Sheet for SRE/Platform Interviews
* Monitoring & Logging Cheat Sheet (Prometheus, Grafana, ELK)
* Scripting Cheat Sheet (Python, Bash, PowerShell)
* Security & Networking Cheat Sheet (TLS, IAM, Secrets, Subnetting, Firewalls)
* Deployment Strategies: Blue/Green, Canary

---

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

---

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

---

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

---

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

---

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

---

## 📊 Monitoring & Logging Cheat Sheet

### Prometheus

* Metrics collection
* PromQL queries for alerts

```promql
rate(http_requests_total[5m])
```

### Grafana

* Dashboards from Prometheus, Loki, Elasticsearch

### ELK Stack

* Elasticsearch: storage/search
* Logstash: parsing/transform
* Kibana: query/visualisation

### Troubleshooting

* Metrics missing? → Exporter issue
* Alerts not firing? → Query or threshold wrong
* Log delay? → ES queue or disk pressure

---

## 🧾 Scripting Cheat Sheet (Python, Bash, PowerShell)

### Python

```python
import requests
r = requests.get("https://api.example.com")
print(r.json())
```

### Bash

```bash
for pod in $(kubectl get pods -o name); do kubectl logs $pod; done
```

### PowerShell

```powershell
Get-AzVM | Where-Object { $_.PowerState -eq "VM running" }
```

---

## 🔐 Security & Networking Cheat Sheet

### TLS

* Managed via cert-manager or cloud-native (ACM, Key Vault)
* mTLS for service-to-service encryption

### IAM / Identity

* **AWS**: IAM Roles, IRSA
* **Azure**: Entra ID + RBAC
* **GCP**: Workload Identity

### Secrets Management

* K8s Secrets (base64, weak)
* Use Vault / External Secrets Operator

### Subnetting

* Private/public split
* NAT for outbound access
* Pod CIDRs must not overlap VPC

### Firewalls

* AWS: SGs + NACLs
* Azure: NSGs
* GCP: Firewall rules (tag-based)

---

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

---
