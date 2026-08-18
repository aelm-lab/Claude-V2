# PILOTAGE.md — Cadence, gate et hygiène documentaire

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** comment on ouvre, classe et clôt un **sprint** et une **session**, quelle porte
> de qualité s'applique à quoi, et comment la documentation reste saine.
> **Ce qui n'est PAS ici :** l'état courant et le Kanban (→ `STATE.md`), les règles de code
> et de test destinées à Gemini (→ `AGENTS.md`), les décisions passées (→ `DECISIONS.md`).
>
> **Remplace et fusionne :** `METHODOLOGY.md`, `PROCESS_TIERS.md`, `PRODUCT_NORTHSTAR.md`
> (supprimés en SE-049).

---

## 1. Sprint ≠ Session — les deux compteurs

C'est la distinction fondatrice du pilotage. Elle a été confondue jusqu'en SE-049 : les
numéros `SXX` désignaient en réalité des **sessions**, ce qui faisait apparaître des sprints
sans livraison (ex. « S38 — aucun code livré », qui était une session d'investigation).

| | **Sprint** | **Session** |
|---|---|---|
| **Identifiant** | `S38`, `S39`… (suite historique conservée) | `SE-049`, `SE-050`… |
| **Défini par** | un **Sprint Goal produit** | une **conversation Claude Chat** |
| **Se ferme quand** | le Goal est atteint et la Sprint Outcome Gate est répondue | la capacité de conversation est atteinte, ou arrêt volontaire |
| **Produit** | un bump de version + une entrée `STATE.md §Versions` | un **handoff** `SE-00X → SE-00Y` |
| **Tracé dans** | `STATE.md §Versions` et `§Sprint Outcome Gate` | `STATE.md §Sessions` (5 dernières, rotatif) |

**Relations possibles**
- Cas normal : **1 sprint = N sessions**. Un sprint qui ne tient pas dans une conversation
  n'est pas un problème — il produit plusieurs handoffs.
- Cas rare : **1 session = 2 sprints**, si un Goal est atteint tôt et qu'on en ouvre un autre.
  Claude signale « Goal atteint, on ouvre le sprint suivant ? » plutôt que d'attendre le handoff.
- Cas hors sprint : une session peut être un **chantier** (documentation, audit, investigation)
  sans Sprint Goal. Elle est alors marquée `hors sprint` et ne produit **aucun bump de version**.

**Conséquence pratique :** une conversation qui meurt sur la capacité est un non-événement
produit. Elle ne consomme pas de numéro de sprint et n'apparaît pas dans l'historique de
version.

### Schéma de version

Le schéma d'origine `0.<sprint>.0` est **abandonné de fait depuis S29** : le retard de bumps
accumulé entre S29 et S35 a été résorbé en S36, désynchronisant durablement version et sprint
(S36 → v0.30.0, S37 → v0.31.0).

**Règle actuelle :** `0.<n>.0`, incrémenté à chaque clôture de sprint **ayant livré du code**.
Patch `0.<n>.1` pour un correctif hors sprint. Un sprint sans livraison ne bumpe pas.
La correspondance sprint ↔ version vit dans `STATE.md §Versions`, elle ne se déduit plus.

---

## 2. Cérémonies

| Cérémonie | Quand | Contenu |
|---|---|---|
| **Ouverture de session** | Début de chaque conversation | Lire la Knowledge → confirmer en 1 phrase + signaler si `STATE.md` semble périmé → afficher le Kanban → **remesurer les faits externes** (§6) → questions de clarification → présenter le PLAN → attendre le `go` du PO |
| **Sprint Planning** | Ouverture de sprint | Définir le **Sprint Goal** + choisir les US (déjà raffinées au sprint précédent) + vérifier le **budget dette** (§5) avant de composer le sprint |
| **Backlog Refinement** | En continu | À chaque US close, Claude pose **1 question de refinement** sur une US du sprint suivant (décision produit en attente OU lecture R3 à prévoir). But : la 1ʳᵉ US du sprint suivant démarre déjà raffinée |
| **TNR** (non-régression) | Avant chaque merge | Porte selon la classe de risque (§3). Preuves Gemini **irrecevables** — seule la machine PO fait foi (R1) |
| **Sprint Outcome Gate** | Clôture de sprint | 1 ligne : gain ressenti / gain de fiabilité / dette justifiée (§5) |
| **Release** | Clôture de sprint avec livraison | Bump de version + entrée `STATE.md §Versions`. Le déploiement est déjà continu (Cloud Run) → la release est un **repère versionné**, pas une mise en production |
| **Clôture de session** | Fin de chaque conversation | Handoff + double checklist (§7) |

---

## 3. Classe de risque et porte de qualité

**Principe :** garder la rigueur là où elle a attrapé de vrais bugs, l'alléger là où elle ne
fait que taxer le débit. Le gabarit « boot/persistance » ne doit pas s'appliquer tel quel à
une US CSS de 44 px. La gate suit le risque, pas un réflexe uniforme.

Le détail des règles `R1→R7`, `R-CODE`, `R-SCOPE` vit dans `AGENTS.md`. Ce document dit
**lesquelles s'appliquent à quoi**.

| Classe | Périmètre | Gate exigée | Justification |
|---|---|---|---|
| **🔴 CRITIQUE** | EPIC 8 (Boot / Persistance / Sync), EPIC 10 moteur (`recEngine`, `useRecommendations`) + toute US touchant l'orchestration, le store ou le câblage de composables | **Gate complète** : R1 (triple preuve) + R2 (test runtime) + R4 (E2E DOM si écran) + R6 (audit live avant clôture d'epic) + lecture zéro-confiance du code brut | C'est là que sont nés les 4 bugs runtime de s6 et le P0 de s16. La rigueur est *méritée* |
| **🟠 STANDARD** | `[FEATURE]` / `[UX]` sur un composant existant, `[REC]` hors moteur | R1 + R4 (E2E ciblé) | Touche l'écran → un clic réel doit valider |
| **🟢 LÉGÈRE** | `[CSS]` / `[A11Y]` pur, `[UX-copy]`, libellés, styles dans `style.css`, suppression de code mort prouvée | **R1 allégée** : type-check + `test:run` + build, les 3 verts, 1 run groupé. Pas d'E2E imposé — **sauf si un `v-if` interactif est ajouté → R4-bis obligatoire** | Une couleur ou un padding ne casse pas l'orchestration runtime. Le coût E2E n'a pas de ROI ici |

**Cas particulier — US spec-only** (`*.spec.ts` uniquement, 0 logique) : 1 run groupé tous
les ~6 tests suffit.

### Règles de classement (sans ambiguïté)

1. **Une US hérite de la classe la plus haute qu'elle touche.** Une US « CSS » qui modifie
   aussi `usePersistence` est 🔴, pas 🟢.
2. **En cas de doute → classe supérieure.** Le doute coûte moins cher que le bug silencieux.
3. **Le tag `[DETTE]` ne baisse jamais la classe.** Une dette sur le boot reste 🔴.
4. **Garde-fou :** si une US 🟢 régresse un comportement runtime, elle remonte définitivement
   en 🟠 par antipattern logué. On apprend la frontière par l'usage, pas par la peur a priori.

### Ce qui ne change JAMAIS, quelle que soit la classe

- Zéro `any` (R-CODE-1).
- Périmètre = fichiers listés (R-SCOPE-1) ; démarrage de session = état des fichiers modifiés.
- Sortie de commande = terminal brut, jamais de paraphrase.
- Contenu intégral des fichiers livrés.
- **Impact utilisateur + recommandation** sur chaque US (PO non technique).
- `npm run test:run` reste vert. Seule la partie **E2E** se module.
- **L'auteur du test ≠ l'auteur du code** (R7). Aucune exception, même pour un test visuel
  jugé « simple ».

### Tag des US

