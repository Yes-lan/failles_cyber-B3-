# FAILLE INJECTION SQL

## INTRODUCTION

L'injection SQL est une technique permettant d'injecter du code SQL malveillant dans une requête,
exploitant une mauvaise validation des entrées utilisateur pour manipuler la base de données.

L'attaquant profite du fait que l'application construit des requêtes SQL en concaténant directement 
des données fournies par l'utilisateur, sans les valider ou les échapper correctement.

Cette faille permet de lire, modifier ou supprimer des données de la base, voire d'exécuter des 
commandes système dans certains cas.

---

## FONCTIONNEMENT

L'attaquant insère du code SQL dans un champ de saisie (formulaire, URL, etc.) qui est ensuite
exécuté par la base de données sans validation appropriée.

Le code malveillant est interprété comme faisant partie de la requête légitime, permettant à 
l'attaquant de manipuler la logique de la requête ou d'accéder à des données non autorisées.

---

## EXEMPLES D'ATTAQUES

### 1. Bypass d'authentification :
```
Login : admin' OR '1'='1' --
Password : anything

Requête générée : SELECT * FROM users WHERE login='admin' OR '1'='1' --' AND password='...'
Résultat : Connexion réussie sans mot de passe
```

### 2. Extraction de données :
```sql
ID : 1 UNION SELECT username, password FROM admin_users --
```

### 3. Suppression de données :
```sql
ID : 1; DROP TABLE users; --
```

### 4. Lecture de fichiers système :
```sql
ID : 1 UNION SELECT LOAD_FILE('/etc/passwd') --
```

---

## TYPES D'INJECTIONS

• **Injection classique (in-band) :** résultats visibles directement dans la réponse  
• **Injection aveugle (blind SQL injection) :** pas de résultat visible, mais réaction différente  
• **Injection basée sur le temps (time-based) :** détection via le temps de réponse  
• **Injection de second ordre :** données malveillantes stockées puis exécutées plus tard

---

## CODE VULNÉRABLE

```php
// PHP brut (DANGEREUX)
$id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = $id";
$result = mysqli_query($conn, $query);

// Symfony avec DQL mal construit (VULNÉRABLE)
$em = $this->getDoctrine()->getManager();
$query = $em->createQuery("SELECT u FROM App\Entity\User u WHERE u.email = '" . $email . "'");
```

---

## CONSÉQUENCES POSSIBLES

• Vol massif de données (dump de base de données)  
• Modification ou suppression de données  
• Bypass de l'authentification  
• Escalade de privilèges  
• Accès au système de fichiers  
• Exécution de commandes système (dans certains cas)  
• Compromission totale du serveur

---

## PROTECTION

Plusieurs techniques permettent d'éviter les injections SQL :

• **TOUJOURS** utiliser des requêtes préparées (prepared statements)

• Utiliser un ORM (Doctrine pour Symfony)

• Valider et assainir **TOUTES** les entrées utilisateur

• Utiliser des paramètres bindés

• Appliquer le principe du moindre privilège (comptes DB limités)

• Encoder les sorties

• Utiliser des listes blanches pour les valeurs autorisées

• Désactiver les messages d'erreur détaillés en production

---

## CAS SYMFONY

Symfony, via Doctrine ORM, offre une protection native contre les injections SQL grâce aux 
requêtes préparées et au QueryBuilder.

Tant que le développeur utilise correctement ces outils, les injections SQL sont pratiquement 
impossibles.

---

## EXEMPLES SYMFONY SÉCURISÉS

```php
// Doctrine QueryBuilder (SÉCURISÉ)
$qb = $repository->createQueryBuilder('u')
    ->where('u.email = :email')
    ->setParameter('email', $email)
    ->getQuery()
    ->getResult();

// Requête préparée DQL (SÉCURISÉ)
$query = $em->createQuery('SELECT u FROM App\Entity\User u WHERE u.id = :id')
    ->setParameter('id', $id);

// Repository avec méthode find (SÉCURISÉ)
$user = $repository->find($id);

// Requête SQL native avec paramètres (SÉCURISÉ)
$conn = $em->getConnection();
$sql = 'SELECT * FROM users WHERE email = :email';
$stmt = $conn->prepare($sql);
$stmt->executeQuery(['email' => $email]);
```

---

## BONNES PRATIQUES API PLATFORM

```php
// Utiliser les filtres intégrés
#[ApiFilter(SearchFilter::class, properties: ['email' => 'exact'])]
#[ApiFilter(OrderFilter::class, properties: ['createdAt' => 'DESC'])]

// Les paramètres sont automatiquement sécurisés par Doctrine
```

---

## DÉTECTION ET TESTS

• Scanner de vulnérabilités : SQLMap, Acunetix, Nessus  
• WAF (Web Application Firewall)  
• Tests d'intrusion manuels  
• Code review automatisé  
• Tests unitaires avec entrées malveillantes

---

## PAYLOAD DE TEST CLASSIQUES

```
' OR '1'='1
' OR '1'='1' --
' OR '1'='1' ({
' OR '1'='1' /*
admin' --
admin' #
' UNION SELECT NULL--
' AND 1=0 UNION ALL SELECT 'admin', 'password'--
```

⚠️ **N'utilisez ces payloads QUE sur vos propres applications dans un cadre légal !**

---

## CONCLUSION

L'injection SQL est l'une des vulnérabilités les plus dangereuses et les plus anciennes du web.

Dans notre cas, Symfony avec Doctrine ORM offre une protection robuste grâce aux requêtes préparées 
et au QueryBuilder qui paramètrent automatiquement toutes les valeurs.

**La règle d'or : ne JAMAIS concaténer directement des données utilisateur dans une requête SQL.**

---

**NIVEAU DE RISQUE : CRITIQUE**  
**FACILITÉ D'EXPLOITATION : MOYENNE**  
**TOP OWASP : #3**
