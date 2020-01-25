# **Installation de base du serveur :**

## **Création d'un espace de swap :**

> _Pour vérifier si il existe un espace de swap :_

`sudo swapon --show`

> _Création d'une partition swap de 2G :_

`sudo fallocate -l 2G /swapfile`

`sudo chmod 600 /swapfile`

`sudo mkswap /swapfile`

`sudo swapon /swapfile`

## **Installation LAMP Stack :**

### **Installation Apache Web Server  :**

`sudo apt-get install apache2 apache2-doc`

#### Ajout de la gestion HTTPS :

https://linuxhint.com/setup_free_ssl_cert_apache_debian/

`sudo apt install ca-certificates apt-transport-https`

### **Modification des droits sur dossier /var/www/ :**

`sudo chown -R $USER:www-data /var/www`

`sudo chmod -R g+rw /var/www`

#### Dossier de création des VirtualHosts :

> _Par defaut le site existant est :_

`sudo nano /etc/apache2/sites-available/000-default.conf`

> _On peut créer des nouveaux fichiers pour chacun de nos sites hébergés :_

`sudo nano /etc/apache2/sites-available/site-exemple.conf`

> _Exemple de contenu du fichier_ :

```
<VirtualHost *:80>

        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        <Directory /var/www/html>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Require all granted
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf

</VirtualHost>
```
### **Ajout du repo Sury pour Debian 10 (contient notament les dernières versions de PHP et Modules) :**

`wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -`

`sudo echo "deb https://packages.sury.org/php/ buster main" | sudo tee /etc/apt/sources.list.d/php.list`


### **Installation de Node.js :**

https://github.com/nodesource/distributions