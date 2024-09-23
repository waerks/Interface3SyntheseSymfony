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
Utilisez la commande suivante pour créer un contrôleur :
````powershell
symfony console make:controller ExemplesTwig
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

## 2. Structure d'un Contrôleur
Un contrôleur est essentiellement une méthode PHP dans une classe qui est associée à une route via des annotations ou des fichiers de configuration YAML/XML. 

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


## 3. Les templates Twig
Symfony utilise Twig comme moteur de templates. Pour rendre une vue depuis un contrôleur :
````php
return $this->render('my_template.html.twig', [
    'variable' => $value,
]);
````
Ou 
````php
$vars = [
    'variable' => $value,
];

return $this->render('my_template.html.twig', $vars);
````
Placer les fichiers Twig dans le répertoire ``templates/``.

### Syntaxe de base de Twig
Twig utilise une syntaxe simple et lisible. Voici quelques constructions de base :

- ``{{ ... }}`` pour afficher une variable.
- ``{% ... %}`` pour des instructions de contrôle comme les boucles et les conditions.

Afficher une variable :
````twig
{{ product.name }}
````

Conditions :
````twig
{% if product.stock > 0 %}
    In Stock
{% else %}
    Out of Stock
{% endif %}
````

Boucles :
````twig
{% for product in products %}
    <li>{{ product.name }} - {{ product.price }}</li>
{% endfor %}
````

### Extension de templates (Heritage)
Twig permet d'utiliser l'héritage de templates, ce qui facilite la réutilisation du code. Par exemple, vous pouvez définir un layout (modèle de base) commun dans ``base.html.twig`` :
````twig
<!DOCTYPE html>
<html>
    <head>
        <title>{% block title %}My App{% endblock %}</title>
    </head>
    <body>
        <header>
            <h1>Welcome to My App</h1>
        </header>
        
        <div>
            {% block body %}{% endblock %}
        </div>
    </body>
</html>
````

Ensuite, dans vos autres templates, vous pouvez hériter de ce layout en utilisant ``{% extends %}`` et remplir les blocs définis dans le layout (``title``, ``body``, etc.) :

````twig
{% extends 'base.html.twig' %}

{% block title %}Product List{% endblock %}

{% block body %}
    <ul>
    {% for product in products %}
        <li>{{ product.name }} - {{ product.price }}</li>
    {% endfor %}
    </ul>
{% endblock %}
````

### Inclusion des templates (Heritage)
En utilisant la directive ``{% include %}``, vous pouvez diviser votre page en plusieurs parties réutilisables. Par exemple, au lieu de définir l'en-tête et le pied de page dans chaque template, vous pouvez les extraire dans des fichiers séparés et les inclure là où c'est nécessaire.
````twig
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Page Title{% endblock %}</title>
</head>
<body>
    <header>
        {% include 'exemples_twig_heritage/header.html.twig' %}
    </header>
    <main>
        {% block contenuPrincipal %}{% endblock %}
    </main>
    <footer>
        {% include 'exemples_twig_heritage/footer.html.twig' %}
    </footer>
</body>
</html>
````

Et créez les actions et les routes.

````php
#[Route("/exemples/twig/heritage/contenu1/master/page2")]
public function contenu1MasterPage2()
{
    return $this->render('exemples_twig_heritage/contenu_1_master_page_2.html.twig');
}

#[Route("/exemples/twig/heritage/contenu2/master/page2")]
public function contenu2MasterPage2()
{
    return $this->render('exemples_twig_heritage/contenu_2_master_page_2.html.twig');
}
````

## 4. Retourner une Vue (HTML)
L'une des principales fonctions d'un contrôleur est de renvoyer une vue Twig en réponse à une requête HTTP. Voici un exemple où une action retourne une page HTML avec des variables passées au template :
````php
#[Route('/product/list', name: 'product_list')]
public function list(): Response
{
    $products = [
        ['name' => 'Laptop', 'price' => 1000],
        ['name' => 'Phone', 'price' => 500]
    ];

    $vars = [
        'products' => $products,
    ];

    return $this->render('product/list.html.twig', $vars);
}
````

Le template Twig associé ``product/list.html.twig`` :
````twig
<h1>Product List</h1>
<ul>
    {% for product in products %}
        <li>{{ product.name }} - {{ product.price }}</li>
    {% endfor %}
</ul>
````

## 5. Redirection
Parfois, on a besoin de rediriger l'utilisateur vers une autre page après une certaine action, par exemple, après avoir soumis un formulaire ou créé un enregistrement. Utilisez la méthode ``redirectToRoute()`` pour rediriger l'utilisateur vers une autre route :
````php
#[Route('/product/create', name: 'product_create')]
public function create(): Response
{
    // Logique de création de produit

    return $this->redirectToRoute('product_list');
}
````

