# **Installation de base du serveur :**

## **Création d'un espace de swap :**

> **Pour vérifier si il existe un espace de swap :**

`sudo swapon --show`

> **Création d'une partition swap de 2G :**

`sudo fallocate -l 2G /swapfile`

`sudo chmod 600 /swapfile`

`sudo mkswap /swapfile`

`sudo swapon /swapfile`

## **Installation LAMP Stack :**

### **Installation Apache Web Server :**

`sudo apt-get install apache2 apache2-doc`

#### Ajout de la gestion HTTPS :

https://linuxhint.com/setup_free_ssl_cert_apache_debian/

`sudo apt install certbot python-certbot-apache ca-certificates apt-transport-https -y`

`sudo certbot --apache`

```
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/api.geekoz.fr/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/api.geekoz.fr/privkey.pem
   Your cert will expire on 2020-04-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
```

> **Pour renouveller tous les certificats :**

`certbot renew`

#### Modification des droits sur dossier /var/www/ :

`sudo chown -R $USER:www-data /var/www/ozone`

`sudo chmod -R g+rw /var/www/ozone`

#### Dossier de création des VirtualHosts :

> **Par defaut le site existant est :**

`sudo nano /etc/apache2/sites-available/000-default.conf`

> **On peut créer des nouveaux fichiers pour chacun de nos sites hébergés :**

`sudo nano /etc/apache2/sites-available/site-exemple.conf`

> **Exemple de contenu du fichier 000-default.conf (pour frontend React + backend Api Symfo sur le même serveur)** :

```
# FRONTEND oZone
<VirtualHost *:80>
ServerName www.geekoz.fr
ServerAdmin webmaster@localhost
DocumentRoot "/var/www/html/ozone/frontend/dist"

<Directory /var/www/html/ozone/frontend/dist>
        Options +Indexes +Includes +FollowSymLinks +MultiViews
        AllowOverride All
                <IfModule mod_rewrite.c>
                        RewriteEngine On
                        # If an existing asset or directory is requested go to it as it is
                        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -f [OR]
                        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -d
                        RewriteRule ^ - [L]
                        # If the requested resource doesn't exist, use index.html
                        RewriteRule ^ /index.html
                </IfModule>
        Require all granted
</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
RewriteCond %{SERVER_NAME} =www.geekoz.fr
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

# BACKEND oZone
<VirtualHost *:80>
ServerName api.geekoz.fr
ServerAdmin webmaster@localhost
DocumentRoot "/var/www/html/ozone/backend/public"
DirectoryIndex /index.php

<IfModule mod_headers.c>
        Header set Access-Control-Allow-Origin "*"
</IfModule>

<Directory /var/www/html/ozone/backend/public>
        AllowOverride None
        Require all granted
        Allow from All
        FallbackResource /index.php
</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
RewriteEngine on
# Authorization  header
# Règles spécifiques pour le bon fonctionnement du JWT
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
</VirtualHost>
```

> **Pour la version 000-default-le-ssl.conf (connexion en HTTPS)** :

```
# FRONTEND oZone
<IfModule mod_ssl.c>
<VirtualHost *:443>
ServerName www.geekoz.fr
ServerAdmin webmaster@localhost
DocumentRoot "/var/www/html/ozone/frontend/dist"

<Directory /var/www/html/ozone/frontend/dist>
        Options +Indexes +Includes +FollowSymLinks +MultiViews
        AllowOverride All
        <IfModule mod_rewrite.c>
                RewriteEngine On
                # If an existing asset or directory is requested go to it as it is
                RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -f [OR]
                RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -d
                RewriteRule ^ - [L]
                # If the requested resource doesn't exist, use index.html
                RewriteRule ^ /index.html
        </IfModule>
        Require all granted
</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateFile /etc/letsencrypt/live/www.geekoz.fr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/www.geekoz.fr/privkey.pem
</VirtualHost>
</IfModule>

# BACKEND oZone
<IfModule mod_ssl.c>
<VirtualHost *:443>
ServerName api.geekoz.fr
ServerAdmin webmaster@localhost
DocumentRoot "/var/www/html/ozone/backend/public"
DirectoryIndex /index.php

<IfModule mod_headers.c>
        Header set Access-Control-Allow-Origin "*"
</IfModule>

RewriteEngine On
RewriteCond %{REQUEST_METHOD} ^OPTIONS
RewriteRule .* - [F]
<Directory /var/www/html/ozone/backend/public>
        AllowOverride None
        Require all granted
        Allow from All
        FallbackResource /index.php
</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
RewriteEngine on
# Règles spécifiques pour le bon fonctionnement du JWT
# Authorization  header
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
SSLCertificateFile /etc/letsencrypt/live/api.geekoz.fr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/api.geekoz.fr/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
```

### **Ajout du repo Sury pour Debian 10 :**

> _Contient notament les dernières versions de PHP et ses Modules_

`wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -`

`sudo echo "deb https://packages.sury.org/php/ buster main" | sudo tee /etc/apt/sources.list.d/php.list`

### **Installation de Node.js :**

https://github.com/nodesource/distributions

### **Installation de LexikJWTBundle :**

https://github.com/lexik/LexikJWTAuthenticationBundle

> **Règles à ajouter dans la config d'apache (VirtualHost ou .htaccess)** :

```
# Règles spécifiques pour le bon fonctionnement du JWT
# Authorization  header
RewriteEngine on
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
```

> **Génération des clefs private et public sur le serveur** :

_Saisir la JWT passphrase renseignée dans le fichier .env lorsqu'il la demande_

`mkdir -p config/jwt`

`openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096`

`openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout`
