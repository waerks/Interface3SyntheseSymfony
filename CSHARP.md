# Introduction au C# et à .NET
## 1. Historique et objectifs du langage C#

C# a été développé par Microsoft en 2000 comme un langage de programmation orienté objet et inspiré de C++ et Java. Il a été conçu pour être facile à apprendre et robuste, principalement pour le développement d’applications sous Windows. Depuis, C# est devenu un langage polyvalent, utilisé aussi pour le web, les jeux, les applications mobiles, et plus encore.

## 2. Introduction au framework .NET et à son écosystème

.NET est un framework développé par Microsoft, qui offre des bibliothèques, des outils et une plateforme commune pour le développement d'applications. Il supporte plusieurs langages, dont C#, et fonctionne sur différentes plateformes comme Windows, Linux, et macOS. .NET Core et, plus récemment, .NET 5+ sont des versions open-source et multiplateformes, tandis que .NET Framework est spécifique à Windows.


# Premiers Pas avec C#
## 1. Structure de base d'un programme C#

Un programme C# typique se compose de classes, de méthodes, et d'espaces de noms. Voici un exemple minimal de programme en C# :

````cs
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Bonjour, C#!");
    }
}
````

- ``using System``; : Cette ligne permet d'inclure l'espace de noms ``System``, qui contient des classes de base, y compris ``Console``, utilisée ici pour interagir avec la console.
- ``class Program`` : Ici, nous déclarons une classe nommée ``Program``. En C#, tout le code doit être organisé dans des classes (même dans un petit programme).
- ``static void Main()`` : C'est la méthode principale, ou "point d'entrée", où le programme commence à s'exécuter.

## 2. La Méthode ``Main`` et le Point d'Entrée du Programme

La méthode Main est essentielle dans chaque application C#. C'est le premier bloc de code qui s'exécute quand le programme est lancé. Voici ce que chaque partie de ``Main`` signifie :

- ``static`` : Ce mot-clé signifie que ``Main`` appartient à la classe ``Program`` elle-même et non à une instance d'objet de cette classe. Cela est nécessaire car ``Main`` est le point d'entrée du programme et doit être accessible sans avoir besoin de créer un objet de la classe ``Program``.
- ``void`` : Le type de retour ``void`` indique que la méthode ``Main`` ne renvoie aucune valeur.
- ``Main()`` : ``Main`` est un nom spécial en C# ; il doit toujours être orthographié exactement de cette façon pour que le programme puisse démarrer correctement.
Optionnellement, ``Main`` peut accepter un tableau de chaînes de caractères (``string[] args``), qui permet de passer des arguments au programme lors de son exécution.

**Exemple**
````cs
static void Main(string[] args)
{
    Console.WriteLine("Nombre d'arguments : " + args.Length);
}
````

## 3. Utilisation de ``Console.WriteLine`` et ``Console.ReadLine`` pour l'Interaction de Base

En C#, ``Console.WriteLine`` et ``Console.ReadLine`` permettent d'afficher du texte à l'écran et de lire les entrées de l'utilisateur depuis la console.

- ``Console.WriteLine`` : Cette commande affiche du texte dans la console et passe à la ligne suivante. On l’utilise souvent pour afficher des messages à l'utilisateur.

**Exemple**
````cs
Console.WriteLine("Bienvenue dans votre programme C#!");
````

- ``Console.ReadLine`` : Cette commande lit une ligne de texte que l'utilisateur entre dans la console. Elle retourne ce texte sous forme de chaîne (``string``), ce qui permet de l'utiliser ou de le stocker dans une variable.

**Exemple**
````cs
Console.WriteLine("Veuillez entrer votre nom :");
string nom = Console.ReadLine();
Console.WriteLine("Bonjour, " + nom + " !");
````

Dans cet exemple :

- ``Console.WriteLine("Veuillez entrer votre nom :");`` demande à l'utilisateur de saisir son nom.
- ``string nom = Console.ReadLine();`` lit le nom saisi par l'utilisateur et le stocke dans la variable nom.
- ``Console.WriteLine("Bonjour, " + nom + " !");`` affiche ensuite un message de bienvenue en utilisant le nom saisi.

## 4. Exemple Complet

````cs
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Bienvenue dans le programme C#!");

        Console.Write("Veuillez entrer votre prénom : ");
        string prenom = Console.ReadLine();

        Console.WriteLine("Bonjour, " + prenom + "! Comment allez-vous ?");
    }
}
````

# Variables et Types de Données
## 1. Types de base

En C#, chaque variable doit avoir un type de données, qui détermine le type de valeur que la variable peut stocker. Voici les types de base les plus utilisés :

- ``int`` (entier) : Pour stocker des nombres entiers sans décimales.

````cs
int age = 25;
````

