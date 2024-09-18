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
## **1. Créer un nouveau projet Symfony**
Utiliser Symfony CLI pour créer un projet. Cela configure un environnement de base avec les dépendances nécessaires.
````powershell
symfony new --webapp MyProjet
````
> Note : --webapp crée une application avec les bundles et configurations de base pour un projet web.
> 
## **2. Naviguer dans le répertoire du projet**
````powershell
cd MyProjet
````

## **3. Configurer un serveur de développement**
````powershell
symfony server:start
````
Ou
````powershell
symfony serve
````
Pour l'arrêter, utiliser CTRL+C ou lancer
````powershell
symfony server:stop
````
Allumer XAMPP et aller dans le navigateur, via [http://localhost:8000/](http://localhost:8000/)

## **4. Configurer la base de données**
Configurer les paramètres de connexion à la base de données dans le fichier ``.env``.
````php
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=8.0"
````
> [!IMPORTANT]
> ATTENTION : La version de MariaDB (MySql) doit correspondre à celle de XAMPP. Pour la connaitre, aller dans la page d'accueil de phpmyadmin

Créer la base de données.
````powershell
symfony console doctrine:database:create
````

# Gérer les routes
Symfony gère les routes dans les fichiers de configuration config/routes/ ou directement via les annotations dans les contrôleurs. Pour ajouter une route via annotation :
````php
#[Route ('/exemples/twig/exemple1',name:'exempleTwig')]
public function exemple (){
    return $this->render ('exemples_twig/exemple.html.twig');
}
````

## **1. Route avec Paramètres**
On peut passer des paramètres dynamiques aux routes dans Symfony. Par exemple, si on a besoin de récupérer un produit par son ID, ajouter un paramètre à la route.
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
Ici, la route ``/product/{id}`` acceptera n'importe quel ID dans l'URL et le transmettra à l'action ``show()``. Le produit correspondant est alors récupéré depuis la base de données.

# Créer des contrôleurs
## **1. Créer un controller simpple**
Utiliser la commande suivante pour créer un contrôleur :
````powershell
symfony console make:controller ExemplesTwig
````
Cette commande génère un contrôleur de base dans le répertoire src/Controller avec un squelette de base.

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

## **2. Structure d'un Contrôleur**
Un contrôleur est essentiellement une méthode PHP dans une classe qui est associée à une route via des annotations ou des fichiers de configuration YAML/XML. 
### Les principales parties d'un contrôleur

- **Route** : Définit l'URL qui déclenche l'action du contrôleur. Ici, ``#[Route('/product', name: 'product')]`` indique que cette méthode est accessible à l'URL ``/product``.
- **Action** : Une méthode dans le contrôleur qui gère la logique pour une requête spécifique.
- **Réponse** : Le contrôleur retourne une instance de ``Symfony\Component\HttpFoundation\Response``, qui peut être une vue HTML, JSON, ou autre format.
- **Request** : Le contrôleur reçoit une instance de ``Symfony\Component\HttpFoundation\Request``, qui contient les données envoyées par l'utilisateur, comme les paramètres de requête, les données ``POST``, les fichiers, etc.

### Les types de réponses d'un controller
- ``render()`` : Permet d'afficher une vue.
- ``redirect()`` : Permet de rediriger le navigateur vers une autre adresse, sans ou avec de paramètres.
- ``redirectToRoute()`` : Permet de lancer une action mais en utilisant le nom (name) d'une route. On va voir des exemples dans ce chapitre
forward permet de déléguer l'action (pas d'erreur http).
- ``renderView()`` : Permet d'obtenir le rendu d'une vue sans les en-têtes HTML. C'est à dire c'est une action qui ne chargera pas une page (à différence de toutes les précédentes)... elle nous donne simplement le HTML qui correspond à la vue choisie. C'est utile si on utilise AJAX.

## **3. Utiliser des templates avec Twig**
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

## **4. Retourner une Vue (HTML)**
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

## **5. Redirection**
Parfois, on a besoin de rediriger l'utilisateur vers une autre page après une certaine action, par exemple, après avoir soumis un formulaire ou créé un enregistrement. Utiliser la méthode ``redirectToRoute()`` pour rediriger l'utilisateur vers une autre route :
````php
#[Route('/product/create', name: 'product_create')]
public function create(): Response
{
    // Logique de création de produit

    return $this->redirectToRoute('product_list');
}
````