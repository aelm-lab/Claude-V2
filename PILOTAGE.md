# PILOTAGE.md — Cadence, gate et hygiène documentaire

> **Rôle :** comment on ouvre, classe et clôt un **sprint** et une **session**, quelle porte de qualité s'applique à quoi, et comment la documentation reste saine.
> **Pas ici :** l'état courant et le Kanban (→ `STATE.md`), les règles de code et de test (→ `AGENTS.md`), les décisions (→ `DECISIONS.md`).

---

## 1. Sprint ≠ Session — les deux compteurs

| | **Sprint** | **Session** |
|---|---|---|
| **Identifiant** | `S38`, `S39`… | `SE-049`, `SE-050`… |
| **Défini par** | un **Sprint Goal produit** | une **conversation Claude Chat** |
| **Se ferme quand** | le Goal est atteint et la Sprint Outcome Gate est répondue | la capacité de conversation est atteinte, ou arrêt volontaire |
| **Produit** | un bump de version + une ligne dans `HISTORIQUE.md §Versions` | un **handoff** `SE-00X → SE-00Y` |
| **Tracé dans** | `HISTORIQUE.md` | `HISTORIQUE.md §Sessions` |

**Relations possibles**
- Cas normal : **1 sprint = N sessions**. Un sprint qui ne tient pas dans une conversation n'est pas un problème — il produit plusieurs handoffs.
- Cas rare : **1 session = 2 sprints**, si un Goal est atteint tôt. Claude signale « Goal atteint, on ouvre le sprint suivant ? » plutôt que d'attendre le handoff.
- Cas hors sprint : une session peut être un **chantier** (documentation, audit, investigation) sans Sprint Goal. Elle est marquée `hors sprint` et ne produit **aucun bump de version**.

**Conséquence pratique :** une conversation qui meurt sur la capacité est un non-événement produit. Elle ne consomme pas de numéro de sprint et n'apparaît pas dans l'historique de version.

### Schéma de version

**Règle actuelle :** `0.<n>.0`, incrémenté à chaque clôture de sprint **ayant livré du code**. Patch `0.<n>.1` pour un correctif hors sprint. Un sprint sans livraison ne bumpe pas. La correspondance sprint ↔ version vit dans `HISTORIQUE.md §Versions`, elle ne se déduit pas.

---

## 2. Cérémonies

| Cérémonie | Quand | Contenu |
|---|---|---|
| **Ouverture de session** | Début de chaque conversation | Lire la Knowledge → confirmer en 1 phrase + signaler si `STATE.md` semble périmé → afficher le Kanban → **remesurer les faits externes** (§6) → questions de clarification → présenter le PLAN → attendre le `go` du PO |
| **Sprint Planning** | Ouverture de sprint | Définir le **Sprint Goal** + choisir les US (déjà raffinées au sprint précédent) + vérifier le **budget dette** (§5) |
| **Backlog Refinement** | En continu | À chaque US close, Claude pose **1 question de refinement** sur une US du sprint suivant. But : la 1ʳᵉ US du sprint suivant démarre déjà raffinée |
| **TNR** (non-régression) | Avant chaque merge | Porte selon la classe de risque (§3). Preuves de l'agent **irrecevables** — seule la machine PO fait foi (R1) |
| **Sprint Outcome Gate** | Clôture de sprint | 1 ligne : gain ressenti / gain de fiabilité / dette justifiée (§5) |
| **Release** | Clôture de sprint avec livraison | Bump de version + ligne dans `HISTORIQUE.md`. Le déploiement est déjà continu → la release est un **repère versionné**, pas une mise en production |
| **Clôture de session** | Fin de chaque conversation | Handoff + double checklist (§7) |

---

## 3. Classe de risque et porte de qualité

**Principe :** garder la rigueur là où elle a attrapé de vrais bugs, l'alléger là où elle ne fait que taxer le débit. Le gabarit « boot/persistance » ne doit pas s'appliquer tel quel à une US CSS de 44 px. La gate suit le risque, pas un réflexe uniforme.

Le détail des règles `R1→R7`, `R-CODE`, `R-SCOPE` vit dans `AGENTS.md`. Ce document dit **lesquelles s'appliquent à quoi**.

