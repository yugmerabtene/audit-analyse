# Audit et Analyse des Réseaux — JOUR 2

## Sécuriser, restituer, auditer de bout en bout (09/10/2026)

---

## Comment lire ce support

La première journée a appris à regarder : on a inventorié les machines, capturé
le trafic, mesuré la qualité du lien. La deuxième journée passe à l’acte : on
juge la sécurité, on écrit ce que l’on a trouvé, puis on refait tout l’exercice
sur un réseau inconnu, en temps imparti.

Chaque chapitre suit la même progression qu’hier. Un préambule pose le problème
en langage courant. Un tableau « Le vocabulaire en clair » traduit les mots
 techniques avant leur emploi. Le contenu explique la théorie et ses tableaux.
Les techniques, pas à pas, disent ce que mesure chaque commande. Le LAB met les
mains sur le clavier et produit un fichier que le chapitre suivant consomme.

Le jour 2 s’appuie directement sur les livrables du jour 1 : l’inventaire et les
mesures de performance alimentent les constats, la capture est jointe au dossier d’audit comme
annexe. Rien n’est réinventé.

## Objectifs du Jour

- Vérifier la sécurité d’un réseau : points de contrôle, segmentation, listes de contrôle d’accès, risques courants
- Structurer un rapport d’audit et prioriser les recommandations en plan d’actions
- Réaliser un audit complet sur un réseau de laboratoire, en temps limité et en binôme
- Restituer les constats de manière factuelle, et identifier les axes d’amélioration

---

## Planning du Jour

| Chapitre | Horaires | Thème | Durée |
|----------|----------|-------|-------|
| C1 | 9h00–10h30 | M5 — Sécurité réseau | 1h30 |
| — | 10h30–10h45 | Pause | 15min |
| C2 | 10h45–12h15 | M6 — Rapport d’audit et recommandations | 1h30 |
| — | 12h15–13h45 | Pause déjeuner | 1h30 |
| C3 | 13h45–15h15 | M7 — Audit complet d’un cas d’étude | 1h30 |
| — | 15h15–15h30 | Pause | 15min |
| C4 | 15h30–17h00 | M7 (suite) — Restitution et débrief | 1h30 |

Volume pédagogique : 6 h par jour (quatre chapitres de 1 h 30), 2 h de pauses.

## Le fil du jour

La journée est une boucle : chaque LAB produit un livrable que le suivant consomme.

| Chapitre | LAB | Produit réutilisé par la suite |
|----------|-----|-------------------------------|
| C1 — M5 | Vérifier la sécurité d’un hôte | `~/audit/constats-securite.md` : constats C-01, C-02 et C-03, dont le service telnet et la limite de périmètre |
| C2 — M6 | Rédiger un rapport d’audit | `~/audit/rapport-audit-modele.md` : trame et constats priorisés, alimentés par le fichier de C1, puis complétés par C3 |
| C3 — M7 | Auditer un réseau de filiale | `~/audit/constats-filiale.md` : constats `F-` du second réseau, versés dans le rapport de C2 |
| C4 — M7 (suite) | Restituer et débriefer | `~/audit/support-binome.md` : page de restitution, et `~/audit/debrief-collectif.md` : axes d’amélioration retenus pour le prochain audit |

Les données viennent de la journée 1 : l’inventaire et les mesures de performance
alimentent les constats de sécurité, la capture est conservée en annexe. Chaque livrable est
réutilisé par le chapitre suivant : les constats par le rapport, le rapport par
la restitution, la restitution par le débrief.

---

## Chapitre 1 — M5 : Sécurité réseau (9h00–10h30)

### Préambule — une porte fermée, c’est une porte qu’on ne passe pas

Un réseau d’entreprise ressemble à un immeuble à plusieurs entrées. Chaque
service en écoute est une porte. Hier, on a listé ces portes sans juger. Une
porte n’est pas dangereuse en soi : ce qui compte, c’est de savoir lesquelles
ne sont même pas ouvertes à tous, et lesquelles ne devraient pas exister du
tout.

L’erreur la plus courante de l’auditeur est de confondre « accessible » et
« normal ». Un service d’administration accessible depuis le couloir du réseau
n’a rien d’anormal. Le même service accessible depuis Internet en est un. Un
port oublié, ouvert depuis six mois, ne sert jamais à personne et laisse passer
tout ce qui essaie.

C’est exactement ce que la journée traite : ce sur quoi l’auditeur statue, et
quelle commande fournit la preuve de ce jugement.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Point de contrôle | Un équipement qui regarde chaque paquet qui passe et décide de le laisser passer ou de le bloquer | Routeur, pare-feu, commutateur de niveau 3 |
| Segmentation | Découper le réseau en zones séparées, pour que le passage de l’une à l’autre soit contrôlé | Zone utilisateurs, zone serveurs, zone d’administration |
| Zone de confiance | Un ensemble de machines considérées aussi fiables les unes que les autres | Segment de production, segment de serveurs |
| DMZ | Une zone intermédiaire entre le réseau interne et l’extérieur, où l’on place ce qui doit être joignable depuis Internet | Serveurs applicatifs exposés |
| Liste de contrôle d’accès | Une liste de règles qui dit ce qui a le droit de passer d’une zone à l’autre, et ce qui est refusé | ACL sur un routeur ou un pare-feu |
| Moindre privilège | N’accorder à un compte que le strict minimum nécessaire à son travail | Un compte de lecture ne peut pas modifier la configuration |
| Journalisation | Le fait de garder une trace horodatée de ce qui a été fait sur le réseau | Les journaux du pare-feu, du routeur |
| Surface d’exposition | L’ensemble des points d’accès atteignables depuis un point donné | La liste des ports ouverts d’un serveur |
| Masque de sous-réseau | La suite de bits qui distingue la partie réseau d’une adresse IP ; le masque `0.0.0.0/0` n’en distingue aucune et laisse donc passer toute adresse source | C1, règle trop permissive |

### Objectifs du chapitre

- Identifier les points de contrôle et les zones de confiance d’un petit réseau
- Vérifier une segmentation et une liste de contrôle d’accès sans rien modifier
- Reconnaître les risques courants : ports oubliés, gestion non chiffrée, comptes par défaut
- Distinguer un constat démontré d’une simple observation

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 25min |
| Techniques, pas à pas | 7min |
| LAB | 45min |
| Exercices | 5min |

### Contenu du chapitre (25min)

#### 1. Points de contrôle, segmentation et zones de confiance (12min)

Un point de contrôle examine chaque paquet selon ses règles. La segmentation
découpe le réseau en zones, et la frontière entre deux zones est justement un
point de contrôle. Le principe du moindre privilège s’applique à ces points :
une règle est d’autant plus solide qu’elle refuse ce qui n’est pas explicitement
autorisé.

| Concept | Définition | Vérification |
|---------|-----------|--------------|
| Point de contrôle | Équipement traversé par les flux | Règles, ordre des règles |
| Segmentation | Division du réseau en zones | VLAN, sous-réseaux, zones de confiance |
| Moindre privilège | N’autoriser que ce qui est nécessaire | Règle explicite plutôt que règle par défaut |
| Journalisation | Trace des événements et des tentatives | Journaux, rétention, corrélation |

Le principe du moindre privilège se traduit par une question simple : cette
règle existe-t-elle pour une raison identifiée, ou seulement parce que personne
ne l’a supprimée ?

#### 2. Listes de contrôle d’accès, risques courants et vérifications (13min)

Les listes de contrôle d’accès concentrent l’essentiel des risques de
configuration, pour une raison simple : elles sont écrites vite, lues rarement,
modifiées souvent. Elles s’évaluent dans l’ordre de leurs règles, et la première
règle qui correspond l’emporte.

| Domaine | Risque courant | Vérification |
|---------|---------------|--------------|
| Listes de contrôle d’accès | Règles trop permissives, ordre erroné, masque trop large | Lire l’ordre, les masques, tester règle par règle |
| Services exposés | Port de gestion laissé ouvert, protocole en clair | Inventaire des ports, protocole et chiffrement |
| Configuration | Compte par défaut, micrologiciel non mis à jour | Relevé de version, comptes présents |
| Détection | Balayage, trafic anormal, diffusion excessive | Captures, statistiques, journaux |

L’erreur classique d’un masque : `0.0.0.0/0` signifie « n’importe quelle
adresse source » ; combiné à un port quelconque, il n’impose aucune restriction du tout. Il est
aussi facile d’écrire un trou que de trop ouvrir : une règle qui bloque tout le
trafic de gestion empêche l’administrateur de travailler.

### Les techniques, pas à pas

