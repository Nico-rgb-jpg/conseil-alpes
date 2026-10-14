# Plan de tests

Tests à réaliser une fois la maquette configurée. Les pings se font depuis **Desktop > Command Prompt**,
les pages web depuis **Desktop > Web Browser**.

> Le premier ping vers un nouveau réseau perd souvent un ou deux paquets (le temps de la résolution ARP) :
> c'est normal, il faut relancer le ping.

## 1. Adressage et DHCP

| # | Test | Commande | Résultat attendu |
|---|---|---|---|
| 1.1 | Chaque poste reçoit une adresse | `ipconfig` | adresse dans le bon réseau, passerelle en .1, DNS 192.168.50.10 |
| 1.2 | VLAN bien affectés | `show vlan brief` sur chaque switch | ports dans les bons VLAN |
| 1.3 | Trunks actifs | `show interfaces trunk` | G0/1, G0/2 et Fa0/24 en trunk, native 999 |

## 2. Routage inter-VLAN

| # | Depuis | Vers | Commande | Résultat attendu |
|---|---|---|---|---|
| 2.1 | PC-DIR1 | sa passerelle | `ping 192.168.10.1` | OK |
| 2.2 | PC-DIR1 | PC-TECH1 | `ping <IP de PC-TECH1>` | OK |
| 2.3 | PC-TECH1 | SRV-01 | `ping 192.168.50.10` | OK |
| 2.4 | PC-COMPTA1 | PC-COMPTA2 | `ping <IP de PC-COMPTA2>` | OK (même VLAN, pas de passage par le routeur) |

## 3. Services

| # | Depuis | Test | Résultat attendu |
|---|---|---|---|
| 3.1 | PC-TECH1 | `nslookup intranet.alpes.local` | 192.168.50.10 |
| 3.2 | PC-DIR1 | navigateur : `intranet.alpes.local` | page « Intranet Alpes Conseil » |
| 3.3 | PC-COMPTA1 | navigateur : `intranet.alpes.local` | page « Intranet Alpes Conseil » |

## 4. Accès Internet (NAT)

| # | Depuis | Test | Résultat attendu |
|---|---|---|---|
| 4.1 | PC-TECH1 | `ping 198.51.100.10` | OK |
| 4.2 | PC-DIR1 | navigateur : `www.site-externe.com` | page « Site externe » |
| 4.3 | R1 | `show ip nat translations` | des entrées avec l'adresse inside global 203.0.113.2 |
| 4.4 | R-ISP | `show ip route` | aucune route vers 192.168.x.x : le FAI ne connaît que l'adresse publique |

## 5. Sécurité : ACL

| # | Depuis | Test | Résultat attendu |
|---|---|---|---|
| 5.1 | PC-TECH1 | `ping <IP de PC-COMPTA1>` | **échec** (ACL COMPTA) |
| 5.2 | PC-DIR1 | `ping <IP de PC-COMPTA1>` | OK (la Direction a accès à la comptabilité) |
| 5.3 | PC-COMPTA1 | `ping <IP de PC-TECH1>` | OK (la réponse est autorisée par la règle `echo-reply`) |
| 5.4 | LAPTOP-INV1 | `ping <IP de PC-DIR1>` | **échec** (ACL INVITES) |
| 5.5 | LAPTOP-INV1 | `ping 192.168.40.1` | OK |
| 5.6 | LAPTOP-INV1 | navigateur : `www.site-externe.com` | OK (DNS autorisé, Internet autorisé) |
| 5.7 | LAPTOP-INV1 | navigateur : `intranet.alpes.local` | **échec** (seul le DNS est autorisé vers le serveur) |
| 5.8 | R1 | `show access-lists` | les compteurs des règles `deny` augmentent après les tests 5.1, 5.4 et 5.7 |

## 6. Sécurité : accès aux équipements

| # | Depuis | Test | Résultat attendu |
|---|---|---|---|
| 6.1 | PC-TECH1 | `ssh -l admin 192.168.99.2` | connexion SSH à SW-CORE (mot de passe `Admin@2026`) |
| 6.2 | PC-TECH1 | `ssh -l admin 192.168.99.1` | connexion SSH à R1 |
| 6.3 | PC-DIR1 | `ssh -l admin 192.168.99.2` | **refusé** (access-class : seul le VLAN 30 est autorisé) |
| 6.4 | PC-TECH1 | `telnet 192.168.99.2` | **refusé** (seul SSH est accepté) |

## 7. Sécurité : switchs

| # | Test | Résultat attendu |
|---|---|---|
| 7.1 | `show port-security interface fa0/1` sur SW-ETG1 | Port Security Enabled, adresse MAC apprise en sticky |
| 7.2 | Brancher un nouveau PC sur SW-ETG1 Fa0/20 | pas de lien : le port est désactivé (VLAN 999, shutdown) |
| 7.3 | `show interfaces status` | ports non utilisés en état `disabled` |