- ``double`` (nombre à virgule flottante) : Pour stocker des nombres avec décimales. C’est un type de nombre décimal avec une précision plus élevée que ``float``.

````cs
double pi = 3.14159;
````

- ``char`` (caractère) : Pour stocker un seul caractère. Les caractères sont entourés de guillemets simples (``'A'``, ``'7'``).

````cs
char lettre = 'A';
````

- ``string`` (chaîne de caractères) : Pour stocker des séquences de caractères. Les chaînes sont entourées de guillemets doubles (``"Bonjour"``).

````cs
string nom = "Alice";
````

- ``bool`` (booléen) : Pour stocker des valeurs ``true`` ou ``false``.

````cs
bool estMajeur = true;
````

## 2. Type nullable et ``var``

- **Type nullable** : Par défaut, les types de valeur (comme ``int``, ``double``) ne peuvent pas contenir de valeur nulle (``null``). Cependant, on peut rendre un type nullable en ajoutant ``?`` après le type. Cela est utile pour indiquer qu’une variable peut ne pas avoir de valeur.

````cs
int? nombreNullable = null;  // Peut stocker un nombre ou `null`
````

- ``var`` : C# permet d'inférer le type d’une variable avec ``var``. Lorsqu’on utilise ``var``, le type est déterminé automatiquement par la valeur assignée.

````cs
var nombre = 10;           // Déduit comme `int`
var pi = 3.14;             // Déduit comme `double`
var message = "Bonjour";   // Déduit comme `string`
````
> [!IMPORTANT]
> Attention : ``var`` ne signifie pas que C# est dynamique ; il reste un langage statiquement typé. Une fois le type inféré, il ne peut pas changer.

## 3. Constantes et énumérations (``enum``)

- **Constantes** : Une constante est une valeur qui ne change jamais. En C#, les constantes sont définies avec le mot-clé ``const`` et doivent être initialisées au moment de leur déclaration.

````cs
const double GRAVITY = 9.81;
````

- Énumérations (``enum``) : Un ``enum`` est un type de données défini par l’utilisateur qui permet de nommer un ensemble de valeurs liées entre elles. Il est souvent utilisé pour des choix ou des états limités.

````cs
enum JourSemaine
{
    Lundi,
    Mardi,
    Mercredi,
    Jeudi,
    Vendredi,
    Samedi,
    Dimanche
}

// Utilisation
JourSemaine aujourdHui = JourSemaine.Lundi;
````

## 4. Conversion de types et opérateurs

### a. Conversions implicites et explicites

- **Conversion implicite** : Certaines conversions sont automatiques en C# lorsqu'elles ne risquent pas de perdre des informations (ex., ``int`` vers ``double``).

````cs
int nombre = 10;
double nombreDouble = nombre;  // Conversion implicite de `int` à `double`
````

- **Conversion explicite** : Certaines conversions nécessitent un cast, car elles peuvent entraîner une perte de données (ex., ``double`` vers ``int``).

````cs
double pi = 3.14;
int nombreEntier = (int)pi;  // Conversion explicite, `nombreEntier` est 3
````

### b. Méthodes de conversion

- ``int.Parse()`` et ``double.Parse()`` : Ces méthodes convertissent une chaîne en nombre.

````cs
string texte = "123";
int nombre = int.Parse(texte);  // nombre est 123
````

- ``Convert.ToString()`` : Convertit une valeur en chaîne de caractères.

````cs
int age = 25;
string ageTexte = Convert.ToString(age);  // ageTexte est "25"
````

- ``TryParse()`` : Cette méthode est plus sûre que ``Parse()`` car elle retourne false en cas d'erreur, au lieu de lever une exception. C'est utile pour gérer des entrées non valides.

````cs
string texte = "abc";
int nombre;

if (int.TryParse(texte, out nombre))
{
    Console.WriteLine("Conversion réussie : " + nombre);
}
else
{
    Console.WriteLine("Conversion échouée");
}
````

## 5. Exemple Complet

````cs
using System;

class Program
{
    enum Couleur
    {
        Rouge,
        Vert,
        Bleu
    }

    static void Main()
    {
        // Types de base
        int age = 30;
        double salaire = 55000.75;
        bool estEmploye = true;
        char initiale = 'A';
        string nom = "Alice";

        // Constante
        const double TAUX_TVA = 0.20;

        // Énumération
        Couleur couleurPreferee = Couleur.Bleu;

        // Affichage des valeurs
        Console.WriteLine("Nom : " + nom);
        Console.WriteLine("Age : " + age);
        Console.WriteLine("Salaire : " + salaire);
        Console.WriteLine("Est employé : " + estEmploye);
        Console.WriteLine("Initiale : " + initiale);
        Console.WriteLine("Couleur préférée : " + couleurPreferee);

        // Conversion
        string ageTexte = age.ToString();
        Console.WriteLine("Age (en texte) : " + ageTexte);

        string input = "50";
        if (int.TryParse(input, out int nombreConverti))
        {
            Console.WriteLine("Nombre converti : " + nombreConverti);
        }
        else
        {
            Console.WriteLine("Conversion échouée");
        }
    }
}
````


