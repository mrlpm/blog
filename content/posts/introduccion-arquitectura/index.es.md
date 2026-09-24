---
title: "Post 1: Introducción y Arquitectura General del Homelab DevOps"
description: "Visión general de una plataforma de laboratorios Linux en línea construida con OpenTofu, Proxmox, Talos Linux, Kubernetes, ArgoCD, GitHub Actions, Go y React."
date: "2026-09-23T19:12:00-06:00"
draft: false
categories:
  - "DevOps & SRE"
  - "Platform & Infrastructure"
series:
  - "Homelab DevOps: praxis-lab"
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

## 1. Visión del Proyecto

Este es el primer artículo de una serie dedicada a documentar, pieza por pieza, la implementación de mi **homelab DevOps**. El proyecto consiste en una plataforma de laboratorios Linux en línea, denominada **praxis-lab**, diseñada para impartir cursos de Linux y proveer a cada estudiante un entorno aislado de práctica: una micromáquina virtual accesible desde el navegador.

La elección de un homelab como entorno de implementación responde a tres objetivos:

1. **Control total sobre el ciclo de vida** de la infraestructura, desde el aprovisionamiento de máquinas virtuales hasta el despliegue de la aplicación.
2. **Costo cero en nube pública**, lo que permite iterar sin restricciones presupuestales sobre arquitecturas complejas.
3. **Portfolio técnico**: cada componente se documenta con el nivel de detalle que se exigiría en un entorno de producción, una práctica directamente transferible a roles de SRE y DevOps.

El resultado es una plataforma end-to-end automatizada: el estudiante abre un navegador, selecciona un laboratorio y obtiene una terminal Linux interactiva, sin intervención manual de un administrador.

---

## 2. Arquitectura General del Sistema

El sistema se divide en cinco capas bien diferenciadas: hardware virtualizado, plataforma Kubernetes, aplicación, GitOps y CI/CD. A continuación se presenta el diagrama de arquitectura a alto nivel:

{{< figure
    src="architecture.png"
    alt="General Architecture from the DevOps Homelab"
    caption="General Architecture from the DevOps Homelab"
    default=true
    >}}

El principio organizador de la arquitectura es la **separación de responsabilidades por repositorio**:

| Repositorio        | Responsabilidad                                                              | Tecnología                                                    |
| ------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `infra-opentofu`   | Provisionamiento declarativo de VMs, configuración del SO y bootstrap GitOps | OpenTofu, `bpg/proxmox`, `siderolabs/talos`, `hashicorp/helm` |
| `gitops-manifests` | Fuente única de verdad para el estado deseado de la plataforma en Kubernetes | ArgoCD, Kustomize, Ingress-NGINX, External Secrets Operator   |
| `praxis-lab`       | Código fuente de la aplicación de laboratorios                               | Go, SQLite, React, TypeScript, Vite, xterm.js                 |

El flujo de control fluye en una sola dirección: **código → CI/CD → registro de imágenes → repositorio GitOps → ArgoCD → Kubernetes**. Ningún cambio se aplica directamente al clúster; toda mutación de estado pasa por un commit, lo que garantiza trazabilidad completa y reversibilidad.

---

## 3. Stack Tecnológico

### 3.1. Infraestructura, Virtualización y Sistema Operativo (`infra-opentofu`)

| Componente                      | Rol en la arquitectura                                                                                                                                                |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxmox VE**                  | Hypervisor tipo-1 bare-metal que hospeda las máquinas virtuales del clúster Kubernetes.                                                                               |
| **Talos Linux (Sidero Labs)**   | Distribución Linux inmutable, mínima y endurecida en seguridad, diseñada exclusivamente para Kubernetes. Sin SSH: se administra completamente vía API con `talosctl`. |
| **OpenTofu (>= 1.6.0)**         | Motor de Infraestructura como Código. Define de forma declarativa el cómputo, la configuración del sistema operativo y el bootstrap del pipeline GitOps.              |
| **Provider `bpg/proxmox`**      | Provisionamiento declarativo de las máquinas virtuales sobre Proxmox VE.                                                                                              |
| **Provider `siderolabs/talos`** | Configuración de las máquinas Talos, bootstrap del clúster y generación del `kubeconfig`.                                                                             |
| **Provider `hashicorp/helm`**   | Despliegue del chart oficial de ArgoCD directamente en el clúster Talos: el punto de entrada que "siembra" el motor GitOps.                                           |

Esta cadena es particularmente interesante desde la perspectiva SRE: la infraestructura se aprovisiona **por completo desde código** (HCL para las VMs, configuración Talos declarativa) y el clúster resultante es un sistema **inmutable** — no hay estado local modificable ni sesión de shell en los nodos —, una propiedad que elimina una clase entera de problemas de deriva de configuración.

