## **Installation de LexikJWTBundle :**

https://github.com/lexik/LexikJWTAuthenticationBundle

> **Génération des clefs `private` et `public` sur le serveur** :
> 1) Création du dossier qui va contenir les clefs
> 
>`mkdir -p config/jwt`
>
> 2) Génération de la clef `private.pem`
>
>`openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096`
>
>
>_Saisir la `JWT passphrase` renseignée dans le fichier `.env` lorsqu'il la demande_
>
> 3) Génération de la clef `public.pem`
>
>`openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout`
>
>_Saisir la `JWT passphrase` renseignée dans le fichier `.env` lorsqu'il la demande_
>

> **Règles à ajouter dans la config d'Apache (VirtualHost ou .htaccess)** :
>```
># Règles spécifiques pour le bon fonctionnement du JWT
># Authorization  header
>RewriteEngine on
>RewriteCond %{HTTP:Authorization} ^(.*)
>RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
>```