| Classe | Périmètre | Gate exigée | Justification |
|---|---|---|---|
| **🔴 CRITIQUE** | EPIC 8 (Boot / Persistance / Sync), EPIC 10 moteur (`recEngine`, `useRecommendations`) + toute US touchant l'orchestration, le store ou le câblage de composables | **Gate complète** : R1 (triple preuve) + R2 (test runtime) + R4 (E2E DOM si écran) + R6 (audit live avant clôture d'epic) + lecture zéro-confiance du code brut | C'est là que sont nés les 4 bugs runtime de s6 et le P0 de s16. La rigueur est *méritée* |
| **🟠 STANDARD** | `[FEATURE]` / `[UX]` sur un composant existant, `[REC]` hors moteur | R1 + R4 (E2E ciblé) | Touche l'écran → un clic réel doit valider |
| **🟢 LÉGÈRE** | `[CSS]` / `[A11Y]` pur, `[UX-copy]`, libellés, styles dans `style.css`, suppression de code mort prouvée | **R1 allégée** : type-check + `test:run` + build, les 3 verts, 1 run groupé. Pas d'E2E imposé — **sauf si un `v-if` interactif est ajouté → R4-bis obligatoire** | Une couleur ou un padding ne casse pas l'orchestration runtime. Le coût E2E n'a pas de ROI ici |

**Cas particulier — US spec-only** (`*.spec.ts` uniquement, 0 logique) : 1 run groupé tous les ~6 tests suffit.

### Règles de classement (sans ambiguïté)

1. **Une US hérite de la classe la plus haute qu'elle touche.** Une US « CSS » qui modifie aussi `usePersistence` est 🔴, pas 🟢.
2. **En cas de doute → classe supérieure.** Le doute coûte moins cher que le bug silencieux.
3. **Le tag `[DETTE]` ne baisse jamais la classe.** Une dette sur le boot reste 🔴.
4. **Garde-fou :** si une US 🟢 régresse un comportement runtime, elle remonte définitivement en 🟠 par antipattern logué.

### Ce qui ne change JAMAIS, quelle que soit la classe

- Zéro `any` (R-CODE-1).
- Périmètre = fichiers listés (R-SCOPE-1) ; démarrage de session = état des fichiers modifiés.
- Sortie de commande = terminal brut, jamais de paraphrase.
- Contenu intégral des fichiers livrés.
- **Impact utilisateur + recommandation** sur chaque US (PO non technique).
- `npm run test:run` reste vert. Seule la partie **E2E** se module.
- **L'auteur du test ≠ l'auteur du code** (R7). Aucune exception, même pour un test visuel jugé « simple ».

### Tag des US

`US-XXX [EPIC][SECTION][TYPE] Titre` + **classe de risque** en tête de l'US.
- **[EPIC]** = le OÙ (page / surface / système) — voir `EPICS.md`.
- **[SECTION]** = sous-zone (`WEEK`, `FOR-YOU`, `BOOT`, `PERSIST`, `SYNC`, `ENGINE`…).
- **[TYPE]** = nature transverse : `[FEATURE][UX][CSS][TEST][PERF][DETTE][A11Y][CI]`.

→ Vue par page (priorisation PO) **et** vue par type (filtrer toute la dette CSS) simultanées.

---

## 4. North Star produit

> **« Un nouvel utilisateur suit son premier anime sur son calendrier en moins de 2 minutes, et revient ≥ 2 jours la première semaine. »**

Tout sprint se demande : *est-ce que ce qu'on livre rapproche ou éloigne de cette phrase ?* Si la réponse est « ni l'un ni l'autre » (dette pure), le sprint doit le déclarer explicitement.

**Pourquoi cette section existe :** jusqu'à sa création, la seule métrique au mur était le vert CI. Le vert prouve la mécanique, pas la valeur.

| Métrique | Question à laquelle elle répond | Cible directionnelle |
|---|---|---|
| **TTFA** — Time To First Anime | Combien de temps / de clics pour suivre son 1ᵉʳ anime depuis une session vierge ? | ↓ (< 2 min, < 5 clics) |
| **Adds / semaine** | L'app crée-t-elle l'habitude d'ajouter des séries ? | ↑ |
| **Jours-retour S1** | Sur la 1ʳᵉ semaine, combien de jours distincts l'utilisateur revient ? | ↑ (≥ 2/7) |

