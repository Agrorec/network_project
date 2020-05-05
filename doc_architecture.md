#Documentation sur l'architecture du projet

L'objectif est de mettre ne place un VPN pour se relier à un serveur NAS sécurisé en local.

## Le réseau

La Raspberry est relié à la box de sacha à Bordeaux.
Elle abrite un VPN OpenVPN qui va nous permettre de nous connecter à la rasspberry de façon sécurisée comme si nous étions connecter au réseau local.
Le VPN nous connecte en utilisant une liaison de type `remplir ici`

## Le Transfère de fichier

Le tranfère de fichier est assuré par le logiciel samba.
Nous pourrons donc nous connecter directement avec un utilisateur et un mot de passe samba au partage de fichier comme si il était un disque locale de notre machine.
Ce lien ce fait via le VPN comme si nous étions connecter sur le réseau local.


