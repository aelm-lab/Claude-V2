# PILOTAGE.md — Cadence, gate et hygiène documentaire

> **Rôle :** comment on ouvre, classe et clôt un **sprint** et une **session**, quelle porte de
> qualité s'applique à quoi, et comment la documentation reste saine.
> **Pas ici :** l'état courant (→ `STATE.md`) · les règles de code et de test (→ `AGENTS.md`) ·
> les décisions (→ `DECISIONS.md`).

**Format de réponse au PO (`DEC-188`).** Quatre blocs, pas plus : 📍 où on en est · 👤 impact
utilisateur · 🔜 prochaine US anticipée · ⏭️ actions numérotées. Le raisonnement technique vit
dans l'US ou dans les documents, jamais dans le chat. Sur incident : deux lignes. Le détail
n'est produit que sur demande explicite.

---

## 1. Sprint ≠ Session

| | **Sprint** | **Session** |
|---|---|---|
| **Identifiant** | `S44`, `S45`… | `SE-074`, `SE-075`… |
| **Défini par** | un **Sprint Goal produit** | une **conversation Claude Chat** |
| **Se ferme quand** | le Goal est atteint et la Sprint Outcome Gate est répondue | la capacité est atteinte, ou arrêt volontaire |
| **Produit** | un bump de version + une ligne dans `HISTORIQUE §Versions` | un **handoff** + 1 ligne dans `HISTORIQUE §Sessions` |

- Cas normal : **1 sprint = N sessions**. Un sprint qui ne tient pas dans une conversation produit plusieurs handoffs.
- Cas rare : **1 session = 2 sprints** si un Goal est atteint tôt. Claude signale « Goal atteint, on ouvre le suivant ? ».
- Cas hors sprint : une session peut être un **chantier** (documentation, audit). Marquée `hors sprint`, **aucun bump**.

**Version :** `0.<n>.0`, incrémenté à chaque clôture de sprint **ayant livré du code**.
Patch `0.<n>.1` pour un correctif hors sprint. La correspondance sprint ↔ version vit dans
`HISTORIQUE §Versions` ; elle ne se déduit pas.

**`DEC-155` — un sprint fait 10 slots :** 7 US planifiées + 3 flex (bugs sortis en route,
retours bêta qualifiés). **Clôture à date, jamais à épuisement** — le non-fini glisse dans
`ROADMAP.md`.

---

## 2. Cérémonies

| Cérémonie | Quand | Contenu |
|---|---|---|
| **Ouverture de session** | Début de conversation | Lire la Knowledge → confirmer en 1 phrase + **signaler toute contradiction entre documents** → afficher le Kanban (`STATE` **+** `HISTORIQUE §Tampon`) → **remesurer les faits externes** (§5) → questions → PLAN → attendre le `go` |
| **Sprint Planning** | Ouverture de sprint | Sprint Goal + choix des US + vérification du budget dette (§4) |
| **Backlog Refinement** | En continu | À chaque US close, 1 question de refinement sur une US du sprint suivant |
| **TNR** | Avant chaque merge | Porte selon la classe de risque (§3). Preuves de l'agent **irrecevables** — seule la machine PO fait foi (R1) |
| **Sprint Outcome Gate** | Clôture de sprint | 1 ligne : gain ressenti / gain de fiabilité / dette justifiée (§4) |
| **Release** | Clôture avec livraison | Bump + ligne dans `HISTORIQUE`. Le déploiement étant continu, la release est un **repère versionné** |
| **Clôture de session** | Fin de conversation | Handoff + **1 append dans `HISTORIQUE`** (§6) |
| **Clôture de sprint** | Fin de sprint | **Le lot documentaire** (§6) — la seule fois où plusieurs documents changent |

---

## 3. Classe de risque et porte de qualité

**Principe :** garder la rigueur là où elle a attrapé de vrais bugs, l'alléger là où elle ne fait
que taxer le débit. Le détail des règles `R1→R7`, `R-CODE`, `R-SCOPE` vit dans `AGENTS.md`.

| Classe | Périmètre | Gate exigée |
|---|---|---|
| **🔴 CRITIQUE** | Boot / Persistance / Sync, moteur de reco (`recEngine`, `useRecommendations`), et toute US touchant l'orchestration, le store ou le câblage de composables | **Complète** : R1 + R2 (test runtime) + R4 (E2E DOM si écran) + lecture zéro-confiance du code brut |
| **🟠 STANDARD** | `[FEATURE]` / `[UX]` sur un composant existant, `[REC]` hors moteur | R1 + R4 (E2E ciblé) |
| **🟢 LÉGÈRE** | `[CSS]` / `[A11Y]` pur, `[UX-copy]`, libellés, styles, suppression de code mort prouvée | **R1 allégée** : type-check + `test:run` + build, 1 run groupé. Pas d'E2E — **sauf si un `v-if` interactif est ajouté → R4-bis obligatoire** |

**Cas particulier — US spec-only** (`*.spec.ts`, 0 logique) : 1 run groupé tous les ~6 tests.

