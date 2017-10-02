##### Raspberry

```batch
sudo apt-get update upgrade
sudo apt-get install -y nginx php5 php5-fpm mysql-server php5-mysql
```

Desactiver Apache, ainsi qu'au demarrage

```batch
sudo /etc/init.d/apache2 stop
sudo systemctl disable apache2
```

Editer

```batch
nano /etc/nginx/sites-available/default
```
chercher la ligne

```batch
index index.html index.htm index.nginx-debian.html
```

Remplacer la par

```batch
index index.html index.htm index.php
```

modifier les ligne suivante

```batch
 #location ~ \.php$ {
 # include snippets/fastcgi-php.conf;
 #
 # # With php5-cgi alone:
 # fastcgi_pass 127.0.0.1:9000;
 # # With php5-fpm:
 # fastcgi_pass unix:/var/run/php5-fpm.sock;
 #}
```
 
 par

```batch
  location ~ \.php$ {
 include snippets/fastcgi-php.conf;
 fastcgi_pass unix:/var/run/php5-fpm.sock;
 }
```

modifier les droits

```batch
sudo chown -R www-data:pi /var/www/html/
sudo chmod -R 770 /var/www/html/
```

Redemarrer nginx

```batch
sudo /etc/init.d/nginx restart
```

Si message d'ereur mettre les droits comme ceux-ci

```batch
 chown www-data:www-data /var/www
 chmod 744 /var/www
 ```
 Se connecter sur mysql pour creer la base de donnée
 
 ```batch
 mysql -u <root> -p<password>
 ```
 
  Creer une base de donnée
 
 ```batch
 mysql -u <root> -p<password>
 create database wordpress;
 GRANT ALL PRIVILEGES ON wordpress.* TO <nom_utilisateur>@"localhost" IDENTIFIED BY <mot_de_passe>;
 ``` 
 
 Telecharger wordpress
 
 ```batch
cd var/www/html
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo chown -R www-data /var/www/html/wordpress
 ``` 
 
 ecrire le fichier suivant
 
  ```batch
/etc/nginx/sites-available/wordpress

server {
 listen 80;
 root /var/www/html/wordpress;
 index index.php;
 server_name <nom_de_votre_site> www.<nom_de_votre_site>;
 access_log /var/log/nginx/<nom_de_votre_site>.access_log;
 error_log /var/log/nginx/<nom_de_votre_site>.error_log
 notice;
 location / {
  try_files $uri $uri/ /index.php?$args;
 }
 location ~ \.php$
 {
  include snippets/fastcgi-php.conf;
  fastcgi_pass unix:/var/run/php5-fpm.sock;
 }
}
 ``` 
 
 Creer un lien symbolique
 
  ```Batch
 ln -s /etc/nginx/sites-available/<nom_de_votre_site> /etc/nginx/sites-enabled/<nom_de_votre_site>
 ``` 
 
 Redemarrer nginx

```batch
sudo /etc/init.d/nginx restart
```

Tapez l'adresse IP du raspberry pie dans un navigateur
