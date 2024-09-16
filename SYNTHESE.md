# Étapes de création du projet Symfony
## 1. Créer un nouveau projet Symfony
Utiliser Symfony CLI pour créer un projet. Cela configure un environnement de base avec les dépendances nécessaires.
````powershell
symfony new --webapp MyProjet
````
> Note : --webapp crée une application avec les bundles et configurations de base pour un projet web.
> 
## 2. Naviguer dans le répertoire du projet
````powershell
cd MyProjet
````

## 3. Configurer un serveur de développement
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

## 4. Configurer la base de données
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

## 5. Créer des entités
Utiliser Doctrine pour créer une entité.
````powershell
symfony console make:entity
````
Cette commande demande de :
- Donner un nom à l'entité
- Ajouter des propriétés avec leurs types
Voici un exemple de sortie :
````powershell
 Class name of the entity to create or update (e.g. BraveChicken):
 > Product

 New property name (press <return> to stop adding fields):
 > name

 Field type (enter ? to see all types) [string]:
 > string

 Field length [255]:
 > 255
````
Cela génère une classe dans ``src/Entity/Product.php`` qui ressemble à ceci :
````php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity()]
class Product
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private $id;

    #[ORM\Column(type: 'string', length: 255)]
    private $name;

    #[ORM\Column(type: 'float')]
    private $price;

    // Getters et Setters
}
````
#### Types de colonnes Doctrine
Les types les plus courants pour les propriétés d'entité incluent :
- ``string`` : chaîne de caractères.
- ``integer`` : entier.
- ``float`` : nombre à virgule flottante.
- ``boolean`` : vrai/faux.
- ``datetime`` : date et heure.
- ``text`` : texte long.

#### Relations entre Entités
- ``OneToMany/ManyToOne`` : Une entité est liée à plusieurs autres.
- ``ManyToMany`` : Deux entités sont liées mutuellement à plusieurs autres.
  
Par exemple, une entité ``Category`` peut être liée à plusieurs ``Product`` :
````php
#[ORM\Entity()]
class Category
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private $id;

    #[ORM\Column(type: 'string', length: 255)]
    private $name;

    #[ORM\OneToMany(targetEntity: Product::class, mappedBy: 'category')]
    private $products;
}
````
Dans l'entité ``Product`` :
````php
#[ORM\Entity()]
class Product
{
    #[ORM\ManyToOne(targetEntity: Category::class, inversedBy: 'products')]
    private $category;
}
````

## 6. Créer et exécuter des migrations
Après avoir créé ou modifié des entités, générer une migration.
````powershell
symfony console make:migration
````
Appliquer ensuite la migration pour mettre à jour la **Base de Données**.
````powershell
symfony console doctrine:migration:migrate
````

## 7. Créer des contrôleurs
### a. Créer un controller simpple
Utiliser la commande suivante pour créer un contrôleur :
````powershell
symfony console make:controller ExemplesTwig
````
> Cette commande génère un contrôleur de base dans le répertoire src/Controller avec un squelette de base.

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

### b. Structure d'un Contrôleur
Un contrôleur est essentiellement une méthode PHP dans une classe qui est associée à une route via des annotations ou des fichiers de configuration YAML/XML. Voici les principales parties d'un contrôleur :

- **Route** : Définit l'URL qui déclenche l'action du contrôleur. Ici, ``#[Route('/product', name: 'product')]`` indique que cette méthode est accessible à l'URL ``/product``.
- **Action** : Une méthode dans le contrôleur qui gère la logique pour une requête spécifique.
- **Réponse** : Le contrôleur retourne une instance de ``Symfony\Component\HttpFoundation\Response``, qui peut être une vue HTML, JSON, ou autre format.

### c. Retourner une Vue (HTML)
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

### d. Redirection
Parfois, on a besoin de rediriger l'utilisateur vers une autre page après une certaine action, par exemple, après avoir soumis un formulaire ou créé un enregistrement. Utiliser la méthode redirectToRoute() pour rediriger l'utilisateur vers une autre route :
````php
#[Route('/product/create', name: 'product_create')]
public function create(): Response
{
    // Logique de création de produit

    return $this->redirectToRoute('product_list');
}
````