### 3.2. GitOps y Plataforma Kubernetes (`gitops-manifests`)

| Componente                          | Rol en la arquitectura                                                                                                                                |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Kubernetes**                      | Orquestador de contenedores, bootstrapado y administrado por Talos Linux. Ejecuta la plataforma de laboratorios y los servicios de soporte.           |
| **ArgoCD**                          | Controlador GitOps: bucle de reconciliación continua pull-based, ApplicationSets y patrón App-of-Apps.                                                |
| **Kustomize**                       | Motor de templating declarativo: manifiestos `base/` con overlays (`dev`, `prod`) para parametrizar por entorno sin duplicar contenido.               |
| **Ingress-NGINX Controller**        | Proxy inverso L7 en el borde del clúster, configurado para soportar **upgrades de WebSockets** — requisito crítico para las terminales interactivas.  |
| **External Secrets Operator (ESO)** | Sincronización declarativa de secretos desde un gestor externo hacia Kubernetes, evitando el versionamiento de credenciales en el repositorio GitOps. |

### 3.3. Aplicación de Laboratorios (`praxis-lab`)

**Backend (Go):**

| Componente                                    | Rol en la arquitectura                                                                                                                                                                                    |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Go (>= 1.22)**                              | Lenguaje principal. Implementa la API REST y el servidor de WebSockets. La compilación estática produce binarios pequeños y eficientes, ideales para contenedores.                                        |
| **SQLite**                                    | Persistencia embebida, ligera y sin dependencias de un servicio externo. En Kubernetes se configura con **PVC para persistencia** y modo **WAL** (Write-Ahead Logging) para concurrencia lector/escritor. |
| **Gorilla WebSocket / `nhooyr.io/websocket`** | Comunicación full-duplex en tiempo real entre el navegador del estudiante y los procesos de terminal.                                                                                                     |
| **Linux PTY (`creack/pty`)**                  | Asignación de pseudoterminales dentro del contenedor para ejecutar shells de comandos interactivos.                                                                                                       |

**Frontend (React):**

| Componente                                  | Rol en la arquitectura                                                                                                                                   |
| ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **React (>= 18) + TypeScript**              | Interfaz de usuario de la SPA con tipado estático en componentes y servicios.                                                                            |
| **Vite**                                    | Compilación, empaquetado y servidor de desarrollo con hot-reloading.                                                                                     |
| **xterm.js** (+ addons `fit` y `web-links`) | Emulador de terminal en el navegador: consume el stream del WebSocket y lo renderiza como una sesión de shell real.                                      |
| **NGINX (Alpine)**                          | Servidor web estático dentro del contenedor del frontend: sirve la SPA y resuelve el enrutamiento del cliente (fallback a `index.html` para deep links). |

### 3.4. CI/CD, Contenedores y Entorno Local

| Componente                         | Rol en la arquitectura                                                                                                                                                                   |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Docker**                         | Construcción de imágenes multi-stage: el backend Go se compila en una etapa y se copia sobre una imagen base mínima; el frontend se construye con Vite y se sirve desde NGINX Alpine.    |
| **ttl.sh (Registry temporal)**     | Registro de imágenes anónimo y efímero con **etiquetado inmutable mediante Git commit SHA y TTL** (`ttl.sh/<uuid>/praxis:sha-<commit>`), eliminando la gestión de credenciales en CI/CD. |
| **GitHub Actions**                 | Pipelines de CI: triggers por push, ejecución de linting y pruebas, build de imágenes, publicación en ttl.sh y commit automático de las etiquetas en `gitops-manifests`.                 |
| **GNU Make**                       | Automatización de tareas tanto en el repositorio raíz (`homelab/Makefile`) como en la aplicación (`praxis-lab/Makefile`).                                                                |
| **Vim / Vi**                       | Editor estándar de terminal para la gestión y edición local de archivos de configuración y scripts.                                                                                      |

---

## 4. Flujo End-to-End: del Código al Laboratorio

El siguiente diagrama de secuencia describe el ciclo completo de una modificación de código hasta que el estudiante usa el cambio:

{{< figure
    src="sequence.png"
    alt="Sequence diagram"
    caption="Sequence diagram"
    default=true
    >}}

Y el flujo de ejecución de un laboratorio desde la perspectiva del estudiante:

{{< figure
    src="flow.png"
    alt="Execution flow"
    caption="Execution flow"
    default=true
    >}}

---

## 5. Principios de Diseño

La arquitectura se fundamenta en principios directamente aplicables a operaciones SRE/DevOps de producción:

