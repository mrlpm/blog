---
title: "About Me"
description: "Professional background, technical philosophy, and projects by Luis Pérez."
showDate: false
showAuthor: false
showReadingTime: false
showTableOfContents: true
showTaxonomies: false
---

{{< lead >}}
Hi! I'm **Luis Pérez**, a cybersecurity enthusiast, SRE, and platform engineer.
{{< /lead >}}

I'm passionate about designing and building **immutable, declarative, and secure-by-design infrastructure**. I use my homelab as a testing ground to simulate production-grade environments with cloud-native technologies, GitOps automation, and internal tool development.

---

## 🚀 Technical Philosophy

1. **Infrastructure as Code (IaC):** Everything deployed must be defined in version-controlled code (OpenTofu/Terraform) to guarantee full reproducibility and zero configuration drift.
2. **Immutable & API-First Nodes:** I favor minimal, security-hardened operating systems like **Talos Linux**, eliminating SSH access and manual mutations in favor of declarative APIs.
3. **GitOps as Single Source of Truth:** Cluster state changes are never applied directly; they flow through Git commits and continuous reconciliation loops with **ArgoCD**.
4. **Security by Design (DevSecOps):** Decoupled secret management (External Secrets Operator), immutable images, and multi-layer network segmentation.

---

## 🛠️ Stack & Areas of Interest

### Platform & Cloud Native

- **Orchestration & Systems:** Kubernetes, Talos Linux, Proxmox VE.
- **IaC & Automation:** OpenTofu, Terraform, GNU Make.
- **GitOps & Delivery:** ArgoCD, Kustomize, GitHub Actions, Docker, `ttl.sh`.

### Cybersecurity & Pentesting

- **Offensive Security & Pentesting:** Penetration testing, vulnerability assessments across network and web applications.
- **DevSecOps & Hardening:** External Secrets Operator (ESO), NetworkPolicies, RBAC, immutable infrastructure, Threat Modeling.

### Software Engineering

- **Backend:** Go (REST APIs, WebSockets, Linux PTYs, SQLite with WAL).
- **Frontend:** React, TypeScript, Vite, Tailwind CSS, xterm.js.

---

## 🔬 Featured Project: _praxis-lab_

**praxis-lab** is an online Linux labs platform running on my self-hosted homelab. It provides students with interactive, real-time practice environments (micro-VMs in isolated pods) directly in their web browser, fully orchestrated via Kubernetes and GitOps.

You can follow the full architecture and implementation series on the [Blog](/en/posts/).

---

## 📬 Connect With Me

- **GitHub:** [@mrlpm](https://github.com/mrlpm)
- **GitLab:** [@mrlpm](https://gitlab.com/mrlpm)
- **LinkedIn:** [luisperezgt](https://linkedin.com/in/luisperezgt)
- **X (Twitter):** [@luispmarin](https://x.com/luispmarin)
- **Email:** [luis.perez@proton.me](mailto:luis.perez@proton.me)