**Technique 1 — Cartographier la surface d’exposition.** La commande
`nmap -sT` envoie une demande de connexion sur les ports demandés et lit la
réponse. Elle prouve qu’un port accepte ou refuse la connexion depuis le poste
d’audit. Elle ne prouve pas qui est autorisé à l’utiliser depuis un autre
réseau, et ne dit rien du chiffrement.

**Technique 2 — Lire les règles sans les modifier.** `iptables -L -n -v` et
`nft list ruleset` affichent les règles en place. Ces commandes exigent un compte
privilégié. Sans ce compte, elles échouent : l’absence de règles ne doit jamais
être déduite d’une commande qui n’a pas pu s’exécuter.

**Technique 3 — Distinguer port fermé et port filtré.** Un port `closed`
répond que rien n’écoute. Un port `filtered` ne répond pas du tout : un
équipement intermédiaire a bloqué la demande. Le premier est un constat, le
second oriente vers la segmentation.

**Technique 4 — Séparer l’observation du jugement.** L’observation est
factuelle : « le port 23 est ouvert ». Le jugement est raisonné : « le port 23
est ouvert sur un hôte de production ». Les deux figurent au rapport, dans deux
colonnes distinctes.

### LAB — Vérifier la sécurité d’un hôte du laboratoire (45min)

#### Prérequis

- Le dossier de travail `~/audit` doit exister. Le jour 1 l’a créé ; si le stage démarre directement sur cette journée, on le crée ici :

```bash
mkdir -p ~/audit && ls -la ~/audit
```

- Cinq fichiers du jour 1 sont repris par ce chapitre. **Le laboratoire les fournit déjà** : le geste de vérification ci-dessous sert à le constater, pas à rattraper une absence.

```bash
ls -l ~/audit/
```

| Fichier repris | Ce que ce chapitre en fait |
|----------------|---------------------------|
| `inventaire-brut.txt` | Repris comme modèle de méthode, non ouvert dans ce LAB : la procédure d’inventaire appliquée la veille est celle des étapes 1 et 2 |
| `inventaire-complet.md` | L’adresse de l’hôte applicatif et celle du serveur de mesure, donc le périmètre à auditer |
| `ports-20.txt` | Ouvert à l’étape 3 pour comparer l’état des ports de `172.16.0.20` à celui de la veille : cet inventaire sert de référence pour vérifier que le constat telnet est stable, et non un état passager |
| `audit_capture.pcap` | Non ouvert dans ce LAB ; conservé en annexe du rapport par le chapitre 4 |
| `mesures-performance.txt` | Ouvert à l’étape 3 pour relire les latences de référence, et distinguer un incident de sécurité d’une lenteur déjà mesurée |

Le laboratoire ajoute deux fichiers suffixés `-essai`, qui sont des modèles vierges de reprise : ils ne sont pas des livrables et n’entrent pas dans l’inventaire. Un message `No such file or directory` désigne un fichier absent : dans ce cas, annoncer au formateur que l’étape de préparation du laboratoire n’a pas été exécutée, et non poursuivre sur des valeurs inventées. Une mesure non réalisée se note « non mesurée », jamais par un chiffre plausible.

- Aucun droit administrateur n’est nécessaire pour suivre ce LAB : il ne modifie aucune règle, il les lit. L’étape 2 s’exécute sans privilège et se contente de renvoyer un code retour ; pour obtenir la preuve, on relance la même commande avec `sudo`, par exemple `sudo nft list ruleset`.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Client d’audit | Kali Linux / Parrot OS — adresses 172.16.0.10/24 et 192.168.50.10/24 |
| Réseau audité | 172.16.0.0/24, routeur 172.16.0.1 |
| Hôte applicatif | 172.16.0.20 — services ouverts attendus : 22 (SSH), 23 (telnet, volontairement exposé), 80 (HTTP) ; 443 et 3389 doivent rester fermés |
| Serveur de mesure | 172.16.0.50 — port 5201 ouvert pour les mesures du jour 1 |
| Constats attendus | C-01 : telnet en clair sur 172.16.0.20, repris au chapitre 2. C-02 : application servie en clair sur le port 80. C-03 : règles de filtrage non vérifiables en compte ordinaire |
| Outils | `ip`, `nmap`, `ping`, `ss`, `nano`, et en lecture `iptables` ou `nft` si le compte le permet |
| Fichier produit | `~/audit/constats-securite.md` |

#### Étapes du LAB

**Étape 1 — Cartographier la surface d’exposition (15min)**

On liste ce qui est en écoute, puis on demande explicitement l’état des ports
sensibles. Les numéros de ports sont énumérés un par un : une plage large
s’arrêterait avant le port 3389 et laisserait un angle mort.

```bash
ip route show
ss -tulnp 2>/dev/null || netstat -tulnp 2>/dev/null
nmap -sT -T4 -p 22,23,80,443,3389 172.16.0.20 2>/dev/null
```

> **Résultat attendu :** `ip route show` affiche une route par défaut via 172.16.0.1 ; `ss -tulnp` liste les services en écoute du poste — la colonne qui donne le nom du processus reste vide tant que les droits administrateur ne sont pas élevés, et la commande s’exécute normalement dans les deux cas ; nmap renvoie les cinq ports demandés, où 22, 23 et 80 sont `open` et 443 et 3389 sont `closed`. Les ports demandés étant énumérés un par un, nmap n’affiche aucune ligne « Not shown » : ce message n’apparaît que sur un balayage de plage, où il récapitule les ports non affichés. Tout port ouvert absent de la liste attendue est un écart à consigner.

**Étape 2 — Lire les règles de filtrage sans les modifier (15min)**

La lecture des règles demande un compte privilégié. On sépare explicitement les
deux cas : des règles absentes, ou une commande qui n’a pas pu s’exécuter. L’option
`--line-numbers` numérote les règles, ce qui rend l’ordre d’évaluation visible :
c’est cet ordre, et non le nombre de règles, qui décide du comportement réel.

```bash
if command -v nft >/dev/null 2>&1; then nft list ruleset 2>/dev/null; echo "code retour : $?"; fi
if command -v iptables >/dev/null 2>&1; then iptables -L -n -v --line-numbers 2>/dev/null; echo "code retour : $?"; fi
```

> **Résultat attendu :** avec un compte privilégié, `iptables -L -n -v --line-numbers` affiche les trois chaînes de filtrage du noyau (INPUT, FORWARD, OUTPUT) avec leur politique, leurs compteurs et un numéro de règle par ligne. `nft list ruleset` affiche l’ensemble des tables présentes — dans un environnement de laboratoire conteneurisé, seule la table de traduction d’adresses créée automatiquement par le moteur de conteneurs peut apparaître, sans aucune table de filtrage, ce qui est normal et ne signifie pas que le filtrage a été désactivé. Une politique d’acceptation par défaut sur les trois chaînes constitue un constat à consigner.
>
> Sans compte privilégié, les deux commandes échouent et les messages partent sur la sortie d’erreur, que la redirection `2>/dev/null` masque : le code retour affiché reste néanmoins significatif. Il vaut **4** pour `iptables`, qui signale par ce code une erreur d’autorisation, et **1** pour `nft`, qui utilise le code générique d’échec. Un code retour non nul ne prouve donc pas l’absence de règles : dans ce cas, on note « non vérifiable en compte ordinaire » au lieu de conclure quoi que ce soit sur le filtrage. Pour lever le doute, on relance **chacune des deux commandes séparément**, sans la redirection `2>/dev/null`, afin de lire le message d’erreur que la redirection masquait.
>
> Une limite de périmètre s’applique ici et doit être énoncée dans le rapport. Les règles lues sont celles du poste d’audit, et non celles de l’hôte `172.16.0.20` : un audit sans accès administratif à l’équipement distant ne peut pas affirmer que sa liste de contrôle d’accès est correcte. Ce que l’étape établit, c’est la méthode de lecture et la position du poste d’audit, ce qui est une preuve ; ce qu’elle n’établit pas, c’est la configuration de l’hôte audité, ce qui est une limite. Un rapport qui présente la seconde comme la première est un rapport faux, et c’est le cas le plus fréquent de sur-interprétation en audit réseau.

**Étape 3 — Contrôler, puis consigner les constats (15min)**

On confirme que l’hôte répond, puis on reprend l’état des ports sensibles avec
un second balayage plus ciblé, pour vérifier la stabilité du constat. On écrit
ensuite le fichier de constats : c’est lui que le chapitre 2 consomme, il ne se
déduit pas du dialogue.

```bash
ping -c 3 -W 1 172.16.0.20
nmap -sT -T4 -p 23,443,3389 172.16.0.20 2>/dev/null
# Confronter l'état des ports à celui de la veille
cat ~/audit/ports-20.txt
# Relire les latences de référence, pour ne pas confondre lenteur et incident
grep -iE 'aller-retour|p95' ~/audit/mesures-performance.txt
# Rédiger le livrable que le chapitre 2 va consommer
nano ~/audit/constats-securite.md
```

