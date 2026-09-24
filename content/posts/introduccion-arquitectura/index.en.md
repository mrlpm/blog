---
title: "Post 1: Introduction and Overall Architecture of the DevOps Homelab"
description: "An overview of an online Linux labs platform built with OpenTofu, Proxmox, Talos Linux, Kubernetes, ArgoCD, GitHub Actions, Go, and React."
date: "2026-09-23T19:12:03-06:00"
draft: false
categories:
  - "DevOps & SRE"
  - "Platform & Infrastructure"
series:
  - "DevOps Homelab: praxis-lab"
series_order: 1
tags:
  - "homelab"
  - "kubernetes"
  - "talos-linux"
  - "proxmox"
  - "opentofu"
  - "argocd"
  - "gitops"
  - "go"
  - "react"
---

## 1. Project Vision

This is the first article in a series documenting, piece by piece, the implementation of my **DevOps homelab**. The project is an online Linux labs platform, called **praxis-lab**, designed to deliver Linux courses and give every student an isolated practice environment: a micro virtual machine accessible from the browser.

Choosing a homelab as the implementation environment is driven by three goals:

1. **Full control over the entire lifecycle**, from virtual machine provisioning all the way to application deployment.
2. **Zero public-cloud cost**, which allows iterating on complex architectures without budget constraints.
3. **Technical portfolio**: every component is documented with the level of detail that would be expected in a production environment — a practice that transfers directly to SRE and DevOps roles.

The result is a fully automated end-to-end platform: a student opens a browser, selects a lab, and gets an interactive Linux terminal — with zero manual intervention by an administrator.

---

## 2. Overall System Architecture

The system is divided into five clearly separated layers: virtualized hardware, Kubernetes platform, application, GitOps, and CI/CD. The high-level architecture diagram is shown below:

{{< figure
    src="architecture.png"
    alt="General Architecture from the DevOps Homelab"
    caption="General Architecture from the DevOps Homelab"
    default=true
    >}}

The organizing principle of the architecture is **separation of responsibilities per repository**:

| Repository         | Responsibility                                                             | Technology                                                    |
| ------------------ | -------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `infra-opentofu`   | Declarative VM provisioning, OS configuration, and GitOps bootstrap        | OpenTofu, `bpg/proxmox`, `siderolabs/talos`, `hashicorp/helm` |
| `gitops-manifests` | Single source of truth for the desired state of the platform in Kubernetes | ArgoCD, Kustomize, Ingress-NGINX, External Secrets Operator   |
| `praxis-lab`       | Application source code for the labs platform                              | Go, SQLite, React, TypeScript, Vite, xterm.js                 |

The flow of control moves in a single direction: **code → CI/CD → image registry → GitOps repository → ArgoCD → Kubernetes**. No change is ever applied directly to the cluster; every state mutation goes through a commit, which guarantees full traceability and reversibility.

---

## 3. Technology Stack

### 3.1. Infrastructure, Virtualization & Operating System (`infra-opentofu`)

| Component                       | Role in the architecture                                                                                                                            |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxmox VE**                  | Type-1 bare-metal hypervisor hosting the Kubernetes virtual machines.                                                                               |
| **Talos Linux (Sidero Labs)**   | Immutable, minimal, security-hardened Linux distribution designed exclusively for Kubernetes. No SSH: fully managed through its API via `talosctl`. |
| **OpenTofu (>= 1.6.0)**         | Infrastructure as Code engine. Declaratively defines compute, operating system configuration, and the GitOps pipeline bootstrap.                    |
| **`bpg/proxmox` provider**      | Declarative provisioning of virtual machines on Proxmox VE.                                                                                         |
| **`siderolabs/talos` provider** | Talos machine configuration, cluster bootstrap, and kubeconfig generation.                                                                          |
| **`hashicorp/helm` provider**   | Deploys the official ArgoCD chart directly into the Talos cluster: the entry point that "seeds" the GitOps engine.                                  |

This chain is particularly interesting from an SRE perspective: the infrastructure is provisioned **entirely from code** (HCL for the VMs, declarative Talos configuration), and the resulting cluster is an **immutable** system — there is no mutable local state and no shell session on the nodes — a property that eliminates an entire class of configuration-drift problems.