# Structures de Contrôle
## 1. Conditions : ``if``, ``else if``, ``else``, ``switch``

### a. ``if``, ``else if``, ``else``
- ``if`` : Utilisé pour tester une condition. Si elle est vraie, le bloc de code associé est exécuté.
- ``else if`` : Utilisé pour tester une condition différente si la première est fausse.
- ``else`` : Définit le code à exécuter si toutes les conditions précédentes sont fausses.

**Exemple**

````cs
int age = 20;

if (age >= 18)
{
    Console.WriteLine("Vous êtes majeur.");
}
else if (age > 12)
{
    Console.WriteLine("Vous êtes adolescent.");
}
else
{
    Console.WriteLine("Vous êtes enfant.");
}
````

### b. ``switch``
``switch`` est utilisé pour tester une variable contre plusieurs valeurs. Il est particulièrement utile pour des choix multiples.

````cs
int jour = 3;

switch (jour)
{
    case 1:
        Console.WriteLine("Lundi");
        break;
    case 2:
        Console.WriteLine("Mardi");
        break;
    case 3:
        Console.WriteLine("Mercredi");
        break;
    default:
        Console.WriteLine("Jour inconnu");
        break;
}
````

## 2. Boucles : ``for``, ``while``, ``do-while``, ``foreach``

### a. ``for``
``for`` est utilisé lorsque le nombre d’itérations est connu à l’avance.

**Exemple**

````cs
for (int i = 0; i < 5; i++)
{
    Console.WriteLine("Itération : " + i);
}
````

### b. ``while``
``while`` est utilisé lorsque le nombre d’itérations n'est pas connu à l'avance. La boucle continue tant que la condition est vraie.

**Exemple**

````cs
int compteur = 0;

while (compteur < 5)
{
    Console.WriteLine("Compteur : " + compteur);
    compteur++;
}
````

### c. ``do-while``
``do-while`` est similaire à while, mais la condition est vérifiée après chaque itération. La boucle est donc exécutée au moins une fois, même si la condition est fausse dès le départ.

**Exemple**

````cs
int compteur = 0;

do
{
    Console.WriteLine("Compteur : " + compteur);
    compteur++;
} while (compteur < 5);
````

### d. ``foreach``
``foreach`` est utilisé pour parcourir les éléments d’une collection (comme un tableau ou une liste) sans se soucier de l’indice.

**Exemple**

````cs
string[] jours = { "Lundi", "Mardi", "Mercredi" };

foreach (string jour in jours)
{
    Console.WriteLine(jour);
}
````

## 3. Manipulation de chaînes et formats : ``string.Format`` et interpolation de chaînes

### a. ``string.Format``
``string.Format`` permet de formater une chaîne de caractères en combinant du texte et des valeurs de variables. Les emplacements ``{0}``, ``{1}``, etc., servent de positions pour les valeurs.

**Exemple**

````cs
string nom = "Alice";
int age = 30;

string message = string.Format("Nom : {0}, Age : {1}", nom, age);
Console.WriteLine(message);
````

### b. Interpolation de chaînes
Depuis C# 6, on peut utiliser l'interpolation de chaînes en préfixant la chaîne avec ``$``. Cela rend le code plus lisible en permettant d'insérer des expressions directement dans les accolades.

**Exemple**

````cs
string nom = "Alice";
int age = 30;

string message = $"Nom : {nom}, Age : {age}";
Console.WriteLine(message);
````

## 4. Exemple Complet

````cs
using System;

class Program
{
    static void Main()
    {
        // Conditions
        int heure = DateTime.Now.Hour;

        if (heure < 12)
        {
            Console.WriteLine("Bonjour !");
        }
        else if (heure < 18)
        {
            Console.WriteLine("Bon après-midi !");
        }
        else
        {
            Console.WriteLine("Bonsoir !");
        }

        // Boucle `for`
        Console.WriteLine("Comptez jusqu'à 5 :");
        for (int i = 1; i <= 5; i++)
        {
            Console.WriteLine(i);
        }

        // Boucle `foreach` et manipulation de chaînes
        string[] prenoms = { "Alice", "Bob", "Charlie" };
        foreach (string prenom in prenoms)
        {
            Console.WriteLine($"Bonjour, {prenom} !");
        }
    }
}
````


# Fonctions (Méthodes)
## 1. Déclaration et utilisation des méthodes

Une méthode se déclare avec un nom, des paramètres (facultatifs), un type de retour (ou ``void`` si elle ne renvoie rien) et un bloc de code. 
<br>Voici la syntaxe générale :

