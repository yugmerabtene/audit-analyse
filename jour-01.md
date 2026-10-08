# Audit et Analyse des Réseaux — JOUR 1

## Poser les fondements, cartographier, observer, mesurer (08/10/2026)

---

## Comment lire ce support

Ce support se lit comme un livre. Chaque chapitre suit toujours le même parcours :

1. **Un préambule** qui pose le problème en langage courant, avec une image concrète.
2. **Le vocabulaire en clair**, qui traduit chaque terme technique avant son emploi.
3. **Le contenu du chapitre**, qui pose les notions et les tableaux de référence.
4. **Les techniques, pas à pas**, qui expliquent chaque méthode avant de la faire exécuter.
5. **Un LAB**, qui met les mains sur le clavier et produit un livrable réutilisé ensuite.
6. **Des exercices**, pour vérifier l acquis.

Le mot « LAB » désigne un laboratoire pratique. Aucun terme n’est employé sans avoir été expliqué dans le tableau de vocabulaire.

---

## Objectifs du Jour

- Comprendre les principes, les objectifs, le périmètre et la méthodologie d’un audit réseau
- Élaborer la cartographie et l’inventaire du réseau : topologie, adressage, services, points critiques
- Analyser le trafic : capturer, filtrer, interpréter, détecter les anomalies
- Mesurer les performances : latence, bande passante, pertes, jitter et qualité de service (QoS)

---

## Planning du Jour

| Chapitre | Horaires | Thème | Durée |
|----------|----------|-------|-------|
| C1 | 9h00–10h30 | M1 — Principes d’audit réseau | 1h30 |
| — | 10h30–10h45 | Pause | 15min |
| C2 | 10h45–12h15 | M2 — Cartographie et inventaire | 1h30 |
| — | 12h15–13h45 | Pause déjeuner | 1h30 |
| C3 | 13h45–15h15 | M3 — Analyse du trafic | 1h30 |
| — | 15h15–15h30 | Pause | 15min |
| C4 | 15h30–17h00 | M4 — Mesure de performance | 1h30 |

Volume pédagogique : 6 h de formation (quatre chapitres de 1 h 30), 2 h de pauses.

---

## Le fil du jour

La journée est une chaîne : chaque LAB produit un livrable que le suivant consomme.

| Chapitre | LAB | Produit réutilisé par la suite |
|----------|-----|-------------------------------|
| C1 — M1 | Préparer l’audit | `~/audit/inventaire-brut.txt` : adresses, route par défaut, voisins, hôtes actifs |
| C2 — M2 | Cartographier et inventorier | `~/audit/inventaire-complet.md` : tableau d’inventaire, alimenté par le fichier de C1 |
| C3 — M3 | Analyser le trafic | `~/audit/audit_capture.pcap` : preuve de trafic horodatée |
| C4 — M4 | Mesurer les performances | `~/audit/mesures-performance.txt` : mesures et seuils, base des constats de performance |

Le même réseau de laboratoire sert toute la journée : `172.16.0.0/24`, poste d’audit en `172.16.0.10`.

---

## Chapitre 1 — M1 : Principes d’audit réseau (9h00–10h30)

### Préambule — ce que vient faire un auditeur, en clair

Un réseau est une installation qu’on ne voit pas mais dont tout le monde dépend. Quand la messagerie tombe, personne ne songe aux câbles ni aux routeurs : on songe à l’écran noir. L’auditeur est précisément la personne à qui l’on songe quand on veut savoir **pourquoi** — et surtout **ce qu’il faut faire** pour que cela n’arrive plus.

L’image utile est celle du contrôle technique automobile. On ne vient pas pour bricoler la voiture : on vient pour le carnet de bord, les freins, les pneus, l’état des feux. Le rapport d’audit joue le rôle du carnet : il décrit l’état réel, ce qui fonctionne, ce qui ne fonctionne pas, et par quoi commencer. Un rapport d’audit utile ne dit jamais seulement « le réseau va mal ». Il dit : à tel endroit, tel service, tel risque, avec telle preuve, et voici l’action prioritaire.

Mais un contrôle technique commence toujours par une décision : qu’est-ce que l’on va contrôler ? Un moteur, une voiture, ou une flotte entière ? Cette première décision s’appelle le **périmètre**, et c’est elle qui évite les deux pièges classiques : l’audit flou, qui promet trop et ne prouve rien, et l’audit sans fin, qui n’aboutit jamais parce que la frontière n’a pas été posée.

Le second point, moins évident, est l'**autorisation**. Sonder un réseau qui n’est pas le sIEN, ou le sien mais sans mandat écrit, est un délit. L’auditeur travaille toujours sous un document qui nomme le périmètre, les horaires et les personnes à contacter. Ce document, on l’appelle un ordre de mission.

Le troisième point est la **traçabilité**. Une affirmation d’audit qui n’est pas prouvée ne vaut rien. Chaque constat doit s’appuyer sur une donnée reproductible : la commande tapée, sa sortie, la date et l’heure. C’est cette discipline, et non l’intuition, qui distingue un auditeur d’un utilisateur mécontent.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Audit réseau | Examen méthodique de l’infrastructure pour trouver les points forts, les faiblesses et les risques | C1, définition |
| Périmètre | La frontière de ce qui est contrôlé, et de ce qui ne l’est pas | C1, ordre de mission |
| Ordre de mission | Document écrit qui autorise l’audit et en fixe le périmètre | C1, livrable |
| Méthodologie | La progression en étapes qui garantit qu’aucune vérification n’est oubliée | C1, cinq phases |
| Partie prenante | Le service ou la personne qui attend le rapport et qui subit ses conclusions | C1, exemple |
| Preuve | Donnée reproductible qui soutient un constat : commande, sortie, date | C1, collecte |
| Constat | Écart constaté entre l’état observé et l’état attendu | C1, analyse |
| Recommandation | Action corrective proposée, priorisée | C1, rapport |
| ICMP | Protocole de diagnostic réseau, employé ici par la découverte d’hôtes | C1, étape de découverte |
| ARP | Protocole qui traduit une adresse IP en adresse de carte réseau, pour poser une trame | C1, filtrage des trames |

### Objectifs du chapitre

- Définir un audit réseau, ses objectifs et son périmètre
- Identifier les cinq phases d’une méthodologie d’audit
- Déterminer les informations à collecter : actifs, flux, politiques, contraintes
- Préparer un cadre d’audit exploitable et traçable

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 22min |
| Techniques, pas à pas | 7min |
| LAB | 38min |
| Exercices | 15min |

### Contenu du chapitre (22min)

#### 1. Définition, objectifs et périmètre (11min)

Un audit se construit sur des bases claires : c’est l’objet du tableau suivant.

| Concept | Définition | Application |
|---------|-----------|-------------|
| Audit réseau | Examen méthodique et structuré de l’infrastructure réseau | Identifier points forts, faiblesses, écarts et risques |
| Objectifs | Mesurer, vérifier, documenter, recommander | Améliorer la sécurité, de la résilience et des performances |
| Périmètre | Sites, segments, équipements, services, flux | Délimiter clairement ce qui est audité, inclusions et exclusions |
| Parties prenantes | Exploitation, sécurité, architecture | Valider attentes, contraintes et délais |

Le périmètre se formule toujours par inclusion et par exclusion. « On audite le segment de production `172.16.0.0/24`, hors segment de sauvegarde et hors accès distant » est un périmètre utilisable. « On regarde le réseau » ne l’est pas.

#### 2. Méthodologie d’audit (11min)

La méthodologie s’articule en cinq phases, chacune produisant un livrable exploitable pour la suite.

| Phase | Actions | Livrables |
|------|--------|-----------|
| Pré-audit | Cahier des charges, périmètre, autorisations | Ordre de mission, périmètre validé |
| Collecte | Inventaire, configuration, captures, mesures | Données brutes, preuves |
| Analyse | Interprétation, corrélation, anomalies | Hypothèses, constats |
| Validation | Tests de cohérence, vérifications | Constats validés |
| Rapport | Recommandations, priorisation | Rapport d’audit et plan d’actions |

L’ordre n’est pas négociable : on ne valide pas ce que l’on n’a pas collecté, et on ne rapporte pas ce que l’on n’a pas validé. Chaque phase a une porte de sortie : si la collecte est incomplète, on retourne à la collecte, on ne passe pas à l’analyse.

### Les techniques, pas à pas

**Technique 1 — Recenser ses propres outils avant de sonder autrui.** Avant toute commande, on vérifie que l’outil existe et ce qu’il mesure exactement. `command -v` répond par un chemin si le programme est présent, et ne renvoie rien sinon. Cette vérification évite un rapport sans fond sur une mesure que l’outil n’a pas faite.

**Technique 2 — Établir l’inventaire de départ.** Les trois premières commandes d’un audit donnent le point de départ : `ip -br addr show` (mes adresses), `ip route show` (la route par défaut, donc la sortie du réseau), `ip neigh show` (les voisins directs, c’est-à-dire ce qui est à un saut). Elles donnent l’état du poste d’audit, pas celui du réseau audité : c’est un point de départ, pas un résultat.

**Technique 3 — Documenter au fil de l’eau.** Toute commande dont la sortie sert au rapport est redirigée vers un fichier de preuve, par `tee` ou par `>`. Un rapport d’audit se relit trois mois plus tard, souvent par une autre personne : sans la sortie brute, il n’est plus vérifiable. L’horodatage n’est pas dans le nom du fichier mais dans son contenu, puisque `ip` écrit l’heure sur chaque ligne.

**Technique 4 — Chercher l’inattendu, pas seulement l’attendu.** La valeur d’un inventaire n’est pas dans la liste des services connus, mais dans le service qui n’a rien à faire là. Tout port ouvert non annoncé est un constat, même s’il fonctionne.

### LAB — Préparer un audit réseau (38min)

#### Prérequis

- Un poste Linux de la famille Debian, sur le segment `172.16.0.0/24` : Kali Linux, Parrot OS ou l’image Ubuntu du laboratoire. Les trois reposent sur la même base, utilisent les mêmes paquets et la même commande d’installation `apt`.
- Les machines de la maquette sont préparées à l’avance par le formateur, qui monte le laboratoire puis le vérifie. Le poste d’audit, le routeur, l’hôte applicatif et le serveur de mesure sont donc déjà joignables ; si ce n’est pas le cas, le signaler avant de commencer.
- Les droits administrateur disponibles pour les étapes qui le demandent. Le poste d’audit est un compte ordinaire : la différence avec un compte privilégié est exploitée au chapitre 3.
- Aucune donnée de ce LAB n’est réutilisée par un chapitre précédent : c’est le point de départ de la journée. En revanche, ce LAB produit `inventaire-brut.txt`, que le chapitre 2 consommera.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Client d’audit | Linux, famille Debian — IP : 172.16.0.10/24 |
| Réseau audité | Segment de test : 172.16.0.0/24 |
| Équipements du lab | Routeur 172.16.0.1, hôte applicatif 172.16.0.20, serveur iperf3 172.16.0.50 : ce sont les trois seules machines du segment, avec le poste d’audit lui-même |
| Seconde interface du poste | `eth1` en 192.168.50.10/24 : le poste est raccordé au second réseau du laboratoire, qui n’est audité que le jour 2. L’interface n’est pas exploitée pendant l’audit de `172.16.0.0/24` ; ses adresses sont bien relevées par `ip`, mais elles n’entrent pas dans l’inventaire du chapitre 2 |
| Services ouverts sur 172.16.0.20 | 22/tcp (SSH), 23/tcp (telnet, volontairement exposé), 80/tcp (HTTP) ; 443/tcp et 3389/tcp sont fermés. Le telnet est la faille du laboratoire : c’est sur elle que repose le constat C-01 du jour 2 |
| Outils | `ip` et `ss` de la suite iproute2, `ping` d’iputils, `nmap`, `tee`, `whoami`, `nano`. `traceroute` et `tracepath` ne servent qu’au chapitre 2 |
| Fichier produit | `~/audit/inventaire-brut.txt` |

#### Étapes du LAB

**Étape 0 — Vérifier les outils (3min)**

La technique 1 s’applique ici : on vérifie que chaque outil est présent avant de s’en servir, faute de quoi le rapport s’appuierait sur une mesure que l’outil n’a pas faite.

```bash
for o in ip ping nmap ss tee whoami nano; do printf '%-9s %s\n' "$o" "$(command -v "$o" || echo ABSENT)"; done
```

> **Résultat attendu :** sept lignes, chacune affichant un chemin, par exemple `/usr/sbin/nmap`. Aucune ligne ne doit afficher `ABSENT`. Si `nmap` manque, l’installer par `sudo apt install nmap`, puis reprendre l’étape.

