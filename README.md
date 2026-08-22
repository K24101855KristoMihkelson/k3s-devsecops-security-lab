# Kubernetes (K3s) DevSecOps & Runtime Security Lab

A comprehensive, hands-on DevSecOps laboratory demonstrating container infrastructure provisioning, automated vulnerability scanning, kernel-level runtime threat detection (eBPF), and zero-trust network segmentation on Ubuntu Linux.

---

## Table of Contents
1. [Overview & Objectives](#overview--objectives)
2. [Architecture & Tech Stack](#architecture--tech-stack)
3. [Prerequisites](#prerequisites)
4. [Step-by-Step Implementation](#step-by-step-implementation)
   - [Phase 1: Cluster Provisioning (K3s)](#phase-1-cluster-provisioning-k3s)
   - [Phase 2: Helm Package Manager Setup](#phase-2-helm-package-manager-setup)
   - [Phase 3: Static Vulnerability Scanning (Trivy)](#phase-3-static-vulnerability-scanning-trivy)
   - [Phase 4: Runtime Threat Detection (Falco & eBPF)](#phase-4-runtime-threat-detection-falco--ebpf)
   - [Phase 5: Attack Simulation & Incident Response](#phase-5-attack-simulation--incident-response)
   - [Phase 6: Zero-Trust Network Policy Enforcement](#phase-6-zero-trust-network-policy-enforcement)
5. [Verification & Security Logs](#verification--security-logs)
6. [Key Takeaways & Core Competencies](#key-takeaways--core-competencies)
7. [Project Showcase Summary (LinkedIn / Portfolio)](#project-showcase-summary-linkedin--portfolio)

---

## Overview & Objectives

In modern cloud environments, securing containerized workloads requires security controls across the entire lifecycle:
* **Build-Time:** Proactive vulnerability and CVE scanning before images reach production.
* **Run-Time:** Continuous kernel-level syscall monitoring to detect intrusions and lateral movement.
* **Network-Level:** Enforcing least-privilege network segmentation inside Kubernetes clusters.

This project implements a fully reproducible DevSecOps pipeline inside a lightweight Kubernetes environment.

---

## Architecture & Tech Stack

* **Platform / OS:** Ubuntu 24.04 LTS (x86_64)
* **Container Orchestration:** K3s v1.36 (Rancher Lightweight Kubernetes)
* **Package Management:** Helm v3.21+
* **Vulnerability Scanner:** Aquasec Trivy (Static CVE Scanner)
* **Runtime Security Engine:** Falco v0.44+ (modern-eBPF probe)
* **Network Policy Controller:** Kubernetes Native NetworkPolicy Engine

---

## Prerequisites

* Ubuntu Linux (bare-metal or VirtualBox VM)
* Sudo privileges
* Internet access for package and container repository fetching

---

## Step-by-Step Implementation

### Phase 1: Cluster Provisioning (K3s)

1. Update system repositories:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git -y

    Install K3s using the official automated installer:

Bash

curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -

    Configure user permissions for kubectl:

Bash

mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R $USER:$USER ~/.kube
export KUBECONFIG=~/.kube/config
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc

    Verify cluster health:

Bash

kubectl get nodes -o wide

Phase 2: Helm Package Manager Setup

Install Helm v3 for deploying cloud-native security charts:
Bash

curl [https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3](https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3) | bash
helm version

Phase 3: Static Vulnerability Scanning (Trivy)

    Add the official Trivy repository:

Bash

sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - [https://aquasecurity.github.io/trivy-repo/deb/public.key](https://aquasecurity.github.io/trivy-repo/deb/public.key) | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] [https://aquasecurity.github.io/trivy-repo/deb](https://aquasecurity.github.io/trivy-repo/deb) $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install trivy -y

    Scan a legacy/vulnerable container image to evaluate CVE exposure:

Bash

trivy image nginx:1.14.0

    Result: Successfully detected 289 total vulnerabilities (39 Critical, 107 High, 65 Medium, 69 Low).

Phase 4: Runtime Threat Detection (Falco & eBPF)

    Add Falco Helm repository:

Bash

helm repo add falcosecurity [https://falcosecurity.github.io/charts](https://falcosecurity.github.io/charts)
helm repo update

    Deploy Falco configured with the modern eBPF driver:

Bash

helm install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --set driver.kind=modern_ebpf

    Confirm Falco DaemonSet execution:

Bash

kubectl get pods -n falco

Phase 5: Attack Simulation & Incident Response

    Deploy a test target pod into the default namespace:

Bash

kubectl run test-target --image=alpine -- sleep 3600

    Simulate an active compromise by accessing sensitive system authentication files (/etc/shadow):

Bash

kubectl exec -it test-target -- cat /etc/shadow

    Review Falco runtime security logs:

Bash

kubectl logs -n falco -l app.kubernetes.io/name=falco --tail=50

Phase 6: Zero-Trust Network Policy Enforcement

Create isolate-db.yaml to restrict inbound network traffic so that only designated backend pods can reach the database:
YAML

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 5432

Apply the policy:
Bash

kubectl apply -f isolate-db.yaml

Verification & Security Logs
Falco eBPF Security Alert Output:
Plaintext

18:36:25.231548601: Warning Sensitive file opened for reading by non-trusted program | 
file=/etc/shadow gparent=<NA> ggparent=<NA> gggparent=<NA> 
evt_type=open user=root user_uid=0 user_loginuid=-1 process=cat 
proc_exepath=/bin/busybox parent=systemd command=cat /etc/shadow 
terminal=34816 container_id=9727be962b72 container_name=test-target 
container_image_repository=docker.io/library/alpine container_image_tag=latest 
k8s_pod_name=test-target k8s_ns_name=default

Key Takeaways & Core Competencies

    Linux Kernel Observability: Implemented real-time system call monitoring without kernel modules using eBPF probes.

    DevSecOps Integration: Automated static image auditing to block unpatched container base layers.

    Incident Response Verification: Confirmed immediate security alerting during unauthorized file access attempts.

    Zero-Trust Network Segmentation: Enforced declarative network boundaries across Kubernetes workloads.O
# k3s-devsecops-security-lab