`US-XXX [EPIC][SECTION][TYPE] Titre` + **classe de risque** en tête de l'US.
- **[EPIC]** = le OÙ (page / surface / système) — voir `EPICS.md`.
- **[SECTION]** = sous-zone (`WEEK`, `FOR-YOU`, `BOOT`, `PERSIST`, `SYNC`, `ENGINE`…).
- **[TYPE]** = nature transverse : `[FEATURE][UX][CSS][TEST][PERF][DETTE][A11Y][CI]`.

→ Vue par page (priorisation PO) **et** vue par type (filtrer toute la dette CSS) simultanées.

---

## 4. North Star produit

> **« Un nouvel utilisateur suit son premier anime sur son calendrier en moins de 2 minutes,
> et revient ≥ 2 jours la première semaine. »**

Tout sprint se demande : *est-ce que ce qu'on livre rapproche ou éloigne de cette phrase ?*
Si la réponse est « ni l'un ni l'autre » (dette pure), le sprint doit le déclarer explicitement.

**Pourquoi cette section existe :** jusqu'à sa création, la seule métrique au mur était le
vert CI. Le vert prouve la mécanique, pas la valeur.

| Métrique | Question à laquelle elle répond | Cible directionnelle |
|---|---|---|
| **TTFA** — Time To First Anime | Combien de temps / de clics pour suivre son 1ᵉʳ anime depuis une session vierge ? | ↓ (< 2 min, < 5 clics) |
| **Adds / semaine** | L'app crée-t-elle l'habitude d'ajouter des séries ? | ↑ |
| **Jours-retour S1** | Sur la 1ʳᵉ semaine, combien de jours distincts l'utilisateur revient ? | ↑ (≥ 2/7) |

Leurs **valeurs** vivent dans `STATE.md §Métriques produit`, à côté du compteur de tests,
jamais à la place. Elles n'ont de sens qu'à partir du moment où un sprint produit les rend
mesurables.

---

## 5. Sprint Outcome Gate & budget dette

**Un sprint ne se clôt pas sans répondre, en une ligne, à :**

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait
> pas avant ce sprint ? »*