**Étape 1 — Vérifier l’environnement de travail (5min)**

On contrôle d’abord l’adresse du poste et la route par défaut : sans cette information, aucune autre mesure n’est interprétable.

```bash
ip -br addr show
ping -c 1 172.16.0.1
whoami
```

> **Résultat attendu :** `ip -br addr show` affiche une interface `eth0` en état UP avec l’adresse `172.16.0.10/24` ; le `ping` vers la passerelle `172.16.0.1` renvoie 0 % de pertes et un temps de réponse inférieur à 1 ms sur un segment local ; `whoami` affiche le compte à consigner dans le journal d’audit, par exemple `etudiant`, `kali` ou le nom d’ouverture de session. Le nom exact n’a pas d’importance, seule sa reproductibilité compte.
>
> `whoami` et non `echo $USER` : la variable `USER` n’est pas toujours renseignée, notamment dans un conteneur ou une session ouverte par un script, et la commande ne renverrait alors aucune valeur. `whoami` interroge le noyau et répond dans tous les cas.
>
> Un état DOWN ou une adresse absente : corriger la configuration réseau avant de continuer.

**Étape 2 — Collecter les informations de base (14min)**

Les trois commandes de la technique 2 produisent l’inventaire brut. Le symbole `>` crée le fichier, `>>` y ajoute. Conformément à la technique 3, la sortie de `ss` est elle aussi conservée : elle sert de preuve au rapport.

```bash
mkdir -p ~/audit
{ ip -br addr show; ip route show; ip neigh show; } | tee ~/audit/inventaire-brut.txt
ss -tulnp 2>/dev/null | tee -a ~/audit/inventaire-brut.txt || netstat -tulnp 2>/dev/null | tee -a ~/audit/inventaire-brut.txt
```

> **Résultat attendu :** `ip route show` contient une route par défaut via `172.16.0.1` ; `ip neigh show` affiche une entrée pour `172.16.0.1` à l’état `REACHABLE`, ou à l’état `STALE` si cette entrée a déjà servi — les deux états prouvent que l’adresse répond ; seuls `FAILED` et `INCOMPLETE` constitueraient un constat. Une entrée non rafraîchie disparaît au bout de 60 secondes, d’où le `ping` de l’étape 1 qui précède cette lecture.
>
> `ss -tulnp` liste les ports en écoute, et chaque service inattendu est noté comme constat. Sans les droits administrateur, les PID ne sont pas affichés : c’est une limitation à consigner, pas une panne.
>
> Une station de bureau affiche aussi des services internes à la machine — `cups`, `avahi`, le trousseau de clés de la session — mais tous à l’adresse de boucle interne `127.0.0.1`. Ils ne sont ni des services métier ni un risque : on les écarte, et ils ne figurent pas dans l’inventaire. Seules les écoutes sur une adresse du réseau audité entrent dans l’inventaire, et toute autre écoute sur `eth0` est un constat. Sur le poste du laboratoire, l’absence de toute autre écoute est un résultat, pas une panne.

**Étape 3 — Découvrir les machines du segment (16min)**

L’option `-sn` désactive l’analyse de ports et se limite à la **découverte d’hôtes** : c’est la bonne façon de lister les machines présentes sans les sonder en profondeur. L’option `--reason` affiche la cause de l’état observé, ce qui aide à distinguer une machine éteinte d’une machine qui bloque le trafic.

```bash
nmap -sn 172.16.0.0/24 --reason | tee -a ~/audit/inventaire-brut.txt
```

> Remarque : n’exécuter ce type de commande que sur un environnement de laboratoire autorisé, au titre de l’ordre de mission.

> **Résultat attendu :** quatre hôtes sont signalés comme actifs : `172.16.0.1` (routeur), `172.16.0.10` (poste d’audit), `172.16.0.20` (hôte applicatif) et `172.16.0.50` (serveur iperf3). La ligne de chaque hôte se présente sous la forme `Host is up, received <cause> (0.xxx s latency)`, où `<cause>` indique par quel canal la machine a répondu. Quatre valeurs sont possibles selon les droits dont dispose le compte, et il faut savoir les lire :

| Cause affichée | Ce qu’elle prouve |
|----------------|------------------|
| `echo-icmp-reply` | La machine a répondu au message ICMP de test : le trafic de diagnostic est autorisé. Cette sonde exige les droits administrateur ; en compte ordinaire, elle n’est pas émise. |
| `conn-refused` | La machine a renvoyé un refus de connexion : elle est bien à cette adresse, mais le port sondé n’y est pas ouvert. C’est la réponse normale en compte ordinaire. |
| `syn-ack` | La machine a accusé réception de la connexion sans la valider : le port sondé est ouvert, ou un pare-feu a accepté la connexion à la place du service. Une capture permet de départager les deux cas. |
| `arp-response` | La machine a répondu à la résolution d’adresse de la couche liaison. C’est la réponse normale sur un segment local, où `nmap` interroge d’abord ses voisins directs ; elle ne prouve rien sur les services. |

La présence d’un hôte est donc établie même si le message ICMP est filtré. Sur un pare-feu mal configuré, `nmap` peut n’afficher que des `syn-ack` sur des machines qui n’existent pas : c’est un faux positif à consigner comme tel. Tout hôte supplémentaire non annoncé sur la maquette du lab est signalé avant de poursuivre, car il modifie le périmètre.

> Un mot sur la méthode : `nmap` émet par défaut quatre sondes — ICMP echo request, TCP SYN vers le port 443, TCP ACK vers le port 80 et ICMP timestamp request. En compte ordinaire, il se limite à deux connexions TCP, vers les ports 80 et 443, et n’émet aucune sonde ICMP. C’est pourquoi un hôte joignable mais fermé apparaît en `conn-refused` et non en `echo-icmp-reply`. Avec les droits administrateur, la table ci-dessus s’applique entièrement.

#### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| Un outil affiche `ABSENT` à l’étape 0 | Le paquet correspondant n’est pas installé | Installer le paquet puis reprendre l’étape : `sudo apt install nmap` |
| `ping : network is unreachable` | Le poste n’a pas d’adresse sur le réseau audité | Reprendre l’étape 1 du LAB : `ip -br addr show` doit afficher `172.16.0.10/24` |
| `nmap` renvoie zéro hôte actif | Le réseau est filtré, ou l’interface d’observation n’est pas `eth0` | Vérifier `ip -br link show` : la bonne interface est `eth0` |
| `nmap` renvoie un hôte supplémentaire | Un équipement non prévu est présent | Le consigner comme constat et le signaler avant de poursuivre |
| `ip neigh show` affiche des centaines de lignes `FAILED` | La découverte d’hôtes de l’étape précédente a fait résoudre toutes les adresses du segment | Aucune : c’est un effet normal du balayage, la table se vide d’elle-même. Filtrer avec `ip neigh show 172.16.0.1` |
| Le fichier `~/audit/inventaire-brut.txt` est vide | La redirection `>` a écrasé un fichier précédent sans résultat | Reprendre la commande du bloc entier, accolades comprises |

### Ce que ce LAB produit

Le fichier `~/audit/inventaire-brut.txt` contient l’adresse du poste, la route par défaut, la table de voisinage et la liste des hôtes actifs. Le chapitre 2 s’appuie directement sur ce fichier pour compléter l’inventaire du réseau. Conserver ce fichier jusqu’au rapport final.

### Exercices — Chapitre 1

**Exercice 1.1 — Formuler un périmètre**

1. Rédiger le périmètre d’un audit d’un réseau d’entreprise de trois sites, en indiquant ce qui est inclus, ce qui est exclu et la frontière horaire.
2. Lister les trois éléments à formaliser avant le démarrage : autorisation, contraintes, parties prenantes.

**Exercice 1.2 — Ordre des phases**

1. Voici les cinq phases dans un ordre erroné : Rapport, Analyse, Pré-audit, Validation, Collecte. Les remettre dans l’ordre, puis justifier en une phrase par phase en précisant ce qui se passerait si l’on inversait deux phases consécutives.
2. Indiquer les deux preuves à conserver dès la phase de collecte, et dire pourquoi.

---

## Chapitre 2 — M2 : Cartographie et inventaire (10h45–12h15)

### Préambule — cartographier, c’est savoir où sont les portes

Un bâtiment inconnu ne se décrit pas par ses murs : il se décrit par ses portes, ses fenêtres et les chemins qui mènent de l’entrée à chaque pièce. Un réseau se décrit exactement de la même façon. Avant de mesurer quoi que ce soit, il faut savoir **où sont les équipements, comment ils sont reliés, et par où le trafic entre et sort**.

La métaphore utile est celle du plan d’un immeuble avec la notion de zones. On ne dit pas seulement « il y a des bureaux » : on dit « il y a une zone d’accueil, une zone serveurs, une zone de direction, et voici par quels portes on passe de l’une à l’autre ». En réseau, ces zones s’appellent des segments, et les portes s’appellent des routeurs, des commutateurs de niveau 3 ou des pare-feu. Le document qui décrit tout cela est la **topologie**.

Mais un plan seul ne suffit pas. Un bâtiment peut avoir un plan parfait et un compteur d’eau jamais remplacé. De même, un réseau peut être cartographié avec précision et rester ingérable, parce qu’on ignore qui est branché où, avec quel **micrologiciel** — le programme enregistré dans la mémoire non volatil d’un équipement — et quels ports répondent. C’est pourquoi l’audit produit **deux** documents liés : la **topologie**, qui dit comment c’est branché, et l'**inventaire**, qui dit ce que chaque machine est et fait. L’un sans l’autre est incomplet : la topologie sans inventaire ne dit pas où chercher une panne, l’inventaire sans topologie ne dit pas ce qu’un hôte compromis peut atteindre.

Enfin, tous les équipements ne se valent pas. Un routeur qui relie tout le site à tout le monde est plus critique qu’un poste de travail : sa panne arrête tout, sa compromission ouvre tout. Savoir repérer ces **points critiques**, c’est savoir par quoi commencer, et c’est ce qui fera la différence entre un rapport lu et un rapport applicable.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Topologie | Le plan des équipements et de leurs liaisons | C2, logique et physique |
| Topologie logique | Les découpages en segments, VLAN, sous-réseaux et routage | C2, tableau |
| Topologie physique | Les liaisons réelles : câbles, commutateurs, points d’accès | C2, tableau |
| VLAN | Sous-domaine de commutation : des machines qui partagent un réseau, isolées des autres | C2, adressage |
| Segmentation | Découpage du réseau en zones de confiance séparées | C2, inventaire |
| Inventaire | La liste de ce qui existe, avec ce que chaque élément fait | C2, LAB |
| Point critique | Élément dont la panne ou la compromission arrête ou expose l’ensemble | C2, exercices |
| SPOF | Single Point Of Failure, un seul point de défaillance : élément unique dont la panne bloque tout | C2, exercices |
| DMZ | Zone tampon entre le réseau interne et Internet, pour les services exposés | C2, segmentation |
| DHCP | Protocole qui attribue automatiquement une adresse IP et une passerelle à un poste | C2, adressage |
| ACL | Liste de contrôle d’accès : règle appliquée par un équipement pour autoriser ou refuser un trafic selon ses caractéristiques | C2, segmentation |
| WAN | Réseau large, reliant des sites distants, par opposition au LAN, le réseau local d’un site | C2, segmentation |
| MTU | Taille maximale du paquet IP qu’une interface transmet sans fragmenter ; 1 500 octets par défaut sur Ethernet | C2, étape 3 |
| Micrologiciel | Programme enregistré dans la mémoire non volatil d’un équipement réseau, qui le fait fonctionner | C2, préambule |

### Objectifs du chapitre

- Élaborer une cartographie réseau : topologie logique et topologie physique
- Réaliser l’inventaire : adressage, équipements, services, interfaces
- Identifier les points critiques et les dépendances
- Structurer une documentation exploitable

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 22min |
| Techniques, pas à pas | 7min |
| LAB | 38min |
| Exercices | 15min |

### Contenu du chapitre (22min)

#### 1. Topologie et adressage (11min)

La cartographie distingue ce qui est câblé de ce qui est configuré, comme le résume le tableau.

| Élément | Détails | Outils |
|---------|--------|-------|
| Topologie logique | VLAN, sous-réseaux, routage, segmentation | Schémas, tables d’adressage |
| Topologie physique | Liens, commutateurs, points d’accès | Plans, inventaire matériel |
| Adressage | IPv4 et IPv6, plages, réservations, attribution automatique par DHCP | `ip` en standard, `ifconfig` en dernier recours sur d’anciens systèmes, tables |
| Segmentation | DMZ, LAN, WAN, zones de confiance | ACL, politiques |