> **Résultat attendu :** trois réponses ICMP, aucun paquet perdu, temps de réponse inférieur à 1 ms sur un segment local ; le second balayage confirme 23 `open` (le constat C-01) et 443 et 3389 `closed`. Une réponse différente entre les deux passages est notée avec son horodatage. Les deux passages se confrontent enfin à `~/audit/ports-20.txt`, que la commande `cat` affiche : un port qui a changé d’état entre le premier jour et le second est un constat plus grave qu’un port ouvert constant, car il signale une modification non tracée. La commande `grep`, avec son option `-i` qui ignore la casse, relit les deux latences de référence du jour 1, inférieures à 1 ms : le temps de réponse relevé ici étant du même ordre, l’hôte n’est pas lent, et l’ouverture du port 23 ne s’explique donc pas par une dégradation du lien. Cette distinction est celle que l’audit doit toujours opérer : un service exposé et un service lent sont deux constats de nature différente, et les confondre conduit à une recommandation fausse.
>
> L’éditeur s’ouvre sur un fichier vide : on y inscrit trois entrées, chacune avec ses cinq champs. **C-01**, le service telnet joignable en clair sur `172.16.0.20`, criticité Élevée. **C-02**, l’application servie en clair sur le port 80, ce qui expose identifiants et données applicatives, criticité Moyenne. **C-03**, la limite de périmètre trouvée à l’étape 2 : les règles de filtrage de l’hôte audité n’ont pas pu être lues, la mention exacte est « non vérifiable en compte ordinaire » et la criticité est « Sans objet », puisqu’une limite de périmètre n’est pas un constat de sécurité. Le fichier contient donc **trois entrées** : le chapitre 2 a besoin de trois lignes dans son tableau des constats, et deux lignes le laisseraient incomplet. C-02 n’est pas une doublure de C-01 : les deux services répondent au même défaut, mais un protocole d’administration et une application ne présentent pas le même risque ni la même correction. Le fichier est enregistré avant de quitter l’éditeur, et `cat ~/audit/constats-securite.md` en relit le contenu.

##### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `ls` renvoie `No such file or directory` sur un livrable du jour 1 | L’étape de préparation du laboratoire n’a pas été exécutée | Annoncer la situation au formateur, et inscrire « non disponible » dans le livrable plutôt que d’inventer une valeur |
| `mkdir -p ~/audit` échoue | Le dossier existe déjà et appartient à un autre compte | Vérifier le compte courant avec `whoami`, puis reprendre avec les droits adequats |
| `nano` refuse d’ouvrir le fichier | Le dossier `~/audit` n’existe pas encore | Le dossier a déjà été créé à l’étape de préparation ; vérifier le chemin avec `pwd`, puis recommencer |
| `iptables` ou `nft` renvoie `you must be root` | Le compte est ordinaire, ce qui est le cas attendu | Noter « non vérifiable en compte ordinaire », puis relancer avec `sudo` pour obtenir la preuve |
| La commande affiche l’en-tête `Chain INPUT` puis aucune ligne de règle | Aucune règle de filtrage n’est configurée sur cette interface, la colonne de numérotation reste donc vide | Le constater et le consigner : l’absence de règle sur le poste d’audit n’est pas une faille du réseau audité, mais elle interdit d’affirmer que le filtrage est en place |
| `nmap` renvoie `Host seems down` sur une adresse qui existe | Adresse mal saisie, ou ICMP filtré | Relire l’adresse, puis relancer avec `-Pn` |
| Le second balayage contredit `~/audit/ports-20.txt` | L’état d’un port a changé depuis la veille | Le consigner comme constat, avec la date des deux relevés : une modification non tracée est plus grave qu’un port ouvert constant |
| Le fichier `constats-securite.md` ne contient que deux entrées | L’entrée de limite de périmètre a été omise | La rédiger : le chapitre 2 attend trois lignes dans son tableau des constats |

### Ce que ce LAB produit

`~/audit/constats-securite.md` : la liste des constats bruts du jour, chacun avec
son identifiant, sa commande de preuve et son extrait de sortie. C-01 est
l’exemple de référence : service telnet joignable en clair sur 172.16.0.20. C-02
concerne le port 80 du même hôte, C-03 consigne la limite de périmètre. Le
chapitre 2 consommera les trois entrées pour écrire le rapport.

### Exercices — Chapitre 1

**Exercice 1.1 — Analyse de risque**

1. Citer deux risques liés à une liste de contrôle d’accès trop permissive, et l’impact de chacun.
2. Proposer une vérification simple pour établir si un service de gestion est joignable depuis un autre réseau que celui du serveur.

**Exercice 1.2 — Port fermé et port filtré**

1. Expliquer la différence entre un port `closed` et un port `filtered` du point de vue de l’auditeur.
2. Dire lequel des deux résultats justifie d’examiner la segmentation en priorité.

---

## Chapitre 2 — M6 : Rapport d’audit et recommandations (10h45–12h15)

### Préambule — un constat sans preuve ne vaut rien

Un rapport d’audit se juge sur une chose simple : ce qui n’est pas clair n’est pas fait. L’image utile est celle du
dossier d’instruction : chaque affirmation est rattachée à une pièce. Un
rapport d’audit suit la même logique. S’il contient une affirmation sans
preuve reproductible, cette affirmation peut être contestée, et c’est souvent
tout le rapport qui s’effondre avec elle.

Le deuxième jour est souvent le jour où l’auditeur découvre que son travail
n’existe pas encore. Les constats sont dans un carnet, les mesures dans un
tableur, et personne ne peut répondre à la question simple : « qu’est-ce qu’on
a trouvé, et qu’est-ce qu’on propose de faire ? »

Ce chapitre construit ce livrable, avec une contrainte simple : chaque section
a une fonction, et l’ordre des sections suit l’ordre des questions que se pose
le lecteur.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Rapport d’audit | Le document qui rassemble ce qui a été trouvé et ce qu’il faut faire | Livrable final du chapitre |
| Constat | Une affirmation factuelle sur le réseau, accompagnée d’une preuve | Ligne du tableau des constats |
| Preuve | La commande exécutée et son extrait de sortie, avec la date | Colonne « preuve » du constat |
| Criticité | La gravité du constat, de faible à critique | Colonne de priorisation |
| Recommandation | L’action proposée pour traiter un constat | Tableau des recommandations |
| Plan d’actions | L’ordre d’exécution : qui fait quoi, et pour quand | Tableau du plan d’actions |
| Échéance | La date à laquelle l’action doit être terminée | Colonne du plan d’actions |

### Objectifs du chapitre

- Structurer un rapport en sections stables, qui répondent toujours aux mêmes questions
- Rédiger un constat factuel : identifiant, description, preuve, impact, criticité
- Prioriser des recommandations à l’aide de critères explicites
- Construire un plan d’actions avec responsable et échéance

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 25min |
| Techniques, pas à pas | 7min |
| LAB | 45min |
| Exercices | 5min |

### Contenu du chapitre (25min)

#### 1. Les six sections d’un rapport d’audit (12min)

Un rapport se lit en deux minutes dans sa synthèse, puis se parcourt par
sections. Chaque section répond à une question précise, et les questions
sont toujours posées dans le même ordre.

| Section | Contenu | Exigence |
|---------|---------|----------|
| Synthèse | Constats majeurs et risques principaux | Lisible en deux minutes |
| Périmètre et méthode | Réseau audité, outils, dates, limites | Transparence sur ce qui a été couvert |
| Constats | Fait, preuve, impact, criticité | Factuel et reproductible |
| Recommandations | Action, cible, priorité, effort | Actionnable |
| Plan d’actions | Ordre d’exécution, responsable, échéance | Exécutable |
| Annexes | Sorties de commande, captures, inventaires | Traçabilité |

L’ordre n’est pas décoratif. Un lecteur pressé lit la synthèse et s’arrête.
Un lecteur technique lit le périmètre pour savoir ce qui a été audité, puis les
constats, puis les annexes pour vérifier. Cet ordre évite qu’on doive
relire le rapport pour comprendre ce qu’il couvre.

#### 2. Priorisation et plan d’actions (13min)

Prioriser consiste à croiser deux grandeurs : l’impact du problème, et la
probabilité qu’il se matérialise. Le plan d’actions répond ensuite à une autre
question : dans quel ordre, et par qui.

