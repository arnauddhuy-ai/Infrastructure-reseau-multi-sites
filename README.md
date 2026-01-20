# TP Réseau : Multi-Sites Avancé avec VLAN, Routage Inter-VLAN et Sécurité

## 0. Prérequis matériels et logiciels

### 0.1 Équipement nécessaire

| Équipement | Quantité | Modèle recommandé |
|------------|----------|-------------------|
| **Routeurs** | 3 | Cisco 2911 |
| **Switches** | 3 | Cisco 2960 |
| **PCs** | 9 | PCs Packet Tracer |
| **Serveurs** | 2 | DNS/HTTP et FTP |
| **Câbles Ethernet** | Plusieurs | PC ↔ Switch ↔ Routeur |
| **Câbles série** | 2 | R1 ↔ R2, R2 ↔ R3 |

### 0.2 Logiciels requis

- **Cisco Packet Tracer** 8.0 ou supérieur (ou équipement physique Cisco)
- **Système d'exploitation** : Windows, Linux ou macOS

### 0.3 Connaissances préalables requises

- Modèle OSI et TCP/IP
- Adressage IP et sous-réseaux
- Configuration de base routeurs/switches Cisco
- Notions de VLAN et routage inter-VLAN
- Protocoles de routage (OSPF, EIGRP)

### 0.4 Durée estimée

**6 à 8 heures** (configuration complète + tests)

---

## 1. Introduction

Dans un contexte d'entreprise moderne, la communication interne et externe repose sur une infrastructure réseau performante, sécurisée et hautement disponible. L'augmentation du nombre d'utilisateurs, la diversité des services et la nécessité de protéger les données sensibles imposent une conception réseau rigoureuse et structurée.

La segmentation du réseau, la centralisation des services, ainsi que le contrôle précis des flux de communication constituent des éléments clés pour garantir à la fois la performance opérationnelle et la sécurité des échanges. Ces mécanismes permettent également d'assurer l'évolutivité de l'infrastructure face aux besoins futurs de l'organisation.

Ce travail pratique a pour objectif de concevoir, déployer et sécuriser une infrastructure réseau multi-sites, composée de trois sites distants interconnectés par des routeurs via des liaisons WAN. Chaque site est organisé autour de VLANs afin de séparer logiquement les services, limiter les domaines de broadcast et renforcer la sécurité interne.

L'architecture mise en place intègre une DMZ centralisée, hébergeant les services critiques accessibles depuis l'ensemble des sites, tout en garantissant leur protection contre les accès non autorisés.

### Services et protocoles implémentés :

- **DHCP** : automatisation de l'attribution des adresses IP, des passerelles et des serveurs DNS
- **DNS** : résolution de noms facilitant l'accès aux services réseau
- **HTTP et FTP** : services applicatifs hébergés dans une zone démilitarisée (DMZ)
- **OSPF et EIGRP** : protocoles de routage dynamique avec redistribution inter-protocoles
- **ACL étendues** : mise en place de règles de filtrage pour contrôler les flux réseau
- **QoS** : priorisation du trafic critique afin d'optimiser les performances sur les liaisons WAN

Ce TP s'inscrit dans une démarche réaliste de conception d'infrastructure réseau d'entreprise, combinant connectivité multi-sites, sécurité, services réseau et qualité de service, conformément aux bonnes pratiques professionnelles.

---

## 2. Contexte

### 2.1 Objectifs pédagogiques

L'objectif principal de ce TP est d'acquérir une vision globale et pratique de la mise en œuvre d'un réseau d'entreprise multi-sites. À travers ce projet, les compétences suivantes sont développées :

- Concevoir une architecture réseau hiérarchisée et multi-sites
- Élaborer un plan d'adressage IP cohérent et évolutif
- Mettre en place des VLANs et assurer le routage inter-VLAN
- Déployer et configurer des protocoles de routage dynamique (OSPF et EIGRP)
- Centraliser les services réseau essentiels (DHCP, DNS, HTTP, FTP)
- Appliquer des politiques de sécurité réseau à l'aide d'ACL étendues
- Tester, valider et dépanner la connectivité, la sécurité et la performance du réseau

### 2.2 Contraintes de sécurité

Afin de refléter un environnement professionnel réaliste, plusieurs contraintes de sécurité ont été définies :