L’adressage est le socle de tout le reste. Une plage mal choisie produit des conflits d’adresse, et un conflit d’adresse produit une panne qui ressemble à une panne réseau alors qu’elle est administrative. C’est pourquoi l’inventaire commence toujours par les adresses.

#### 2. Services, inventaire et points critiques (11min)

L’inventaire couvre quatre domaines ; la criticité détermine ce qui sera vérifié en priorité. Le tableau distingue ce que le laboratoire permet réellement de collecter de ce qui devra être demandé à l’exploitation.

| Domaine | À inventorier dans ce laboratoire | Hors périmètre du laboratoire | Critères de criticité |
|--------|---------------|---------------------|---------------------|
| Équipements | Rôle et adresse de chaque hôte | Modèles, versions, micrologiciel : à demander à l’exploitation | Redondance, SPOF, exposition |
| Services | Ports ouverts et intitulés rendus par `nmap` | Dépendances applicatives internes | Sensibilité, disponibilité |
| Interfaces | État actif ou inactif et adresse de `eth0` | Vitesse, duplex, erreurs : à relever avec les droits administrateur | Congestion, pertes |
| Documentation | Schéma de topologie (étape 3) | Configurations et MIB (bases d’information de gestion) | À jour, versionnée |

La colonne de droite est la plus importante. Un équipement redondant n’est pas critique : sa panne n’arrête rien. Un équipement unique qui porte trois services est critique même s’il semble secondaire dans l’organigramme. La colonne du milieu est celle qui protège le rapport : y inscrire ce que l’on n’a pas collecté vaut mieux que de l’omettre, car une omission se découvre toujours devant le client.

### Les techniques, pas à pas

**Technique 1 — Lister les hôtes actifs avec `nmap -sn`.** L’option `-sn` désactive l’analyse de ports et se limite à la découverte d’hôtes : c’est rapide et sans risque d’intrusion. Elle n’utilise pas seulement le message ICMP : `nmap` envoie par défaut quatre sondes — ICMP echo request, TCP SYN vers le port 443, TCP ACK vers le port 80 et ICMP timestamp request. En compte ordinaire, il se limite à deux connexions TCP, vers les ports 80 et 443, la sonde ICMP exigeant les droits administrateur. Un hôte qui ne répond pas à l’ICMP mais refuse la connexion TCP est donc quand même détecté. Un pare-feu qui bloque les deux sondes fait disparaître la machine, et c’est là que la table ARP du routeur prend le relais : elle voit les voisins directs, eux qui ont réellement communiqué.

**Technique 2 — Sonder les ports avec `-sT` quand on n’a pas les droits.** `-sS` (sonde furtive) exige les droits administrateur. `-sT` (sonde par connexion) fonctionne en compte ordinaire : c’est plus lent et plus visible, mais c’est la méthode de référence pour un technicien non privilégié. L’option `-T4` fixe un gabarit de temporisation agressif, c’est-à-dire un espacement plus court entre les sondes : le balayage est plus rapide sans changer les résultats. Sans cette option, le gabarit par défaut est `-T3`. Le choix se déclare dans le rapport : la méthode employée fait partie de la preuve.

**Technique 3 — Tracer le chemin avec `traceroute -n`.** L’option `-n` supprime la résolution de noms, qui sinon ralentit chaque étape. Le résultat indique le nombre de sauts et les équipements traversés. Un chemin qui passe par un équipement inattendu est une information, pas un détail : il modifie la surface exposée.

**Technique 4 — Séparer l’inventaire de l’interprétation.** L’inventaire est factuel : « le port 23 est ouvert ». L’interprétation raisonnée : « le port 23 est ouvert sur un hôte de production ». Les deux vont dans le rapport, mais dans deux colonnes distinctes. Mélanger les deux rend le rapport contestable.

### LAB — Cartographier et inventorier le réseau (38min)

#### Prérequis

- Les droits administrateur ne sont pas obligatoires pour ce LAB : toutes les commandes fonctionnent en compte ordinaire.
- Le fichier `~/audit/inventaire-brut.txt`, produit au chapitre 1, doit exister : il fournit l’adresse du poste et la table de voisinage déjà remplies. Le LAB s’en sert comme point de départ, il ne le recommence pas.
- Réseau du lab inchangé : `172.16.0.0/24`.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Client d’audit | 172.16.0.10/24 (configuration relevée au chapitre 1) |
| Réseau audité | 172.16.0.0/24 |
| Fichier repris | `~/audit/inventaire-brut.txt`, produit par le LAB du chapitre 1 |
| Fichier produit | `~/audit/inventaire-complet.md` (tableau d’inventaire et topologie) |
| Fichier de preuve | `~/audit/ports-20.txt`, sortie brute des deux balayages de ports |
| Outils | `nmap`, `ip` de la suite iproute2, `ping`, `traceroute` ou `tracepath`, `grep`, `head`, `nano` |

#### Étapes du LAB

**Étape 1 — Confirmer adressage et voisins (7min)**

On relit le fichier du chapitre 1 avant d’en déduire le livrable suivant : c’est la méthode, pas la commande, qui fait la différence.

```bash
head -n 5 ~/audit/inventaire-brut.txt
grep 'default via 172.16.0.1' ~/audit/inventaire-brut.txt
ip -br link show
ip -br addr show
# Le ping précède la lecture : une entrée de voisinage expire au bout de 60 secondes
ping -c 1 172.16.0.1 >/dev/null && ip neigh show
```

> **Résultat attendu :** l’interface `eth0` est active et l’adresse est `172.16.0.10/24`. Le `ping` précède `ip neigh show` dans le même bloc, et c’est indispensable : une entrée de la table de voisinage expire au bout de 60 secondes faute de trafic, et le noyau n’en conserve aucune. En revanche, le fichier `~/audit/inventaire-brut.txt` ne se périme pas : il conserve l’entrée de voisinage telle qu’elle a été écrite au chapitre 1, au moment du `ping`. C’est précisément pourquoi cette étape relance un `ping` avant d’afficher la table en mémoire. Le fichier se contrôle sur son contenu et non sur son nombre de lignes : les trois commandes du chapitre 1 n’écrivent aucun en-tête, elles écrivent une ligne par interface, par route et par voisin. On cherche donc la route par défaut, qui est la seule preuve d’un adressage correct.
>
> Après la découverte d’hôtes du chapitre 1, la table en mémoire peut afficher plusieurs centaines de lignes à l’état `FAILED` : le balayage a fait résoudre toutes les adresses du segment auprès des voisins. Ces lignes ne sont pas une anomalie, elles se vident d’elles-mêmes. Pour ne lire que la passerelle, filtrer par `ip neigh show 172.16.0.1`.
>
> `head -n 5` affiche les cinq premières lignes du fichier : les adresses du poste, puis les premières routes. Il ne montre pas encore la table de voisinage, placée plus loin, ni la découverte d’hôtes du chapitre 1, qui a été ajoutée à la suite du fichier. C’est normal : le fichier est le cumul de toutes les étapes, pas un rapport classé par thème. On lit donc `head -n 5` pour une vue rapide, puis `grep` pour retrouver une information précise.

**Étape 2 — Inventorier les hôtes et leurs services (16min)**

```bash
# Le chapitre 1 a déjà consigné la découverte : on la relit, on ne la réécrit pas
grep -c 'Host is up' ~/audit/inventaire-brut.txt
nmap -sT -T4 -p 1-1000 172.16.0.20 2>/dev/null | tee ~/audit/ports-20.txt
# Les ports sensibles hors de la plage 1-1000 se sondent par une liste explicite
nmap -sT -T4 -p 443,3389 172.16.0.20 2>/dev/null | tee -a ~/audit/ports-20.txt
```

> **Résultat attendu :** la première commande renvoie `4`, c’est-à-dire les quatre hôtes du laboratoire, dont le poste d’audit `172.16.0.10` ; elle relit la découverte déjà faite au chapitre 1 et ne l’écrit pas une seconde fois, pour éviter un doublon dans le livrable. Les deux commandes ensemble décrivent exactement ce que le laboratoire offre. Sur la plage `1-1000`, `nmap -sT` n’affiche que les ports ouverts, soit 22, 23 et 80, et résume le reste par la ligne `Not shown: 997 closed tcp ports (conn-refused)` : cette ligne est normale, elle indique que les ports fermés répondent par un refus de connexion, ce qui prouve que la machine répond. Un balayage de plage ne montre jamais un port fermé individuellement, et le port 3389 est hors de la plage 1-1000 : c’est pourquoi la seconde commande le demande par une liste explicite, où nmap affiche bien `443/tcp closed https` et `3389/tcp closed ms-wbt-server`. Si un pare-feu filtrait le trafic, on verrait à la place des ports `filtered`, et la conclusion changerait. Le port 23 ouvert est le constat de sécurité que le jour 2 reprend sous l’identifiant C-01 : le noter ici évite une redécouverte le lendemain.

> Le fichier `~/audit/ports-20.txt` est la preuve brute de cette étape : il est écrit par `tee` et conservé avec le livrable `~/audit/inventaire-complet.md`, qui en reprend les conclusions. Avec les droits administrateur, `nmap -sS -T4 -p 1-1000 172.16.0.20` est plus rapide. Le support privilégie `-sT` pour rester exécutable par tout le monde ; le rapport précise la méthode employée.

**Étape 3 — Relever la topologie par le chemin réseau (7min)**

```bash
traceroute -n -q 1 172.16.0.1 2>/dev/null || tracepath -n 172.16.0.1
```

> **Résultat attendu :** `traceroute` et `tracepath` n’affichent pas la même chose, et le support le signale à l’avance pour que la lecture ne soit pas faussée. `traceroute` écrit une ligne d’en-tête, `traceroute to 172.16.0.1 (172.16.0.1), 30 hops max, 60 byte packets`, puis une ligne par saut, soit ici `1  172.16.0.1  0.412 ms`. L’option `-q 1` limite chaque saut à une seule sonde : sans elle, la commande envoie trois sondes et la ligne contient trois temps séparés par des espaces, ce qui donne l’impression d’une sortie fausse alors que c’est le comportement normal.
>
> `tracepath`, retenu en repli quand `traceroute` est absent, envoie deux sondes par saut. Sa sortie comporte une ligne supplémentaire en tête, `1?: [LOCALHOST] pmtu 1500`, qui annonce la MTU mesurée, c’est-à-dire la taille maximale de paquet que le chemin accepte, puis une ou deux lignes de saut selon le nombre de sondes qui reviennent, avec trois différences visibles : un deux-points après le numéro de saut, le mot `reached` et l’unité de temps accolée (`0.412ms` contre `0.412 ms` pour `traceroute`), puis une ligne `Resume: pmtu 1500 hops 1 back 1`. Cette commande affiche donc trois à quatre lignes là où `traceroute` n’en affiche que deux : c’est normal.
>
> Dans les deux cas, un seul saut signifie que la passerelle est directement joignable : `172.16.0.1` est le routeur, sans équipement intermédiaire. Si plusieurs sauts apparaissent, des équipements intermédiaires existent : ils doivent être ajoutés au schéma de topologie avec leur rôle.

**Étape 4 — Consigner l’inventaire (8min)**

Le tableau d’inventaire est le livrable du chapitre. On le rédige à partir des sorties collectées, pas de mémoire. La commande de rédaction précède le modèle : le livrable est un fichier réellement créé, pas un tableau lu dans le support.

```bash
nano ~/audit/inventaire-complet.md
```

> Dans `nano`, la saisie se fait directement dans la fenêtre. Pour enregistrer, on utilise la combinaison `Ctrl + O` — la lettre O, pas le chiffre zéro — puis on valide le nom du fichier avec la touche `Entrée`. Pour quitter, on utilise `Ctrl + X`. Quitter sans avoir enregistré fait perdre le texte : `nano` le signale, mais un débutant qui ferme la fenêtre perd le travail sans avertissement.

Le contenu à saisir est le modèle suivant, complété à partir des sorties des étapes 1 à 3 :

```markdown
# Inventaire — réseau 172.16.0.0/24

## Topologie

| Équipement | Adresse | Rôle | Sauts constatés |
|------------|---------|------|-----------------|
| Routeur    | 172.16.0.1 | Passerelle du segment | 1 saut depuis 172.16.0.10 |

## Tableau d'inventaire

| Adresse | Rôle | Ports observés | Source de la donnée | Criticité |
|---------|------|----------------|--------------------|-----------|
| 172.16.0.1 | Passerelle | n/a | traceroute (chapitre 2) | Critique |
| 172.16.0.10 | Poste d'audit | n/a | ip -br addr show | Sans objet |
| 172.16.0.20 | Hôte applicatif | 22, 23, 80/tcp open ; 443 et 3389 closed | nmap, sortie conservée dans ports-20.txt | Élevée, telnet en clair |
| 172.16.0.50 | Serveur de mesure | non sondé dans ce LAB | à vérifier au chapitre 4 | Moyenne |
```

