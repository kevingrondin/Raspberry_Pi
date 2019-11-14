# Raspberry

> Configurer un raspberry

## Si vous êtes sur windows 10

Creer votre clée public, et votre clée privée

```Shell
ssh-keygen -t rsa -b 4096 -C "name@email.com"
```

Affiche votre clée public, à copier dans les settings de github

```Shell
type C:\Users\name\.ssh\id_rsa.pub
```

Configurer git

```Shell
git config --global user.name "Your Name"
git config --global user.email your_email@users.noreply.github.com
```

Envoyé la clée OPENSSH au rasp

```Shell
cat ~/.ssh/id_rsa.pub | ssh <USERNAME>@<IP-ADDRESS> 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

## Carte SD

Télécharger l'image [ici](https://www.raspberrypi.org/downloads/raspbian/) la version light est suffisante

#### Formatter la SD

Formater depuis un windows avec l'invite de commande

```BATCH
diskpart
list disk
select disk
clean
create partition primary
active
format fs=exfat quick
assign
exit
exit
```

#### Preparer la SD

Avec rufus ou [etcher](https://etcher.io) 

### Configurer le wifi et ssh en avance

créer un fichier à la racine `ssh` et en même temps `wpa_supplicant.conf` avec ces infos

```SHELL
country=fr
update_config=1
ctrl_interface=/var/run/wpa_supplicant

network={
 scan_ssid=1
 ssid="nameofnetworknotnetwork5GHZ"
 psk="motdepasse"
}
```

Ecrire ces informations en **EOL Unix** avec Notepad++ dans le menu *Edit/EOL Conversion*, les réseau 5GHZ ne fonctionneras pas.
Aprés chaque redemarrage la configuration seras supprimé

### Mettre le clavier en français

```SHELL
sudo raspi-config
```

laisser Generic 105

Choisir French suivant jusqu'a la fin

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
chmod -R ug=rwx,u=rwx,o=rwx /home/shares/public
nano /etc/samba/smb.conf
```

> Rajouter la ligne 
security = user


sous la ligne
*##### Networking ####*

veillez à laisser decomanter les lignes

> interfaces = 127.0.0.0/8 eth0

et

> bind interfaces only = yes


sous la ligne
*##### Authentification ####*

> chercher *\[homes]*

Mettre read only à no

> Ecrire tout en bas

```batch
[public]
  comment= Public Storage
  path = /home/shares/public
  writable = yes
  guest ok = yes
  valid users = @users
  force group = users
  create mask = 0660
  directory mask = 0771
  read only = no
```

> Redemarrer samba

```batch
/etc/init.d/samba restart
```

> Creer l'utilisateur Pi

```batch
sudo smbpasswd -a pi
```

#### Installer nodejs

```batch
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g npm
```

#### Desactiver apache, ainsi qu'au demarrage

```shell
sudo /etc/init.d/apache2 stop
sudo systemctl disable apache2
```

#### Désinstaller LibreOffice

```shell
sudo apt-get remove --purge libreoffice*
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
