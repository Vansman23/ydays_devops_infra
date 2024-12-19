
## **1. Préparation de l'environnement**

### **1.1. Mettre à jour le système**

```bash
sudo apt update && sudo apt upgrade -y
```

### **1.2. Installer les outils nécessaires**

Assurez-vous que des outils de base comme `curl`, `wget`, et `gnupg` sont installés :

````bash
sudo apt install -y curl wget gnupg software-properties-common
`````

## **2. Installation de Grafana**

### **2.1. Ajouter le dépôt officiel de Grafana**

1. Importer la clé GPG du dépôt Grafana :

````bash
sudo wget -q -O /usr/share/keyrings/grafana-archive-keyring.gpg https://packages.grafana.com/gpg.key
````

Ajouter le dépôt Grafana à la liste des sources :

````bash
echo "deb [signed-by=/usr/share/keyrings/grafana-archive-keyring.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
`````

Mettre à jour la liste des paquets :

````bash
sudo apt updates
`````

2.2. Installer Grafana

````bash
sudo apt install -y grafana
`````

## **3. Configuration de Grafana**

### **3.1. Activer et démarrer le service Grafana**

1. Activer le service Grafana pour qu'il démarre automatiquement au démarrage de la machine :

````bash
sudo systemctl enable grafana-server
````

2.3. Démarrer le service Grafana :

````bash
sudo systemctl start grafana-server
`````

Vérifier que Grafana fonctionne correctement :

````bash
sudo systemctl status grafana-server
`````

### **3.2. Configurer le pare-feu (facultatif)**

Si un pare-feu est activé, autorisez le port 3000 utilisé par Grafana :

````ini
sudo ufw allow 3000
sudo ufw reload
`````

## **4. Accès à l'interface web**

1. Ouvrez votre navigateur et accédez à l'interface Grafana en entrant l'URL suivante :

````
http://<adresse-ip-de-la-machine>:3000
`````

Si la machine est locale, utilisez :

````ini
http://localhost:3000
`````

- Identifiants par défaut :
    
    - **Utilisateur** : `admin`
    - **Mot de passe** : `admin`
- Lors de la première connexion, il vous sera demandé de changer le mot de passe administrateur

## **5. Configuration initiale dans l'interface web**

### **5.1. Ajouter une source de données**

1. Accédez à **Configuration > Data Sources**.
2. Sélectionnez la source de données souhaitée (par exemple : **Prometheus**, **InfluxDB**, ou autres).
3. Renseignez les informations nécessaires (URL, authentification, etc.) et testez la connexion.

### **5.2. Créer un tableau de bord**

1. Accédez à **Dashboards > New Dashboard**.
2. Ajoutez des panels et configurez-les pour afficher les données provenant de la source configurée.

---

## **6. Sécurisation de Grafana**

### **6.1. Modifier les paramètres de configuration**

Le fichier de configuration principal se trouve ici : `/etc/grafana/grafana.ini`.

Exemple de modifications importantes :

- **Changer le port par défaut (facultatif)** :

````ini
[server]
http_port = 8080
`````

**Activer le HTTPS (avec certificat)** :

````ini
[server]
protocol = https
cert_file = /path/to/your/certificate.crt
cert_key = /path/to/your/private.key
`````

### **6.2. Restreindre l'accès administrateur**

- Désactiver les inscriptions publiques si vous ne les utilisez pas :

````ini
[users]
allow_sign_up = false
`````

- Restreindre l'accès via le pare-feu ou un proxy inverse comme NGINX.
    

---

## **7. Vérification et maintenance**

### **7.1. Vérifier les journaux**

En cas de problème, consultez les journaux Grafana :

````bash
sudo journalctl -u grafana-server -f
`````

### **7.2. Mettre à jour Grafana**

Pour maintenir votre installation à jour :

````bash
sudo apt update && sudo apt upgrade -y grafana
`````

### **7.3. Sauvegarde**

- Sauvegardez régulièrement vos tableaux de bord et configurations via l'interface web ou en copiant le répertoire `/var/lib/grafana`.
-
Cette procédure fournit un environnement fonctionnel, sécurisé et prêt pour l'utilisation de Grafana sur une machine virtuelle sous Ubuntu 22.04.