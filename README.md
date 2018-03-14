# Raspberry

### Formatter la SD

La préparation de la carte SD commence par un formatage en *FAT32* [SD Formatter](https://www.sdcard.org/downloads/formatter_4/index.html)

### Preparer la SD

### Assigner une adresse IP Fixe

Dans le fichier */etc/network/interfaces*

```batch
iface eth0 inet static
address 192.168.1.69
netmask 255.255.255.0
gateway 192.168.1.1
```

```batch
sudo aptitude update -y && sudo aptitude upgrade -y && sudo reboot
```

Desactiver Apache, ainsi qu'au demarrage

```batch
sudo /etc/init.d/apache2 stop
sudo systemctl disable apache2
```