> **Résultat attendu :** le fichier `~/audit/inventaire-complet.md` existe, et il contient deux sections : la topologie, où figure le routeur avec le nombre de sauts relevé à l’étape 3, et le tableau d’inventaire, avec une ligne par hôte portant l’adresse, le rôle, les ports observés et la commande qui a produit la donnée. La mention `n/a` signifie « non mesuré dans ce LAB » et non « aucun port ». La seconde interface du poste d’audit, `eth1` en 192.168.50.10/24, n’y figure pas : elle appartient au second réseau, audité le jour 2.

#### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `traceroute : command not found` | Le paquet n’est pas installé | Utiliser directement le repli : `tracepath -n 172.16.0.1` |
| `traceroute` n’affiche aucune ligne de saut | Le noyau refuse la socket brute du programme | Le compte ordinaire n’y a pas droit dans certaines configurations : utiliser `tracepath -n 172.16.0.1`, qui donne le même résultat |
| `traceroute` ou `tracepath` affiche `unknown host` | La résolution de nom est en cause | Ajouter l’option `-n`, qui désactive la résolution et travaille en adresse |
| `nmap` affiche `0 hosts up` | Le réseau est filtré, ou l’interface d’observation n’est pas `eth0` | Vérifier `ip -br link show` : la bonne interface est `eth0` |
| `nmap` affiche `Host seems down` alors que la machine existe | Un pare-feu bloque les sondes | Ajouter `-Pn`, qui traite la cible comme active et sonde directement ses ports |
| La commande `grep -c 'Host is up'` renvoie `0` | Le fichier du chapitre 1 a été créé sans la découverte d’hôtes | Reprendre l’étape 3 du chapitre 1, qui ajoute cette sortie au fichier |
| `nano` refuse d’enregistrer | Le fichier appartient à un autre compte, cas d’un poste partagé | Le support presume un compte ordinaire : corriger le propriétaire avec `chown`, ou saisir sous le compte concerné |

### Ce que ce LAB produit

`~/audit/inventaire-complet.md` : le tableau d’inventaire du réseau de laboratoire, avec la source de chaque donnée. Le chapitre 3 s’appuiera sur la ligne `172.16.0.20` pour générer et capturer du trafic, et le chapitre 4 sur la ligne `172.16.0.50` pour les mesures de débit.

### Exercices — Chapitre 2

**Exercice 2.1 — Dessiner une topologie**

1. Proposer une représentation logique d’un réseau à trois segments (sièges, serveurs, administration) sous forme de tableau d’adressage et de description.
2. Justifier les choix de segmentation : pour chaque zone, dire ce qu’une compromission ne doit pas pouvoir atteindre.

**Exercice 2.2 — Repérer les points critiques**

1. Identifier deux SPOF possibles dans le lab et proposer une mitigation pour chacun.
2. Lister les informations minimales pour qu’un inventaire soit traçable six mois plus tard.

---

## Chapitre 3 — M3 : Analyse du trafic (13h45–15h15)

### Préambule — le réseau ne parle pas, il montre des traces

Tout ce que fait un réseau, on ne le voit pas directement : on en voit les **traces**. Quand un utilisateur ouvre une page web, une suite de petits messages circule entre sa machine et le serveur, chacun avec une adresse d’origine, une adresse de destination, un port, et un numéro d’ordre. Cette suite s’appelle une **conversation réseau**. Capturer le trafic, c’est photographier ces conversations.

L’image juste est celle d’une **boîte noire** dans un avion. La boîte n’empêche rien, ne dirige rien : elle enregistre. C’est exactement le rôle d’une capture passive. On place l’enregistreur à un endroit du trajet (un port miroir sur un commutateur, un TAP, ou l’interface du poste de test) et on laisse les messages se passer. Rien n’est modifié, rien n’est bloqué. C’est ce qui distingue la capture passive de toute autre technique : elle est indétectable pour les machines observées et sans impact sur le service.

Mais une photo de trois heures n’apprend rien. Une capture brute contient du bruit : tout le trafic du réseau, y compris les conversations qui n’ont aucun rapport avec le problème étudié. Il faut donc **filtrer**, c’est-à-dire ne garder que ce qui intéresse. Le langage de filtrage le plus répandu s’appelle le BPF (Berkeley Packet Filter) : c’est une grammaire de sélection qui dit, en une ligne, « garde-moi les paquets qui vont de cette machine vers ce port ». Ce chapitre la présente et la met en pratique.

Une fois filtrée, la capture devient une **preuve**. Une preuve d’audit a trois qualités : elle est horodatée, elle est reproductible, et elle dit ce qu’elle ne dit pas. La dernière qualité est la plus oubliée. Une capture ne prouve pas qu’un pare-feu est mal configuré ; elle prouve qu’un paquet a circulé. Le rapprochement entre la preuve et la politique de sécurité, lui, vient de l’analyse. C’est pourquoi ce chapitre apprend d’abord à lire, ensuite seulement à conclure.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Capture passive | Copie du trafic à un point d’observation, sans le modifier | C3, technique 1 |
| Port miroir (SPAN) | Fonction de commutateur qui recopie le trafic d’un port vers un port d’observation | C3, tableau |
| TAP | Équipement qui duplique le trafic à un niveau physique, sans erreur possible | C3, tableau |
| pcap | Format de fichier qui enregistre les paquets capturés, avec leur horodatage | C3, environnement |
| BPF | Grammaire de filtre de tcpdump pour ne garder que certains paquets | C3, technique 2 |
| Filtre de capture | Sélection appliquée au moment de l’enregistrement, par tcpdump ; syntaxe à espaces, par exemple `tcp port 80` | C3, technique 2 |
| Filtre d’affichage | Sélection appliquée à la lecture, par Wireshark et tshark ; syntaxe à points, par exemple `tcp.port == 80` | C3, étape 4 |
| Opérateurs BPF | `src` et `dst` pour le sens, `net` pour un réseau, `and`, `or` et `not` pour combiner, `port` pour un port | C3, tableau et exercices |
| ICMP | Protocole de diagnostic réseau, employé par `ping` pour mesurer la latence et par `nmap` pour détecter un hôte | C3, étapes 1 et 2 |
| ARP | Protocole de résolution d’adresse : il traduit une adresse IP en adresse de carte réseau pour poser une trame ; il ne porte pas d’adresse IP et n’est donc pas retenu par le filtre `ip` | C3, étape 1 |
| Trame (frame) | Un paquet vu au niveau liaison de données, avec son adresse MAC | C3, lecture |
| Drapeau TCP | Lettre indiquant l’état d’une connexion : S pour SYN, . pour ACK, R pour reset | C3, étape 2 |
| 5-tuple | Les cinq éléments qui identifient une conversation : IP source, IP destination, port source, port destination, protocole | C3, tableau |
| Retransmission | Paquet renvoyé parce que l’accusé de réception n’est pas revenu : signe d’une perte | C3, exercices |

### Objectifs du chapitre

- Comprendre les principes de capture et d’analyse de flux
- Appliquer des filtres pertinents pour isoler les flux
- Interpréter les captures pour détecter des anomalies
- Détecter des comportements suspects tout en respectant l’autorisation

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 22min |
| Techniques, pas à pas | 7min |
| LAB | 38min |
| Exercices | 15min |

### Contenu du chapitre (22min)

#### 1. Captures, formats et méthodologie (11min)

Une capture bien préparée garantit des preuves exploitables : voici les notions clés.

| Notion | Détails | Outils |
|-------|--------|-------|
| Capture passive | Copie du trafic (port miroir, TAP) | tcpdump, Wireshark |
| Formats | PCAP, PCAPNG | Stockage, indexation |
| Positionnement | TAP, port miroir | Impact, visibilité |
| Bonnes pratiques | Minimiser, horodater, chiffrer, conserver les preuves | Traçabilité |

La règle qui gouverne toutes les bonnes pratiques est la minimisation : on ne capture que le nécessaire, pendant le temps nécessaire. Une capture de trois heures contient des données personnelles qui n’ont rien à voir avec l’audit. Lesquelles sont dans un rapport ?

#### 2. Filtres, analyse et détection d’anomalies (11min)

Les filtres isolent le trafic, l’analyse le qualifie, et les anomalies s’y révèlent.

| Technique | Exemple | Objectif |
|-----------|---------|----------|
| BPF | `host 172.16.0.10 and port 80` | Filtrage efficace |
| Flux | Conversations 5-tuple : IP source, IP destination, port source, port destination, protocole | Agrégation |
| Indicateurs | Débits, retransmissions, RTT (temps aller-retour), erreurs | Performance |
| Anomalies | Sondages, diffusion excessive, pertes, latences | Détection |

Le 5-tuple est le regroupement de référence en analyse de flux : cinq éléments suffisent à identifier une conversation de façon unique. L’outil `tshark` produit cette table sans configuration, par l’option `-z conv,tcp` appliquée à un fichier de capture. Deux syntaxes de filtre coexistent et ne se mélangent pas : le filtre de capture de `tcpdump` s’écrit avec des espaces (`tcp port 80`), le filtre d’affichage de `tshark` et de Wireshark s’écrit avec des points (`tcp.port == 80`). Les inverser échoue dans les deux cas sur une erreur de syntaxe.

### Les techniques, pas à pas

**Technique 1 — Capturer sans interrompre.** La capture passive observe une copie du trafic. Sur le poste de laboratoire, l’interface `eth0` sert de point d’observation : on lit ce qui y circule. Les droits administrateur sont nécessaires pour ouvrir le descripteur de capture ; sur un poste partagé, on demande les droits à l’administrateur ou on travaille dans une machine de laboratoire dédiée.

**Technique 2 — Filtrer au moment de la capture.** Le filtre BPF s’applique à l’écriture du fichier, pas seulement à la lecture. Filtrer à la capture réduit le fichier, préserve les performances du poste et évite de stocker des données inutiles. C’est la bonne pratique, sauf quand on ne sait pas encore ce que l’on cherche.

**Technique 3 — Lire une ligne de tcpdump de gauche à droite.** Une ligne de sortie se décompose en cinq parties : l’horodatage (quantième de la journée), l’adresse source, le port source, l’adresse et le port du destinataire, puis le protocole et le type de message. Savoir lire ces cinq éléments permet de répondre à la seule question qui compte : qui parle à qui, sur quel port, et dans quel ordre.

**Technique 4 — Recouper le trafic avec les indicateurs.** Une capture seule dit ce qui a circulé ; elle ne dit pas si c’est normal. Pour qualifier, on recoupe avec les mesures de performance du chapitre 4 : une latence élevée dans la capture et dans le `ping` font deux preuves du même phénomène, pas une. Le recoupement se fait à la technique 5 et à l’étape 3 du chapitre 4, qui relisent cette même capture avec l’option `-v` de `tcpdump`.

### LAB — Analyser le trafic avec un outil de capture (38min)

#### Prérequis

- Droits administrateur disponibles sur le poste. La commande `sudo -v` demande le mot de passe une seule fois, puis valide le cache de droits : les commandes en libre accès qui suivent s’exécutent sans nouvelle invite pendant les cinq minutes qui suivent, durée par défaut de ce cache. Au-delà, relancer `sudo -v`.
- `tcpdump` installé (présent par défaut sur les distributions de la famille Debian). `wireshark` et `tshark` utilisés en lecture, sans droits.
- Le répertoire `~/audit/` doit exister : il a été créé à l’étape 2 du chapitre 1. S’il a disparu, le recréer par `mkdir -p ~/audit` avant la capture, sinon `tcpdump` s’interrompt sur un chemin inexistant, sans message explicite.
- Le fichier `~/audit/inventaire-complet.md`, produit au chapitre 2, fournit l’adresse de la cible : `172.16.0.20`, port 80. On capture du trafic vers cette cible.
- Outils de génération de trafic : `ping` et `curl`.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Interface d’observation | `eth0` (adaptateur du poste d’audit, 172.16.0.10/24) |
| Cible du trafic | 172.16.0.20, port 80 (donnée issue de l’inventaire du chapitre 2) |
| Trafic généré | ICMP vers la passerelle, HTTP vers l’hôte applicatif |
| Outils | `tcpdump` (capture, droits administrateur requis), `wireshark` (lecture), `tshark` (repli en ligne de commande) |
| Fichier repris | `~/audit/inventaire-complet.md`, produit par le LAB du chapitre 2 |
| Fichier capture | `~/audit/audit_capture.pcap` |