## 6. Gestion des erreurs
### Exceptions et Erreurs
En Symfony, les erreurs sont souvent traitées sous forme d'exceptions. Une exception est un objet qui représente un événement exceptionnel (erreur) qui interrompt le flux normal d'une application. Symfony utilise une hiérarchie d'exceptions pour représenter les différents types d'erreurs :
- Exceptions générales : Ce sont des exceptions levées par PHP ou par l'application (comme ``LogicException``, ``InvalidArgumentException``, etc.).
- Exceptions HTTP : Elles représentent des erreurs spécifiques à la gestion des requêtes HTTP, telles que ``HttpException`` ou ``NotFoundHttpException``.

### Exemple d'exception personnalisée
Dans un contrôleur, vous pouvez lever une exception si une certaine condition n'est pas remplie :

````php
public function showProduct(int $id)
{
    $product = $this->getDoctrine()->getRepository(Product::class)->find($id);

    if (!$product) {
        throw $this->createNotFoundException('Product not found.');
    }

    // autre logique
}
````
Ici, ``createNotFoundException()`` lance une exception ``NotFoundHttpException``, qui est automatiquement traduite en une réponse 404.

### Gestionnaire d'Exceptions
Symfony utilise un gestionnaire d'exceptions (Exception Listener) pour intercepter les exceptions non capturées et les convertir en réponses HTTP. Cela garantit que toute exception lancée dans une application Symfony sera interceptée, puis transformée en une réponse HTTP appropriée.

**Page d'erreur en mode debug**

Lorsque vous travaillez en environnement de développement ``(APP_ENV=dev)``, Symfony vous montre une page d'erreur avec des détails exhaustifs sur l'exception, y compris :

- Le message d'erreur
- La pile d'appels (stack trace)
- Les variables locales

Cela vous aide à identifier rapidement l'origine de l'erreur.

### Pages d'erreurs HTTP (404, 500, etc.)
Symfony permet de personnaliser les réponses d'erreurs HTTP courantes, comme les erreurs 404 (page non trouvée) ou 500 (erreur serveur). Par défaut, Symfony fournit des pages d'erreur basiques, mais vous pouvez créer vos propres templates pour offrir une meilleure expérience utilisateur.

**Personnalisation des pages d'erreur :**

Pour personnaliser les pages d'erreurs comme 404 ou 500, créez des templates Twig dans le dossier ``templates/bundles/TwigBundle/Exception/`` :

- ``404.html.twig`` : pour les erreurs 404 (page non trouvée).
- ``500.html.twig`` : pour les erreurs 500 (erreur serveur).

