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
sudo su
apt update -y && apt upgrade -y && sudo apt full-upgrade -y && sudo reboot
```

> acces à distance

```batch
sudo apt install -y xrdp
```

> acces depuis windows et mac (rajouter .local pour mac) avec mstsc
```batch
sudo apt install -y samba samba-common-bin
```

> configurer samba
```batch
mkdir /home/shares/public
chown -R root:users /home/shares/public
chmod -R ug=rwx,o=rx /home/shares/public
nano /etc/samba/smb.conf
```

> Rajouter la ligne 
security = user

sous la ligne
*##### Authentification ####*

> chercher *\[homes]*

Mettre read only à no

> Ecrire tout en bas

\[public]
  comment= Public Storage
  path = /home/shares/public
  valid users = @users
  force group = users
  create mask = 0660
  directory mask = 0771
  read only = no

> Redemarrer samba

```batch
/etc/init.d/samba restart
```

#### Installer nodejs

```batch
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g npm
```

#### Desactiver apache, ainsi qu'au demarrage

```batch
sudo /etc/init.d/apache2 stop
sudo systemctl disable apache2
```

## Web

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