````cs
[modificateur] TypeRetour NomMethode([Type Param1, Type Param2, ...])
{
    // Code de la méthode
}
````

- **Modificateur** : Définit la portée d’accès (public, private, protected, internal).
- **TypeRetour** : Le type de donnée que la méthode renvoie (ou void si elle ne renvoie rien).
- **NomMethode** : Le nom de la méthode.
- **Paramètres** : Liste des variables que la méthode peut recevoir en entrée.

**Exemple**

````cs
public int Ajouter(int a, int b)
{
    return a + b;
}
````

Ici, ``Ajouter`` est une méthode qui prend deux entiers en paramètres (``a`` et ``b``) et retourne leur somme.
<br>Elle peut être appelée comme ceci :

````cs
int resultat = Ajouter(5, 3);  // resultat vaut 8
````

## 2. Paramètres et valeurs de retour

Les paramètres permettent de fournir des données d'entrée à une méthode, et le type de retour permet de spécifier la valeur que la méthode renvoie une fois l’exécution terminée.

- **Paramètres d'entrée** : On peut passer des valeurs à la méthode au moment de l’appel. Les paramètres sont définis avec leur type et leur nom.
- **Valeur de retour** : Spécifie ce que la méthode renvoie. Si elle ne renvoie rien, on utilise ``void`` comme type de retour.

**Exemple**

````cs
public string Bonjour(string prenom)
{
    return $"Bonjour, {prenom}!";
}
````

Ici, la méthode ``Bonjour`` prend un paramètre de type ``string``, et retourne un message personnalisé avec le prénom fourni.
<br>Utilisation :

````cs
string message = Bonjour("Alice");  // message vaut "Bonjour, Alice!"
````

## 3. Surcharge de méthodes et paramètres optionnels

### a. Surcharge de méthodes (overloading)
La surcharge de méthode permet de créer plusieurs versions d'une même méthode avec des signatures différentes (différent nombre ou types de paramètres).

**Exemple**

````cs
public int Multiplier(int a, int b)
{
    return a * b;
}

public double Multiplier(double a, double b)
{
    return a * b;
}
````

Ici, ``Multiplier`` est surchargée : une version accepte des entiers, l’autre des ``double``. Lors de l'appel, C# choisit automatiquement la version qui correspond aux types des arguments.

### b. Paramètres optionnels
Les paramètres optionnels permettent de fournir une valeur par défaut pour certains paramètres. Cela signifie que l'appelant peut omettre ces arguments lors de l'appel de la méthode.

**Exemple**

````cs
public void Saluer(string prenom, string message = "Bienvenue")
{
    Console.WriteLine($"{message}, {prenom}!");
}
````

Ici, ``message`` est un paramètre optionnel avec la valeur par défaut ``"Bienvenue"``. Si ``message`` n'est pas fourni lors de l'appel, ``"Bienvenue"`` sera utilisé.

````cs
Saluer("Alice");                    // Affiche "Bienvenue, Alice!"
Saluer("Alice", "Bonjour");         // Affiche "Bonjour, Alice!"
````

## 4. Méthodes static vs méthodes d'instance

### a. Méthodes d'instance
Les méthodes d'instance sont associées à une instance (ou objet) spécifique de la classe. Elles accèdent aux données et méthodes de l’instance (à travers le mot-clé ``this``). Pour utiliser une méthode d'instance, il faut d'abord créer un objet de la classe.

**Exemple**

````cs
public class Personne
{
    public string Nom { get; set; }

    public void SePresenter()
    {
        Console.WriteLine($"Bonjour, je m'appelle {Nom}");
    }
}

// Utilisation
Personne p = new Personne();
p.Nom = "Alice";
p.SePresenter();  // Affiche "Bonjour, je m'appelle Alice"
````

### b. Méthodes ``static``
Les méthodes ``static`` appartiennent à la classe elle-même, et non à une instance. Elles ne peuvent pas accéder aux membres d'instance (car elles n’ont pas de référence ``this``). Les méthodes ``static`` sont utiles pour des actions qui ne dépendent pas d’une instance, comme des utilitaires.

**Exemple**

````cs
public class Calculatrice
{
    public static int Additionner(int a, int b)
    {
        return a + b;
    }
}

// Utilisation
int resultat = Calculatrice.Additionner(5, 10);  // resultat vaut 15
````

## 5. Exemple Complet

````cs
using System;

public class Personne
{
    public string Nom { get; set; }

    // Méthode d'instance
    public void SePresenter(string message = "Enchanté")
    {
        Console.WriteLine($"{message}, je suis {Nom}.");
    }

    // Méthode surchargée
    public void SePresenter(int age)
    {
        Console.WriteLine($"Je m'appelle {Nom} et j'ai {age} ans.");
    }