#### Étapes du LAB

**Étape 1 — Générer et capturer un trafic ciblé (8min)**

Le filtre BPF `host 172.16.0.10` ne garde que les paquets dont l’une des extrémités est le poste d’audit : le fichier ne contient donc que le trafic que nous provoquons, ce qui rend la lecture évidente. La commande `timeout 20` limite la durée de la capture à vingt secondes et l’arrête proprement, sans manipulation de processus.
>
> Ce filtre laisse passer l’ARP, qui ne porte pas d’adresse IP mais une adresse de carte réseau : la capture contient donc aussi les demandes de résolution d’adresse échangées avant l’ouverture des sessions. C’est le comportement attendu du filtre `host`, et ces trames se lisent sans difficulté, avec un libellé `ARP` en tête de ligne. Pour ne conserver que le trafic IP, on écrit `ip and host 172.16.0.10` ; le fichier est alors plus petit, au prix de la trace de la résolution d’adresse.

```bash
# Renouveler les droits sudo pour les cinq minutes qui suivent
sudo -v
# Capture limitée à 20 s, arrêt automatique ; le résumé de tcpdump est sur stderr,
# on ne masque donc pas stderr ici
sudo timeout 20 tcpdump -i eth0 host 172.16.0.10 -w ~/audit/audit_capture.pcap &
sleep 3
# Générer du trafic de test pendant la capture
ping -c 3 172.16.0.1 >/dev/null
curl -s --connect-timeout 2 http://172.16.0.20/ >/dev/null || true
wait
ls -lh ~/audit/audit_capture.pcap
```

> **Résultat attendu :** tcpdump affiche un résumé du type « X packets captured », avec X supérieur ou égal à 3 ; le fichier `~/audit/audit_capture.pcap` pèse quelques kilo-octets. Zéro paquet capturé signifie l’une des trois choses suivantes : mauvaise interface, trafic généré après la fin de la capture, ou droits administrateur manquants. Dans ce dernier cas, la commande `sudo -v` échoue et le fichier n’est pas créé.
>
> Le fichier appartient à l’utilisateur `tcpdump` et non au compte de l’étudiant, parce que la commande a été exécutée avec `sudo`. C’est normal : `ls -lh` affiche donc `tcpdump tcpdump`. Le fichier reste lisible et le rapport peut le joindre tel quel. En revanche, il ne peut pas être écrasé par une nouvelle capture lancée sans `sudo`, qui échouerait sur un fichier appartenant à un autre compte. Le supprimer puis le recréer ne demande pas de droits supplémentaires tant que le fichier se trouve dans le répertoire personnel de l’étudiant, dont il est propriétaire.

**Étape 2 — Lire et filtrer la capture (12min)**

On relit le fichier trois fois : en entier, puis sur ICMP, puis sur le port 80. Chaque filtre répond à une question différente.

```bash
tcpdump -r ~/audit/audit_capture.pcap -n
tcpdump -r ~/audit/audit_capture.pcap -n icmp
tcpdump -r ~/audit/audit_capture.pcap -n 'tcp port 80'
# Regrouper la capture par conversation, une ligne par 5-tuple
tshark -r ~/audit/audit_capture.pcap -q -z conv,tcp
```

> Exemple de sortie à interpréter :
>
> ```text
> 10:15:01.100 IP 172.16.0.10 > 172.16.0.1: ICMP echo request, id 1, seq 1, length 64
> 10:15:01.101 IP 172.16.0.1 > 172.16.0.10: ICMP echo reply, id 1, seq 1, length 64
> 10:15:03.210 IP 172.16.0.10.54321 > 172.16.0.20.80: Flags [S], seq 18243, length 0
> ```
>
> Lecture de gauche à droite : horodatage (10 h 15 min 01 s 100 ms), adresse et port d’origine, adresse et port de destination, protocole, type de message. Les drapeaux TCP indiquent l’état de la connexion : `S` pour l’ouverture (SYN), `.` pour l’accusé de réception (ACK), `R` pour une fermeture brutale (reset).

> **Résultat attendu :** le filtre `icmp` renvoie six lignes, soit trois demandes et trois réponses avec le même identifiant. Le filtre `tcp port 80` renvoie la session complète, de dix à douze lignes selon la façon dont le noyau regroupe les accusés de réception : `S` pour l’ouverture, `S.` pour l’accusé du serveur, `GET` pour la requête, `200 OK` pour la réponse, puis les accusés de réception et les deux fermetures de session. Les horodatages sont croissants. Une seule ligne par message ICMP signifie que les réponses ne sont pas capturées : le filtre est trop restrictif. La dernière commande affiche une ligne par conversation TCP, avec les cinq éléments du 5-tuple et le nombre de trames de chacune ; ici une seule conversation, entre le poste d’audit et le port 80 de l’hôte applicatif.

**Étape 3 — Compter les paquets capturés (5min)**

Compter les paquets dans un seul sens permet de vérifier que la capture contient bien les échanges que nous avons provoqués, et d’estimer le volume réel du trafic.

```bash
tcpdump -r ~/audit/audit_capture.pcap -n | wc -l
tcpdump -r ~/audit/audit_capture.pcap -n 'ip src 172.16.0.10' | wc -l
tcpdump -r ~/audit/audit_capture.pcap -n 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0' | wc -l
```

> **Résultat attendu :** le premier `wc -l` donne le nombre total de paquets de la capture, le deuxième donne le nombre de paquets **émis** par `172.16.0.10`, le troisième le nombre d’ouvertures de connexion TCP. Le troisième filtre se lit ainsi : le drapeau SYN est présent et le drapeau ACK est absent, ce qui définit une ouverture de connexion. La forme longue `tcp[tcpflags] & (tcp-syn) != 0` est la seule acceptée par `tcpdump`. La forme abrégée `tcp.flags.syn == 1` appartient à la syntaxe des filtres d’affichage de Wireshark et de `tshark` : `tcpdump` la refuse dans toutes les versions et répond `syntax error`, exactement comme il refuserait `tcp.port == 80`. L’option `src` du deuxième filtre signifie source : il ne retient que les paquets émis par `172.16.0.10`.
>
> Sur une capture où chaque requête reçoit sa réponse, le deuxième chiffre représente un peu plus du tiers du total, et non la moitié : ici 9 paquets émis sur 26, parce que les huit trames de résolution d’adresse écrites avant l’ouverture des sessions ne portent pas d’adresse IP et ne sont donc pas retenues par le filtre `ip src 172.16.0.10`, à cause du mot-clé `ip`. Rapporté au seul trafic IP, le compte est bien de moitié. Un second chiffre proche du total signifierait que la capture ne contient qu’un seul sens, ce qui arrive quand le filtre BPF de l’étape 1 a été mal écrit. Le troisième chiffre doit être très faible, de l’ordre du nombre de connexions ouvertes : ici une seule, vers le port 80. Un nombre élevé de paquets SYN sans ACK associé signale une activité inattendue sur le segment : connexions refusées en boucle, balayage de ports, ou émulation de trafic légitime.

**Étape 4 — Analyser graphiquement avec Wireshark (13min)**

Ouvrir le fichier dans Wireshark ne demande aucun droit : c’est de la lecture.

```bash
# La présence de Wireshark est vérifiée avant de l'appeler : sans ce test,
# une commande absente ne dit rien et l'étudiant reste devant un écran vide
command -v wireshark >/dev/null && wireshark ~/audit/audit_capture.pcap &
```

Dans l’interface, appliquer le filtre d’affichage, puis observer les flux :

```text
icmp
tcp.port == 80
```

Si Wireshark est présent, une fenêtre s’ouvre sur la capture. Si la commande ne produit rien et ne signale aucune erreur, c’est qu’il est absent. En session sans affichage graphique, Wireshark signale au contraire une erreur de connexion au serveur d’affichage : c’est le même cas du point de vue de l’analyse, et le repli `tshark` ci-dessous prend le relais sans perte, car il rend exactement les mêmes trames.

```bash
tshark -r ~/audit/audit_capture.pcap -Y "icmp"
tshark -r ~/audit/audit_capture.pcap -Y "tcp.port == 80"
```

> `-Y` désigne le filtre d’affichage, qui utilise la même syntaxe que dans Wireshark ; `-r` désigne la lecture d’un fichier.

> **Résultat attendu :** le filtre `icmp` montre six trames, en paires demande-réponse ; le filtre `tcp.port == 80` montre la même conversation qu’à l’étape 2, de dix à douze trames selon la taille de la page renvoyée : ouverture, accusé de réception, requête `GET`, envoi de la réponse, puis les accusés de réception et les deux fermetures de session. Le tableau de `tshark` comporte sept colonnes — numéro, heure, source, destination, protocole, longueur et description. En ligne de commande, aucun en-tête n’est imprimé au-dessus du tableau : la colonne utile, celle qui décrit chaque trame, est la dernière. En session sans affichage graphique, le repli `tshark` donne les mêmes résultats que l’interface de Wireshark.

#### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `tcpdump` affiche `You don't have permission to perform this capture` | La capture a été lancée sans droits administrateur | Relancer la commande avec `sudo` devant `tcpdump` |
| `tcpdump` affiche `0 packets captured` | Le trafic a été produit après l’arrêt de la capture, ou sur une autre interface | Vérifier que `-i eth0` est bien présent et que le `ping` est dans le délai de la capture |
| `tcpdump` s’interrompt sur `Path does not exist` | Le répertoire `~/audit/` a disparu, par exemple après un redémarrage | Le recréer : `mkdir -p ~/audit`, puis reprendre la capture |
| La capture ne contient aucun paquet ICMP | Le filtre `host` ne laisse passer que l’ARP, pas l’ICMP | Utiliser `ip and host 172.16.0.10` pour ne conserver que le trafic IP |
| La nouvelle capture échoue sur `Permission denied` | Le fichier précédent appartient à `tcpdump`, qui l’a créé avec ses propres droits | Le supprimer d’abord : `rm -f ~/audit/audit_capture.pcap`, puis relancer la capture |
| `tshark` répond `Unknown field` ou `syntax error` | Un filtre de capture a été écrit dans la syntaxe d’affichage | Écrire `tcp port 80` pour `tcpdump`, `tcp.port == 80` pour `tshark` : les deux syntaxes ne se mélangent pas |
| `wireshark` s’interrompt sur `could not connect to display` | La session ne dispose d’aucun affichage graphique, cas d’un poste sans bureau ou d’un accès à distance | Utiliser le repli `tshark`, qui rend les mêmes trames en ligne de commande |

### Ce que ce LAB produit

Le fichier `~/audit/audit_capture.pcap` : une preuve de trafic horodatée, filtrée sur le poste d’audit, contenant les conversations ICMP et HTTP provoquées, auxquelles s’ajoutent les trames de résolution d’adresse ARP échangées au préalable. Le chapitre 4 s’en sert pour vérifier le plan de trafic avant de mesurer les performances : un débit mesuré sur un lien dont on ne connaît pas le contenu n’est pas interprétable. Le rapport final s’y réfère également pour prouver l’existence des flux. Le fichier est écrit dans `~/audit/` et non dans `/tmp/`, qui est vidé à chaque redémarrage : c’est le livrable le plus volumineux de la journée, et à l’égal de `inventaire-brut.txt`, `inventaire-complet.md` et `mesures-performance.txt`, il doit survivre à la nuit du 8 au 9 octobre, le jour 2 le reprenant en annexe.

### Exercices — Chapitre 3

**Exercice 3.1 — Écrire des filtres**

1. Écrire un filtre BPF qui isole le trafic DNS, sur les ports 53 en UDP et en TCP.
2. Écrire un filtre qui conserve tout le trafic sauf celui d’une machine donnée, et expliquer en une phrase pourquoi ce filtre est plus coûteux qu’un filtre restrictif.

**Exercice 3.2 — Détecter des anomalies**

1. Lister trois indicateurs observables dans une capture : retransmissions, RTT, débits.
2. Proposer une interprétation pour chacun, en précisant ce que l’indicateur ne prouve pas à lui seul.

---

## Chapitre 4 — M4 : Mesure de performance (15h30–17h00)

### Préambule — mesurer avant de se plaindre

« Le réseau est lent » est la phrase la plus fréquente et la moins exploitable du métier. Elle ne dit ni où, ni quand, ni combien. Un auditeur qui la reçoit doit d’abord la traduire en une question mesurable : **combien de temps pour aller-retour, quel débit obtenu, combien de paquets perdus, quelle variation de latence**. C’est tout l’objet de ce chapitre.

