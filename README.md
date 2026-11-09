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
│       └── validate.yml           # Pipeline CI/CD GitHub Actions (Ansible Lint & Syntax)
├── docs/
│   ├── architecture.md              # Documentation détaillée de l'architecture
│   ├── adressage-ip.md              # Plan de nommage et réseau
│   ├── securite.md                  # Politiques de sécurité et moindre privilège
│   └── incident.md                  # Fiche de diagnostic d'incident
├── inventory/
│   ├── hosts.yml                    # Inventaire de production du laboratoire
│   └── group_vars
│       └── windows              
│           ├── vars.yml             # Variables de communication et d'authentification
│           └── vault.yml            # Ansible vault
│
├── playbooks/
│   ├── linux.yml                    # Provisionnement et durcissement Ubuntu
│   ├── windows.yml                  # Configuration AD DS et services Windows
│   ├── maintenance.yml              # Playbook de mises à jour automatisées
│   └── verification.yml             # Tests post-déploiement et d'idempotence
├── monitoring/
│   └── prometheus.yml               # Configuration de scraping des métriques
├── roles/
├── scripts/
└── README.md                        # Documentation principale du dépôt
```

---


## ⚙️ Prérequis & Installation

### 1. Connectivité & Transport

Génération et déploiement de la clé SSH vers `LINUX01` :
```bash
# Génération de la clé Ed25519 sur OPS01
ssh-keygen -t ed25519 -C "ansible_user@infraready"

# Copie de la clé publique sur LINUX01
ssh-copy-id -i ~/.ssh/id_ed25519.pub ansible_user@192.168.1.201
```

Activation et configuration de WinRM sur `WIN-DC01` (PowerShell Administrateur) :
```powershell
# Configuration de WinRM pour la gestion à distance
winrm quickconfig -q
Set-Item WSMan:\localhost\Service\Auth\Basic $true
Set-Item WSMan:\localhost\Service\AllowUnencrypted $true

# Autoriser les ports dans le pare-feu Windows
New-NetFirewallRule -Name "WinRM-HTTP" -DisplayName "WinRM HTTP" -Enabled True -Direction Inbound -Protocol TCP -LocalPort 5985 -Action Allow
New-NetFirewallRule -Name "WinRM-HTTPS" -DisplayName "WinRM HTTPS" -Enabled True -Direction Inbound -Protocol TCP -LocalPort 5986 -Action Allow
```
### 2. Installation des dépendances Ansible sur `OPS01`

```bash
# Mise à jour et installation des paquets système
sudo apt update && sudo apt install -y ansible python3-pip python3-winrm

# Installation des collections Ansible requises
ansible-galaxy collection install ansible.windows community.windows ansible.posix

# Installation des dépendances Python WinRM
pip install pywinrm requests-credssp
```

## 🤖 Automatisation avec Ansible (Infrastructure as Code)

L'Infrastructure as Code (IaC) permet de décrire l'état cible des serveurs dans des fichiers versionnés et de garantir l'**idempotence** : une seconde exécution produit un résultat `changed=0 / failed=0`.

* **Inventaire (`inventory/hosts.yml`) :**

  ```yaml
  all:
    children:
      linux_servers:
        hosts:
          linux01:
            ansible_host: 192.168.1.201
            ansible_user: ansible_user
      windows_servers:
        hosts:
          win-dc01:
            ansible_host: 192.168.1.200
            ansible_user: Administrator
            ansible_connection: winrm
            ansible_winrm_server_cert_validation: ignore
  ```

* **Commandes de lancement des playbooks :**

  ```bash
  # Provisionnement Linux
  ansible-playbook -i inventory/hosts.yml playbooks/linux.yml
  
  # Provisionnement Windows
  ansible-playbook -i inventory/hosts.yml playbooks/windows.yml
  
  # Playbook de vérification d'idempotence
  ansible-playbook -i inventory/hosts.yml playbooks/verification.yml
  ```

## 📊 Supervision & Métriques

* **Prometheus (`OPS01` - Port 9090) :** Collecte les métriques système toutes les 15 secondes auprès des agents :
  * `Node Exporter` sur `LINUX01` (Port `9100`)
  * `Windows Exporter` sur `WIN-DC01` (Port `9182`)

* **Grafana (`OPS01` - Port 3000) :** Centralise la visualisation en temps réel (CPU, RAM, Disque `/data`, disponibilité `up`).

* **Seuil d'alerte configuré :** Alerte automatique en cas d'injoignabilité d'un hôte (`up == 0`) ou de saturation du stockage (`< 10%` d'espace libre).

## 🔄 Intégration Continue (CI/CD)

Le fichier `.github/workflows/validation.yml` s'exécute automatiquement à chaque `push` ou `pull request` afin d'assurer le contrôle qualité :

```yaml
name: Validation CI/CD
on: [push, pull_request]

jobs:
  ansible-validation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout du code
        uses: actions/checkout@v3

      - name: Installation de Python et Ansible-Lint
        run: |
          python -m pip install --upgrade pip
          pip install ansible-lint

      - name: Validation syntaxique des Playbooks
        run: |
          ansible-playbook --syntax-check playbooks/*.yml

      - name: Contrôle de qualité du code
        run: |
          ansible-lint playbooks/*.yml
```
