# OpenVPN :
Tout d'abord il y a plusieurs façon d'installer OpenVPN, on peut le faire à la mano, ou encore utiliser PIVPN, ou alors on peut utilser une technique digne des plus grands flemmard (Il vaut mieux s'assurer de comprendre comment un VPN fonctionne avant d'utiliser cette technique) :
On va utiliser le dernier exemple pour cela on part d'une VM Debian vierge :

&nbsp;
Tout d'abord on tape la commande :
```bash
wget git.io/vpn -O openvpn_install.sh && bash openvpn_install.sh
```

- **-O openvpn_install.sh :** permet de sauvgarder ce qu'on télécharge dans un fichier
- **&& bash openvpn_install.sh** : si la commande d'avant a fonctionner, cela permet de lancer le script