Exemple d'une page d'erreur 404 personnalisée :
````twig 
{# templates/bundles/TwigBundle/Exception/error404.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Page Not Found{% endblock %}

{% block body %}
    <h1>Oops! This page could not be found.</h1>
    <p>The page you are looking for might have been removed, had its name changed, or is temporarily unavailable.</p>
    <a href="{{ path('homepage') }}">Go back to the homepage</a>
{% endblock %}
````
Cette page personnalisée sera servie automatiquement lorsque Symfony renverra une erreur 404.

### Gestion des Erreurs et des Logs
Symfony utilise le composant Monolog pour gérer les logs d'erreurs. Cela permet de suivre et d'enregistrer toutes les exceptions qui se produisent dans l'application, même celles qui ne sont pas directement visibles pour les utilisateurs. Monolog peut envoyer les logs dans différents supports comme les fichiers, les bases de données, ou des services externes.

Configuration des logs dans ``config/packages/monolog.yaml`` :
````yaml
monolog:
    handlers:
        main:
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            level: error
        console:
            type: console
            process_psr_3_messages: false
            channels: ["!event", "!doctrine"]
````
Cela enregistre les erreurs et les événements importants dans des fichiers de log, qui peuvent être consultés pour diagnostiquer des problèmes ou analyser les performances de l'application.

### Redirection vers une page d'erreur personnalisée
Si vous souhaitez rediriger vers une page d'erreur spécifique en cas d'exception, vous pouvez intercepter l'exception dans un **Event Listener** ou un **Event Subscriber**. Ce processus vous permet de gérer certaines exceptions de manière plus fine et d'afficher des pages d'erreur plus appropriées selon le contexte de votre application.

Exemple d'Event Listener pour gérer les erreurs :
````php
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\RouterInterface;
use Twig\Environment;

class ExceptionListener
{
    private $twig;
    private $router;

    public function __construct(Environment $twig, RouterInterface $router)
    {
        $this->twig = $twig;
        $this->router = $router;
    }

    public function onKernelException(ExceptionEvent $event)
    {
        $exception = $event->getThrowable();
        
        if ($exception instanceof NotFoundHttpException) {
            $response = new Response($this->twig->render('error/404.html.twig'));
            $event->setResponse($response);
        } elseif ($exception instanceof \Exception) {
            $response = new Response($this->twig->render('error/500.html.twig'));
            $event->setResponse($response);
        }
    }
}
````
Cet exemple montre comment capturer différentes exceptions et afficher des pages d'erreur spécifiques selon le type d'exception.

# Le modèle
## 1. Créer des Entités
Pour créer une entité :
````powershell
symfony console make:entity
````
Ou, définir directement le nom de l'entité :
````powershell
symfony console make:entity Product
````

Après avoir nommé l'entité, Symfony vous demande d'ajouter des propriétés. Par exemple :

- Nom : ``name`` (type ``string``)
- Prix : ``price`` (type ``float``)

Ces propriétés seront traduites en colonnes dans la table ``product``.

Une fois terminé, une classe sera générée dans le répertoire ``src/Entity``, par exemple ``src/Entity/Product.php``.
## 2. Ajouter, modifier ou supprimer des propriétés
### Ajouter des propriétés
**Via la commande ``make:entity``**
Par exemple, si vous voulez ajouter une propriété à l'entité ``Product``, utilisez :
````powershell
symfony console make:entity Product
````
Symfony vous demandera si vous voulez ajouter une nouvelle propriété. Répondez par "oui" et définissez la propriété.

Par exemple, pour ajouter une propriété ``description`` de type ``string`` :

- Nom de la propriété : ``description``
- Type de la propriété : ``string``
- Taille : Vous pouvez laisser la taille par défaut ou spécifier la taille maximale (par exemple 255).

Symfony générera automatiquement le getter et le setter pour la nouvelle propriété dans la classe de l'entité.

### Modifier une propriété
Modifiez le fichier de l'entité et ajustez la propriété.

Exemple : si vous voulez changer la taille maximale de la propriété ``description`` de 255 caractères à 500 caractères, modifiez simplement l'annotation Doctrine :

````php
// src/Entity/Product.php

/**
 * @ORM\Column(type="string", length=500)
 */
private $description;
````
Après modification, vous devrez également générer une migration pour appliquer ce changement à la base de données.

### Supprimer une propriété
Pour supprimer une propriété, vous devez retirer son attribut de l'entité et également supprimer les méthodes associées (le getter et le setter) :

Ouvrez le fichier de l'entité (``Product.php`` par exemple).

Supprimez la propriété et ses annotations, par exemple pour la propriété ``description`` :

````php
// Supprimez ce code :
/**
 * @ORM\Column(type="string", length=255)
 */
private $description;

public function getDescription(): ?string
{
    return $this->description;
}

public function setDescription(string $description): self
{
    $this->description = $description;

    return $this;
}
````
Comme pour les autres modifications, générez et appliquez une migration pour supprimer la colonne correspondante dans la base de données.

### Cas particuliers : Propriétés avec contrainte de validation

Si vous voulez ajouter des contraintes de validation à une propriété (par exemple, exiger que le champ ne soit pas vide ou qu'il respecte un certain format), vous pouvez utiliser les annotations de validation Symfony dans l'entité.

Par exemple, pour exiger que la propriété ``name`` ne soit pas vide :

````php
use Symfony\Component\Validator\Constraints as Assert;

class Product
{
    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\NotBlank
     */
    private $name;
}
````
Vous pouvez ajouter diverses contraintes, comme ``@Assert\Length``, ``@Assert\Email``, ``@Assert\Range``, etc. Cela s'applique aussi bien lors de la création que de la modification de propriétés.

## 3. Les relations
Doctrine prend en charge plusieurs types de relations entre entités :

- **OneToOne** (1:1) : Une entité A est liée à une entité B, et vice-versa.
- **OneToMany** (1:n) : Une entité A peut être liée à plusieurs entités B, mais une entité B n'est liée qu'à une seule entité A.
- **ManyToOne** (n:1) : C'est l'inverse de OneToMany.
- **ManyToMany** (n:n) : Une entité A peut être liée à plusieurs entités B, et chaque entité B peut être liée à plusieurs entités A.

## 4. Manipuler les entités
### Ajouter des éléments
````php
$category = new Category();
$category->setName('Electronics');

$product = new Product();
$product->setName('Laptop');
$product->setPrice(1000);
$product->setCategory($category);

// Sauvegarder en base de données
$entityManager = $this->getDoctrine()->getManager();
$entityManager->persist($category);
$entityManager->persist($product);
$entityManager->flush();
````

### Modifier une entité
````php
$product = $this->getDoctrine()->getRepository(Product::class)->find($id);
$product->setPrice(1200);

$entityManager->flush();
````

### Supprimer une entité
````php
$entityManager->remove($product);
$entityManager->flush();
````
Cela supprimera le produit de la base de données.

# Le modèle : Accès à la DB avec Doctrine
## 1. SELECT
### Récupérer une entité par son ID
Si vous voulez récupérer une entité par son ID, utilisez la méthode ``find()`` :
````php
// Récupérer un produit avec l'ID 1
$product = $this->getDoctrine()
    ->getRepository(Product::class)
    ->find(1);
````

### Récupérer toutes les entités
Vous pouvez récupérer toutes les entités en utilisant la méthode ``findAll()`` :
````php
// Récupérer tous les produits
$products = $this->getDoctrine()
    ->getRepository(Product::class)
    ->findAll();
````

### Récupérer des entités avec des critères
Vous pouvez aussi filtrer les résultats en fonction de critères spécifiques avec ``findBy()`` :
````php
// Récupérer tous les produits dont le prix est supérieur à 100
$products = $this->getDoctrine()
    ->getRepository(Product::class)
    ->findBy(['price' => 100]);
````

### Récupérer un seul élément avec des critères
Si vous voulez récupérer un seul élément basé sur un critère spécifique, utilisez ``findOneBy()`` :
````php
// Récupérer un produit avec un nom spécifique
$product = $this->getDoctrine()
    ->getRepository(Product::class)
    ->findOneBy(['name' => 'Laptop']);
````

### Requête personnalisée avec le QueryBuilder
Pour des requêtes plus complexes, vous pouvez utiliser le ``QueryBuilder``. Par exemple, pour récupérer des produits triés par prix :
````php
$products = $this->getDoctrine()
    ->getRepository(Product::class)
    ->createQueryBuilder('p')
    ->orderBy('p.price', 'ASC')
    ->getQuery()
    ->getResult();
````

## 2. INSERT
L'insertion d'une nouvelle entité (enregistrement) dans la base de données se fait en trois étapes :

- Créer l'objet de l'entité.
- Le persister via l'EntityManager.
- Sauvegarder les changements dans la base de données avec ``flush()``.

````php
// Créer un nouvel objet Product
$product = new Product();
$product->setName('Laptop');
$product->setPrice(1000);

// Récupérer l'EntityManager
$entityManager = $this->getDoctrine()->getManager();

// Persister l'objet (préparation pour l'insertion dans la base de données)
$entityManager->persist($product);

// Sauvegarder les changements dans la base de données
$entityManager->flush();
````

## 3. UPDATE
Pour faire une mise à jour (UPDATE) dans Doctrine, vous devez d'abord récupérer l'entité que vous souhaitez modifier, effectuer les changements, puis appeler ``flush()`` pour enregistrer les modifications dans la base de données.

````php
// Récupérer l'entité existante
$product = $this->getDoctrine()
    ->getRepository(Product::class)
    ->find(1);

if ($product) {
    // Modifier la propriété
    $product->setPrice(1200);

    // Sauvegarder les modifications
    $entityManager = $this->getDoctrine()->getManager();
    $entityManager->flush();
}
````

## 4. DELETE
La suppression d'une entité se fait en deux étapes :

- Récupérer l'entité que vous voulez supprimer.
- Utiliser la méthode ``remove()`` de l'EntityManager, puis appeler ``flush()``.
````php
// Récupérer l'entité à supprimer
$product = $this->getDoctrine()
    ->getRepository(Product::class)
    ->find(1);

if ($product) {
    // Supprimer l'entité
    $entityManager = $this->getDoctrine()->getManager();
    $entityManager->remove($product);

    // Sauvegarder la suppression dans la base de données
    $entityManager->flush();
}
````

# Le modèle : Pertsistance
## 1. Concept de Persistance
La persistance consiste à rendre les objets PHP (les entités) persistants, c'est-à-dire à les enregistrer dans une base de données, afin qu'ils puissent être récupérés et manipulés ultérieurement. Doctrine fournit des outils pour mapper les objets PHP à des tables de la base de données et pour effectuer des opérations de persistance telles que :

- Insertion de nouvelles données (INSERT),
- Mise à jour de données existantes (UPDATE),
- Suppression de données (DELETE),
- Récupération de données (SELECT).

## 2. Entity Manager
L'Entity Manager de Doctrine est le composant principal responsable de la gestion de la persistance. Il assure la communication entre les objets PHP et la base de données. Voici ses principales fonctions :

- ``persist()`` : Prépare une entité pour l'insertion ou la mise à jour dans la base de données.
- ``remove()`` : Prépare une entité pour suppression.
- ``flush()`` : Exécute toutes les opérations en attente (insertion, mise à jour, suppression) dans la base de données.

## 3. Cycle de Vie de la Persistance
Voici comment fonctionne le cycle de vie de la persistance avec Doctrine dans Symfony :

**Créer une Entité** : Vous créez un objet d’une entité (ex : un produit, un utilisateur).

**Persistance via l'Entity Manager** :
- Vous utilisez la méthode ``persist()`` de l'Entity Manager pour indiquer que vous souhaitez rendre cet objet persistant.
- Ensuite, vous appelez ``flush()`` pour appliquer ces changements dans la base de données.

````php
$entityManager = $this->getDoctrine()->getManager();

$product = new Product();
$product->setName('Laptop');
$product->setPrice(1000);

$entityManager->persist($product);  // Prépare l'entité pour l'insertion
$entityManager->flush();            // Enregistre réellement dans la base de données
````
**Mise à Jour** : Lorsque vous modifiez une entité déjà persistée, Doctrine suit automatiquement les changements. Vous devez simplement appeler ``flush()`` pour les sauvegarder dans la base de données.
````php
$product = $this->getDoctrine()->getRepository(Product::class)->find(1);
$product->setPrice(1200);

$entityManager->flush();  // Doctrine met à jour l'enregistrement existant
````
**Suppression d'une Entité** : Vous utilisez la méthode ``remove()`` pour marquer une entité pour suppression, puis ``flush()`` pour exécuter l'opération dans la base de données.
````php
$product = $this->getDoctrine()->getRepository(Product::class)->find(1);

$entityManager->remove($product);  // Marque l'entité pour suppression
$entityManager->flush();           // Supprime de la base de données
````

# Le modèle : Hydrate
Pour s'épargner les ``set()`` vous pouvez créer un ``hydrate()`` pour vos entités :
````php
public function __construct(array $init = [])
{
    $this->hydrate($init); 

    // il peut avoir des lignes propres à l'entité
    $this->products = new ArrayCollection();
}

public function hydrate (array $vals = []){
    foreach ($vals as $key=> $val){
        $method = "set" . ucfirst($key); 
        if (method_exists($this,$method)){
            $this->$method ($val);
        }
    }
}
````
Vous pouvez lancer l'hydrate :
````php
use Doctrine\Persistence\ManagerRegistry;
.
.
.
    #[Route ("/exemples/modele/exemple/insert")]
    public function exempleInsert(ManagerRegistry $doctrine)
    {
        $em = $doctrine->getManager();
        // créer l'objet avec le hydrate
        $livre = new Product(
            [
                "titre"=> "Guerre et paix",
                "prix"=> 20,
                ]);
        // lier l'objet avec la BD
        $em->persist($livre);
        // écrire l'objet dans la BD
        $em->flush();

        return $this->render("exemples_modele/exemple_insert.html.twig");
    }
````

# Le modèle : Créer des fixtures
## 1. Créer des fixtures
### Installation de DoctrineFixturesBundle
Avant de créer et utiliser des fixtures, il est nécessaire d'installer le bundle ``DoctrineFixturesBundle``.
````powershell
composer require --dev orm-fixtures
````

### Créer une Fixture
Pour créer une fixture, il suffit de créer une classe qui implémente l'interface ``FixtureInterface`` ou d'étendre la classe abstraite ``Fixture`` dans le répertoire ``src/DataFixtures``.

Voici un exemple de fixture simple pour ajouter des produits à une table ``Product`` :
````php
// src/DataFixtures/ProductFixtures.php

namespace App\DataFixtures;

use App\Entity\Product;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class ProductFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        // Créer un produit
        $product = new Product();
        $product->setName('Ordinateur portable');
        $product->setPrice(1200);
        
        // Persister l'entité dans l'EntityManager
        $manager->persist($product);

        // Appliquer les changements à la base de données
        $manager->flush();
    }
}
````

