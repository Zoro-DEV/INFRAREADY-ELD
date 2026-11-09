# 🚀 Projet Fil Rouge : InfraReady
> Déploiement, automatisation, supervision et documentation d'un système d'information hybride Linux/Windows.

[![CI/CD Validation](https://github.com/Zoro-DEV/INFRAREADY-ELD/actions/workflows/validation.yml/badge.svg)](https://github.com/Zoro-DEV/INFRAREADY-ELD/actions/workflows/validation.yml)
[![Ansible](https://img.shields.io/badge/Ansible-Automation-red?logo=ansible)](https://www.ansible.com/)
[![Windows Server](https://img.shields.io/badge/Windows_Server-2025-blue?logo=windows)](https://www.microsoft.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu_Server-24.04_LTS-orange?logo=ubuntu)](https://ubuntu.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-e6522c?logo=prometheus)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Dashboard-f46800?logo=grafana)](https://grafana.com/)

---

## 📋 Table des matières
1. [Contexte du Projet](#-contexte-du-projet)
2. [Architecture Technique & Réseau](#-architecture-technique--réseau)
3. [Plan d'Adressage IP](#-plan-dadressage-ip)
4. [Structure du Dépôt GitHub](#-structure-du-dépôt-github)
5. [Prérequis & Installation](#-prérequis--installation)
6. [Automatisation avec Ansible](#-automatisation-avec-ansible)
7. [Supervision & Métriques](#-supervision--métriques)
8. [Intégration Continue (CI/CD)](#-intégration-continue-cicd)
9. [Rapport d'Incident](#-rapport-dincident)
10. [Auteurs](#-auteurs)

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
