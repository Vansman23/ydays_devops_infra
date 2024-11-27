
Ce fichier Docker Compose permet de déployer un environnement multi-conteneurs comprenant :

**HAPROXY** : un conteneur de répartition de charge.

**ANSIBLE_WP01** : un conteneur qui exécute des playbooks Ansible.

**WP02 et WP03** : des conteneurs Ubuntu pour exécuter des scripts et interagir avec Ansible.


# 2. Description des Services Docker

#### **2.1. Service HAPROXY**

Ce service utilise l'image officielle **haproxy:2.7** pour créer un conteneur qui agira en tant que répartiteur de charge.

- **Nom du conteneur** : `HAPROXY`
- **Ports exposés** :
    - `80:80` : pour le trafic HTTP
    - `70:70` : un port supplémentaire, probablement pour une interface ou une autre application interne.
- **Volumes** :
    - **./haproxy** : répertoire local contenant les fichiers de configuration personnalisés de HAProxy.
    - **./haproxy/haproxy.cfg** : fichier de configuration de HAProxy monté en mode lecture seule (`ro`).
- **Réseau** :
    - Le conteneur est attaché au réseau `static-network` avec une adresse IP statique définie par la variable d'environnement `${HAPROXY_IPV4_ADDRESS}`.

#### **2.2. Service ANSIBLE_WP01**

Ce conteneur utilise l'image **ansible:latest** pour exécuter des tâches d'automatisation via des playbooks Ansible.

- **Nom du conteneur** : `ANSIBLE_WP01`
- **Volumes** :
    - **./ansible-playbooks** : répertoire local contenant les playbooks Ansible à exécuter dans le conteneur.
- **Réseau** :
    - Le conteneur est connecté au même réseau `static-network` avec une adresse IP statique définie par la variable `${ANSIBLE_WP01_IPV4_ADDRESS}`.

> **Remarque** : Les lignes de commande (`command`) sont actuellement commentées. Il est suggéré d'utiliser ces lignes pour personnaliser le comportement du conteneur, comme l'exécution de commandes d'installation ou d'un script au démarrage.

#### **2.3. Service WP02 et WP03**

Ces services sont basés sur l'image **ubuntu_ssh:latest**, qui démarre un conteneur Ubuntu avec la possibilité d'exécuter des commandes à distance via SSH.

- **Nom du conteneur** : `WP02` et `WP03` respectivement.
- **Réseau** :
    - Chaque conteneur est connecté au réseau `static-network` avec des adresses IP statiques définies par les variables `${WP02_IPV4_ADDRESS}` et `${WP03_IPV4_ADDRESS}`.
- **Volumes** :
    - **./bash** : répertoire local contenant des scripts (probablement des scripts bash à exécuter dans le conteneur).


# 3. Réseau Docker

Tous les services sont connectés à un réseau Docker interne, défini comme `static-network`, de type **bridge**.

#### **Configuration du réseau**

- **Sous-réseau** : `172.20.0.0/16`
- **Réseau statique** : Chaque conteneur a une adresse IP statique, spécifiée dans les variables d'environnement (ex. `${HAPROXY_IPV4_ADDRESS}`).

# 4. Variables d'Environnement

Les adresses IP des conteneurs sont gérées via des variables d'environnement. Ces variables doivent être définies dans un fichier `.env` ou dans l'environnement du système avant de démarrer les conteneurs.

Exemple de fichier `.env`

``bash``
```bash
HAPROXY_IPV4_ADDRESS=172.20.0.2 
ANSIBLE_WP01_IPV4_ADDRESS=172.20.0.3 
WP02_IPV4_ADDRESS=172.20.0.4 
WP03_IPV4_ADDRESS=172.20.0.5
```

### **5. Explications supplémentaires et personnalisation**

#### **HAPROXY**

Vous pouvez personnaliser la configuration de HAProxy en modifiant le fichier `haproxy.cfg`. Ce fichier définit les règles de routage du trafic entre les différents services.

#### **ANSIBLE_WP01**

Ce service est conçu pour exécuter des playbooks Ansible sur les autres conteneurs. Il est possible d'y ajouter des commandes pour automatiser l'installation ou la configuration d'applications.

#### **WP02 et WP03**

Ces conteneurs sont prévus pour exécuter des commandes à distance via SSH. Vous pouvez y déployer des scripts bash pour effectuer des tâches spécifiques ou les gérer à l'aide d'Ansible.

### **6. Commandes Docker Compose**

Voici quelques commandes de base pour démarrer et gérer vos conteneurs à l'aide de Docker Compose :

- **Démarrer les services** :
````bash
docker-compose up -d
````

- Cette commande démarre tous les services définis dans le fichier `docker-compose.yml` en arrière-plan.
    
- **Arrêter les services** :

````bash
docker-compose down
`````

- Cette commande arrête tous les services et supprime les conteneurs.
    
- **Vérifier l'état des conteneurs** :
    
    ````bash
    docker-compose ps
`````

- Affiche l'état de tous les conteneurs associés à votre projet Docker Compose.
    

---

### **7. Problèmes courants et solutions**

- **Problème de réseau** : Si les conteneurs ne parviennent pas à se connecter entre eux, assurez-vous que le réseau `static-network` est correctement défini et que les adresses IP sont disponibles dans le sous-réseau.

- **Problème de permissions avec les volumes** : Vérifiez les permissions des répertoires et fichiers montés dans les conteneurs. Si nécessaire, ajustez-les avec `chmod`.