### e. Récupérer les données depuis la Base de Données
Voici comment récupérer des produits depuis la base de données avec le repository Doctrine.
````php
#[Route('/product/list', name: 'product_list')]
public function list(): Response
{
    $products = $this->getDoctrine()
        ->getRepository(Product::class)
        ->findAll();

    return $this->render('product/list.html.twig', [
        'products' => $products,
    ]);
}
````

### f. Créer et Persister des données
Pour créer un nouveau produit et l'enregistrer dans la base de données, utiliser le ``EntityManager`` de Doctrine pour persister l'entité et la sauvegarder.
````php
#[Route('/product/new', name: 'product_new')]
public function new(Request $request): Response
{
    $product = new Product();
    $product->setName('New Product');
    $product->setPrice(100);

    $entityManager = $this->getDoctrine()->getManager();
    $entityManager->persist($product);
    $entityManager->flush();

    return $this->redirectToRoute('product_list');
}
````

### g. Gérer les Formulaires dans un Contrôleur
Voici un exemple d'utilisation d'un formulaire pour créer un produit :
````php
#[Route('/product/new', name: 'product_new')]
public function new(Request $request): Response
{
    $product = new Product();
    $form = $this->createForm(ProductType::class, $product);

    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        $entityManager = $this->getDoctrine()->getManager();
        $entityManager->persist($product);
        $entityManager->flush();

        return $this->redirectToRoute('product_list');
    }

    return $this->render('product/new.html.twig', [
        'form' => $form->createView(),
    ]);
}
````

Dans cet exemple :

- La méthode ``createForm()`` génère un formulaire à partir de la classe ProductType.
- ``handleRequest()`` traite le formulaire lorsqu'il est soumis.
- Si le formulaire est soumis et valide, l'entité est persistée dans la base de données et l'utilisateur est redirigé.

Le template associé pour le formulaire (``product/new.html.twig``) :
````twig
{{ form_start(form) }}
    {{ form_widget(form) }}
    <button type="submit">Create Product</button>
{{ form_end(form) }}
````

## 8. Gérer les routes
Symfony gère les routes dans les fichiers de configuration config/routes/ ou directement via les annotations dans les contrôleurs. Pour ajouter une route via annotation :
````php
#[Route ('/exemples/twig/exemple1',name:'exempleTwig')]
public function exemple (){
    return $this->render ('exemples_twig/exemple.html.twig');
}
````

#### Route avec Paramètres
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

## 9. Utiliser des templates avec Twig
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
Placer les fichiers Twig dans le répertoire templates/.

## 10. Gérer les formulaires
Créer un formulaire avec la commande suivante :
````powershell
symfony console make:form
````
Dans le contrôleur, gérer l'envoi et la validation du formulaire :
````php
$form = $this->createForm(MyFormType::class, $dataObject);
$form->handleRequest($request);

if ($form->isSubmitted() && $form->isValid()) {
    // Traitement du formulaire
}

return $this->render('form_template.html.twig', [
    'form' => $form->createView(),
]);
````

### a. Créer un formulaire simple
#### Création de la classe formulaire
````powershell
symfony console make:form
````

Nommer votre formulaire et spécifier une entité si le formulaire est lié à une entité. Par exemple, pour créer un formulaire pour une entité ``Product``, générer une classe de formulaire ``ProductType``.
Cette commande générera une classe de formulaire dans ``src/Form/ProductType.php`` :
````php
namespace App\Form;

use App\Entity\Product;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\MoneyType;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'label' => 'Product Name',
            ])
            ->add('price', MoneyType::class, [
                'label' => 'Price',
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Product::class,
        ]);
    }
}
````

#### Afficher le formulaire dans le contrôleur
Une fois que la classe de formulaire est créée, utiliser le contrôleur pour gérer l'envoi et le traitement des données du formulaire.
````php
namespace App\Controller;

