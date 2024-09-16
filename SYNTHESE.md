# Étapes de création du projet Symfony
## 1. Créer un nouveau projet Symfony
Utilisez Symfony CLI pour créer un projet. Cela configure un environnement de base avec les dépendances nécessaires.
````
symfony new --webapp MyProjet
````
> Note : --webapp crée une application avec les bundles et configurations de base pour un projet web.
## 2. Naviguer dans le répertoire du projet
````
cd MyProjet
````
## 3. Configurer un serveur de développement
````
symfony server:start
````
Ou
````
symfony serve
````
Pour l'arrêter, utiliser CTRL+C ou lancer
````
symfony server:stop
````
Allumer XAMPP et aller dans le navigateur, via [http://localhost:8000/](http://localhost:8000/)