- Les flux inter-services doivent être strictement contrôlés
- Le réseau Ressources Humaines (RH) ne doit pas pouvoir accéder au réseau Finance, afin de protéger les données sensibles
- L'accès à la DMZ doit être limité exclusivement aux services autorisés (HTTP, DNS et FTP)
- Les serveurs doivent rester accessibles et fonctionnels, tout en étant protégés contre les accès non autorisés

Ces contraintes illustrent les principes fondamentaux de segmentation, de cloisonnement réseau et de moindre privilège, indispensables dans toute infrastructure réseau d'entreprise.

---

## 3. Architecture et plan d'adressage IP

### 3.1 Architecture logique

```
Site A -- R1 ----[WAN 10.1.12.0/30]---- R2 ----[WAN 10.1.23.0/30]---- R3 -- Site C (DMZ)
                                        |
                                     Site B
```

Cette architecture permet :
- Une communication inter-sites sécurisée
- Une centralisation des services dans la DMZ
- Une séparation logique des services via VLANs

### 3.2 Liaisons WAN (/30)

| Liaison | Réseau | Adresse R1 | Adresse R2 | Adresse R3 |
|---------|--------|------------|------------|------------|
| R1 ↔ R2 | 10.1.12.0/30 | 10.1.12.1 | 10.1.12.2 | — |
| R2 ↔ R3 | 10.1.23.0/30 | — | 10.1.23.1 | 10.1.23.2 |

### 3.3 SITE A — VLANs

| VLAN | Service | Réseau | Passerelle | PC (exemple) |
|------|---------|--------|------------|--------------|
| 10 | IT | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.11 |
| 20 | Finance | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.11 |
| 30 | RH | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.11 |

### 3.4 SITE B — VLANs

| VLAN | Service | Réseau | Passerelle | PC (exemple) |
|------|---------|--------|------------|--------------|
| 10 | IT | 192.168.110.0/24 | 192.168.110.1 | 192.168.110.11 |
| 20 | Finance | 192.168.120.0/24 | 192.168.120.1 | 192.168.120.11 |
| 30 | RH | 192.168.130.0/24 | 192.168.130.1 | 192.168.130.11 |

### 3.5 SITE C — VLANs + DMZ

| VLAN | Service | Réseau | Passerelle |
|------|---------|--------|------------|
| 10 | IT | 192.168.210.0/24 | 192.168.210.1 |
| 20 | Finance | 192.168.220.0/24 | 192.168.220.1 |
| 30 | RH | 192.168.230.0/24 | 192.168.230.1 |
| 99 | DMZ | 192.168.99.0/24 | 192.168.99.1 |

### 3.6 Serveurs DMZ

| Serveur | IP | Passerelle | Services |
|---------|-----|-----------|----------|
| DNS + HTTP | 192.168.99.10 | 192.168.99.1 | DNS, HTTP |
| FTP / Files | 192.168.99.20 | 192.168.99.1 | FTP |

---

## 4. Câblage

### 4.1 Types de câbles

- **PC ↔ Switch** : câble Ethernet droit
- **Switch ↔ Routeur** : lien trunk (Ethernet)
- **Routeur ↔ Routeur** : liaisons série (WAN)

---

## 5. Configuration des Switches

### 5.1 SW-A (Site A)

```cisco
enable
configure terminal
hostname SW-A

! Création des VLAN
vlan 10
 name IT
vlan 20
 name FINANCE
vlan 30
 name RH

! PC IT
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

! PC Finance
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 no shutdown

! PC RH
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 no shutdown

! Trunk vers R1
interface fastEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown

end
write memory
```

### 5.2 SW-B (Site B)

```cisco
enable
configure terminal
hostname SW-B

! Création des VLAN
vlan 10
 name IT
vlan 20
 name FINANCE
vlan 30
 name RH

! PC IT
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

! PC Finance
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 no shutdown

! PC RH
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 no shutdown

! Trunk vers R2
interface fastEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown

end
write memory
```

### 5.3 SW-C (Site C avec DMZ)

```cisco
enable
configure terminal
hostname SW-C

! Création des VLAN
vlan 10
 name IT
vlan 20
 name FINANCE
vlan 30
 name RH
vlan 99
 name DMZ

! PC IT
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

! PC Finance
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 no shutdown

! PC RH
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 no shutdown

! Serveur DNS/HTTP
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 99
 no shutdown

! Serveur FTP
interface fastEthernet 0/5
 switchport mode access
 switchport access vlan 99
 no shutdown

! Trunk vers R3
interface fastEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown

end
write memory
```

---

## 6. Configuration des Routeurs

### 6.1 R1 — Site A (OSPF uniquement)

#### 6.1.1 Routage inter-VLAN

```cisco
enable
configure terminal
hostname R1

! Interface physique (trunk)
interface gigabitEthernet 0/0
 no ip address
 no shutdown

! Sous-interface VLAN 10 (IT)
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 20 (Finance)
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 30 (RH)
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown

! Liaison WAN vers R2
interface serial 0/0/0
 ip address 10.1.12.1 255.255.255.252
 clock rate 2000000
 no shutdown

end
write memory
```

#### 6.1.2 Configuration OSPF

```cisco
enable
configure terminal

router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 10.1.12.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface serial 0/0/0

end
write memory
```

#### 6.1.3 Configuration DHCP

```cisco
enable
configure terminal

! VLAN 10 - IT
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool VLAN10_IT
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.99.10

! VLAN 20 - Finance
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool VLAN20_FINANCE
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 192.168.99.10

! VLAN 30 - RH
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp pool VLAN30_RH
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 192.168.99.10

end
write memory
```

#### 6.1.4 ACL - Blocage RH → Finance

```cisco
enable
configure terminal

! ACL pour bloquer RH vers Finance
ip access-list extended RH_FINANCE
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip any any

! Application sur l'interface VLAN 30 (RH)
interface gigabitEthernet 0/0.30
 ip access-group RH_FINANCE in

end
write memory
```

#### 6.1.5 QoS sur WAN (Priorité IT)

```cisco
enable
configure terminal

! Définir le trafic IT
access-list 101 permit ip 192.168.10.0 0.0.0.255 any

! Créer une classe pour le trafic IT
class-map match-all IT_TRAFFIC
 match access-group 101

! Créer une politique QoS
policy-map WAN_QOS
 class IT_TRAFFIC
  priority percent 30
 class class-default
  fair-queue

! Appliquer la politique sur l'interface WAN
interface serial 0/0/0
 service-policy output WAN_QOS

end
write memory
```

---

### 6.2 R2 — Site B (Routeur central - OSPF + EIGRP + Redistribution)

#### 6.2.1 Routage inter-VLAN

```cisco
enable
configure terminal
hostname R2

! Interface physique (trunk)
interface gigabitEthernet 0/0
 no ip address
 no shutdown

! Sous-interface VLAN 10 (IT)
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.110.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 20 (Finance)
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.120.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 30 (RH)
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.130.1 255.255.255.0
 no shutdown

! Liaison WAN vers R1
interface serial 0/0/0
 ip address 10.1.12.2 255.255.255.252
 no shutdown

! Liaison WAN vers R3
interface serial 0/0/1
 ip address 10.1.23.1 255.255.255.252
 clock rate 2000000
 no shutdown

end
write memory
```

#### 6.2.2 Configuration OSPF + EIGRP avec Redistribution

```cisco
enable
configure terminal

! Configuration OSPF
router ospf 1
 router-id 2.2.2.2
 network 192.168.110.0 0.0.0.255 area 0
 network 192.168.120.0 0.0.0.255 area 0
 network 192.168.130.0 0.0.0.255 area 0
 network 10.1.12.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface serial 0/0/0
 redistribute eigrp 100 subnets

! Configuration EIGRP
router eigrp 100
 no auto-summary
 network 192.168.110.0
 network 192.168.120.0
 network 192.168.130.0
 network 10.1.23.0
 redistribute ospf 1 metric 1000 1 255 1 1500

end
write memory
```

#### 6.2.3 Configuration DHCP

```cisco
enable
configure terminal

! VLAN 10 - IT
ip dhcp excluded-address 192.168.110.1 192.168.110.10
ip dhcp pool VLAN10_IT
 network 192.168.110.0 255.255.255.0
 default-router 192.168.110.1
 dns-server 192.168.99.10

! VLAN 20 - Finance
ip dhcp excluded-address 192.168.120.1 192.168.120.10
ip dhcp pool VLAN20_FINANCE
 network 192.168.120.0 255.255.255.0
 default-router 192.168.120.1
 dns-server 192.168.99.10

! VLAN 30 - RH
ip dhcp excluded-address 192.168.130.1 192.168.130.10
ip dhcp pool VLAN30_RH
 network 192.168.130.0 255.255.255.0
 default-router 192.168.130.1
 dns-server 192.168.99.10

end
write memory
```

#### 6.2.4 QoS sur WAN

```cisco
enable
configure terminal

! Définir le trafic IT
access-list 101 permit ip 192.168.110.0 0.0.0.255 any

! Créer une classe pour le trafic IT
class-map match-all IT_TRAFFIC
 match access-group 101

! Créer une politique QoS
policy-map WAN_QOS
 class IT_TRAFFIC
  priority percent 30
 class class-default
  fair-queue

! Appliquer la politique sur les interfaces WAN
interface serial 0/0/0
 service-policy output WAN_QOS

interface serial 0/0/1
 service-policy output WAN_QOS

end
write memory
```

---

### 6.3 R3 — Site C + DMZ (EIGRP uniquement)

#### 6.3.1 Routage inter-VLAN

```cisco
enable
configure terminal
hostname R3

! Interface physique (trunk)
interface gigabitEthernet 0/0
 no ip address
 no shutdown

! Sous-interface VLAN 10 (IT)
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.210.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 20 (Finance)
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.220.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 30 (RH)
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.230.1 255.255.255.0
 no shutdown

! Sous-interface VLAN 99 (DMZ)
interface gigabitEthernet 0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
 no shutdown

! Liaison WAN vers R2
interface serial 0/0/0
 ip address 10.1.23.2 255.255.255.252
 no shutdown

end
write memory
```

#### 6.3.2 Configuration EIGRP

```cisco
enable
configure terminal

router eigrp 100
 no auto-summary
 network 192.168.210.0 0.0.0.255
 network 192.168.220.0 0.0.0.255
 network 192.168.230.0 0.0.0.255
 network 192.168.99.0 0.0.0.255
 network 10.1.23.0

end
write memory
```

#### 6.3.3 Configuration DHCP

```cisco
enable
configure terminal

! VLAN 10 - IT
ip dhcp excluded-address 192.168.210.1 192.168.210.10
ip dhcp pool VLAN10_IT
 network 192.168.210.0 255.255.255.0
 default-router 192.168.210.1
 dns-server 192.168.99.10

! VLAN 20 - Finance
ip dhcp excluded-address 192.168.220.1 192.168.220.10
ip dhcp pool VLAN20_FINANCE
 network 192.168.220.0 255.255.255.0
 default-router 192.168.220.1
 dns-server 192.168.99.10

! VLAN 30 - RH
ip dhcp excluded-address 192.168.230.1 192.168.230.10
ip dhcp pool VLAN30_RH
 network 192.168.230.0 255.255.255.0
 default-router 192.168.230.1
 dns-server 192.168.99.10

end
write memory
```

#### 6.3.4 ACL - Sécurisation de la DMZ

```cisco
enable
configure terminal

! ACL pour limiter l'accès à la DMZ
ip access-list extended DMZ_ACCESS
 permit tcp any host 192.168.99.10 eq 80
 permit tcp any host 192.168.99.10 eq 443
 permit udp any host 192.168.99.10 eq 53
 permit tcp any host 192.168.99.20 eq 21
 permit tcp any host 192.168.99.20 eq 20
 deny ip any 192.168.99.0 0.0.0.255
 permit ip any any

! Application sur l'interface DMZ
interface gigabitEthernet 0/0.99
 ip access-group DMZ_ACCESS in

end
write memory
```

#### 6.3.5 ACL - Bloquer RH → DMZ (sauf HTTP/DNS)

```cisco
enable
configure terminal

! ACL pour limiter RH vers DMZ
ip access-list extended RH_DMZ
 permit tcp 192.168.230.0 0.0.0.255 host 192.168.99.10 eq 80
 permit udp 192.168.230.0 0.0.0.255 host 192.168.99.10 eq 53
 deny ip 192.168.230.0 0.0.0.255 192.168.99.0 0.0.0.255
 permit ip any any

! Application sur l'interface VLAN 30 (RH) du Site C
interface gigabitEthernet 0/0.30
 ip access-group RH_DMZ in

end
write memory
```

---

## 7. Configuration des Serveurs (DMZ)

### 7.1 Serveur DNS + HTTP (192.168.99.10)

#### Configuration IP :

| Paramètre | Valeur |
|-----------|--------|
| IP | 192.168.99.10 |
| Masque | 255.255.255.0 |
| Passerelle | 192.168.99.1 |
| DNS | 192.168.99.10 (lui-même) |

#### Service DNS :

1. Activer le service DNS
2. Ajouter les enregistrements suivants :
   - `intranet.local` → `192.168.99.10`
   - `ftp.intranet.local` → `192.168.99.20`

#### Service HTTP :

1. **Activer le service HTTP**

2. **Modifier la page d'accueil (index.html)** :
   - Accéder au gestionnaire de fichiers
   - Cliquer sur **(edit)** à côté de `index.html`
   - Remplacer le contenu par :

```html
<html>
<head>
  <title>Intranet</title>
</head>
<body>
  <h1>Bienvenue sur l'intranet</h1>
  <p>Serveur Web de l'entreprise</p>
</body>
</html>
```

### 7.2 Serveur FTP (192.168.99.20)

#### Configuration IP :

| Paramètre | Valeur |
|-----------|--------|
| IP | 192.168.99.20 |
| Masque | 255.255.255.0 |
| Passerelle | 192.168.99.1 |
| DNS | 192.168.99.10 |

#### Service FTP :

1. Activer le service FTP
2. Créer un utilisateur :
   - **Username** : `ftpuser`
   - **Password** : `ftppass`
   - **Permissions** : Read/Write

---

## 8. Configuration des PC

### 8.1 Configuration DHCP (recommandée)

Pour tous les PC :
- **IP Configuration** : DHCP

Les PC obtiendront automatiquement :
- Adresse IP
- Masque de sous-réseau
- Passerelle par défaut
- Serveur DNS

### 8.2 Configuration manuelle (exemple Site A - IT)

Si configuration manuelle souhaitée :

| Paramètre | Valeur |
|-----------|--------|
| IP | 192.168.10.11 |
| Masque | 255.255.255.0 |
| Passerelle | 192.168.10.1 |
| DNS | 192.168.99.10 |

---

## 9. Tests et vérifications

### 9.1 Vérifications sur les Routeurs

```cisco
! Vérifier les interfaces
show ip interface brief

! Vérifier le routage OSPF
show ip ospf neighbor
show ip ospf database
show ip route ospf

! Vérifier le routage EIGRP
show ip eigrp neighbors
show ip eigrp topology
show ip route eigrp

! Vérifier toutes les routes
show ip route

! Vérifier DHCP
show ip dhcp binding
show ip dhcp pool

! Vérifier QoS
show policy-map interface serial 0/0/0

! Vérifier ACL
show access-lists
show ip access-lists
```

### 9.2 Vérifications sur les Switches

```cisco
! Vérifier les VLAN
show vlan brief

! Vérifier les interfaces trunk
show interfaces trunk

! Vérifier une interface spécifique
show interface fa0/24 switchport
```

### 9.3 Tests depuis les PC

#### Test 1 : Connectivité locale et inter-VLAN

```cmd
ping 192.168.10.1
ping 192.168.20.11
```
**Capture :**

![Test 1](./1.%20Test%20de%20connectivité%20locale%20et%20inter-VLAN%20depuis%20le%20Site%20A.png)

Vérification du routage inter-VLAN et de la communication avec la passerelle par défaut.

#### Test 2 : Connectivité inter-VLAN (même site)

```cmd
ping 192.168.110.11
ping 192.168.210.11
tracer 192.168.210.11

```
**Capture :**

![Test 2](./2.%20Test%20de%20connectivité%20inter-sites%20et%20chemin%20WAN%20(ping%20et%20traceroute)%20.png)

Validation du routage dynamique et de la redistribution via un traceroute multi-sauts.

#### Test 3 : Connectivité réseau interne vers DMZ

```cmd
ping 192.168.99.10
ping 192.168.99.20
```
**Capture :**

![Test 3](./3.%20Test%20de%20connectivité%20entre%20le%20réseau%20interne%20et%20la%20DMZ.png)

Test de connectivité ICMP réussi vers les adresses IP des serveurs de la zone DMZ.

#### Test 4 : Services DNS

```cmd
ping intranet.local
ping ftp.intranet.local
```
**Capture :**

![Test 4](./4.%20Test%20de%20résolution%20DNS%20interne.png)

Preuve de la résolution de noms pour les services intranet.local et ftp.intranet.local.

#### Test 5 : Services Web (HTTP)

- Ouvrir un navigateur web
- Aller à : `http://intranet.local` ou `http://192.168.99.10`
  
**Capture :**

![Test 5](./5.%20Test%20d'accès%20HTTP%20à%20l’intranet.png)

Validation de l'accès au portail Web de l'entreprise via le navigateur client.

#### Test 6 : Accès FTP

```cmd
ftp ftp.intranet.local
```
- **Username** : `ftpuser`
- **Password** : `ftppass`

**Capture :**

![Test 6](./6.%20Test%20d'accès%20FTP%20autorisé.png)

Succès de la connexion au serveur de fichiers avec authentification de l'utilisateur.

#### Test 7 : Restriction ACL (RH vers Finance)

```cmd
ping 192.168.20.11
```
**Capture :**

![Test 7](./7.%20Vérification%20de%20l’ACL%20RH%20vers%20Finance.png)

Preuve du blocage du trafic entre le VLAN RH et le VLAN Finance pour la confidentialité des données.

#### Test 8 : Restriction ACL (RH vers DMZ)

```cmd
ping 192.168.99.10
ping 192.168.99.20
```
**Capture :**

![Test 8](./8.%20Restriction%20d’accès%20du%20VLAN%20RH%20vers%20la%20DMZ%20par%20ACL.png)

Validation de la politique de sécurité limitant l'accès direct des postes RH aux serveurs DMZ.

#### Test 9 : Vérification des routes (R2)

```cmd
show ip route
```
![Test 9](./9.%20Vérification%20des%20routes%20inter-sites%20sur%20le%20routeur%20central%20R2.png)

Visualisation de la table de routage sur R2 montrant les routes OSPF et EIGRP redistribuées.

#### Test 10 : Vérification Switching (SW-C)

```cmd
show vlan brief
show interfaces trunk
```
![Test 10](./10.%20Vérification%20des%20VLAN%20et%20du%20lien%20trunk%20sur%20le%20switch%20du%20Site%20C.png)

Vérification de l'état des VLANs et de la configuration du Trunk 802.1q.

--- 

## 10. Résumé des commandes importantes

### 10.1 Diagnostic réseau

```
ping <IP>
traceroute <IP>
show ip route
show ip interface brief
show running-config
```

### 10.2 Sauvegarde

```
copy running-config startup-config
! OU
write memory
! OU
wr
```

### 10.3 Retour en arrière

```
! Supprimer une config
no <commande>

! Recharger la config sauvegardée
reload
```

---

## 11. Conclusion

Ce TP a permis de concevoir et déployer une **infrastructure réseau d'entreprise complète**, intégrant segmentation, routage dynamique, services réseau et sécurité.

### Compétences acquises :

- Mise en œuvre d'un réseau multi-sites
- Configuration avancée de routeurs et switches
- Déploiement de services réseau essentiels
- Sécurisation des flux via ACL
- Validation et dépannage réseau

### Conclusion générale

Ce projet illustre concrètement le fonctionnement d'un réseau professionnel moderne et opérationnel.

Il constitue une base solide pour comprendre et administrer des infrastructures réseau complexes en environnement réel.

---

## Index des éléments réseau

### 1. DHCP (Dynamic Host Configuration Protocol)

- **Fonction principale :** Automatiser l'attribution des **adresses IP**, **passerelles** et **serveurs DNS** aux postes et périphériques.
- **Rôle dans le réseau :** Facilite la **gestion centralisée** des adresses IP et réduit les erreurs humaines.
- **Avantages :**
  - Simplifie la configuration des équipements
  - Réduit les conflits d'adresses IP
  - Permet un déploiement rapide de nouveaux postes
- **Exemple concret :** Un PC sur Site A reçoit automatiquement l'adresse `192.168.10.11`, le masque `255.255.255.0`, la passerelle `192.168.10.1` et le DNS `192.168.99.10`.

---

### 2. DNS (Domain Name System)

- **Fonction principale :** Résolution de **noms de domaine** en adresses IP pour simplifier l'accès aux services.
- **Rôle dans le réseau :** Permet aux utilisateurs de se connecter aux services (HTTP, FTP, intranet) via des noms faciles à mémoriser.
- **Avantages :**
  - Simplifie l'accès aux ressources réseau
  - Centralise la gestion des noms et adresses
  - Réduit les erreurs liées à la saisie d'IP
- **Exemple concret :** `intranet.local` pointe vers `192.168.99.10`, et `ftp.intranet.local` vers `192.168.99.20`.

---

### 3. HTTP (HyperText Transfer Protocol) et FTP (File Transfer Protocol)

- **Fonction principale :** Fournir des **services applicatifs**.
  - **HTTP :** Accès aux pages web internes (intranet).
  - **FTP :** Transfert et gestion de fichiers sur un serveur dédié.
- **Rôle dans le réseau :** Hébergés dans la **DMZ** pour isoler les services accessibles depuis l'extérieur tout en protégeant le réseau interne.
- **Avantages :**
  - Sécurisation des services exposés
  - Centralisation des services pour tous les sites
  - Possibilité de filtrer et contrôler l'accès via ACL
- **Exemple concret :**
  - Serveur web `192.168.99.10` affiche l'intranet.
  - Serveur FTP `192.168.99.20` permet l'upload/download sécurisé de fichiers.

---

### 4. OSPF (Open Shortest Path First) et EIGRP (Enhanced Interior Gateway Routing Protocol)

- OSPF (**Open Shortest Path First**) est un **protocole de routage dynamique** utilisé par les routeurs pour **échanger automatiquement des informations sur les routes disponibles** dans un réseau.
- EIGRP (**Enhanced Interior Gateway Routing Protocol**) est également un **protocole de routage dynamique**, mais développé par Cisco, utilisé pour permettre aux routeurs d'échanger automatiquement des informations sur les routes et de déterminer le meilleur chemin pour le trafic réseau.
- **Fonction principale :** Protocoles de **routage dynamique** permettant aux routeurs de partager automatiquement les routes et de calculer les chemins optimaux.
- **Différences principales :**

| Critère | OSPF | EIGRP |
|---------|------|-------|
| Type | Link-State | Distance-Vector avancé |
| Calcul | Basé sur l'algorithme SPF | Basé sur métrique composite (delay, bandwidth, reliability) |
| Redistribution | Nécessaire pour communiquer avec d'autres protocoles | Peut redistribuer OSPF ou d'autres protocoles |
| Utilisation | Multi-sites, réseaux hiérarchiques | Réseaux intermédiaires et sites multi-protocoles |

- **Rôle dans le réseau :**
  - Assurer la **connectivité inter-sites**
  - Garantir la **redondance** en cas de panne d'un lien
  - Permettre **la redistribution OSPF ↔ EIGRP pour interconnecter tous les sites**
- **Avantages :**
  - Mise à jour automatique des tables de routage
  - Réduction des erreurs humaines
  - Résilience et optimisation des chemins réseau
- **Exemple concret :**
  - OSPF utilisé entre R1 et R2, EIGRP entre R2 et R3, avec redistribution sur R2 pour propager toutes les routes.

---

### 5. ACL étendues (Access Control List)

- **Fonction principale :** Mettre en place des **règles de filtrage** pour contrôler le trafic réseau.
- **Rôle dans le réseau :**
  - Bloquer ou autoriser des flux spécifiques entre VLANs ou vers la DMZ
  - Protéger les ressources critiques
- **Avantages :**
  - Sécurisation fine du réseau
  - Possibilité de filtrer selon IP, protocole, port ou direction
- **Exemple concret :**
  - Bloquer le VLAN RH (`192.168.30.0/24`) vers le VLAN Finance (`192.168.20.0/24`).
  - Autoriser seulement HTTP, FTP et DNS vers la DMZ.

---

### 6. QoS (Quality of Service)

- **Fonction principale :** Prioriser le **trafic critique** pour optimiser la performance du réseau, notamment sur les liaisons WAN.
- **Rôle dans le réseau :**
  - Garantir que le trafic IT prioritaire (applications critiques) soit servi avant les flux moins sensibles
  - Éviter la saturation des liaisons WAN
- **Avantages :**
  - Amélioration de la performance des applications critiques
  - Réduction des retards et pertes de paquets
  - Gestion efficace des ressources réseau
- **Exemple concret :**
  - Sur R1, le trafic du VLAN IT (`192.168.10.0/24`) est priorisé à 30% sur la liaison WAN vers R2.
 
## 📂 Téléchargement du projet
Vous pouvez télécharger le fichier de topologie Packet Tracer complet ici :
[Télécharger le projet Packet Tracer (PKT)](./TP%20Projet%20Multi-Sites%20Avanc%C3%A9%20avec%20VLAN,%20Routage%20Inter-VLAN%20et%20S%C3%A9curit%C3%A9.pkt)