    // Méthode `static`
    public static void SaluerToutLeMonde()
    {
        Console.WriteLine("Bonjour tout le monde !");
    }
}

public class Program
{
    public static void Main()
    {
        // Appel de la méthode `static`
        Personne.SaluerToutLeMonde();

        // Création d'une instance de Personne
        Personne p = new Personne { Nom = "Alice" };

        // Appels de méthodes d'instance
        p.SePresenter();                     // Affiche "Enchanté, je suis Alice."
        p.SePresenter("Salut");              // Affiche "Salut, je suis Alice."
        
        // Surcharge
        p.SePresenter(25);                   // Affiche "Je m'appelle Alice et j'ai 25 ans."
    }
}
````

# Programmation Orientée Objet (POO)
## 1. Concepts de base : Classe, Objet, Encapsulation

### a. Classe

Une classe est un modèle ou un plan qui définit les propriétés et le comportement d’un type d’objet. Elle décrit ce que chaque objet du type peut faire et quelles informations il peut contenir. En C#, une classe se définit par le mot-clé ``class``.

**Exemple**

````cs
public class Voiture
{
    public string Marque;
    public string Modele;
    public int Annee;

    public void AfficherDetails()
    {
        Console.WriteLine($"Marque : {Marque}, Modèle : {Modele}, Année : {Annee}");
    }
}
````

### b. Objet
Un objet est une instance d'une classe, représentant une entité réelle. Chaque objet a ses propres valeurs pour les champs définis dans la classe.

**Exemple**

````cs
Voiture voiture1 = new Voiture();
voiture1.Marque = "Toyota";
voiture1.Modele = "Corolla";
voiture1.Annee = 2020;
voiture1.AfficherDetails();  // Affiche : Marque : Toyota, Modèle : Corolla, Année : 2020
````

### c. Encapsulation
L'encapsulation consiste à restreindre l’accès aux données d’un objet en rendant certains champs ou méthodes privés, c'est-à-dire accessibles uniquement depuis la classe elle-même. En C#, cela se fait avec des modificateurs d’accès comme ``public``, ``private``, ``protected``, etc.

**Exemple**

````cs
public class Voiture
{
    private string marque;

    public string Marque
    {
        get { return marque; }
        set { marque = value; }
    }
}
````

Ici, ``marque`` est privée et ne peut être modifiée qu’avec la propriété ``Marque``, qui contrôle l’accès en lecture et en écriture.

## 2. Membres d’une classe : Propriétés, Champs, Méthodes

### a. Champs
Les champs (ou variables d’instance) sont des données ou variables déclarées dans une classe. Ils représentent l'état ou les caractéristiques d'un objet.

````cs
public class Personne
{
    public string Nom;
    public int Age;
}
````

### b. Propriétés
Les propriétés encapsulent les champs, permettant d'accéder et de modifier les données tout en gardant le contrôle sur l'accès.

````cs
public class Personne
{
    private string nom;
    public string Nom
    {
        get { return nom; }
        set { nom = value; }
    }
}
````

### c. Méthodes
Les méthodes sont des fonctions définies dans une classe, décrivant les actions qu'un objet peut réaliser.

## 3. Constructeurs et Destructeurs

### a. Constructeurs
Les constructeurs sont des méthodes spéciales utilisées pour initialiser un nouvel objet. Ils portent le même nom que la classe et n’ont pas de type de retour.

````cs
public class Voiture
{
    public string Marque;
    public string Modele;

    // Constructeur
    public Voiture(string marque, string modele)
    {
        Marque = marque;
        Modele = modele;
    }
}
````

Utilisation du constructeur :

````cs
Voiture voiture1 = new Voiture("Toyota", "Corolla");
````

### b. Destructeurs
Les destructeurs sont utilisés pour libérer les ressources lorsqu'un objet est supprimé. En C#, les destructeurs sont rarement nécessaires, car le ramasse-miettes (garbage collector) gère la libération de la mémoire automatiquement.
````cs
~Voiture()
{
    // Code pour nettoyer les ressources
}
````

## 4. Héritage et Polymorphisme
### a. Héritage
L'héritage permet de créer une nouvelle classe qui reprend les propriétés et méthodes d'une classe existante. En C#, une classe peut hériter d'une seule autre classe.

````cs
public class Vehicule
{
    public int Vitesse { get; set; }
    public void Rouler() => Console.WriteLine("Le véhicule roule.");
}

public class Voiture : Vehicule
{
    public string Modele { get; set; }
}
````

Ici, ``Voiture`` hérite de ``Vehicule``, et possède donc la propriété ``Vitesse`` et la méthode ``Rouler``.

### b. Polymorphisme
Le polymorphisme permet d’utiliser une méthode de différentes manières selon l'objet. En C#, le polymorphisme se réalise souvent avec l’héritage et les méthodes virtuelles.