### Exécuter les Fixtures
Une fois la fixture créée, vous pouvez l'exécuter avec la commande suivante pour charger les données dans la base de données :
````powershell
symfony console doctrine:fixtures:load
````

## 2. Fixtures dans ManyToMany
Exemple : Supposons que nous ayons deux entités ``Product`` et ``Category``, où un produit peut appartenir à plusieurs catégories, et une catégorie peut contenir plusieurs produits.

Entité ``Product`` :
````php
// src/Entity/Product.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\ArrayCollection;

/**
 * @ORM\Entity()
 */
class Product
{
    /**
     * @ORM\ManyToMany(targetEntity="App\Entity\Category", inversedBy="products")
     * @ORM\JoinTable(name="product_category")
     */
    private $categories;

    public function __construct()
    {
        $this->categories = new ArrayCollection();
    }

    public function addCategory(Category $category): self
    {
        if (!$this->categories->contains($category)) {
            $this->categories[] = $category;
            $category->addProduct($this);
        }

        return $this;
    }
}
````
Entité ``Category`` :
````php
// src/Entity/Category.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\ArrayCollection;

/**
 * @ORM\Entity()
 */
class Category
{
    /**
     * @ORM\ManyToMany(targetEntity="App\Entity\Product", mappedBy="categories")
     */
    private $products;

