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
DATABASE_URL="mysql://root:@127.0.0.1:3306/db_name?serverVersion=10.4.32-MariaDB&charset=utf8mb4"
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
@REM Supprimer les versions antérieures de migration pour éviter les erreurs
@REM Marquer toutes les migrations comme "migrées" dans la DB
symfony console doctrine:migrations:version --delete --all --no-interaction
@REM Supprimer les anciens fichiers de migration
echo yes | del migrations\*.php
@REM Supprime l'ancienne BD
symfony console doctrine:database:drop --force --no-interaction
@REM Crée une nouvelle BD
symfony console doctrine:database:create --no-interaction
@REM Génère de nouvelles migrations
symfony console make:migration --no-interaction
@REM Applique les migrations
symfony console doctrine:migrations:migrate --no-interaction
@REM Supprime et recrée les données dans la DB
symfony console doctrine:fixtures:load --no-interaction
````

## 6. Gestion du fichier .env
> A venir

## 7. Configuration des bundles
> A venir


# Créer et exécuter des migrations
Après avoir créé ou modifié des entités, générer une migration.
````powershell
symfony console make:migration
````
Appliquer ensuite la migration pour mettre à jour la **Base de Données**.
````powershell
symfony console doctrine:migration:migrate
````


# Gérer les routes
Symfony gère les routes dans les fichiers de configuration ``config/routes/`` ou directement via les annotations dans les contrôleurs. Pour ajouter une route via annotation :
````php
#[Route ('/exemples/twig/exemple',name:'exempleTwig')]
public function exemple (){
    return $this->render ('exemples_twig/exemple.html.twig');
}
````

## 1. Route avec Paramètres
Il est possible de passer des paramètres dynamiques aux routes, comme un ID de produit. Par exemple, avec la route ``/product/{id}``, on peut récupérer un produit en fonction de cet ID :
````php
private $doctrine;

public function __construct(ManagerRegistry $doctrine)
{
    $this->doctrine = $doctrine;
}

#[Route('/product/{id}', name: 'product_show')]
public function show(int $id): Response
{
    $product = $this->doctrine->getRepository(Product::class)->find(['id' => $id]);

    return $this->render('product/show.html.twig', [
        'product' => $product,
    ]);
}
````

## 2. Routes avec Contraintes
Les routes avec contraintes permettent de restreindre les paramètres dynamiques à des formats spécifiques en utilisant des expressions régulières. Cela garantit que les valeurs passées respectent certains critères avant d’être traitées par le contrôleur.

### Définir une contrainte sur un paramètre
Dans un fichier de configuration de routes (annotations, YAML ou XML), on peut appliquer des restrictions avec des expressions régulières sur les paramètres.

**Via les annotations :**
````php
#[Route('/product/{id}', name: 'product_show', requirements: ['id' => '\d+'])]
public function show(int $id): Response
{
    // Le paramètre {id} doit être un entier
    // Code pour récupérer et afficher le produit
}
````
Ici, ``\d+`` signifie que le paramètre ``{id}`` doit être un ou plusieurs chiffres. Si une valeur non numérique est passée dans l’URL, Symfony renverra automatiquement une erreur 404.

**Via les fichiers de configuration YAML :**
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
Lorsqu'on définit un paramètre comme optionnel, on doit lui attribuer une valeur par défaut dans la route. Symfony utilisera cette valeur si l'utilisateur ne fournit pas le paramètre dans l'URL.

**Via les annotations :**
````php
#[Route('/product/{id}/{slug?}', name: 'product_show')]
public function show(int $id, string $slug = 'default-slug'): Response
{
    // Si le slug n'est pas fourni, 'default-slug' sera utilisé
    // Code pour gérer le produit
}
````
Dans cet exemple, le paramètre ``slug`` est facultatif. Si l'utilisateur n'inclut pas ``slug`` dans l'URL, le contrôleur utilisera la valeur par défaut ``'default-slug'``.

**Via les fichiers de configuration YAML :**
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
On peut également appliquer des contraintes sur les paramètres optionnels, tout comme on le ferait pour les paramètres obligatoires. Par exemple, si on a un paramètre de page optionnel, on peut toujours restreindre ce paramètre à des valeurs numériques.
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

Dans cet exemple, une route acceptera des valeurs alternatives pour la catégorie, comme "``electronics``", "``books``", ou "``fashion``".
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
Ici, que l'utilisateur accède à ``/product/{id}`` ou à ``/item/{id}``, la même méthode de contrôleur sera exécutée. Cela permet de prendre en charge plusieurs versions d'une URL tout en centralisant le traitement dans un seul contrôleur.
````php
#[Route('/product/{id}', name: 'product_show')]
#[Route('/item/{id}', name: 'item_show')]
public function show(int $id): Response
{
    // Gérer l'affichage du produit, que l'URL soit /product/{id} ou /item/{id}
}
````

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
On peut également définir des valeurs alternatives pour plusieurs paramètres d'une route.
````php
#[Route('/report/{year}/{month}', name: 'report', requirements: ['year' => '\d{4}', 'month' => '0[1-9]|1[0-2]'])]
public function report(int $year, string $month): Response
{
    // Gérer les rapports en fonction de l'année et du mois
}
````


# Créer des contrôleurs
## 1. Créer un controller simple
Utiliser la commande suivante pour créer un contrôleur :
````powershell
symfony console make:controller ExempleTwig
````
Cette commande génère un contrôleur de base dans le répertoire ``src/Controller`` avec un squelette de base.

Voici un exemple d'un contrôleur généré :
````php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ProductController extends AbstractController
{
    #[Route('/product', name: 'product')]
    public function index(): Response
    {
        return $this->render('product/index.html.twig', [
            'controller_name' => 'ProductController',
        ]);
    }
}
````

### Les principales parties d'un contrôleur

- **Route** : Définit l'URL qui déclenche l'action du contrôleur. Ici, ``#[Route('/product', name: 'product')]`` indique que cette méthode est accessible à l'URL ``/product``.
  
- **Action** : Une méthode dans le contrôleur qui gère la logique pour une requête spécifique.
  
- **Réponse** : Le contrôleur retourne une instance de ``Symfony\Component\HttpFoundation\Response``, qui peut être une vue HTML, JSON, ou autre format.
  
- **Request** : Le contrôleur reçoit une instance de ``Symfony\Component\HttpFoundation\Request``, qui contient les données envoyées par l'utilisateur, comme les paramètres de requête, les données POST, les fichiers, etc.

### Types de réponses d'un controller

- ``render()`` : Permet d'afficher une Vue.
  
- ``redirect()`` : Permet de rediriger le navigateur vers une autre adresse, sans ou avec de paramètres.
  
- ``redirectToRoute()`` : Permet de lancer une action mais en utilisant le nom (name) d'une route.
  
- ``forward()`` : Permet de déléguer l'action (pas d'erreur HTTP).
  
- ``renderView()`` : Permet d'obtenir le rendu d'une vue sans les en-têtes HTML. C'est à dire c'est une action qui ne chargera pas une page (à différence de toutes les précédentes). Elle nous donne simplement le HTML qui correspond à la vue choisie. C'est utile si on utilise AJAX.