| Critère | Question posée | Usage |
|---------|----------------|-------|
| Criticité | Échelle fixe en quatre niveaux : Faible, Moyenne, Élevée, Critique. La valeur « Sans objet » s’y ajoute pour les éléments de contexte qui ne sont pas des constats de sécurité, comme une limite de périmètre. La valeur « Sans objet » n’entre pas dans le classement : elle signifie « à documenter, pas à prioriser ». Élevée quand le constat porte sur un service de gestion en clair ; Critique quand il permet la prise de contrôle de la machine | Colonne Criticité des deux tableaux de constats |
| Probabilité | Le risque est-il susceptible de se produire ? | Priorité |
| Effort | Combien de temps, de compétences, de ressources ? | Planification |
| Dépendances | Une action en bloque-t-elle une autre ? | Ordre d’exécution |

L’erreur courante consiste à prioriser sur le seul effort : ce qui est rapide
passe devant tout, et le risque le plus grave attend la semaine suivante. La
criticité d’abord, l’effort ensuite, dans cet ordre.

### Les techniques, pas à pas

**Technique 1 — Rédiger un constat en cinq champs.** Un constat complet porte
un identifiant stable (C-01 sur le réseau du chapitre 1, F-01 sur celui du
chapitre 3), une description factuelle, la commande qui
prouve cette description, un impact en une phrase et une criticité. Le
formulaire exact est donné dans le LAB.

**Technique 2 — Recopier la preuve, pas l’interprétation.** La colonne « preuve »
contient la commande et l’extrait de sortie, pas un résumé en français. Le
lecteur doit pouvoir rejouer la commande et retrouver le même résultat.

**Technique 3 — Rendre la criticité reproductible.** La criticité se justifie
en une phrase : « critique : l’accès donne la main sur le serveur ». « Important »
sans justification n’est pas une criticité, c’est un avis.

**Technique 4 — Séparer l’action de son responsable.** Une recommandation dit
quoi faire. Le plan d’actions dit qui le fait et pour quand. Confondre les deux
produit un rapport qu’on ne peut pas suivre.

### LAB — Rédaction d’un rapport d’audit (45min)

#### Prérequis

- Le fichier `~/audit/constats-securite.md`, produit par le LAB du chapitre 1, fournit la matière première : chaque ligne devient un constat du rapport.
- Le fichier `~/audit/inventaire-complet.md`, produit par le LAB du chapitre 2 du jour 1, sert d’annexe et justifie le périmètre déclaré. Il est disponible : le laboratoire le fournit.
- Le fichier `~/audit/mesures-performance.txt`, produit par le LAB du chapitre 4 du jour 1, fournit les mesures de référence à citer dans la section périmètre et méthode. Lui aussi est disponible.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Matériel | Poste avec éditeur de texte ou tableur |
| Données | `~/audit/constats-securite.md`, `~/audit/inventaire-complet.md`, `~/audit/mesures-performance.txt` |
| Durée | 45 minutes |
| Livrable | `~/audit/rapport-audit-modele.md`, mis à jour au chapitre 4 avec le second réseau et les constats `F-` |

#### Étapes du LAB

**Étape 1 — Poser la trame du rapport (15min)**

La trame reprend exactement les six sections vues dans le contenu du chapitre.
L’ordre des titres ne change pas d’un audit à l’autre. Le fichier est d’abord créé
et ouvert dans l’éditeur, puis son contenu est saisi.

```bash
nano ~/audit/rapport-audit-modele.md
```

Le contenu à saisir dans l’éditeur est le modèle suivant. Dans l’éditeur,
`Ctrl + O` enregistre, `Entrée` confirme le nom du fichier, `Ctrl + X` quitte.

```markdown
# Rapport d'audit réseau — laboratoire 172.16.0.0/24

## Synthèse

## Périmètre et méthode

## Constats

| ID | Description | Preuve | Impact | Criticité |
|----|-------------|--------|--------|-----------|

## Recommandations

| ID | Action | Priorité | Effort |
|----|--------|----------|--------|

## Plan d'actions

| Action | Responsable | Échéance |
|--------|--------------|-----------|

## Annexes
```

> **Résultat attendu :** le fichier `~/audit/rapport-audit-modele.md` a été créé par la commande précédente et contient les six titres de section, dans cet ordre, avec les en-têtes de colonnes des trois tableaux. La section Synthèse reste vide : elle se rédige à la fin, une fois les constats rédigés.

**Étape 2 — Documenter un constat à partir des preuves (15min)**

On prend la première ligne du fichier de constats et on la transforme en ligne
de rapport. Les cinq champs sont obligatoires, et la preuve est recopiée
telle quelle.

| Champ | Exemple rempli |
|-------|----------------|
| ID | C-01 |
| Description | Le service telnet est joignable en clair sur l’hôte 172.16.0.20 |
| Preuve | `nmap -sT -T4 -p 23 172.16.0.20` renvoie la ligne `23/tcp   open   telnet`, les colonnes étant alignées par nmap |
| Impact | Les identifiants transmis sur ce service circulent en clair et sont interceptables |
| Criticité | Élevée : l’accès à l’hôte est possible sans chiffrement |

> **Résultat attendu :** le tableau Constats contient **trois lignes** issues de `~/audit/constats-securite.md`, dont C-01 et C-02. La troisième est l’entrée de limite de périmètre inscrite à l’étape 2 du chapitre 1, avec la mention « non vérifiable en compte ordinaire » et une criticité « Sans objet », puisque ce n’est pas un constat de sécurité. Le tableau accueille les deux natures dans une colonne unique, et la criticité permet de les séparer : un lecteur qui ne veut que les risques filtre sur une criticité différente de « Sans objet ». Chaque ligne porte un identifiant, une description sans jugement, une preuve contenant la commande et un extrait, un impact en une phrase et une criticité justifiée.

**Étape 3 — Rédiger la synthèse, prioriser et planifier (15min)**

Le rapport s’écrit dans le fichier, jamais dans le dialogue. La synthèse se rédige
maintenant, une fois les constats documentés : elle ne s’ajoute pas après coup, elle
se tire d’eux. Trois lignes suffisent, les constats majeurs et le risque
principal.

```bash
# Remplir la trame posée à l'étape 1, synthèse comprise, puis la relire
nano ~/audit/rapport-audit-modele.md
grep -c '^|' ~/audit/rapport-audit-modele.md
```

> **Résultat attendu :** `nano` ouvre la trame de l’étape 1. On remplit Périmètre
> et méthode, Constats, Recommandations, Plan d’actions et Annexes, puis on rédige
> la Synthèse en trois lignes : le réseau audité, les constats majeurs, le risque
> principal. `grep` renvoie le nombre de lignes de tableau du rapport. La trame
> vide de l’étape 1 en compte déjà six, deux lignes par tableau : un en-tête et
> son trait de séparation, pour les trois tableaux Constats, Recommandations et
> Plan d’actions. Un seuil de cinq serait donc satisfait avant même la première
> saisie, et ne prouverait rien. Le rapport complet renvoie **quinze lignes** :
> trois lignes de constats, trois de recommandations, trois de plan d’actions, et
> les six lignes d’en-tête que compte déjà la trame. Les trois tableaux portent
> le même nombre de lignes, la règle étant qu’une recommandation traite un
> constat et qu’une action du plan traite une recommandation. Un nombre proche
> de six signifie que les tableaux sont restés vides : il faut reprendre la
> saisie.

On attribue ensuite une priorité à chaque recommandation, puis on ordonne
l’exécution. Les priorités suivent la criticité, l’ordre d’exécution suit les
dépendances et l’effort.

| Priorité | Critère d’attribution | Exemple d’action | Échéance |
|----------|-----------------------|------------------|----------|
| P1 | Critique et probable | Désactiver le service telnet et n’ouvrir que SSH | Sous 48 h |
| P2 | Impact moyen, effort modéré | N’autoriser SSH que par clés, et interdire le mot de passe distant | Sous 15 jours |
| P3 | Amélioration de confort | Restreindre l’interface de gestion au seul réseau d’audit, après avoir vérifié les règles | Sous 3 mois |

> **Résultat attendu :** le tableau Recommandations contient au moins trois actions, chacune rattachée à l’identifiant du constat qu’elle traite, avec une priorité justifiée par la criticité et la probabilité. Le tableau Plan d’actions contient le même nombre de lignes, avec un responsable et une échéance pour chacune, et l’ordre d’exécution respecte les dépendances. L’action P1 porte une échéance inférieure ou égale à 48 heures.

##### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `nano` refuse d’ouvrir le fichier | Le dossier `~/audit` n’existe pas | Exécuter `mkdir -p ~/audit` |
| `nano` s’ouvre sur un fichier vide alors qu’il était déjà rempli | Un fichier portant le même nom a été créé dans un autre dossier | Vérifier le chemin affiché par l’éditeur et le dossier courant avec `pwd` |
| Le tableau Constats ne compte que deux lignes | Le fichier de constats du chapitre 1 ne contient que deux entrées | Reprendre l’étape 3 du chapitre 1 : les trois entrées C-01, C-02 et C-03 sont attendues |
| `grep -c '^\|'` renvoie `6` | Les tableaux sont restés vides : c’est le nombre de lignes de la trame seule | Reprendre la saisie du fichier `~/audit/rapport-audit-modele.md` |
| `grep -c '^\|'` renvoie un nombre autre que `15` | Les trois tableaux ne portent pas le même nombre de lignes, ou une saisie a été omise | Compter chaque tableau séparément avec `grep -c '^\|' ` suivi d’un nom de fichier distinct, puis rétablir l’égalité à trois lignes de données par tableau |
| Le tableau Recommandations est plus long que le tableau Constats | Une action a été dédoublée ou dupliquée | Vérifier que chaque recommandation porte l’identifiant du constat qu’elle traite, et qu’aucun identifiant n’y figure deux fois |
| Le plan d’actions ne reprend pas toutes les recommandations | Le plan a été rédigé avant la fin des recommandations | Reprendre le tableau Plan d’actions : il doit compter autant de lignes que le tableau Recommandations |
| L’échéance P1 dépasse 48 heures | La priorité a été posée après coup, sans lire le critère | Reprendre le tableau des priorités : P1 signifie « critique et probable », donc sous 48 h |

### Ce que ce LAB produit

`~/audit/rapport-audit-modele.md` : le rapport complet, avec ses trois tableaux
renseignés et sa section Synthèse rédigée. Le chapitre 3 vérifiera sa structure
en produisant le même type de document sur le réseau de la filiale, et le
chapitre 4 en tirera le support de restitution.

### Exercices — Chapitre 2

**Exercice 2.1 — Transformer une observation en constat**

1. Transformer l’observation vague « le réseau est lent » en constat mesurable, en précisant la métrique, la méthode et la période.
2. Justifier en une phrase pourquoi la preuve est obligatoire dans un constat.

**Exercice 2.2 — Ordre d’exécution**

1. Classer trois recommandations par priorité, en justifiant chaque classement avec la criticité.
2. Proposer un responsable et une échéance pour chacune, puis dire laquelle dépend des deux autres.

---

## Chapitre 3 — M7 : Audit complet d’un cas d’étude (13h45–15h15)

### Préambule — recommencer sans modèle, sur un réseau inconnu

Les deux premiers chapitres ont travaillé sur un réseau que l’auditeur connaît
déjà : celui du laboratoire, décrit hier. Un vrai audit commence sur un réseau
que personne n’a encore décrit. C’est là que la méthode se mesure : non plus sur
des informations rangées, mais sur ce que l’on déduit en trois quarts d’heure.

Le cas d’étude est celui d’une petite filiale, entièrement reconstituable au
laboratoire. Il comporte trois machines et une passerelle, soit quatre hôtes au
total, ce qui est peu, et
c’est précisément le but : un auditeur qui ne trouve pas quatre machines en
dix minutes ne trouvera pas un site entier en une semaine.

Le format est le binôme. Deux rôles distincts, une trace écrite commune, et un
temps arrêté. Ces contraintes ne sont pas des formalités : elles sont ce qui
rend le résultat réutilisable.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Cas d’étude | Un réseau de laboratoire reproduisant une situation réelle | Réseau 192.168.50.0/24 |
| Périmètre | L’ensemble des machines et des services que l’auditeur s’engage à couvrir | Les trois machines du tableau, plus la passerelle |
| Binôme | Deux personnes qui se partagent les rôles et un livrable commun | Auditeur 1 et auditeur 2 |
| Horodatage | L’heure exacte à laquelle une commande a été lancée | En-tête de chaque preuve |
| Trouvaille | Un élément inhabituel observé pendant l’audit | Port ouvert inattendu, latence anormale |
| Écart | La différence entre ce qui était attendu et ce qui est observé | Port ouvert non prévu au scénario |

### Objectifs du chapitre

- Appliquer la méthodologie complète de M1 à M6 sur un réseau non décrit
- Répartir les rôles en binôme et tenir un livrable écrit commun
- Documenter de trois à cinq constats prouvés dans le temps imparti
- Rester dans le périmètre : ne rien sonder hors des adresses annoncées

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 8min |
| Contenu du chapitre | 25min |
| Techniques, pas à pas | 7min |
| LAB | 45min |
| Exercices | 5min |

### Contenu du chapitre (25min)

#### 1. Le réseau de la filiale (12min)

Le cas d’étude reproduit un petit réseau d’agence, avec ses habitudes et ses
imperfections. Toutes les valeurs sont dans le tableau ci-dessous : aucune
adresse n’est laissée à l’improvisation.

| Élément | Valeur |
|---------|--------|
| Réseau cible | 192.168.50.0/24, nommé « Filiale test » |
| Poste d’audit | Kali Linux — 192.168.50.10/24 |
| Serveur applicatif | 192.168.50.20, ports 23 et 80 attendus, 3389 fermé |
| Imprimante | 192.168.50.30, ports 80, 515 et 631 attendus |
| Passerelle | 192.168.50.254 |
| Point critique connu | Le port 3389 ne doit être joignable ni sur le serveur applicatif `192.168.50.20` ni sur l’imprimante `192.168.50.30`. C’est ce port qui est sondé, depuis le poste d’audit |

Le tableau distingue deux situations à ne pas confondre. Un port qui répond est
une exposition constatée. Un port qui ne répond pas ne prouve pas qu’il est
fermé : il peut être filtré par un pare-feu. Le chapitre 1, technique 3, apprend
à trancher entre les deux, et cette nuance conditionne la gravité du constat.

#### 2. La méthode intégrée, en quatre temps (13min)

La méthode de la journée 1 n’est pas réinventée : elle est enchaînée en une
boucle, sous contrainte de temps. Le tableau ci-dessous indique, pour chaque
temps, ce qu’on fait et avec quel outil.

| Temps | Action | Outil | Livrable partiel |
|-------|--------|-------|-------------------|
| Découvrir | Inventaire des hôtes du segment | `nmap -sn` | Liste d’adresses et de rôles |
| Identifier | État des ports de chaque hôte | `nmap -sT` | Tableau des ports ouverts |
| Mesurer | Chemin et qualité du lien | `traceroute`, `ping` | Latence et pertes |
| Conclure | Formalisation des constats | Trame du chapitre 2 | Tableau de constats |

Le rôle du binôme se répartit sur ces quatre temps. L’auditeur 1 pilote la
découverte et l’identification, l’auditeur 2 mesure et consigne les preuves.
À la fin de chaque temps, ils se rejoignent : l’un annonce ses adresses, l’autre
annonce ses mesures. C’est ce court échange qui évite les constats fondés sur
une information périmée.

### Les techniques, pas à pas

**Technique 1 — Découvrir avant d’identifier.** `nmap -sn` liste les machines
qui répondent, sans sonder les ports. C’est rapide, et cela évite de sonder les ports
de 254 adresses une par une. La découverte d’abord, le sondage
ensuite, sur les seules machines trouvées.

**Technique 2 — Énumérer les ports attendus, pas une plage entière.** Pour un
audit en temps contraint, on demande les ports que le scénario annonce, et on
ajoute 3389, souvent oublié parce qu’il est au-delà des 1000 premiers ports.
Une plage de 1 à 1000 ne le verrait jamais.

**Technique 3 — Contrôler le chemin et la qualité du lien.** `traceroute -n`
montre le chemin, `ping` donne la latence et les pertes. Un chemin qui passe
par une machine inattendue est une découverte ; une latence anormale est un
écart à documenter, pas un verdict.

**Technique 4 — Consigner au fur et à mesure.** Chaque constat est écrit avec
la commande, l’extrait et l’heure. Consigner à la fin, c’est reconstruire de
mémoire des sorties qu’on n’a plus, et c’est le moment où les preuves se
contestent.

### LAB — Audit complet d’un réseau de filiale (45min)

#### Prérequis

- Le fichier `~/audit/rapport-audit-modele.md`, produit par le LAB du chapitre 2, sert de trame : le tableau de constats de ce LAB reprend les cinq colonnes de la trame du chapitre 2, avec une colonne Hôte supplémentaire, puisque le réseau comporte plusieurs machines.
- Le fichier `~/audit/inventaire-brut.txt`, produit par le LAB du chapitre 1 du jour 1, fournit la méthode d’inventaire appliquée la veille, pour travailler vite et avec les mêmes critères.
- Les droits administrateur ne sont pas nécessaires : toutes les commandes de ce LAB fonctionnent en compte ordinaire.
- Le réseau 192.168.50.0/24 doit être fourni par le laboratoire. S’il ne l’est pas, le LAB se déroule sur 172.16.0.0/24, mais sans scénario : le tableau d’inventaire et les constats sont alors produits à partir des seuls services réellement trouvés, et les ports 515 et 631 du scénario sont simplement absents du constat.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Réseau cible | 192.168.50.0/24 |
| Poste d’audit | 192.168.50.10/24 |
| Durée | 45 minutes, en binôme |
| Rôles | Auditeur 1 : découverte et identification. Auditeur 2 : mesures et preuves écrites |
| Fichier produit | `~/audit/constats-filiale.md` |