    public function __construct()
    {
        $this->products = new ArrayCollection();
    }

    public function addProduct(Product $product): self
    {
        if (!$this->products->contains($product)) {
            $this->products[] = $product;
        }

        return $this;
    }
}
````

**Fixture avec relation ManyToMany :**
````php
// src/DataFixtures/AppFixtures.php

namespace App\DataFixtures;

use App\Entity\Product;
use App\Entity\Category;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        // Créer des catégories
        $category1 = new Category();
        $category1->setName('Electronics');
        $manager->persist($category1);

        $category2 = new Category();
        $category2->setName('Computers');
        $manager->persist($category2);

        // Créer un produit
        $product = new Product();
        $product->setName('Laptop');
        $product->setPrice(1200);
        
        // Associer des catégories au produit
        $product->addCategory($category1);
        $product->addCategory($category2);

        // Persister les entités
        $manager->persist($product);
        $manager->flush();
    }
}
````

## 3. Fixtures dans OneToMany/ManyToOne
Dans une relation OneToMany, une entité (par exemple, une ``Category``) peut être liée à plusieurs autres entités (par exemple, plusieurs ``Product``), mais chaque produit appartient à une seule catégorie.

Entité ``Category`` :
````php
// src/Entity/Category.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\ArrayCollection;

/**
 * @ORM\Entity()
 */
