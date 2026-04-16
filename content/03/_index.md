---
title: "Créer des Playbooks"
description: "ansible"
draft: false
weight: 3
---
### Rôle des Playbooks Ansible

Un **playbook** est un fichier écrit en **YAML** qui décrit l’état souhaité d’un ou plusieurs systèmes gérés par Ansible.  
Il permet d’automatiser des **tâches** de manière **déclarative** (on décrit *ce que l’on veut*, pas *comment le faire*).  

Les playbooks permettent par exemple de :  
- Installer et configurer des logiciels  
- Déployer des applications  
- Gérer des services (démarrage, arrêt, redémarrage)  
- Configurer le réseau, les utilisateurs, la sécurité, etc.  

👉 Contrairement aux **commandes ad hoc**, qui ne lancent qu’une seule action, un playbook peut contenir une **séquence ordonnée de tâches** organisées et réutilisables.

---

### Utiliser Vim pour écrire des fichiers YAML

Lorsque vous éditez des fichiers **YAML** (`.yml`) avec **vim**, il est recommandé de configurer votre éditeur pour faciliter la lecture et éviter les erreurs d’indentation.  

Vous pouvez créer un fichier **`.vimrc`** dans votre répertoire personnel (`~/.vimrc`) afin que ces réglages soient appliqués automatiquement à chaque ouverture de vim.  

**Exemple du fichier `.vimrc`**

```yaml
filetype plugin indent on
syntax on
set cuc nu et sw=2 ts=2 sts=2 
```

- Activer la détection de type de fichier et l’indentation automatique : *`filetype plugin indent on`*
- Activer la coloration syntaxique : *`syntax on`*
- Mettre en évidence la colonne du curseur: *`set cuc       `*       
- Numéroter les lignes : *`set nu             `*
- Remplacer les tabulations par des espaces : *`set et`*      
- Largeur d'indentation = 2 espaces : *`set sw=2`*           
- Taille des tabulations = 2 espaces : *`set ts=2`*         
- Décalage des tabulations = 2 espaces : *`set sts=2`*    

Avec cette configuration, vous évitez les erreurs fréquentes de tabulation dans les fichiers YAML et facilitez l’écriture de vos playbooks Ansible.

---
### Syntaxe générale d’un Playbook

Un playbook est composé d’une ou plusieurs **plays**.  
Chaque play associe un ou plusieurs **hôtes** (ou groupes d’hôtes) à une série de **tâches**.

#### Exemple de structure minimale
```yaml
---
- name: Nom du play (description)
  hosts: nom_du_groupe ou hôte
  become: yes     # Exécuter avec privilèges (sudo)

  tasks:
    - name: Description de la tâche
      module_ansible:
        parametre1: valeur1
        parametre2: valeur2
```

#### Exemple concret
```yaml

$ vim playbook1.yml

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

---

### Aide sur les modules

Chaque tâche dans un playbook repose sur un **module Ansible**.  

#### Lister tous les modules
```yaml
$ ansible-doc -l
```
👉 Cette commande affiche la liste de tous les modules disponibles avec leur description.  

#### Afficher la documentation d’un module précis

Pour découvrir les options disponibles pour un module, utilisez la commande :  

```yaml
ansible-doc <nom_module>
```

**Exemple :**
```yaml
$ ansible-doc dnf
> ANSIBLE.BUILTIN.DNF    (/usr/lib/python3.9/site-packages/ansible/modules/dnf.py)

        Installs, upgrade, removes, and lists packages and groups with the `dnf' package
        manager.

ADDED IN: version 1.9 of ansible-core

OPTIONS (= is mandatory):
...
```

#### Astuce dans l’aide interactive
Une fois la documentation d’un module ouverte, vous pouvez taper :  

```
/EX

EXAMPLES:

- name: Install the latest version of Apache
  ansible.builtin.dnf:
    name: httpd
    state: latest

- name: Install Apache >= 2.4
  ansible.builtin.dnf:
    name: httpd >= 2.4
    state: present
...
```

👉 Cela affiche directement les **exemples d’utilisation** (syntaxe à copier dans vos playbooks).

---

### Exécution d’un Playbook

Pour exécuter un playbook, utilisez la commande :  

```bash
$ ansible-playbook nom_du_playbook.yml
```

#### Options utiles
- `-i inventory_file` → préciser un fichier d’inventaire si différent de la config par défaut.  
- `-l groupe` → limiter l’exécution à un groupe d’hôtes.  
- `--syntax-check` → vérifier la syntaxe du fichier 
- `--check` → exécuter en mode simulation (*dry-run*) pour voir les changements sans les appliquer.  
- `-v` ou `-vvv` → augmenter le niveau de verbosité (debug).  

---

### Bonnes pratiques
- Toujours commencer le playbook avec `---` (début YAML).  
- Donner des noms clairs aux **plays** et aux **tâches**.  
- Utiliser `become: yes` pour les actions nécessitant des privilèges root.  
- Tester vos playbooks avec `--check` avant une exécution réelle.  
- Utiliser `ansible-doc` pour explorer les modules et leurs paramètres.  

---

✅ Avec les playbooks, Ansible devient un outil puissant pour **industrialiser l’automatisation** et assurer la cohérence de vos infrastructures.
