# OpenVPN :
Tout d'abord il y a plusieurs façon d'installer OpenVPN, on peut le faire à la mano, ou encore utiliser PIVPN, ou alors on peut utilser une technique digne des plus grands flemmard (Il vaut mieux s'assurer de comprendre comment un VPN fonctionne avant d'utiliser cette technique) :
On va utiliser le dernier exemple pour cela on part d'une VM Debian vierge coonecter en réseau host-only:


## Prérequis :
Premièrement il est important d'ouvrir un port sur sa box.
&nbsp;
Pour cela ouvrer votre navigateur web puis taper `192.168.1.1` ou `192.168.1.254`, connecter vous avec l'identifiant `admin` et les huits premiers caractères du mot de passe de votre box.
&nbsp;
Rendez-vous dans la partie `Réseaux`, puis `NAT/PAT`, enfin choississez le port de votre choix (ici de préférence le 1194), pour le protocole choississez les deux, (ou ici de préférence UDP), enfin dans appareil choississez votre VM

&nbsp;
Tout d'abord on tape la commande :
```bash
wget git.io/vpn -O openvpn_install.sh && bash openvpn_install.sh
```

- **-O openvpn_install.sh :** permet de sauvgarder ce qu'on télécharge dans un fichier
- **&& bash openvpn_install.sh** : si la commande d'avant a fonctionner, cela permet de lancer le script

---

1. Ici on renseigne l'ip publique sur laquelle on doit taper pour accéder à notre vpn depuis un réseau extérieur. Si votre adresse IP se modifient vous pouvez mettre un nom de domain cela permettre que le nom de domaine suive l'IP.
```bash
This server is behind NAT. What is the public IPv4 address or hostname?
Public IPv4 address / hostname [X;X;X;X] : 
```

&nbsp;
2. Ici on choisis le protocole de communication (Ici on préfera choisir UDP) cela permettre une connexion plus rapide, et les echanges sont déjà sécuriser grâce au VPN.
```bash
Which protocol do you want for OpenVPN connections?
  1) UDP (recommended)
  2) TCP
 Protocol [1]:
```

&nbsp;
3. On choisit ici le port qu'on a ouvert sur notre box (ici 1194 est le port par défaut d'OpenVPN)
```bash
What port do you want OpenVPN listening to?
Port [1194]:
```

&nbsp;
4. Maintenant on choisis le DNS (ici on peut laisser par défaut)
```bash
Which DNS do you want to use with the VPN?
  1) Current system resolvers
  2) 1.1.1.1
  3) Google
  4) OpenDNS
  5) NTT
  6) AdGuard
DNS [1]:
```

&nbsp;
5. Enfin on choisis le premier "client" qui pourra se connecter au VPN.
```
Finally, tell me a name for the client certificate.
Client name [client]:
```

&nbsp;
Appuyez de nouveau sur `Entrée` et comme par magie votre vpn va se monter.
&nbsp;
Maintenant on peut copier coller le contenue de `clientVPN` ou le transferer sur un autre PC, et lancer le fichier de config `client OpenVPN` depuis un autre réseaux et vérifier que la connexion s'établit bien.

Bien joué votre VPN est bien monté :thumbs_up:
