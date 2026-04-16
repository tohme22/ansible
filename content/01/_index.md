---
title: "Introduction à Ansible"
description: "ansible"
draft: false
weight: 1
---
## Introduction à Ansible

Ansible offre une **automatisation open source** qui simplifie les tâches et s'exécute partout. 

Ansible vous permet **d'automatiser pratiquement toutes les tâches**. 

Voici quelques cas d'utilisation courants :

- Éliminer les répétitions et simplifier les flux de travail.  
- Gérer et maintenir la configuration du système.  
- Déployer en continu des logiciels complexes.  
- Effectuer des mises à jour progressives sans temps d'arrêt.  

Ansible utilise des scripts simples et lisibles, appelés **`playbooks`**, pour automatiser vos tâches. 
Vous déclarez l’état souhaité d'un système local ou distant dans votre playbook. Ansible garantit que le système reste dans cet état.  

En tant que technologie d'automatisation, Ansible est conçu autour des principes suivants :  

#### Architecture sans agent
Faibles frais de maintenance en évitant l’installation de logiciels supplémentaires sur l’infrastructure informatique.  

#### Simplicité
Les playbooks d'automatisation utilisent une syntaxe **`YAML`** simple pour un code qui se lit comme de la documentation. 
Ansible est également décentralisé, utilisant **`SSH`** avec les identifiants du système d'exploitation pour accéder aux machines distantes.  

#### Évolutivité
Faites évoluer facilement et rapidement les systèmes que vous automatisez grâce à une conception modulaire qui prend en charge une large gamme de **`systèmes d'exploitation`**, de **`plates-formes cloud`** et de **`périphériques réseau`**.  

#### Prévisibilité
Lorsque le système est dans l’état décrit par votre playbook, Ansible ne change rien, même si le playbook s’exécute plusieurs fois.  

--------
## Cibles d’automatisation avec Ansible

Ansible est conçu pour être **sans agent** et **modulaire**, ce qui lui permet d’automatiser une vaste gamme de systèmes et de périphériques. Voici les principales catégories de nœuds pouvant être automatisés :

---

#### 🖥️ Infrastructure & Virtualisation
- **Serveurs Linux** – configuration système, gestion des paquets, services.
- **Hôtes Windows** – mises à jour, services, registre, intégration Active Directory.
- **VMware vSphere / ESXi** – cycle de vie des VM, modèles, instantanés.
- **Proxmox VE** – gestion des clusters, VM, stockage et réseau.
- **KVM/libvirt** – création et gestion de machines virtuelles.
- **OpenStack** – provisionnement des ressources de calcul, stockage et réseau.
- **Fournisseurs cloud** – automatisation AWS, Azure, GCP.

---

#### 📦 Conteneurs
- **Docker** – construction d’images, exécution de conteneurs, gestion réseaux et volumes.
- **Podman** – automatisation des conteneurs rootless.
- **Kubernetes / OpenShift** – déploiement d’applications, gestion des namespaces, RBAC.

---

#### 📡 Réseau & Périphériques de sécurité
- **Cisco** – routeurs, commutateurs, pare-feux.
- **Juniper, Arista** – automatisation du routage et du switching.
- **Fortinet, Palo Alto** – règles de pare-feu, VPN, automatisation de la sécurité.
- **F5 BIG-IP** – configuration de load balancer et WAF.
- **pfSense / OPNsense** – politiques de pare-feu et de routage.
- **Check Point** – automatisation des règles de sécurité et pare-feu.

---

#### 📊 Supervision & Services IT
- **Zabbix / Nagios / Prometheus** – configuration de la supervision et alertes.
- **ELK / OpenSearch** – gestion des journaux et piles SIEM.
- **FreeIPA / Active Directory** – gestion des utilisateurs, groupes et politiques.

---

#### ☁️ Outils SaaS & Collaboration
- **GitHub / GitLab** – dépôts, utilisateurs, contrôle d’accès, mise en place CI/CD.
- **Atlassian (Jira, Confluence)** – gestion de projet et flux documentaires.
- **Slack / Microsoft Teams** – bots, création de canaux, notifications.

---

#### 🛠️ Postes clients & Divers
- **macOS** – gestion de configuration et installation d’applications (via Homebrew).
- **Objets connectés (IoT)** – Raspberry Pi ou autres dispositifs avec accès SSH/API.
- **Imprimantes, UPS, IPMI/iLO/iDRAC** – mises à jour firmware, contrôle de l’alimentation.
- **Bases de données** – automatisation de schémas et configuration MySQL, PostgreSQL, MongoDB.

---

