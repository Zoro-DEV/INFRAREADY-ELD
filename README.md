# 🚀 Projet Fil Rouge : InfraReady
> Déploiement, automatisation et supervision d’un système
d’information Linux/Windows

<p align="center">
  <img src="https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white" alt="Ansible">
  <img src="https://img.shields.io/badge/Windows_Server-0078D6?style=for-the-badge&logo=windows&logoColor=white" alt="Windows Server">
  <img src="https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu">
  <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" alt="Prometheus">
  <img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white" alt="Grafana">
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="GitHub Actions">
</p>

---

## 🎯 Contexte du Projet
L'entreprise fictive **InfraReady** disposait d'une infrastructure hétérogène (Linux/Windows) caractérisée par des configurations manuelles, un suivi complexe des mises à jour et des délais de diagnostic trop longs en cas d'incident. 

Ce projet a pour mission de livrer une plateforme de laboratoire standardisée, reproductible grâce à l'Infrastructure as Code (**Ansible**), supervisée en temps réel (**Prometheus / Grafana**) et qualifiée automatiquement via un pipeline **GitHub Actions**.

---

## 🏗️ Architecture Technique & Réseau
L'infrastructure repose sur 4 machines virtuelles hébergées sous **VMware Workstation**, cloisonnées par un routeur/pare-feu **pfSense**.

```text
                        [ RÉSEAU EXTÉRIEUR / WAN ]
                                    │
                                    ▼
                 [ ROUTEUR / PARE-FEU PFSENSE : 192.168.1.1 ]
                                    │
            ┌───────────────────────┼───────────────────────┐
            │                       │                       │
            ▼                       ▼                       ▼
     ┌──────────────┐        ┌──────────────┐        ┌──────────────┐
     │  WIN-DC01    │        │   LINUX01    │        │    OPS01     │
     │192.168.1.200 │        │192.168.1.201 │        │192.168.1.202 │
     ├──────────────┤        ├──────────────┤        ├──────────────┤
     │ • Windows 25 │        │ • Ubuntu 24  │        │ • Ubuntu 24  │
     │ • AD DS      │        │ • Nginx Web  │        │ • Ansible    │
     │ • DNS        │        │ • Ext4 /data │        │ • Prometheus │
     │ • WinExporter│        │ • NodeExport │        │ • Grafana    │
     └──────────────┘        └──────────────┘        └──────────────┘
```

---

## 📂 Structure du Dépôt
```text
INFRAREADY-ELD/
├── .github/
│   └── workflows/
│       └── validation.yml           # Pipeline CI/CD GitHub Actions (Ansible Lint & Syntax)
├── docs/
│   ├── architecture.md              # Documentation détaillée de l'architecture
│   ├── adressage-ip.md              # Plan de nommage et réseau
│   ├── securite.md                  # Politiques de sécurité et moindre privilège
│   ├── maintenance.md               # Procédures de màj et politique de snapshots
│   └── incident.md                  # Fiche de diagnostic d'incident (#INC-2026-DNS-01)
├── inventory/
│   └── hosts.yml                    # Inventaire de production du laboratoire
├── playbooks/
│   ├── linux.yml                    # Provisionnement et durcissement Ubuntu
│   ├── windows.yml                  # Configuration AD DS et services Windows
│   ├── maintenance.yml              # Playbook de mises à jour automatisées
│   └── verification.yml             # Tests post-déploiement et d'idempotence
├── monitoring/
│   ├── prometheus.yml               # Configuration de scraping des métriques
│   └── dashboards/
│       └── infraready-overview.json # Export du tableau de bord Grafana
├── scripts/
│   └── check_connectivity.sh        # Script de validation inter-machines
└── README.md                        # Documentation principale du dépôt
```

---

## ⚙️ Prérequis & Installation

