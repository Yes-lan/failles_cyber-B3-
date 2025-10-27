# FAILLE CSRF (Cross-Site Request Forgery)

## INTRODUCTION

Le CSRF (Cross-Site Request Forgery) ou falsification de demande intersites en français est un 
type d'attaque qui force un navigateur authentifié à exécuter une requête non voulue vers un site 
où l'utilisateur est connecté.

L'objectif est de faire effectuer une action (changer mot de passe, transférer de l'argent, 
supprimer un enregistrement) au nom de la victime sans son consentement.

---

## FONCTIONNEMENT

1. Tout d'abord la victime se connecte à un site, le site que vise l'attaquant

2. Le site donne à l'utilisateur un cookie de session et/ou persistant

3. L'attaquant fait visiter à la victime une page qui semble officielle, provenant soi-disant du 
   site sain, mais avec une requête cachée

4. La victime clique sur un lien envoyant ainsi la requête vers le site original. La requête peut 
   également être envoyée automatiquement dans certains cas comme auto-submit, redirection, 
   malvertising

5. Le site identifie la requête comme venant de la personne identifiée et non de l'attaquant 
   et l'accepte

---

## VECTEURS D'ATTAQUE

L'attaquant possède toutes sortes de moyens pour envoyer cette page piégée :

• **Email / phishing :** message contenant un lien vers le site piégé

• **Réseaux sociaux / message :** partage d'un lien (DM, post)

• **Publicité malveillante (malvertising) :** une publicité qui redirige automatiquement vers 
  la page piégée

• **Sites compromis / widgets tiers :** un site légitime mais compromis qui inclut le code piégé 
  (ou un script tiers infecté)

• **Forum / commentaire :** poster un lien ou une image qui déclenche la requête

• **Pièce jointe ou document contenant un lien (PDF, Word)**

Dans tous les cas, il suffit que la victime soit redirigée d'une manière ou d'une autre sur 
la page piégée pour que la requête se lance, sans qu'elle n'ait besoin de faire quoi que ce 
soit d'autre.

Il est important de savoir que la victime n'a pas besoin d'avoir le site cible encore ouvert 
pour que l'attaque fonctionne, il suffit que le cookie d'identification ne soit pas encore 
expiré ou supprimé.

Cela peut aller tant que le navigateur reste ouvert avec les cookies de session ou bien de 
plusieurs heures à plusieurs jours avec les cookies persistants, qui sont maintenus même 
lorsque le navigateur est fermé.

---

## CONDITIONS REQUISES

Les attaques CSRF ont cependant plusieurs contraintes pour fonctionner :

1. Il faut évidemment que la victime soit authentifiée sur le site cible au moment de l'attaque

2. Il faut que le site cible n'utilise que les cookies pour identifier l'utilisateur

3. Le site doit accepter les requêtes GET ou POST capables de faire des actions avec des 
   conséquences sans aucune autre preuve que le cookie d'identification

Si une seule de ces conditions n'est pas validée, alors l'attaque échoue.

---

## PROTECTION

La technique de défense la plus utilisée est la **protection par token (synchronizer token)** :

1. Lorsqu'une action de l'utilisateur crée un formulaire, le site génère un token CSRF aléatoire 
   stocké côté serveur

2. Le serveur inclut ensuite le token dans le formulaire

3. À la soumission, le serveur vérifie que le token envoyé correspond au token en session

De cette manière, même si l'attaquant peut inclure un token dans son propre formulaire, il ne 
pourra pas être valide car un site extérieur ne peut pas connaître, et donc répliquer, la valeur 
des tokens d'autres sites → la requête est rejetée.

Il existe d'autres techniques de protection, souvent utilisées en tandem avec le synchronizer token :

• **SameSite Cookies (Lax/Strict) :** empêche l'envoi automatique du cookie dans certaines requêtes 
  cross-site. Ce qui limite beaucoup d'attaques de type form auto-submit.

• **Vérification des en-têtes Origin / Referer :** serveur vérifie si la requête vient bien d'un 
  domaine autorisé.

• **Double-submit cookie :** serveur met un cookie et le front envoie le même token dans le 
  body/header; le serveur vérifie ensuite l'égalité.

---

## CAS SYMFONY

Symfony est un peu spécial dans l'utilisation de ses tokens. Il n'utilise pas de token global, 
comme ceux décrits précédemment, mais des token strings.

Un token global sera relié à une « intention » d'un formulaire, cependant cette intention est 
assez générique et est souvent réutilisée partout. Par exemple l'intention appelée communément 
"global-form".