L’image juste est celle du **chronomètre et du compteur de voiture**. Sur une route, on ne juge pas la circulation à l’impression générale. On mesure le temps entre deux points, la distance parcourue par heure, et surtout la régularité : deux voitures qui passent à une seconde d’intervalle régulier, et deux voitures qui passent par paquets de cinq puis par groupes de trois, ne décrivent pas la même route, même si la moyenne est identique. Cette régularité, c’est ce que l’on appelle le **jitter**. Il ne se voit pas dans une moyenne, et c’est pourtant lui qui gêne le plus l’utilisateur : une connexion à 100 Mbit/s qui saccade est plus désagréable qu’une connexion stable à 20 Mbit/s.

Le deuxième point est la différence entre le **débit** et la **latence**. Le débit, c’est la taille du tuyau. La latence, c’est la longueur du chemin. On peut avoir un tuyau énorme et un très long chemin : les fichiers se téléchargent très vite, mais chaque clic attend une seconde. Ce sont deux problèmes différents, qui appellent deux corrections différentes. Un rapport d’audit qui les confond conduit à des recommandations inutiles.

Le troisième point est la **qualité de service**, ou QoS. Sur un lien partagé entre la voix, la visioconférence et le transfert d’un fichier, tout le monde ne se valent pas. Un appel téléphonique devient saccadé dès qu’il perd 1 % de ses paquets, alors qu’un transfert de fichier ne s’en aperçoit pas. Le QoS consiste à marquer les paquets selon leur importance et à leur réserver un traitement différent. La question d’audit n’est pas « le QoS est-il activé » mais « le marquage est-il cohérent avec la politique écrite, et les files sont-elles dimensionnées pour le trafic réel ».

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Latence | Temps d’aller-retour d’un petit message | C4, indicateur 1 |
| RTT | Round Trip Time, le temps d’aller-retour, mesuré en millisecondes | C4, indicateur 1 |
| Bande passante | Débit maximal supporté par un lien | C4, indicateur 2 |
| Pertes | Proportion de paquets qui ne parviennent jamais à destination | C4, indicateur 3 |
| Jitter | Variation de la latence d’un paquet à l’autre | C4, indicateur 4 |
| Goulot d’étranglement | Limite de débit d’un chemin, par exemple un lien de 100 Mbit/s sur un segment annoncé à 1 Gbit/s | C4, étape 2 |
| QoS | Qualité de service : traiter différemment les flux selon leur importance | C4, indicateur 5 |
| DSCP | Valeur portée dans les six bits de poids fort de l’octet de priorité d’un en-tête IP, qui classe le paquet par priorité | C4, technique 5 |
| ECN | Notification de congestion : deux bits de poids faible du même octet, qui signalent un engorgement sans jeter le paquet | C4, technique 5 |
| Percentile | Seuil au-delà duquel ne se situe qu’un pourcentage donné des mesures ; le p95 est le seuil au-delà duquel ne se situent que 5 % des mesures, et non 95 % | C4, étape 1 |
| SLA | Accord de niveau de service : engagement contractuel de disponibilité et de performance, base des seuils opposables | C4, hors du laboratoire |
| Duplex | Mode d’échange d’une liaison Ethernet : le full-duplex émet et reçoit en même temps, le half-duplex alterne ; un appariement incorrect entre les deux extrémités fait chuter le débit et augmenter les pertes | C4, technique 2 |
| MSS | Maximum Segment Size, la taille maximale de la charge utile d’un segment TCP : 1 460 octets sur un lien de MTU 1 500, 1 448 lorsque les horodatages TCP sont actifs | C4, étape 3 |
| Somme de contrôle | Valeur calculée à l’émission pour détecter la corruption ; certains systèmes la calculent à la sortie de la carte, d’où les messages d’erreur de `tcpdump` | C4, étape 3 |
| Constat | Une affirmation factuelle sur le réseau, accompagnée de sa preuve | C4, livrable |
| Preuve | La commande exécutée et son extrait de sortie | C4, livrable |
| Criticité | La gravité d’un constat, graduée de Faible à Critique | C4, livrable |

### Objectifs du chapitre

- Mesurer latence, bande passante, pertes et jitter
- Analyser et interpréter la qualité de service (QoS)
- Identifier les goulots d’étranglement et leur impact
- Proposer des indicateurs mesurables et reproductibles

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 19min |
| Techniques, pas à pas | 9min |
| LAB | 39min |
| Exercices | 15min |

### Contenu du chapitre (19min)

#### 1. Les cinq indicateurs de performance (9min)

Les cinq indicateurs mesurés en audit s’appuient sur les méthodes et les seuils du tableau suivant. La colonne « seuil retenu au laboratoire » donne la valeur à partir de laquelle un constat est posé sur le réseau du laboratoire, et non une norme universelle.

| Indicateur | Définition | Méthode de mesure | Seuil retenu au laboratoire | Outil |
|-----------|-----------|-------------------|-----------------------------|-------|
| Latence | Temps aller-retour (RTT) | ICMP, mesuré par `ping` | RTT moyen supérieur à 1 ms sur un segment local | `ping` |
| Bande passante | Débit utile | Transfert de données, débit soutenu | Moins de 80 % de la vitesse nominale de l’interface | `iperf3` |
| Pertes | Proportion de paquets perdus | Série ICMP, statistiques de transport | Plus de 0 % sur un segment local | `ping`, `iperf3` |
| Jitter | Variation de latence d’un paquet à l’autre | Paquets en temps réel | 30 ms ou plus sur un flux voix ou visioconférence, repère d’usage à sourcer | `iperf3` en UDP |
| QoS | Priorisation, engagement de service | Marquage DSCP, classes, files d’attente | Aucun paquet marqué DSCP dans la capture du poste d’audit | `tcpdump`, `tc` |

La mesure du RTT d’établissement de session, celui de l’accusé de réception TCP, nécessiterait un outil dédié comme `hping3` ou `tcping`, qui ne fait pas partie du laboratoire : le support mesure donc la latence par ICMP, et le précise dans le rapport.

Les seuils sont des repères, pas des normes, et chacun doit être tracé. Les seuils de latence, de pertes et de débit sont justifiés physiquement dans ce chapitre. Les seuils de 1 % de pertes et de 30 ms de jitter sont des repères d’usage de la voix sur IP, que le support n’appuie sur aucune norme : un rapport d’audit les mentionne comme tels et renvoie à la politique écrite du client pour les sourcer. Un rapport cite toujours le seuil appliqué, le trajet de référence et la source du seuil.

Le seuil se juge en outre par rapport au trajet de référence : 4 ms est un excellent résultat entre deux villes distantes de 400 km, car la lumière parcourt 800 km aller-retour dans la fibre à environ 200 000 km/s, soit déjà 4 ms, c’est-à-dire une milliseconde par tranche de 200 km de trajet simple. Le même 4 ms serait un résultat médiocre sur un segment local, où l’on attend moins de 1 ms.

#### 2. Analyse de la qualité de service (10min)

L’analyse de la qualité de service (QoS, ensemble des mécanismes qui font traiter les flux selon leur priorité) vérifie la cohérence entre le marquage, les files d’attente et les constats mesurés. Elle se conduit sur des commandes, détaillées à la technique 5 : sans lecture du marquage réel des paquets, la qualité de service reste une affirmation.

Trois lectures sont nécessaires, et chacune répond à une question distincte. Quelle file d’attente traite les paquets, et combien en a-t-elle jeté ? Comment les paquets capturés sont-ils marqués en priorité ? Une politique de classes est-elle configurée sur l’interface ? Un rapport qui répond à ces trois questions par des commandes n’affirme rien ; un rapport qui n’en répond pas par des commandes ne prouve rien.

### Les techniques, pas à pas

**Technique 1 — Mesurer la latence et les pertes avec `ping`.** `ping` envoie un petit message ICMP et mesure le temps d’aller-retour. `-c` fixe le nombre d’envois, ce qui rend la mesure reproductible : `-c 10` donne un échantillon suffisant pour un segment local. La sortie fournit trois informations : le nombre de paquets perdus, le RTT minimal, moyen et maximal, et le mdev. Le mdev est l’écart type des temps de réponse : c’est notre indicateur de gêne temporelle sur un flux de contrôle, distinct du jitter d’un flux temps réel mesuré à l’étape 3.

**Technique 2 — Mesurer le débit soutenu avec `iperf3`.** `-t 10` fixe dix secondes de transfert, `-P 1` limite la mesure à un seul flux : un agrégat de plusieurs flux additionnerait leurs limites et masquerait la limite du flux mesuré. Le débit obtenu se compare ensuite à la vitesse nominale de l’interface : un écart important trahit un goulot, souvent un duplex mal apparié ou un équipement sous dimensionné. La colonne `Retr` compte les retransmissions : une valeur élevée signale une instabilité, même si le débit paraît correct.

**Technique 3 — Mesurer le jitter et les pertes en UDP.** `-u` sélectionne UDP, `-b 1M` fixe le débit à 1 Mbit/s, valeur faible et volontaire : elle permet de mesurer la qualité du lien sans le saturer. `-l 1460` fixe la taille du datagramme, ce qui rend le nombre total de datagrammes reproductible d’une mesure à l’autre. Un flux temps réel perturbé se voit ici immédiatement, en jitter et en datagrammes perdus, là où une mesure TCP le masquerait par ses retransmissions.

**Technique 4 — Ne jamais mesurer en plein jour sans référence.** Une mesure de performance prise à 10 h un mardi ne signifie rien sans la valeur du même lien à 3 h un dimanche. Un rapport qui ne donne pas le moment de la mesure est un rapport qui sera contesté.

**Technique 5 — Lire le marquage et les files d’attente.** L’outil `tc`, qui appartient à la suite iproute2 au même titre que `ip`, lit les files d’attente et les classes configurées sur une interface. Installation si l’outil manque : `sudo apt install iproute2`. Le filtrage des paquets capturés se fait, lui, avec `tcpdump` et son option `-v`. Ces deux lectures se font sur des données déjà collectées : la capture du chapitre 3 et l’interface du poste. Rien n’est généré, rien n’est mesuré, c’est de la lecture.

```bash
# Lire la file d'attente active de l'interface et ses compteurs
tc -s qdisc show dev eth0
# Lire le marquage de priorité des paquets capturés
tcpdump -r ~/audit/audit_capture.pcap -n -v 'ip and greater 8'
# Lister les classes configurées ; une sortie vide ne demande aucun droit particulier
tc class show dev eth0
```

> **Résultat attendu :** `tc -s qdisc show dev eth0` décrit la file d’attente par défaut. Sur une interface simple, elle se nomme `noqueue`, c’est-à-dire aucune file : le trafic passe immédiatement. Sur un lien plus complexe, `fq_codel` et `pfifo_fast` sont deux algorithmes d’attente, le premier avec partage équitable du lien, le second premier arrivé premier servi. La commande affiche ensuite les compteurs de la file.
>
> Un compteur `dropped` qui croît pendant un transfert `iperf3` signale une file saturée : c’est un constat de goulot. Le compteur `backlog`, lui, donne l’occupation de la file à cet instant, en octets et en paquets : il décrit l’occupation, pas le temps d’attente de chaque paquet, et ne prouve donc rien à lui seul. Sur l’interface du laboratoire, ces compteurs restent à zéro pendant un transfert, la file virtuelle du conteneur n’étant pas instrumentée : c’est une limite de l’outillage du poste, qui se consigne comme telle.
>
> La commande `tc class show dev eth0` renvoie une sortie vide : c’est normal et non un défaut de droits, car aucune classe de priorité n’est configurée sur cette interface. Une liste vide signifie « aucune politique de qualité de service en place », ce qui constitue en soi le constat à porter au rapport.
>
> `tcpdump -v` affiche le champ sous la forme `tos 0x0`. Attention à la lecture : cet octet n’est pas le DSCP. Il réunit le DSCP dans ses six bits de poids fort et les deux bits de notification de congestion dans ses deux bits de poids faible. Le DSCP s’obtient donc en divisant la valeur affichée par quatre, arrondie à l’unité supérieure. Le DSCP 46, classe EF réservée aux flux voix, apparaît comme `tos 0xb8`, car 46 multiplié par quatre vaut 184. Le DSCP 40, classe CS5, apparaît comme `tos 0xa0`, car 40 multiplié par quatre vaut 160. Le DSCP 0, classe par défaut, apparaît comme `tos 0x0`. C’est le découpage défini par la RFC 2474. Un flux voix marqué `tos 0x0` alors que la politique le déclare en EF est un écart de qualité de service à consigner.
>
> Le critère `greater 8` du filtre n’écarte que d’éventuels paquets tronqués de moins de huit octets ; sur une capture complète il ne supprime rien, et l’on pourrait écrire simplement `ip`. Il est conservé ici parce qu’il figure dans la plupart des manuels, et pour que l’étudiant sache qu’un critère de ce type n’a pas d’effet sur une capture saine.

