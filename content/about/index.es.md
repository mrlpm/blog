---
title: "Sobre mí"
description: "Perfil profesional, filosofía técnica y proyectos de Luis Pérez."
showDate: false
showAuthor: false
showReadingTime: false
showTableOfContents: true
showTaxonomies: false
---

{{< lead >}}
¡Hola! Soy **Luis Pérez**, entusiasta de la ciberseguridad, la fiabilidad de sistemas (SRE) y la ingeniería de plataformas.
{{< /lead >}}

Me apasiona diseñar y construir **infraestructura inmutable, declarativa y segura desde el diseño**. Utilizo mi homelab como banco de pruebas para simular entornos de producción con tecnologías cloud-native, automatización GitOps y desarrollo de herramientas internas.

---

## 🚀 Filosofía Técnica

1. **Infraestructura como Código (IaC):** Todo lo que se despliega debe estar definido en código versionado (OpenTofu/Terraform) para garantizar reproducibilidad total y cero deriva de configuración.
2. **Nodos Inmutables & API-First:** Prefiero sistemas operativos mínimos y endurecidos como **Talos Linux**, eliminando SSH y mutaciones manuales en favor de APIs declarativas.
3. **GitOps como Fuente de Verdad:** Los cambios de estado no se aplican directamente al clúster; fluyen a través de commits y reconciliación continua con **ArgoCD**.
4. **Seguridad por Diseño (DevSecOps):** Gestión de secretos desacoplada (External Secrets Operator), imágenes inmutables y segmentación de red en cada capa.

---

## 🛠️ Stack & Áreas de Interés

### Plataforma & Cloud Native

- **Orquestación & Sistemas:** Kubernetes, Talos Linux, Proxmox VE.
- **IaC & Automatización:** OpenTofu, Terraform, GNU Make.
- **GitOps & Delivery:** ArgoCD, Kustomize, GitHub Actions, Docker, `ttl.sh`.

### Ciberseguridad & Pentesting

- **Seguridad Ofensiva & Pentesting:** Pruebas de penetración, evaluación de vulnerabilidades en redes y aplicaciones web.
- **DevSecOps & Hardening:** External Secrets Operator (ESO), NetworkPolicies, RBAC, infraestructura inmutable, Threat Modeling.

### Desarrollo de Software

- **Backend:** Go (REST APIs, WebSockets, PTYs de Linux, SQLite con WAL).
- **Frontend:** React, TypeScript, Vite, Tailwind CSS, xterm.js.

---

## 🔬 Proyecto Principal: _praxis-lab_

**praxis-lab** es una plataforma de laboratorios Linux en línea ejecutada sobre mi propio homelab. Permite a estudiantes disponer de entornos de práctica interactivos (micromáquinas virtuales en pods aislados) accesibles desde el navegador web en tiempo real, orquestados íntegramente mediante Kubernetes y GitOps.

Puedes seguir la serie completa de su arquitectura y construcción en el [Blog](/posts/).

---

## 📬 Conecta Conmigo

- **GitHub:** [@mrlpm](https://github.com/mrlpm)
- **GitLab:** [@mrlpm](https://gitlab.com/mrlpm)
- **LinkedIn:** [luisperezgt](https://linkedin.com/in/luisperezgt)
- **X (Twitter):** [@luispmarin](https://x.com/luispmarin)
- **Email:** [luis.perez@proton.me](mailto:luis.perez@proton.me)
