# Raspberry

## Carte SD

#### Formatter la SD

La préparation de la carte SD commence par un formatage en *FAT32* [SD Formatter](https://www.sdcard.org/downloads/formatter_4/index.html)

#### Preparer la SD

Documentation non écrite

## Reseaux

#### Assigner une adresse IP Fixe

Dans le fichier */etc/network/interfaces*

```batch
iface eth0 inet static
address 192.168.1.69
netmask 255.255.255.0
gateway 192.168.1.1
```

## Systeme

#### Mettre à jour le système

```batch
sudo apt-get update -y && sudo apt-get upgrade -y && sudo reboot
```

#### Installer nodejs

```batch
wget http://node-arm.herokuapp.com/node_latest_armhf.deb
dpkg -i node_latest_armhf.deb
```

#### Desactiver apache, ainsi qu'au demarrage

```batch
sudo /etc/init.d/apache2 stop
sudo systemctl disable apache2
```

## Web

#### Nginx

Installation

```batch
sudo apt-get install nginx
sudo service nginx start
```

Configuration

```batch
cd /etc/nginx
sudo rm site-enabled/default
cd sites-available
sudo nano myapp
```

## Monitoring

```batch
npm install pm2 -g
```

demander a pm2 de ce lancer au demarrage du raspberry

```batch
pm2 startup
```

rentrer les information de votre bucket

```batch
pm2 link [Cle Publique] [Cle secrète]
```

monitorer le serveur complet

```batch
pm2 install pm2-server-monit
```
