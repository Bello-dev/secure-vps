# 🔒 Secure VPS - Script de Sécurisation Automatique

Script Bash pour sécuriser automatiquement un serveur VPS (Virtual Private Server) avec support multi-utilisateurs.

## 📋 Fonctionnalités

- ✅ **Création de plusieurs utilisateurs SSH** (jusqu'à 50+ utilisateurs possibles)
- ✅ **Génération automatique de clés SSH ED25519** pour chaque utilisateur
- ✅ **Configuration SSH sécurisée** (désactivation du root, authentification par clé uniquement)
- ✅ **Changement du port SSH** vers un port personnalisé
- ✅ **Configuration du firewall** (UFW ou Firewalld selon la distribution)
- ✅ **Restriction d'accès SSH par IP/CIDR**
- ✅ **Installation et configuration de Fail2Ban**
- ✅ **Désactivation optionnelle d'IPv6**
- ✅ **Limitation optionnelle des réponses ICMP (ping)**
- ✅ **Sauvegarde automatique des configurations**
- ✅ **Fonction de rollback** en cas d'erreur

## 🖥️ Distributions Supportées

- **Ubuntu** / **Debian** (avec UFW)
- **CentOS** / **RHEL** / **Rocky Linux** / **AlmaLinux** (avec Firewalld)
- **Fedora** (avec Firewalld)
- **Arch Linux** (avec UFW)

## ⚡ Installation Rapide

### Prérequis

- Accès root ou sudo au serveur
- Distribution Linux supportée
- Connexion internet active

### Exécution

```bash
# Télécharger le script
wget https://raw.githubusercontent.com/Bello-dev/secure-vps/main/secure-vps.sh

# Rendre le script exécutable
chmod +x secure-vps.sh

# Exécuter le script en tant que root
sudo ./secure-vps.sh
```

## 📝 Guide d'Utilisation

### 1. Nombre d'utilisateurs

Le script vous demandera combien d'utilisateurs vous souhaitez créer :

```
Combien d'utilisateurs souhaitez-vous créer ? [1]:
```

- Vous pouvez créer autant d'utilisateurs que nécessaire (testé avec plus de 50 utilisateurs)
- Chaque utilisateur aura son propre compte et ses propres clés SSH

### 2. Noms des utilisateurs

Pour chaque utilisateur, le script proposera un nom par défaut :

```
Nom de l'utilisateur #1 ? [secureuser1]:
Nom de l'utilisateur #2 ? [secureuser2]:
...
```

Les noms d'utilisateurs doivent respecter les règles Linux :
- Commencer par une lettre minuscule
- Contenir uniquement des lettres minuscules, chiffres, tirets et underscores
- Maximum 64 caractères

### 3. Configuration SSH

```
Quel port souhaitez-vous pour SSH ? [port_aléatoire]:
```

- Le script propose un port aléatoire entre 10000 et 65535
- Vous pouvez choisir votre propre port (entre 1024 et 65535)
- Le port 22 sera automatiquement désactivé

### 4. Restriction d'accès par IP (optionnel)

```
Entrez les IPs ou CIDR autorisés pour SSH (séparées par des espaces) []:
```

Exemples :
- IP unique : `203.0.113.1`
- Plusieurs IPs : `203.0.113.1 198.51.100.50`
- Réseau CIDR : `203.0.113.0/24`
- Mixte : `203.0.113.1 198.51.100.0/24`

### 5. Options de sécurité

```
Voulez-vous désactiver IPv6 ? [oui]:
Voulez-vous limiter les réponses ICMP (Ping) ? [oui]:
```

## 🔑 Récupération des Clés SSH

Après l'exécution, le script affichera toutes les clés privées générées :

```
📂 Toutes les clés privées sont sauvegardées dans : /root/ssh_keys_YYYY-MM-DD_HHMMSS

==========================================
🔑 Clé privée SSH pour l'utilisateur : secureuser1
==========================================
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
==========================================
```

### Sauvegarde des clés

**IMPORTANT** : Copiez et sauvegardez chaque clé privée sur votre machine locale avant de fermer la session !

Sur votre machine locale :

```bash
# Créer un fichier pour chaque clé
nano ~/.ssh/secureuser1_id_ed25519

# Coller la clé privée, puis sauvegarder

# Définir les permissions correctes
chmod 600 ~/.ssh/secureuser1_id_ed25519
```

## 🔐 Connexion SSH

Pour vous connecter après la configuration :

```bash
ssh -i ~/.ssh/nom_utilisateur_id_ed25519 -p PORT_SSH nom_utilisateur@VOTRE_IP_SERVEUR
```

Exemple :

```bash
ssh -i ~/.ssh/secureuser1_id_ed25519 -p 42022 secureuser1@203.0.113.1
```

### Configuration SSH Client (Recommandé)

Pour simplifier vos connexions, ajoutez une configuration dans `~/.ssh/config` :

```
Host mon-vps-user1
    HostName 203.0.113.1
    Port 42022
    User secureuser1
    IdentityFile ~/.ssh/secureuser1_id_ed25519
    
Host mon-vps-user2
    HostName 203.0.113.1
    Port 42022
    User secureuser2
    IdentityFile ~/.ssh/secureuser2_id_ed25519
```

Ensuite, connectez-vous simplement avec :

```bash
ssh mon-vps-user1
```

## 🛡️ Sécurité

### Ce que le script configure

1. **SSH sécurisé** :
   - Authentification par clé uniquement (mot de passe désactivé)
   - Root login désactivé
   - Port personnalisé
   - PAM désactivé

2. **Firewall** :
   - Politique par défaut : tout bloqué en entrée
   - Autorisation HTTP (80) et HTTPS (443)
   - SSH restreint au port configuré (avec option de restriction par IP)
   - Port 22 explicitement bloqué

3. **Fail2Ban** :
   - Protection contre les attaques brute-force
   - Ban de 1 heure après 5 tentatives échouées
   - Surveillance du nouveau port SSH

4. **Options supplémentaires** :
   - IPv6 désactivable
   - Limitation des réponses ICMP

### Rollback automatique

En cas d'erreur, le script restaure automatiquement les configurations d'origine.

## ⚠️ Avertissements Importants

1. **Ne fermez pas votre session SSH actuelle** avant d'avoir testé la nouvelle connexion
2. **Sauvegardez toutes les clés privées** affichées par le script
3. **Testez la connexion SSH** depuis une autre session/terminal avant de fermer la session actuelle
4. **Notez le nouveau port SSH** - vous en aurez besoin pour chaque connexion

## 🧪 Tests

### Test du script

Pour tester le script sans risque, utilisez une machine virtuelle ou un VPS de test :

```bash
# Sur votre machine de test
sudo ./secure-vps.sh

# Suivez les instructions
# Testez ensuite la connexion depuis une autre machine
```

### Vérifications recommandées

Après l'exécution :

1. **Test de connexion SSH** avec la nouvelle configuration
2. **Vérification du firewall** : `sudo ufw status` ou `sudo firewall-cmd --list-all`
3. **Vérification de Fail2Ban** : `sudo fail2ban-client status sshd`
4. **Test des restrictions IP** (si configurées)

## 🔄 Gestion Multi-Utilisateurs

### Avantages

- **Séparation des accès** : chaque utilisateur a son propre compte
- **Audit** : traçabilité des actions par utilisateur
- **Sécurité** : révocation individuelle possible
- **Flexibilité** : permissions personnalisables par utilisateur

### Cas d'usage

- **Équipe DevOps** : un compte par membre de l'équipe
- **Environnement de développement** : comptes séparés pour dev/staging/prod
- **Clients multiples** : accès isolés pour différents clients
- **Services automatisés** : comptes dédiés pour CI/CD, monitoring, etc.

### Bonnes pratiques

1. **Nommage cohérent** : Utilisez des conventions claires (ex: `prenom.nom`, `service-role`)

2. **Documentation** : Le fichier `README_CONNEXION.txt` généré contient toutes les informations nécessaires

3. **Distribution sécurisée** : 
   - Envoyez les clés privées via un canal sécurisé (pas par email)
   - Utilisez un gestionnaire de mots de passe d'équipe
   - Considérez un système de gestion de secrets (Vault, etc.)

4. **Rotation des accès** :
   - Supprimez les comptes d'utilisateurs qui n'ont plus besoin d'accès
   - Régénérez les clés périodiquement pour les comptes critiques

5. **Monitoring** :
   - Vérifiez régulièrement les logs d'authentification
   - Utilisez `last` et `lastlog` pour surveiller les connexions

6. **Permissions** :
   - Ne donnez pas sudo à tous les utilisateurs par défaut
   - Créez des groupes avec permissions spécifiques selon les besoins

## 📊 Structure des Fichiers

Après l'exécution, le script crée :

```
/root/ssh_keys_YYYY-MM-DD_HHMMSS/
├── secureuser1_id_ed25519
├── secureuser1_id_ed25519.pub
├── secureuser2_id_ed25519
├── secureuser2_id_ed25519.pub
└── ...

/home/secureuser1/.ssh/
├── id_ed25519
├── id_ed25519.pub
└── authorized_keys

/etc/ssh/
├── sshd_config
└── sshd_config.bak_YYYY-MM-DD_HHMMSS

/etc/fail2ban/
└── jail.local
```

## 🐛 Dépannage

### Problème : Impossible de se connecter après la configuration

1. **Vérifiez le service SSH** :
   ```bash
   sudo systemctl status sshd  # ou ssh
   ```

2. **Vérifiez les logs** :
   ```bash
   sudo journalctl -xe | grep ssh
   sudo tail -f /var/log/auth.log
   ```

3. **Vérifiez le firewall** :
   ```bash
   sudo ufw status verbose  # Ubuntu/Debian
   sudo firewall-cmd --list-all  # CentOS/RHEL
   ```

### Problème : Port SSH bloqué

Assurez-vous que votre fournisseur VPS n'a pas de restrictions sur les ports personnalisés.

### Problème : Clé privée refusée

Vérifiez les permissions :
```bash
chmod 600 ~/.ssh/nom_utilisateur_id_ed25519
```

### Problème : Gestion de nombreux utilisateurs

Si vous créez beaucoup d'utilisateurs (50+) :

1. **Organisation des clés** : Utilisez un gestionnaire de clés comme `ssh-agent`
   ```bash
   ssh-add ~/.ssh/user1_id_ed25519
   ssh-add ~/.ssh/user2_id_ed25519
   ```

2. **Fichier de configuration SSH** : Organisez vos connexions dans `~/.ssh/config`

3. **Rotation des clés** : Pour désactiver un utilisateur :
   ```bash
   sudo userdel -r nom_utilisateur  # Supprime l'utilisateur et son répertoire
   ```

4. **Audit des connexions** : Surveillez les connexions actives
   ```bash
   who           # Voir les utilisateurs connectés
   last          # Historique des connexions
   ```

## 📄 Licence

Copyright MozzyPC (https://www.youtube.com/@mozzypc)

## 🤝 Contributions

Les contributions sont les bienvenues ! N'hésitez pas à ouvrir une issue ou une pull request.

## ⭐ Support

Si ce script vous a aidé, n'hésitez pas à mettre une étoile ⭐ sur le repository !

## 📚 Ressources Supplémentaires

- [OpenSSH Documentation](https://www.openssh.com/)
- [UFW Guide](https://help.ubuntu.com/community/UFW)
- [Fail2Ban Documentation](https://www.fail2ban.org/)
- [Best Practices SSH](https://infosec.mozilla.org/guidelines/openssh)

---

**Dernière mise à jour** : 02/2025