use App\Entity\Product;
use App\Form\ProductType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    public function new(Request $request): Response
    {
        $product = new Product();
        $form = $this->createForm(ProductType::class, $product);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // Sauvegarder le produit dans la base de données
            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($product);
            $entityManager->flush();

            return $this->redirectToRoute('product_success');
        }

        return $this->render('product/new.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}
````

#### Afficher le formulaire dans Twig
Ensuite, dans la template Twig (``product/new.html.twig``), rendre le formulaire avec une simple ligne de code :
````twig
{{ form_start(form) }}
    {{ form_widget(form) }}
    <button type="submit">Submit</button>
{{ form_end(form) }}
````
- ``form_start(form)`` génère la balise ``<form>``.
- ``form_widget(form)`` génère les champs du formulaire.
- ``form_end(form)`` génère la balise de fermeture ``</form>``.

### b. Formulaire de Recherche
#### Créer une classe de formulaire de recherche
Créer un formulaire de recherche en générant une classe de formulaire, sans l'associer à une entité :
````powershell
php bin/console make:form SearchType
````

Ensuite, dans la classe ``SearchType``, ajouter un champ pour la requête de recherche :
````php
namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\SearchType as SearchFieldType;
use Symfony\Component\OptionsResolver\OptionsResolver;

class SearchType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('query', SearchFieldType::class, [
                'label' => 'Search',
                'required' => false,
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([]);
    }
}
````

#### Afficher et gérer le formulaire de recherche dans le contrôleur
Ensuite, dans le contrôleur, gérer ce formulaire de recherche. Lorsque le formulaire est soumis, récupérer la requête de recherche et l'utiliser pour filtrer les données :
````php
namespace App\Controller;

use App\Form\SearchType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    public function search(Request $request): Response
    {
        $form = $this->createForm(SearchType::class);
        $form->handleRequest($request);

        $products = [];

        if ($form->isSubmitted() && $form->isValid()) {
            $query = $form->get('query')->getData();

            // Rechercher des produits en fonction de la requête
            $products = $this->getDoctrine()
                ->getRepository(Product::class)
                ->findByQuery($query);
        }

        return $this->render('product/search.html.twig', [
            'form' => $form->createView(),
            'products' => $products,
        ]);
    }
}
````

Dans votre template Twig (``product/search.html.twig``), afficher le formulaire et les résultats :
````twig
{{ form_start(form) }}
    {{ form_row(form.query) }}
    <button type="submit">Search</button>
{{ form_end(form) }}

{% if products is not empty %}
    <ul>
    {% for product in products %}
        <li>{{ product.name }} - {{ product.price }}</li>
    {% endfor %}
    </ul>
{% else %}
    <p>No products found.</p>
{% endif %}
````

#### Méthode dans le Repository pour la recherche
Enfin, dans le repository, créer une méthode pour exécuter la recherche. Par exemple, dans ``ProductRepository.php`` :
````php
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

    public function findByQuery(string $query): array
    {
        return $this->createQueryBuilder('p')
            ->andWhere('p.name LIKE :query OR p.description LIKE :query')
            ->setParameter('query', '%' . $query . '%')
            ->getQuery()
            ->getResult();
    }
}
````

## 11. Créer des fixtures (données factices)
Pour ajouter des données de test :
````powershell
symfony composer req --dev orm-fixtures
````

Créer une classe de fixtures :
````powershell
symfony console make:fixture
````
Cela crée une classe de fixture dans ``src/DataFixtures/``. Voici un exemple de fixture pour ajouter des produits dans la base de données :
````php
namespace App\DataFixtures;

use App\Entity\Product;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class ProductFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        for ($i = 1; $i <= 10; $i++) {
            $product = new Product();
            $product->setName('Product '.$i);
            $product->setPrice(mt_rand(10, 100));

            $manager->persist($product);
        }

        $manager->flush();
    }
}
````

Charger les fixtures dans la base de données :
````powershell
symfony console doctrine:fixtures:load --append
````

## 12. Manipulation des Données : Repository et QueryBuilder
#### Le Repository
Chaque entité a un repository associé, une classe qui contient des méthodes pour interagir avec la base de données. Par défaut, un repository simple est créé pour chaque entité dans ``src/Repository``.
Par exemple, le repository de ``Product`` pourrait ressembler à ceci :
````php
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

    // Méthodes personnalisées pour récupérer des données
}
````

#### Le QueryBuilder
Pour effectuer des requêtes complexes, utiliser le ``QueryBuilder`` :
````php
public function findExpensiveProducts(float $minPrice): array
{
    return $this->createQueryBuilder('p')
        ->andWhere('p.price > :minPrice')
        ->setParameter('minPrice', $minPrice)
        ->orderBy('p.price', 'DESC')
        ->getQuery()
        ->getResult();
}
````