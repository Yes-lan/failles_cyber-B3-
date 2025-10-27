# FAILLE MVC (Model-View-Controller)

## INTRODUCTION

Les vulnérabilités liées à l'architecture MVC surviennent lorsque la séparation des responsabilités 
n'est pas correctement respectée, permettant des accès non autorisés ou des manipulations de données.

L'architecture MVC (Model-View-Controller) est un pattern de conception qui sépare une application 
en trois composants principaux : le Modèle (données), la Vue (présentation) et le Contrôleur (logique).

Lorsque ces responsabilités ne sont pas strictement séparées, des failles de sécurité peuvent 
apparaître.

---

## TYPES DE FAILLES MVC

### 1. Logique métier dans les vues
• Exposition de données sensibles directement dans les templates  
• Traitement de données non validées côté présentation

### 2. Contrôleurs mal sécurisés
• Absence de vérification des permissions  
• Validation insuffisante des entrées utilisateur  
• Exposition directe des méthodes du modèle

### 3. Modèles non protégés
• Mass assignment non contrôlé  
• Accès direct aux propriétés sensibles  
• Absence de validation au niveau du modèle

---

## EXEMPLES D'EXPLOITATION

• Modification de paramètres non autorisés via mass assignment  
• Accès à des données sensibles via des vues mal configurées  
• Bypassing de la logique métier en accédant directement aux modèles

---

## CONSÉQUENCES POSSIBLES

• Accès non autorisé à des données sensibles  
• Modification de données critiques  
• Escalade de privilèges  
• Bypass de la logique métier  
• Exposition d'informations système

---

## PROTECTION

Plusieurs techniques permettent d'éviter les failles MVC :

• Utiliser des FormTypes Symfony pour valider toutes les entrées

• Implémenter le Voter system pour les autorisations

• Utiliser des DTOs (Data Transfer Objects) pour limiter l'exposition

• Valider les données au niveau du modèle avec Constraints

• Ne jamais faire confiance aux données provenant du client

• Séparer strictement les responsabilités (Model/View/Controller)

• Utiliser des normalizers pour contrôler la sérialisation

---

## CAS SYMFONY

Symfony offre plusieurs outils pour sécuriser l'architecture MVC :

• Security Voters pour les permissions  
• FormTypes avec validation intégrée  
• Composant Validator pour les entités  
• Groupes de sérialisation API Platform  
• DTOs pour API Platform

---

## EXEMPLES SYMFONY

```php
// Mauvais exemple (VULNÉRABLE) - Mass Assignment
#[Route('/user/update', methods: ['POST'])]
public function update(Request $request, EntityManagerInterface $em): Response
{
    $user = $this->getUser();
    $data = json_decode($request->getContent(), true);
    
    // DANGER : L'utilisateur peut modifier n'importe quelle propriété
    foreach ($data as $key => $value) {
        $setter = 'set' . ucfirst($key);
        if (method_exists($user, $setter)) {
            $user->$setter($value);  // Il peut modifier isAdmin, roles, etc.
        }
    }
    
    $em->flush();
    return $this->json(['status' => 'updated']);
}

// Bon exemple (SÉCURISÉ) - Utilisation de FormType
#[Route('/user/update', methods: ['POST'])]
public function update(Request $request, EntityManagerInterface $em): Response
{
    $user = $this->getUser();
    $form = $this->createForm(UserProfileType::class, $user);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        // Seuls les champs définis dans le FormType peuvent être modifiés
        $em->flush();
        return $this->json(['status' => 'updated']);
    }
    
    return $this->json(['error' => 'Invalid data'], 400);
}

// FormType avec validation
class UserProfileType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Length(['max' => 100])
                ]
            ])
            ->add('email', EmailType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Email()
                ]
            ]);
            // isAdmin, roles, etc. ne sont PAS exposés
    }
}

// Security Voter pour les autorisations
class ArticleVoter extends Voter
{
    protected function supports(string $attribute, mixed $subject): bool
    {
        return in_array($attribute, ['VIEW', 'EDIT', 'DELETE'])
            && $subject instanceof Article;
    }

    protected function voteOnAttribute(
        string $attribute, 
        mixed $subject, 
        TokenInterface $token
    ): bool {
        $user = $token->getUser();
        
        if (!$user instanceof User) {
            return false;
        }

        /** @var Article $article */
        $article = $subject;

        return match($attribute) {
            'VIEW' => true,
            'EDIT', 'DELETE' => $article->getAuthor() === $user 
                || in_array('ROLE_ADMIN', $user->getRoles()),
            default => false,
        };
    }
}
```

---

## CONCLUSION

Les failles MVC reposent sur une mauvaise séparation des responsabilités dans l'architecture 
de l'application.

Dans notre cas, Symfony offre des outils robustes (FormTypes, Voters, Validators) qui, lorsqu'ils 
sont correctement utilisés, permettent de maintenir une séparation stricte des responsabilités et 
de prévenir efficacement ces vulnérabilités.

---

**NIVEAU DE RISQUE : MOYEN À ÉLEVÉ**  
**FACILITÉ D'EXPLOITATION : MOYENNE**
