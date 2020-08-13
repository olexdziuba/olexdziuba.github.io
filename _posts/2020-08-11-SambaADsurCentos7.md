---
title: "Installation et configuration de serveur Samba-AD sur CentOS7"
date: 2020-08-11
tags: [Samba, CentOS, Samba-AD, Linux]
header:
  image: "/images/dc.jpg"
excerpt: "System administration"
mathjax: "true"
--- 

------------------------

1.Préparation de CentOS7 
------------------------

Configuration:

 `dc1.domaine.lan`

administrateur CentOS:`root`

administrateur SAMBA: `administrator`

Configuration réseau:

        CentOS: eth0: `NAT, DHCP (accès internet)`

                    `eth1: 192.168.64.1/24 (interne)`

        Windows7: eth0: `192.168.64.10/24 (interne)`

1.1 Changement de hostname: 
------------------------

-   modifier le fichier `/etc/hostname` et changer hostname: 
     `vim  /etc/hostname`
-   vérifier: `cat /etc/hostname`

<img src="/images/samba_centos7/image2.png">

-   changer `/etc/hosts: vim /etc/hosts`

 (ne modifie pas les lignes contenant localhost )

 <img src="/images/samba_centos7/image10.png">



* * * * *

1.2 Configurer l’adresse IP (statique) 
--------------------------------------

J’utilise:

eth0 (ens32 pour vmware) - DHCP pour avoir accès à l'internet

`/etc/sysconfig/network-scripts`

 <img src="/images/samba_centos7/image8.png">

eth1 (ens33 pour vmware) static

`ip 192.168.64.1/24`

`/etc/sysconfig/network-scripts`

 <img src="/images/samba_centos7/image12.png">

Pas  proxy!

Pour configurer le réseau on peut aussi  utiliser `nmtui`

Pour vérification de la configuration: `ip a`

1.3 Désactivation de SELinux 
-------------------------

Vérifier que SELinux est désactivé :

`vim /etc/selinux/config`

`selinux=disabled`

 <img src="/images/samba_centos7/image16.png">

Faire `reboot`.

 Vérifier status SELinux avec commande

 `sestatus`

 <img src="/images/samba_centos7/image9.png">

Activer le mode routeur et le NAT 
---------------------------------

Pour ajouter temporairement :

`sysctl -w net.ipv4.ip\_forward=1`

Pour faire le changement permanent, il faut ajouter dans le file:

`vim /usr/lib/sysctl.d/50-default.conf`
`net.ipv4.ip\_forward = 1`

 <img src="/images/samba_centos7/image3.png">

pour appliquer le changement:

`/sbin/sysctl -p`

or `reboot`

#### Enable NAT

Le nœud interne doit maintenant pouvoir accéder à l'Internet public via le serveur de passerelle.

IP masquerading doit être activé avec iptables (utiliser bon IP et carte réseau).

`firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s 192.168.64.0/24`
 
`firewall-cmd --reload`
 
 <img src="/images/samba_centos7/image14.png">


Pour vérification (après avoir installé  Windows et joint une machine au domaine):

`ping 8.8.8.8` 
ou 
`ping 1.1.1.1`

 <img src="/images/samba_centos7/image15.png">

Installation de Samba-AD 
---------------------

### Configurer les règles de pare-feu (plus d'info [ici](https://wiki.samba.org/index.php/Samba_AD_DC_Port_Usage)) : 

`systemctl start firewalld`

`systemctl enable firewalld`

`firewall-cmd --zone=public --add-port=53/tcp --add-port=53/udp --permanent`

`firewall-cmd --zone=public --add-port=88/tcp --add-port=88/udp --permanent`

`firewall-cmd --zone=public --add-port=135/tcp --permanent`
`firewall-cmd --zone=public --add-port=389/tcp --add-port=389/udp --permanent`

`firewall-cmd --zone=public --add-port=445/tcp --permanent`
`firewall-cmd --zone=public --add-port=464/tcp --add-port=464/udp --permanent`
 
`firewall-cmd --zone=public --add-port=636/tcp --permanent`

`firewall-cmd --zone=public --add-port=3268/tcp --permanent`

