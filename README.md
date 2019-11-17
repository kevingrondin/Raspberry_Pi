# Raspberry

> Configurer un raspberry

## Carte SD

Télécharger l'image [ici](https://www.raspberrypi.org/downloads/raspbian/) la version light est suffisante

### Formatter la SD

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

### Preparer la SD

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

## Fixer une adresse IP

```SHELL
sudo nano /etc/dhcpcd.conf
```

```BASH
interface wlan0
static ip_address=192.168.1.2/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

## Mettre le clavier en français

```SHELL
sudo raspi-config
```

changer les options de locatisations (FR UTF8)

## Systeme

#### Mettre à jour le système

```batch
sudo su
apt update -y && apt upgrade -y && sudo apt full-upgrade -y 
apt install ntpdate
apt install ntp
```

Changer le mot de passe

```Shell
passwd
```

## Connexion avec clée SSH

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

### acces à distance

```batch
sudo apt install -y xrdp
```

> acces depuis windows et mac (rajouter .local pour mac) avec mstsc
```batch
sudo apt install -y samba
```

> configurer samba
```batch
mkdir /home/pi/shares
chmod 777 /home/shares/public

sudo rm /etc/samba/smb.conf
sudo nano /etc/samba/smb.conf
```

```Bash
[global]
        netbios name = server
        server string = server
        workgroup = WORKGROUP
        security = user

[server]
        comment = server
        path = "/home/pi/shares"
        public = yes
        guest ok = yes
        read only = no
```

```Shell
sudo pdbedit -a -u pi
sudo service smbd restart
```
#### Installer nodejs

Façon Node

```batch
curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g npm
```

Façon linux

```batch
//verfier si arm7 ou plus
cat /proc/cpuinfo

// aller sur navigateur web chercher la bonne installation
wget https://nodejs.org/dist/...
// decompresser et supprimer le fichier compresser aussi
tar xf node-v...

// renomer en node
mv node-v... node

// creer les lien 
ln -s /var/www/node/bin/node /usr/sbin
ln -s /var/www/node/bin/npm /usr/sbin
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