### LAB — Mesurer les performances réseau (39min)

#### Prérequis

- `iperf3` installé sur **les deux** postes : le client `172.16.0.10` et le serveur `172.16.0.50`. Installation si l’outil manque : `sudo apt install iperf3` sur chaque poste.
- Le service de mesure doit écouter sur le serveur. En salle, on le démarre dans un second terminal, sur le serveur : `iperf3 -s`. Au laboratoire, le service est déjà démarré sur `172.16.0.50` : on le vérifie depuis le poste d’audit, sans avoir accès au serveur, avec la commande de l’étape 1.
- Le fichier `~/audit/inventaire-complet.md`, produit au chapitre 2, fournit l’adresse du serveur de mesure : `172.16.0.50`. La capture `~/audit/audit_capture.pcap` du chapitre 3 reste disponible comme plan de trafic de référence du poste d’audit ; elle ne contient aucun transfert `iperf3` et ne peut donc pas servir de preuve des débits mesurés aux étapes 2 et 3.
- À faire dans l’ordre : la latence se mesure avant le débit, sinon le lien est déjà saturé par le transfert.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Serveur iperf3 (laboratoire) | 172.16.0.50, port 5201 |
| Client | 172.16.0.10, interface `eth0` |
| Vitesse nominale du lien | 1 000 Mbit/s, valeur imposée par la configuration du laboratoire, et non relevée sur l’interface : le seuil de débit de 80 % s’y réfère, soit 800 Mbit/s |
| Protocole | TCP pour le débit, UDP pour le jitter et les pertes |
| Outils | `ping` d’iputils, `iperf3` (à installer des deux côtés si absent), `nmap`, `tc` de la suite iproute2, `tcpdump`, puis `tail`, `awk`, `grep`, `tr`, `sort` et `sed` de la distribution |
| Fichier repris | `~/audit/inventaire-complet.md` et `~/audit/audit_capture.pcap`, produits par les LAB des chapitres 2 et 3 |
| Fichier produit | `~/audit/mesures-performance.txt` |

#### Étapes du LAB

**Étape 1 — Latence, pertes et jitter (12min)**

```bash
# Le service de mesure doit répondre avant toute mesure de débit
nmap -sT -p 5201 172.16.0.50
# Le second ping envoie 20 paquets espacés d'une seconde : comptez vingt secondes
ping -c 10 172.16.0.1 | tail -5          # connectivité du segment
ping -c 20 172.16.0.50 2>&1 | awk '/rtt|loss/'   # latence vers la cible des mesures
# 95e centile de la latence : trier les 20 temps puis lire le 19e, rang du p95 sur 20 mesures
ping -c 20 172.16.0.50 | grep 'bytes from' | awk '{print $7}' | tr -d 'time=' | sort -n | sed -n '19p'
```

> **Résultat attendu pour la première commande :** la ligne `5201/tcp open` confirme que le service répond ; nmap affiche ici un intitulé technique hérité du registre de ports, `targus-getdata1`, qui ne décrit pas le service réel et ne doit pas être recopié dans le rapport. Un port `closed` signifie que le service n’est pas démarré et qu’il faut intervenir sur le serveur ; un port `filtered` signifie que le port est bloqué par un pare-feu, ce qui est un constat de sécurité à consigner.
>
> Le second `ping` met vingt secondes à s’exécuter, parce qu’il envoie un paquet par seconde. C’est normal : la commande n’est pas bloquée, il suffit d’attendre la fin du décompte. Le dernier `ping` en demande autant.

> **Résultat attendu :** le premier `ping` valide la connectivité du segment vers la passerelle, le second mesure la latence vers `172.16.0.50`, qui est la machine servie par `iperf3` : une latence mesurée vers la passerelle ne qualifie pas un débit mesuré vers le serveur. La sortie contient la ligne `0% packet loss` et la ligne `rtt min/avg/max/mdev`, par exemple `rtt min/avg/max/mdev = 0.045/0.062/0.089/0.011 ms`. Sur un segment local, on attend 0 % de pertes et un RTT moyen inférieur à 1 ms. Un mdev supérieur à 10 ms sur un segment local justifie une investigation, car il signale une forte variation. En cas d’échec, deux messages distincts doivent être lus séparément : `ping` signale `100% packet loss` s’il n’obtient aucune réponse, et `ping: connect: Network is unreachable` si aucun réseau ne mène à la cible ; dans les deux cas, c’est bien un problème réseau, de type lien coupé ou erreur d’adressage.
>
> La dernière commande affiche une valeur en millisecondes, le 95e centile de la latence : le seuil au-delà duquel ne se situent que 5 % des mesures. Les 95 % les plus rapides sont donc en dessous, et les 5 % les plus lents au-dessus : c’est ce que la moyenne ne montre pas. Elle sert à comparer la latence subie par les paquets les plus lents à la moyenne. Sur un segment local, les deux valeurs sont du même ordre de grandeur ; un écart important entre le p95 et la moyenne signale une congestion intermittente que la moyenne seule masque. C’est ce que l’exercice 4.2 demande de justifier.

**Étape 2 — Bande passante TCP (11min)**

```bash
# Sur le serveur, dans un second terminal : iperf3 -s
iperf3 -c 172.16.0.50 -t 10 -P 1 2>&1 | tail -15
```

> **Résultat attendu :** la sortie contient un en-tête `Interval Transfer Bitrate Retr`, puis deux lignes de synthèse, l’une `sender` et l’autre `receiver`, de la forme `[  5]   0.00-10.00   sec  1.02 GBytes  874 Mbits/sec    0    sender`. La colonne `Retr` compte les retransmissions : un total non nul signale des segments perdus sur le lien. Avec un seul flux parallèle, `-P 1`, iperf3 n’affiche aucune ligne `[SUM]` : ce total n’apparaît qu’à partir de deux flux parallèles, par exemple `-P 4`.
>
> Un débit mesuré tourne toujours **en dessous** de la vitesse nominale, parce qu’une part du trafic sert à acheminer les paquets eux-mêmes. Le calcul se fait segment par segment, sur un lien Ethernet à 1 Gbit/s. Un segment TCP utile de 1 460 octets est précédé de 20 octets d’en-tête IP et de 32 octets d’en-tête TCP, soit 52 octets d’en-têtes. Il faut encore compter, sur le câble, 7 octets de préambule, 1 octet de commencement de trame, 14 octets d’adresses et de type, puis 4 octets de contrôle de trame, et enfin 12 octets d’espacement entre trames. Le total sur le câble atteint donc 1 460 + 52 + 26 + 12 = 1 550 octets, pour 1 460 octets utiles. Le rendement utile du lien est de 1 460 sur 1 550, soit 94 % : sur un lien à 1 000 Mbit/s, un transfert TCP plafonne donc mathématiquement autour de 940 Mbit/s, avant même de compter le délai de propagation des acquittements, la charge de l’hyperviseur qui héberge le laboratoire et les pertes, qui coûtent cher en retransmissions.
>
> Le débit relevé au laboratoire, de l’ordre de 860 à 900 Mbit/s, reste sous ce plafond théorique, ce qui est normal et ne constitue pas un constat. Cette variation d’une exécution à l’autre vient de la charge de la machine qui héberge le laboratoire : le lien est bridé à 1 Gbit/s, mais la machine ne l’est pas toujours. La valeur exacte n’est pas reproductible, et c’est pourquoi elle n’est pas utilisée comme référence.
>
> Le seuil retenu ici est donc de 800 Mbit/s, soit 80 % de la vitesse nominale, et non 90 %. La raison n’est pas que 900 Mbit/s seraient toujours dépassés : c’est au contraire qu’ils le sont parfois, selon la charge de la machine. Un seuil placé à 900 Mbit/s donnerait un verdict « conforme » puis un verdict « non conforme » pour le même lien, sur la seule variation de la charge. Un critère qui change de conclusion d’une mesure à l’autre n’est pas un critère, et le rapport n’est pas reproductible. À 800 Mbit/s, l’écart au seuil reste plus large que la variation observée : le verdict tient. Un débit inférieur à 800 Mbit/s indique un goulot probable : examiner le duplex, la charge de l’équipement, puis la capacité du lien intermédiaire. Un débit **supérieur** à la vitesse nominale, lui, n’est jamais un bon résultat : il signale que l’interface réelle n’est pas celle qui est documentée, et l’inventaire est à corriger. Deux messages d’échec doivent être lus séparément. `iperf3: error - unable to connect to server ... Connection refused` signifie qu’un hôte a répondu par un refus : le service `iperf3 -s` n’est pas démarré sur `172.16.0.50`, ou il n’écoute pas sur le port 5201 ; un port filtré ne produit jamais un refus, il produit une temporisation. On le vérifie sur le serveur avec `sudo ss -tulpn | grep 5201`. Si le message est `Connection timed out`, le port est filtré, l’hôte est éteint ou la route manque : on reprend alors le test de latence de l’étape 1, qui distingue les trois cas.

**Étape 3 — Jitter et pertes en UDP (8min)**

```bash
iperf3 -c 172.16.0.50 -u -t 10 -b 1M -l 1460 2>&1 | tail -20
```

> Si `iperf3` est absent, consigner la dépendance et poursuivre avec les mesures des étapes 1 et 2 : ne pas supposer la présence de l’outil.

> **Résultat attendu :** l’en-tête de la section réceptrice indique `Jitter` puis `Lost/Total Datagrams`, et la ligne correspondante a la forme `[  5]   0.00-10.00  sec  1.19 MBytes  1.00 Mbits/sec  0.013 ms  0/857 (0.0%)  receiver`. Le pourcentage de pertes est écrit entre parenthèses. Le paramètre `-l 1460` fixe la taille de chaque datagramme, et il n’est pas décoratif : sans lui, iperf3 applique sa taille par défaut, qui varie selon la version et la machine installées, si bien que le nombre total de datagrammes diffère d’une exécution à l’autre. À 1 Mbit/s pendant 10 s, le volume utile est de 1,19 MBytes, soit environ 857 datagrammes de 1460 octets ; le total est donc reproductible. C’est aussi la valeur réellement obtenue au laboratoire, et elle est vérifiable : 857 datagrammes de 1 460 octets font 1 251 220 octets, soit 1,19 MBytes, ce qui correspond bien au volume affiché par la commande. Le total n’est pas arrondi, il compte les datagrammes reçus.
>
> Cette taille déclenche un avertissement de la forme `WARNING: UDP block size 1460 exceeds TCP MSS 1448, may result in fragmentation` : l’avertissement compare la taille du bloc UDP à celle d’un segment TCP, alors que la mesure ne fait aucun TCP. Il n’a aucune incidence ici et n’affecte ni le débit mesuré ni le nombre de datagrammes, mais il est consigné pour que sa présence ne soit pas interprétée à tort comme une anomalie du lien. La valeur 1 448 correspond au MSS d’un segment TCP pourvu d’horodatages, ce qui confirme que l’avertissement parle bien de TCP et non du flux mesuré.
>
> Pour un flux temps réel comme la voix ou la visioconférence, un jitter supérieur ou égal à 30 ms ou des pertes supérieures ou égales à 1 % constituent un constat d’anomalie. Ces deux valeurs sont des repères d’usage de la voix sur IP : le support ne les rattache à aucune norme, et un rapport d’audit les présente comme telles, en renvoyant à la politique écrite du client. Des pertes non nulles signalent une file de sortie saturée. Le paramètre `-b 1M` garde la mesure reproductible : c’est un débit faible volontaire, pas une erreur de saisie.
>
> En relisant la capture du chapitre 3 avec l’option `-v`, on rencontre des mentions de contrôle de somme erroné, de la forme `cksum 0x586d (incorrect -> 0xee35)`, sur les paquets TCP. Ce message ne signale pas une capture corrompue : il vient du fait que la somme de contrôle n’a pas été calculée à l’émission, la machine virtuelle ne le faisant pas. Le résultat est identique. Un rapport le mentionne comme une limite de l’outil et ne s’en sert pas pour conclure à une corruption du réseau.

**Étape 4 — Marquage DSCP et files d’attente (3min)**

Cette étape ne mesure rien : elle relève l’état de la qualité de service, indicateur annoncé au tableau de synthèse et jusqu’ici jamais exécuté. Sans elle, la ligne « QoS » du livrable resterait une affirmation sans preuve.

