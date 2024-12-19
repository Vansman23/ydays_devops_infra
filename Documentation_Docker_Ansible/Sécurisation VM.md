
## **1. Préparation initiale**

### **1.1. Mettre à jour le système**

Assurez-vous que le système est à jour :

````bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove --purge -y
`````

### **1.2. Installer les outils nécessaires**

Installez des outils utiles pour la gestion et la surveillance de la sécurité :

````bash
sudo apt install -y ufw fail2ban unattended-upgrades auditd clamav lynis
`````

## **2. Configuration de base du système**

### **2.1. Modifier les permissions utilisateur**

1. Créez un utilisateur non root pour éviter d'utiliser directement le compte root :
````bash
sudo adduser <nom_utilisateur>
sudo usermod -aG sudo <nom_utilisateur>
`````

Désactivez l'accès direct à root via SSH : Éditez le fichier `/etc/ssh/sshd_config` :

````bash
sudo nano /etc/ssh/sshd_config
`````

Recherchez et modifiez les lignes suivantes :

````bash
PermitRootLogin no
PasswordAuthentication no
`````

Redémarrez le service SSH :

````bash
sudo systemctl restart ssh
`````

### **2.2. Configurer un mot de passe fort**

Utilisez des mots de passe robustes et uniques pour tous les comptes locaux :

````bash
sudo passwd <nom_utilisateur>
`````

## **4. Configuration des mises à jour automatiques**

Activez les mises à jour automatiques pour garantir l'application rapide des correctifs de sécurité:

````bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
`````

## **5. Configuration des mises à jour automatiques**

### **5.2. Installer et configurer Fail2Ban**

1. Configurez Fail2Ban pour limiter les tentatives d'intrusion SSH :

````bash
sudo nano /etc/fail2ban/jail.local
`````

Exemple de configuration pour SSH :

````bash
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
`````

Redémarrez Fail2Ban :

````bash
sudo systemctl restart fail2ban
`````

## **6. Analyse et renforcement du système**

### **6.1. Configurer et exécuter Lynis**

Lynis est un outil d'audit de sécurité :

Lancez un audit de sécurité :

````bash
sudo lynis audit system
`````

### **6.2. Activer le journal d'audit avec Auditd**

1. Configurez Auditd pour surveiller les activités critiques :
````bash
sudo nano /etc/audit/auditd.conf
`````

Exemple de configuration :

`````bash
log_format = ENRICHED
max_log_file = 50
num_logs = 5
`````

Démarrez et activez le service :

`````bash
sudo systemctl enable auditd
sudo systemctl start auditd
`````

## **7. Protection contre les logiciels malveillants**

### **7.1. Installer ClamAV pour détecter les malwares**

1. Mettez à jour les bases de signatures :

````bash
sudo freshclam
`````

Lancez un scan manuel :

`````bash
sudo clamscan -r /home
`````

Activez le service ClamAV si besoin :

````bash
sudo systemctl enable clamav-freshclam
`````

## **8. Sauvegarde et restauration**

### **8.1. Configurer des sauvegardes régulières**

1. Installez **rsync** ou utilisez des outils comme **BorgBackup** pour sauvegarder vos données. Exemple avec rsync :

````bash
rsync -av --delete /important/data /backup/location
`````

#### **Automatisation avec cron**

Automatisez les sauvegardes avec `cron` pour une exécution régulière.

1. Ouvrez l'éditeur `cron` pour l'utilisateur :

````bash
crontab -e
`````

Ajoutez une tâche pour effectuer une sauvegarde quotidienne à 2h du matin :

````bash
0 2 * * * rsync -av --delete /chemin/dossier/source /chemin/dossier/destination
`````

### 9.1. Sauvegarde avec BorgBackup

BorgBackup est une solution efficace de sauvegarde dédupliquée.

#### **Installation de BorgBackup**

````bash
sudo apt install borgbackup -y
`````

#### **Initialisation d'un dépôt de sauvegarde**

1. Créez un dépôt sur un disque externe ou distant :

````bash
borg init --encryption=repokey /chemin/vers/destination
`````

Lancez une sauvegarde :

````bash
borg create --stats /chemin/vers/destination::backup-{now:%Y-%m-%d} /chemin/dossier/source
`````

#### **Automatisation avec Borg**

Ajoutez une tâche `cron` similaire à celle décrite plus haut :

````bash
0 2 * * * borg create --stats /chemin/vers/destination::backup-{now:%Y-%m-%d} /chemin/dossier/source
`````

### 10.1. Sauvegarde avec Timeshift (pour les snapshots système)

Timeshift est idéal pour créer des snapshots de l'ensemble du système (similaire à la restauration système sur Windows).

#### **Installation de Timeshift**

````bash
sudo apt install timeshift -y
`````

#### **Configuration de Timeshift**

1. Lancez l'interface de configuration :

````bash
sudo timeshift-gtk
`````

Choisissez :

- Le type de sauvegarde (RSYNC ou BTRFS).
- La fréquence (horaire, quotidienne, hebdomadaire, etc.).
- Les dossiers à inclure/exclure.


