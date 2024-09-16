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
> ATTENTION : La version de MariaDB (MySql) doit correspondre à celle de XAMPP. Pour la connaitre, allez dans la page d'accueil de phpmyadmin

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
Utiliser la commande suivante pour créer un contrôleur :
````powershell
symfony console make:controller ExemplesTwig
````
> Cette commande génère un contrôleur de base dans le répertoire src/Controller.

## 8. Gérer les routes
Symfony gère les routes dans les fichiers de configuration config/routes/ ou directement via les annotations dans les contrôleurs. Pour ajouter une route via annotation :
````php
#[Route ('/exemples/twig/exemple1',name:'exempleTwig')]
public function exemple1 (){
    return $this->render ('exemples_twig/exemple.html.twig');
}
````

## 9. Utiliser des templates avec Twig
Symfony utilise Twig comme moteur de templates. Pour rendre une vue depuis un contrôleur :
````php
return $this->render('my_template.html.twig', [
    'variable' => $value,
]);
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

## 11. Créer des fixtures (données factices)
Pour ajouter des données de test :
````powershell
symfony composer req --dev orm-fixtures
````

Créez une classe de fixtures :
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

Chargez les fixtures dans la base de données :
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