```bash
# File d'attente appliquée à l'interface de sortie du poste
tc -s qdisc show dev eth0
# Files d'attente dérivées de la file principale
tc class show dev eth0
# Octet de priorité relevé sur la capture du chapitre 3
tcpdump -r ~/audit/audit_capture.pcap -v 'ip and greater 8' 2>&1 | grep -m 3 'tos'
```

> **Résultat attendu :** la première commande affiche `qdisc noqueue 0: root refcnt 2` et des compteurs à zéro. Le mot `noqueue` signifie que l’interface n’applique aucune file d’attente : le paquet part dès que la carte réseau est prête. La seconde commande n’affiche **aucune ligne** : il n’existe donc aucune classe de traffic shaping configurée. La troisième affiche des mentions de la forme `tos 0x0` : la valeur est nulle, donc aucun bit DSCP n’est positionné et le trafic n’est pas marqué.
>
> Ce résultat n’est pas un constat de sécurité : `noqueue` et l’absence de marquage sont la configuration normale d’un poste d’audit, dont tout le trafic doit être traité de la même façon. Le consigner comme « sans objet » dans le livrable est un résultat correct, et c’est précisément ce qui distingue un indicateur vérifié d’un indicateur supposé. Sur un équipement de production, en revanche, un serveur de voix sans marquage partagerait sa file avec le reste du trafic : c’est là que l’absence de DSCP deviendrait un constat, et le tableau du chapitre permet de le formuler.

**Étape 5 — Consigner les mesures (5min)**

Le tableau se rédige dans le fichier de livrable, à partir des sorties déjà collectées, jamais de mémoire. Comme au chapitre 2, on enregistre avec `Ctrl + O` puis `Entrée`, et on quitte avec `Ctrl + X`.

```bash
nano ~/audit/mesures-performance.txt
```

Le contenu à saisir est le modèle suivant, où la colonne « Résultat observé » est remplie avec les valeurs réellement lues aux étapes 1 à 4 :

```markdown
# Mesures de performance — 172.16.0.50 depuis 172.16.0.10

| Indicateur | Commande | Résultat observé | Seuil appliqué | Constat |
|------------|----------|-------------------|----------------|---------|
| Pertes | ping -c 20 172.16.0.50 | à relever | 0 % sur segment local | À juger |
| RTT moyen | ping -c 20 172.16.0.50 | à relever | < 1 ms sur segment local | À juger |
| p95 de latence | idem, 19e valeur triée | à relever | non fixé par le support, valeur de comparaison à produire | À juger |
| Gêne temporelle (mdev) | ping -c 20 172.16.0.50 | à relever | < 10 ms sur segment local | À juger |
| Débit TCP | iperf3 -c 172.16.0.50 -t 10 -P 1 | à relever | 800 Mbit/s, soit 80 % des 1 000 Mbit/s nominaux | À juger |
| Jitter UDP | iperf3 -c 172.16.0.50 -u -t 10 -b 1M -l 1460 | à relever | < 30 ms, repère d'usage voix sur IP | À juger |
| Pertes UDP | iperf3 -c 172.16.0.50 -u -t 10 -b 1M -l 1460 | à relever | < 1 %, repère d'usage voix sur IP | À juger |
| Marquage DSCP | tcpdump -r ~/audit/audit_capture.pcap -v 'ip and greater 8' | à relever | Un marquage cohérent sur les flux prioritaires | À juger |
| Files d'attente | tc -s qdisc show dev eth0 | à relever | Aucun seuil chiffré : l'absence de file est normale sur un poste d'audit | À juger |
```

> **Résultat attendu :** le fichier `~/audit/mesures-performance.txt` contient une ligne par indicateur, avec la commande, le résultat observé, le seuil appliqué et la conclusion. Une mesure non réalisée est notée « non mesurée », jamais laissée vide. Le tableau distingue quatre choses qu’il ne faut pas confondre : la gêne temporelle, calculée par `ping` sur un flux de contrôle, le jitter, mesuré par `iperf3` sur un flux temps réel, le percentile p95, qui dépend de l’ordre des mesures et non de leur moyenne, et le marquage DSCP, qui décrit une file d’attente et non une mesure de débit. Les quatre indicateurs existent, ils ne répondent pas à la même question.

#### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `iperf3` se termine sur `Connection refused` | Le service de mesure n’écoute pas sur le serveur | Vérifier depuis le poste d’audit : `nmap -sT -p 5201 172.16.0.50` |
| `iperf3` se termine sur `Connection timed out` | Le port est filtré, ou l’hôte est éteint, ou la route manque | Reprendre le test de latence de l’étape 1, qui distingue les trois cas |
| `iperf3 : command not found` | L’outil n’est pas installé | Installer des deux côtés : `sudo apt install iperf3`, puis reprendre l’étape 1 |
| `iperf3` renvoie un débit supérieur à la vitesse nominale | L’interface réelle n’est pas celle qui est documentée | Reprendre l’inventaire du chapitre 2 : la vitesse documentée est erronée |
| La commande `iperf3` n’affiche aucune ligne `[SUM]` | Un seul flux parallèle a été demandé | C’est le comportement attendu avec `-P 1` ; le total n’apparaît qu’à partir de `-P 4` |
| `tc class show dev eth0` n’affiche aucune ligne | Aucune classe de traffic shaping n’est configurée sur l’interface | C’est l’état attendu du poste d’audit : le consigner comme « sans objet » et non comme un échec de la commande |
| `tcpdump -r … -v` n’affiche aucune ligne `tos` | Le fichier de capture est vide ou ne contient aucun paquet IP | Vérifier le fichier : `ls -l ~/audit/audit_capture.pcap`, puis reprendre l’étape 4 |
| Le volume UDP n’est pas reproductible d’une exécution à l’autre | Le paramètre `-l` a été omis, ou remplacé par sa valeur par défaut | Reprendre la commande complète, option `-l 1460` comprise |
| Un avertissement `UDP block size ... exceeds TCP MSS` s’affiche | L’avertissement compare la taille du bloc UDP à celle d’un segment TCP, sans rapport avec la mesure en cours | Le consigner et poursuivre : ni le débit ni le nombre de datagrammes ne sont affectés |
| Un `ping` long ne rend pas la main | C’est normal : `-c 20` envoie un paquet par seconde | Attendre la fin du décompte, vingt secondes |
| Le p95 calculé est vide | Une réponse ICMP a été perdue, donc il reste moins de vingt valeurs à trier | Reprendre la commande, ou noter « non mesurée » dans le livrable |

### Ce que ce LAB produit

`~/audit/mesures-performance.txt` : le jeu de mesures de référence du réseau de laboratoire, avec les seuils appliqués et leur justification. Les chapitres 1 et 2 du jour 2 en tireront les constats, pour distinguer un incident de sécurité d’une lenteur connue, et le rapport final s’appuiera sur ce tableau. Ne pas comparer une mesure future à une mesure prise dans un autre contexte, par exemple à une autre heure ou vers une autre cible.

### Exercices — Chapitre 4

**Exercice 4.1 — Interpréter des métriques**

1. Différencier le jitter et la latence, avec un exemple d’impact concret sur un appel vocal.
2. Interpréter un taux de pertes de 2 % sur un flux temps réel : quel indicateur faut-il regarder en plus, et que concluez-vous ?
3. Une mesure donne 940 Mbit/s sur un lien à 1 000 Mbit/s, et une autre 780 Mbit/s. Pour chacune, indiquez si un constat est posé, et justifiez avec le seuil retenu au laboratoire.

**Exercice 4.2 — Choisir des indicateurs auditables**

1. Proposer trois indicateurs de performance mesurables pour un audit, avec leur unité et leur fréquence de mesure.
2. Justifier l’usage des percentiles plutôt que de la moyenne seule, avec un exemple de distribution où la moyenne masque une gêne réelle.
3. Une interface est documentée à 1 Gbit/s et mesure 810 Mbit/s. Justifiez en une phrase pourquoi ce résultat ne déclenche pas de constat avec le seuil de 800 Mbit/s retenu par le support.

---

## Synthèse

### Couverture Jour 1

| Chapitre | Module | Thème | Livrable du LAB |
|----------|--------|-------|-----------------|
| C1 | M1 | Principes d’audit réseau | `~/audit/inventaire-brut.txt` |
| C2 | M2 | Cartographie et inventaire | `~/audit/inventaire-complet.md` |
| C3 | M3 | Analyse du trafic | `~/audit/audit_capture.pcap` |
| C4 | M4 | Mesure de performance | `~/audit/mesures-performance.txt` |

### Mémo — commandes essentielles

> Extraits de référence. Les paramètres sont à adapter à l’environnement réel du lab ; ne pas exécuter tels quels hors du laboratoire autorisé.

```text
# Découverte et inventaire
ip -br addr show                 # adresses et interfaces
ip route show                    # routage, route par défaut
ip neigh show                    # voisins directs et leur état
nmap -sn 172.16.0.0/24           # découvrir les hôtes actifs
nmap -sT -T4 -p 1-1000 172.16.0.20  # sonder les ports, sans droits
nmap -sT -p 5201 172.16.0.50     # vérifier qu'un service écoute
ss -tulnp                        # sockets en écoute
traceroute -n 172.16.0.1         # chemin réseau, sans résolution de noms

# Trafic
sudo timeout 20 tcpdump -i eth0 host 172.16.0.10 -w ~/audit/audit_capture.pcap  # capture passive
tcpdump -r ~/audit/audit_capture.pcap -n icmp            # lecture filtrée
tcpdump -r ~/audit/audit_capture.pcap -n 'tcp port 80'   # filtre sur un port
tcpdump -r ~/audit/audit_capture.pcap -n -v 'ip and greater 8'  # marquage de priorité
wireshark ~/audit/audit_capture.pcap                      # analyse visuelle (lecture)
tshark -r ~/audit/audit_capture.pcap -Y "tcp.port == 80" # repli en ligne de commande

# Performance
ping -c 20 172.16.0.50 | awk '/rtt|loss/'   # pertes et RTT vers la cible des mesures
ping -c 20 172.16.0.50 | grep 'bytes from' | awk '{print $7}' | tr -d 'time=' | sort -n | sed -n '19p'  # p95
iperf3 -c 172.16.0.50 -t 10 -P 1            # débit TCP soutenu
iperf3 -c 172.16.0.50 -u -t 10 -b 1M -l 1460  # jitter et pertes en UDP

# Files d'attente et marquage
tc -s qdisc show dev eth0                     # file d'attente et ses compteurs
tc class show dev eth0                        # classes de priorité configurées
```

---

## Ressources supplémentaires

### Outils utilisés

| Outil | Version indicative | Usage |
|-------|--------------------|-------|
| iproute2 | >= 5.x | Interfaces, routage, voisins |
| net-tools | >= 2.10 | `arp`, `netstat` en secours d'`iproute2` |
| nmap | >= 7.x | Découverte des hôtes, sondage des ports |
| tcpdump | >= 4.9 | Capture et filtres BPF |
| Wireshark | >= 3.x | Analyse visuelle des captures (lecture seule) |
| tshark | >= 4.x | Repli en ligne de commande, filtre d’affichage `-Y` |
| iperf3 | >= 3.1 | Mesures de débit, de jitter et de pertes |
| curl | >= 7.x | Génération de trafic HTTP |
| iputils-ping | >= 3.x | `ping` : latence, pertes et 95e centile, par l’option `-c` et le tri des temps |
| traceroute | >= 2.1 | `traceroute` : chemin réseau, un envoi par saut |
| iputils-tracepath | >= 20211215 | `tracepath`, repli quand `traceroute` est absent |
| iproute2 | >= 5.x | Fournit aussi `tc`, lecture des files d’attente et des classes |

### Références

| Document | Lien |
|----------|------|
| RFC 1242 — Terminologie des mesures de performance | https://www.rfc-editor.org/rfc/rfc1242.html |
| RFC 2544 — Méthodologie des mesures de performance | https://www.rfc-editor.org/rfc/rfc2544.html |
| RFC 2474 — Définition du champ de services différenciés | https://www.rfc-editor.org/rfc/rfc2474.html |
| RFC 4594 — Directives de configuration des classes de service différenciées | https://www.rfc-editor.org/rfc/rfc4594.html |
| Guide de référence nmap | https://nmap.org/book/man.html |
| Manuel de tcpdump | https://www.tcpdump.org/manpages/tcpdump.1.html |
| Documentation iperf3 | https://iperf.fr/iperf-doc.php |
| Guide de l’utilisateur Wireshark | https://www.wireshark.org/docs/wsug_html_chunked/ |
| IETF — groupe de travail sur la DiffServ | https://datatracker.ietf.org/wg/diffserv/about/ |
| CIS Controls | https://www.cisecurity.org/ |
