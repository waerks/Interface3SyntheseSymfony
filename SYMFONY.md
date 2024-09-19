# Récapitulatif des commandes clés
- Créer un projet : ``symfony new --webapp MyProjet``
- Démarrer le serveur : ``symfony server:start``
- Créer une base de données : ``symfony console doctrine:database:create``
- Créer une entité : ``symfony console make:entity``
- Créer une migration : ``symfony console make:migration``
- Appliquer les migrations : ``symfony console doctrine:migrations:migrate``
- Créer un contrôleur : ``symfony console make:controller``
- Nettoyer le cache : ``symfony console cache:clear``


# Étapes de création du projet Symfony
## 1. Créer un nouveau projet Symfony
Aller dans un dossier ``htdocs`` choisi et créer un projet squelette :
````powershell
symfony new --webapp MyProjet
````

## 2. Naviguer dans le répertoire du projet
````powershell
cd MyProjet
````

## 3. Configurer un serveur de développement
Lancer le serveur web interne de Symfony depuis la console :
````powershell
symfony server:start
````
Ou :
````powershell
symfony serve
````
Pour arreter le serveur, utiliser CTRL-C ou lancer depuis une autre console :
````powershell
symfony server:stop
````
Allumer XAMPP et aller dans le navigateur, via [http://localhost:8000/](http://localhost:8000/)

## 4. Configurer la base de données
Configurer les paramètres de connexion à la base de données dans le fichier ``.env`` :
````php
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=8.0"
````
> [!IMPORTANT]
> ATTENTION : La version de MariaDB (MySql) doit correspondre à celle de XAMPP. Pour la connaitre, aller dans la page d'accueil de phpmyadmin

Créer la base de données.
````powershell
symfony console doctrine:database:create
````

## 5. Configurer un fichier .bat
Lancer un fichier migration.bat pour se simplifier la ré-création de la BD et lancer les fixtures. Le fichier doit se trouver dans la racine du projet.
> [!IMPORTANT]
> ATTENTION : Ce fichier efface la BD existante !

````powershell
@REM @REM permet de commenter du code dans un fichier bat
@REM Supprimer les versions antérieurs de migration pour éviter les erreurs
@REM (Commande du systemene d'exploitation et non de symfony)
    echo yes | del migrations
@REM Pour créer la base de donnée cf env. pour le nom de la BD
@REM Supprime l'ancienne BD
    symfony console doctrine:database:drop --force --no-interaction
@REM Crée nouvelle BD
    symfony console doctrine:database:create --no-interaction
@REM Migration
    symfony console make:migration --no-interaction
    symfony console doctrine:migration:migrate --no-interaction
@REM supprime et recrée les données dans la DB
    symfony console doctrine:fixtures:load --no-interaction
@REM ajoute données dans la DB après donc risque doublons
    @REM symfony console doctrine:fixtures:load --append
````

## 6. Gestion du fichier .env
> A venir

## 7. Configuration des bundles
> A venir

# Gérer les routes
Symfony gère les routes dans les fichiers de configuration ``config/routes/`` ou directement via les annotations dans les contrôleurs. Pour ajouter une route via annotation :
````php
#[Route ('/exemples/twig/exemple1',name:'exempleTwig')]
public function exemple (){
    return $this->render ('exemples_twig/exemple.html.twig');
}
````

## 1. Route avec Paramètres
Il est possible de passer des paramètres dynamiques aux routes, comme un ID de produit. Par exemple, avec la route ``/product/{id}``, on peut récupérer un produit en fonction de cet ID :
````php
#[Route('/product/{id}', name: 'product_show')]
public function show(int $id): Response
{
    $product = $this->getDoctrine()
        ->getRepository(Product::class)
        ->find($id);

    if (!$product) {
        throw $this->createNotFoundException('No product found for id '.$id);
    }

    return $this->render('product/show.html.twig', [
        'product' => $product,
    ]);
}
````
Ici, le paramètre ``{id}`` est passé dynamiquement dans l'URL et utilisé pour récupérer le produit depuis la base de données. Si aucun produit n'est trouvé, une exception est levée. Ensuite, le produit est affiché dans un template Twig.

## 2. Routes avec Contraintes
Les routes avec contraintes permettent de restreindre les paramètres dynamiques à des formats spécifiques en utilisant des expressions régulières. Cela garantit que les valeurs passées respectent certains critères avant d’être traitées par le contrôleur.

### Définir une contrainte sur un paramètre
Dans un fichier de configuration de routes (annotations, YAML ou XML), on peut appliquer des restrictions avec des expressions régulières sur les paramètres.

Via les annotations :
````php
#[Route('/product/{id}', name: 'product_show', requirements: ['id' => '\d+'])]
public function show(int $id): Response
{
    // Le paramètre {id} doit être un entier
    // Code pour récupérer et afficher le produit
}
````
Ici, ``\d+`` signifie que le paramètre ``{id}`` doit être un ou plusieurs chiffres. Si une valeur non numérique est passée dans l’URL, Symfony renverra automatiquement une erreur 404.

Via les fichiers de configuration YAML :
````yaml
product_show:
    path: /product/{id}
    controller: App\Controller\ProductController::show
    requirements:
        id: '\d+'
````

### Contraintes multiples
Il est possible d’ajouter plusieurs contraintes sur différents paramètres. Par exemple, on peut exiger que le ``slug`` d’un produit soit composé uniquement de lettres minuscules et de tirets.
````php
#[Route('/product/{id}/{slug}', name: 'product_show', requirements: ['id' => '\d+', 'slug' => '[a-z\-]+'])]
public function show(int $id, string $slug): Response
{
    // Le paramètre {id} doit être un entier et {slug} doit être composé de lettres minuscules et tirets
    // Code pour récupérer et afficher le produit
}
````

### Valeurs par défaut et contraintes optionnelles
On peut combiner des contraintes avec des paramètres optionnels. Par exemple, si le paramètre ``page`` est facultatif mais doit, s’il est fourni, être un entier :
````php
#[Route('/product/{id}/{page?1}', name: 'product_show', requirements: ['id' => '\d+', 'page' => '\d+'])]
public function show(int $id, int $page): Response
{
    // Page est optionnel, mais doit être un entier si présent
    // Code pour gérer l'affichage paginé des produits
}
````

## 3. Routes avec Paramètres optionnels
Il est possible de définir des paramètres optionnels dans les routes. Cela permet de rendre un ou plusieurs paramètres facultatifs, ce qui rend l'URL plus flexible tout en fournissant une valeur par défaut si le paramètre n'est pas présent.

### Utiliser une valeur par défaut
Lorsque vous définisser un paramètre comme optionnel, vous devez lui attribuer une valeur par défaut dans la route. Symfony utilisera cette valeur si l'utilisateur ne fournit pas le paramètre dans l'URL.

Exemple en annotations :
````php
#[Route('/product/{id}/{slug?}', name: 'product_show')]
public function show(int $id, string $slug = 'default-slug'): Response
{
    // Si le slug n'est pas fourni, 'default-slug' sera utilisé
    // Code pour gérer le produit
}
````
Dans cet exemple, le paramètre ``slug`` est facultatif. Si l'utilisateur n'inclut pas ``slug`` dans l'URL, le contrôleur utilisera la valeur par défaut ``'default-slug'``.

Exemple en YAML :
````yaml
product_show:
    path: /product/{id}/{slug}
    controller: App\Controller\ProductController::show
    defaults:
        slug: 'default-slug'
````
Ici aussi, si le ``slug`` n'est pas fourni dans l'URL, la valeur par défaut ``'default-slug'`` sera utilisée.

### Routes optionnelles avec un seul paramètre
Un autre exemple consiste à utiliser un seul paramètre optionnel. Par exemple, une route pour afficher une liste d'articles paginée, où la page par défaut est ``1`` si elle n'est pas spécifiée :
````php
#[Route('/articles/{page?1}', name: 'article_list')]
public function list(int $page): Response
{
    // $page sera 1 si l'utilisateur n'a pas spécifié de page dans l'URL
    // Code pour afficher les articles paginés
}
````
Ici, le paramètre ``page`` est optionnel et prendra la valeur ``1`` si l'utilisateur ne la spécifie pas dans l'URL.

### Utilisation de plusieurs paramètres optionnels
Il est possible d'avoir plusieurs paramètres optionnels dans une route, chacun avec sa propre valeur par défaut.
````php
#[Route('/search/{category?}/{page?1}', name: 'search')]
public function search(string $category = 'all', int $page = 1): Response
{
    // $category sera 'all' et $page sera 1 si non spécifiés
    // Code pour gérer la recherche
}
````
Dans cet exemple, si l'utilisateur ne spécifie ni ``category`` ni ``page``, les valeurs ``'all'`` et ``1`` seront utilisées par défaut.

### Contraintes sur les paramètres optionnels
Vous pouvez également appliquer des contraintes sur les paramètres optionnels, tout comme vous le feriez pour les paramètres obligatoires. Par exemple, si vous avez un paramètre de page optionnel, vous pouvez toujours restreindre ce paramètre à des valeurs numériques.
````php
#[Route('/articles/{page?1}', name: 'article_list', requirements: ['page' => '\d+'])]
public function list(int $page): Response
{
    // La page doit être un nombre entier
}
````
Cela garantit que, même s'il est optionnel, le paramètre respecte certaines règles s'il est fourni.

## 4. Routes avec Valeurs alternatives
Les routes avec valeurs alternatives permettent de définir plusieurs chemins possibles pour une même action du contrôleur. Cela offre la possibilité de gérer différentes versions d'une URL ou d'accepter plusieurs formats d'URL pour la même logique métier, en fonction des besoins de l'application.

### Utiliser des valeurs alternatives avec des expressions régulières
**Exemple : plusieurs catégories**

Dans cet exemple, une route acceptera des valeurs alternatives pour la catégorie, comme "electronics", "books", ou "fashion".
````php
#[Route('/product/{category}', name: 'product_category', requirements: ['category' => 'electronics|books|fashion'])]
public function showByCategory(string $category): Response
{
    // Gérer les produits en fonction de la catégorie
}
````
Ici, le paramètre ``{category}`` peut prendre trois valeurs possibles : ``electronics``, ``books``, ou ``fashion``. Si l'utilisateur entre une autre valeur, Symfony renverra une erreur 404.

**Exemple : plusieurs formats de date**

Vous pouvez également utiliser des expressions régulières pour autoriser plusieurs formats dans l'URL. Par exemple, permettre à un utilisateur de fournir une date au format ``YYYY-MM-DD`` ou ``DD-MM-YYYY`` :
````php
#[Route('/events/{date}', name: 'events_by_date', requirements: ['date' => '\d{4}-\d{2}-\d{2}|\d{2}-\d{2}-\d{4}'])]
public function eventsByDate(string $date): Response
{
    // Gérer les événements en fonction de la date
}
````

### Définir plusieurs routes pour un même contrôleur
**Exemple avec annotations :**
````php
#[Route('/product/{id}', name: 'product_show')]
#[Route('/item/{id}', name: 'item_show')]
public function show(int $id): Response
{
    // Gérer l'affichage du produit, que l'URL soit /product/{id} ou /item/{id}
}
````
Ici, que l'utilisateur accède à ``/product/{id}`` ou à ``/item/{id}``, la même méthode de contrôleur sera exécutée. Cela permet de prendre en charge plusieurs versions d'une URL tout en centralisant le traitement dans un seul contrôleur.

### Utiliser des valeurs par défaut avec des alternatives
Il est également possible de combiner des valeurs par défaut avec des valeurs alternatives dans les paramètres de route. Par exemple, permettre plusieurs langues dans l'URL avec une valeur par défaut :
````php
#[Route('/{_locale}/contact', name: 'contact', defaults: ['_locale' => 'en'], requirements: ['_locale' => 'en|fr|es'])]
public function contact(string $_locale): Response
{
    // Gérer la page de contact en fonction de la langue
}
````
Dans cet exemple, la route ``/en/contact`` sera la valeur par défaut, mais l'utilisateur peut aussi accéder à ``/fr/contact`` ou ``/es/contact`` pour les versions en français et espagnol. Symfony redirigera vers la bonne version de la page en fonction de la langue fournie dans l'URL.

### Gérer plusieurs paramètres avec des valeurs alternatives
Vous pouvez également définir des valeurs alternatives pour plusieurs paramètres d'une route.
````php
#[Route('/report/{year}/{month}', name: 'report', requirements: ['year' => '\d{4}', 'month' => '0[1-9]|1[0-2]'])]
public function report(int $year, string $month): Response
{
    // Gérer les rapports en fonction de l'année et du mois
}
````

# Créer et exécuter des migrations
Après avoir créé ou modifié des entités, générer une migration.
````powershell
symfony console make:migration
````
Appliquer ensuite la migration pour mettre à jour la **Base de Données**.
````powershell
symfony console doctrine:migration:migrate
````

# Créer des contrôleurs
## 1. Créer un controller simpple
## 2. Structure d'un Contrôleur
### Les principales parties d'un contrôleur
### Types de réponses d'un controller
## 3. Utiliser des templates avec Twig
## 4. Retourner une Vue (HTML)
## 5. Redirection
## 6. Gestion des erreurs

# Le modèle
## 1. Créer des Entités
## 2. Rajouter et effacer des propriétés
## 3. Les relations

# Le modèle : Accès à la DB avec Doctrine
## 1. SELECT
## 2. INSERT
## 3. UPDATE
## 4. DELETE

# Le modèle : Pertsistance

# Le modèle : Hydrate

# Le modèle : Créer des fixtures
## 1. Créer des fixtures
## 2. Fixtures dans ManyToMany
## 3. Fixtures dans OneToMany/ManyToOne
## 4. Fixtures dans OneToOne

# Accès à la DB : DQL
## 1. SELECT
## 2. INSERT
## 3. UPDATE
## 4. DELETE

# Accès à la DB : Repository
# Accès à la DB : QueryBuilder

# Le formulaire
## 1. Création d'un formulaire simple
## 2. Valider un formulaire
## 3. Formulaire avec ManyToMany
## 4. Formulaire avec OneToMany/ManyToOne
## 5. Formulaire avec OneToOne
## 6. Formulaire d'insertion
## 7. Formulaire d'update
## 8. Formulaire de recherche

# Entité User

# Authentification
## 1. Inscription
## 2. Connexion
## 3. Déconnexion
## 4. Rôles
## 5. Sécurité

# Response JSON

# Suppléments
## 1. CSS
## 2. Bootstrap et Tailwind CSS
## 3. Filtre
## 4. Pagination
## 5. Mail