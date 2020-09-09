---
title: "Installation et configuration d'un serveur OpenVPN sur UBUNTU (partie 1/2)"
date: 2020-09-07
tags: [OpenVPN, Ubuntu, VPN, Linux]
header:
  image: "/images/dc.jpg"
excerpt: "System administration"
mathjax: "true"
--- 

================================

On va faire l'installation et configuration  d'OpenVPN sur le serveur Ubuntu 16.04 en 2 étapes.

Dans le premier étape on va installer et configurer OpenVPN serveur, créer 1 client en utilisant un script et utiliser le premiere client pour se connecter à VPN sur l'ordinateur avec Windows.

Dans le deuxième étape on va créer le deuxième et le troisième clients manuellement. Le deuxième client on va utiliser pour connecter telephone portable a la réseau VPN et le troisième client on va utiliser pour connecter l'ordinateur avec Linux dans le réseau VPN en utilisant linge de commande. Après, parce que les 2 ordinateurs (Windows et Linux) vont être dans le même réseau virtuel on va utiliser l'ordinateur Windows pour accéder à l'ordinateur Linux en ssh. 

Aujourd'hui je vais faire le première étape.

Tout d’abord, je vais installer le serveur OpenVPN sur [VPS](https://serverum.com/virtual-servers) avec 1 GB RAM, 1 CPU et 10 GB HDD et seulement 10 Mbit réseau qui coûte 
juste 1\$ par mois. Je vais créer 3 utilisateur VPN. Parce que c’est seulement projet pour test, je vais faire login comme root.

* * * * *
### L’installation des logicieles : 

Il faut installer:

-   openvpn
-   openssl
-   easy-rsa iptables
-   bash-completion

Il faut exécuter la commande suivante:

*apt-get install openvpn openssl easy-rsa iptables bash-completion -y*

<img src="/images/openvpn/image25.png">

* * * * *
### Préparation des  variables pour génération des clés: 

Premièrement il faut configurer CA Directory: créer  ~/openvpn-ca et copier  easy-rsa dans ce dossier en utilisant:

*make-cadir ~/openvpn-ca*

<img src="/images/openvpn/image10.png">

Après on va aller dans ce dossier:

*cd ~/openvpn-ca*

<img src="/images/openvpn/image33.png"> 

On va verifier le contenu ~/openvpn-ca 

*ls -l*

<img src="/images/openvpn/image11.png">

On commence la rédaction du fichier vars. Il faut ouvrir vars: 

*vim vars*

On va remplir les champs suivants:

<img src="/images/openvpn/image37.png">

* * * * *
### Génération des certificats et clés du serveur: 

Après il faut exécuter:

*source vars*

<img src="/images/openvpn/image2.png">

Et puis il faut supprimer tous les anciens certificats:

*./clean-all*

<img src="/images/openvpn/image45.png">

On va commencer a crée les nouveaux certificats. Premièrement on va creer build-ca:

*./build-ca*

Il faut confirmer tout, parce que toutes les infos on a déjà écrit dans le vars.

<img src="/images/openvpn/image20.png">

Après on va créer key-server:

*./build-key-server server*

<img src="/images/openvpn/image29.png">

Ensuite, on va générer une signature HMAC pour renforcer les capacités de vérification d’intégrité TLS du serveur:

*openvpn --genkey --secret keys/ta.key*

<img src="/images/openvpn/image31.png">

* * * * *

### Générez un certificat client et une paire de clés (toujours no password): 

*./build-key client1* 

*./build-key client2*

*./build-key client3*

<img src="/images/openvpn/image28.png">

Pour generer keys il faut exécuter: 

*./build-dh*

<img src="/images/openvpn/image30.png">

Le serveur a également besoin d'un fichier de paramètres DH. Cela peut être créé en utilisant OpenSSL:

*openssl dhparam -out dh2048.pem 2048*

<img src="/images/openvpn/image3.png">

* * * * *
### Configurez le service OpenVPN. 

Il faut copier les fichiers dans /etc/openvpn en exécutant:

*cd ~/openvpn-ca/keys*

<img src="/images/openvpn/image14.png">

*cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn*

*gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz* \| *sudo tee /etc/openvpn/server.conf*

<img src="/images/openvpn/image15.png">

* * * * *
### Configurer la configuration OpenVPN 

On va ouvrir server.conf:

*vim /etc/openvpn/server.conf*

Il faut supprimer  “;” pour activer la ligne tls-auth:

*tls-auth ta.key 0*

*cipher AES-128-CBC*

*user nobody*

*group nogroup*

Sous “cipher AES-128-CBC” il faut ajouter:

*auth SHA256*

<img src="/images/openvpn/image35.png">

Aussi il faut supprimer “;” pour décommenter la ligne:

*push "redirect-gateway def1 bypass-dhcp"*

<img src="/images/openvpn/image38.png">

* * * * *
### Configurer la configuration réseau de votre serveur 

Premièrement, on va configurer le transfert IP en redaction:

*vim /etc/sysctl.conf*

Il faut activer: *net.ipv4.ip_forward=1*

<img src="/images/openvpn/image17.png">

Pour appliquer le changement il faut exécuter:

*sudo sysctl -p*

<img src="/images/openvpn/image21.png">

Pour activer forwarding il faut exécuter:

*echo 1 >> /proc/sys/net/ipv4/conf/all/forwarding*

<img src="/images/openvpn/image32.png">

* * * * *
### Configurer les règles UFW pour masquer les connexions client 

Premièrement, on va vérifier la carte réseau de notre serveur:

*ip route* \| *grep default*

<img src="/images/openvpn/image43.png">

On va ouvrir before.rules:

*vim /etc/ufw/before.rules*

Et on va ajouter les lignes suivantes:

*\# START OPENVPN RULES*
*\# NAT table rules*
*\*nat*
*:POSTROUTING ACCEPT [0:0]*
*\# Allow traffic from OpenVPN client to ens3*
*-A POSTROUTING -s 10.8.0.0/24 -o ens3 -j MASQUERADE*
*COMMIT*
*\# END OPENVPN RULES*

<img src="/images/openvpn/image12.png">

Pour changer les règles Firewall il faut corriger: /etc/default/ufw

*vim /etc/default/ufw*

Il faut changer:

*DEFAULT\_FORWARD\_POLICY="DROP"* to

*DEFAULT\_FORWARD\_POLICY="ACCEPT"*

<img src="/images/openvpn/image24.png">

On va ouvrir les ports 1194/udp et OpenSSH (je vais utiliser le port par défaut)

*ufw allow 1194/udp && ufw allow OpenSSH*

<img src="/images/openvpn/image27.png">

Pour appliquer les changement il faut redémarrer FW:

*ufw disable && ufw enable*

<img src="/images/openvpn/image18.png">

Pour verifier le status de firewall il faut exécuter:

*ufw status*

<img src="/images/openvpn/image5.png">

Pour verifier NAT il faut exécuter:

*iptables -L -t nat*

<img src="/images/openvpn/image46.png"> 


* * * * *

### Activer le service OpenVPN 

Si la vérification passe bien on va démarrer openVPN en exécutant:

*systemctl start openvpn@server*

<img src="/images/openvpn/image22.png">

Pour vérifier s'il démarre bien il faut exécuter:

*systemctl status openvpn@server*

<img src="/images/openvpn/image36.png">

Pour vérification  disponibilité de tun0 dans l'interface OpenVPN il faut faire:

 *ip addr show tun0*

<img src="/images/openvpn/image7.png">

Si OpenVPN ne marche pas, essayez de faire restart d'OpenVPN ou faire restart du serveur au complet.

*service openvpn restart*

<img src="/images/openvpn/image9.png">

S'il y a le problème il faut regarder le log:

*tail -f /var/log/syslog*

Si tout fonctionne bien, passe à l'étape suivante :)

* * * * *
### Créer une configuration de client 

Premièrement on va créer un dossier ccd/files pour clients:

*mkdir -p \~/ccd/files*

<img src="/images/openvpn/image40.png">

Apres on va changer permissions:

*chmod 700 \~/ccd/files*

<img src="/images/openvpn/image39.png">

Il faut copier example file dans dossier de client:

*cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/ccd/base.conf*

<img src="/images/openvpn/image42.png">

Après on va changer base.conf en exécutant:

*vim \~/ccd/base.conf*

Il faut taper IP publique de votre serveur et port (comme pendant configuration serveur):

<img src="/images/openvpn/image23.png">

On utilise protocol UDP. Pour activer ce protocol il faut supprimer “;”:

<img src="/images/openvpn/image26.png">

Pour le client Windows on active user nobody et group nogroup (il faut supprimer “;”), mais si vous utiliser comme client OpenVPN Linux, il faut laisser cela inactive:

<img src="/images/openvpn/image6.png">

Il faut désactiver prochaines lignes (ajouter “;”):

<img src="/images/openvpn/image16.png">

Aussi on va ajouter les lignes prochaines:

<img src="/images/openvpn/image44.png">

Aussi il faut ajouter la key-direction 1 (parce que c’est un client, pour serveur on a utilisé 0):

<img src="/images/openvpn/image1.png">

Pour faire la configuration  du client on peut utiliser un script ou faire un fichier manuellement. Je vais montrer deux possibilités :)

