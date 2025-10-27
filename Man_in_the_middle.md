# FAILLE MAN IN THE MIDDLE (MITM)

## INTRODUCTION

Une attaque Man-in-the-Middle (MITM) ou "Homme du milieu" se produit lorsqu'un attaquant 
s'intercale secrètement entre deux parties qui communiquent, pouvant ainsi intercepter, 
lire et modifier les données échangées sans que les victimes ne s'en aperçoivent.

L'objectif est de se positionner entre le client et le serveur pour espionner ou manipuler 
les communications, tout en faisant croire aux deux parties qu'elles communiquent directement 
entre elles.

---

## FONCTIONNEMENT

1. L'attaquant se positionne entre le client et le serveur

2. Il intercepte les communications dans les deux sens

3. Il peut lire, enregistrer ou modifier les données

4. Les deux parties croient communiquer directement entre elles

---

## TYPES D'ATTAQUES MITM

### 1. ARP SPOOFING
• Falsification de tables ARP sur un réseau local  
• Redirection du trafic vers la machine de l'attaquant

### 2. DNS SPOOFING
• Corruption du cache DNS  
• Redirection vers de faux serveurs

### 3. SSL STRIPPING
• Interception HTTPS et downgrade vers HTTP  
• L'attaquant maintient HTTPS avec le serveur mais HTTP avec le client

### 4. WIFI EVIL TWIN
• Création d'un faux point d'accès WiFi  
• Les utilisateurs se connectent au réseau piégé

### 5. SESSION HIJACKING
• Vol de cookies de session  
• Usurpation d'identité

### 6. EMAIL HIJACKING
• Interception d'emails  
• Modification de virements bancaires

---

## SCÉNARIOS D'ATTAQUE

### Scénario 1 : Café Public
• Victime se connecte au WiFi "Starbucks_Free"
• En réalité : faux point d'accès contrôlé par l'attaquant
• L'attaquant intercepte toutes les communications

### Scénario 2 : Réseau d'Entreprise
• Attaquant compromet un switch réseau
• Effectue un ARP spoofing
• Intercepte les communications entre employés et serveurs

### Scénario 3 : Phishing par DNS
• Compromission du serveur DNS
• Redirection de bank.com vers un faux site
• Vol des identifiants bancaires

---

## OUTILS UTILISÉS PAR LES ATTAQUANTS

• **Wireshark :** Capture de paquets réseau  
• **Ettercap :** ARP spoofing et MITM  
• **SSLstrip :** Suppression de la couche SSL/TLS  
• **Cain & Abel :** Sniffing et cracking  
• **BetterCAP :** Framework MITM moderne  
• **mitmproxy :** Proxy MITM pour HTTP/HTTPS  
• **Burp Suite :** Interception et modification de requêtes

---

## DONNÉES EXPOSÉES

• Identifiants de connexion (login/password)  
• Cookies de session  
• Données de cartes bancaires  
• Emails et messages  
• Données personnelles sensibles  
• Clés API et tokens  
• Informations confidentielles d'entreprise

---

## CONSÉQUENCES POSSIBLES

• Vol d'identifiants et de sessions  
• Espionnage de communications privées  
• Vol de données bancaires  
• Usurpation d'identité  
• Modification de transactions financières  
• Installation de malwares  
• Accès à des ressources protégées  
• Compromission totale de comptes

---

## SIGNES D'UNE ATTAQUE MITM

⚠️ Certificat SSL invalide ou non reconnu  
⚠️ Connexion HTTPS qui passe en HTTP sans raison  
⚠️ Performances réseau inhabituellement lentes  
⚠️ Adresses URL légèrement différentes  
⚠️ Comportement inhabituel du navigateur  
⚠️ Messages d'avertissement de sécurité

---

## PROTECTION - CÔTÉ APPLICATION

Plusieurs techniques permettent d'éviter les attaques MITM :

• **FORCER HTTPS** sur toute l'application

• Implémenter HSTS (HTTP Strict Transport Security)

• Utiliser des certificats SSL/TLS valides

• Activer Certificate Pinning

• Implémenter la validation de certificats

• Utiliser des cookies sécurisés (Secure, HttpOnly, SameSite)

• Implémenter le chiffrement de bout en bout

