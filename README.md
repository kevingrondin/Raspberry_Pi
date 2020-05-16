# Linux

## Structure de dossier

| Dossier | Description  |
| --------| :----------- |
| /boot   | Contient le fichier qui est utilisé par le boot loader (grub.cfg) |
| /root   | Le repertoire de l'utilisateur root différent de / |
| /dev    | les périphériques |
| /etc    | configuration |
| /bin -> /usr/bin    | Les commandes quotidienne de l'utilisateur |
| /sbin -> /usr/sbin    | commandes système/sytèle de fichier |
| /opt    | application complémentaire optionnelle ( pas partie de l'OS ) |
| /proc   | processus en cour  ( n'existe qu'en mémoire ) |
| /lib -> /usr/lib    | Fichiers de librairie du langage C dont ont besoin les commandes et les applications `strace -e open pwd` |
| /tmp   | repertoire temporaires |
| /home   | repertoire utilisateur |
| /var   | log system |
| /run | Démons système qui démarrent très tôt (systemd, udev ) pour stocker les fichiers d'exécution temporaires comme les fichiers PID |
| /mnt | pour monter un système de fichiers externe |
| /media | CDROM |

## Installation (Raspberry)

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

## Mettre à jour le système

```batch
sudo su
apt update -y && apt upgrade -y && sudo apt full-upgrade -y 
apt install ntpdate
apt install ntp
apt-get install bash-completion
```

Changer le mot de passe

```Shell
passwd
```

Mettre a jour le Noyau

```Shell
yum update kernel -y
```

Activier l'auto-complétion

```Shell
nano /etc/bash.bashrc
```

Decommentez les lignes comme ci-dessous

```Bash
# enable bash completion in interactive shells
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

## Changer le hostname

```SHELL
cat /etc/hostname
hostnamectl set-hostname <newname>
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

Mettre les bon droits au fichier générés

```Shell
sudo chmod 600 ~/.ssh/id_rsa
sudo chmod 600 ~/.ssh/id_rsa.pub
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

## VSCODE server (ne fonctionne pas sur des architectures arm)

```Shell
sudo apt-get install libx11-dev libxkbfile-dev libsecret-1-dev fakeroot rpm
```

[code-server](https://github.com/cdr/code-server)

```Shell
docker run -it -p 127.0.0.1:8080:8080 -v "${HOME}/.local/share/code-server:/home/coder/.local/share/code-server" -v "$PWD:/home/coder/project" codercom/code-server:v2
```

**npm rebuild** dans vscode pour recompiler si des problèmes persistes

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

```Shell
apt install sysstat

# Regarder la crontab
cat /etc/cron.d/sysstat

# les fichiers générés
ls /var/log/sa

# CPU
sar -u

# Memoire
sar -r

# Swap
sar -W

# Etat des IO disque
sar -b

# Entree sortie 
sar -d

# Une période 
sadf -s 05:00:01 -e 06:00:01 -dT /var/log/sa/sa15 -- -A
```

## Installer Pyhton pour du machine learning

```Shell
curl -LO "https://repo.anaconda.com/miniconda/Miniconda2-4.7.12.1-Linux-x86_64.sh"
```

**path** correspond à notre repertoire de travail exemple : `/maboite/miniconda3`

```Shell
sh Miniconda2-4.7.12.1-Linux-x86_64.sh -b -p <path>
```

```Shell
conda install -y "jupyter" "numpy" "pandas"
```

## Intimité sous BATCH

```BATCH
history
```

supprimer une ligne génante 

```BATCH
history -d numeroligne
```

Tout nettoyer

```BATCH
history -c
```

Si je veux l'historique sans les ligne de commande ls -l

```BATCH
export HISTIGNORE='ls -l:pwd:history'
```

tout l'historique sont sauvegardé ici

`/home/yourname/.bash_history`

*ctrl+r* permet de faire une recherche dans les lignes de commande tapé

### System Monitoring

```Shell
df -h
top
dmesg
iostat 1
free
cat /proc/cpuinfo
cat /proc/meminfo
```

Installer `ncdu` avec apt pour visualiser les fichiers trop volumineux

ce placer à la racine
```shell
ncdu -x
```

Installer `tmpreaper` avec apt

> lors de l'installation avec apt, il faut commenter la ligne showing dans le fichier etc/tmpreaper

Analyser les log avec `lnav` installation simple avec apt

```Shell
lnav /var/log/messages*

journalctl | lnav

journalctl -f | lnav

journalctl -o short-iso | lnav

journalctl -o json | lnav

journalctl -a -o json | lnav

journalctl -o json --output-fields=MESSAGE,PRIORITY,_PID,SYSLOG_IDENTIFIER,_SYSTEMD_UNIT | lnav
```

### Suppression

Il peux arriver qu'un dossier comporte tellement de fichier que vider le dossier deviens impossible meme avec un rm

```Shell
find . -name "*.toto" -exec rm {} \;
```

### Rechercher 

Rechercher une chaine de caractère dans tout les fichiers du repertoire courant

```Shell
grep -rnw './' -e 'machainearechercher'
```

### Wildcard

Cette ligne de commande creer 9 fichiers

```Shell
touch abcd{1..9}-wyz
```

### Afficher un message à chaque connexion

Pour afficher un message il faudra l'écrire la dedans

/etc/motd

### Process and Jobs

systemctl
ps -aux
top
crontab -e pour creer un crontab
crontab -l pour lister les crontab

on peux definir plus facilement les cron dans /etc/

ls -l | grep cron

### Process Management

Crtl-z, jobs, bg, fg, nohup &, ps, pkill, nice 

### Bloquer les connexions utiliateurs

Pour afficher un message il faudra ce deplacer ici

> /etc/nologin

```Shell
cat nologin
```
Ecrire votre message

### Changer mot de passe

```Shell
passwd
```

### Commande sed

Remplacer tout les mots `ancien` par les mots `nouveau` du fichier `MonFichier` 

```Shell
sed 's/ancien/nouveau/g' MonFichier
```

Rajouter l'argument -i pour que cela soit actif

```Shell
sed -i 's/ancien/nouveau/g' MonFichier
```

Supprimer toutes les lignes dans le fichier contenant le mot `ancien` dans `MonFichier`

```Shell
sed '/ancien/d' MonFichier
```

Supprimer toutes les lignes vide dans `MonFichier`

```Shell
sed -i '/^$/d' MonFichier
```

### Tester le chiffrement TLS/SSL ou service en ligne (STARTTLS)

```BATCH
git clone –depth 1 https://github.com/drwetter/testssl.sh.git
cd testssl.sh
./testssl.sh https://qwant.com
```

### Tableau bord serveur

Surveillez son linux en temp réel sur l'adresse htt://127.0.0.1:19999

```BATCH
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

### Detecter le traffic malicieux

https://github.com/stamparm/maltrail

### Mise à jour

Mise a jour

```SHELL
apt-get update
apt-get upgrade
```

### Voir les paquets installé

```SHELL
dpkg -l | grep paquet_recherché
```

### Savoir si un repertoire est vide ou non
```Shell
$ if ./empty-dir /home/sk/ostechnix; then echo "It is empty" ; fi
It is empty
````

### Génère un mot de passe avec 15 charactère 

```Shell
./randpass -n 15
```

### Voir les dernière modif d'un fichier

```Shell
./since /var/log/apt/history.log
```

### Exécuter une commande sur une période

Exécuter un htop pendant 10h10m10s

```Shell
$ ./timeout -t 10:10:10 htop
```

### Système log monitor

Voir les log 

```SHELL
cd var/log

ls -ltr
ll | more
```

Voir les log de demarrage toujours dans `var/log`

```SHELL
more boot.log
```

voir quand est-ce que boot.log a été modifié

```SHELL
ls -l boot.log
```

voir les log du matériel
```SHELL
dmesg
```

### Utilisateur

Creer l'utilisateur `toto` et l'ajouter au group sudo

```SHELL
useradd toto
usermod -aG sudo toto
```

Supprimer un utilisateur

```SHELL
userdel -r toto
```

Chercher toto dans /etc/group

```SHELL
grep toto /etc/group
```

Creer un groupe TEST

```SHELL
groupadd TEST
```

Voir la liste de tout les groupes

```SHELL
cat /etc/group/
```

### rsync

Synchroniser deux repertoire sur un serveur local

```BATCH
rsync -zvr /var/opt/installation/inventory /root/temp
```

### Partage de fichier Windows

Creer un groupe sharing

```Shell
addgroup --system sharing
```

Creer un groupe utilisateur sharing appartenant au groupe sharing

```Shell
adduser --system sharing --ingroup sharing
```

Mot de passe

```Shell
passwd sharing
```

Creer un repertoire partagé

```Shell
mkdir /home/sharing/public
chmod 777 /home/sharing/public
```

Installer Samba

```Shell
apt update && apt install samba -y

sudo rm /etc/samba/smb.conf
sudo nano /etc/samba/smb.conf
```

Copier dans le fichier

```Shell
[global]
        netbios name = server
        server string = server
        workgroup = WORKGROUP
        security = user

[server]
        comment = server
        path = "/home/sharing/public"
        public = yes
        guest ok = yes
        read only = no
```

```Shell
sudo pdbedit -a -u guest
sudo service smbd restart
```

### Exécuter une ligne de commande sur une liste de serveur

On recupére l'espace disque sur tout les serveurs

```Shell
sudo apt-get install python-pip
sudo pip install pssh

pssh -h pssh-hosts -l root -A -i "df -hT"
```

### Recupération de disque dur

Recupérer [dd_rescue](http://www.garloff.de/kurt/linux/ddrescue/)

Décompressez, puis make pour créer le binaire

Si le disque enfommagé est /dev/sdb1
Si le disque libre est /dev/sdc1

Information du disque

```BATCH
fdisk -l
```

Se placer dans le repertoire dd_rescue

```BATCH
dd_rescue -l transfert_errors.log /dev/sdb1 /dev/sdc1
```

Le fichier log va être créé et qui va lister les blocs qu'il n'a pas plus traiter

Choisisser un des lignes de commande ci-dessous selon votre système de fichiers
```BATCH
fsck.ext2
fsck.ext3
fsck.ext4
fsck.vfat
```

## WSL 2

### Windows

PowerShell en admin

```Shell
Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName VirtualMachinePlatform
Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName Microsoft-Windows-Subsystem-Linux
Restart-Computer

wsl --list 
wsl --set-version Ubuntu 2
wsl ~ -d Ubuntu
```

Installer [VcXsrv](https://sourceforge.net/projects/vcxsrv/) choisir **One Large Windows**

Choisir une distribution

- Ubuntu 16.04 LTS
- Ubuntu 18.04 LTS
- OpenSUSE Leap 15
- OpenSUSE Leap 42
- SUSE Linux Enterprise Server 12
- SUSE Linux Enterprise Server 15
- Kali Linux
- Debian GNU/Linux
- Fedora Remix pour WSL
- Pengwin
- Pengwin Enterprise
- Alpine WSL

### WSL

```Shell
sudo apt-get install xfce4
```

Lancer

```Shell
xfce4-session --display=:0.0
```

Ouvrir l'explorateur windows 

```Shell
explorer.exe .
```

## CRON

chronic execute une commande en silence sauf en cas d'échec, Il est utile pour les travaux de cron. Au lieu d'essayer de garder la commande silencieuse, et d'avoir à traiter des mails contenant des sorties accidentelles quand elle réussit, et pas assez verbeux quand elle échoue, vous pouvez simplement l'exécuter toujours verbalement, et utiliser Chronic pour cacher la sortie réussie.

Exemple :

Lors de la création d'un nouveau cron job, au lieu d'utiliser la ligne suivante ;

```CRON
#0 1 * * * backup >/dev/null 2>&1
0 1 * * * chronic backup
```