**Règles de classement :** une US hérite de la **classe la plus haute qu'elle touche** · en cas
de doute → **classe supérieure** · le tag `[DETTE]` ne baisse jamais la classe · une US 🟢 qui
régresse un comportement runtime remonte définitivement en 🟠.

**Ce qui ne change JAMAIS, quelle que soit la classe :** zéro `any` · périmètre = fichiers listés
· sortie de commande en terminal brut · contenu intégral des fichiers livrés · impact utilisateur
+ recommandation · `npm run test:run` vert · **l'auteur du test ≠ l'auteur du code (R7)** sur tout
ce qui touche **un écran ou la persistance**.

**Tag des US :** `US-XXX [EPIC][SECTION][TYPE] Titre` + classe de risque en tête.
`[EPIC]` = la surface (`ARCHITECTURE_FONCTIONNELLE §1`) · `[SECTION]` = sous-zone ·
`[TYPE]` = `[FEATURE][UX][CSS][TEST][PERF][DETTE][A11Y][CI]`.

**`DEC-189` — maquette avant toute US visible.** Toute US touchant une surface vue par
l'utilisateur est précédée d'une maquette validée par le PO, montrant l'état actuel, les options,
et le coût de chacune (dont les specs E2E qu'elle casse). Pas de maquette validée, pas d'US.

---

## 4. North Star, Sprint Outcome Gate & budget dette

> **« Un nouvel utilisateur suit son premier anime sur son calendrier en moins de 2 minutes,
> et revient ≥ 2 jours la première semaine. »**

| Métrique | Question | Cible |
|---|---|---|
| **TTFA** — Time To First Anime | Combien de temps / de clics depuis une session vierge ? | ↓ (< 2 min, < 5 clics) |
| **Adds / semaine** | L'app crée-t-elle l'habitude d'ajouter ? | ↑ |
| **Jours-retour S1** | Combien de jours distincts sur la 1ʳᵉ semaine ? | ↑ (≥ 2/7) |

🔻 **Aucune des trois n'a jamais reçu de valeur.** Leurs valeurs vivent dans `STATE.md §Métriques`
dès qu'un sprint les rend mesurables. Jusque-là, la seule métrique au mur est le vert CI — et le
vert prouve la mécanique, pas la valeur.

**Un sprint ne se clôt pas sans répondre en une ligne à :** *« Qu'est-ce que l'utilisateur peut
faire / voir / ressentir aujourd'hui qu'il ne pouvait pas avant ce sprint ? »*
1. **Gain ressenti** → ✅ nominal. 2. **Gain de fiabilité visible** → ✅ acceptable.
3. **Aucun gain visible — dette / audit** → ⚠️ **exception, pas routine.** Justifié ET suivi d'un sprint produit.

**Garde-fous :** pas plus d'**1 sprint « aucun gain visible » consécutif** · budget dette ≤ **1 US
de dette pure pour 1 US à gain visible** par sprint.

**`DEC-157` — boucle bêta.** Retour brut du PO → qualification par Claude (impact utilisateur,
effort, risque) → **slot flex du sprint courant** ou `ROADMAP.md`. Jamais de correctif « au vol »
hors US. **Critère d'entrée en flex :** le défaut **empêche une action** (perdre sa bibliothèque,
ne pas s'inscrire, écran blanc, ajout impossible). Un irritant visuel va au cap, pas en flex.

---

## 5. Faits externes — remesure obligatoire

**Tout fait externe en standby est remesuré à l'ouverture de chaque session, avec la requête
EXACTE émise par le code** — jamais une version simplifiée. Un endpoint testé avec d'autres
paramètres est un autre endpoint.

*Origine : une panne externe portée **5 sprints** sur la foi d'un unique curl jamais rejoué,
gelant 2 items du backlog et se propageant dans 5 documents. Aucune règle de qualité de code
n'a protégé contre ça — c'était un défaut de **fraîcheur de fait**.*

**Corollaire :** toute US sortant du backlog démarre par un **grep du fichier concerné avant
rédaction**. Trois US ont été planifiées « à faire » alors qu'elles étaient en production.

L'état courant des faits externes vit dans `STATE.md §Faits externes`, avec sa date de mesure.

---

## 6. Cadence documentaire (`DEC-190`)

> Supersede `DEC-146`. Motif chiffré : sur SE-064 → SE-073, la régénération par session a produit
> **2 366 lignes de patchs pour 31 US**, dont **79 % sur `STATE.md` et `ROADMAP.md` seuls**.

### En cours de sprint — **1 seul geste par session**

À la clôture de chaque session, Claude produit **un unique bloc à coller à la fin de
`HISTORIQUE.md`** :
- **1 ligne** dans `§Sessions` — le résumé de la session ;
- **1 ligne par élément neuf** dans `§Tampon de sprint` — chaque `DEC-xxx`, `AUD-xx` ou piège
  créé pendant la session, en une ligne, avec son numéro **déjà attribué**.

Rien d'autre n'est touché. `STATE.md` reste tel quel et **peut dater de plusieurs sessions** :
c'est assumé, le tampon porte le delta.

**Exception, hors cadence :** `AGENTS.md` se patche **tout de suite** quand une règle d'agent
change, se **commite seul**, et est **redéployé à la racine de `A-Anime` avant l'envoi de toute
nouvelle US** (`AP-PROCESS-3`). Un fichier de gouvernance non commité sera embarqué par le
prochain commit de l'agent et produira un faux signal `R-SCOPE-1` contre lui.

### À la clôture de sprint — **le lot documentaire**

Cinq écritures, une seule fois par sprint. Toutes les additions vont **à la fin du document**.

| Document | Geste |
|---|---|
| `STATE.md` | **Remplacement intégral** — Kanban, métriques, faits externes, trous |
| `ROADMAP.md` | **Remplacement intégral** — cap, parking. Se régénère **entier ou pas du tout** |
| `DECISIONS.md` | **Append** en `§10 Sprint courant` |
| `AUDIT.md` | **Append** en `§Sprint courant` + bascule des constats soldés en 1 ligne |
| `ANTIPATTERNS.md` | **Append** en `§Sprint courant` |

Le `§Tampon` d'`HISTORIQUE.md` est **vidé** par ce lot : ses lignes ont rejoint leur document.
`HISTORIQUE §Sessions` et `§Versions` ne sont **jamais** réécrits.

**Double checklist de clôture de session**, même si la réponse est « rien à changer » :
1. **Côté Gemini** — `AGENTS.md` a-t-il changé ? Si oui, déployé à la racine de `A-Anime` ?
2. **Côté État** — la ligne `HISTORIQUE` et le tampon reflètent-ils la session ?

**Interdit :** différer un `DEC` au seul handoff. Un handoff est une source secondaire faillible
(`DEC-87`) — le tampon d'`HISTORIQUE` est la trace opposable, le handoff ne l'est pas.
**Vérification de dette par preuve, pas par recopie :** toute dette héritée d'un handoff se
vérifie contre le dépôt avant d'être reportée une fois de plus.

---

## 7. Hygiène documentaire

Le coût réel d'un doublon n'est pas le stockage, c'est **l'éviction** : chaque doublon consomme
un extrait de recherche qui aurait pu porter la bonne information.

| # | Règle |
|---|---|
| **H1** | **Un chiffre n'existe qu'une fois** — dans `STATE.md`. Ailleurs : un renvoi. *Exception unique : `AGENTS.md`, lu par un agent sans accès à la Knowledge* |
| **H2** | **Un backlog n'existe qu'une fois** — `STATE.md` pour le sprint courant, `ROADMAP.md` pour le cap. Aucun autre document ne porte de planification |
| **H3** | **Une règle n'existe qu'une fois** — `AGENTS.md` (code, livraison, test) ou `PILOTAGE.md` (cadence, gate). Les autres renvoient |
| **H4** | **Un antipattern s'écrit à la 2ᵉ occurrence, jamais à la 1ʳᵉ.** Une occurrence unique vit dans le tampon |
| **H5** | **Une décision périmée n'est jamais supprimée : elle est marquée** `⛔ SUPERSEDED PAR DEC-xxx` et bascule dans `DECISIONS_ARCHIVE.md`. Les renvois restent résolvables. **Un numéro n'est jamais réattribué** |
| **H6** | **Plafond : 250 lignes par document.** Une seule exception nommée : `TYPES_CONTRACT.md` (359) — mono-intention, à titres stables, et seul rempart contre l'invention de types dans une US autoportante. **Jamais de dossier d'archive** (`OLD/`, `archive/`) : la Knowledge indexe tout le dépôt |
| **H7** | **Titres thématiques, jamais chronologiques.** Un document organisé par session produit des extraits qui se contredisent d'une section à l'autre. *Seules exceptions : `HISTORIQUE §Sessions` et `§Versions`, qui sont des journaux par nature* |
| **H8** | **Purge tous les 5 sprints** (S45, S50…). Tout document dont l'état de référence a plus de 3 sprints de retard est relu ou marqué périmé. Sur `HISTORIQUE`, la purge **condense**, elle ne supprime jamais |

**Élagage du contexte de conversation.** Le PO peut éditer un message déjà envoyé pour en retirer
le volume devenu inutile : le texte supprimé ne revient pas dans le contexte. **Condition :** ce
qui est retiré est remplacé par sa version compressée **et** par les sorties brutes qui prouvent
le résultat — une sortie de terminal ne s'élague jamais. **Limite :** l'élagage allège la session
courante, il ne transmet rien à la suivante.

**`DEC-174` — le handoff se déclenche à 80 % de capacité, pas 90 %.** Un handoff coûte 8-10 % à
produire ; déclencher à 90 % garantit un document tronqué.

Les instructions personnalisées du projet ne vivent pas dans le dépôt : elles se collent dans la
configuration du projet Claude Chat.
