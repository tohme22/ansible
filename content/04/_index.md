---
title: "Automatiser Linux"
description: "ansible"
draft: false
weight: 4
---
## Création et utilisation d’un Playbook Ansible - Linux

Dans cet exemple, nous allons automatiser la configuration de trois serveurs Linux avec **Ansible** :  

- `serveura` → groupe `dev`  
- `serveurb` → groupe `prod`  
- `serverc` → groupe `database`  
- `dev` et `prod` regroupés dans le groupe `webservers`  

---
### 1. Préparation des connexions

#### Donnez à votre hôte de contrôle un nom d'hôte :

```yaml
[atohme@localhost ~]$ hostnamectl set-hostname controle
[atohme@localhost ~]$ bash
[atohme@controle ~]$ 
```

#### Configurer le mappage entre adresses IP et noms d’hôte

> 💡 **Remarque** : Normalement, les adresses IP de ces serveurs sont définies dans un **serveur DNS**.  
> Pour vos tests en laboratoire, vous pouvez utiliser le fichier `/etc/hosts` de votre machine de contrôle pour associer des noms (comme `serveura`, `serveurb`, `serverc`) à leurs adresses IP respectives.

```ini
$ sudo vim /etc/hosts

192.168.122.164 serveura
192.168.122.140 serveurb
192.168.122.163 serveurc
```

#### Configurer les connexions SSH avec les nœuds à gérer

```yaml
# Créer la paire de clés SSH sur l'hôte de contrôle
$ ssh-keygen 
Generating public/private rsa key pair.
...

# Copier la clé publique vers chaque serveur à gérer
$ ssh-copy-id atohme@serveura
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
atohme@serveura's password: 

Number of key(s) added: 1

$ ssh-copy-id atohme@serveurb
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
atohme@serveurb's password: 

Number of key(s) added: 1

$ ssh-copy-id atohme@serveurc
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
atohme@serveurc's password: 

Number of key(s) added: 1

```

### 2. Définir l’inventaire `/etc/ansible/hosts`

Éditez le fichier `/etc/ansible/hosts` pour définir vos serveurs et groupes :  

```ini
$ sudo vim /etc/ansible/hosts 

[dev]
serveura

[prod]
serveurb

[database]
serveurc

[webservers:children]
dev
prod
```

---

### 3. Configurer `/etc/ansible/ansible.cfg`

Créez (ou modifiez) le fichier `/etc/ansible/ansible.cfg` pour inclure les options nécessaires.  

Exemple:  

```ini
$ sudo vim /etc/ansible/ansible.cfg

[defaults]
inventory = /etc/ansible/hosts
remote_user = atohme
ask_pass = false
host_key_checking = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = true
```

---

### 4. Tester la connectivité avec une commande ad hoc

Avant de lancer un playbook, vérifiez que vos serveurs répondent :  

```yaml
$ ansible all -m ping
BECOME password: 

serveura | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
serveurb | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
serveurc | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
```

👉 Cela doit renvoyer **pong** pour chaque hôte.

---

### 5. Exemple de Playbook

Créons un playbook `webservers.yml` pour installer Apache (`httpd`), activer le pare-feu, et déployer une page web simple :  

```yaml
$ vim webservers.yml

---
- name: Configuration des serveurs web
  hosts: webservers
  become: yes

  tasks:
    - name: Installer le paquet httpd
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Démarrer et activer le service httpd
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

    - name: Configurer le pare-feu
      ansible.posix.firewalld:
        port: 80/tcp
        permanent: yes
        immediate: yes
        state: enabled

    - name: Créer le fichier index.html
      ansible.builtin.copy:
        content: |
          Mon site web sur {{ inventory_hostname }}
        dest: /var/www/html/index.html
```

---

### 6. Lancer le Playbook

Exécutez le playbook avec la commande :  

```yaml
# Vérifie uniquement la syntaxe du playbook (ne l’exécute pas)
$ ansible-playbook --syntax-check webservers.yml
playbook: webservers.yml

# Fait un "dry-run" : simule l’exécution pour voir ce qui serait changé, sans rien appliquer
$ ansible-playbook --check webservers.yml

# Exécute réellement le playbook et applique toutes les tâches sur les hôtes
$ ansible-playbook webservers.yml
BECOME password: 

PLAY [Configuration des serveurs web] ***************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [serveurb]
ok: [serveura]

TASK [Installer le paquet httpd] *********************************************************************************
changed: [serveurb]
changed: [serveura]

TASK [Démarrer et activer le service httpd] **********************************************************************
changed: [serveurb]
changed: [serveura]

TASK [Configurer le pare-feu] ************************************************************************************
changed: [serveurb]
changed: [serveura]

TASK [Créer le fichier index.html] *******************************************************************************
changed: [serveurb]
changed: [serveura]

PLAY RECAP *******************************************************************************************************
serveura                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serveurb                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

---

### 7. Vérifier le résultat

Vous pouvez tester l’accès aux serveurs web avec :  

```yaml
$ curl http://serveura
$ curl http://serveurb
```

👉 Vous devriez voir la page :  
`Mon site web sur serveura`  
ou  
`Mon site web sur serveurb`  

---

### 8. Tester avec des commandes ad hoc

Vous pouvez aussi vérifier l’installation du paquet **httpd** via une **commande ad hoc** :  

```yaml
# Exécute la commande "dnf list httpd" sur tous les hôtes du groupe webservers
$ ansible webservers -a "dnf list httpd"
```

Ou vérifier l’état du service :  

```yaml
# Exécute la commande "systemctl status httpd" sur tous les hôtes du groupe webservers
$ ansible webservers -a "systemctl status httpd"
```
---

✅ Vous avez maintenant un inventaire, un fichier de configuration, un playbook et des tests pour automatiser vos serveurs web avec Ansible !