`firewall-cmd --zone=public --add-port=3269/tcp --permanent`

`firewall-cmd --zone=public --add-port=50000-51000/tcp --permanent`

`firewall-cmd --zone=public --add-port=49152-65535/tcp --permanent`

`systemctl restart firewalld`

Pour vérification:

`firewall-cmd --list-ports`

### Désactiver avahi-daemon (protocol mDNS / bonjour) : 

`systemctl stop avahi-daemon.service avahi-daemon.socket`

`systemctl disable avahi-daemon.service avahi-daemon.socket`

### Ajouter repo EPEL 

`yum update -y`

`yum install -y epel-release`

`yum install -y wget sudo screen nmap telnet tcpdump rsync net-tools`

`bind-utils htop`

### Récupérer les paquets nécessaires 

Récupérer la clé de signature RPM et configuration d’un dépôt YUM :

`wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-TISSAMBA-7  http://samba.tranquil.it/RPM-GPG-KEY-TISSAMBA-7
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-TISSAMBA-7`
 
`rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-TISSAMBA-7`

`echo "[tis-samba]`

`name=tis-samba`

`baseurl=http://samba.tranquil.it/centos7/samba-4.11/`

`gpgcheck=1`

`gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-TISSAMBA-7" > /etc/yum.repos.d/tissamba.repo`

### Vérifier l’emprunte de la clef avec sha256sum : 

`sha256sum /etc/pki/rpm-gpg/RPM-GPG-KEY-TISSAMBA-7
b3cd8395e3d211a8760e95b9bc239513e9384d6c954d17515ae29c18d32a4a11  /var/www/samba/RPM-GPG-KEY-TISSAMBA-7`

### Installer les paquets Samba-AD pour CentOS : 

`yum install -y samba samba-winbind samba-winbind-clients krb5-workstation ldb-tools bind chrony bind-utils samba-client python3-crypto`

Instancier le domaine Active Directory Samba 
--------------------------------------------

### Configurer Kerberos 

Modifier le fichier /etc/krb5.conf et remplacer tout son contenu par les 4 lignes suivantes en précisant le domaine Active Directory de votre organisation (ici  dc1.domaine.lan):

`[libdefaults]`

  `default\_realm = DOMAINE.LAN`
  `dns\_lookup\_kdc = false`
  `dns\_lookup\_realm = false`
  
`[realms]`
 ` DOMAINE.LAN = {`
 
  `kdc = 127.0.0.1`
  
  `}`
 

### Configurer Samba 

#### Effacer le fichier /etc/smb/smb.conf s’il a déjà été généré 

(il sera régénéré par la commande d’instanciation) :

   
`rm -f /etc/samba/smb.conf`

#### Configurer Samba avec le rôle de contrôleur de domaine. 

Dans la ligne qui suit, vous penserez à changer à la fois le nom du
royaume kerberos, et le nom court du domaine (nom netbios):

`samba-tool domain provision --realm=DOMAINE.LAN --domain DOMAINE--server-role=dc`

#### Réinitialiser le mot de passe administrator : 

`samba-tool user setpassword administrator`

#### Vérifier la ligne dns forwarder = xxx.xxx.xxx.xxx.dans votre fichier /etc/samba/smb.conf 

`vim /etc/samba/smb.conf`

Elle doit pointer vers un serveur DNS valide, par ex. :

`dns forwarder = 1.1.1.1`


 <img src="/images/samba_centos7/image4.png">

#### Reconfigurer la résolution DNS pour la machine locale.

 Dans le fichier /etc/sysconfig/network-scripts/ifcfg-xxxx de l’interface réseau, remplacer la ligne suivante :
 `DNS1=127.0.0.1`

 <img src="/images/samba_centos7/image17.png">

Relancer NetworkManager pour prendre en compte les changements

`systemctl restart NetworkManager`

#### Il faut supprimer /var/lib/samba/private/krb5.conf et le remplacer par un lien symbolique vers le fichier /etc/krb5.conf : 

 `rm -f /var/lib/samba/private/krb5.conf`
  
`ln -s /etc/krb5.conf /var/lib/samba/private/krb5.conf`