Trois réponses possibles :
1. **Gain ressenti** (ex. « il peut s'onboarder en 90 s ») → ✅ sprint produit, nominal.
2. **Gain de fiabilité visible** (ex. « il ne perd plus ses données en silence ») → ✅ acceptable.
3. **Aucun gain visible — dette / audit** → ⚠️ **autorisé en exception, pas en routine.**
   Doit être justifié (« filet avant correctif », « risque silencieux ») ET suivi d'un sprint produit.

**Garde-fou anti-dérive :** pas plus d'**1 sprint « aucun gain visible » consécutif**. Deux
d'affilée = signal d'alarme, on bascule sur un levier produit.

**Budget dette — plafond :** ≤ **1 US de dette pure pour 1 US à gain visible** par sprint.
La dette invisible est réelle et doit être traitée, mais sous plafond, jamais en flux libre.
Un audit qui génère plusieurs US de dette s'**étale** sur plusieurs sprints ; il ne se déverse
pas en un seul.

Cette gate s'ajoute à la porte technique (§3) et aux règles de clôture de session (§7).
Elle n'en remplace aucune.

---

## 6. Faits externes — remesure obligatoire

**Règle (AP-PROCESS, gravée S38) : tout fait externe inscrit en standby est remesuré à
l'ouverture de chaque session, avec la requête EXACTE émise par le code — jamais une version
simplifiée.** Un endpoint testé avec d'autres paramètres est un autre endpoint.

Origine de la règle : « Jikan est en panne » a été porté **5 sprints** (S33 → S38) sur la foi
d'un unique curl jamais rejoué, gelant 2 items du backlog et se propageant dans 5 documents.
Aucune règle de qualité de code n'a protégé contre ça — c'était un défaut de **fraîcheur de
fait**, pas de code.

Corollaire : **toute US sortant du backlog démarre par un grep du fichier concerné avant
rédaction.** Origine : en S36, US-140, US-127 et US-SEARCH-3 étaient planifiées « à faire »
alors que les trois étaient en production.

L'état courant des faits externes vit dans `STATE.md §Faits externes`, avec la date de la
dernière mesure.

---

## 7. Clôture de session & handoff

**Mise à jour minimale obligatoire.** Aucune session ne se termine sans qu'au moins
`STATE.md` soit mis à jour (Kanban + sessions + toute US mergée ou débloquée). Si la session
a touché une règle de gouvernance (nouvel antipattern, décision d'architecture, changement de
clé ou de contrat), le document concerné (`AGENTS.md` / `ANTIPATTERNS.md` / `DECISIONS.md` /
`TYPES_CONTRACT.md`) est mis à jour dans la **même** session — jamais reporté. Un handoff qui
ne touche aucun document est un signal d'alerte, pas un raccourci.
**Cadence documentaire (DEC-146).** À chaque clôture de session, sans exception :
1. `STATE.md` — **régénéré intégralement**, jamais patché.
2. `HISTORIQUE.md` — **+1 ligne** de session (+1 ligne de version si le sprint se clôt).
3. Tout document de gouvernance touché par la session (`DECISIONS`, `TYPES_CONTRACT`,
   `ANTIPATTERNS`, `AUDIT`, `AGENTS`) — **patché dans la même session, jamais reporté.**

**Interdit : différer un DEC au handoff.** Un handoff est une source secondaire faillible.
DEC-137→145 y ont vécu deux sessions, et le même handoff affirmait par ailleurs une dette
`AGENTS.md` déjà soldée depuis trois sessions.

**Vérification de dette par preuve, pas par recopie.** Toute dette de gouvernance héritée d'un
handoff se vérifie contre le dépôt (`git log`, `findstr`) **avant** d'être reportée une fois
de plus.
**Isolation du déploiement `AGENTS.md` (`AP-PROCESS-3`).** Le redéploiement d'`AGENTS.md` à la
racine de `A-Anime` se **commite seul**, en clôture de session, **avant** l'envoi de toute
nouvelle US. Un fichier de gouvernance qui traîne non commité dans l'arbre de travail sera
embarqué par le prochain commit de Gemini et produira un faux signal `R-SCOPE-1` contre lui.
Corollaire : **`git status` vide est condition d'ouverture d'une livraison**, pas seulement de
sa fermeture. Et un fichier inattendu dans un diffstat se **lit** avant d'être imputé.

**Double checklist obligatoire.** Chaque handoff répond explicitement à deux questions, même
si la réponse est « rien à changer » :
1. **Côté Gemini** — `AGENTS.md` a-t-il changé cette session ? Si oui, a-t-il été **déployé à
   la racine de `A-Anime`** ? Une désynchronisation a-t-elle été détectée ?
2. **Côté State** — `STATE.md` reflète-t-il la session (Kanban, sessions, métriques, faits
   externes) ?

**Nommage du handoff :** `HANDOFF SE-0XX → SE-0YY (sprint SXX)`. Il porte toujours le sprint
dans lequel il se trouve, pour qu'une reprise sache si elle continue un Goal ou en ouvre un.

---

## 8. Hygiène documentaire

Ces règles existent parce que le corpus avait atteint 172 Ko dont ~40 % de process sur le
process, avec la règle R1 dupliquée dans 8 fichiers sur 13 et 3 versions contradictoires du
même fait. Le coût réel n'est pas le stockage : c'est **l'éviction**. Chaque doublon consomme
un extrait de recherche qui aurait pu porter la bonne information.

| # | Règle | Quand | Qui tranche |
|---|---|---|---|
| **H1** | **Un chiffre n'existe qu'une fois.** Tout compteur (tests, E2E, build, hash, version) vit dans `STATE.md`. Ailleurs : un renvoi, jamais une valeur. *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à `STATE.md`* | Chaque session | Claude applique |
| **H2** | **Un backlog n'existe qu'une fois** — `STATE.md`. `EPICS.md` ne porte que du livré | Chaque session | Claude |
| **H3** | **Une règle n'existe qu'une fois** — `AGENTS.md` (code, livraison, test) ou `PILOTAGE.md` (cadence, gate). Les autres documents y renvoient | À chaque nouvelle règle | PO |
| **H4** | **Un antipattern s'écrit à la 2ᵉ occurrence, jamais à la 1ʳᵉ.** Une erreur unique va dans le handoff, pas dans le corpus permanent | Review | Claude propose, PO valide |
| **H5** | **Une décision périmée n'est jamais supprimée : elle est marquée** `⛔ SUPERSEDED PAR DEC-xxx` sur sa propre ligne. Les renvois existants restent valides | À chaque DEC contredisant un DEC | Claude |
| **H6** | **Archiver = `git rm`.** L'historique git est l'archive. **Jamais de dossier d'archive dans un dépôt indexé** : la Knowledge indexe tout le dépôt, un `OLD/` continue de remonter en recherche. Une ligne dans `DECISIONS.md` mentionne le commit | À la suppression | PO |
| **H7** | **Plafond : 250 lignes par document de Knowledge.** Le critère réel est la **dispersion thématique** : au-delà de 250 lignes, un document mêle en général plusieurs intentions et renvoie des extraits sans rapport avec la question. Deux exceptions nommées, parce qu'elles sont **mono-intention à titres stables** : `AGENTS.md` (lecture linéaire par Gemini, plafond 350) et `TYPES_CONTRACT.md` (document de référence pur, plafond 350). Aucune autre exception sans validation PO | Continu | Claude alerte |
| **H8** | **Titres thématiques, jamais chronologiques.** Un document organisé par session produit des extraits qui se contredisent d'une section à l'autre | À l'écriture | Claude |
| **H9** | **Purge tous les 5 sprints.** Tout document dont l'état de référence a plus de 3 sprints de retard est relu ou marqué périmé | S40, S45… | PO déclenche |
**Plafond de `STATE.md` : 200 lignes** (DEC-146). Il ne porte plus que le présent — position,
métriques, Kanban, faits externes, trous. Sessions et versions vivent dans `HISTORIQUE.md`,
qui n'a pas de plafond (append-only, purge H9 par condensation, jamais par suppression).
  document qui indexe l'ensemble du projet (versions, sessions, métriques, backlog, trous)
  au lieu de traiter un sujet unique. Au-delà de 350, sortir une section entière vers un
  doc satellite hors ordre de lecture, sur le modèle d'`AUDIT.md` (SE-051).
### Carte des documents

| Doc | Rôle | Lu par |
|---|---|---|
| `CLAUDE.md` | Carte d'entrée : ce qu'est l'app, qui code, où trouver le reste | Claude |
| `STATE.md` | État mesuré : sprint/session, versions, compteurs, Kanban, faits externes, trous | Claude |
| `PILOTAGE.md` | Ce fichier : cadence, gate, North Star, hygiène | Claude |
| `ARCHITECTURE_TECHNIQUE.md` | Couches, modules, boot, réseau, registre des clés | Claude |
| `ARCHITECTURE_FONCTIONNELLE.md` | Parcours utilisateur + pont fonctionnel ↔ technique | Claude |
| `TYPES_CONTRACT.md` | Contrat TypeScript — seule source d'interfaces | Claude |
| `EPICS.md` | Taxonomie des EPICs + acquis fonctionnel durable | Claude |
| `DECISIONS.md` | Pourquoi un choix irréversible a été fait | Claude |
| `ANTIPATTERNS.md` | Pièges **répétés**, classés par thème | Claude |
| `AGENTS.md` | Canon des règles Gemini — écrit ici, **déployé à la racine de `A-Anime`** | Gemini (+ Claude) |
| `HISTORIQUE.md` | Journal append-only des sessions closes et des versions livrées. **Hors ordre de lecture** — ouvert à la demande | Claude |

Les instructions personnalisées du projet ne vivent pas dans le dépôt : elles se collent dans
la configuration du projet Claude Chat, comme un handoff.