````cs
public class Vehicule
{
    public virtual void Rouler()
    {
        Console.WriteLine("Le véhicule roule.");
    }
}

public class Voiture : Vehicule
{
    public override void Rouler()
    {
        Console.WriteLine("La voiture roule.");
    }
}
````

En utilisant ``override``, ``Voiture`` peut remplacer le comportement de ``Rouler`` de ``Vehicule``.

## 5. Interfaces et Classes Abstraites

### a. Interfaces
Une interface définit un ensemble de méthodes et propriétés sans implémentation. Les classes qui implémentent l'interface doivent fournir une implémentation pour ces membres.

````cs
public interface IVehicule
{
    void Demarrer();
}

public class Voiture : IVehicule
{
    public void Demarrer()
    {
        Console.WriteLine("La voiture démarre.");
    }
}
````

``Voiture`` doit implémenter la méthode ``Demarrer``.

### b. Classes Abstraites
Une classe abstraite est une classe partiellement implémentée qui ne peut pas être instanciée. Elle peut contenir des méthodes abstraites (sans corps) que les classes dérivées doivent implémenter.

````cs
public abstract class Animal
{
    public abstract void Crier();
}

public class Chien : Animal
{
    public override void Crier()
    {
        Console.WriteLine("Le chien aboie.");
    }
}
````

Ici, ``Animal`` est abstrait et ``Chien`` fournit une implémentation pour ``Crier``.

## 6. Le Mot-clé this

Le mot-clé ``this`` fait référence à l'instance actuelle de la classe. Il est souvent utilisé pour désambiguïser entre les champs de la classe et les paramètres de la méthode ayant le même nom.

**Exemple**

````cs
public class Voiture
{
    private string marque;

    public Voiture(string marque)
    {
        this.marque = marque; // `this.marque` fait référence au champ, `marque` au paramètre
    }
}
````

Ici, ``this.marque`` fait référence au champ ``marque`` de l’instance de la classe, alors que ``marque`` se réfère au paramètre du constructeur.

## 7. Exemple Complet

````cs
using System;

public abstract class Animal
{
    public string Nom { get; set; }
    public abstract void Crier();
}

public class Chien : Animal
{
    public override void Crier()
    {
        Console.WriteLine($"{Nom} aboie !");
    }
}

public class Chat : Animal
{
    public override void Crier()
    {
        Console.WriteLine($"{Nom} miaule !");
    }
}

public interface IVehicule
{
    void Demarrer();
}

public class Voiture : IVehicule
{
    public void Demarrer()
    {
        Console.WriteLine("La voiture démarre.");
    }
}

class Program
{
    static void Main()
    {
        Animal chien = new Chien { Nom = "Rex" };
        chien.Crier(); // Affiche : "Rex aboie !"

        Animal chat = new Chat { Nom = "Luna" };
        chat.Crier(); // Affiche : "Luna miaule !"

        IVehicule voiture = new Voiture();
        voiture.Demarrer(); // Affiche : "La voiture démarre."
    }
}
````


# Types Avancés et Collections
## 1. Tableaux (``array``)
Les tableaux en C# sont des structures de données qui permettent de stocker plusieurs éléments du même type dans une séquence. Ils sont de taille fixe, c'est-à-dire que leur taille est déterminée lors de leur création et ne peut pas être modifiée par la suite.

### a. Déclaration et initialisation d'un tableau :

````cs
int[] nombres = new int[5];  // Déclare un tableau de 5 entiers
nombres[0] = 10;             // Initialise la première valeur du tableau
nombres[1] = 20;             // Initialise la deuxième valeur du tableau
````

- Un tableau en C# peut contenir des éléments de n'importe quel type (entiers, chaînes de caractères, objets, etc.).
- Les indices des tableaux commencent toujours à 0.

**Exemple de tableau multidimensionnel (matrice)**

Un tableau multidimensionnel peut être utilisé pour des structures comme des matrices.

````cs
int[,] matrice = new int[2, 3];  // Tableau à 2 lignes et 3 colonnes
matrice[0, 0] = 1;
matrice[0, 1] = 2;
matrice[0, 2] = 3;
matrice[1, 0] = 4;
matrice[1, 1] = 5;
matrice[1, 2] = 6;
````

### b. Limitations des tableaux :
La taille des tableaux est fixe après leur création. Si vous avez besoin d'une taille dynamique, des collections comme ``List<T>`` ou ``ArrayList`` seront plus adaptées.

## 2. Listes (List<T>) et autres collections génériques
Les collections génériques en C# offrent une plus grande flexibilité que les tableaux, notamment en permettant des tailles dynamiques et en fournissant des méthodes utiles pour ajouter, supprimer ou trier les éléments.

