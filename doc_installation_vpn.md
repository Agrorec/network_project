# Installation d'un VPN

- [OpenVPN] : 
- [Wireguard] : 

## OpenVPN
Tout d'abord il y a plusieurs façon d'installer OpenVPN, on peut le faire à la mano, ou encore utiliser PIVPN, ou alors on peut utilser une technique digne des plus grands flemmard (Il vaut mieux s'assurer de comprendre comment un VPN fonctionne avant d'utiliser cette technique) :
On va utiliser le dernier exemple pour cela on part d'une VM Debian vierge coonecter en réseau host-only:


### Prérequis :
Premièrement il est important d'ouvrir un port sur sa box.
&nbsp;
Pour cela ouvrer votre navigateur web puis taper `192.168.1.1` ou `192.168.1.254`, connecter vous avec l'identifiant `admin` et les huits premiers caractères du mot de passe de votre box.
&nbsp;
Rendez-vous dans la partie `Réseaux`, puis `NAT/PAT`, enfin choississez le port de votre choix (ici de préférence le 1194), pour le protocole choississez les deux, (ou ici de préférence UDP), enfin dans appareil choississez votre VM

---

&nbsp;
Tout d'abord on tape la commande :
```bash
wget git.io/vpn -O openvpn_install.sh && bash openvpn_install.sh
```

- **-O openvpn_install.sh :** permet de sauvgarder ce qu'on télécharge dans un fichier
- **&& bash openvpn_install.sh** : si la commande d'avant a fonctionner, cela permet de lancer le script

---

&nbsp;

1. Ici on renseigne l'ip publique sur laquelle on doit taper pour accéder à notre vpn depuis un réseau extérieur. Si votre adresse IP se modifient vous pouvez mettre un nom de domain cela permettre que le nom de domaine suive l'IP.
```bash
This server is behind NAT. What is the public IPv4 address or hostname?
Public IPv4 address / hostname [X;X;X;X] : 
```

&nbsp;

2. Ici on choisit le protocole de communication (Ici on préfera choisir UDP) cela permettre une connexion plus rapide, et les echanges sont déjà sécuriser grâce au VPN.
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

Bien joué votre VPN est bien monté 

&nbsp; 

---

## Wireguard
Wireguard est le dernier en terme de VPN open-source, il est performant, facile à prendre en main etc...

### Prérequis :
Premièrement il est important d'ouvrir un port sur sa box.
&nbsp;
Pour cela ouvrer votre navigateur web puis taper `192.168.1.1` ou `192.168.1.254`, connecter vous avec l'identifiant `admin` et les huits premiers caractères du mot de passe de votre box.
&nbsp;
Rendez-vous dans la partie `Réseaux`, puis `NAT/PAT`, enfin choississez le port de votre choix, pour le protocole choississez les deux, enfin dans appareil choississez votre VM.

&nbsp;
De plus il faudra installer wireguard pour cela je vous renvoi sur la doc officielle https://www.wireguard.com/install/

---

### Génération des clés publique et privée
Tout d'abord on va commencer par générer les clés privées et publique sur la machine serveur et la machine client. Pour cela on se rend dans `/etc/wireguard` puis on tape la commande :
```bash
wg genkey | tee public.key | wg pubkey > public.key
```

---

### Création du fichier de configuration du serveur

Premièrement dans le dossier `/etc/wireguard` on crée un fichier `wg0.conf`.
À l'intérieur on va configurer l'interface :
```bash
[Interface]
Address = 10.135.27.1/24
```

&nbsp;

Ensuite on ajoute le port en écoute sur le serveur :
```bash
ListenPort = 1194
```

&nbsp;