• Surveiller les anomalies de trafic

---

## CAS SYMFONY

Symfony offre plusieurs mécanismes pour se protéger contre les attaques MITM :

• Configuration HTTPS forcée  
• Gestion sécurisée des cookies de session  
• Headers de sécurité via NelmioSecurityBundle  
• Support HSTS intégré

---

## EXEMPLES SYMFONY - CONFIGURATION SÉCURISÉE

```yaml
# config/packages/security.yaml
security:
    firewalls:
        main:
            # Forcer HTTPS
            access_control:
                - { path: ^/, roles: PUBLIC_ACCESS, requires_channel: https }

# config/packages/framework.yaml
framework:
    session:
        cookie_secure: 'auto'  # Force HTTPS en production
        cookie_httponly: true
        cookie_samesite: 'strict'
```

```apache
# .htaccess ou configuration Nginx/Apache
# Redirection HTTP vers HTTPS
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</IfModule>

# HSTS Header
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

---

## HTTP STRICT TRANSPORT SECURITY (HSTS)

```php
// Symfony - NelmioSecurityBundle
composer require nelmio/security-bundle
```

```yaml
# config/packages/nelmio_security.yaml
nelmio_security:
    forced_ssl:
        enabled: true
        hsts_max_age: 31536000  # 1 an
        hsts_subdomains: true
        hsts_preload: true
```

---

## HEADERS DE SÉCURITÉ

```apache
# .htaccess ou configuration serveur
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
Header set X-XSS-Protection "1; mode=block"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
```

---

## DOCKER / SYMFONY - CONFIGURATION HTTPS

```yaml
# docker-compose.yml
services:
  nginx:
    ports:
      - "443:443"
    volumes:
      - ./certs:/etc/nginx/certs
    environment:
      - SSL_CERTIFICATE=/etc/nginx/certs/cert.pem
      - SSL_CERTIFICATE_KEY=/etc/nginx/certs/key.pem
```

```nginx
# nginx.conf
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/certs/cert.pem;
    ssl_certificate_key /etc/nginx/certs/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

---

## PRÉVENTION - CÔTÉ UTILISATEUR

• Vérifier le cadenas HTTPS dans le navigateur  
• Vérifier la validité du certificat SSL  
• Ne jamais ignorer les avertissements de sécurité  
• Éviter les réseaux WiFi publics non sécurisés  
• Utiliser un VPN sur réseaux non fiables  
• Maintenir le système et les applications à jour  
• Utiliser des mots de passe uniques et forts  
• Activer l'authentification à deux facteurs (2FA)

---

## VPN ET CHIFFREMENT

• Utiliser un VPN réputé sur réseaux publics  
• Privilégier WPA3 pour les réseaux WiFi  
• Utiliser des protocoles chiffrés (TLS 1.3)  
• Éviter les protocoles obsolètes (SSL, TLS 1.0, TLS 1.1)

---

## MONITORING ET DÉTECTION

• Surveiller les certificats avec Certificate Transparency logs  
• Utiliser des outils de détection d'intrusion (IDS)  
• Analyser les logs pour détecter des anomalies  
• Implémenter des alertes sur les tentatives suspectes  
• Vérifier régulièrement les configurations SSL/TLS

---

## OUTILS DE TEST

• SSL Labs (ssllabs.com/ssltest)  
• Security Headers (securityheaders.com)  
• testssl.sh  
• Observatory by Mozilla  
• Qualys SSL Server Test

---

## CONCLUSION

Les attaques Man-in-the-Middle représentent une menace sérieuse pour la confidentialité et 
l'intégrité des communications.

Dans notre cas, Symfony offre des outils robustes (HTTPS forcé, HSTS, cookies sécurisés) qui, 
combinés à une infrastructure correctement configurée (certificats SSL/TLS valides, protocoles 
modernes), permettent de se protéger efficacement contre ces attaques.

**La règle d'or : toujours utiliser HTTPS et ne jamais ignorer les avertissements de sécurité.**

---

**NIVEAU DE RISQUE : CRITIQUE**  
**FACILITÉ D'EXPLOITATION : MOYENNE (selon le contexte)**  
**NÉCESSITE :** Proximité réseau ou compromission infrastructure
