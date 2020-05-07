# Installation d'un VPN

- [OpenVPN](#OpenVPN)
- [Wireguard](#Wireguard)

## OpenVPN
Tout d'abord il y a plusieurs façons d'installer OpenVPN. On peut le faire à la mano, utiliser PIVPN, ou alors on peut se servir d'une technique digne des plus grands flemmards (il vaut mieux s'assurer de comprendre comment un VPN fonctionne avant d'utiliser cette technique) :
On va utiliser le dernier exemple. Pour cela on part d'une VM Debian vierge connectée en réseau host-only:


### Prérequis :
Premièrement il est important d'ouvrir un port sur sa box.
&nbsp;
Pour cela ouvrez votre navigateur web puis tapez `192.168.1.1` ou `192.168.1.254`, connectez-vous avec l'identifiant `admin` et les huits premiers caractères du mot de passe de votre box.
&nbsp;
Rendez vous dans la partie `Réseaux`, puis `NAT/PAT`, choississez le port de votre choix (ici de préférence le 1194), pour le protocole choississez les deux, (ou ici de préférence UDP), enfin dans appareil choississez votre VM.

---

&nbsp;
Tout d'abord on tape la commande :
```bash
wget git.io/vpn -O openvpn_install.sh && bash openvpn_install.sh
```

- **-O openvpn_install.sh :** permet de sauvegarder ce qu'on télécharge dans un fichier
- **&& bash openvpn_install.sh** : si la commande d'avant a fonctionné, cela permet de lancer le script

---

&nbsp;

1. Ici on renseigne l'IP publique sur laquelle on doit taper pour accéder à notre vpn depuis un réseau extérieur. Si votre adresse IP se modifie vous pouvez mettre un nom de domaine, cela permettra que le nom de domaine suive l'IP.
```bash
This server is behind NAT. What is the public IPv4 address or hostname?
Public IPv4 address / hostname [X;X;X;X] : 
```

&nbsp;

2. Ici on choisit le protocole de communication (ici on préférera choisir UDP) cela permettra une connexion plus rapide et les échanges seront déjà sécurisés grâce au VPN.
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

4. Maintenant on choisit le DNS (ici on peut laisser par défaut)
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

5. Enfin on choisit le premier "client" qui pourra se connecter au VPN.
```
Finally, tell me a name for the client certificate.
Client name [client]:
```

&nbsp;
Appuyez de nouveau sur `Entrée` et comme par magie votre vpn va se monter.
&nbsp;

Maintenant on peut copier-coller le contenu de `clientVPN` ou le transférer sur un autre PC et lancer le fichier de config `client OpenVPN` depuis un autre réseau et vérifier que la connexion s'établit bien.

Bien joué votre VPN est bien monté.

&nbsp; 

---

## Wireguard
Wireguard est le dernier en terme de VPN open-source, il est performant, facile à prendre en main etc...

### Prérequis :
Premièrement il est important d'ouvrir un port sur sa box.
&nbsp;
Pour cela ouvrez votre navigateur web puis tapez `192.168.1.1` ou `192.168.1.254`, connectez-vous avec l'identifiant `admin` et les huits premiers caractères du mot de passe de votre box.
&nbsp;
Rendez-vous dans la partie `Réseaux`, puis `NAT/PAT`, choississez le port de votre choix, pour le protocole choississez les deux, enfin dans appareil choississez votre VM.

&nbsp;
De plus il faudra installer wireguard, pour cela je vous renvoie sur la doc officielle https://www.wireguard.com/install/

---

### Génération de clés publique et privée
Tout d'abord on va commencer par générer les clés privée et publique sur la machine serveur et la machine client. Pour cela on se rend dans `/etc/wireguard` puis on tape la commande :
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

On ajoute maintenant la clé privée à la config :
> tips : WireGuard utilise de simples clés publique et privée Curve25519 pour la cryptographie entre les pairs. Curve25519 a été publié pour la première fois par Daniel J. Bernstein en 2005 (source Wikipédia : https://en.wikipedia.org/wiki/Curve25519)
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

On crée de nouveau un fichier `wg0.conf` dans lequel on ajoute :
```bash
[Interface]
Address = 10.135.27.20/24
PrivateKey = [private.key]
```

On peut tester de nouveau un `sudo wg-quick up wg0`
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

Tout cela permet d'assigner les paquets à `AllowedIPs` chiffrés par `PublicKey` et à destination de `Endpoint`.

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

&nbsp;

### Ajout de règles IPTABLES

Sur le serveur on va maintenant ajouter des règles **IPTABLES** pour autoriser le traffic à être routé par le serveur.
Dans la partie `[Interface]` du fichier de configuration du serveur on ajoute les lignes suivantes :
```bash
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

&nbsp;

**Important** il faut autoriser la machine serveur à router le traffic. Pour ça on peut modifier le fichier `/proc/sys/net/ipv4/ip_forward` et passer le `0` à `1` ce qui permet de rendre ça permanent. Sinon pour que ça se réinitialise au reboot de la machine on peut taper la commande `sysctl net.ipv4.ip_forward=1`.

&nbsp;

---

On peut aussi définir la passerelle du client par défaut vers le serveur en modifiant la ligne `AllowedIPs = 10.135.27.1` par `AllowedIPs = 0.0.0.0/0,::0`

---

&nbsp;

### Récapitulons

Nous avons donc monté à la mano notre serveur VPN avec Wireguard. Je vous joins ci-dessous les fichiers de conf avec des commentaires :

- Serveur :
```bash
[Interface]
# Addresse virtuelle du serveur avec son masque de sous-réseaux
Address = 10.135.27.1/24 
# Ici on autorise dans le firewall le fait de router le traffic (pensez à changer votre interface si besoin)
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# Ici on enlève les autorisations dans le firewall pour router le traffic lorsqu'on éteint le VPN (pensez à changer votre interface si besoin)
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# Le port qui est en écoute pour le VPN (celui qu'on a ouvert sur notre box)
ListenPort = 51845
# Collez ici votre clé privée située dans /etc/wireguard/private.key
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
# Collez ici votre clé privée située dans /etc/wireguard/private.key
PrivateKey = [private.key]

[Peer]
# Ici on entre l'IP publique
Endpoint = X.X.X.X:1194
# On entre ici la clé publique du serveur
PublicKey = [public.key]
# Ici on assigne les paquets
AllowedIPs = 0.0.0.0/0
```

Félicitations

&nbsp;

![](https://media.giphy.com/media/xu3nTl5OdCuqs/giphy.gif)



---

## Vagrant

Ici se trouve à disposition un Vagrantfile :
http://www.mediafire.com/file/3f7lzlgq4q0ohaa/Vagrantfile/file

&nbsp;

```bash
Vagrant.configure("2") do |config|
  config.vm.box = "debian/stretch64"

  config.vm.network "forwarded_port", guest:1194, host: 1194, protocol: "udp"

  config.vm.provision "shell", inline: <<-SHELL
	apt update
	apt upgrade -y
	apt install vim -y
	wget https://git.io/vpn -O openvpn_installation.sh
	chmod +x openvpn_installation.sh
  SHELL
end
```

&nbsp;

Cela va vous permettre de monter une VM Debian vanilla avec vim et git.io/vpn déjà monté.
Pour cela il faut créer un dossier dans lequel vous insérez `Vagrantfile` puis on tape les commandes :
- `vagrant up`
- `vagrant ssh`
- `./openssh_installation`

Puis je vous renvoie au tuto [OpenVPN](#OpenVPN)