Pour la configuration du client avec le script vous devez suivre ces étapes:

##### 1.  Création d'un script de génération de configuration.
Vous pouvez soit télécharger ce script [d'ici](https://github.com/olexdziuba/vpn/blob/master/make_config.sh), soit le créer manuellement.
Commencer par:
*vim \~/ccd/make\_config.sh*

À l'intérieur, collez le script suivant:

*\#!/bin/bash*

*\# First argument: Client identifier*

*KEY\_DIR=\~/openvpn-ca/keys*

*OUTPUT\_DIR=\~/ccd/files*

*BASE\_CONFIG=\~/ccd/base.conf*

*cat \${BASE\_CONFIG} \\*

*\<(echo -e '\<ca\>') \\*

*\${KEY\_DIR}/ca.crt \\*

*\<(echo -e '\</ca\>\\n\<cert\>') \\*

*\${KEY\_DIR}/\${1}.crt \\*

*\<(echo -e '\</cert\>\\n\<key\>') \\*

*./make\_config.sh client*

*\<(echo -e '\</key\>\\n\<tls-auth\>') \\*

*\${KEY\_DIR}/ta.key \\*

*\<(echo -e '\</tls-auth\>') \\*

*\> \${OUTPUT\_DIR}/\${1}.ovpn*

<img src="/images/openvpn/image4.png">

##### 2.  Changer la permission du fichier:

*chmod 700 \~/ccd/make\_config.sh*

<img src="/images/openvpn/image8.png">

##### 3. Générer un fichier de configuration client

*cd \~/ccd*

*./make\_config.sh client1*

<img src="/images/openvpn/image13.png">

On verifie si le fichier est crée:

*ls \~/ccd/files*

<img src="/images/openvpn/image19.png">

Il faut aussi vérifier ce file, il faut ajouter *remote* avant l'adresse IP:

<img src="/images/openvpn/image34.png">

Après on copie ce file sur l'ordinateur de client. Sur l'ordinateur avec Windows il faut installer [OpenVPN](https://openvpn.net/community-downloads/) et ajouter nouveau client. Aussi il faut exécuter OpenVPN comme administrateur ("run as admin"). S'il la connection est bloqué, c’est un bonne idée de lire le logfile :)

Et voilà, on a connecté. Je mesure la vitesse d'internet avec speedtest. Le ping est grand et vitesse 10 Mbps upload et download, mais c’est correct, parce que le serveur a ses restrictions.


<img src="/images/openvpn/image41.png">
