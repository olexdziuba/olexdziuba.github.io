---
title: "Installation de deux sites web sur UBUNTU en utilisant APACHE"
date: 2021-06-19
header:
excerpt: "System administration"
mathjax: "true"
--- 

## Installation de deux sites web sur UBUNTU en utilisant APACHE

Pour l'installation je vais utiliser Ubuntu 18.04 installé sur  VMWare Workstation, carte réseau je vais configurer en mode bridge pour avoir accès à partir de MobaXterm. Pour vérifier l'accès aux sites web je vais utiliser Raspberry Pi qui est dans le même réseau.  Pour installer quelques sites sur un serveur on va configurer les virtual hosts sur Apache.

Premièrement il faut  faire une mise à jour de votre système:

*olex@ubuntu:\~\$ sudo apt update && sudo apt upgrade -y*

<img src="/images/ubuntu_apache/image20.png">
<img src="/images/pi_hole/pihole_img16.png">

### Installation d'apache

*olex@ubuntu:\~\$ sudo apt-get install apache2*

![](/images/ubuntu_apache/image1.png)

Pour installer quelques sites web sur le même serveur il faut créer quelques dossiers. Je vais créer 2: site 1 et site 2.

*olex@ubuntu:\~\$ sudo mkdir -p /var/www/site1.com/public\_html*

*olex@ubuntu:\~\$ sudo mkdir -p /var/www/site2.com/public\_html*

![](/images/ubuntu_apache/image16.png)

 Il faut changer droit d'accès pour curent user

*olex@ubuntu:\~\$ sudo chown -R \$USER:\$USER /var/www/site1.com/public\_html*

*olex@ubuntu:\~\$ sudo chown -R \$USER:\$USER /var/www/site2.com/public\_html*

![](/images/ubuntu_apache/image7.png)

 Pour faire les sites accessibles aux utilisateurs dans le navigateur, vous devez autoriser la lecture des fichiers dans ces répertoires

*olex@ubuntu:\~\$ sudo chmod -R 755 /var/www*

![](/images/ubuntu_apache/image4.png)

On va créer deux  *index.html* pour faire la vérification et bien identifier chaque site.

*olex@ubuntu:\~\$ sudo vim /var/www/site1.com/public\_html/index.html*

![](/images/ubuntu_apache/image22.png)

Et ajouter le texte suivant:

 *\<html\>*  
    *\<body\>*  
          *This is our site1.com website*  
    *\</body\>*  
  *\</html\>*  

![](/images/ubuntu_apache/image19.png)

Pour *site2.com* il faut faire même chose:

*olex@ubuntu:\~\$ sudo vim /var/www/site2.com/public\_html/index.html*

![](/images/ubuntu_apache/image8.png)

 

Et ajouter le texte suivant:

*\<html\>*  
   *\<body\>*  
       *This is our site2.com website*  
   *\</body\>*  
  *\</html\>*  

![](/images/ubuntu_apache/image14.png)

Création de nouveaux  virtual hosts files.

Après l'installation, Apache crée le configuration file par défaut dans le dossier */etc/apache2/sites-available/000-default.conf*

On va copier se file pour chaque site avec extension *.conf*

*olex@ubuntu:\~\$ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/site1.com.conf*

![](/images/ubuntu_apache/image9.png)

*olex@ubuntu:\~\$ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/site2.com.conf*

![](/images/ubuntu_apache/image12.png)

On va configurer ses files:

*olex@ubuntu:\~\$ sudo vim /etc/apache2/sites-available/site1.com.conf*

![](/images/ubuntu_apache/image23.png)

Il faut ajouter/corriger l'information suivante:

    *ServerAdmin admin@site1.com*  
    *ServerName site1.com*  
    *ServerAlias www.site1.com*  
    *DocumentRoot /var/www/site1.com/public\_html*  

   

![](/images/ubuntu_apache/image5.png)

Pour *site2.com* il faut faire même chose

olex@ubuntu:\~\$ sudo vim /etc/apache2/sites-available/site2.com.conf

![](/images/ubuntu_apache/image3.png)

Il faut ajouter/corriger l'information suivante:

    *ServerAdmin admin@site2.com*  
    *ServerName site2.com*  
    *ServerAlias www.site2.com*  
    *DocumentRoot /var/www/site2.com/public\_html*  

![](/images/ubuntu_apache/image21.png)

Nous avons créé des fichiers de configuration pour les hôtes virtuels. Maintenant, vous devez les activer:

*sudo a2ensite site1.com.conf*  
*sudo a2ensite site2.com.conf*  

![](/images/ubuntu_apache/image6.png)



Restart Apache

*olex@ubuntu:\~\$ sudo service apache2 restart*

![](/images/ubuntu_apache/image11.png)

#### Vérification de firewall

Vous devez vérifier que votre pare-feu autorise le trafic HTTP et HTTPS:

*olex@ubuntu:\~\$ sudo ufw app list*

![](/images/ubuntu_apache/image10.png)

Vérification du profil Apache Full, il devrait autoriser le trafic pour les ports 80 et 443 :

*olex@ubuntu:\~\$ sudo ufw app info "Apache Full"*

![](/images/ubuntu_apache/image15.png)

### Vérification d'apache

Pour vérifier Apache  il faut taper adresse IP dans votre browser

![](/images/ubuntu_apache/image17.png)

Si vous voyez cette page, votre serveur Web est correctement installé et accessible via un pare-feu.

### Vérification  d'accès de site1.com et site2.com

Parce que ses sites sont créés pour le test et on n'a pas enregistré les domain names, pour vérification on va configurer *hosts* file d'ordinateur local. Si vous utiliser Windows:

*C:\\windows\\system32\\drivers\\etc\\hosts*

Pour Linux il faut corriger:

*/etc/hosts*

Il faut ajouter:

*xxx.xxx.xxx.xxx site1.com*  
*xxx.xxx.xxx.xxx site2.com*  

(*xxx.xxx.xxx.xxx* adresse IP votre serveur web)

Dans mon cas cela:

*192.168.0.49 site1.com*  
*192.168.0.49 site2.com*  

![](/images/ubuntu_apache/image13.png)

Et les résultats:

![](/images/ubuntu_apache/image18.png)

et

![](/images/ubuntu_apache/image2.png)
