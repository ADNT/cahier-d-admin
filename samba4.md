#samba 4 
Créer un serveur active directory avec samba 4. 
 * nom du contrôleur : dc1
 * nom du domaine   : mydomain.intra
 * domaine NT       : mydomain
 * IP du contrôleur : 192.168.1.10

## installation et configuration



```bash
localhost#> aptitude -y install samba krb5-config 
```
réponse :

* realm: MYDOMAIN.INTRA
* server: dc1.mydomain.intra
* admin server: dc1.mydomain.intra

```bash
localhost#>rm /etc/samba/smb.conf
localhost#>samba-tool domain provision
localhost#>samba-tool domain level raise --domain-level 2008_R2 --forest-level 2008_R2
```

## Configuration du serveur NTP

```bash
  apt-get install ntpdate
  ntpdate pool.ntp.org
  apt-get install ntp
```


pour plus d'information voir la documentation ici :
[wiki samba](https://wiki.samba.org/index.php/Setup_a_Samba_Active_Directory_Domain_Controller)

## Configuration d'une réplication du DC

* nom : dc2
* IP  : 192.168.1.11


Effectuer la même installation que pour le contrôleur, mais sans provisionnement.
configurer le fichier /etc/resolv.conf comme suis :

```bash
search mydomain.intra
nameserver 192.168.1.10
```

configurer le fichier /etc/krb5.conf :

```bash
[libdefaults]
    default_realm = MYDOMAIN.INTRA
    dns_lookup_kdc = true
    dns_lookup_realm=false

[realms]
    MYDOMAIN.INTRA = {
    kdc = 192.168.1.10   
    kdc = 192.168.1.11   
    }

[domain_realms]
    .mydomain.intra = MYDOMAIN.INTRA
    mydomain.lan = MYDOMAIN.INTRA
    
```

Tester la connexion avec

```
localhost#>kinit administrator
```

Promouvoir le contrôleur en membre du domaine

```
localhost#>samba-tool domain join mydomain.intra DC -U administrator --realm=MYDOMAIN.INTRA -W MYDOMAIN
```

ajouter une entrée DNS pour le nouveau serveur

```
localhost#>samba-tool dns add dc1.mydomain.intra mydomain.intra mydomain.intra NS dc2.mydomain.intra
```

création des entrées DNS nécessaire pour la réplication

```
samba_dnsupdate --use-samba-tool
```

Après installation vous pouvez configurer le système pour qu'il utilise l'adresse de loopback.


### répliquer les sysvol

Actuellement samba ne supporte pas la réplication des sysvol alors la solution actuelle
et de mettre en place une tache cron qui va répliquer les données entre les serveurs.
Quand la réplication est faite, vous pouvez checker les acl sur les sysvol.

```
samba-tool ntacl sysvolcheck 
samba-tool ntacl sysvolreset
```

### Imposible de pinger le domain *.local

Les domaines en .local font partie de la norme avahi et quand votre
système reçoit une demande de résolution il utilise des requêtes
m-DNS.
 
Les solutions sont soit :  vous supprimer le service, ou sinon vous
utiliser un autre suffixe dns (ex: intra).

### comment obtenir la liste des connexions

```
    bash:# smbstatus
```
