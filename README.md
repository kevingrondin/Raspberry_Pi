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

Creer votre clée public, et votre clée privée, laissez les double côte

```Shell
# Version secure
ssh-keygen -t rsa -b 4096 -C "email_address"

# Version basic
ssh-keygen -t rsa -C "email_address"
```

Pour tester votre clée avec github, une fois votre clée publique sur github

```Shell
ssh -T git@github.com
```

Sous windows

```Shell
type C:\Users\name\.ssh\id_rsa.pub
```

Envoyé la clée OPENSSH au rasp

```Shell
cat ~/.ssh/id_rsa.pub | ssh <USERNAME>@<IP-ADDRESS> 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

Autoriser les authorized_keys dans sshd_config

```Shell
nano /etc/ssh/sshd_config
```

et decommentez

```Bash
AuthorizedKeysFile   /.ssh/authorized_keys
```

et recharger la config ssh

```Bash
/etc/init.d/ssh force-reload
```

vous pouvez vous connectez avec cette ligne de commande

```Shell
ssh -i /root/.ssh/<KEY> <USERNAME>@<IP_ADRESSE>
```

Marche aussi avec Rsync

```Shell
rsync -e « ssh -i /root/.ssh/<KEY> » -av <USERNAME>@<IP_ADRESSE>:/source/destination/
```

Si vous êtes sur windows vous pouvez utiliser **Pageant** et **wsl-ssh-pageant-amd64-gui** pour ne plus à avoir entrer les clée privé à chaque fois

Dans votre dossier .ssh creer un fichier *config* sans extension, vous pouvez créer des alias pour vous connectez plus rapidement

```Bash
Host *
   ForwardAgent yes
        
Host <ALIAS>
   HostName <SERVEUR>
   Port     <PORT>
   User     <USER>
   # Tunnel SSH
   # ProxyCommand     ssh <USER>@<SERVEUR> -W %h:%p
```

Sur unix dans votre dossier **home**

```Bash
nano ~/.ssh/config
chmod 600 ~/.ssh/config
chown $USER ~/.ssh/config
```

dans *~/.ssh* creer un fichier config de la même façon que pour windows pour les racourcis

### Acces à distance

#### Bureau à distance

```Shell
sudo apt install -y xrdp
```

acces depuis windows et mac (rajouter .local pour mac) avec mstsc

#### Serveur de fichier Simple

```Shell
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

#### Serveur de fichier Avancé

Installation de samba

```Shell
sudo apt install -y samba
```

On créer un groupe

```Shell
groupadd <group>
```

On créer un <user> dans le groupe <group>

```Shell
useradd -g <group> <user>
```

On lui donne un mot de passe pour samba

```Shell
smbpasswd -a <user>
```

On créer un dossier partagé avec les droit adéquat

```Shell
mkdir -p /<partage>/<groupe>
chown -R root:<groupe> /<partage>/<groupe>/
chmod -R 770 /<partage>/<groupe>/
```

On historise la configuration par defaut de samba

```Shell
mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

```Bash
vi /etc/samba/smb.conf
 
# Ajouter la configuration ci-dessous
 
#======================= Global Settings =======================
 
[global]
workgroup = WORKGROUP
server string = %h
public = yes
security = user
encrypt passwords = true

#======================= Share Definitions =======================

[<group>]
comment = répertoire du <groupe>
  path = <partage>/<groupe>
  valid users = @<groupe>
  writable = yes
  directory mask = 0770
  create mask = 0770
  force create mode = 0770
```

dans **valid users** on écri *@groupe1, @groupe2* et *user1, user2*

On test si tout es bon

```Shell
testparm smb.conf
```

On redemarre si tout est bon 

```Shell
/etc/init.d/samba restar
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

#### Installer Docker

```Shell
curl -sSL get.docker.com | sh
```

## Web

## Monitoring

### PM2

```batch
npm install pm2 -g
```

demander a pm2 de ce lancer au demarrage du raspberry

```batch
pm2 startup
```

monitorer le serveur complet

```batch
pm2 install pm2-server-monit
```

### Netdata

Supervision en temp réel

```Bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```
### ncdu

Installer ncdu avec apt pour visualiser les fichiers trop volumineux

ce placer à la racine

```Shell
ncdu -x
````

### tmpreaper

Installer tmpreaper avec apt

lors de l'installation avec apt, il faut commenter la ligne showing dans le fichier etc/tmpreaper

### lnav

Analyser les log avec lnav installation simple avec apt

```Shell
lnav /var/log/messages*

journalctl | lnav

journalctl -f | lnav

journalctl -o short-iso | lnav

journalctl -o json | lnav

journalctl -a -o json | lnav

journalctl -o json --output-fields=MESSAGE,PRIORITY,_PID,SYSLOG_IDENTIFIER,_SYSTEMD_UNIT | lnav
```

### Si on utilise pas journalctl, mais les log normale

**logrotate** est déja présent sur les distribution linux, il permet d'effecctuer des actions de rotations et de compression sur les logs


Assurons-nous que cette ligne n'est pas commentez

```Shell
cat /etc/logrotate.conf | grep "include"
```

Voici un exemple

```Bash
/var/log/mylogs/auth.log {
     su <user> <group>
     monthly
     rotate 3
     compress
     missingok
     create 644 <user> <group>
}
 
/var/log/mylogs/errors.log {
     su <user> <group>
     monthly
     rotate 3
     compress
     missingok
     create 644 <user> <group>
}
```

1. **monthly** : la rotation se fait mensuellement
1. **rotate 3** : le nombre de fichiers qu’on souhaite conserver
1. **compress** : les anciens fichiers sont compressés
1. **missingok** : ne considère pas l’absence du fichier comme une erreur
1. **create 644 root root** : créer le fichier de log immédiatement après la rotation avec les droits adéquats

Petite verification

```Shell
logrotate --force /etc/logrotate.d/mylogs
```

### ecouter le traffic

```Shell
iftop
```

```Shell
nethogs
```

```Shell
iptraf-ng
```


