
#connecter un disque iscsi sous Debian


##Près-requis

Installer le packet suivant:

 * open-iscsi

commande:

```
   bash:#apt-get install open-isci
```

S'assurer que le service est démarré:

```
     /etc/init.d/open-iscsi start
```

##Recherche les volumes disponibles

Découvrir les volumes disponnible sur l'hote

```
   bash:#iscsiadm -m discovery -t st -p 192.168.1.22
```
Il est possible d'afficher les volumes découverts avec la commande:
```
  bash:#iscsiadm -m node
```
résultat:
```
  192.168.1.22:3260,0 iqn.2000-01.com.synology:store-01.target-1.95c8d660a4
```
##Connecter le volume iscsi

Saisir les informations de login si nécessaire:

```
   bash:#iscsiadm -m node\ 
                  --targetname "iqn.2000-01.com.synology:store-01.target-1.95c8d660a4"\ 
                  --portal "192.168.1.22:3260"\ 
	          --op=update \ 
	          --name node.session.auth.authmethod \
	          --value=CHAP 
   bash:#iscsiadm -m node \
                  --targetname "iqn.2000-01.com.synology:store-01.target-1.95c8d660a4"\ 
                  --portal "192.168.1.22:3260" \
                  --op=update --name node.session.auth.username\
                  --value=username 
   bash:#iscsiadm -m node 
                  --targetname "iqn.2000-01.com.synology:store-01.target-1.95c8d660a4"\ 
                  --portal "192.168.1.22:3260" \
                  --op=update \
	          --name node.session.auth.password \
	          --value=password 
```
connecter le volume:

```
   bash:#iscsiadm -m node\
                  --targetname "iqn.2000-01.com.synology:store-01.target-1.95c8d660a4"\ 
                  --portal "192.168.1.22:3260"\ 
		  --login 

   Logging in to [iface: default, target: 
      iqn.2000-01.com.synology:store-01.target-1.95c8d660a4, 
      portal: 192.168.1.22,3260] (multiple) 
   Login to [iface: default, target:
      iqn.2000-01.com.synology:store-01.target-1.95c8d660a4, portal:
      192.168.1.22,3260] successful.  
   bash:#
```
maintenant que le volume est connecter vous pouvez travailler avec
comme si c'était un disque standard.  Un exemple de formatage
ci-dessous:
```
   bash:#fdisk -l

   Disque /dev/sda : 240.1 Go, 240057409536 octets
   255 têtes, 63 secteurs/piste, 29185 cylindres, total 468862128 secteurs
   Unités = secteurs de 1 * 512 = 512 octets
   Taille de secteur (logique / physique) : 512 octets / 512 octets
   taille d'E/S (minimale / optimale) : 512 octets / 512 octets
   Identifiant de disque : 0x000ddb2e

   Périphérique Amorce  Début        Fin      Blocs     Id  Système
   /dev/sda1   *        2048      499711      248832   83  Linux
   /dev/sda2          499712   468860927   234180608   83  Linux

   Disque /dev/sdb : 7991.9 Go, 7991885561856 octets
   255 têtes, 63 secteurs/piste, 971624 cylindres, total 15609151488 secteurs
   Unités = secteurs de 1 * 512 = 512 octets
   Taille de secteur (logique / physique) : 512 octets / 512 octets
   taille d'E/S (minimale / optimale) : 512 octets / 512 octets
   Identifiant de disque : 0x00000000
```
   Le disque /dev/sdb ne contient pas une table de partitions valable


dans ce cas la périphérique sdb est apparût. Vous pouvez maintenant 
formatter le volume et l'utiliser.


##monter automatiquement le volume au boot

Après avoir découvert les volumes disposible et avoir tester la
connexion.  Il faut effectuer les points suivants:

Modifier le fichier de configuration de open-iscsi au mettant le
startup à automatique.

```
   #file: /etc/iscsi/iscsid.conf
   node.startup = automatic
```

ensuite modifier le fichier fstab pour monter automatiquement le
system de fichier en ajouter la ligne suivante:

```
   #file: /etc/fstab
   /dev/sdb1       /media/backup-store ext4   defaults,auto,_netdev 0  0
```

Attention! Si lors du discovery vous avez découvert des volumes que
vous ne vouliez pas utiliser. Il faut aller dans le répertoire
/etc/iscsi/nodes et supprimer les volumes que vous ne voulez pas
utiliser.


Après tous ça vous pouvez redemarrer le service open-iscsi. Si tous
c'est bien passé. Vous devrier avoir les volumes connectées et les FS
monté.
