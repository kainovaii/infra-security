# 🚨 Procédure de récupération serveur (SSH bloqué)

> Guide de récupération lorsque vous êtes bloqué hors de votre serveur VPS à cause d'une erreur UFW/SSH

---

## 🔴 Symptômes

Vous ne pouvez plus vous connecter en SSH :

```bash
ssh ubuntu@51.178.80.30
# ssh: connect to host 51.178.80.30 port 22: Connection refused
```

**Causes fréquentes** :
- Port 22 non autorisé dans UFW
- UFW activé sans avoir autorisé SSH
- Port SSH changé sans mise à jour du firewall

---

## ✅ Solution : Mode Rescue OVH

### Étape 1 : Activer le mode rescue

1. Connexion à l'**espace client OVH** : https://www.ovh.com/manager/
2. Sélectionner le VPS : `vps-492e9cd2`
3. Onglet **"Accueil"**
4. Cliquer sur **"Redémarrer"** ou **"Netboot"**
5. Sélectionner **"Rescue mode"** (rescue64-pro)
6. Confirmer le redémarrage
7. **Attendre 2-3 minutes**

### Étape 2 : Récupérer les identifiants

OVH envoie un email avec :
```
Login: root
Password: XxXxXxXxXxXx
```

Si vous n'avez pas reçu l'email, vous pouvez le redemander depuis l'espace client OVH.

### Étape 3 : Se connecter en mode rescue

Depuis votre PC :

```bash
ssh root@51.178.80.30
# Entrer le mot de passe reçu par email
```

**Note** : En mode rescue, vous utilisez le mot de passe, pas votre clé SSH.

### Étape 4 : Passer en AZERTY (optionnel)

Si votre clavier est en français :

```bash
loadkeys fr
```

### Étape 5 : Monter le disque système

```bash
# Lister les disques disponibles
lsblk

# Résultat typique :
# sda      8:0    0   40G  0 disk
# └─sda1   8:1    0   40G  0 part

# Monter le disque principal (généralement sda1 ou vda1)
mount /dev/sda1 /mnt

# Vérifier que c'est bien monté
ls /mnt
# Vous devriez voir : bin boot dev etc home lib ...
```

### Étape 6 : Réparer UFW

**Option A - Désactiver UFW** (simple) :

```bash
echo "ENABLED=no" > /mnt/etc/ufw/ufw.conf
```

**Option B - Ajouter la règle SSH** :

```bash
echo "-A ufw-user-input -p tcp --dport 22 -j ACCEPT" >> /mnt/etc/ufw/user.rules
```

### Étape 7 : Vérifier la modification

```bash
cat /mnt/etc/ufw/ufw.conf
# Doit afficher : ENABLED=no
```

### Étape 8 : Démonter et préparer le redémarrage

```bash
umount /mnt
```

### Étape 9 : Redémarrer en mode normal

1. Retourner sur l'**espace client OVH**
2. Sélectionner le VPS
3. Onglet **"Accueil"**
4. Cliquer sur **"Netboot"**
5. Sélectionner **"Démarrer depuis le disque dur"**
6. Confirmer le redémarrage
7. **Attendre 2-3 minutes**

### Étape 10 : Reconnexion SSH

```bash
ssh ubuntu@51.178.80.30
# Vous devriez pouvoir vous connecter !
```

### Étape 11 : Reconfigurer UFW correctement

```bash
# ⚠️ IMPORTANT : Autoriser SSH AVANT d'activer UFW
sudo ufw allow 22/tcp

# Autoriser HTTP/HTTPS pour Caddy
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Réactiver UFW
sudo ufw enable

# Vérifier la configuration
sudo ufw status
```

---

## 🎯 Configuration UFW sécurisée

Voici la configuration recommandée pour éviter de vous bloquer :

```bash
# 1. Politique par défaut
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 2. Autoriser SSH (TOUJOURS EN PREMIER !)
sudo ufw allow 22/tcp

# 3. Autoriser les autres services
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS

# 4. Activer UFW (UNIQUEMENT après avoir autorisé SSH)
sudo ufw enable

# 5. Vérifier
sudo ufw status verbose
```

---

## 📋 Checklist avant d'activer UFW

- [ ] Port SSH autorisé : `sudo ufw allow 22/tcp`
- [ ] Vérification : `sudo ufw status`
- [ ] Test : ouvrir une **deuxième session SSH** avant d'activer UFW
- [ ] Si la deuxième session fonctionne → activer UFW : `sudo ufw enable`

---

## ⚠️ Rappels importants

1. **TOUJOURS** autoriser le port SSH **AVANT** d'activer UFW
2. **TOUJOURS** garder une session SSH ouverte lors de modifications firewall
3. Si vous changez le port SSH (ex: 2222), pensez à l'autoriser dans UFW **avant** de recharger SSH
4. Gardez les identifiants de la console noVNC d'OVH en cas d'urgence

---

## 🔧 Autres cas de figure

### Si vous avez changé le port SSH

```bash
# Autoriser le nouveau port
sudo ufw allow 2222/tcp

# Puis supprimer l'ancien
sudo ufw delete allow 22/tcp
```

### Si vous voulez tester UFW sans risque

```bash
# Activer UFW avec un timer de désactivation automatique
sudo ufw enable && sleep 60 && sudo ufw disable
# Vous avez 60 secondes pour tester. Si ça ne marche pas, UFW se désactive automatiquement.
```

---

**Dernière mise à jour** : 12 janvier 2026  
**Serveur** : vps-492e9cd2 (51.178.80.30)
