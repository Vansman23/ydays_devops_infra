La configuration HAProxy fournie permet de configurer un répartiteur de charge HTTP pour distribuer le trafic entre plusieurs serveurs backend. Elle inclut également une interface de statistiques accessible via un navigateur.

---

#### **1.1. Section `global`**

La section `global` définit les paramètres globaux pour l'ensemble de l'instance HAProxy.

````bash
global
  log /dev/log    local0
  log /dev/log    local1 notice
  maxconn 4096
`````

- **log /dev/log local0** et **log /dev/log local1 notice** : Ces lignes définissent les sources de logs pour HAProxy. Les logs sont envoyés au journal système (`/dev/log`) avec différents niveaux de priorité.
- **maxconn 4096** : Limite le nombre maximal de connexions simultanées à 4096.

---

#### **1.2. Section `defaults`**

La section `defaults` configure les paramètres par défaut pour tous les fronts et backends.

````bash
defaults
  log global
  mode http
  option httplog
  option dontlognull
  timeout connect 5000
  timeout client 50000
  timeout server 50000
`````

- **log global** : Active les paramètres de logging globaux définis dans la section `global`.
- **mode http** : Définit HAProxy pour qu'il fonctionne en mode HTTP (plutôt que TCP).
- **option httplog** : Active la journalisation des requêtes HTTP.
- **option dontlognull** : Évite de journaliser les connexions nulles (sans requêtes).
- **timeout connect 5000** : Définit un délai d'attente de 5 secondes pour établir une connexion avec un serveur backend.
- **timeout client 50000** et **timeout server 50000** : Définissent un délai d'attente de 50 secondes pour les interactions avec le client et le serveur backend respectivement.

#### **1.3. Section `listen stats`**

La section `listen stats` définit une interface web de statistiques pour surveiller l'état de HAProxy.

````bash
listen stats
  bind 0.0.0.0:70
  mode http
  stats enable
  stats hide-version
  stats scope .
  stats realm Haproxy\ Statistics
  stats uri /
  stats auth admin:"admin"
  stats auth user:pass
`````

- **bind 0.0.0.0:70** : Permet d'écouter sur toutes les interfaces réseau sur le port `70` pour l'interface de statistiques.
- **mode http** : Définit que l'interface de statistiques fonctionne en mode HTTP.
- **stats enable** : Active l'interface de statistiques.
- **stats hide-version** : Cache la version de HAProxy dans l'interface de statistiques.
- **stats scope .** : Permet de définir les statistiques à afficher pour tous les serveurs.
- **stats realm Haproxy\ Statistics** : Définit le domaine de l'authentification pour accéder aux statistiques.
- **stats uri /** : L'URL pour accéder à l'interface de statistiques (par défaut, `/`).
- **stats auth admin:"admin"** et **stats auth user:pass** : Spécifie les informations d'authentification pour accéder à l'interface de statistiques avec les utilisateurs `admin` et `user`, avec les mots de passe `admin` et `pass` respectivement.

#### **1.4. Section `frontend balancer`**

La section `frontend balancer` définit l'interface qui reçoit les requêtes des clients. Elle redirige ces requêtes vers le backend approprié.

````bash

frontend balancer
  bind 0.0.0.0:80
  default_backend web_backends
`````

- **bind 0.0.0.0:80** : HAProxy écoute sur toutes les interfaces réseau et le port `80` (HTTP) pour les requêtes entrantes.
- **default_backend web_backends** : Dirige les requêtes reçues vers le backend `web_backends`, défini plus loin.

---

#### **1.5. Section `backend web_backends`**

La section `backend web_backends` définit les serveurs vers lesquels les requêtes seront envoyées par le frontend.

````bash
backend web_backends
  balance          roundrobin
  server s1        172.20.128.2:80 check
  server s2        172.20.128.3:80 check
  server s3        172.20.128.4:80 check
`````

- **balance roundrobin** : La méthode de répartition de charge est définie comme **round-robin**, ce qui signifie que les requêtes seront envoyées aux serveurs backend de manière cyclique.
- **server s1 172.20.128.2:80 check**, **server s2 172.20.128.3:80 check**, **server s3 172.20.128.4:80 check** : Ces lignes définissent trois serveurs backend, chacun ayant une adresse IP statique. L'option **check** active la vérification de la santé de chaque serveur backend, permettant à HAProxy de vérifier si un serveur est en ligne avant d'envoyer du trafic vers lui.
### **2. Points Clés de la Configuration**

- **Répartition de charge** : HAProxy distribue le trafic HTTP entre trois serveurs backend, assurant ainsi la haute disponibilité et l'équilibrage de charge.
- **Interface de statistiques** : Permet de surveiller l'état de HAProxy en temps réel à l'aide d'une interface web accessible via le port `70`.
- **Sécurisation de l'interface de statistiques** : L'accès à l'interface de statistiques est protégé par authentification (avec des utilisateurs `admin` et `user`).

---

### **3. Bonnes Pratiques et Personnalisation**

- **Sécuriser l'interface de statistiques** : Assurez-vous de modifier les mots de passe `admin` et `user` pour des valeurs plus sécurisées dans un environnement de production.
- **Ajouter des serveurs** : Si vous souhaitez ajouter des serveurs backend supplémentaires, vous pouvez simplement ajouter des lignes `server s4`, `server s5`, etc., avec les adresses IP et les ports correspondants.
- **Optimisation des délais** : Les délais de connexion et de réponse (`timeout connect`, `timeout client`, `timeout server`) peuvent être ajustés selon les besoins de votre environnement.

---

### **4. Conclusion**

Ce fichier de configuration HAProxy est essentiel pour gérer un environnement avec plusieurs serveurs backend et répartir efficacement les requêtes entrantes. L'interface de statistiques permet de surveiller et d'administrer HAProxy en temps réel, tandis que la section de backend garantit la disponibilité et la répartition des charges entre plusieurs serveurs.