class Category
{
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\Product", mappedBy="category")
     */
    private $products;

    public function __construct()
    {
        $this->products = new ArrayCollection();
    }

    public function addProduct(Product $product): self
    {
        if (!$this->products->contains($product)) {
            $this->products[] = $product;
            $product->setCategory($this);
        }

        return $this;
    }
}
````
Entité ``Product`` :
````php
// src/Entity/Product.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class Product
{
    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\Category", inversedBy="products")
     * @ORM\JoinColumn(nullable=false)
     */
    private $category;

    public function setCategory(?Category $category): self
    {
        $this->category = $category;

        return $this;
    }
}
````
**Fixture avec relation OneToMany :**
````php
// src/DataFixtures/AppFixtures.php

namespace App\DataFixtures;

use App\Entity\Product;
use App\Entity\Category;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        // Créer une catégorie
        $category = new Category();
        $category->setName('Electronics');
        $manager->persist($category);

        // Créer des produits
        $product1 = new Product();
        $product1->setName('Laptop');
        $product1->setPrice(1200);
        $product1->setCategory($category);
        $manager->persist($product1);

        $product2 = new Product();
        $product2->setName('Smartphone');
        $product2->setPrice(800);
        $product2->setCategory($category);
        $manager->persist($product2);

        // Appliquer les changements
        $manager->flush();
    }
}
````

**Fixture avec une relation ManyToOne**

La relation ManyToOne est simplement l'inverse de OneToMany. Chaque produit appartient à une seule catégorie, mais une catégorie peut avoir plusieurs produits.

L'exemple de relation OneToMany ci-dessus montre également un exemple de ManyToOne du point de vue de l'entité ``Product``.

## 4. Fixtures dans OneToOne
Dans une relation OneToOne, une entité est liée à une autre entité de manière unique. Par exemple, un utilisateur peut avoir une seule adresse, et une adresse peut être liée à un seul utilisateur.

Exemple d'entité ``User`` et ``Address`` :
````php
// src/Entity/User.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class User
{
    /**
     * @ORM\OneToOne(targetEntity="App\Entity\Address", cascade={"persist", "remove"})
     * @ORM\JoinColumn(nullable=false)
     */
    private $address;

    public function setAddress(Address $address): self
    {
        $this->address = $address;

        return $this;
    }
}
````
````php
// src/Entity/Address.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class Address
{
    /**
     * @ORM\OneToOne(targetEntity="App\Entity\User", mappedBy="address")
     */
    private $user;
}
````
**Fixture avec relation OneToOne :**
````php
// src/DataFixtures/AppFixtures.php

namespace App\DataFixtures;

use App\Entity\User;
use App\Entity\Address;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        // Créer une adresse
        $address = new Address();
        $address->setCity('Paris');
        $manager->persist($address);

        // Créer un utilisateur
        $user = new User();
        $user->setName('John Doe');
        $user->setAddress($address); // Associer l'adresse à l'utilisateur

        $manager->persist($user);
        $manager->flush();
    }
}
````

# Accès à la DB : DQL
## 1. SELECT
> A venir
## 2. INSERT
> A venir
## 3. UPDATE
> A venir
## 4. DELETE
> A venir

# Accès à la DB : Repository
## 1. Accès aux Repositories
Lorsque vous créez une entité dans Symfony (par exemple avec la commande ``make:entity``), un repository est automatiquement généré dans le répertoire ``src/Repository``.

Exemple de récupération d'un repository :
````php
$productRepository = $this->getDoctrine()->getRepository(Product::class);
````

## 2. Méthodes Prédéfinies dans un Repository
Doctrine vous fournit plusieurs méthodes prêtes à l'emploi pour interagir avec vos entités :

