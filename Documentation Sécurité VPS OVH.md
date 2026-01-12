# 🔒 Documentation Sécurité VPS

> Configuration de sécurité du serveur VPS hébergeant le bot Discord Guardian

---

## 🖥️ Informations serveur

**Hébergeur** : OVH  
**Hostname** : `vps-492e9cd2`  
**IP publique** : `51.178.80.30`  
**OS** : Ubuntu 22.04 LTS  
**Domaine principal** : `kainovaii.dev`

---

## 🏗️ Architecture réseau

```
Internet
  ↓
Cloudflare Proxy (DDoS + Cache)
  ↓
VPS 51.178.80.30
  ├─ Port 22  : SSH (clé uniquement)
  ├─ Port 80  : HTTP → Caddy
  └─ Port 443 : HTTPS → Caddy
       ↓
Caddy Reverse Proxy
  ├─ kainovaii.dev          → localhost:8888
  ├─ unitpanel.kainovaii.dev → localhost:9292
  └─ guardian.kainovaii.dev  → localhost:9393
```

**Principe** : Seuls les ports 22, 80, 443 sont exposés. Les services internes (8888, 9292, 9393) ne sont accessibles que via localhost.

---

## 🔐 Sécurité SSH

- ✅ Authentification par clé SSH uniquement (pas de mot de passe)
- ✅ Root login désactivé
- ✅ Un seul utilisateur : `ubuntu`
- ✅ Port : 22

**Connexion** :
```bash
ssh ubuntu@51.178.80.30
```

---

## 🛡️ Firewall UFW

**Configuration active** :
```
22/tcp   ALLOW   (SSH)
443/tcp  ALLOW   (HTTPS)
```

**Politique** : Tout le reste est bloqué par défaut.

---

## 🔄 Reverse Proxy Caddy

**Caddyfile** : `/etc/caddy/Caddyfile`

```caddy
kainovaii.dev {
    reverse_proxy localhost:8888
    encode gzip
}

www.kainovaii.dev {
    reverse_proxy localhost:8888
    encode gzip
}

unitpanel.kainovaii.dev {
    reverse_proxy localhost:9292
    encode gzip
}

guardian.kainovaii.dev {
    reverse_proxy localhost:9393
    encode gzip
}
```

**SSL** : Automatique via Let's Encrypt

---

## ☁️ Cloudflare

**Configuration DNS** :

| Sous-domaine | IP           | Proxy    |
|--------------|--------------|----------|
| @            | 51.178.80.30 | 🟠 Proxied |
| www          | 51.178.80.30 | 🟠 Proxied |
| unitpanel    | 51.178.80.30 | 🟠 Proxied |
| guardian     | 51.178.80.30 | 🟠 Proxied |

**Mode SSL/TLS** : Full (strict)

---

## 🤖 Services déployés

### 1. Site principal
- **URL** : https://kainovaii.dev
- **Port** : 8888

### 2. UnitPanel
- **URL** : https://unitpanel.kainovaii.dev
- **Port** : 9292

### 3. Guardian Bot
- **URL** : https://guardian.kainovaii.dev
- **Port** : 9393

---

## 🎯 Checklist de sécurité

- [x] SSH par clé uniquement
- [x] Root désactivé
- [x] Firewall UFW actif
- [x] Ports 80, 443 uniquement exposés
- [x] Services internes sur localhost
- [x] Reverse proxy Caddy
- [x] SSL automatique
- [x] Protection Cloudflare

---

**Dernière mise à jour** : 12 janvier 2026  
**Responsable** : KainoVaii
