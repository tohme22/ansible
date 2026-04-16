---
title: "Automatiser Cisco"
description: "ansible"
draft: false
weight: 5
---
## Création et utilisation d’un Playbook Ansible - Cisco

Dans cet exemple, nous allons automatiser la configuration de routers et switches Cisco avec **Ansible** : 

- `R1` et `R2`→ groupe `routers`  
- `SW1` → groupe `switches`  
- `routers` et `switches` regroupés dans le groupe `network`  

---


### 1. Définir l’inventaire `/etc/ansible/hosts`

Éditez le fichier `/etc/ansible/hosts` pour définir vos serveurs et groupes :  

```ini
[routers]
R1 ansible_host=10.1.1.1
R2 ansible_host=10.1.1.3
 
[switches]
SW1 ansible_host=10.1.1.2
 
[network:children]
routers
switches
 
[network:vars]
ansible_network_os=ios
ansible_user=admin
ansible_password=cisco
ansible_connection=network_cli
ansible_ssh_common_args='-oKexAlgorithms=+diffie-hellman-group14-sha1'
```
---

### 2. Configurer `/etc/ansible/ansible.cfg`

Créez (ou modifiez) le fichier `/etc/ansible/ansible.cfg` pour inclure les options nécessaires.  

Exemple :  

```yaml
[defaults]
inventory = /etc/ansible/hosts
host_key_checking = False
```

---

### 3. Tester la connectivité avec une commande ad hoc

Avant de lancer un playbook, vérifiez que vos serveurs répondent :  

```yaml
ansible all -m ping
R2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
SW1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
R1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

👉 Cela doit renvoyer **ping** pour chaque hôte.

---

### 4. Exemples de Playbook

#### Exemple 1 : Lister la configuration actuelle des équipements réseaux

```yaml
$ vim show.yml

---
- name: Cisco show version example
  hosts: network
  gather_facts: false

  tasks:
    - name: Execute show run sur les équipements réseaux
      ios_command:
        commands: show run
      register: output

    - name: print output
      ansible.builtin.debug:
        var: output.stdout_lines
```

#### Exemple 2 : Configurer un interface réseau d'un routeur

```yaml
$ vim int_routeur.yml

---
- name: Configure Router1
  hosts: R1
  gather_facts: false

  tasks:
    - name: Global config setting
      ios_config:
        lines:
          - interface g0/1
          - no shutdown
          - ip address 192.168.1.10 255.255.255.0
```

#### Exemple 3 : Modifier l'adresse IP d'un interface réseau d'un routeur

```yaml
$ vim ip_routeur.yml

---
- name: Configure Router1
  hosts: R1
  gather_facts: false

  tasks:
    - name: Global config setting
      ios_config:
        lines:
          - interface g0/1
          - no shutdown
          - ip address 192.168.2.10 255.255.255.0

    - name: Gather running config after changes
      ios_command:
        commands:
          - show ip int br
      register: after_config

    - name: Display new configuration
      ansible.builtin.debug:
        var: after_config.stdout_lines
```

---

### 5. Lancer les Playbooks

Exécutez le playbook avec la commande :  

```yaml
# Vérifier le syntaxe du playbook
$ ansible-playbook --syntax-check ip_routeur.yml
playbook: show.yml

# Tester l'exécution du playbook 
$ ansible-playbook --check ip_routeur.yml

# Exécuter le playbook
$ ansible-playbook ip_routeur.yml

PLAY [Configure Router1] **************************************************************************************************************

TASK [Global config setting] **********************************************************************************************************
running configuration on device
changed: [R1]

TASK [Gather running config after changes] ********************************************************************************************
ok: [R1]

TASK [Display new configuration] ******************************************************************************************************
ok: [R1] => {
    "after_config.stdout_lines": [
        [
            "Interface                  IP-Address      OK? Method Status                Protocol",
            "GigabitEthernet0/0         10.1.1.1        YES NVRAM  up                    up      ",
            "GigabitEthernet0/1         192.168.2.10    YES manual down                  down    ",
            "GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    ",
            "GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down"
        ]
    ]
}

PLAY RECAP ****************************************************************************************************************************
R1                         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```






