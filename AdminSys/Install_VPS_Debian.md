# **Installation de base du serveur :**

## **Mise en place de la sécurisation de la connection SSH :**
_(Dans cet exemple le serveur sera 'ozone.best')_

> **Première connexion pour créer une entrée dans le known_hosts :**
>
>`ssh root@ozone.best` _(saisir le mot de passe root SSH qui vous a été communiqué)_

> **Ajout de la clef SSH au serveur pour éviter de devoir rentrer le password à chaque connection :**
>
>[Lien vers documentation complète](https://www.ssh.com/ssh/copy-id#setting-up-public-key-authentication)
>
>`ssh-copy-id -i ~/.ssh/id_rsa root@ozone.best`

## **Création d'un nouvel utilisateur sudo :**
_(Dans cet exemple l'utilisateur sera 'dbl-lnx')_

> **Création de l'utilisateur :**
>
>`adduser dbl-lnx`

> **Ajout de l'utilisateur au groupe sudo :**
>
>`usermod -aG sudo dbl-lnx`

> **Connection sur ce nouvel utilisateur :**
>
>`su - dbl-lnx`

## **Création d'un espace de swap :**

> **Pour vérifier si il existe un espace de swap :**
>
>`sudo swapon --show`
>
>_Si un espace de SWAP existe çà doit retourner quelque chose comme ceci :_
>
>`NAME      TYPE SIZE USED PRIO`
>
>`/swapfile file   2G   0B   -2`
>
>_Si aucun espace de SWAP n'existe çà ne retourne rien_

> **Création d'une partition swap de 2G :**
>
>`sudo fallocate -l 2G /swapfile`
>
>`sudo chmod 600 /swapfile`
>
>`sudo mkswap /swapfile`
>
>`sudo swapon /swapfile`

## **Installation LAMP Stack :**

### **Ajout du repo Sury pour Debian 10 :**

> **Ajout de GNUPG pour la gestion sécurisée des repositories additionnels :**
> 
> `sudo apt install gnupg`
>
> _(Contient notament les dernières versions de PHP et Extensions)_
>
>`wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -`
>
>`sudo echo "deb https://packages.sury.org/php/ buster main" | sudo tee /etc/apt/sources.list.d/php.list`

### **Installation Apache Web Server :**
>[Lien vers la documentation complète](https://linuxize.com/post/how-to-install-apache-on-debian-10/)
>
>`sudo apt-get install apache2 apache2-doc`

#### Modification des droits sur dossier /var/www/ :

>`sudo chown -R $USER:www-data /var/www/html/ozone.best`
>
>`sudo chmod -R g+rw /var/www/html/ozone.best`

#### Dossier de création des VirtualHosts :

> **Par defaut le site existant est :**
>
>`sudo nano /etc/apache2/sites-available/000-default.conf`

> **On peut créer des nouveaux fichiers pour chacun de nos sites hébergés :**
>
>`sudo nano /etc/apache2/sites-available/site-exemple.conf`

> **Exemple de contenu du fichier 000-default.conf (pour frontend React + backend Api Symfo sur le même serveur)** :
>
>```
># FRONTEND oZone
><VirtualHost *:80>
>ServerName ozone.best
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot "/var/www/html/ozone/frontend/dist"
>
><Directory /var/www/html/ozone/frontend/dist>
>        Options +Indexes +Includes +FollowSymLinks +MultiViews
>        AllowOverride All
>                <IfModule mod_rewrite.c>
>                        RewriteEngine On
>                        # If an existing asset or directory is requested go to it as it is
>                        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -f [OR]
>                        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -d
>                        RewriteRule ^ - [L]
>                        # If the requested resource doesn't exist, use index.html
>                        RewriteRule ^ /index.html
>                </IfModule>
>        Require all granted
></Directory>
>
>ErrorLog ${APACHE_LOG_DIR}/error.log
>CustomLog ${APACHE_LOG_DIR}/access.log combined
>RewriteCond %{SERVER_NAME} =ozone.best
>RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
></VirtualHost>
>
># BACKEND oZone
><VirtualHost *:80>
>ServerName api.ozone.best
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot "/var/www/html/ozone/backend/public"
>DirectoryIndex /index.php
>
><IfModule mod_headers.c>
>        Header set Access-Control-Allow-Origin "*"
></IfModule>
>
><Directory /var/www/html/ozone/backend/public>
>        AllowOverride None
>        Require all granted
>        Allow from All
>        FallbackResource /index.php
></Directory>
>
>ErrorLog ${APACHE_LOG_DIR}/error.log
>CustomLog ${APACHE_LOG_DIR}/access.log combined
>RewriteEngine on
># Authorization  header
># Règles spécifiques pour le bon fonctionnement du JWT
>RewriteCond %{HTTP:Authorization} ^(.*)
>RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
></VirtualHost>
>```
>_On sauvegarde avec CTRL+X puis O ou Y si en Anglais_

> **Pour la version 000-default-le-ssl.conf (connexion en HTTPS)** :
>
>```
># FRONTEND oZone
><IfModule mod_ssl.c>
><VirtualHost *:443>
>ServerName ozone.best
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot "/var/www/html/ozone/frontend/dist"
>
><Directory /var/www/html/ozone/frontend/dist>
>        Options +Indexes +Includes +FollowSymLinks +MultiViews
>        AllowOverride All
>        <IfModule mod_rewrite.c>
>                RewriteEngine On
>                # If an existing asset or directory is requested go to it as it is
>                RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -f [OR]
>                RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -d
>                RewriteRule ^ - [L]
>                # If the requested resource doesn't exist, use index.html
>                RewriteRule ^ /index.html
>        </IfModule>
>        Require all granted
></Directory>
>
>ErrorLog ${APACHE_LOG_DIR}/error.log
>CustomLog ${APACHE_LOG_DIR}/access.log combined
>
>Include /etc/letsencrypt/options-ssl-apache.conf
>SSLCertificateFile /etc/letsencrypt/live/ozone.best/fullchain.pem
>SSLCertificateKeyFile /etc/letsencrypt/live/ozone.best/privkey.pem
></VirtualHost>
></IfModule>
>
># BACKEND oZone
><IfModule mod_ssl.c>
><VirtualHost *:443>
>ServerName api.ozone.best
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot "/var/www/html/ozone/backend/public"
>DirectoryIndex /index.php
>
><IfModule mod_headers.c>
>        Header set Access-Control-Allow-Origin "*"
></IfModule>
>
>RewriteEngine On
>RewriteCond %{REQUEST_METHOD} ^OPTIONS
>RewriteRule .* - [F]
><Directory /var/www/html/ozone/backend/public>
>        AllowOverride None
>        Require all granted
>        Allow from All
>        FallbackResource /index.php
></Directory>
>
>ErrorLog ${APACHE_LOG_DIR}/error.log
>CustomLog ${APACHE_LOG_DIR}/access.log combined
>RewriteEngine on
># Règles spécifiques pour le bon fonctionnement du JWT
># Authorization  header
>RewriteCond %{HTTP:Authorization} ^(.*)
>RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
>SSLCertificateFile /etc/letsencrypt/live/api.ozone.best/fullchain.pem
>SSLCertificateKeyFile /etc/letsencrypt/live/api.ozone.best/privkey.pem
>Include /etc/letsencrypt/options-ssl-apache.conf
></VirtualHost>
></IfModule>
>```
>_On sauvegarde avec CTRL+X puis O ou Y si en Anglais_

### **Installation des certificats HTTPS pour Apache :**

https://linuxhint.com/setup_free_ssl_cert_apache_debian/
>
>`sudo apt install certbot python-certbot-apache ca-certificates apt-transport-https -y`
>
>`sudo certbot --apache`
>
>```
> - Congratulations! Your certificate and chain have been saved at:
>   /etc/letsencrypt/live/api.ozone.best/fullchain.pem
>   Your key file has been saved at:
>   /etc/letsencrypt/live/api.ozone.best/privkey.pem
>   Your cert will expire on 2020-04-24. To obtain a new or tweaked
>   version of this certificate in the future, simply run certbot again
>   with the "certonly" option. To non-interactively renew *all* of
>   your certificates, run "certbot renew"
> - Your account credentials have been saved in your Certbot
>   configuration directory at /etc/letsencrypt. You should make a
>   secure backup of this folder now. This configuration directory will
>   also contain certificates and private keys obtained by Certbot so
>   making regular backups of this folder is ideal.
>```

> **Pour renouveller tous les certificats :**
>
>`certbot renew`

> **Forcer la redirection sur HTTPS via VirtualHosts :**
>
>```
># FRONTEND oZone
><VirtualHost *:80>
>ServerName ozone.best
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot "/var/www/html/ozone.best/frontend/dist"
>Redirect permanent / https://ozone.best/
>
># BACKEND oZone
><VirtualHost *:80>
>ServerName api.ozone.best
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot "/var/www/html/ozone.best/backend/public"
>Redirect permanent / https://api.ozone.best/
>```

### **Installation PHP avec Apache (+ quelques extensions indispensables) :**
[Lien vers la documentation complète](https://linuxize.com/post/how-to-install-php-on-debian-10/)

`sudo apt install php libapache2-mod-php php-json php-mbstring php-zip php-gd php-xml php-curl php-mysql`

>**Installation de toutes les extensions Symfony pour PHP :**
>
>`sudo apt install php-symfony*`

### **Installation de MariaDB Server :**
[Lien vers la documentation complète](https://linuxize.com/post/how-to-install-mariadb-on-debian-10/)

`sudo apt install mariadb-server`

> **Sécuriser l'installation de MariaDB server**
>
>`sudo mysql_secure_installation`
>
>```
>Enter current password for root (enter for none):
>...
>Set root password? [Y/n] Y
>New password: 
>Re-enter new password: 
>...
>Remove anonymous users? [Y/n] Y
>...
>Disallow root login remotely? [Y/n] Y
>...
>Remove test database and access to it? [Y/n] Y
>...
>Reload privilege tables now? [Y/n] Y
>...
>Thanks for using MariaDB!
>```

> **Modifier la méthode d'authentification root par defaut :**
>
>`sudo mysql`
>
>```sql
>ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password;
>ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_root_password';
>```
>_(Pour vérifier que çà fonctionne on tente de se reconnecter sans le sudo avec le mdp défini 'new_root_password')_
>
>`mysql -u root -p`

> **Créer un nouvelle utilisateur de la base de données avec les pleins pouvoirs :**
> 
>_(remplacer 'user' et 'password' par les les identifiants souhaités)_
>```sql
>GRANT ALL ON *.* TO 'user'@'localhost' IDENTIFIED BY 'password';
>FLUSH PRIVILEGES;
>```

### **Installation de PHPMyAdmin :**
[Lien vers la documentation complète](https://linuxhint.com/install_phpmyadmin_debian_10/)

> **[Télécharger la dernière version de phpMyAdmin](https://www.phpmyadmin.net/downloads/) :**
>
>_(penser à adapter les commandes ci-dessus avec les bons numéros de version ici '5.0.2')_
>
>`wget https://files.phpmyadmin.net/phpMyAdmin/5.0.2/phpMyAdmin-5.0.2-all-languages.zip`

> **Installation du paquet unzip permettant de décompresser le package :**
>
>`sudo apt install unzip`

> **Décompression du package dans le dossier '/opt' :**
>
>`sudo unzip phpMyAdmin-5.0.2-all-languages.zip -d /opt`

> **Renommer le dossier généré en quelque chose de plus simple :**
>
>`sudo mv /opt/phpMyAdmin-5.0.2-all-languages /opt/phpMyAdmin`

> **On modifie les droits sur le dossier phpMyAdmin pour Apache :**
>
>`sudo chown -Rfv www-data:www-data /opt/phpMyAdmin`

> **On edite un fichier de conf Apache pour phpMyAdmin :**
>
>`sudo nano /etc/apache2/sites-available/phpmyadmin.conf`

> **On met celà dedans :** _(Modifier le port *:9000 par autre chose si nécessaire et adresse mail 'ServerAdmin')_
>
>```
><VirtualHost *:9000>
>ServerAdmin dbl.bzh@mailfence.com
>DocumentRoot /opt/phpMyAdmin
> 
><Directory /opt/phpMyAdmin>
>Options Indexes FollowSymLinks
>AllowOverride none
>Require all granted
></Directory>
>ErrorLog ${APACHE_LOG_DIR}/error_phpmyadmin.log
>CustomLog ${APACHE_LOG_DIR}/access_phpmyadmin.log combined
></VirtualHost>
>```
>_On sauvegarde avec CTRL+X puis O ou Y si en Anglais_

> **On modifie les ports écoutés par Apache :** _(On ajoute la ligne 'Listen 9000' ou autre si port changé à l'étape d'avant)_
>
>`sudo nano /etc/apache2/ports.conf`
>
>```
># If you just change the port or add more ports here, you will likely also
># have to change the VirtualHost statement in
># /etc/apache2/sites-enabled/000-default.conf
>
>Listen 80
>Listen 9000
>
><IfModule ssl_module>
>        Listen 443
></IfModule>
>
><IfModule mod_gnutls.c>
>        Listen 443
></IfModule>
>
># vim: syntax=apache ts=4 sw=4 sts=4 sr noet
>```
>_On sauvegarde avec CTRL+X puis O ou Y si en Anglais_

> **On charge la conf créée pour phpMyAdmin dans Apache :**
>
>`sudo a2ensite phpmyadmin.conf`

> **On redémarre le service Apache :**
>
>`sudo systemctl restart apache2`

> **On copie le fichier de conf phpMyAdmin par défaut :**
>
>`sudo cp /opt/phpMyAdmin/config.sample.inc.php /opt/phpMyAdmin/config.inc.php`

> **On génère un 'blowfish_secret' plus fort :**
> 
> [Strong Password Generator](https://passwordsgenerator.net/)
> 
> 'Password_Length :' 64
>  
> 'Exclude Ambiguous Characters :' On coche
> 
> Puis on fait 'Generate'
> 
> exemple de password généré : XaHLA&rQh+5xXmc3Q#&4D#T!#=eX*y#sRx6F_fxXnuZYugBvFa+3xj9^es=Esj#W


> **On modifie le blowfish_secret :**
>
>`sudo nano /opt/phpMyAdmin/config.inc.php`
>
>```
>$cfg['blowfish_secret'] = 'XaHLA&rQh+5xXmc3Q#&4D#T!#=eX*y#sRx6F_fxXnuZYugBvFa+3xj9^es=Esj#W';
>```
>_On sauvegarde avec CTRL+X puis O ou Y si en Anglais_


### **Installation de Node.js :**

https://github.com/nodesource/distributions

### **Installation de LexikJWTBundle :**

https://github.com/lexik/LexikJWTAuthenticationBundle

> **Génération des clefs private et public sur le serveur** :
>
>_Saisir la JWT passphrase renseignée dans le fichier .env lorsqu'il la demande_
>
>`mkdir -p config/jwt`
>
>`openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096`
>
>`openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout`

> **Règles à ajouter dans la config d'Apache (VirtualHost ou .htaccess)** :
>
>```
># Règles spécifiques pour le bon fonctionnement du JWT
># Authorization  header
>RewriteEngine on
>RewriteCond %{HTTP:Authorization} ^(.*)
>RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
>```