### 3.2. GitOps & Kubernetes Platform (`gitops-manifests`)

| Component                           | Role in the architecture                                                                                                                     |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **Kubernetes**                      | Container orchestrator, bootstrapped and managed by Talos Linux. Runs the labs platform and its supporting services.                         |
| **ArgoCD**                          | GitOps controller: continuous pull-based reconciliation loop, ApplicationSets, and the App-of-Apps pattern.                                  |
| **Kustomize**                       | Declarative templating engine: `base/` manifests with `dev` and `prod` overlays to parameterize per environment without duplicating content. |
| **Ingress-NGINX Controller**        | L7 reverse proxy at the cluster edge, configured to support **WebSocket upgrades** — a critical requirement for the interactive shells.      |
| **External Secrets Operator (ESO)** | Declarative synchronization of secrets from an external secret manager into Kubernetes, keeping credentials out of the GitOps repository.    |

### 3.3. Labs Application (`praxis-lab`)

**Backend (Go):**

| Component                                     | Role in the architecture                                                                                                                                                                                  |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Go (>= 1.22)**                              | Primary language. Implements the REST API and the WebSocket server. Static compilation produces small, efficient binaries, ideal for containers.                                                          |
| **SQLite**                                    | Lightweight embedded persistence with no external service dependency. In Kubernetes it is configured with a **PVC for persistence** and **WAL mode** (Write-Ahead Logging) for reader/writer concurrency. |
| **Gorilla WebSocket / `nhooyr.io/websocket`** | Real-time full-duplex communication between the student's browser and the terminal processes.                                                                                                             |
| **Linux PTY (`creack/pty`)**                  | Pseudo-terminal allocation inside the container to run interactive command shells.                                                                                                                        |

**Frontend (React):**

| Component                                     | Role in the architecture                                                                                                                    |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **React (>= 18) + TypeScript**                | Single Page Application user interface with static typing across components and services.                                                   |
| **Vite**                                      | Fast bundling, development server with hot-reloading, and asset generation.                                                                 |
| **xterm.js** (+ `fit` and `web-links` addons) | Browser-based terminal emulator: consumes the WebSocket stream and renders it as a real shell session.                                      |
| **NGINX (Alpine)**                            | Static web server inside the frontend container: serves the SPA and resolves client-side routing (fallback to `index.html` for deep links). |

### 3.4. CI/CD, Containers & Local Environment

| Component                          | Role in the architecture                                                                                                                                                                             |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Docker**                         | Multi-stage builds: the Go backend is compiled in one stage and copied onto a minimal base image; the frontend is built with Vite and served from NGINX Alpine.                                      |
| **ttl.sh (Ephemeral Registry)**    | Anonymous, ephemeral container registry with **immutable tags via Git commit SHA and TTL** (`ttl.sh/<uuid>/praxis:sha-<commit>`), eliminating credential management and token rotation in CI/CD. |
| **GitHub Actions**                 | CI pipelines: push triggers, linting and tests, image builds, publishing to ttl.sh, and an automatic commit of the new tags into `gitops-manifests`.                                                 |
| **GNU Make**                       | Task automation at both the repository root (`homelab/Makefile`) and the application (`praxis-lab/Makefile`).                                                                                        |
| **Vim / Vi**                       | Standard terminal editor for local configuration and script editing.                                                                                                                                 |

---

## 4. End-to-End Flow: From Code to Lab

The following sequence diagram describes the full cycle of a code change until a student uses it:

{{< figure
    src="sequence.png"
    alt="Sequence diagram"
    caption="Sequence diagram"
    default=true
    >}}

And the lab execution flow from the student's perspective:

{{< figure
    src="flow.png"
    alt="Execution flow"
    caption="Execution flow"
    default=true
    >}}

---

## 5. Design Principles

The architecture is grounded in principles that apply directly to production SRE/DevOps operations:

