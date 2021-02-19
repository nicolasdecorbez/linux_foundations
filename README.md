# Linux Foundations

Projet du module FDI-SYS1 visant à appréhender les fondamentaux de l’environnement Linux et d'appronfondir nos connaissances en administration système.

## Sommaire

- [Jour 1](#jour-1)
    - [**Étape 1** : Modification du port d'écoute SSH](#étape-1--modification-du-port-découte-ssh)
    - [**Étape 2** : Création de 3 groupes et de leurs utilisateurs](#étape-2--création-de-3-groupes-et-de-leurs-utilisateurs)
    - [**Étape 3** : Création des répertoires dans `/data`](#étape-3--création-des-répertoires-dans-data)
    - [**Étape 4** : Création d'un répertoire et de 3 fichiers dans `/data/compta`](#étape-4--création-dun-répertoire-et-de-3-fichiers-dans-datacompta)
- [Jour 2](#jour-2)
    - [**Étape 5** : Ajout du groupe "Admin" au sudoers](#étape-5--ajout-du-groupe-admin-au-sudoers)
    - [**Étape 6** : Création des variables d'environnement -> *invalide*](#étape-6--création-des-variables-denvironnement-invalide)
    - [**Étape 6 v2** : Création des variables d'environnement -> **correction**](#étape-6-v2--création-des-variables-denvironnement-correction)
    - [**Étape 7** : Installation d'*Apache* et de *PHP*](#étape-7--installation-dapache-et-de-php)
    - [**Étape 8** : Connexion SSH à une autre VM](#étape-8--connexion-ssh-à-une-autre-vm)
    - [**Étape 9** : Synchronisation de répertoires entre 2 VM](#étape-9--synchronisation-de-répertoires-entre-2-vm)
- [Jour 3](#jour-3)
    - [**Étape 10** : Installation et configuration de *Wordpress*](#étape-10--installation-et-configuration-de-wordpress)
    - [**Étape 11** : Renseignements sur *PHP*](#étape-11--renseignements-sur-php)
    - [**Étape 12** : Installation de *Symfony*](#étape-12--installation-de-symfony)
- [Jour 4](#jour-4)
    - [**Étape 13** : Mise en place d'un *CronJob*](#étape-13--mise-en-place-dun-cronjob)
    - [**Étape 14** : Installation de *Nginx* et désactivation d'*Apache*](#étape-14--installation-de-nginx-et-désactivation-dapache)
    - [**Étape 15** : Sécurisation de notre serveur avec *fail2ban*](#étape-15--sécurisation-de-notre-serveur-avec-fail2ban)


## Jour 1

### Étape 1 : Modification du port d'écoute SSH

On modifie la ligne `Port 22` dans le fichier **/etc/ssh/sshd_config** avec les droits *root* :
```
# Specifies the port number that sshd listens on. The default is
# 22. Multiple options of this type are permitted. See also
# ListenAddress.

Port 2242
```

### Étape 2 : Création de 3 groupes et de leurs utilisateurs

On crée un groupe avec la commande `sudo groupadd [name] -g [gid]`.

On vérifie ensuite si les modifications ont bien été prises en compte en regardant dans le fichier **/etc/group** et on peut lire, après la création des 3 groupes :
```
admin:x:4242:
compta:x:4343:
commercial:x:4444:
```

On crée ensuite un utilisateur avec la commande `sudo useradd [name] -g [gid_du_groupe]`.

On l'ajoute ensuite à un groupe avec `sudo gpasswd -a [name] [group_name]`.

On vérifie ensuite dans le fichier **/etc/group** si les utilisateurs sont bien assignés aux groupes désirés :
```
admin:x:4242:toto,bob,john
compta:x:4343:alice,martine
commercial:x:4444:marc,jean,tata
```

### Étape 3 : Création des répertoires dans `/data`

On crée notre répertoire **/data** ainsi que les sous-répertoires **/data/admin**, **/data/compta** et **/data/commercial** avec `sudo mkdir`.

On applique ensuite les permissions requises avec la commande `sudo setfacl -Rm g:[groupname]:[perms] /data/[repo]`. On vérifie ensuite avec la commande `getfacl /data/[repo]/` :
```
getfacl: Removing leading '/' from absolute path names
# file: data/admin/
# owner: root
# group: root
user::rwx
group::---
group:admin:rwx
group:compta:---
group:commercial:---
mask::rwx
other::r-x
```

Pour `[perms]`, on rentre donc `7` pour définir tous les droits, `4` pour les droits de lecture et `0` pour aucun des droits.

### Étape 4 : Création d'un répertoire et de 3 fichiers dans `/data/compta`

On crée notre répertoire `coms` dans `data/compta` et 3 fichier (`com_marc`, `com_jean` et `com_tata`) dans ce dernier.

Avec la configuration actuelle, il est impossible pour les membres du groupe `commercial` d'acceder à ce dossier. Pour ce faire, on doit rentrer la commande `sudo setfacl -Rm g:commercial:7 /data/compta/coms` afin de leurs permettre d'accéder aux fichiers. Aussi, pour permettre de nous déplacer dans `/data/compta/coms`, nous devons donner les droits `x` à `commercial` pour le dossier `compta` : `sudo setfacl -m g:commercial:1 /data/compta`.

On se connecte ensuite avec chaque utilisateur du groupe `commercial` et crée les fichiers respectifs par utilisateurs. Enfin, on se connecte avec `Marc` et on essaye d'ouvrir `com_jean`. Ce fichier à les permissions `r--`, il peut donc l'ouvrir pour le lire mais ne peut pas l'éditer.

Les utilisateurs du groupe `commercial` auront tous accès aux fichiers présents dans ce répertoire car aucune règle n'est spécifié afin de restreindre l'accès à ces fichiers.

Pour restreindre l'accès, nous pourrions utiliser `sudo setfacl -m u:jean:7 /data/compta/coms/com_jean` puis `sudo setfacl -m u::0 /data/compta/coms/com_jean` afin qu'il soit le seul à pouvoir y avoir accès.

## Jour 2

### Étape 5 : Ajout du groupe "Admin" au sudoers

On modifie le fichier `sudoers` situé dans `/etc`, et on ajoute les deux lignes suivantes :
```
# Members of the group 'admin' may gain root privileges
%admin ALL=(ALL) NOPASSWD:ALL
```

### Étape 6 : Création des variables d'environnement -> *invalide*

Pour ajouter des variables d'environnement persistentes, on écrit dans le fichier `/etc/environment` les lignes suivantes :
```
bureau={VALUE}
etage={VALUE}
surnom={VALUE}
```

Afin d'afficher toutes les variables d'environnement, on utilise la commande `env`.

### Étape 6 v2 : Création des variables d'environnement -> **correction**

Suite à l'absence des directory *home* de nos utilisateurs, il faut les créer avec la commande `mkhomedir_helper <username>`.

> Une méthode pour optimiser cette partie serait d'écrire un script qui parcourt le fichier `/etc/passwd`, regarde si un répertoire *home* existe pour chaque utilisateur, et le crée s'il est absent.

Ensuite, on modifie le fichier `/home/$user/.profile` en ajoutant ces 3 lignes :
```
[...]
bureau={VALUE}
etage={VALUE}
surnom={VALUE}
```

On se connecte ensuite avec un utilisateur avec `su $user` ou en *ssh* et lance la commande `env` pour afficher les variables d'environnement définies.

> Pour aller plus vite, je vais utiliser cette combinaison de commandes pour éviter de chercher : `printenv | grep bureau && printenv | grep etage && printenv | grep surnom`.

```
$ printenv | grep -F -e bureau -e etage -e surnom
bureau=1
etage=2
surnom=3
```

### Étape 7 : Installation d'Apache et de PHP

Pour l'installation d'*Apache 2.4*, on utilise simplement `sudo apt install apache2`. En revanche, pour l'installation de *PHP 7.4*, la méthode est un peu plus complexe :

On commence par télécharger la **clé GPG** : `sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg`.
Puis on ajoute notre **dépôt PPA** : `echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list`.
Enfin, on installe **PHP 7.4** : `sudo apt -y install php7.4`.

On teste notre version avec `php -v`:
```
decorb_n@vm046:~$ php -v
PHP 7.4.15 (cli) (built: Feb 12 2021 14:49:58) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.15, Copyright (c), by Zend Technologies
```

Ensuite, on bloque les mise à jours de ces paquets avec la commande `apt-mark hold <nom-du-paquet>`. Ce qui nous donne :
```
decorb_n@vm046:~$ apt-mark showhold
apache2
chef
php7.4
```

Tout est bon !

### Étape 8 : Connexion SSH à une autre VM

Pour cette partie, nous devions faire en sorte qu'il soit possible de se connecter à une autre VM en tant que bob, sans mot de passe. J'ai choisi de me connecter à la *VM055* à l'adresse **172.16.234.55**.

D'abord, on crée un clé ssh de type **RSA 2048 bits** avec la commande `ssh-keygen -t rsa -b 2048` qui sera stockée dans `~/.ssh/id_rsa.pub`. J'envoie cette clé à mon partenaire, et il m'envoie la sienne.

Ensuite, je me connecte à *bob* via `su bob`, puis je crée le dossier `.ssh` dans `/home/bob`. Dans ce dossier, nous créons le fichier `authorized_keys` où on rentre la clé SSH de notre partenaire :
```
ssh-rsa <key> scholt_k@vm055
```

On teste ensuite de se connecter, en tant que *bob*, à la VM de notre partenaire :
```
decorb_n@vm046:~/.ssh$ ssh bob@172.16.234.55 -p 2242
Linux vm055 4.9.0-8-amd64 #1 SMP Debian 4.9.144-3.1 (2019-02-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
$
```

Pas de mot de passe, ça a donc marché.

### Étape 9 : Synchronisation de répertoires entre 2 VM

D'abord, nous devons configurer nos clés SSH entre les deux machines virtuelles afin de pouvoir synchroniser les dossiers (Non essentiel, mais permet d'éviter de s'échanger des mots de passes lors de la connexion SSH).

Ensuite, nous pouvons rentrer la commande pour synchroniser les dossier via SSH :
```
rsync -vae 'ssh -p2242' --no-p --delete /data/admin/configurations <username>@<VM>:/data/admin/
```

Nous pouvons décomposer la commande de la sorte :
- `v` : on affiche ce que la commande fait, pour voir s à un moment elle échoue
- `e` : permet de spécifier le protocole SSH et le port de connexion (`ssh -p2242`)
- `a` : préserve les dates, permissions, etc … des fichiers. Inclus l'option récursivité.
- `no-p` : on annule la sync des permissions.
- `delete` : on supprime, dans la destination, les fichiers qui ne sont plus présents dans notre source.

On rentre ensuite nos chemins : la source (`/data/admin/configurations`) puis la destination (`<username>@<VM>:/data/admin/`, où `<username>@<VM>` est l'adresse de connexion SSH). On lance, et on obtient :
```
decorb_n@vm046:~$ rsync -vae 'ssh -p2242' --no-p --delete /data/admin/configurations scholt_k@172.16.234.55:/data/admin/
sending incremental file list
configurations/ben
configurations/kev
configurations/nico
configurations/val

sent 297 bytes  received 93 bytes  780.00 bytes/sec
total size is 0  speedup is 0.00
```

Puis on vérifie notre dossier sur la machine distante :
```
scholt_k@vm055:~$ ls -l /data/admin/configurations/
total 0
-rw-r--r-- 1 scholt_k scholt_k 0 Feb 16 19:59 ben
-rw-r--r-- 1 scholt_k scholt_k 0 Feb 16 19:59 kev
-rw-r--r-- 1 scholt_k scholt_k 0 Feb 16 19:59 nico
-rw-r--r-- 1 scholt_k scholt_k 0 Feb 16 19:59 val
```

## Jour 3

### Étape 10 : Installation et configuration de Wordpress

Pour l'installation de Wordpress, nous devons d'abord installer et configurer quelques outils, appelés plus communément LAMP (pour Linux, Apache, MySQL et PHP).

On commence par installer notre gestionnaire de base de données, **MariaDB** avec `sudo apt install mariadb-server`. Puis nous allons créer un nouvel utilisateur et une nouvelle base de données :
```
decorb_n@vm046:~$ sudo mysql

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 121
Server version: 10.1.48-MariaDB-0+deb9u1 Debian 9.13

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database wordpress;
MariaDB [(none)]> create user wbuser identified by 'wbuser';
MariaDB [(none)]> exit;

Bye
```

Ensuite, on va faire en sorte qu'**Apache2** utilise **PHP 7.4** et non PHP 7.0. Pour ça, on rentre ces commandes :
```
decorb_n@vm046:~$ sudo a2dismod php7.0
Module php7.0 successfully disabled

decorb_n@vm046:~$ sudo a2enmod php7.4
Considering dependency mpm_prefork for php7.4:
Considering conflict mpm_event for mpm_prefork:
Considering conflict mpm_worker for mpm_prefork:
Module mpm_prefork successfully enabled
Considering conflict php5 for php7.4:
Module php7.4 successfully enabled

Run the following command to apply the changes made :
    systemctl restart apache2
```

Et on installe enfin une (ou des) extension pour **PHP 7.4** :
```
sudo apt-get -y install php7.4-mysql
```

> On pourrait d'ailleurs installer d'autres extensions comme php7.4-curl pour utiliser le CMS WordPress, ou encore php7.4-xml, php7.4-dev, etc. en fonction de nos besoins.

Une fois la préparation de notre système fait, on peut commencer à intégrer WordPress. On lance ces commandes :
```
decorb_n@vm046:~$ cd /var/www/html
decorb_n@vm046:/var/www/html$ sudo wget http://fr.wordpress.org/latest-fr_FR.tar.gz
decorb_n@vm046:/var/www/html$ sudo tar -xf latest-fr_FR.tar.gz
decorb_n@vm046:/var/www/html$ rm index.html
decorb_n@vm046:/var/www/html$ sudo mv worpress/* .
```

On crée le fichier `phpinfo.php` dans notre dossier **WordPress** pour avoir des informations sur notre version de **PHP** utilisée, et on écrit les lignes suivantes dans ce dernier :
```php
<?php
phpinfo();
?>
```

On peut y avoir accès à l'adresse : `<ip>/phpinfo.php`.

Ensuite, on accède à notre site via l'adresse IP de notre VM, on lance la configuration en rentrant nos informations de notre **database**, le nom du site, puis on peut enfin accéder à notre site web pour effectuer des modifications !

### Étape 11 : Renseignements sur PHP

Pour afficher l'emplacement du binaire PHP, on peut utiliser les commandes `php -i | grep "_ => /"` et `type php` :
```
decorb_n@vm046:~$ php -i | grep "_ => /"
_ => /usr/bin/php
decorb_n@vm046:~$ type php
php is hashed (/usr/bin/php)
```

Pour afficher la version de PHP installée :
```
decorb_n@vm046:~$ php -v
PHP 7.4.15 (cli) (built: Feb 12 2021 14:49:58) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.15, Copyright (c), by Zend Technologies
```

L'installation de **PHP 5.6** se fait facilement avec `apt install php5.6`. On vérifie qu'il est bien installé avec la commande `php5.6 -v` :
```
decorb_n@vm046:~$ php5.6 -v
PHP 5.6.40-41+0~20210213.47+debian9~1.gbp6fd845 (cli)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```

Afin que **PHP 7.4** soit notre version de PHP par défaut, il faut lancer la commande `sudo update-alternatives --config php`, puis de regarder si les modifications ont bien été réalisées :
```
decorb_n@vm046:~$ sudo update-alternatives --config php
There are 3 choices for the alternative php (providing /usr/bin/php).

  Selection    Path             Priority   Status
------------------------------------------------------------
* 0            /usr/bin/php8.0   80        auto mode
  1            /usr/bin/php5.6   56        manual mode
  2            /usr/bin/php7.4   74        manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
update-alternatives: using /usr/bin/php7.4 to provide /usr/bin/php (php) in manual mode

decorb_n@vm046:~$ php -v
PHP 7.4.15 (cli) (built: Feb 12 2021 14:49:58) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.15, Copyright (c), by Zend Technologies
```
### Étape 12 : Installation de Symfony

Sur le site des prérequis pour l'installation de [Symfony](https://symfony.com/doc/current/setup.html#technical-requirements), on en note 2 majeures :

- plusieurs extensions de PHP : `ctype`, `iconv`, `JSON`, `PCRE`, `Session`, `SimpleXML`, et `Tokenizer`.
- [Composer](https://getcomposer.org/)

On va commencer par l'installation de **Composer** :
```
decorb_n@vm046:~$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
decorb_n@vm046:~$ php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Installer verified

decorb_n@vm046:~$ php composer-setup.php
All settings correct for using Composer
Downloading...

Composer (version 2.0.9) successfully installed to: /home/decorb_n/composer.phar
Use it: php composer.phar

decorb_n@vm046:~$ php -r "unlink('composer-setup.php');"
```

Puis on le rend accessible depuis partout sur la machine :
```
decorb_n@vm046:~$ mv composer.phar composer
decorb_n@vm046:~$ chmod a+x composer
decorb_n@vm046:~$ sudo mv composer /usr/local/bin/composer
decorb_n@vm046:~$ composer
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.0.9 2021-01-27 16:09:27
[...]
```

Ensuite, on va installer **Symfony CLI**, qui va nous permettre de créer facilement des applications, et qui embarques plusieurs outils très utiles comme le `symfony check:requirements`. On va également faire en sorte de l'installer globalement :
```
decorb_n@vm046:~$ wget https://get.symfony.com/cli/installer -O - | bash
[...]
The Symfony CLI v4.22.0 was installed successfully!

Use it as a local file:
  /home/decorb_n/.symfony/bin/symfony

Or add the following line to your shell configuration file:
  export PATH="$HOME/.symfony/bin:$PATH"

Or install it globally on your system:
  mv /home/decorb_n/.symfony/bin/symfony /usr/local/bin/symfony

Then start a new shell and run 'symfony'

decorb_n@vm046:~$ logout
Connection to 172.16.234.46 closed.

PS C:\Users\Nico\AppData\Roaming\eDEX-UI> ssh decorb_n@172.16.234.46 -p 2242
decorb_n@172.16.234.46's password:

decorb_n@vm046:~$ symfony
Symfony CLI version v4.22.0 (c) 2017-2021 Symfony SAS
Symfony CLI helps developers manage projects, from local code to remote infrastructure
[...]
```

On fait donc une vérification de notre système avec `symfony check:requirements` :
```
decorb_n@vm046:~$ symfony check:requirements

Symfony Requirements Checker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

> PHP is using the following php.ini file:
/etc/php/7.4/cli/php.ini

> Checking Symfony requirements:

.............................

 [OK]
 Your system is ready to run Symfony projects

[...]
```

Tout est donc prêt pour la configuration de *Symfony*. On commence par créer une application avec `symfony new <app_name>`, puis y intégrer le **pack Apache** pour *Symfony* avec `composer require symfony/apache-pack`.

> L'intégration du *pack Apache* nécessite, en plus des extensions PHP requises par Symfony, l'installation de `php-sqlite3`.

On se rend dans le dossier de notre apllication et lance `symfony server:start`, puis nous nous rendons à l'adresse **172.16.234.46:8000** : l'app est fonctionnelle !

On va maintenant la déplacer dans le dossier `/var/www` pour la rendre accessible par **Apache**, qu'on va mainteant configurer. Je vais créer le fichier `symfony_application.conf` ([Disponible ici](src/symfony_application.conf)) qui contiendra la configuration du *Virtual Host* de notre application.

Une fois notre fichier de configuration réalisé, on peut le placer dans `/etc/apache2/sites-available`, puis on exécute ces commandes :
```
decorb_n@vm046:~$ sudo rm /etc/apache2/sites-enabled/000-default.conf
decorb_n@vm046:~$ sudo ln -s /etc/apache2/sites-available/symfony_application.conf /etc/apache2/sites-enabled/symfony_application.conf
```

Les fichiers présents dans `/etc/apache2/sites-enabled` sont uniquement des liens symboliques faisant référence aux *Virtual Hosts* présents dans `/etc/apache2/sites-available`. On supprime donc le lien symbolique vers le fichier par défaut, puis on en crée un nouveau vers notre nouveau fichier de configuration. Enfin, on redémarre Apache avec `sudo systemctl restart apache2`.

En se connectant avec notre adresse IP, on arrive directement sur notre `index.php`. Il faut maintenant aller modifier le fichier `hosts` sur notre machine hôte.

Sur Windows, le chemin pour accéder à ce fichier est `C:WINDOWS/system32/drivers/etc/hosts`. On l'édite en tant qu'administrateur et on ajoute cette ligne :
```
172.16.234.46   symfony.apache2.local
```

On sauvegarde, et on teste d'accéder à l'adresse désirée (une capture d'écran est disponible **[ici](/src/symfony.JPG)**).

## Jour 1

### Étape 13 : Mise en place d'un CronJob

Pour mettre en place un `CronJob` sur l'utilisateur `john`, on lance la commande `sudo crontab -u john -e` et on écrit cette ligne à la fin du fichier :

```
42 23 * * 3,6 systemctl restart apache2
```

Le redémarrage d'`Apache` aura donc lieu chaque **mercredi** et **samedi** à **23h42**

### Étape 14 : Installation de `Nginx` et désactivation d'`Apache`

On commence par arrêter le service `apache2`, puis on le désactive :

```
sudo systemctl stop apache2
sudo systemctl disable apache2
```

Puis on installe `Nginx` avec `apt install nginx`. Ses fichiers de configurations sont stockés dans `/etc/nginx`.

> La mise en place d'un serveur `nginx` qui héberge des applications en `php` nécessite l'installation de plusieurs extensions, notamment `php-fpm`, `php-xml` ou encore `php-curl`.

Dans un premier temps, nous allons modifier le fichier `nginx.conf` situé dans `/etc/nginx`. On sauvegarde la version de base en `.old` et on rempli le nouveau fichier de configuration avec ces lignes :
```
# ------------------------------------ #
#        Modified startup file.        #
#     configuration épurée et opti     #
# pour la compatibilité avec wordpress #
# ------------------------------------ #

user www-data www-data;

#usually equal to number of CPU (we only have access to 1 proc here.)
worker_processes  1;
worker_cpu_affinity 1;

error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include mime.types;
    default_type       application/octet-stream;
    access_log         /var/log/nginx/access.log;
    sendfile           on;
    keepalive_timeout  3;

    #php max upload limit
    client_max_body_size 13m;

    include sites-enabled/*;
}
```

Ensuite nous allons configurer nos **`Server Blocks`** : ce sont les équivalents, chez **Nginx**, des `Virtual Hosts` d'**Apache**. Nous allons donc configurer d'un côté notre page **Wordpress** à l'adresse `wordpress.nginx.local`, et **Symfony** accessible via `symfony.nginx.local`.

On commence par se rendre dans le dossier `/etc/nginx/sites-availables`, où nous allons créer le *Server Block* de **Symfony** :
```
# ------------------------------------ #
#         Modified server file         #
#       symfony.nginx.local.conf       #
# ------------------------------------ #

server {
        listen 80;
        listen [::]:80;

        # On rentre le nom du serveur -> très important
        # pour accéder à tel ou tel server block
        server_name symfony.nginx.local;

        root /var/www/project/public;
        index index.php;

        location / {
                # On teste d'abord le root en tant que fichier
                # puis en tannt que dossier, sinon on envoie 404.
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
```

Puis celui de **Wordpress** :
```
# ------------------------------------ #
#         Modified server file         #
#      wordpress.nginx.local.conf      #
# ------------------------------------ #

upstream php {
        # On crée un upstream afin de pouvoir accéder au module fastcgi, via php-cgi.
        server unix:/tmp/php7.4-cgi.socket;
        server 127.0.0.1:9000;
}

server {
        server_name wordpress.nginx.conf;

        root /var/www/wordpress;
        index index.php;

        # Les deux location ci-dessous sont spécifique à Wordpress
        location = /favicon.ico {
                log_not_found off;
                access_log off;
        }

        location = /robots.txt {
                allow all;
                log_not_found off;
                access_log off;
        }

        location / {
                # On ajoute "?$args" pour éviter d'éventuels conflits avec les permalinks
                try_files $uri $uri/ /index.php?$args;
        }

        # Attention à bien rentrer la bonne version de PHP à la ligne fastcgi_pass.
        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }

        # Spécifique à Wordpress
        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                expires max;
                log_not_found off;
        }
}
```

Maintenant que nos deux *Server Blocks* sont crées, on va devoir les activer. On va donc créer des liens symboliques depuis `sites-availables` vers `sites-enabled`.

On commence par supprimer le lien par défaut, puis on écrit ces commandes :
```
decorb_n@vm046 [02:22:53] [~]
-> $ cd /etc/nginx/sites-enabled

decorb_n@vm046 [02:23:10] [/etc/nginx/sites-enabled]
-> $ sudo ln -s ../sites-availables/wordpress.nginx.local.conf .

decorb_n@vm046 [02:22:22] [/etc/nginx/sites-enabled]
-> $ sudo ln -s ../sites-availables/symfony.nginx.local.conf .

decorb_n@vm046 [02:22:38] [/etc/nginx/sites-enabled]
-> $ ls -l
total 0
lrwxrwxrwx 1 root root  25 Feb 18 21:31 wordpress.nginx.local.conf -> ../sites-available/wordpress.nginx.local.conf
lrwxrwxrwx 1 root root  47 Feb 18 21:41 symfony.nginx.local.conf -> ../sites-available/symfony.nginx.local.conf
```

Nos sites sont mainteant activés, on peut redémarrer *nginx* et vérifier si tout est fonctionnel avec `systemctl status nginx`.

Maintenant, sur notre machine hôte, dans notre fichier `C:WINDOWS/system32/drivers/etc/hosts` (ou `/etc/hosts`) et on ajoute les deux lignes suivantes :
```
172.16.234.46  symfony.nginx.local
172.16.234.46  wordpress.nginx.local
```

Puis on teste ces adresses dans un navigateur : la redirection se fait parfaitement en fonction de l'adresse utilisée !

### Étape 15 : Sécurisation de notre serveur avec `fail2ban`

Afin de sécuriser notre serveur, on va installer puis intégrer `fail2ban` à `nginx` et `ssh` pour chercher des tentatives répétées de connexions échouées, puis de bannir en fonction de sa configuration.

L'installation se fait assez simplement avec `apt install fail2ban`, on peut commencer à travailler dessus : la sécurisation de `nginx` va se faire en 2 étapes :

- Création d'un filtre
- Création d'une *jail*

D'abord, on commence par aller jeter un coup d'ailleurs à l'apparence de ce qu'on recherche, dans les logs de nginx (ils se trouvent dans `/var/log/nginx/error.log`). Ci-dessous se trouvent celles associées à une erreur **403** :
```
decorb_n@vm046 [17:42:07] [~]
-> $ sudo cat /var/log/nginx/error.log | grep forbidden
[...]
2021/02/18 11:35:44 [error] 115784#115784: *1 directory index of "/var/www/html/" is forbidden, client: 172.16.14.96, server: wordpress.nginx.local, request: "GET / HTTP/1.1", host: "172.16.234.46"
2021/02/18 11:36:27 [error] 115784#115784: *3 directory index of "/var/www/html/" is forbidden, client: 172.16.14.107, server: wordpress.nginx.local, request: "GET / HTTP/1.1", host: "172.16.234.46"
2021/02/18 14:15:20 [error] 120236#120236: *26 directory index of "/var/www/html/" is forbidden, client: 172.16.128.126, server: wordpress.nginx.local, request: "GET / HTTP/1.1", host: "wordpress.nginx.local"
[...]
```

On remarque donc un schéma composé de :
- Une balise **`[error]`** qui nous indique que la ligne est bien une erreur. (Il existe également `[emerge]`, `[alert]`, etc.)
- Un nombre (**`115784`** par exemple), puis un **`#`** et de nouveau un nombre.
- Du mot **`forbidden`** qui nous indique que c'est bien une erreur 403.

Nous allons donc écrire une expression standard afin de réaliser ce schéma et de récupérer l'ip du client, puis on le met dans un nouveau filtre `nginx-forbidden.conf` dans `/etc/fail2ban/filter.d` :
```
# -------------------------------- #
# Filtre pour l'erreur 403 (Nginx) #
# -------------------------------- #

[Definition]

failregex = .* \[error\] \d*\#\d*: \*\d* .* forbidden.* client: <HOST>, .*$
ignoreregex =
```

On teste le fitre avec `fail2ban-regex`. J'utilise également `grep -c` afin de comparer les résultats obtenus:
```
decorb_n@vm046 [17:21:06] [/etc/fail2ban/filter.d]
-> $ sudo fail2ban-regex /var/log/nginx/error.log /etc/fail2ban/filter.d/nginx-forbidden.conf

Running tests
=============

Use   failregex filter file : nginx-forbidden, basedir: /etc/fail2ban
Use         log file : /var/log/nginx/error.log
Use         encoding : UTF-8


Results
=======

Failregex: 177 total
|-  #) [# of hits] regular expression
|   1) [177] .* \[error\] \d*\#\d*: \*\d* .* forbidden.* client: <HOST>, .*$
`-

Ignoreregex: 0 total

Date template hits:
|- [# of hits] date format
|  [283] Year(?P<_sep>[-/.])Month(?P=_sep)Day 24hour:Minute:Second(?:,Microseconds)?
`-

Lines: 283 lines, 0 ignored, 177 matched, 106 missed
[processed in 0.02 sec]

Missed line(s): too many to print.  Use --print-all-missed to print all 106 lines

decorb_n@vm046 [17:57:26] [/etc/fail2ban]  
-> $ sudo cat /var/log/nginx/error.log | grep forbidden -c
177

```

Mon filtre est donc opérationnel. Je crée donc la *jail* `nginx-forbidden.conf` dans `/etc/fail2ban/jail.d` :
```
# ------------------------------ #
# Jail pour l'erreur 403 (Nginx) #
# ------------------------------ #

[nginx-forbidden]

enabled = true
filter = nginx-forbidden
port = 80
logpath = /var/log/nginx/*error*.log
maxretry = 5
```

J'en profite pour aller modifier le temps de ban par défaut dans le fichier `/etc/fail2ban/jail.local` la ligne suivante :
```
[DEFAULT]

ignoreip = 127.0.0.1/8
findtime = 600

# On set notre bantime à 600 sec = 10 min
bantime  = 600
maxretry = 5
```

Puis lance la commande `fail2ban-client -d` pour tester ma nouvelle *jail* :
```
decorb_n@vm046 [18:16:45] [/etc/fail2ban]
-> $ sudo fail2ban-client -d
[...]
['start', 'nginx-forbidden']
```

Notre `nginx` est mainteant sécurisé. On peut maintenant faire la même chose pour `SSH`. On commence par créer la *jail* `ssh-mod.conf` en copiant `defaults-debian.conf.old` (qui deviendra `defaults-debian.conf.old` par la suite), et on y écrit ces lignes :
```
# ----------------------- #
# Jail modifiée pour sshd #
# ----------------------- #

[sshd]

enabled = true

# On override la règle de base
maxretry = 7

# On ajoute notre port modifié pdt l'étape 1
port = 2242
```

Puis on teste de nouveau :
```
decorb_n@vm046 [18:27:07] [/etc/fail2ban]
-> $ sudo fail2ban-client -d
[...]
['set', 'sshd', 'maxretry', 7]
['set', 'sshd', 'bantime', 600]
[...]
['set', 'sshd', 'action', 'iptables-multiport', 'port', '2242']
[...]
['start', 'sshd']
```

Et voilà, notre `fail2ban` est correctement configuré ! On peut maintenant utiliser les commandes suivantes pour :
- **`fail2ban-client status`** : Voir les jails actives.
- **`fail2ban-client set <jail> unbanip <IP>`** : Débloquer une IP bloquée.
- **`fail2ban-client status <jail>`** : Voir quelles IPs ont été bloquées (par jail).

> Un autre méthode pour voir les IPs bloquées est d'affichier les logs de `fail2ban` puis de grep dessus (`cat /var/log/fail2ban.log | grep -F -e "Ban"`). Néanmoins, cette méthode affichera des IPs bloquées dans le passé, mais peut-être débloquées depuis.