On ajoute maintenant la clé privé à la config :
> tips : WireGuard utilise de simples clés publiques et privées Curve25519 pour la cryptographie entre les pairs. Curve25519 a été publié pour la première fois par Daniel J. Bernstein en 2005 (source Wikipédia : https://en.wikipedia.org/wiki/Curve25519)
```bash
PrivateKey = [private.key]
```

&nbsp;

On peut maintenant essayer de lancer l'interface avec la commande `sudo wg-quick up wg0` :
```bash
staz@pc ~# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip address add 10.135.27.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

On peut maintenant couper l'interface avec la commande `sudo wg-quick down wg0`

---

### Création du fichier de configuration du client

On crée de nouveaux un fichier `wg0.conf` dans lequel on ajoute :
```bash
[Interface]
Address = 10.135.27.20/24
PrivateKey = [private.key]
```

On peut tester de nouveaux un `sudo wg-quick up wg0`
```bash
  agrorec@pc ~> wg-quick up wg0
[#] wireguard-go utun
...
INFO: (utun1) 2019/01/27 14:36:58 Starting wireguard-go version 0.0.20181222
[+] Interface for wg0 is utun1
[#] wg setconf utun1 /dev/fd/63
[#] ifconfig utun1 inet 10.135.27.20/24 10.135.27.20 alias
[#] ifconfig utun1 up
[+] Backgrounding route monitor
```
Pensez à bien couper l'interface avec `sudo wg-quick down wg0`

---

### Configuration des pairs

- Pour le client, on ajoute les lignes suivantes :
La clé publique du serveur
```bash
PublicKey = [public.key]
```

```bash
Endpoint = [Adresse publique:1194]
```

```bash
AllowedIPs = 10.135.27.1/32
```

```bash
[Peer]
PublicKey = [public.key]
Endpoint = [Adresse publique:1194]
AllowedIPs = 10.135.27.1/32
```

Tout cela permet d'assigner les paquets à `AllowedIPs`, chiffrer par `PublicKey`, et à destination de `Endpoint`.

&nbsp;

- Pour le serveur on ajoute les lignes suivantes :
La clé publique du client
```bash
PublicKey = [public.key]
```

```bash
AllowedIPs = 10.135.27.20/32
```

```bash
[Peer]
PublicKey = [public.key]
AllowedIPs = 10.135.27.20/32
```

---

### Ajout de règles IPTABLES

Sur le serveur on va maintenant ajouter des règles **IPTABLES** pour autoriser le traffic à être router par le serveur.
Dans la partie `[Interface]` du fichier de configuration du serveur on ajoute les lignes suivantes :
```bash
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

**Important** il faut autoriser la machine serveur à router le traffic pour ça on peut modifier le fichier `/proc/sys/net/ipv4/ip_forward` et passer le `0` à `1` ce qui permet de rendre ça permanent. Sinon pour que ça se réinitialise au reboot de la machine on peut taper la commande `sysctl net.ipv4.ip_forward=1`.

---

On peut aussi définir la passerelle du client par défaut vers le serveur en modifiant la ligne `AllowedIPs = 10.135.27.1` par `AllowedIPs = 0.0.0.0/0,::0`

---

### Récapitulons

Nous avons donc monté à la mano notre serveur VPN avec Wireguard. Je vous joins ci-dessous les fichiers de conf avec des commentaires :

- Serveur :
```bash
[Interface]
# Addresse virtuelle du serveur avec son masque de sous-réseaux
Address = 10.135.27.1/24 
# Ici on autorise dans le firewall le fait de router le traffic (penser à changer votre interface si besoin)
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# Ici on enlève les autorisations dans le firewall pour router le traffic lorsqu'on éteint le VPN (penser à changer votre interface si besoin)
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# Le port qui est en écoute pour le VPN (celui qu'on a ouvert sur notre box)
ListenPort = 51845
# Coller ici votre clé privé situé dans /etc/wireguard/private.key
PrivateKey = [private.key]

[Peer]
# On entre ici la clé publique du serveur
PublicKey = [public.key]
# Ici on assigne les paquets
AllowedIPs = 10.135.27.20/24
```

&nbsp;

- Client :
```bash
[Interface]
# Addresse virtuelle du serveur avec son masque de sous-réseaux
Address = 10.135.27.20/24   
# Le port qui est en écoute pour le VPN (celui qu'on a ouvert sur notre box)
ListenPort = 51845
# Coller ici votre clé privé situé dans /etc/wireguard/private.key
PrivateKey = [private.key]

[Peer]
# Ici on entre l'IP publique
Endpoint = X.X.X.X:1194
# On entre ici la clé publique du serveur
PublicKey = [public.key]
# Ici on assigne les paquets
AllowedIPs = 0.0.0.0/0
```

Félicitation

&nbsp;

![](https://media.giphy.com/media/xu3nTl5OdCuqs/giphy.gif)
