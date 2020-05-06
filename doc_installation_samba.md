# Documentation sur l'installation sur le Samba

Ici nous prendrons une machine sous linux Debian pour notre exemple.

## Téléchargement et installation de Samba

Pour ce faire vous devrez utiliser votre gestionnaire de paquet, pour debian c'est `apt` :

```bash 
$sudo apt install samba
```

## Configuration du Samba

Tout d'abord nous devons crée un fichier à partager, ici ce sera `/home/staz/sambashare` :

```bash
$mkdir /home/staz/sambashare
```

Ensuite nous devons éditer le fichier de configuration de samba qui se trouve au `/etc/samba/smb.conf` :

```bash
$sudo vim /etc/samba/smb.conf
```
Et y ajouter à la fin :

```bash
[share]
comment = Exemple
path = /home/staz/sambashare
readonly = no
browsable = yes
```
 - Le `[share]` est le nom du partage.
 - Le `comment` permet de laisser un commentaire.
 - Le `path` indique le chemin pour accéder au fichier à partager.
 - Le `readonly` permet la modification de fichier.

Ensuite nous devons définir un mot de passe :

```bash
sudo smbpasswd -a staz
```
`Staz` est le nom de l'utilisateur, nous devrons ensuite entrer le mot de passe.

Et voila, le samba est configurer ! 
