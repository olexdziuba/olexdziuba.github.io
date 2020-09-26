---
title: "L'utilisation de VPN pour utiliser l'itinérance gratuite "
date: 2020-09-26
header:
  image: "/images/dc.jpg"
excerpt: "System administration"
mathjax: "true"
--- 

================================

L'utilisation de l'itinérance en voyage coûte généralement cher. Par exemple, mon opérateur de téléphonie mobile propose ce service pour 10
\$ par jour et pas pour tous les pays du monde!

J'ai trouvé un moyen de communiquer à l'étranger presque gratuitement et de sécuriser  votre connection.

Vous devez  posséder des connaissances minimales pour savoir comment vous connecter à un serveur en ssh en utilisant [Putty](https://www.putty.org/) ou
l’autre client ssh et avoir un téléphone intelligent qui supporte les appels Wi-Fi. Aussi faites attention parce que cela pourrait être interdit selon la loi de votre pays :)

### Pour faire cela il faut entreprendre les étapes suivantes:

1.  Louer  le service VPS qui se trouve physiquement dans votre pays  et     encore mieux dans votre ville où vous habitez. On peut utiliser le
    service le moins cher avec 512 Mo RAM et 1 core avec trafic illimité. Cela coûte généralement entre 1 \$ et 5 \$ par mois, selon
    la ville et le pays.  Pour économiser de  l’argent, vous pouvez utiliser de grandes entreprises multinationales pour louer un
    service virtuel, l'essentiel étant qu'elles disposent d'un centre de données dans votre pays / ville. J’ai loué le service VPS à Montréal
    parce que mon numéro de téléphone est enregistré à Montréal.
2.  Installer et configurer OpenVPN sur ce serveur selon les instructions que je viens d'écrire [ici](https://olexdziuba.github.io/OpenVPNsurUBUNTU-1/)
3.  Installer le client OpenVPN sur votre téléphone et  connecter le téléphone portable au réseau VPN selon les instructions que je viens d'écrire [ici](https://olexdziuba.github.io/OpenVPNsurUBUNTU-2/).
4.  Activer les appels Wi-Fi sur votre téléphone:

Sur IPhone:

<img src="/images/openvpn3/image01.jpg">

Sur Android:

<img src="/images/openvpn3/image02.png">

Avant de quitter votre pays vous pouvez activer le Mode Avion pour couper le réseau cellulaire de votre téléphone (pour tout le voyage)
 activer le  Wi-Fi. Pour appeler dans les pays étrangers, il faut vous connecter au Wi-Fi et à la connection VPN. Pour l'opérateur de
téléphonie mobile vous semblerez toujours être dans votre ville parce que votre l’adresse IP sera dans votre ville. Aujourd’hui c’est facile
d'utiliser cette connection comme cela parce que dans plusieurs villes il y a beaucoup de Wi-Fi publics en plus des Wi-Fi des hôtels ou chez
vos amis. De plus, votre connexion Wi-Fi sera bien sécurisée par VPN. J’ai testé cette méthode en Suisse,  en Ukraine et en Allemagne.

Bonne chance!



