

### **1. Définir les Pré-requis**

Avant de commencer, assurez-vous d'avoir :

- **Ansible installé** : Configurez votre machine avec Ansible et vérifiez l'accès aux serveurs cible via SSH.
- **Infrastructure cible** : Une ou plusieurs machines prêtes pour le déploiement des runners (physiques ou virtuelles).
- **Configuration des droits utilisateurs** : L'utilisateur utilisé par Ansible doit avoir les privilèges nécessaires pour installer des paquets et configurer des services.

````bash

playbooks/
├── site.yml          # Point d'entrée principal
├── roles/
│   ├── gitlab-runner/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   ├── templates/
│   │   ├── handlers/
│   │   │   ├── main.yml
│   │   ├── vars/
│   │   │   ├── main.yml
│   │   ├── defaults/
│   │   │   ├── main.yml
│   │   ├── files/
│   │   ├── meta/
│   │   │   ├── main.yml
│   │   ├── README.md
│   ├── common/       # Si d'autres rôles sont nécessaires (e.g., pour préparer les machines)
inventory/
├── hosts             # Fichier d'inventaire décrivant les machines cible
group_vars/
├── all.yml           # Variables globales
`````

### **3. Écriture du Playbook**

Un exemple de playbook pour déployer un GitLab Runner :

#### **Fichier `site.yml`**


````bash
- name: Deploy GitLab Runner
  hosts: runners
  become: yes
  roles:
    - gitlab-runner
`````

### **4. Développer le Rôle `gitlab-runner`**

#### **`tasks/main.yml`**

````bash
---
# Tâches principales pour déployer GitLab Runner
- name: Update and install dependencies
  apt:
    update_cache: yes
    name:
      - curl
      - git
    state: present

- name: Add GitLab Runner repository
  apt_repository:
    repo: 'deb https://packages.gitlab.com/runner/gitlab-runner/ubuntu/ focal main'
    state: present
    filename: 'gitlab-runner'

- name: Install GitLab Runner
  apt:
    name: gitlab-runner
    state: latest

- name: Register GitLab Runner
  command: >
    gitlab-runner register
    --non-interactive
    --url "{{ gitlab_url }}"
    --registration-token "{{ registration_token }}"
    --executor "{{ executor }}"
    --description "{{ runner_name }}"
    --tag-list "{{ runner_tags | join(",") }}"
    --run-untagged "{{ run_untagged }}"
    --locked "{{ locked }}"
  args:
    creates: /etc/gitlab-runner/config.toml

- name: Ensure GitLab Runner service is running
  service:
    name: gitlab-runner
    state: started
    enabled: yes

`````

#### **`defaults/main.yml`**

Les valeurs par défaut :

````bash
gitlab_url: "https://gitlab.com/"
registration_token: "YOUR_REGISTRATION_TOKEN"
executor: "docker"
runner_name: "My GitLab Runner"
runner_tags: []
run_untagged: true
locked: false
`````

#### **`templates/config.toml.j2`**

Si vous devez personnaliser davantage la configuration :

````bash
concurrent = {{ concurrent | default(1) }}
check_interval = {{ check_interval | default(0) }}

[[runners]]
  name = "{{ runner_name }}"
  url = "{{ gitlab_url }}"
  token = "{{ runner_token }}"
  executor = "{{ executor }}"
  {{ executor_specific_config }}

`````

### **5. Inventaire**

Un fichier `inventory/hosts` pourrait ressembler à ceci :

````bash
[runners]
runner1 ansible_host=192.168.1.10 ansible_user=ubuntu
runner2 ansible_host=192.168.1.11 ansible_user=ubuntu

`````

### **6. Gestion des Variables**

Les variables peuvent être définies dans `group_vars/all.yml` ou en ligne de commande :

````bash

gitlab_url: "https://gitlab.example.com/"
registration_token: "YOUR_PROJECT_REGISTRATION_TOKEN"
executor: "shell"
runner_name: "example-runner"
runner_tags:
  - "ci"
  - "docker"
````

### **7. Exécution du Playbook**

Lancer le playbook :

````bash
ansible-playbook -i inventory/hosts site.yml
`````

### **8. Tests et Débogage**

- **Mode verbeux** : Utilisez `-vvv` pour plus de détails.
- **Vérification sans appliquer** : Ajoutez `--check` pour un mode "dry-run".




### **1. Structure du fichier d'inventaire `hosts`**

Un fichier d'inventaire Ansible peut être écrit en plusieurs formats, comme **INI**, **YAML**, etc. Le fichier que vous avez montré est au format **INI**.

Voici une explication de chaque section et ligne de ce fichier d'inventaire.

---

#### **1.1. Définition des hôtes (Groupes d'hôtes)**

````bash
[servers]
server1 ansible_host=172.20.128.2
server2 ansible_host=172.20.128.3
server3 ansible_host=172.20.128.4

`````

- **[servers]** : Il s'agit d'un groupe d'hôtes. Les groupes permettent d'organiser les machines cibles et de spécifier des tâches communes à un sous-ensemble d'hôtes. Dans ce cas, le groupe `servers` contient trois serveurs identifiés par des alias (`server1`, `server2`, `server3`).
- **server1 ansible_host=172.20.128.2** : L'alias `server1` représente un hôte avec l'adresse IP `172.20.128.2`. Le paramètre `ansible_host` permet de définir explicitement l'adresse IP ou le nom d'hôte utilisé pour se connecter à cet hôte. Cela est utile si l'alias d'un serveur ne correspond pas à son adresse IP réelle.

Les autres serveurs (`server2` et `server3`) sont définis de la même manière avec leurs adresses IP respectives. Ainsi, lorsque vous exécutez une commande Ansible sur le groupe `servers`, elle sera appliquée à tous les serveurs définis sous ce groupe.

### **2. Explication des éléments de configuration**

#### **2.1. Groupes d'hôtes**

Les groupes d'hôtes, tels que `[servers]` dans cet exemple, permettent de regrouper plusieurs machines et de leur appliquer des configurations ou des tâches communes. Vous pouvez définir autant de groupes que nécessaire, par exemple :

````bash

[webservers]
web1 ansible_host=192.168.1.2
web2 ansible_host=192.168.1.3

[dbservers]
db1 ansible_host=192.168.1.10
db2 ansible_host=192.168.1.11

`````

### **5. Conclusion**

Le fichier d'inventaire **`hosts`** d'Ansible est crucial pour définir les hôtes sur lesquels les playbooks seront exécutés. Il permet également de définir des variables et de structurer les hôtes en groupes pour organiser et simplifier la gestion des tâches. L'exemple que vous avez fourni montre bien comment définir des hôtes cibles et comment forcer l'utilisation de Python 3 sur les hôtes distants.