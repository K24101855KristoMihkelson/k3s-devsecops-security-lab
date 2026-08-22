

Skip to content
Using Gmail with screen readers
(no subject)
Inbox
Kristo Mihkelson
	
	Attachments7:21 PM (5 minutes ago)
 
Kristo Mihkelson <krismihkel@gmail.com>
	
7:23 PM (3 minutes ago)
	
	
to me
# Kubernetes (K3s) DevSecOps & Runtime Security Lab

A fully documented, hands-on DevSecOps and cloud-native security laboratory built on Ubuntu Linux. This project covers Kubernetes cluster provisioning, automated container vulnerability scanning, kernel-level runtime intrusion detection using eBPF, and zero-trust network segmentation.

---

## 📑 Table of Contents
1. [Overview & Security Objectives](#-overview--security-objectives)
2. [Architecture & Technology Stack](#-architecture--technology-stack)
3. [Prerequisites & System Specifications](#-prerequisites--system-specifications)
4. [Step-by-Step Implementation Guide](#-step-by-step-implementation-guide)
   - [Phase 1: Lightweight Cluster Provisioning (K3s)](#phase-1-lightweight-cluster-provisioning-k3s)
   - [Phase 2: Cloud-Native Package Manager (Helm)](#phase-2-cloud-native-package-manager-helm)
   - [Phase 3: Static Container Image Scanning (Trivy)](#phase-3-static-container-image-scanning-trivy)
   - [Phase 4: Kernel Runtime Security Engine (Falco & modern-eBPF)](#phase-4-kernel-runtime-security-engine-falco--modern-ebpf)
   - [Phase 5: Attack Simulation & Incident Triggering](#phase-5-attack-simulation--incident-triggering)
   - [Phase 6: Declarative Zero-Trust Network Policy](#phase-6-declarative-zero-trust-network-policy)
5. [Raw Execution Verification & Logs](#-raw-execution-verification--logs)
   - [Trivy Vulnerability Audit Report](#trivy-vulnerability-audit-report)
   - [Falco eBPF Kernel Intrusion Alert](#falco-ebpf-kernel-intrusion-alert)
6. [Manifests & Configurations](#-manifests--configurations)
7. [Repository File Structure](#-repository-file-structure)
8. [Core Competencies & Key Takeaways](#-core-competencies--key-takeaways)
9. [Portfolio & Professional Showcase](#-portfolio--professional-showcase)

---

## 🎯 Overview & Security Objectives

In cloud-native environments, perimeter security alone is insufficient. Modern container security requires a layered defense model:

* **Build/Admission Phase:** Static vulnerability and secret scanning to prevent high-risk images from being scheduled.
* **Runtime Phase:** Continuous kernel-level syscall tracing via eBPF to detect zero-day exploits, terminal injections, and unauthorized file reads.
* **Network Phase:** Explicit zero-trust ingress and egress rules to block lateral movement within cluster networks.

This laboratory provides an end-to-end implementation and validation of these security controls.

---

## 🛠 Architecture & Technology Stack

| Layer | Component | Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Host OS** | Ubuntu Linux | 24.04 LTS (x86_64) | Base Linux environment with eBPF-capable kernel |
| **Container Engine** | containerd (k3s-embedded) | v1.7+ | CRI-compliant container runtime |
| **Orchestration** | K3s | v1.36+ | CNCF-certified lightweight Kubernetes distribution |
| **Package Manager** | Helm | v3.21+ | Kubernetes application deployment and management |
| **Static Scanner** | Aqua Trivy | v0.71+ | Comprehensive vulnerability (CVE) scanner |
| **Runtime Engine** | Falco | v0.44+ | Threat detection engine leveraging modern-eBPF probes |
| **Network Security** | Kubernetes NetworkPolicy | v1 | Ingress/Egress microsegmentation |

---

## 💻 Prerequisites & System Specifications

* **Operating System:** Ubuntu 22.04 / 24.04 LTS (Physical machine or VirtualBox VM)
* **Resources:** Minimum 2 vCPUs, 4 GB RAM, 20 GB Disk
* **Access:** Non-root user with `sudo` privileges
* **Connectivity:** Unrestricted outbound HTTP/HTTPS access

---

## 🚀 Step-by-Step Implementation Guide

### Phase 1: Lightweight Cluster Provisioning (K3s)

1. Update package indexes and install core utilities:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget gnupg lsb-release -y
Deploy the K3s control-plane node:

Bash
curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -
Configure user environment for kubectl access:

Bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R $USER:$USER ~/.kube
export KUBECONFIG=~/.kube/config
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
Verify cluster node status:

Bash
kubectl get nodes -o wide
Phase 2: Cloud-Native Package Manager (Helm)
Install Helm v3 for managing complex Kubernetes deployments:

Bash
curl [https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3](https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3) | bash
helm version
Phase 3: Static Container Image Scanning (Trivy)
Configure the official Aqua Security repository and install Trivy:

Bash
wget -qO - [https://aquasecurity.github.io/trivy-repo/deb/public.key](https://aquasecurity.github.io/trivy-repo/deb/public.key) | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] [https://aquasecurity.github.io/trivy-repo/deb](https://aquasecurity.github.io/trivy-repo/deb) $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install trivy -y
Execute a vulnerability scan against a legacy target image (nginx:1.14.0):

Bash
trivy image nginx:1.14.0
Phase 4: Kernel Runtime Security Engine (Falco & modern-eBPF)
Add and update the Falco Helm repository:

Bash
helm repo add falcosecurity [https://falcosecurity.github.io/charts](https://falcosecurity.github.io/charts)
helm repo update
Deploy Falco into a dedicated namespace using modern eBPF probes:

Bash
helm install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --set driver.kind=modern_ebpf
Confirm pod operational status:

Bash
kubectl get pods -n falco
Phase 5: Attack Simulation & Incident Triggering
Spawn an interactive test pod:

Bash
kubectl run test-target --image=alpine -- sleep 3600
Simulate unauthorized credential access inside the running container:

Bash
kubectl exec -it test-target -- cat /etc/shadow
Retrieve Falco security detection logs:

Bash
kubectl logs -n falco -l app.kubernetes.io/name=falco --tail=50
Phase 6: Declarative Zero-Trust Network Policy
Create and apply the isolate-db.yaml manifest to restrict incoming TCP traffic on port 5432 strictly to pods labeled role: backend:

Bash
kubectl apply -f isolate-db.yaml
📊 Raw Execution Verification & Logs
Trivy Vulnerability Audit Report
Plaintext
Target: nginx:1.14.0 (debian 9.5)
Total Vulnerabilities Detected: 289
--------------------------------------------------
CRITICAL : 39
HIGH     : 107
MEDIUM   : 65
LOW      : 69
UNKNOWN  : 9

Key Findings:
- dpkg: CVE-2022-1664 (CRITICAL) - Dpkg::Source::Archive arbitrary file overwrite
- libssl1.1: CVE-2018-0732 (HIGH) - Malicious DH prime denial of service
- glibc (libc6): CVE-2017-18269 (CRITICAL) - Memory corruption in memcpy
- shadow-utils (passwd/login): CVE-2017-12424 (CRITICAL) - Buffer overflow
Falco eBPF Kernel Intrusion Alert
Plaintext
18:36:25.231548601: Warning Sensitive file opened for reading by non-trusted program |
file=/etc/shadow gparent=<NA> ggparent=<NA> gggparent=<NA>
evt_type=open user=root user_uid=0 user_loginuid=-1 process=cat
proc_exepath=/bin/busybox parent=systemd command=cat /etc/shadow
terminal=34816 container_id=9727be962b72 container_name=test-target
container_image_repository=docker.io/library/alpine container_image_tag=latest
k8s_pod_name=test-target k8s_ns_name=default
📜 Manifests & Configurations
isolate-db.yaml
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
📁 Repository File Structure
Plaintext
k3s-devsecops-security-lab/
├── README.md           # Comprehensive laboratory guide, architecture, and logs
└── isolate-db.yaml     # Kubernetes NetworkPolicy declarative configuration
🧠 Core Competencies & Key Takeaways
eBPF Observability: Real-time visibility into process executions, network connections, and file access at the Linux kernel layer.

Automated Image Security: Pre-deployment CVE identification and software bill of materials (SBOM) triage via Trivy.

Incident Triage & Response: Validating SIEM-ready security events generated by containerized runtime sensors.

Microsegmentation: Restricting east-west lateral movement inside Kubernetes using native NetworkPolicy objects.

On Sat, Aug 22, 2026 at 9:21 PM Kristo Mihkelson <krismihkel@gmail.com> wrote:


