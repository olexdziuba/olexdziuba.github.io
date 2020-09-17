---
title: "Installation et configuration d'un serveur OpenVPN sur UBUNTU (partie 2/2)"
date: 2020-09-07
tags: [OpenVPN, Ubuntu, VPN, Linux]
header:
  image: "/images/dc.jpg"
excerpt: "System administration"
mathjax: "true"
--- 

================================


On continue la configuration d’OpenVPN lequel on a commencé dans la [première partie](https://olexdziuba.github.io/OpenVPNsurUBUNTU-1/). Dans cet étape on va créer le deuxième et le troisième clients manuellement. Le deuxième client on va utiliser pour connecter le téléphone portable au réseau VPN, et le troisième client on va utiliser pour connecter l'ordinateur avec Linux dans le réseau VPN en utilisant une ligne de commande. Après, parce que les 2 ordinateurs (Windows et Linux) vont être dans le même réseau virtuel on va utiliser l'ordinateur Windows pour accéder à l'ordinateur Linux en ssh et sftp. 

### Configuration manuelle pour les clients

Vous pouvez télécharger le fichier [client2.ovpn](https://github.com/olexdziuba/vpn/blob/master/client2.ovpn) de mon GitHub sur votre ordinateur. Aussi il faut télécharger les fichiers suivants de votre serveur (si vous avez suivi mes instructions au post sur mon blog, il sont dans \~/openvpn-ca/keys)  en utilisant sftp/ftp:

-   ca.crt
-   client2.crt
-   client2.key
-   ta.key

<img src="/images/openvpn2/image2.png">

Ensuite, vous devez copier les informations des fichiers téléchargés depuis le serveur (ca.crt, client2.crt, client2.key, ta.key) dans les champs correspondants de fichier client2.ovpn en suivant les instructions du fichier. De la même manière, vous devez créer client3.ovpn.



### Installation d’OpenVPN client sur le téléphone (Android) 

Premièrement il faut Installer OpenVPN sur votre téléphone. Après il faut télécharger  client2.ovpn de votre ordinateur sur le téléphone, par exemple, en utilisant le câble.

On ouvre OpenVPN et choisi fichier client2.ovpn et clique sur Import

<img src="/images/openvpn2/image13c.png">

Si le fichier est correct, il faut choisir “Connect after import” et cliquer sur ADD:

<img src="/images/openvpn2/image9c.png">

Si tout est bien configuré - vous pouvez vous connecter:

<img src="/images/openvpn2/image4c.png">

### Installation d’OpenVPN client sur Raspberry PI 

Par défaut Raspberry Pi utilise le système raspbian qui est basé sur Debian, c’est pourquoi l’installation et la configuration sont similaire à Ubuntu/Debian. On va utiliser la connection à partir de ligne de commande.

Premièrement, il faut installer OpenVPN sur Raspberry Pi:

*sudo apt-get update && sudo apt-get upgrade -y*

*sudo apt install openvpn -y*

<img src="/images/openvpn2/image3.png">

<img src="/images/openvpn2/image1.png">

Pour la connexion de Raspberry PI on va utiliser client3. On va copier client3.ovpn dans le dossier \~/client3 sur Raspberry PI:

<img src="/images/openvpn2/image12.png">

Après on exécute:

*sudo openvpn --config client3.ovpn*

<img src="/images/openvpn2/image8.png">

Si tout est configuré bien, on connecte au VPN et l'adresse IP va changer. Pour la vérification de l’adresse IP externe on utilise le web browser sur Raspberry Pi et utilise n'importe quel site pour vérifier l’adresse IP externe. J’utilise https://whatismyipaddress.com/:

<img src="/images/openvpn2/image11.png">

Pour la vérification d’adresse IP interne on utilise le commande:

*ip a*

<img src="/images/openvpn2/image7.png">

Maintenant on va connecter l’ordinateur avec Windows à notre VPN en utilisant la configuration lequel on a créé dans [la première partie](https://olexdziuba.github.io/OpenVPNsurUBUNTU-1/).
Après la connection on vérifie l’adresse IP interne en utilisant le commande:

*ipconfig*

<img src="/images/openvpn2/image5.png">

On essaye de pinger client3 a partir de client1.

<img src="/images/openvpn2/image10.png">

S’il pinge bien, on essaye de connecter en ssh. Et voilà:

<img src="/images/openvpn2/image6.png">

On peut aussi utiliser Filezilla pour se connecter à Raspberry PI en stfp:

<img src="/images/openvpn2/image14.png">

Comme ça on peut utiliser Raspberry PI comme un entrepôt de données accessible de n'importe où dans le monde où il y a l’Internet.

Pour aller plus loin vous pouvez aussi installer un SAMBA server :)
