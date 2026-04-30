# `🌐` ︲ Documentation TP : Installer et configurer un service DNS

Ce dépôt présente un guide complet pour la mise en place et la configuration d'un service DNS (Domain Name System) sous Windows Server 2022, dans la continuité du TP DHCP.

---

## 📑 ︲ Sommaire

- [📘 ︲ Introduction](#introduction)
  - [❔ ︲ Contexte et objectifs](#contexte)
  - [🧰 ︲ Présentation des outils](#outils)
- [🧩 ︲ Mission 1 : Préparer le serveur Windows](#mission1)
- [🧩 ︲ Mission 2 : Préparer le serveur Web](#mission2)
- [🧩 ︲ Mission 3 : Tester le fonctionnement du serveur Web](#mission3)
- [🧩 ︲ Mission 4 : Configurer le service DNS](#mission4)
- [🧩 ︲ Mission 5 : Créer des hôtes et tester leur fonctionnement](#mission5)
- [🧩 ︲ Mission 6 : Créer des alias et tester leur fonctionnement](#mission6)
- [🧩 ︲ Mission 7 : Observer le fichier de configuration DNS](#mission7)
- [🧩 ︲ Mission 8 : Observer les serveurs racine](#mission8)
- [🧩 ︲ Mission 9 : Définir une redirection](#mission9)
- [🧩 ︲ Mission 10 : Mettre en place un serveur DNS secondaire](#mission10)
- [🧩 ︲ Mission 11 : Louer un nom de domaine](#mission11)
- [🧩 ︲ Mission 12 : Identifier les adresses IP de serveurs](#mission12)

---

# `📘` ︲ Introduction

---

## `❔` ︲ Contexte et objectifs du TP

> [!NOTE]
> **Ce TP fait suite à la mise en place du service DHCP pour l'entreprise Gal Cosmetic.** L'objectif est désormais de mettre en place le service DNS (Domain Name System) afin de permettre la résolution de noms au sein du réseau local. Cela inclut la création de zones directe et inversée, d'enregistrements d'hôtes, d'alias (CNAME), ainsi que la mise en place d'un serveur DNS secondaire pour assurer la redondance.

---

## `🧰` ︲ Présentation des outils et prérequis

> [!IMPORTANT]
> **Présentation des outils et prérequis :**
>
> - `🖥️` ︱ **Serveur principal :** Windows Server 2022 — `srv-win` — `192.168.0.1`
> - `🖥️` ︱ **Serveur secondaire :** Windows Server 2022 — `srv-win2` — `192.168.0.2`
> - `🌍` ︱ **Serveur Web 1 :** Linux/Apache — `srv-web` — `192.168.0.3`
> - `🌍` ︱ **Serveur Web 2 :** Linux/Apache — `srv-web2` — `192.168.0.4`
> - `🗄️` ︱ **Serveur BDD 1 :** `srv-bd` — `192.168.0.5`
> - `🗄️` ︱ **Serveur BDD 2 :** `srv-bd2` — `192.168.0.6`
> - `💻` ︱ **Client :** Windows 11
> - `🛠️` ︱ **Outil réseau :** Wireshark
> - `🖼️` ︱ **Virtualisation :** Hyperviseur de Type 2 (mode Réseau Interne)

### Schéma réseau

```mermaid
graph TD

  
    Routeur["Routeur/Box 192.168.0.254"]

    subgraph Serveurs_Gal_Cosmetic["Serveurs Gal Cosmetic"]

        Switch["🔀 Switch"]

        SRV_WIN["srv-win 
        192.168.0.1"]
        SRV_WIN2["srv-win2
        192.168.0.2"]
        SRV_WEB["srv-web
        192.168.0.3"]
        SRV_WEB2["srv-web2
        192.168.0.4"]
        SRV_BD["srv-bd
        192.168.0.5"]
        SRV_BD2["srv-bd2
        192.168.0.6"]

        Switch --- SRV_WIN
        Switch --- SRV_WIN2
        Switch --- SRV_WEB
        Switch --- SRV_WEB2
        Switch --- SRV_BD
        Switch --- SRV_BD2
    end

    Routeur --- Switch

    style SRV_WIN fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style SRV_WIN2 fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style SRV_WEB fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style SRV_WEB2 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style SRV_BD fill:#FAEEDA,stroke:#854F0B,color:#633806
    style SRV_BD2 fill:#FAEEDA,stroke:#854F0B,color:#633806
    style Routeur fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style Switch fill:#F1EFE8,stroke:#5F5E5A,color:#444441
```
---

# `🧩` ︲ Mission 1 : Préparer le serveur Windows

## `🎯` ︲ Objectif

Renommer le serveur, installer le rôle DNS et configurer le serveur pour qu'il utilise sa propre adresse IP comme serveur DNS.

> [!TIP]
> Pour afficher les captures d'écrans, cliquez sur le menu déroulant avec l'émoji : 📸

## `🛠️` ︲ Étape 1 : Renommer le serveur

1. Ouvrir le `Gestionnaire de serveur` si celui ci ne l'est pas déjà.
2. Aller dans `Serveur local`.
3. Cliquer sur le nom actuel du serveur.
4. Dans l'onglet `Nom de l'ordinateur`, cliquer sur `Modifier`.
5. Saisir le nouveau nom : `srv-win`.
6. Redémarrer le serveur pour appliquer le changement.

<details>
  <summary><strong>📸︲Renommage du serveur</strong></summary>
  <img src="captures/2026-03-25 12_06_10-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
</details>

## `🛠️` ︲ Étape 2 : Installer le rôle DNS

1. Se rendre dans le `Gestionnaire de serveur`.
2. Cliquer sur `Gérer` puis `Ajouter des rôles et fonctionnalités`.
3. Sélectionner `Installation basée sur un rôle ou une fonctionnalité`.
4. Choisir le `serveur local`.
5. Cocher le rôle `Serveur DNS`.
6. `Valider` et lancer l'installation.

<details>
  <summary><strong>📸︲Installation du rôle DNS</strong></summary>
  <img src="captures/2026-03-25 12_02_33-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-03-25 12_09_03-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
</details>

## `🛠️` ︲ Étape 3 : Configurer l'adresse IP du serveur DNS

Dans la configuration réseau du serveur, il faut indiquer `sa propre adresse IP` comme **serveur DNS préféré** :

Pour ce faire, 
1. Dans le `Gestionnaire de serveur` puis cliquer sur l'onglet `Serveur local` dans le menu gauche.
1. Cliquer sur `l'IP`.
2. Dans la nouvelle fenêtre qui s'affiche `double cliquer` sur la carte `Ethernet`, puis `Protocole Internet version 4`.
3. Dans la partie `Utiliser l'addresse de serveur DNS suivante :` saisir `192.168.0.1` dans `Serveur DNS préféré`. 

<details>
  <summary><strong>📸︲Configuration DNS</strong></summary>
  <img src="captures/2026-03-25 12_22_03-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
</details>

## `🛠️` ︲ Étape 4 : Ajouter l'option DNS dans le service DHCP

Afin que les clients reçoivent automatiquement l'adresse du serveur DNS :

1. Ouvrir la console `DHCP`, pour ce faire dans le `Gestionnaire de Serveur` cliquer sur `Outils` puis `DHCP`.
2. Aller dans `win-srv` → `IPv4` → `Étendue [192.168.0.0] Réseau local` puis `Options d'étendue`.
3. `Clic droit` et `Configurer les options`.
4. Cocher l'option `006 - Serveurs DNS`.
5. `Ajouter` l'adresse IP : `192.168.0.1` puis valider.

<details>
  <summary><strong>📸︲Installation du Service DNS</strong></summary>
  <img src="captures/2026-03-25 12_10_27-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-03-25 12_20_56-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-03-25 12_21_30-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-03-25 12_23_12-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
</details>

## `🛠️` ︲ Étape 5 : Vérification côté client

Sur le client Windows 11 :

1. Ouvrir une `Invite de commandes`.
2. Renouveler l'adresse IP :

```bash
ipconfig /release
```
et 
```bash
ipconfig /renew
```

3. Vérifier que le serveur DNS est bien `192.168.0.1` avec la commande:

```bash
ipconfig /all
```

<details>
  <summary><strong>📸︲Vérification</strong></summary>
  <img src="captures/2026-04-30 13_19_27-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
</details>

---

# `🧩` ︲ Mission 2 : Préparer le serveur Web

## `🎯` ︲ Objectif

Démarrer le serveur Web et lui attribuer une adresse IP statique conforme au schéma réseau.

---

## `🛠️` ︲ Procédure

1. **Démarrer** la VM `srv-web`.
2. Identifier son adresse IP actuelle (attribuée dynamiquement par le DHCP) :

```
ip a
```



3. **Modifier l'adressage** pour passer en IP statique :

| Paramètre  | Valeur          |
| ---------- | --------------- |
| Adresse IP | `192.168.0.3`   |
| Masque     | `255.255.255.0` |
| DNS        | `192.168.0.1`   |

Pour ce faire :

```bash
sudo nano /etc/network/interfaces
```

Observer la ligne `iface eth0 inet dhcp` puis remplacer et ajouter :

```bash
iface eth0 inet static
address 192.168.0.3
netmask 255.255.255.0
dns-nameservers 192.168.0.1
```
Puis redémarrer avec :
```
sudo reboot
```
<details>
  <summary><strong>📸︲Identification de l'IP</strong></summary>
  <img src="captures/2026-04-05 11_36_59-GALL COSMETIC - Debian sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
</details>

<details>
  <summary><strong>📸︲Configuration IP statique du serveur Web</strong></summary>
  <img src="captures/2026-04-05 11_40_23-GALL COSMETIC - Debian sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
  <br>
  <img src="captures/2026-04-05 11_43_00-GALL COSMETIC - Debian sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
</details>

---

# `🧩` ︲ Mission 3 : Tester le fonctionnement du serveur Web

## `🎯` ︲ Objectif

Vérifier la communication entre le client et le serveur Web, et observer le comportement du DNS via Wireshark.

---

## `🛠️` ︲ Étape 1 : Test de ping

Depuis le client Windows 11, exécuter :

```bash
ping 192.168.0.3
```

Le ping doit aboutir, confirmant la communication réseau entre les deux machines.

<details>
  <summary><strong>📸︲Ping vers le serveur Web</strong></summary>
  <img src="captures/2026-04-30 11_25_41-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
</details>

---

## `🛠️` ︲ Étape 2 : Accès via navigateur (par IP)

Sur le client, ouvrir un navigateur et saisir :

```
http://192.168.0.3
```

👉 La page d'accueil du serveur Web doit s'afficher.

<details>
  <summary><strong>📸︲Page d'accueil du serveur Web (accès par IP)</strong></summary>
  <img src="captures/2026-04-30 10_50_30-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
</details>

---

## `🛠️` ︲ Étape 3 : Capture Wireshark sur le port 53

1. Ouvrir **Wireshark** sur le client Windows 11.
2. Démarrer une capture sur l'interface **Ethernet** avec le filtre :

```
port 53
```

3. Dans un navigateur, accéder à :

```
http://srv-web.galcosmetic.fr
```

4. Stopper la capture et repérer la première trame **DNS query**.
5. Observer le champ **Domain Name System**.
6. Faire un `clic droit` sur l'attribut **Transaction ID** → `Appliquer comme un filtre` → `Sélectionné`.

### Analyse des trames DNS

| Trame            | Rôle                                                            |
| ---------------- | --------------------------------------------------------------- |
| **DNS Query**    | Le client demande l'IP correspondant à `srv-web.galcosmetic.fr` |
| **DNS Response** | Le serveur DNS répond avec l'adresse IP `192.168.0.3`           |

> [!NOTE]
> **Le port 53** est le port standard utilisé par le protocole DNS pour la résolution de noms de domaine (en UDP pour les requêtes classiques, en TCP pour les transferts de zone).

<details>
  <summary><strong>📸︲Capture Wireshark port 53</strong></summary>
  <img src="captures/2026-04-30 10_58_51-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
  <br>
  <img src="captures/2026-04-30 11_01_41-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
  <br>
  <img src="captures/2026-04-30 11_03_41-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
  <br>
  <img src="captures/2026-04-30 11_05_01-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
  <br>
  <img src="captures/2026-04-30 11_06_24-GALL COSMETIC - Windows 11 sur STBN-DELL5510 - Connexion à un ordinateur virtuel.png"/>
</details>

---

# `🧩` ︲ Mission 4 : Configurer le service DNS


## `🎯` ︲ Objectif

Créer les zones de recherche directe et inversée pour le domaine `galcosmetic.fr`.

---

## `🛠️` ︲ Étape 1 : Créer la zone de recherche directe

1. Ouvrir le `Gestionnaire de serveur` puis `Outils` et `DNS`.
2. `Clic droit` sur `Zones de recherche directes` puis `Nouvelle zone`.
3. Choisir `Zone principale`.
4. Saisir le nom de zone : `galcosmetic.fr`
5. Laisser l'option de `création automatique du fichier de zone` cochée.
6. `Terminer` l'assistant.

<details>
  <summary><strong>📸︲Création de la zone de recherche directe</strong></summary>
  <img src="captures/2026-04-30 11_32_21-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_33_14-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_34_25-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_35_05-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
</details>

---

## `🛠️` ︲ Étape 2 : Créer la zone de recherche inversée

1. `Clic droit` sur **`Zones de recherche inversées`** → **`Nouvelle zone`**.
2. Choisir **`Zone principale`**.
3. Saisir l'**ID réseau** : `192.168.0`
4. Laisser l'option de **création automatique du fichier de zone** cochée.
5. **Terminer** l'assistant.

<details>
  <summary><strong>📸︲Création de la zone de recherche inversée</strong></summary>
  <img src="captures/2026-04-30 11_35_50-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_36_42-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_37_28-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_38_16-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_39_00-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
  <br>
  <img src="captures/2026-04-30 11_40_29-GALL COSMETIC - Windows Serveur 2025 sur STBN-DELL5510 - Connexion à un ordinate.png"/>
</details>



# `📖` ︲ Généralités et Concepts DNS

## `🔍` ︲ Rôles des zones Directe et Inversée

*   **Zone de recherche directe** : C'est le rôle principal du DNS. Elle permet de traduire un nom d'hôte (FQDN) en une adresse IP. Par exemple, lorsqu'un utilisateur saisit `srv-web.galcosmetic.fr`, le DNS interroge la zone directe pour renvoyer l'IP `192.168.0.3`.
*   **Zone de recherche inversée** : Elle effectue l'opération inverse en traduisant une adresse IP en un nom d'hôte. Elle est essentielle pour certains protocoles de sécurité, pour le diagnostic réseau (ex: `ping -a`) et pour éviter que les emails ne soient considérés comme spams.

## `🏷️` ︲ Signification des termes techniques

*   **SOA (Start Of Authority)** : L'enregistrement de "début d'autorité". Il s'agit de l'enregistrement le plus important d'une zone. Il désigne le serveur DNS principal qui détient les informations de la zone et contient des paramètres de gestion comme le numéro de série (utilisé pour la réplication vers les serveurs secondaires) et les durées de cache (TTL).
*   **NS (Name Server)** : Cet enregistrement liste les serveurs DNS qui font autorité pour le domaine. Il indique quel(s) serveur(s) possède(nt) les fichiers de zone et peuvent répondre aux requêtes concernant ce domaine.

---

**Esteban GUILLERMIN EGIDIO** | **BTS SIO SISR**