Leurs **valeurs** vivent dans `STATE.md`, à côté du compteur de tests, jamais à la place. Elles n'ont de sens qu'à partir du moment où un sprint produit les rend mesurables.

---

## 5. Sprint Outcome Gate & budget dette

**Un sprint ne se clôt pas sans répondre, en une ligne, à :**

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait pas avant ce sprint ? »*

Trois réponses possibles :
1. **Gain ressenti** (ex. « il peut s'onboarder en 90 s ») → ✅ sprint produit, nominal.
2. **Gain de fiabilité visible** (ex. « il ne perd plus ses données en silence ») → ✅ acceptable.
3. **Aucun gain visible — dette / audit** → ⚠️ **autorisé en exception, pas en routine.** Doit être justifié ET suivi d'un sprint produit.

**Garde-fou anti-dérive :** pas plus d'**1 sprint « aucun gain visible » consécutif**. Deux d'affilée = signal d'alarme.

**Budget dette — plafond :** ≤ **1 US de dette pure pour 1 US à gain visible** par sprint. Un audit qui génère plusieurs US de dette s'**étale** sur plusieurs sprints.

Cette gate s'ajoute à la porte technique (§3) et aux règles de clôture (§7). Elle n'en remplace aucune.

---

## 6. Faits externes — remesure obligatoire

**Règle : tout fait externe inscrit en standby est remesuré à l'ouverture de chaque session, avec la requête EXACTE émise par le code — jamais une version simplifiée.** Un endpoint testé avec d'autres paramètres est un autre endpoint.

Origine : une panne externe a été portée **5 sprints** sur la foi d'un unique curl jamais rejoué, gelant 2 items du backlog et se propageant dans 5 documents. Aucune règle de qualité de code n'a protégé contre ça — c'était un défaut de **fraîcheur de fait**, pas de code.

Corollaire : **toute US sortant du backlog démarre par un grep du fichier concerné avant rédaction.** Trois US ont été planifiées « à faire » alors qu'elles étaient en production.

L'état courant des faits externes vit dans `STATE.md`, avec la date de la dernière mesure.

---

## 7. Clôture de session & handoff

**Cadence documentaire (DEC-146).** À chaque clôture de session, sans exception :
1. `STATE.md` — **régénéré intégralement**, jamais patché.
2. `HISTORIQUE.md` — **+1 ligne** de session (+1 ligne de version si le sprint se clôt).
3. Tout document de gouvernance touché par la session (`DECISIONS`, `TYPES_CONTRACT`, `ANTIPATTERNS`, `AUDIT`, `AGENTS`) — **patché dans la même session, jamais reporté.**

Un handoff qui ne touche aucun document est un signal d'alerte, pas un raccourci.

**Interdit : différer un DEC au handoff.** Un handoff est une source secondaire faillible — des décisions y ont vécu deux sessions, et le même handoff affirmait par ailleurs une dette déjà soldée depuis trois sessions.

**Vérification de dette par preuve, pas par recopie.** Toute dette de gouvernance héritée d'un handoff se vérifie contre le dépôt (`git log`, `findstr`) **avant** d'être reportée une fois de plus.

**Isolation du déploiement `AGENTS.md` (`AP-PROCESS-3`).** Le redéploiement d'`AGENTS.md` à la racine de `A-Anime` se **commite seul**, en clôture de session, **avant** l'envoi de toute nouvelle US. Un fichier de gouvernance non commité sera embarqué par le prochain commit de l'agent et produira un faux signal `R-SCOPE-1` contre lui. Corollaire : **`git status` vide est condition d'ouverture d'une livraison**, pas seulement de sa fermeture.

**Double checklist obligatoire.** Chaque handoff répond explicitement à deux questions, même si la réponse est « rien à changer » :
1. **Côté Gemini** — `AGENTS.md` a-t-il changé cette session ? Si oui, a-t-il été **déployé à la racine de `A-Anime`** ?
2. **Côté State** — `STATE.md` reflète-t-il la session (Kanban, métriques, faits externes) ?

**Nommage :** `HANDOFF SE-0XX → SE-0YY (sprint SXX)`. Il porte toujours le sprint, pour qu'une reprise sache si elle continue un Goal ou en ouvre un.

---

## 8. Hygiène documentaire

Le coût réel d'un doublon n'est pas le stockage : c'est **l'éviction**. Chaque doublon consomme un extrait de recherche qui aurait pu porter la bonne information.

| # | Règle | Quand | Qui tranche |
|---|---|---|---|
| **H1** | **Un chiffre n'existe qu'une fois.** Tout compteur (tests, E2E, build, hash, version) vit dans `STATE.md`. Ailleurs : un renvoi. *Exception unique : `AGENTS.md`, lu par un agent qui n'a pas accès à `STATE.md`* | Chaque session | Claude applique |
| **H2** | **Un backlog n'existe qu'une fois** — `STATE.md`. `EPICS.md` ne porte que du livré | Chaque session | Claude |
| **H3** | **Une règle n'existe qu'une fois** — `AGENTS.md` (code, livraison, test) ou `PILOTAGE.md` (cadence, gate). Les autres documents y renvoient | À chaque nouvelle règle | PO |
| **H4** | **Un antipattern s'écrit à la 2ᵉ occurrence, jamais à la 1ʳᵉ** | Review | Claude propose, PO valide |
| **H5** | **Une décision périmée n'est jamais supprimée : elle est marquée** `⛔ SUPERSEDED PAR DEC-xxx` et bascule dans `DECISIONS_ARCHIVE.md`. Les renvois existants restent résolvables | À chaque DEC contredisant un DEC | Claude |
| **H6** | **Archiver = un fichier nommé, hors ordre de lecture.** **Jamais de dossier d'archive** (`OLD/`, `archive/`) : la Knowledge indexe tout le dépôt et un dossier continue de remonter en recherche | À la suppression | PO |
| **H7** | **Plafond : 250 lignes par document de Knowledge.** Le critère réel est la **dispersion thématique** : au-delà, un document mêle plusieurs intentions et renvoie des extraits sans rapport. Trois exceptions nommées, mono-intention à titres stables : `AGENTS.md` (350), `TYPES_CONTRACT.md` (350), `STATE.md` (200, DEC-146). Les satellites append-only (`HISTORIQUE`, `AUDIT`, `DECISIONS_ARCHIVE`) n'ont pas de plafond | Continu | Claude alerte |
| **H8** | **Titres thématiques, jamais chronologiques.** Un document organisé par session produit des extraits qui se contredisent d'une section à l'autre | À l'écriture | Claude |
| **H9** | **Purge tous les 5 sprints.** Tout document dont l'état de référence a plus de 3 sprints de retard est relu ou marqué périmé. Sur les satellites append-only, la purge **condense**, elle ne supprime jamais | S45, S50… | PO déclenche |

### Carte des documents

| Doc | Rôle | Lu par |
|---|---|---|
| `CLAUDE.md` | Carte d'entrée : ce qu'est l'app, qui code, où trouver le reste | Claude |
| `STATE.md` | État mesuré : sprint/session, compteurs, Kanban, faits externes, trous | Claude |
| `PILOTAGE.md` | Ce fichier : cadence, gate, North Star, hygiène | Claude |
| `EPICS.md` | Taxonomie des EPICs + acquis fonctionnel durable | Claude |
| `ARCHITECTURE_FONCTIONNELLE.md` | Parcours utilisateur + pont fonctionnel ↔ technique | Claude |
| `ARCHITECTURE_TECHNIQUE.md` | Couches, modules, boot, réseau, registre des clés | Claude |
| `TYPES_CONTRACT.md` | Contrat TypeScript — seule source d'interfaces | Claude |
| `DECISIONS.md` | Les choix encore appliqués | Claude |
| `ANTIPATTERNS.md` | Pièges **répétés**, classés par thème | Claude |
| `AGENTS.md` | Canon des règles de l'agent — écrit ici, **déployé à la racine de `A-Anime`** | Gemini (+ Claude) |
| **Satellites — hors ordre de lecture, ouverts à la demande** | | |
| `HISTORIQUE.md` | Sessions closes et versions livrées, append-only | Claude |
| `DECISIONS_ARCHIVE.md` | Décisions closes, périmées et superseded | Claude |
| `AUDIT.md` | Constats de code par campagne, append-only | Claude |
| `BENCHMARK.md` | Comparatif concurrentiel trié | Claude |

Les instructions personnalisées du projet ne vivent pas dans le dépôt : elles se collent dans la configuration du projet Claude Chat.