- ``find($id)``: Récupère une entité par son identifiant.
- ``findAll()``: Récupère toutes les entités de la table correspondante.
- ``findBy(array $criteria)``: Récupère les entités selon certains critères (par exemple, une colonne particulière).
- ``findOneBy(array $criteria)``: Récupère une seule entité selon certains critères.
- ``count(array $criteria)``: Compte le nombre d'entités correspondant à certains critères.

## 3. Customiser le Repository
En plus des méthodes par défaut, vous pouvez personnaliser un repository en ajoutant des méthodes supplémentaires qui implémentent une logique de récupération de données plus complexe. Pour ce faire, vous étendez simplement la classe repository générée par défaut.

Exemple :
````php
// src/Repository/ProductRepository.php

namespace App\Repository;

use App\Entity\Product;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

class ProductRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Product::class);
    }

    // Méthode personnalisée pour trouver les produits avec un prix supérieur à un certain montant
    public function findExpensiveProducts(float $price): array
    {
        return $this->createQueryBuilder('p')
            ->andWhere('p.price > :price')
            ->setParameter('price', $price)
            ->getQuery()
            ->getResult();
    }
}
````
Vous pouvez ensuite utiliser cette méthode dans un contrôleur ou un service :
````php
$expensiveProducts = $productRepository->findExpensiveProducts(1000);
````

# Accès à la DB : QueryBuilder
## 1. Création d'un QueryBuilder
Le QueryBuilder s'obtient généralement via le repository de l'entité :
````php
$qb = $this->getDoctrine()->getRepository(Product::class)->createQueryBuilder('p');
````
Dans cet exemple, ``'p'`` est un alias pour l'entité ``Product``.

## 2. Construction d'une Requête avec QueryBuilder
Le QueryBuilder fournit une API fluide pour construire des requêtes SQL. Voici un exemple de requête pour obtenir des produits avec un prix supérieur à 1000 :
````php
$qb = $this->getDoctrine()->getRepository(Product::class)->createQueryBuilder('p')
    ->where('p.price > :price')
    ->setParameter('price', 1000)
    ->orderBy('p.price', 'DESC');

$query = $qb->getQuery();
$products = $query->getResult();
````
Ici, nous utilisons les méthodes ``where()``, ``setParameter()``, et ``orderBy()`` pour construire une requête qui récupère tous les produits avec un prix supérieur à 1000, triés par prix décroissant.

## 3. Utiliser des Alias
Lorsque vous travaillez avec plusieurs entités ou relations, vous pouvez utiliser des alias dans le QueryBuilder pour référencer différentes tables.

Exemple de jointure entre deux entités :
````php
$qb = $this->getDoctrine()->getRepository(Product::class)->createQueryBuilder('p')
    ->innerJoin('p.category', 'c') // Jointure avec l'entité Category
    ->addSelect('c')               // Sélectionner les champs de la catégorie
    ->where('p.price > :price')
    ->setParameter('price', 1000)
    ->orderBy('p.price', 'DESC');

$query = $qb->getQuery();
$products = $query->getResult();
````
Ici, nous faisons une jointure interne avec la table ``Category`` (``innerJoin()``), et nous ajoutons les colonnes de ``Category`` dans la requête.

## 4. Pagination et Limitation
Vous pouvez également paginer vos résultats avec ``setFirstResult()`` et ``setMaxResults()``.

Exemple de pagination :
````php
$qb = $this->getDoctrine()->getRepository(Product::class)->createQueryBuilder('p')
    ->where('p.price > :price')
    ->setParameter('price', 1000)
    ->setFirstResult(0)    // Débuter à l'enregistrement 0
    ->setMaxResults(10);   // Limiter à 10 résultats

$query = $qb->getQuery();
$products = $query->getResult();
````

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
Symfony fournit un outil pour générer automatiquement une entité User avec certaines propriétés standard comme ``email``, ``password``, et ``roles`` via le bundle Symfony Security.

Vous pouvez créer une entité ``User`` en exécutant la commande suivante :
````powershell
symfony console make:user
````
Cela va générer une classe ``User`` avec les propriétés nécessaires. Voici un exemple de ce à quoi pourrait ressembler une entité ``User`` :
````php
// src/Entity/User.php

namespace App\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class User implements UserInterface
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=180, unique=true)
     */
    private $email;

    /**
     * @ORM\Column(type="json")
     */
    private $roles = [];

    /**
     * @ORM\Column(type="string")
     */
    private $password;

    // Getters et setters...

    public function getRoles(): array
    {
        $roles = $this->roles;
        // Assure que tous les utilisateurs ont le rôle "ROLE_USER"
        if (!in_array('ROLE_USER', $roles)) {
            $roles[] = 'ROLE_USER';
        }

        return $roles;
    }

    public function setPassword(string $password): self
    {
        $this->password = $password;

        return $this;
    }
    
    // Autres méthodes de UserInterface...
}
````