Tandis qu'un token string sera relié à une intention plus précise, différente pour chaque action, 
par exemple "delete-item" + id de l'item. Le token attendu est donc distinct selon l'action.

---

## EXEMPLE CONCRET (ILLUSTRATIF)

### 1. Token « global » :
- **Intention :** global-form
- **Token généré :** z9y8... (même pour tous les formulaires utilisant global-form)
- Tous les formulaires qui envoient _token=z9y8... passent la validation si l'intention 
  attendue est global-form.

### 2. Token string lié à une intention spécifique :
- **Intention :** delete-item-42
- **Token string généré :** a1b2...
- **Validation :** serveur compare a1b2... avec le token attendu pour delete-item-42.

---

## DIFFÉRENCES PRATIQUES ET IMPLICATIONS DE SÉCURITÉ

### 1. Portée
- **Par-intention (par-action) :** token n'est valable que pour l'action ciblée (ex. suppression 
  d'un item précis). Même si un token est volé, il ne servira que pour cette action précise.
- **Global :** token peut être réutilisé pour plusieurs actions → plus de surface d'attaque si vol.

### 2. Réutilisabilité
- Un token par-intention limite la réutilisation : il est peu utile pour autre chose.
- Un token global peut être réutilisé pour déclencher toute action qui accepte cette même 
  intention.

### 3. Stockage
- Symfony stocke typiquement la valeur du token en session indexée par l'intention. Ainsi il 
  peut y avoir plusieurs tokens différents simultanément (un par intention).

---

## CONCLUSION

De cette manière on a pu voir non seulement quelles failles peut cibler une attaque CSRF, ses 
limites mais également comment s'en protéger efficacement, que ce soit sur des sites classiques 
ou des sites possédant Symfony.

---

## ANNEXE

### DIFFÉRENCE SYNCHRONIZER ET DOUBLE-SUBMIT

**Synchronizer token (token CSRF classique) :**
• Le serveur génère un token aléatoire, le stocke côté serveur (généralement dans la session), 
  et l'insère dans le formulaire.
• À la soumission, le serveur vérifie que le token reçu correspond à celui attendu côté serveur.
• **Avantage :** même si un attaquant connaît ou prédit la valeur d'un autre token, il ne peut pas 
  deviner celui lié à une session spécifique.

**Double-submit cookie :**
• Le serveur ne stocke pas le token côté serveur.
• Il envoie un cookie CSRF au navigateur et le client renvoie le même token dans le body ou 
  dans un header.
• Le serveur vérifie que la valeur du cookie et la valeur envoyée correspondent.
• **Avantage :** pas besoin de stockage serveur, utile pour applications stateless (ex. APIs avec 
  sessionless auth).
• **Limite :** si un XSS est présent, l'attaquant peut lire le cookie et le soumettre → protection 
  CSRF compromise.

---

### EXPLICATION STRICT LAX

**Strict**
• Le cookie n'est envoyé que pour les requêtes provenant du même site (same-site).
• Toute requête cross-site ne recevra pas le cookie automatiquement, même si l'utilisateur clique 
  sur un lien vers le site cible depuis un autre site.

**Lax (valeur par défaut dans beaucoup de navigateurs)**
• Le cookie est envoyé automatiquement pour certaines requêtes cross-site sûres : principalement 
  les navigations top-level GET (cliquer sur un lien normal dans un mail ou moteur de recherche).
• Mais pour les requêtes « unsafe » comme POST, PUT, DELETE cross-site (ex. formulaire auto-submit, 
  AJAX cross-site), le cookie n'est pas envoyé automatiquement → CSRF bloqué.

---

### DIFFÉRENCE ORIGIN REFERER

**Origin**
• Contient le schéma (http/https), le domaine et le port de la page qui a initié la requête.
• `Origin: https://mon-site.example`
• Utilisé principalement pour les requêtes POST, PUT, DELETE, PATCH.
• **Avantage :** toujours présent pour les requêtes dites « unsafe » cross-site, même si le Referer 
  est supprimé pour confidentialité.
• **Limite :** pour les requêtes GET simples, l'Origin peut parfois être absent.

**Referer (oui, avec une faute historique dans HTTP)**
• Contient l'URL complète de la page qui a initié la requête.
• `Referer: https://mon-site.example/page-formulaire`
• **Avantage :** plus détaillé que Origin, contient chemin + paramètres, utile pour journalisation 
  ou audits.
• **Limite :** certains navigateurs ou extensions peuvent supprimer ou tronquer le Referer pour 
  la vie privée.

---

**NIVEAU DE RISQUE : ÉLEVÉ**  
**FACILITÉ D'EXPLOITATION : MOYENNE**
