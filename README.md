# Kubernetes (K3s) DevSecOps & Runtime Security Lab

A hands-on DevSecOps laboratory demonstrating container infrastructure provisioning, automated vulnerability scanning, kernel-level runtime threat detection (eBPF), and zero-trust network segmentation on Ubuntu Linux.

---

## 🏗 Architecture & Security Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **OS / Host** | Ubuntu 24.04 LTS | Linux kernel foundation for eBPF operations |
| **Orchestration** | K3s (v1.36) | Lightweight CNCF-certified Kubernetes distribution |
| **Package Manager** | Helm (v3) | Declarative deployment of cluster security tooling |
| **Image Security** | Aqua Trivy | Static vulnerability (CVE) & misconfiguration scanner |
| **Runtime Security** | Falco (v0.44) | Real-time system call monitoring via modern-eBPF |
| **Network Security** | NetworkPolicy | Ingress traffic isolation and network segmentation |

---

## 📋 Implementation Steps

### 1. Cluster Provisioning (K3s)
Update system dependencies and deploy the K3s runtime:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git -y
curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -
