# 🏦 CCNA2 — VLANs + Router-on-a-Stick + STP + DHCPv4 | AfriBank Bénin

![Cisco](https://img.shields.io/badge/Cisco-CCNA2-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-Router--on--a--Stick-purple?style=for-the-badge)
![VLANs](https://img.shields.io/badge/VLANs-4-red?style=for-the-badge)
![PCs](https://img.shields.io/badge/PCs-12-yellow?style=for-the-badge)
![DHCP](https://img.shields.io/badge/DHCPv4-4%20Pools-green?style=for-the-badge)

---

## 📋 Description

Ce TP simule l'infrastructure réseau de la banque **AfriBank Bénin** 🇧🇯.
Le réseau est segmenté en **4 VLANs** représentant les départements
de la banque. Le routage inter-VLAN est assuré par **Router-on-a-Stick**,
la stabilité du réseau par **STP Rapid PVST+**, et la distribution des
adresses IP par **DHCPv4** avec **4 pools automatiques**.

### Objectifs
- ✅ Créer et configurer **4 VLANs** + VLAN natif 99
- ✅ Configurer **Router-on-a-Stick** sur Router0
- ✅ Configurer **STP Rapid PVST+** avec Root Bridge
- ✅ Configurer **DHCPv4** avec 4 pools sur Router0
- ✅ Tester la distribution automatique des IP sur 12 PC

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🌐 Routeur | Cisco 1941 | Router0 | Router-on-a-Stick + DHCP |
| 🔌 Switch | Cisco 2960-24TT | Switch1 | Root Bridge — switch central |
| 🔌 Switch | Cisco 2960-24TT | Switch0 | Switch VLAN 10/20 |
| 🔌 Switch | Cisco 2960-24TT | Switch2 | Switch VLAN 30/40 |
| 💻 PC | PC-PT | PC1-PC3 | Département Direction (VLAN10) |
| 💻 PC | PC-PT | PC4-PC6 | Département Finance (VLAN20) |
| 💻 PC | PC-PT | PC7-PC9 | Département Caisse (VLAN30) |
| 💻 PC | PC-PT | PC10-PC12 | Département Sécurité (VLAN40) |

---

## 🗺️ Topologie

```
                          [Router0]
                              │
                          Gig0/0
                              │
                          Gig0/2
                              │
                         [Switch1]
                        (Root Bridge)
                         ╱          ╲
                    Gig0/1          Fa0/23
                     ╱                  ╲
               Gig0/2                  Fa0/23
                ╱                          ╲
          [Switch0]                    [Switch2]
           ╱      ╲                     ╱      ╲
      Fa0/1-3   Fa0/4-6            Fa0/1-3   Fa0/4-6
         │          │                  │          │
    PC1-PC3    PC4-PC6            PC7-PC9   PC10-PC12
    VLAN10      VLAN20             VLAN30      VLAN40
```

<p align="center">
  <img src="images/topologie.png" width="800">
  <br>
  <em>Capture Cisco Packet Tracer — Topologie AfriBank</em>
</p>

---

## 🔌 Câblage

| De | Port | Vers | Port | Rôle |
|----|------|------|------|------|
| Router0 | Gig0/0 | Switch1 | Gig0/2 | Trunk router-on-a-stick |
| Switch1 | Gig0/1 | Switch0 | Gig0/2 | Trunk VLAN10, VLAN20 |
| Switch1 | Fa0/23 | Switch2 | Fa0/23 | Trunk VLAN30, VLAN40 |
| Switch0 | Fa0/1-3 | PC1, PC2, PC3 | Fa0 | Accès VLAN10 Direction |
| Switch0 | Fa0/4-6 | PC4, PC5, PC6 | Fa0 | Accès VLAN20 Finance |
| Switch2 | Fa0/1-3 | PC7, PC8, PC9 | Fa0 | Accès VLAN30 Caisse |
| Switch2 | Fa0/4-6 | PC10, PC11, PC12 | Fa0 | Accès VLAN40 Sécurité |

---

## 📊 Plan d'adressage

| VLAN | Nom | Réseau | Passerelle | Plage DHCP |
|------|-----|--------|-----------|------------|
| 10 | DIRECTION | 192.168.10.0/24 | 192.168.10.1 | .11 - .254 |
| 20 | FINANCE | 192.168.20.0/24 | 192.168.20.1 | .11 - .254 |
| 30 | CAISSE | 192.168.30.0/24 | 192.168.30.1 | .11 - .254 |
| 40 | SECURITE | 192.168.40.0/24 | 192.168.40.1 | .11 - .254 |
| 99 | NATIF | — | — | Aucun PC |

---

## ⚙️ Configuration complète

### 🔧 Switch1 — Root Bridge (switch central)

```cisco
enable
configure terminal
hostname Switch1

vlan 10
name DIRECTION
exit
vlan 20
name FINANCE
exit
vlan 30
name CAISSE
exit
vlan 40
name SECURITE
exit
vlan 99
name NATIF
exit

! Priorité STP la plus basse = Root Bridge
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40 root primary

interface gig0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
exit

interface gig0/2
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
exit

interface fa0/23
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
exit

end
write
```

---

### 🔧 Switch0 — VLAN 10 & 20

```cisco
enable
configure terminal
hostname Switch0

vlan 10
name DIRECTION
exit
vlan 20
name FINANCE
exit
vlan 99
name NATIF
exit

spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40 root secondary

interface gig0/2
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
exit

interface range fa0/1-3
switchport mode access
switchport access vlan 10
spanning-tree portfast
exit

interface range fa0/4-6
switchport mode access
switchport access vlan 20
spanning-tree portfast
exit

end
write
```

---

### 🔧 Switch2 — VLAN 30 & 40

```cisco
enable
configure terminal
hostname Switch2

vlan 30
name CAISSE
exit
vlan 40
name SECURITE
exit
vlan 99
name NATIF
exit

spanning-tree mode rapid-pvst

interface fa0/23
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
exit

interface range fa0/1-3
switchport mode access
switchport access vlan 30
spanning-tree portfast
exit

interface range fa0/4-6
switchport mode access
switchport access vlan 40
spanning-tree portfast
exit

end
write
```

---

### 🔧 Router0 — Router-on-a-Stick + DHCP

```cisco
enable
configure terminal
hostname Router0

! Sous-interfaces pour chaque VLAN
interface gig0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

interface gig0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit

interface gig0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
exit

interface gig0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
exit

interface gig0/0.99
encapsulation dot1Q 99 native
exit

! Activer l'interface physique
interface gig0/0
no shutdown
exit

! Pools DHCP — un par VLAN
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool VLAN10-DIRECTION
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit

ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool VLAN20-FINANCE
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit

ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp pool VLAN30-CAISSE
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 8.8.8.8
exit

ip dhcp excluded-address 192.168.40.1 192.168.40.10
ip dhcp pool VLAN40-SECURITE
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 8.8.8.8
exit

end
write
```

---

## 🔍 Commandes de vérification

```cisco
! Vue globale du routeur
Router0# show ip interface brief
Router0# show ip route

! Voir les baux DHCP distribués
Router0# show ip dhcp binding

! Configuration VLAN et trunk
Switch1# show vlan brief
Switch1# show interfaces trunk

! État du Spanning Tree
Switch1# show spanning-tree
Switch0# show spanning-tree
Switch2# show interfaces trunk

! Configuration complète
Switch1# show running-config
```

---

### 📊 Résultat attendu — show spanning-tree (Switch1)

```
VLAN0010
Root ID   Priority    24586
          Address     0001.9614.5A32
          This bridge is the root

Bridge ID Priority    24586
          Address     0001.9614.5A32
```

| Élément | Signification |
|---------|---------------|
| **This bridge is the root** | Switch1 = Root Bridge ✅ |
| **Gig0/1, Gig0/2, Fa0/23** | Tous en forwarding (FWD) |
| **Topologie étoile** | Aucune boucle = aucun port bloqué |

---

## 🧪 Tests de connectivité

```
✅ PC1 (VLAN10)  → obtient IP via DHCP (192.168.10.x)
✅ PC4 (VLAN20)  → obtient IP via DHCP (192.168.20.x)
✅ PC7 (VLAN30)  → obtient IP via DHCP (192.168.30.x)
✅ PC10 (VLAN40) → obtient IP via DHCP (192.168.40.x)

✅ PC1 → ping 192.168.20.11   (inter-VLAN via Router0)
✅ PC7 → ping 192.168.40.11   (inter-VLAN via Router0)
```

---

## 🎭 Scénario — Vérification DHCP automatique

### Sur chaque PC

```
PC1> ipconfig /release
PC1> ipconfig /renew
```

### Vérifier le bail sur le routeur

```cisco
Router0# show ip dhcp binding
```

**Résultat :**
```
IP address       Client-ID/Hardware address    Lease expiration
192.168.10.11    0001.4271.5A01                 --- Infinite ---
192.168.20.11    0001.4271.5A02                 --- Infinite ---
192.168.30.11    0001.4271.5A03                 --- Infinite ---
192.168.40.11    0001.4271.5A04                 --- Infinite ---
```

---

## 🛠️ Dépannage

| Problème | Cause | Solution |
|---------|-------|---------|
| PC n'obtient pas d'IP | Gig0/0 down sur Router0 | `no shutdown` sur Gig0/0 |
| Ping inter-VLAN échoue | VLAN natif différent | Harmoniser `native vlan 99` partout |
| Trunk down entre switches | Port en mode access | `switchport mode trunk` |
| Pool DHCP inactif | Adresse exclue mal définie | Vérifier `ip dhcp excluded-address` |

```cisco
! Diagnostic détaillé
Router0# show ip dhcp pool
Switch1# show interfaces status

! Réinitialiser un bail DHCP en conflit
Router0# clear ip dhcp binding *
```

---

## 💡 Points clés à retenir

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `interface gig0/0.10` | Créer une sous-interface pour le VLAN10 |
| `encapsulation dot1Q 10` | Taguer la sous-interface au VLAN10 |
| `spanning-tree vlan 10,20,30,40 root primary` | Forcer Switch1 en Root Bridge |
| `ip dhcp pool VLAN10-DIRECTION` | Créer un pool DHCP par VLAN |
| `show ip dhcp binding` | Vérifier les baux distribués |
| `show spanning-tree` | Vérifier l'état des ports STP |

---

## 📊 Comparatif avant/après

| | Sans segmentation | Avec VLAN + Router-on-a-Stick |
|---|---|---|
| **Sécurité** | Tout le trafic mélangé | Départements isolés (VLAN) |
| **Adressage** | Manuel | Automatique (DHCP, 4 pools) |
| **Routage inter-VLAN** | ❌ Impossible | ✅ Via Router0 |
| **Stabilité réseau** | ❌ Boucles possibles | ✅ Root Bridge STP défini |

---

## 🛠️ Outils

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-8.x-orange?style=flat-square&logo=cisco)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-15.x-blue?style=flat-square)
![GitHub](https://img.shields.io/badge/GitHub-black?style=flat-square&logo=github)

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Libre d'utilisation pour l'apprentissage et la formation réseau.
