# Configuration des serveurs et des postes

Ces équipements se configurent avec l'interface graphique de Packet Tracer (onglets **Desktop** et **Services**).

## SRV-01 (Server-PT), VLAN 50

**Desktop > IP Configuration** (adresse fixe)

| Champ | Valeur |
|---|---|
| IPv4 Address | 192.168.50.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.50.1 |
| DNS Server | 192.168.50.10 |

### Services > DHCP

Activer le service (**On**) et créer un pool par VLAN utilisateur.
Le pool `serverPool` créé par défaut ne peut pas être supprimé : il est modifié pour le VLAN 50.

| Pool Name | Default Gateway | DNS Server | Start IP Address | Subnet Mask | Maximum Users |
|---|---|---|---|---|---|
| VLAN10-DIRECTION | 192.168.10.1 | 192.168.50.10 | 192.168.10.100 | 255.255.255.0 | 100 |
| VLAN20-COMPTA | 192.168.20.1 | 192.168.50.10 | 192.168.20.100 | 255.255.255.0 | 100 |
| VLAN30-TECHNIQUE | 192.168.30.1 | 192.168.50.10 | 192.168.30.100 | 255.255.255.0 | 100 |
| VLAN40-INVITES | 192.168.40.1 | 192.168.50.10 | 192.168.40.100 | 255.255.255.0 | 50 |
| serverPool | 192.168.50.1 | 192.168.50.10 | 192.168.50.100 | 255.255.255.0 | 50 |

Pour chaque pool : remplir les champs puis cliquer sur **Add** (ou **Save** pour `serverPool`).

Les adresses de .2 à .99 restent libres dans chaque réseau pour les équipements à adresse fixe (imprimantes, etc.).

### Services > DNS

Activer le service (**On**) et ajouter deux enregistrements de type **A Record** :

| Name | Address |
|---|---|
| intranet.alpes.local | 192.168.50.10 |
| www.site-externe.com | 198.51.100.10 |

### Services > HTTP

Laisser HTTP sur **On**. Modifier `index.html` pour afficher une page d'intranet, par exemple :

```html
<html><body><h1>Intranet Alpes Conseil</h1><p>Bienvenue sur l'intranet.</p></body></html>
```

## WEB-EXT (Server-PT), réseau « Internet »

| Champ | Valeur |
|---|---|
| IPv4 Address | 198.51.100.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 198.51.100.1 |

**Services > HTTP** : On. Modifier `index.html` :

```html
<html><body><h1>Site externe</h1><p>Si cette page s'affiche, l'accès Internet fonctionne.</p></body></html>
```

Désactiver les autres services (DHCP, DNS…) qui ne servent pas ici.

## Postes clients

Sur chaque PC et sur le portable invité : **Desktop > IP Configuration > DHCP**.

| Poste | Switch / port | VLAN | Adresse attendue |
|---|---|---|---|
| PC-DIR1 | SW-ETG1 Fa0/1 | 10 | 192.168.10.100 à .199 |
| PC-COMPTA1 | SW-ETG1 Fa0/5 | 20 | 192.168.20.100 à .199 |
| PC-COMPTA2 | SW-ETG1 Fa0/6 | 20 | 192.168.20.100 à .199 |
| PC-TECH1 | SW-ETG2 Fa0/1 | 30 | 192.168.30.100 à .199 |
| PC-TECH2 | SW-ETG2 Fa0/2 | 30 | 192.168.30.100 à .199 |
| LAPTOP-INV1 | SW-ETG2 Fa0/11 | 40 | 192.168.40.100 à .149 |

Si un poste reste en `169.254.x.x`, attendre quelques secondes et recliquer sur **DHCP** :
au démarrage, les ports des switchs mettent un peu de temps à passer en état de transmission.
