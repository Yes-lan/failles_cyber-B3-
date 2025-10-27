# LES API (Application Programming Interface)

## INTRODUCTION

API soit Application Programming Interface, ou interface de programmation d'application, est un 
ensemble de règles qui permet à deux programmes ou systèmes de communiquer entre eux, sans qu'ils 
aient besoin de connaître leurs fonctionnements internes.

Une API agit comme un intermédiaire standardisé entre différentes applications, permettant 
l'échange de données et de fonctionnalités sans exposer le code source ou la logique interne 
de chaque système.

**Exemple :**
Quand vous utilisez une application météo sur votre téléphone, elle ne calcule pas elle-même 
la météo. Elle envoie une requête vers une API qui formate correctement la requête puis l'envoie 
vers le serveur météo, qui renvoie les données de nuages, de pluie, de température, etc.

Il existe principalement deux types d'API dans le développement web moderne : les API REST et 
les API GraphQL.

---

## LES API REST

### Présentation

L'un des types d'API les plus répandus est le REST, pour Representational State Transfer.

Il utilise les méthodes du protocole HTTP (GET, POST, PUT, DELETE, etc.) du client vers le serveur, 
pour indiquer les actions à effectuer sur les ressources.

Tandis que les données échangées du serveur vers le client sont généralement structurées sous forme 
de JSON (JavaScript Object Notation), un format léger et facile à lire.

REST est simple et largement compatible, mais il peut devenir lourd en informations car on reçoit 
parfois plus de données que nécessaire.

---

### Exemple illustré : Application Météo

Pour illustrer comment fonctionne une API REST, revenons sur l'exemple précédent de l'application 
météo :

1. L'application envoie une requête HTTP :
   ```
   GET /api/weather?city=Paris
   ```

2. Le serveur reçoit la requête et renvoie les données au format JSON :
   ```json
   {
     "city": "Paris",
     "temperature": 15,
     "condition": "nuageux",
     "humidity": 65,
     "wind_speed": 20
   }
   ```

3. L'application affiche les informations pertinentes à l'utilisateur

Le problème ici : même si l'application n'a besoin que de la température et de la condition météo, 
elle reçoit toutes les données (humidity, wind_speed, etc.). C'est là qu'intervient GraphQL.

---

## LES API GRAPHQL

### Présentation

GraphQL, bien qu'étant une alternative à REST, n'est pas une API intermédiaire. Le serveur GraphQL 
est lui-même l'API, recevant ainsi directement les requêtes du client sans intermédiaire.

Sauf qu'ici, le client définit exactement les données qu'il veut recevoir.

L'API GraphQL ne possède qu'une seule URL d'entrée, souvent /graphql, et le client envoie depuis 
le body une requête structurée sous forme de graphe. Là où l'URL d'entrée de REST dépend de la 
requête, qui est elle-même dans l'URL.

---

### Exemple illustré

Au lieu de la requête REST GET /users?id=1 qui renvoie toutes les informations d'un utilisateur,

dans le cas où on recherche une journaliste, on peut demander de n'avoir seulement que son nom, 
ses articles et leurs dates de publications, rien de plus.

**Requête GraphQL :**
```graphql
{
  user(id: 1) {
    name
    articles {
      title
      publishedDate
    }
  }
}
```

De son côté, une fois la requête reçue, le serveur enverra exactement ce qui a été demandé :
```json
{
  "data": {
    "user": {
      "name": "Marie Dupont",
      "articles": [
        {
          "title": "Article sur le climat",
          "publishedDate": "2025-10-20"
        },
        {
          "title": "Nouveau rapport économique",
          "publishedDate": "2025-10-15"
        }
      ]
    }
  }
}
```

Pas de données superflues, juste ce qui est nécessaire.

---

## COMPARAISON REST vs GRAPHQL

Pour comparer les 2 API :

### REST :
**Avantages :**
- Simple
- Largement utilisé
- Compatible avec presque tous les clients
- Facile à mettre en place

**Inconvénients :**
- Parfois lourd en informations
- Chaque ressource a sa propre URL
- Peut nécessiter plusieurs requêtes pour obtenir des données liées

### GraphQL :
**Avantages :**
- Récupère exactement les données demandées
- Une seule URL d'entrée
- Pratique pour des relations complexes entre données
- Réduit le nombre de requêtes nécessaires

**Inconvénients :**
- Mise en place plus complexe
- Nécessite un schéma précis
- Moins pratique pour de petits projets simples
- Courbe d'apprentissage plus importante

---

## EXEMPLES SYMFONY

### API REST avec API Platform

```php
// Entity User
#[ApiResource(
    operations: [
        new Get(),
        new GetCollection(),
        new Post(),
        new Put(),
        new Delete()
    ]
)]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    private ?int $id = null;

    #[ApiProperty]
    private string $name;

    #[ApiProperty]
    private string $email;
}

// Automatiquement accessible via :
// GET    /api/users       (liste tous les utilisateurs)
// GET    /api/users/1     (détails d'un utilisateur)
// POST   /api/users       (créer un utilisateur)
// PUT    /api/users/1     (modifier un utilisateur)
// DELETE /api/users/1     (supprimer un utilisateur)
```

---

### API GraphQL avec API Platform

```php
// Installer le support GraphQL
composer require api-platform/graphql

// La même Entity devient automatiquement disponible en GraphQL
// Query pour récupérer exactement ce qu'on veut :
{
  user(id: "/api/users/1") {
    name
    email
  }
}

// Mutation pour créer un utilisateur :
mutation {
  createUser(input: {
    name: "Jean Martin"
    email: "jean@example.com"
  }) {
    user {
      id
      name
    }
  }
}
```

---

## SÉCURITÉ DES API

Que ce soit REST ou GraphQL, la sécurité est primordiale :

### Authentication et Authorization :
- Tokens JWT (JSON Web Tokens)
- OAuth 2.0
- API Keys

### Rate Limiting :
- Limiter le nombre de requêtes par utilisateur/IP
- Prévenir les attaques DDoS

### Validation des données :
- Valider toutes les entrées
- Utiliser les Constraints Symfony

### CORS (Cross-Origin Resource Sharing) :
- Configurer correctement les origines autorisées

### HTTPS :
- Toujours utiliser HTTPS en production

**Exemple Symfony (config/packages/nelmio_cors.yaml) :**
```yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['^https?://localhost:[0-9]+']
        allow_methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS']
        allow_headers: ['Content-Type', 'Authorization']
        max_age: 3600
```

---

## CONCLUSION

Pour résumer :

• Une API permet la communication entre applications,

• Les API REST sont simples et largement compatibles mais lourdes en informations,

• Tandis que les API GraphQL sont plus précises dans les données transférées mais aussi plus 
  complexes.

Aujourd'hui, REST reste le plus utilisé, mais GraphQL gagne du terrain, notamment dans les projets 
web complexes et possédant un grand nombre de données.

Symfony, via API Platform, supporte excellemment les deux approches, permettant aux développeurs 
de choisir la solution la plus adaptée à leurs besoins.

---

**NIVEAU DE COMPLEXITÉ REST : FAIBLE**  
**NIVEAU DE COMPLEXITÉ GRAPHQL : MOYEN À ÉLEVÉ**