#### Étapes du LAB

**Étape 1 — Découvrir le périmètre (10min)**

L’auditeur 1 exécute, l’auditeur 2 note. On cherche quatre adresses, et on leur
associe un rôle lu dans le scénario.

```bash
ip -br addr show
nmap -sn 192.168.50.0/24
```

> **Résultat attendu :** la découverte renvoie quatre hôtes : 192.168.50.10, le poste d’audit, 192.168.50.20, le serveur applicatif, 192.168.50.30, l’imprimante, et 192.168.50.254, la passerelle. Le tableau d’inventaire est rempli : adresse, rôle, état. Un hôte supplémentaire est consigné comme trouvaille. Ce tableau reprend la **méthode** de `~/audit/inventaire-brut.txt`, appliquée la veille sur le premier réseau : mêmes commandes, même enchaînement. Il n’en reprend pas les colonnes, ce fichier ne contenant que des sorties brutes (`ip -br addr show`, `ip route show`, `ip neigh show`, `ss`, `nmap -sn`). Les colonnes « rôle » et « état » viennent du scénario et des constats, pas du fichier de la veille.

**Étape 2 — Identifier les services (15min)**

On interroge explicitement les ports annoncés par le scénario, ports de gestion
sensibles compris. Aucun balayage de plage large n’est nécessaire.

```bash
nmap -sT -T4 -p 23,80,3389 192.168.50.20 2>/dev/null
nmap -sT -T4 -p 80,515,631,3389 192.168.50.30 2>/dev/null
```

> **Résultat attendu :** sur 192.168.50.20, les ports 23 et 80 sont `open` et 3389 est `closed`, conformément au scénario ; le telnet se retrouve donc sur les deux réseaux du stage, ce qui en fait le fil rouge. Sur 192.168.50.30, les ports 80, 515 et 631 sont `open` et 3389 est `closed`. Les ports 515 et 631 en clair constituent un constat potentiel : ils sont documentés avec la commande et l’extrait de sortie.

**Étape 3 — Vérifier le chemin et les échanges (10min)**

L’auditeur 2 mesure pendant que l’auditeur 1 vérifie la table d’inventaire. Le
chemin doit être direct, la latence doit rester faible sur un segment local.

```bash
traceroute -n 192.168.50.254 2>/dev/null || tracepath -n 192.168.50.254
ping -c 5 192.168.50.20
# Rédiger le fichier de constats que la restitution consommera
nano ~/audit/constats-filiale.md
```

> `traceroute` a besoin de droits pour émettre les sondes brutes, selon la version installée et la configuration du noyau. Sur un compte ordinaire, la commande peut donc échouer. C’est la raison du repli `|| tracepath`, qui n’exige pas ces droits : si les deux commandes échouent, on ne conclut rien sur le chemin et on note la limite de périmètre au lieu de laisser la ligne vide.

> **Résultat attendu :** le test de connectivité renvoie cinq réponses, aucune perte, et un temps de réponse moyen inférieur à 1 ms. Pour le chemin, la lecture dépend de la commande qui a répondu. `traceroute -n` écrit une ligne par saut, soit `1  192.168.50.254  0.070 ms  0.013 ms  0.009 ms` : trois sondes par saut, donc trois temps séparés par des espaces, ce qui est le comportement normal. `tracepath`, utilisé en repli quand `traceroute` est absent ou sans droits, écrit **trois à quatre lignes** : une ligne d’amorce `1?: [LOCALHOST]  pmtu 1500`, une ou deux lignes de saut `1:  192.168.50.254  0.048ms reached` selon le nombre de sondes qui reviennent, puis une ligne de reprise `Resume: pmtu 1500 hops 1 back 1`, qui donne la taille maximale de paquet découverte. Le nombre de lignes n’est donc pas le nombre de sauts, et il n’est pas stable d’une exécution à l’autre : ici, quatre lignes au plus pour un seul saut, et c’est le numéro de saut qui compte. Dans les deux cas, un seul saut confirme que `192.168.50.254` est la passerelle et qu’aucun équipement intermédiaire n’est traversé. Un temps supérieur à 1 ms ou une perte non nulle est consigné comme écart, avec son horodatage.

> Attention à la saisie : les deux commandes portent sur la passerelle `192.168.50.254`, puis sur le serveur `192.168.50.20`. Une adresse mal tapée, comme `192.16.0.20`, ne provoque pas un blocage : `nmap` répond `Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn` et se termine avec un code retour nul. Ce code ne signifie donc pas que la cible existe : seul le message indique qu’aucune réponse n’est arrivée. Lire la sortie, jamais le code de retour.

**Étape 4 — Rédiger les constats (10min)**

Les constats sont écrits dans la trame du chapitre 2, avec ses cinq colonnes et la colonne Hôte. L’ordre des colonnes est celui de la trame, la colonne Hôte venant après l’identifiant : `| ID | Hôte | Description | Preuve | Impact | Criticité |`. On ne renumérote pas les colonnes et on n’en ajoute aucune.
Le chapitre 3 audite un autre réseau, `192.168.50.0/24` : ses constats portent le
préfixe `F-` et non `C-`, afin qu’un identifiant ne désigne jamais deux constats
différents dans le même rapport. Les identifiants `F-01` à `F-05` sont réservés.

```markdown
# Constats — Filiale test (192.168.50.0/24)

| ID | Hôte | Description | Preuve | Impact | Criticité |
|----|------|-------------|--------|--------|-----------|
| F-02 | 192.168.50.20 | Port 23 ouvert, administration en clair | `nmap -sT -T4 -p 23,80,3389 192.168.50.20` | Identifiants d'administration interceptables sur le segment | Élevée |
| F-01 | 192.168.50.30 | Ports 515 et 631 ouverts en clair | `nmap -sT -T4 -p 80,515,631,3389 192.168.50.30` | Les travaux d'impression sont transmis en clair, identifiants compris | Moyenne |
| F-03 | 192.168.50.30 | Port 80 ouvert sans restriction constatée | `nmap -sT -T4 -p 80,515,631,3389 192.168.50.30` | Interface d'administration de l'imprimante accessible à tout le segment | Moyenne |
```

> Le modèle est volontairement trié par criticité décroissante : F-02, de criticité Élevée, vient en premier, puis F-01 et F-03, de criticité Moyenne. Les identifiants ne suivent pas l’ordre d’affichage, et c’est normal : ils identifient un constat de façon stable, alors que l’ordre du tableau porte un sens, celui de la gravité. Un rapport où les identifiants réapparaissent dans l’ordre de lecture se prête à la confusion, car le lecteur prend F-01 pour le plus grave.

> **Résultat attendu :** le fichier `~/audit/constats-filiale.md` contient les trois constats du modèle, un pour chacun des deux hôtes sondés (`192.168.50.20` et `192.168.50.30`), plus un que le scénario peut ajouter si les relevés le justifient, jusqu’à cinq. Chaque constat porte son identifiant, sa commande de preuve et sa criticité. F-02 reprend le même défaut que C-01 du chapitre 1, sur un autre réseau : le préfixe `F-` distingue les deux constats, qui doivent rester tous les deux au rapport.
>
> Le port 3389 fermé sur les deux hôtes ne donne lieu à aucun constat : c’est un point de contrôle qui fonctionne, et il se consigne comme tel dans la section Périmètre et méthode du rapport. De même, le chemin direct en un saut vers la passerelle n’est pas un constat. Un rapport d’audit qui ne contient que des constats donne une image biaisée du réseau : l’absence de constat sur un point de contrôle vérifié est une information, et elle se dit. Ce tableau alimente la section Constats du rapport final : au chapitre 4, le rapport du chapitre 2 est repris et complété.

##### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `nano` refuse d’ouvrir le fichier | Le dossier `~/audit` n’existe pas | Exécuter `mkdir -p ~/audit` |
| La découverte renvoie moins de quatre hôtes | Le réseau `192.168.50.0/24` n’est pas fourni par le laboratoire | Annoncer la situation, puis relancer la commande sur le réseau disponible, en notant le changement de périmètre |
| `traceroute` et `tracepath` renvoient tous deux une erreur | Les deux commandes exigent des droits que le compte ordinaire n’a pas | Noter « non vérifiable en compte ordinaire » et laisser la ligne de chemin explicitement vide |
| `traceroute` est absent et `tracepath` renvoie trois ou quatre lignes | C’est le comportement normal de `tracepath` : une ligne d’amorce, une ou deux lignes de saut, une ligne de reprise | Lire le numéro de saut, pas le nombre de lignes, qui varie d’une exécution à l’autre |
| `nmap` renvoie `Host seems down` sur une adresse qui existe | Adresse mal saisie, ou ICMP filtré | Relire l’adresse, puis relancer avec `-Pn` |
| Le tableau ne contient que la ligne F-01 | Les deux autres lignes du modèle n’ont pas été saisies | Reprendre le modèle complet : trois constats sont attendus avant tout ajout |
| Un constat porte le préfixe `C-` au lieu de `F-` | La trame du chapitre 2 a été recopiée sans changer les identifiants | Remplacer tous les `C-` par `F-` : un identifiant ne doit jamais désigner deux constats du même rapport |

### Ce que ce LAB produit

`~/audit/constats-filiale.md` : le tableau de constats du réseau de la
filiale, prêt à être versé dans le rapport. Le chapitre 4 s’en sert comme
support de restitution, et chaque constat conserve l’identifiant `F-` qui lui a été
attribué au chapitre 3.

### Exercices — Chapitre 3

**Exercice 3.1 — Plan d’audit express**

1. Lister les quatre temps de la méthode, en indiquant l’outil associé à chacun.
2. Justifier le choix de `-sT` dans un environnement où l’on ne dispose pas des droits administrateur.

**Exercice 3.2 — Traçabilité**

1. Expliquer pourquoi chaque constat doit porter une date, une heure et un outil.
2. Proposer la preuve minimale à joindre pour un port exposé.

---

## Chapitre 4 — M7 (suite) : Restitution et débrief (15h30–17h00)

### Préambule — ce qui n’est pas dit n’est pas fait

Un audit qui reste dans un tiroir n’a produit aucun effet. La restitution est
l’étape où l’auditeur rend la main : il expose ce qu’il a trouvé, à qui il
propose de le traiter, et dans quel délai. C’est le seul moment où le rapport
rencontre les personnes qui peuvent agir.

L’image utile est celle d’une remise de dossier. Le lecteur a dix minutes, il
a beaucoup de sujets en tête, et il ne lira pas les annexes. Ce qui doit passer en
premier est donc en petit nombre : le périmètre, les trois constats les plus graves, ce
qu’on propose, et la date de la première action.

Le chapitre se termine par un débrief collectif, qui transforme l’expérience
du jour en méthode pour le prochain audit.

### Le vocabulaire en clair

| Terme | En clair | Dans ce chapitre |
|-------|----------|------------------|
| Restitution | La présentation orale des constats devant le groupe | Quatre minutes par binôme |
| Synthèse orale | Les trois messages à retenir, dans l’ordre d’importance | Trois constats priorisés |
| Support | La page unique qui accompagne la présentation | Contexte, constats, preuves, actions |
| Débrief | L’échange collectif qui fait le bilan de la journée | Quatre thèmes, notés |
| Axe d’amélioration | Un changement de méthode décidé après le débrief | Deux axes retenus |
| Chronomètre | Le minuteur qui garantit le respect du temps imparti | Un par binôme |

### Objectifs du chapitre

- Présenter des constats en quatre minutes, sans perdre la démonstration
- Employer un support unique, adapté à un lecteur pressé
- Animer un débrief collectif sur la méthode, les outils et l’organisation
- Décider de deux axes d’amélioration concrets pour le prochain audit

### Répartition du temps

| Partie | Durée |
|--------|-------|
| Préambule, vocabulaire et objectifs | 5min |
| Contenu du chapitre | 15min |
| Techniques, pas à pas | 5min |
| LAB | 60min |
| Exercices | 5min |

### Contenu du chapitre (15min)

#### 1. Les règles d’une restitution efficace (8min)

Une restitution qui dépasse les quatre minutes est interrompue, et l’auditeur perd la
main sur la suite. Les quatre minutes se préparent avant la présentation : la structure du message et le support sont
prêts. Pendant la présentation, le ton reste factuel et le chronomètre est tenu par le binôme suivant.

| Point | Règle | Raison |
|-------|-------|--------|
| Temps | Quatre minutes par binôme, questions comprises, trois constats au maximum | Le lecteur est pressé |
| Structure | Contexte, constat, preuve, recommandation | Le message se suit d’un bout à l’autre |
| Support | Une page unique : contexte, constats, preuves, actions | Le document reste lisible |
| Ton | Factuel, sans jargon inutile | La crédibilité repose sur la preuve |

#### 2. Le débrief et les axes de progrès (7min)

Le débrief n’est pas une discussion libre : chaque binôme apporte une réponse
aux mêmes quatre questions. Cette symétrie évite que la discussion ne porte
que sur les derniers constats produits.

| Thème | Question posée au groupe |
|-------|---------------------------|
| Méthode | Quelles étapes ont été utiles, lesquelles ont manqué, quel temps ont-elles pris ? |
| Outils | Lesquels se sont révélés fiables, lesquels étaient limités, quoi ajouter ? |
| Organisation | La répartition des rôles a-t-elle fonctionné, l’information a-t-elle circulé ? |
| Prochain audit | À quelle fréquence, sur quel périmètre, avec quelles améliorations ? |

### Les techniques, pas à pas

**Technique 1 — Ramener à trois constats.** Six constats ne se retiennent pas en
quatre minutes. On garde les plus critiques, on annonce que les autres figurent
dans le rapport, et on ne défend que trois messages.

**Technique 2 — Annoncer la preuve avant l’interprétation.** Le fil de la
présentation est toujours le même : ce que l’on a observé, la commande qui le
prouve, puis ce que l’on en conclut. L’inversion — conclure puis chercher la
preuve — est la plus fréquente des présentations ratées.

**Technique 3 — Terminer par une demande datée.** Une restitution se termine
par une demande précise : qui fait quoi, et pour quand. Sans demande datée,
l’auditoire quitte la salle avec des intentions, pas avec des actions.

### LAB — Restitution orale et débrief collectif (60min)

#### Prérequis

- Le fichier `~/audit/constats-filiale.md`, produit par le LAB du chapitre 3, fournit la matière de la présentation : trois constats au moins, chacun avec sa preuve.
- Le fichier `~/audit/rapport-audit-modele.md`, produit par le LAB du chapitre 2, fournit la structure du support de restitution.
- Le fichier `~/audit/constats-securite.md`, produit par le LAB du chapitre 1, sert de repli si le binôme n’a pas terminé le chapitre 3 ; ses constats ne sont pas présentés, ils figurent déjà dans le rapport.
- Le fichier `~/audit/audit_capture.pcap`, produit par le LAB du chapitre 3 du jour 1, est repris en annexe du dossier d’audit : il prouve l’existence des flux et n’a pas vocation à produire un constat.

#### Environnement

| Élément | Valeur |
|---------|--------|
| Salle | Présentiel, six binômes |
| Durée | 60 minutes : 15 minutes de préparation du support, 30 minutes de présentations, 15 minutes de débrief collectif |
| Support | Une page par binôme, tirée du rapport |
| Fichier produit | `~/audit/support-binome.md` par binôme, puis `~/audit/debrief-collectif.md` |

#### Étapes du LAB

**Étape 1 — Préparer le support de restitution (15min)**

Le support tient sur une page et suit l’ordre du chapitre 2 : contexte,
constats, preuves, recommandations, prochaines étapes.

Le support est **tiré du rapport**, qui a été rédigé au chapitre 2 sur le réseau
`172.16.0.0/24` et qui ne contient donc aucun constat de la filiale
`192.168.50.0/24`. Avant de rédiger le support, on met donc le rapport à jour : on
y complète la section « Périmètre et méthode » déjà présente dans la trame, pour
qu’elle cite aussi le second réseau, puis on y
reprend les constats `F-` du chapitre 3. C’est cette mise à jour qui rend le
support traçable jusqu’au rapport.

```bash
# 1. Reporter dans le rapport les constats de la filiale
nano ~/audit/rapport-audit-modele.md
# 2. Écrire le support à partir du rapport mis à jour
nano ~/audit/support-binome.md
# 3. Vérifier que le support tient sur une page
wc -l ~/audit/support-binome.md
```

```markdown
# Restitution — binôme

1. Contexte : réseau 192.168.50.0/24 de la filiale, périmètre audité, outils utilisés
2. Constats majeurs : F-02, F-01, F-03, dans l'ordre de criticité décroissante
3. Preuves : commande et extrait pour chacun des trois constats
4. Recommandations : deux actions, chacune rattachée à l'identifiant du constat traité
5. Prochaines étapes : responsable et échéance de la première action
```

