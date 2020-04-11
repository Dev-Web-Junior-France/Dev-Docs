# **CREATION D'UN NOUVEAU PROJET SYMFONY**

## **ETAPE 1 :**

### **Création du nouveau Projet :**

https://symfony.com/doc/current/setup.html _(Lien vers la documentation officielle)_

**Initialisation d'un projet complet :**

> `symfony new projet --dir='.'`

- _`projet='nom du projet'`_
- _`--dir='.'` (signifie qu'on créé le nouveau projet dans le dossier courant si on précise pas il créé un nouveau dossier portant le nom du projet dans le dossier courant depuis lequel est lancé la commande)_
- _`--version=lts` (pour créer un projet sur la version LTS)_
- _`--version=4.4` (pour créer un projet sur la version dont le numéro est précisé)_

**Relier le projet à un repository github précédement créé et faire le premier commit :**

> `git remote add origin git@github.com:account/projet.git`

- _`project='nom du projet'`_
- _`account='nom du compte github'`_

> _Pour ajouter tout le dossier courant au versionning GIT :_
> 
> `git add .`

> _Créer le premier commit du projet :_
> 
> `git commit -m "First Commit"` 

> _Envoyer le premier commit sur la branche master du repo github relié :_
> 
> `git push --set-upstream origin master`

**Créer le fichier .env.local (qui ne sera pas versionné sur GIT car contient les infos d'accès à la base de données) :**

> `symfoenv`
- _Saisir le nom de la nouvelle base de données à créer_
- _Le fichier .env.local est alors créé avec les paramètres par défaut_
- _Modifier ce fichier si nécessaire pour les paramètres de connexion à la base de données_

**Installer l'ORM Doctrine :**

> `composer require symfony/orm-pack`
> 
**Créer la base de données dont le nom et les paramètres ont été précisés à l'étape précèdente :**

> `bin/console doctrine:database:create`
> 
> _Si la base de données n'existe pas elle sera créee sinon un message d'erreur stipulera qu'elle existe déjà_
> 
**Installation du MakerBundle (outil qui permet de créer les Controllers/Entities/User/CRUD etc...**

> `composer require symfony/maker-bundle --dev`
> 
> _Disponible en environnement de DEV uniquement_

**Installation du composant 'security' qui s'occupe de la sécurisation de l'App**

> `composer require security`

## **ETAPE 2 :**

### **Mise en place de la partie qui gère les utilisateurs et authentification :**



### **Création des entitées (qui correspondent aux tables de la base de données) :**
### **Création des entitées (qui correspondent aux tables de la base de données) :**