### a. Listes (``List<T>``)
Une ``List<T>`` est une collection générique qui peut contenir des éléments de n'importe quel type spécifié (``T``). Contrairement aux tableaux, une liste peut croître ou se rétrécir en fonction des besoins.

**Exemple**

````cs
List<int> nombres = new List<int>();  // Crée une liste vide de types entiers
nombres.Add(10);                      // Ajoute un élément
nombres.Add(20);
nombres.Add(30);

Console.WriteLine(nombres[1]);        // Affiche 20
nombres.Remove(20);                   // Supprime l'élément 20
````

**Méthodes courantes**

- ``Add(T item)``: Ajoute un élément à la fin de la liste.
- ``Remove(T item)``: Supprime le premier élément qui correspond à l'élément spécifié.
- ``Count``: Obtient le nombre d'éléments dans la liste.
- ``Sort()``: Trie les éléments de la liste.

### b. Dictionnaires (``Dictionary<TKey, TValue>``)
Un ``Dictionary<TKey, TValue>`` est une collection générique qui associe une clé (``TKey``) à une valeur (``TValue``). Chaque clé est unique et permet d'accéder rapidement à la valeur associée.

**Exemple**

````cs
Dictionary<string, int> ages = new Dictionary<string, int>();
ages.Add("Alice", 30);
ages.Add("Bob", 25);

Console.WriteLine(ages["Alice"]);  // Affiche 30
````

**Méthodes courantes**
- ``Add(TKey key, TValue value)``: Ajoute une nouvelle paire clé-valeur.
- ``ContainsKey(TKey key)``: Vérifie si une clé existe dans le dictionnaire.
- ``Remove(TKey key)``: Supprime une entrée par clé.
- ``Keys``: Obtient toutes les clés du dictionnaire.
- ``Values``: Obtient toutes les valeurs du dictionnaire.

**Autres collections génériques**
- ``Queue<T>`` : Représente une file d'attente (FIFO). Les éléments sont ajoutés à la fin et retirés du début.
- ``Stack<T>`` : Représente une pile (LIFO). Les éléments sont ajoutés et retirés du haut de la pile.

**Exemple de Queue**

````cs
Queue<string> queue = new Queue<string>();
queue.Enqueue("A");
queue.Enqueue("B");
queue.Enqueue("C");

Console.WriteLine(queue.Dequeue());  // Affiche "A"
````

**Exemple de Stack**

````cs
Stack<string> stack = new Stack<string>();
stack.Push("X");
stack.Push("Y");
stack.Push("Z");

Console.WriteLine(stack.Pop());  // Affiche "Z"
````

## 3. Collections non génériques
Les collections non génériques étaient utilisées avant l'introduction des collections génériques en C#. Elles offrent moins de sécurité de type et sont moins efficaces que les collections génériques. Cependant, elles peuvent encore être utilisées dans certains cas où une flexibilité de type est requise.

### a. ``ArrayList``
Un ``ArrayList`` est similaire à une ``List<T>``, mais il ne spécifie pas de type pour les éléments, ce qui permet de stocker des objets de types différents. Cela peut poser des risques en termes de performance et de sécurité des types.

**Exemple**

````cs
ArrayList list = new ArrayList();
list.Add(10);
list.Add("Hello");
list.Add(true);

Console.WriteLine(list[0]);  // Affiche 10
Console.WriteLine(list[1]);  // Affiche "Hello"
````

Les éléments dans un ``ArrayList`` peuvent être de n'importe quel type, mais vous devrez effectuer des conversions (cast) lorsque vous les récupérez.

``Hashtable``
Un ``Hashtable`` est similaire à un ``Dictionary<TKey, TValue>``, mais contrairement au Dictionary, il peut contenir des clés et des valeurs de n'importe quel type. Il est également moins performant en raison de la gestion des types non spécifiés.

**Exemple**

````cs
Hashtable hashtable = new Hashtable();
hashtable.Add("clé1", "valeur1");
hashtable.Add("clé2", 123);

Console.WriteLine(hashtable["clé1"]);  // Affiche "valeur1"
Console.WriteLine(hashtable["clé2"]);  // Affiche 123
````

Comme avec l'``ArrayList``, les ``Hashtable`` sont moins sûrs que les versions génériques comme ``Dictionary``.

## 4. Types de collections immuables
Les collections immuables ne peuvent pas être modifiées après leur création. En C#, ces collections sont souvent utilisées pour des raisons de sécurité et de performance, en particulier dans des environnements où l'état des données doit être garanti comme constant.

**Exemple de collection immuable**

````cs
using System.Collections.Immutable;

var liste = ImmutableList<int>.Empty
    .Add(1)
    .Add(2)
    .Add(3);

Console.WriteLine(liste[0]);  // Affiche 1
````

