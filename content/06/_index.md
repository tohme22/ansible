---
title: "Gathering Facts"
description: "ansible"
draft: false
weight: 6
---
### C’est quoi les “facts” ?
Les **facts** sont des **variables** collectées automatiquement par Ansible (via le module **`setup`**) au début d’un play.  
Elles décrivent chaque hôte : OS, version, CPU/RAM, interfaces réseau, adresses IP, FQDN, etc.  
Exemples : `ansible_fqdn`, `ansible_hostname`, `ansible_os_family`, `ansible_distribution`, `ansible_memtotal_mb`, `ansible_default_ipv4.address`, etc.

**À quoi ça sert ?**
- **Conditionner** des tâches (selon l’OS, la version, la RAM…)
- **Remplir des templates/fichiers** avec des infos de l’hôte
- **Diagnostiquer** (afficher des infos pendant un play)

---

### Comment voir les facts
Ad hoc (pour un aperçu rapide) :
```bash
# Tous les facts
ansible all -m setup

# Filtrer (ex. infos IP par défaut)
ansible all -m setup -a 'filter=ansible_default_ipv4'
```

---

### Comment les utiliser (exemples)

#### 1) Afficher quelques facts
```yaml
---
- name: Exemple d’utilisation des facts
  hosts: all
  gather_facts: true
  tasks:
    - name: Afficher le FQDN et l’IP principale
      ansible.builtin.debug:
        msg: >
          Hôte={{ ansible_fqdn }},
          IP={{ ansible_default_ipv4.address }},
          OS={{ ansible_distribution }} {{ ansible_distribution_version }}
```

### 2) Conditionner une tâche selon l’OS
```yaml
- name: Installer httpd seulement sur les systèmes de famille RedHat
  hosts: all
  gather_facts: true
  become: yes
  tasks:
    - name: Installer httpd
      ansible.builtin.dnf:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
```

#### 3) Insérer des facts dans un fichier
```yaml
- name: Créer un index.html avec des facts
  hosts: web
  gather_facts: true
  become: yes
  tasks:
    - name: Générer /var/www/html/index.html
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: |
          Hôte : {{ ansible_fqdn }}
          IP   : {{ ansible_default_ipv4.address }}
          OS   : {{ ansible_distribution }} {{ ansible_distribution_version }}
```

---

### Comment désactiver la collecte (si inutile)
Utilisz l'option **`gather_facts: false`** pour désactiver la collection des facts et accélérer un play.

```yaml
- name: Play sans collecte de facts
  hosts: all
  gather_facts: false
  tasks:
    - name: Faire une action sans dépendre des facts
      ansible.builtin.command:
        cmd: echo "Pas de facts ici"
```

---
