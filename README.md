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

```batch
  location ~ \.php$ {
 include snippets/fastcgi-php.conf;
 fastcgi_pass unix:/var/run/php5-fpm.sock;
 }
  ```