- **100% declarative, reproducible infrastructure**: from the VMs in Proxmox to the ArgoCD chart, all state is defined in versioned code. The entire environment can be rebuilt from the repositories alone.
- **Immutable nodes**: Talos Linux eliminates traditional server administration (no SSH, no mutable local state). The only management path is its API and its declarative configuration.
- **GitOps**: the `gitops-manifests` repository is the single source of truth. ArgoCD continuously reconciles; any deviation of cluster state is either corrected or recorded as _drift_.
- **Immutable, traceable images**: the SHA tag binds each deployed image to an exact commit. Rollback is a tag change, not a rebuild.
- **Security by design**: secrets never live in the repository (ESO injects them at runtime), traffic is segmented with NetworkPolicies, and access is controlled with RBAC.
- **Low-friction local development**: unified Makefiles at both levels let any pipeline task run locally with a single command.

---

## 6. Decision Records

A few decisions deserve explicit mention:

| Decision                                                                          | Alternative                                                               | Rationale                                                                                                                                                       |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxmox VE + Talos Linux**                                                      | Managed Kubernetes in the public cloud, or kubeadm on traditional distros | A type-1 homelab with immutable nodes eliminates the administration surface and configuration drift; Talos, being API-first, is fully manageable from OpenTofu. |
| **OpenTofu with `bpg/proxmox` + `siderolabs/talos` + `hashicorp/helm` providers** | Manual steps with `talosctl` and `helm install`                           | The complete bootstrap (VMs → cluster → ArgoCD) must be idempotent, reproducible, and auditable from a single IaC layer.                                        |
| **SQLite + WAL + PVC** instead of PostgreSQL                                      | External database service                                                 | For a homelab, an embedded database removes an operational dependency. WAL allows concurrent writes alongside reads; the PVC guarantees volume persistence.     |
| **xterm.js + PTY over WebSocket**                                                 | VNC, Serial Console, Jupyter                                              | A real POSIX terminal with a minimal surface: no graphical protocol, no notebook server — just a full-duplex byte stream.                                       |
| **ArgoCD App-of-Apps + ApplicationSets**                                          | Manual list of Applications                                               | Manages _n_ applications (currently praxis-lab; soon the labs themselves) from a single controlling Application, scaling without duplication.                   |
| **Ephemeral registry ttl.sh with SHA tags**                                      | Traditional private registries with credentials / `latest` tags           | Removes token and secret management from CI/CD; TTL handles automatic cleanup while SHA guarantees exact traceability.                                         |

---

## 7. Series Structure

Each article in the series dives deep into one layer of the architecture described in section 2:

| #   | Article                                                                                                          | Layer              |
| --- | ---------------------------------------------------------------------------------------------------------------- | ------------------ |
| 1   | **Introduction and Overall Architecture** (this article)                                                         | Big picture        |
| 2   | Infrastructure as Code: Proxmox, Talos Linux, and OpenTofu (`bpg/proxmox`, `siderolabs/talos`, `hashicorp/helm`) | `infra-opentofu`   |
| 3   | GitOps with ArgoCD, Kustomize, and secret management with ESO                                                    | `gitops-manifests` |
| 4   | CI/CD with GitHub Actions, Docker multi-stage, and ephemeral publishing on ttl.sh                                | CI/CD              |
| 5   | The praxis-lab application: Go backend (REST, WebSockets, PTY, SQLite) and React frontend (xterm.js)             | `praxis-lab`       |
| 6   | Observability and resilience: metrics, logs, alerting, and recovery strategy                                     | Operations         |

---

## 8. Conclusions

This project demonstrates a complete DevOps pipeline implemented on self-owned hardware: **IaC** (OpenTofu provisioning Proxmox and Talos), **GitOps** (ArgoCD + Kustomize), **CI/CD** (GitHub Actions + Docker + ttl.sh), and a **business application** with non-trivial real-time requirements (interactive terminals over WebSocket) and persistence (SQLite on Kubernetes).

The homelab serves as a proving ground: the patterns documented are identical to those that would be applied on a managed public-cloud cluster, with the difference that every decision — and every failure — is fully inspectable and reproducible.

The next article dives deep into the infrastructure layer: the complete provisioning chain — VMs on Proxmox with `bpg/proxmox`, immutable cluster bootstrap with `siderolabs/talos`, and ArgoCD seeding with `hashicorp/helm`.

---
