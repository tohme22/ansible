---
title: "Configurer Ansible "
description: "ansible"
draft: false
weight: 2
---
## Inventaire 

Ansible automatise les tâches sur les **nœuds gérés**, ou « **hôtes** », de votre infrastructure à l'aide d'une liste ou d'un groupe de listes appelé **inventaire**.  

Un inventaire est constitué d'une ou plusieurs **sources d'inventaire**.  
- Cela peut être une simple liste de noms d'hôtes passée en ligne de commande.  
- Mais la plupart du temps, les utilisateurs créent des **fichiers d'inventaire**.  

Votre inventaire définit :  
- Les **nœuds gérés** que vous automatisez.  
- Les **groupes** d’hôtes pour les cibler ou définir des variables en masse.  
- Les **variables** associées à ces hôtes.  

---

#### Emplacement et définition

L'inventaire le plus simple est un fichier unique contenant une liste d'hôtes et de groupes.  
- Emplacement par défaut : **`/etc/ansible/hosts`**
- Vous pouvez aussi spécifier une ou plusieurs sources d'inventaire avec l’option `-i <path or expression>` en ligne de commande.  

---

#### Formats d’inventaire

Vous pouvez créer votre inventaire dans plusieurs formats, selon les **plugins d’inventaire** disponibles.  
Les plus courants sont :  
- **INI**  
- **YAML**  

---

#### Exemple d’inventaire INI

```ini
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

👉 Les titres entre crochets `[]` correspondent aux **groupes**.  

Ces groupes servent à :  
- Classer les hôtes.  
- Déterminer quels hôtes cibler pour l’automatisation.  

---
#### Groupes dans des groupes

Vous pouvez regrouper des groupes dans d’autres groupes parents.  

Cela permet d’appliquer une configuration automatisée directement à l’ensemble du **groupe parent**.  

##### Exemple
Les serveurs web de production situés à **Montréal** et **Toronto** peuvent être regroupés dans un groupe parent `webservers` :  

```ini
[prod-mtl]
server3.example.com
server4.example.com

[prod-tor]
server5.example.com
server6.example.com

[webservers:children]
prod-mtl
prod-tor
```
- `prod-mtl` et `prod-tor` sont deux groupes définissant chacun des serveurs spécifiques.  
- Le groupe `webservers:children` inclut ces deux groupes.  
- Ainsi, toute configuration appliquée au groupe `webservers` sera automatiquement appliquée aux serveurs de **Montréal** et **Toronto**.

---

#### Groupes par défaut

Même sans définir de groupes, Ansible crée automatiquement deux groupes :  

- **all** : contient tous les hôtes.  
- **ungrouped** : contient les hôtes qui n’appartiennent à aucun autre groupe.  

Chaque hôte appartient donc **au moins à deux groupes** :  
- `all` et `ungrouped`  
- ou `all` et un autre groupe.  

##### Exemple  
- `mail.example.com` appartient à `all` et `ungrouped`.  
- `two.example.com` appartient à `all` et `dbservers`.  

---

#### Variables d’hôtes

Vous pouvez affecter des **variables spécifiques à un hôte** directement dans votre inventaire.  

##### Exemple:
```ini
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

##### Exemple avec port SSH personnalisé :
```ini
[production]
badwolf.example.com:5309
```

---

#### Variables de connexion

Les **variables de connexion** configurent les plugins `host`, `connection` et `user`.  

##### Exemple :
```ini
[routers]
R1 ansible_host=10.1.1.1
R2 ansible_host=10.1.1.3

[switches]
SW1 ansible_host=10.1.1.2

[network_devices:children]
routers
switches

[network_devices:vars]
ansible_connection=ssh
ansible_user=admin
ansible_password=cisco
```
Ces paramètres permettent d’indiquer à Ansible **comment se connecter** aux hôtes (SSH, utilisateur spécifique, etc.).

---
## Le fichier de configuration Ansible

Le fichier **ansible.cfg** permet de définir le comportement par défaut d’Ansible.  

Il peut exister à trois niveaux :  
1. Dans le répertoire de travail courant (`./ansible.cfg`)  
2. Dans le répertoire personnel de l’utilisateur (`~/.ansible.cfg`)  
3. Dans le fichier global du système (`/etc/ansible/ansible.cfg`)  

👉 L’ordre de priorité est le suivant : **local > utilisateur > global**.  

---

#### Structure du fichier `ansible.cfg`

Le fichier est organisé en **sections** (INI-style) :

#### `[defaults]`
Paramètres généraux par défaut :  
- `inventory = /etc/ansible/hosts` → chemin du fichier d’inventaire par défaut  
- `remote_user = atohme` → utilisateur distant utilisé pour exécuter les tâches 
- `ask_pass = false`  → ansible ne demandera pas de mot de passe SSH à chaque connexion
- `host_key_checking = false` → désactive la vérification des clés SSH  
- `timeout = 10` → délai d’expiration de la connexion SSH (en secondes)  

---

#### `[privilege_escalation]`
Paramètres liés à l’élévation de privilèges (sudo) si les hôtes sont des machines Linux:  
- `become = true` → active sudo par défaut  
- `become_method = sudo` → méthode d’élévation utilisée  
- `become_user = root` → utilisateur cible par défaut  
- `become_ask_pass = true` → demande (ou non) le mot de passe sudo  

---

#### `[ssh_connection]`
Options de connexion SSH :  
- `ssh_args = -o ControlMaster=auto -o ControlPersist=60s` → permet de réutiliser la connexion SSH pour accélérer les tâches  
- `pipelining = true` → active le pipelining pour améliorer les performances  

---

#### Exemple du fichier ansible.cfg

```yaml
# Paramètres généraux par défaut
[defaults]
inventory =/etc/ansible/hosts
remote_user = atohme
ask_pass = false
host_key_checking = false

# Utiliser juste avec les noeuds Linux
[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = true

# Options de connexion SSH
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = true
```