- **Infraestructura 100% declarativa y reproducible**: desde las VMs en Proxmox hasta el chart de ArgoCD, todo estado se define en código versionado. El entorno completo puede reconstruirse íntegramente desde los repositorios.
- **Nodos inmutables**: Talos Linux elimina la administración de servidores tradicional (sin SSH, sin estado local mutable). La única vía de gestión es la API y su definición declarativa.
- **GitOps**: el repositorio `gitops-manifests` es la única fuente de verdad. ArgoCD reconcilia continuamente; cualquier desviación del estado del clúster se corrige o queda registrada como _drift_.
- **Imágenes inmutables y trazables**: el tag SHA vincula cada imagen desplegada con un commit exacto. El rollback es un cambio de etiqueta, no una reconstrucción.
- **Seguridad por diseño**: los secretos nunca residen en el repositorio (ESO los inyecta en runtime), el tráfico se segmenta con NetworkPolicies, y el acceso se controla con RBAC.
- **Desarrollo local de bajo fricción**: Makefiles unificados en ambos niveles permiten que cualquier tarea del pipeline se ejecute localmente con un solo comando.

---

## 6. Decision Records

Algunas decisiones merecen mención explícita:

| Decisión                                                                         | Alternativa                                                                | Justificación                                                                                                                                                                     |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxmox VE + Talos Linux**                                                     | Clúster Kubernetes gestionado en nube, o kubeADM con distros tradicionales | Un homelab tipo-1 con nodos inmutables elimina la superficie de administración y la deriva de configuración; Talos, al ser API-first, es gestionable íntegramente desde OpenTofu. |
| **OpenTofu con providers `bpg/proxmox` + `siderolabs/talos` + `hashicorp/helm`** | Pasos manuales con `talosctl` y `helm install`                             | El bootstrap completo (VMs → clúster → ArgoCD) debe ser idempotente, reproducible y auditable desde una única capa de IaC.                                                        |
| **SQLite + WAL + PVC** en lugar de PostgreSQL                                    | Servicio de base de datos externo                                          | Para un homelab, una BD embebida elimina una dependencia operativa. WAL habilita escrituras concurrentes junto con lecturas; el PVC garantiza la persistencia del volumen.        |
| **xterm.js + PTY sobre WebSocket**                                               | VNC, Serial Console, Jupyter                                               | Una terminal POSIX real con la mínima superficie: sin protocolo gráfico, sin servidor de notebook; solo un stream full-duplex de bytes.                                           |
| **ArgoCD App-of-Apps + ApplicationSets**                                         | Lista manual de Applications                                               | Permite gestionar _n_ aplicaciones (actualmente praxis-lab; pronto los laboratorios) desde un único Application de control, escalando sin duplicación.                            |
| **Registro efímero ttl.sh con etiquetas SHA**                                   | Registros privados tradicionales con credenciales / tags `latest`          | Elimina la necesidad de credenciales o tokens en CI/CD; el TTL maneja la limpieza automática mientras el SHA garantiza trazabilidad exacta.      |

---

## 7. Estructura de la Serie

Cada artículo de la serie profundiza en una capa de la arquitectura descrita en la sección 2:

| #   | Artículo                                                                                                           | Capa               |
| --- | ------------------------------------------------------------------------------------------------------------------ | ------------------ |
| 1   | **Introducción y Arquitectura General** (este artículo)                                                            | Visión global      |
| 2   | Infraestructura como Código: Proxmox, Talos Linux y OpenTofu (`bpg/proxmox`, `siderolabs/talos`, `hashicorp/helm`) | `infra-opentofu`   |
| 3   | GitOps con ArgoCD, Kustomize y gestión de secretos con ESO                                                         | `gitops-manifests` |
| 4   | CI/CD con GitHub Actions, Docker multi-stage y publicación efímera en ttl.sh                                       | CI/CD              |
| 5   | La aplicación praxis-lab: backend Go (REST, WebSockets, PTY, SQLite) y frontend React (xterm.js)                   | `praxis-lab`       |
| 6   | Observabilidad y resiliencia: métricas, logs, alertas y estrategia de recuperación                                 | Operaciones        |

---

## 8. Conclusiones

Este proyecto demuestra la implementación de un pipeline DevOps completo sobre hardware propio: **IaC** (OpenTofu aprovisionando Proxmox y Talos), **GitOps** (ArgoCD + Kustomize), **CI/CD** (GitHub Actions + Docker + ttl.sh) y una **aplicación de negocio** con requisitos no triviales de tiempo real (terminales interactivas vía WebSocket) y persistencia (SQLite en Kubernetes).

El homelab actúa como banco de pruebas: los patrones documentados son idénticos a los que se aplicarían en un clúster gestionado en nube pública, con la diferencia de que cada decisión —y cada error— es completamente explorable y reproducible.

En el siguiente artículo se profundiza en la capa de infraestructura: la cadena completa de aprovisionamiento —VMs en Proxmox con `bpg/proxmox`, bootstrap inmutable del clúster con `siderolabs/talos` y seed de ArgoCD con `hashicorp/helm`—.

---