#### Activer Samba pour qu’il démarre automatiquement au prochain reboot : 
`systemctl enable samba`
 
`systemctl start samba`

#### Redémarrer la machine 

`reboot`

 

#### Tester le kerberos 

Taper le mot de passe du compte administrator que vous avez défini ci-dessus avec la commande `samba-tool setpassword` 

Si ça ne renvoie rien ou que vous obtenez un message concernant l’expiration du mot de passe, c’est que c’est bon:

`kinit administrator`
 
`klist`

 <img src="/images/samba_centos7/image11.png">

#### Tester les DNS : 

`dig @localhost google.com`

`dig @localhost srvads.domaine.lan`

`dig -t SRV @localhost \_ldap.\_tcp.domaine.lan`

 <img src="/images/samba_centos7/image13.png">

Joindre une machine au domaine, installer RSAT 
-------------------------------------------------

#### Installer  Windows 7 

Il faut installer Windows ( Windows 7), configuration réseau: static, même range que CentOS7. (192.168.64.10/24)

#### Joindre Windows au domaine 

Vous pouvez désormais joindre un poste client Windows dans votre nouveau domaine. Ajouter le nom de l’ordinateur et du domaine.Il faut utiliser l'administrateur de Samba (administrator) et password.

 <img src="/images/samba_centos7/image7.png">

Redémarrer Windows, après redémarrage vous pouvez entrer comme administrateur Samba:

 <img src="/images/samba_centos7/image6.png">

#### Installer RSAT

Pour gérer votre nouveau domaine, il faut installer les interfaces de management sur un poste Windows. La ligne de commande Samba est efficace pour de nombreuses tâches d’administration, et les interfaces graphiques RSAT sont un bon complément à la ligne de commande.

Les étapes d'installation:

-   Télécharger le pack d’outils [depuis le site officiel de
    Microsoft](https://www.microsoft.com/fr-fr/download/details.aspx?id%3D7887&sa=D&ust=1597182781659000&usg=AOvVaw1fdTseuIF4KUVgIaHCZeot).
-   Installer RSAT
-   Dans Panneau de configuration -\> Programmes et Fonctionnalités     Windows, cliquez sur le lien à droite de la fenêtre Activer ou     désactiver des fonctionnalités Windows. Vous pouvez sélectionner:

 <img src="/images/samba_centos7/image1.png">

#### Vérification finale 

Une fois RSAT installé à partir de MMC vous pouvez avoir accès:

-   créer et supprimer un enregistrement DNS à partir de la console DNS     Active Directory ;
-   créer et supprimer un compte utilisateur ou un compte machine à     partir de la console Utilisateurs et Ordinateurs Active Directory ;
-   créer une nouvelle GPO ;

 <img src="/images/samba_centos7/image5.png">

Super, si vous êtes parvenu jusqu’à cette étape, c’est que  tout s'est passé bien  et que vous avez un nouveau domaine Samba Active Directory opérationnel.

* * * * *

 

Mes sources d’inspiration: 
--------------------------

[How to Install and Configure Samba on CentOS 7](https://linuxize.com/post/how-to-install-and-configure-samba-on-centos-7/)

[Installer et configurer Samba-AD sur CentOS7](https://dev.tranquil.it/samba/fr/samba_config_server/centos7/server_install_samba_centos.html#)

[How to Configure CentOS 7 Network Settings](https://www.serverlab.ca/tutorials/linux/administration-linux/how-to-configure-centos-7-network-settings/)

[Installer les outils d’administration (RSAT) sous Windows 7](http://www.oameri.com/installer-outils-dadministration-rsat-windows-7/#:~:text=En%20premier%20lieu%2C%20t%C3%A9l%C3%A9charger%20le,Next%2C%20cela%20lancera%20le%20t%C3%A9l%C3%A9chargement).

[Installer Samba 4 – Active Directory sous Linux](https://www.jjworld.fr/active-directory-linux-avec-samba-4/)

[Samba AD DC Port Usage](https://wiki.samba.org/index.php/Samba_AD_DC_Port_Usage)

[Configuring a CentOS 7 Kerberos KDC](https://www.theurbanpenguin.com/configuring-a-centos-7-kerberos-kdc/)

 