#### ✅ Principe clé
Si un système dispose de :
1. **Un accès SSH**, ou  
2. **Une interface API/REST**, ou  
3. **Une collection/module Ansible existant**  

➡️ Alors il peut être automatisé avec Ansible.

---

## Installation

#### Redhat/Fedora/AlmaLinux

Fedora Linux fournit à la fois le package **Ansible complet** et le package **ansible-core minimal** via les référentiels standard.  

Installez le package complet **ansible** :

```yaml
$ sudo dnf install ansible
$ ansible --version
```

#### Ubuntu/Debian/Kali

Ubuntu fournit des packages Ansible via une archive de packages personnels (PPA) qui contient des versions plus récentes que les référentiels standard.  

Les versions Ubuntu sont disponibles [dans un PPA ici](https://launchpad.net/~ansible/+archive/ubuntu/ansible).  

Configurez le PPA sur votre système et installez Ansible :  

```yaml
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

#### Windows

Vous ne pouvez pas utiliser un système **Windows** comme nœud de contrôle Ansible.  

👉 Consultez la documentation officielle : [Utiliser Windows comme nœud de contrôle](https://docs.ansible.com/ansible/latest/os_guide/intro_windows.html#windows-control-node)

--------
## Architecture générale

Ansible automatise la gestion des systèmes distants et contrôle leur état souhaité.  

![](../images/1.png?height=500&classes=border,shadow,inline)

Comme le montre la figure ci-dessus, la plupart des environnements Ansible reposent sur deux composants principaux :  

#### Nœud de contrôle
- Un système sur lequel Ansible est installé.  
- Contient **Ansible** et l’**Inventory**.  
- C’est depuis ce nœud que vous exécutez les commandes et gérez l’infrastructure.  

#### Nœud géré
- Les systèmes distants (**serveurs**, **hôtes**, **routeurs** etc.) qui sont administrés par Ansible.  
--------
## Composants clés

Ces composants sont communs à toutes les utilisations d'Ansible. Il est important de les comprendre avant de commencer à utiliser Ansible.  

#### Inventaire
- Liste de nœuds gérés organisés logiquement.  
- L’inventaire est créé sur le **nœud de contrôle** pour décrire les déploiements d’hôtes vers Ansible.  

#### Playbooks
- Contexte principal d'exécution d'Ansible, un **playbook** associe les nœuds gérés (hôtes) aux tâches.  
- Il contient des **variables**, des **rôles** et une liste ordonnée de **tâches**, et peut être exécuté de manière répétée.  
- Le playbook fonctionne comme une boucle implicite sur les hôtes et les tâches mappés, définissant comment les parcourir.  

#### Tâches
- Définissent une **action** à appliquer à un hôte géré.  
- Vous pouvez exécuter une tâche unique avec une commande ad hoc en utilisant `ansible`.  

#### Modules

Les **modules** sont le code ou les binaires qu’Ansible copie et exécute sur chaque nœud géré (lorsque nécessaire) pour accomplir l’action définie dans chaque **tâche**.  

Chaque module a une utilité particulière :  
- Administration des utilisateurs dans un type spécifique de base de données.  
- Gestion des interfaces VLAN sur un type particulier de périphérique réseau.  

Vous pouvez :  
- Invoquer un **seul module** avec une tâche.  
- Invoquer **plusieurs modules différents** dans un playbook.  

👉 Pour avoir une idée du nombre de modules incluses dans Ansible, consultez le [Collection Index](https://docs.ansible.com/ansible/latest/collections/index.html#list-of-collections).

--------
## Syntaxe

Ansible repose sur la syntaxe **YAML** (*Yet Another Markup Language*).  

- **Utilisation principale** : YAML est employé pour rédiger les **Playbooks**, qui définissent les tâches à exécuter sur les nœuds gérés.  
- **Format lisible par l’humain** : les fichiers YAML sont faciles à lire et à comprendre, même pour les débutants.  
- **Indentation obligatoire** : la structure repose sur l’indentation (espaces), ce qui rend la hiérarchie claire.  


#### Exemple d'un fichier écrit en YAML

Le fichier ci-dessous installe le paquet `httpd` (serveur web Apache) sur un hôte distant :

```yaml
---
- name: Installer Apache et activer le service
  hosts: webservers
  become: yes

  tasks:
    - name: Installer le paquet httpd
      ansible.builtin.dnf:
        name: httpd
        state: present
    
	- name: Activer et démarrer le service httpd
      ansible.builtin.service:
        name: httpd
        state: started
		enabled: yes
```
👉 Grâce à cette syntaxe, vous pouvez décrire facilement l’état souhaité de vos systèmes sans avoir besoin d’écrire du code complexe. 


 


