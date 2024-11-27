**README : Déploiement d'un Stack HAProxy avec 3 Nœuds WordPress via Ansible**

Introduction
------------

Ce dépôt contient les playbooks Ansible nécessaires pour déployer et gérer un environnement hautement disponible pour WordPress, composé de :

-   **HAProxy** : Un équilibreur de charge pour distribuer le trafic entre les différents nœuds WordPress.
-   **3 Nœuds WordPress** : Des instances WordPress identiques, chacune pouvant gérer la charge.

Le déploiement est entièrement automatisé grâce à Ansible, ce qui permet de reproduire facilement l'environnement et de le mettre à jour.

Prérequis
---------

-   **Ansible** : Installé et configuré sur l'un des trois nœuds.
-   **Inventaire Ansible** : Un fichier `hosts` contenant les adresses IP des trois nœuds.
-   **Clés SSH** : Des clés SSH sans mot de passe configurées pour permettre à l'utilisateur Ansible de se connecter sans interaction à tous les nœuds.

Structure du Répertoire
-----------------------

-   **playbooks/:** Contient les playbooks Ansible pour chaque étape du déploiement.
-   **roles/:** Contient les rôles Ansible réutilisables pour installer et configurer les différents composants.
-   **vars/:** Contient les variables utilisées dans les playbooks (e.g., noms d'hôtes, ports, etc.).
-   **group_vars/:** Contient les variables spécifiques à chaque groupe d'hôtes (e.g., les nœuds WordPress).

Utilisation
-----------

1.  **Modifier l'inventaire:**

    -   Ouvrez le fichier `hosts` et ajoutez les adresses IP des trois nœuds, en les regroupant selon leur rôle (HAProxy, WordPress).
2.  **Exécuter les playbooks:**

    -   Depuis le répertoire racine du projet, exécutez les playbooks suivants dans l'ordre :
        -   `ansible-playbook playbooks/prepare.yml` : Prépare les nœuds (mises à jour, installation de packages de base).
        -   `ansible-playbook playbooks/install_haproxy.yml` : Installe et configure HAProxy.
        -   `ansible-playbook playbooks/install_wordpress.yml` : Installe et configure WordPress sur les 3 nœuds.