- ``ImmutableList<T>`` : Une version immuable de ``List<T>``. Une fois une valeur ajoutée, la liste ne peut plus être modifiée.
- ``ImmutableDictionary<TKey, TValue>`` : Une version immuable de ``Dictionary<TKey, TValue>``.
- ``ImmutableArray<T>`` : Une version immuable de ``Array``.

Les collections immuables sont particulièrement utiles dans des scénarios multithreads, car elles garantissent que les données ne changent pas pendant que plusieurs threads les utilisent.

## 5. Résumé des collections en C#

| Type de collection  |  Description |  Utilisation |
|:--|:--|:--|
| ``Array``  | Tableau de taille fixe.  | Utilisé pour des ensembles de données de taille connue.  |
| ``List<T>``  |  Liste dynamique avec des méthodes pratiques. |  Utilisé pour des données dynamiques avec des types spécifiques. |
| ``Dictionary<TKey, TValue>``  | Table de hachage associant des clés à des valeurs.  | Utilisé pour rechercher rapidement des éléments via une clé unique.  |
| ``Queue<T>``  | File d'attente (FIFO).  | Utilisé pour les files d'attente.  |
| ``Stack<T>``  | Pile (LIFO).  | Utilisé pour les piles.  |
| ``ArrayList``  | Liste dynamique non typée.  | Utilisé lorsque vous avez besoin de flexibilité avec les types.  |
| ``Hashtable``  | Table de hachage non typée.  | Utilisé pour stocker des paires clé-valeur sans type spécifique.  |
| ``ImmutableList<T>``  | Liste immuable.  | Utilisé lorsque vous avez besoin de garantir que la collection ne changera pas.  |

		
# Gestion des Erreurs et Exceptions
## 1. Blocs ``try-catch-finally``
Le mécanisme principal pour gérer les erreurs en C# est l'utilisation des blocs try-catch-finally. Ce mécanisme permet de capturer et de traiter les erreurs tout en continuant l'exécution du programme de manière contrôlée.

**Structure générale**

````cs
try
{
    // Code susceptible de générer une exception
}
catch (ExceptionType1 ex)
{
    // Code pour traiter l'exception de type ExceptionType1
}
catch (ExceptionType2 ex)
{
    // Code pour traiter l'exception de type ExceptionType2
}
finally
{
    // Code qui s'exécute toujours, qu'il y ait eu une exception ou non
}
````

- ``try`` : Le code qui pourrait générer une exception est placé dans ce bloc.
- ``catch`` : Si une exception est lancée dans le bloc ``try``, le contrôle passe dans le bloc ``catch`` où vous pouvez traiter l'erreur.
- ``finally`` : Ce bloc est toujours exécuté, qu'une exception soit levée ou non. Il est souvent utilisé pour libérer des ressources (fermeture de fichiers, déconnexion de bases de données, etc.).

**Exemple**

````cs
try
{
    int a = 10;
    int b = 0;
    int resultat = a / b;  // Cette ligne provoque une exception
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("Erreur: Division par zéro!");
}
finally
{
    Console.WriteLine("Bloc finally toujours exécuté.");
}
````

Dans cet exemple, une ``DivideByZeroException`` est levée dans le bloc ``try``, et le contrôle passe dans le bloc ``catch`` où l'erreur est traitée. Le bloc ``finally`` est exécuté à la fin, même si une exception a été capturée.

## 2. Création et Levée d'Exceptions (``throw``)

C# permet de lever (ou "jeter") des exceptions de manière explicite en utilisant le mot-clé ``throw``. Cela permet de signaler qu'un problème s'est produit et de transmettre des informations sur l'erreur, que ce soit à l'intérieur d'un programme ou pour interagir avec un appelant (comme une fonction principale).

**Syntaxe de ``throw``**

````cs
throw new Exception("Message d'erreur");
````

**Exemple**
````cs
public void Diviser(int a, int b)
{
    if (b == 0)
    {
        throw new DivideByZeroException("Erreur: Impossible de diviser par zéro.");
    }
    Console.WriteLine(a / b);
}
````

Dans cet exemple, si ``b`` est égal à zéro, une ``DivideByZeroException`` est levée avec un message personnalisé.

**Types d'exceptions courantes**

- ``ArgumentNullException`` : Levée lorsqu'un argument de méthode est nul alors qu'il ne devrait pas l'être.
- ``DivideByZeroException`` : Levée lorsqu'une division par zéro est tentée.
- ``FileNotFoundException`` : Levée lorsqu'un fichier spécifié est introuvable.
- ``IndexOutOfRangeException`` : Levée lorsqu'un indice hors des limites d'un tableau ou d'une collection est utilisé.

Vous pouvez aussi créer des exceptions personnalisées en créant une classe qui hérite de la classe de base ``Exception``.