> **Résultat attendu :** `nano` crée le support du binôme, que l’on remplit avec le modèle ci-dessus ; `wc -l` renvoie une valeur **inférieure ou égale à 35**. Le modèle ci-dessus tient en sept lignes : chacune peut s’allonger d’une ou deux lignes pour porter un extrait de preuve, ce qui laisse le support largement sous le seuil de 35 tout en restant lisible en une page. Une valeur supérieure signale que les preuves sont trop détaillées : on ne recopie dans le support que la ligne utile de chaque sortie de commande. Le support présente le contexte, les trois constats triés par criticité, une preuve par constat, les recommandations issues du rapport et une première action datée. Les constats `C-` du premier réseau ne sont pas présentés ici : ils figurent dans le rapport, dont le support constitue un extrait, et le fichier `constats-securite.md` sert de repli si le binôme n’a pas terminé le chapitre 3. Le fichier `~/audit/debrief-collectif.md` est réservé à l’étape 3 et se remplit sous la conduite du formateur.

**Étape 2 — Présenter et chronométrer (30min)**

Chaque binôme présente quatre minutes, questions du formateur comprises. Un binôme tient le chronomètre et signale
la dernière minute. Les autres notent un constat réutilisable de chaque
présentation, en une phrase.

> **Résultat attendu :** six présentations de quatre minutes chronométrées, questions du formateur comprises, le passage de main entre binômes laissant six minutes au total, et pour chaque auditeur en écoute un constat réutilisable noté. Le chronomètre est tenu par le binôme suivant, qui l’annonce à l’oral.
>
> **Si le stage se déroule en solo**, cette étape se conduit seule : préparer le support demande les quinze minutes de l’étape 1, puis on le présente pendant quatre minutes devant le groupe, en annonçant soi-même le passage de la dernière minute. Les six minutes de passage de main se réduisent alors à une transition immédiate, et le temps restant de l’étape est consacré aux questions du groupe. Le total de l’étape reste de trente minutes, et le support produit est identique.

**Étape 3 — Conduire le débrief collectif (15min)**

Le débrief suit les quatre thèmes du contenu du chapitre. Le formateur note les
réponses au fil des questions, sans débat sur la technique d’un cas
particulier.

```bash
# Consigner les quatre thèmes et les deux axes d'amélioration
nano ~/audit/debrief-collectif.md
```

```markdown
# Débrief collectif

- Méthode : étapes efficaces, étapes manquantes, temps constaté
- Outils : outils fiables, limites rencontrées, compléments nécessaires
- Organisation : répartition des rôles, circulation de l'information
- Prochain audit : fréquence, périmètre, améliorations retenues
```

> **Résultat attendu :** le fichier `~/audit/debrief-collectif.md` est renseigné sur les quatre thèmes, avec au moins un point fort par thème, puis le groupe retient deux axes d’amélioration, inscrits dans le fichier. Le fichier est conservé comme trace de la méthode retenue.

##### Dépannage

| Problème | Cause la plus probable | Correction |
|-----------|----------------------|------------|
| `nano` refuse d’ouvrir le fichier | Le dossier `~/audit` n’existe pas | Exécuter `mkdir -p ~/audit` |
| Le rapport ne contient aucun constat `F-` | Le rapport n’a pas été mis à jour avant la rédaction du support | Reprendre l’étape 1 : reporter les constats `F-` dans `~/audit/rapport-audit-modele.md` avant d’écrire le support |
| Le support cite des constats absents du rapport | Le support a été rédigé de mémoire, ou avant la mise à jour du rapport | Le refaire à partir du rapport mis à jour, jamais de mémoire : c’est la règle du chapitre 2 |
| `wc -l` renvoie une valeur supérieure à 35 | Les preuves ont été recopiées en entier | Ne recopier que la ligne utile de chaque sortie de commande |
| Le support présente les constats dans un ordre qui n’est pas celui de la criticité | Le tri a été omis sous l’effet du temps | Trier par criticité décroissante, et annoncer ce tri à l’oral : c’est le message principal de la présentation |
| La présentation dépasse les quatre minutes | Le support a été lu intégralement, au lieu d’être présenté | Ne lire que les cinq points, laisser les preuves en annexe du support |
| Le débrief reste vide à la fin de l’étape | Le fichier n’a pas été enregistré avant de quitter l’éditeur | Enregistrer avec `Ctrl + O` puis `Entrée`, et relire avec `cat ~/audit/debrief-collectif.md` |

### Ce que ce LAB produit

`~/audit/support-binome.md` : la page unique de chaque binôme, tirée du rapport et
non de la mémoire, qui sert de trame à la présentation orale.
`~/audit/debrief-collectif.md` : la trace écrite du débrief, avec les deux axes
d’amélioration retenus par le groupe. Ces axes alimentent la documentation
méthodologique du laboratoire et la préparation du prochain audit.

### Exercices — Chapitre 4

**Exercice 4.1 — Réduire à trois messages**

1. Ramener six constats à trois messages de trente secondes chacun, en conservant l’ordre de criticité.
2. Justifier le choix des trois messages retenus et l’ordre dans lequel ils sont présentés.

**Exercice 4.2 — Plan de progrès**

1. Proposer deux améliorations de méthode applicables dès le prochain audit.
2. Écrire la trame de restitution réutilisable, en trois rubriques.

---

## Synthèse

### Couverture Jour 2

| Chapitre | Module | Thème |
|----------|--------|-------|
| C1 | M5 | Sécurité réseau |
| C2 | M6 | Rapport d’audit et recommandations |
| C3 | M7 | Audit complet d’un cas d’étude |
| C4 | M7 (suite) | Restitution et débrief |

### Mémo — commandes essentielles

> Commandes du laboratoire, à relire avant exécution et à adapter aux adresses du jour.

```text
# Sécurité réseau
nmap -sT -T4 -p 22,23,80,443,3389 172.16.0.20   # Surface d'exposition
iptables -L -n -v --line-numbers                 # Règles iptables, dans leur ordre (compte privilégié)
nft list ruleset                                 # Règles nftables (compte privilégié)
ss -tulnp                                        # Services en écoute

# Audit complet
nmap -sn 192.168.50.0/24                         # Découverte des hôtes
nmap -sT -T4 -p 23,80,3389 192.168.50.20        # Ports du serveur : 23 attendu ouvert, 3389 attendu fermé
nmap -sT -T4 -p 80,515,631,3389 192.168.50.30    # Ports de l'imprimante
traceroute -n 192.168.50.254 || tracepath -n 192.168.50.254   # Chemin vers la passerelle
ping -c 5 192.168.50.20                          # Latence et pertes

# Rapport et restitution
grep -c '^|' ~/audit/rapport-audit-modele.md     # Lignes de tableau du rapport
wc -l ~/audit/support-binome.md                  # Longueur du support de restitution
```

---

## Ressources supplémentaires

### Outils utilisés

| Outil | Version indicative | Usage |
|-------|--------------------|-------|
| nmap | 7.x ou plus récent | Découverte, ports, surface d’exposition |
| iproute2 | 5.x ou plus récent | Interfaces, routage |
| ss | iproute2 | Services en écoute |
| iputils-ping | version affichée par `ping -V` | `ping` : latence et pertes. Depuis 2022, la commande affiche une date et non un numéro : `ping -V` renvoie par exemple `ping from iputils 20250605` |
| iptables | selon distribution | Lecture des règles de filtrage |
| nftables | selon distribution | Lecture des règles de filtrage |
| traceroute | 2.1 ou plus récent | `traceroute` : chemin réseau |
| iputils-tracepath | 20211215 ou plus récent | `tracepath`, repli quand `traceroute` est absent |
| net-tools | 2.10 ou plus récent | `netstat`, repli quand `ss` est absent |
| nano | 2.10 ou plus récent | Éditeur de texte, utilisé pour écrire chaque livrable. Enregistrer avec `Ctrl + O` puis `Entrée`, quitter avec `Ctrl + X` |


### Références

| Document | Lien |
|----------|------|
| OWASP Web Security Testing Guide (WSTG) | https://owasp.org/www-project-web-security-testing-guide/ |
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks/ |
| RFC 2544 — Méthodologie de mesure | https://www.rfc-editor.org/rfc/rfc2544.html |
| ISO/IEC 27002 — Contrôles de sécurité | https://www.iso.org/standard/75652.html |
| ANSSI — Recommandations de sécurité | https://www.ssi.gouv.fr/ |
| Guide de référence nmap | https://nmap.org/book/man.html |
