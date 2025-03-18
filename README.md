# Documentation du Serveur NAS sous Debian

## Introduction
Ce document décrit la mise en place d'un serveur NAS sous Debian. Il couvre les aspects suivants :
- Configuration du RAID 5
- Synchronisation avec Rsync
- Configuration de SFTP
- Partage de fichiers avec Samba
- Mise en place de WebDAV
- Sécurisation avec HTTPS

---

## 1. Configuration du RAID 5

### 1.1 Création du RAID 5
```bash
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
```

### 1.2 Vérification de la synchronisation
```bash
cat /proc/mdstat
```

### 1.3 Formatage et montage du RAID
```bash
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/raid
sudo mount /dev/md0 /mnt/raid
```

### 1.4 Configuration automatique du montage
```bash
sudo blkid /dev/md0
```
Ajouter dans `/etc/fstab` :
```bash
UUID=<UUID_Obtenu> /mnt/raid ext4 defaults 0 2
```

---

## 2. Synchronisation avec Rsync

### 2.1 Installation de Rsync
```bash
sudo apt install rsync
```

### 2.2 Automatisation des sauvegardes
Configurer une tâche CRON :
```bash
crontab -e
```
Ajouter la ligne suivante pour une sauvegarde quotidienne à 2h du matin :
```bash
0 2 * * * rsync -av /mnt/raid/ user@backup-server:/mnt/backup/
```

---

## 3. Configuration de SFTP

### 3.1 Installation d'OpenSSH
```bash
sudo apt install openssh-server
```

### 3.2 Configuration SFTP dans `/etc/ssh/sshd_config`
Ajouter à la fin du fichier :
```plaintext
Match User jericho
    ChrootDirectory /mnt/raid/jericho
    ForceCommand internal-sftp
    AllowTcpForwarding no
```

### 3.3 Redémarrer le service SSH
```bash
sudo systemctl restart ssh
```

---

## 4. Partage de fichiers avec Samba

### 4.1 Installation de Samba
```bash
sudo apt install samba
```

### 4.2 Configuration des partages
Éditer `/etc/samba/smb.conf` :
```plaintext
[Public]
    path = /mnt/raid/Public
    guest ok = yes
    browseable = yes
    read only = no
```

### 4.3 Redémarrer Samba
```bash
sudo systemctl restart smbd
```

---

## 5. Mise en place de WebDAV

### 5.1 Installation d'Apache et WebDAV
```bash
sudo apt install apache2 apache2-utils
sudo a2enmod dav dav_fs
```

### 5.2 Configuration WebDAV
Créer `/etc/apache2/sites-available/webdav.conf` :
```plaintext
<VirtualHost *:80>
    ServerName nas.ecole.com
    DocumentRoot /mnt/raid/WebDAV
    <Directory /mnt/raid/WebDAV>
        DAV On
        AuthType Basic
        AuthName "WebDAV"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
```

### 5.3 Création des utilisateurs WebDAV
```bash
sudo htpasswd -c /etc/apache2/.htpasswd jericho
```

### 5.4 Activation et redémarrage Apache
```bash
sudo a2ensite webdav.conf
sudo systemctl restart apache2
```

---

## 6. Sécurisation avec HTTPS

### 6.1 Installation de Certbot
```bash
sudo apt install certbot python3-certbot-apache
```

### 6.2 Obtention du certificat SSL
```bash
sudo certbot --apache
```

### 6.3 Configuration du renouvellement automatique
```bash
sudo certbot renew --dry-run
```

---

## 7. Tests et Validation
- Tester l'accès SFTP :
```bash
sftp jericho@nas.ecole.com
```
- Tester Samba via `smb://nas.ecole.com/Public`
- Tester WebDAV via `https://nas.ecole.com`

---

## Conclusion
Cette documentation couvre l’installation et la configuration d’un serveur NAS sécurisé sous Debian. Vous pouvez maintenant l’utiliser pour stocker et partager vos fichiers en toute sécurité.