# Authentification
## 1. Inscription
Étapes de base :

- Créez un formulaire d'inscription pour collecter les informations utilisateur.
- Hash le mot de passe avec un password encoder avant de le stocker dans la base de données.
- Persister les données de l'utilisateur dans la base de données.

````php
// Controller d'inscription
public function register(Request $request, UserPasswordHasherInterface $passwordHasher, EntityManagerInterface $entityManager)
{
    $user = new User();
    $form = $this->createForm(RegistrationFormType::class, $user);
    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        // Hashage du mot de passe
        $user->setPassword(
            $passwordHasher->hashPassword(
                $user,
                $form->get('plainPassword')->getData()
            )
        );
        
        // Persister le nouvel utilisateur
        $entityManager->persist($user);
        $entityManager->flush();

        return $this->redirectToRoute('app_home');
    }

    return $this->render('registration/register.html.twig', [
        'registrationForm' => $form->createView(),
    ]);
}
````
## 2. Connexion
Symfony gère l'authentification via un firewall (pare-feu) qui contrôle les accès et permet la connexion.

Pour la connexion d’un utilisateur, Symfony utilise généralement un formulaire de connexion ou un système de token (comme JWT pour les API).

````php
// Controller de connexion
public function login(AuthenticationUtils $authenticationUtils): Response
{
    // Récupérer une erreur de connexion s'il y en a
    $error = $authenticationUtils->getLastAuthenticationError();
    
    // Dernier nom d'utilisateur saisi
    $lastUsername = $authenticationUtils->getLastUsername();

    return $this->render('security/login.html.twig', [
        'last_username' => $lastUsername,
        'error' => $error,
    ]);
}
````
Ensuite, vous devez configurer le fichier de sécurité ``security.yaml`` pour indiquer les routes de connexion/déconnexion et les règles de protection des pages.
````yaml
# config/packages/security.yaml
firewalls:
    main:
        pattern: ^/
        form_login:
            login_path: login
            check_path: login
        logout:
            path: logout
````

## 3. Déconnexion
La déconnexion est une action simple dans Symfony. Elle est configurée dans le fichier ``security.yaml`` avec la route de déconnexion.
````yaml
# config/packages/security.yaml
firewalls:
    main:
        logout:
            path: app_logout
            target: homepage
````
Ici, la route ``/logout`` est définie pour déconnecter l'utilisateur et le rediriger vers la page d'accueil (``homepage``).

## 4. Rôles
Les rôles dans Symfony permettent de définir des permissions pour chaque utilisateur. Un utilisateur peut avoir plusieurs rôles, définis dans son entité sous forme d’un tableau (``roles``).

Exemple de rôles :

- ROLE_USER : rôle de base pour tous les utilisateurs.
- ROLE_ADMIN : rôle administrateur avec des permissions étendues.
Vous pouvez contrôler l'accès à certaines routes ou actions en fonction des rôles dans le fichier security.yaml ou dans les contrôleurs.

````yaml
# config/packages/security.yaml
access_control:
    - { path: ^/admin, roles: ROLE_ADMIN }
````

Cela signifie que seules les personnes avec le rôle ``ROLE_ADMIN`` peuvent accéder aux pages dont l'URL commence par ``/admin``.

Dans les contrôleurs, vous pouvez utiliser des annotations pour restreindre l'accès :
````php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

/**
 * @IsGranted("ROLE_ADMIN")
 */
public function adminPage()
{
    // Code de la page admin
}
````

## 5. Sécurité
La sécurité est un aspect crucial lors de la gestion des utilisateurs dans Symfony. Voici quelques bonnes pratiques :

**Hashage des mots de passe :** Les mots de passe doivent toujours être stockés de manière sécurisée avec un algorithme de hashage (comme bcrypt ou Argon2i).

Exemple avec le service ``UserPasswordHasherInterface`` :
````php
$hashedPassword = $passwordHasher->hashPassword($user, $plainPassword);
````
**Contrôle d'accès :** Utilisez le fichier security.yaml pour définir les firewalls et les access_control. Vous pouvez aussi utiliser des annotations dans vos contrôleurs pour restreindre l’accès à certaines actions.

**Protection CSRF :** Assurez-vous que les formulaires de connexion et d'inscription sont protégés contre les attaques CSRF en incluant des tokens CSRF.
````php
$builder->add('_csrf_token', CsrfTokenType::class);
````
**Limitation des accès sensibles :** Utilisez les rôles pour protéger les routes sensibles et éviter qu’un utilisateur sans privilège y accède.

**Authentification à deux facteurs (2FA) :** Vous pouvez améliorer la sécurité en intégrant un système d'authentification à deux facteurs (2FA).



# Response JSON

# Suppléments
## 1. CSS
## 2. Bootstrap et Tailwind CSS
## 3. Filtre
## 4. Pagination
## 5. Mail