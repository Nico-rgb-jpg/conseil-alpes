# Reconstruire la maquette dans Packet Tracer

## 1. Placer les équipements

| Nom | Modèle Packet Tracer | Rôle |
|---|---|---|
| R1 | 2911 | routeur de l'entreprise |
| R-ISP | 2911 | routeur qui simule le fournisseur d'accès |
| SW-CORE | 2960-24TT | switch central |
| SW-ETG1 | 2960-24TT | switch d'accès 1er étage |
| SW-ETG2 | 2960-24TT | switch d'accès 2e étage |
| SRV-01 | Server-PT | DHCP, DNS, intranet |
| WEB-EXT | Server-PT | site web « Internet » |
| PC-DIR1, PC-COMPTA1, PC-COMPTA2, PC-TECH1, PC-TECH2 | PC-PT | postes |
| LAPTOP-INV1 | Laptop-PT | poste invité |

Renommer chaque équipement (onglet **Config > Display Name**) pour que le schéma reste lisible.

## 2. Câbler

| De | Port | Vers | Port | Câble |
|---|---|---|---|---|
| R1 | G0/0 | SW-CORE | Fa0/24 | droit |
| R1 | G0/1 | R-ISP | G0/0 | croisé |
| R-ISP | G0/1 | WEB-EXT | Fa0 | croisé (routeur ↔ serveur) |
| SW-CORE | G0/1 | SW-ETG1 | G0/1 | croisé |
| SW-CORE | G0/2 | SW-ETG2 | G0/1 | croisé |
| SW-CORE | Fa0/1 | SRV-01 | Fa0 | droit |
| SW-ETG1 | Fa0/1 | PC-DIR1 | Fa0 | droit |
| SW-ETG1 | Fa0/5 | PC-COMPTA1 | Fa0 | droit |
| SW-ETG1 | Fa0/6 | PC-COMPTA2 | Fa0 | droit |
| SW-ETG2 | Fa0/1 | PC-TECH1 | Fa0 | droit |
| SW-ETG2 | Fa0/2 | PC-TECH2 | Fa0 | droit |
| SW-ETG2 | Fa0/11 | LAPTOP-INV1 | Fa0 | droit |

## 3. Configurer, dans cet ordre

1. **Les switchs** : `configs/SW-CORE.txt`, `configs/SW-ETG1.txt`, `configs/SW-ETG2.txt`
2. **R-ISP** puis **R1** : `configs/R-ISP.txt`, `configs/R1.txt`
3. **Les serveurs** : `configs/serveurs-et-postes.md`
4. **Les postes** en DHCP

Pour les équipements Cisco : ouvrir l'onglet **CLI**, appuyer sur Entrée pour passer le message de démarrage
(répondre `no` à la question sur le dialogue de configuration initiale), puis coller le fichier.
Il vaut mieux coller par blocs (une section à la fois) pour repérer facilement une commande refusée.

Après la configuration, la console demande un identifiant : `admin` / `Admin@2026`.

Si une version de Packet Tracer refuse `crypto key generate rsa general-keys modulus 1024`,
taper seulement `crypto key generate rsa` et répondre `1024` à la question sur la taille de la clé.

## 4. Vérifier

Suivre le plan de tests : `docs/tests.md`.

## Commandes de vérification utiles

```
show vlan brief                  ! VLAN et ports associés
show interfaces trunk            ! trunks actifs et VLAN autorisés
show ip interface brief          ! état et adresses des interfaces
show ip route                    ! table de routage
show ip nat translations         ! traductions NAT en cours
show access-lists                ! ACL avec le nombre de paquets qui correspondent à chaque règle
show port-security interface fa0/1
show running-config
```
