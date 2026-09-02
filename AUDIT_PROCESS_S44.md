# AUDIT_PROCESS_S44.md

> 🔻 **DOCUMENT TEMPORAIRE — à supprimer après décision du PO.**
> Hors ordre de lecture, hors corpus, hors plafond H7. Auditeur externe, mandat radical.
> Mesures faites le 2026-09-02 sur `aelm-lab/Claude-V2` @ `00f095a` et `aelm-lab/A-Anime` @ `399892c`.
>
> **Périmètre réellement lu :** les 15 `.md` de Claude-V2 + `AAA doc knowledge`, l'`AGENTS.md`
> déployé dans A-Anime, `package.json`, `tests/e2e/`, et l'historique git complet des deux dépôts
> (282 commits Knowledge, 212 commits code, désuperficialisés).
> **Le dossier `OLD/` n'existe pas** : supprimé par le commit `766ac65` (« Delete OLD directory »),
> conformément à H6. Rien n'a été inféré.

---

## 1. VERDICT

Le process n'est pas cassé : il est **alourdi par deux documents qui se réécrivent entiers**.
`STATE.md` et `ROADMAP.md` produisent **79 % du volume de patchs** alors qu'ils ne portent
aucune connaissance durable. La douleur déclarée par le PO est objectivée et localisée à ces deux
fichiers — pas au corpus entier, pas aux règles de code. Les règles qui protègent réellement le
code sont peu nombreuses, tracées, et coûtent presque rien. Le reste est de la cérémonie
autoréférentielle : 4 documents ne changent plus depuis 4 sprints, 2 documents se contredisent
eux-mêmes dans leur propre corps, et le process n'a **aucun chemin** pour un bug remonté par un
testeur.

---

## 2. LES CHIFFRES

### 2.1 Corpus aujourd'hui (Claude-V2 @ `00f095a`)

| Document | Lignes | Mots | Position |
|---|---:|---:|---|
| AUDIT.md | 586 | 8 005 | satellite |
| TYPES_CONTRACT.md | 359 | 1 813 | ordre de lecture (7) |
| ROADMAP.md | 332 | 2 983 | **non déclaré dans PILOTAGE §8** |
| AGENTS.md | 279 | 3 122 | ordre de lecture (10) |
| ARCHITECTURE_TECHNIQUE.md | 258 | 1 819 | ordre de lecture (6) |
| DECISIONS.md | 222 | 6 047 | ordre de lecture (8) |
| ANTIPATTERNS.md | 217 | 3 330 | ordre de lecture (9) |
| PILOTAGE.md | 206 | 2 515 | ordre de lecture (3) |
| EPICS.md | 142 | 1 158 | ordre de lecture (4) |
| ARCHITECTURE_FONCTIONNELLE.md | 138 | 1 437 | ordre de lecture (5) |
| BENCHMARK.md | 117 | 1 424 | satellite |
| DECISIONS_ARCHIVE.md | 98 | 1 393 | satellite |
| CLAUDE.md | 91 | 938 | ordre de lecture (1) |
| HISTORIQUE.md | 81 | 1 603 | satellite |
| STATE.md | 80 | 696 | ordre de lecture (2) |
| AAA doc knowledge | 10 | 37 | non déclaré |
| **Ordre de lecture (10 docs)** | **1 992** | **22 875** | lu à chaque ouverture |
| **Satellites (5 docs)** | **1 214** | **15 408** | à la demande |
| **TOTAL** | **3 206** | **38 283** | |

### 2.2 Courbe de croissance (lignes, au commit de clôture de sprint)

| Document | S38 (`0379581`) | S40 (`516a0d0`) | S42 (`94c7618`) | S44 (`HEAD`) | S40→S44 |
|---|---:|---:|---:|---:|---:|
| AUDIT.md | 194 | 280 | 499 | **586** | **×2,09** |
| ROADMAP.md | — | — | 128 | **332** | **×2,59** (S42→S44) |
| ANTIPATTERNS.md | 228 | 148 | 192 | 217 | ×1,47 |
| DECISIONS.md | 248 | 161 | 209 | 222 | ×1,38 |
| HISTORIQUE.md | — | 63 | 69 | 81 | ×1,29 |
| PILOTAGE.md | 243 | 195 | 195 | 206 | ×1,06 |
| AGENTS.md | 351 | 274 | 275 | 279 | ×1,02 |
| TYPES_CONTRACT.md | 347 | 359 | 359 | 359 | ×1,00 |
| ARCHITECTURE_TECHNIQUE.md | 291 | 258 | 258 | 258 | ×1,00 |
| ARCHITECTURE_FONCTIONNELLE.md | 211 | 138 | 138 | 138 | ×1,00 |
| EPICS.md | 176 | 142 | 142 | 142 | ×1,00 |
| BENCHMARK.md | — | 117 | 117 | 117 | ×1,00 |
| CLAUDE.md | 130 | 90 | 91 | 91 | ×1,01 |
| STATE.md | 193 | 133 | 121 | 80 | ×0,60 |
| **TOTAL corpus** | **2 612** | **2 450** | **2 893** | **3 206** | **+31 %** |

**Deux documents ont doublé : `AUDIT.md` et `ROADMAP.md`. À eux deux : +510 lignes sur les +756
de croissance S40→S44, soit 67 %.** Aucun des deux n'est dans l'ordre de lecture.

### 2.3 Volume de patchs (SE-064 → SE-073, 2026-08-20 → 2026-09-01)

| Mesure | Valeur |
|---|---|
| Commits Knowledge | 68 — **1,01 fichier par commit** (un commit = un document) |
| Commits Knowledge par session | **6,8 en moyenne** (min 2, max 10) |
| Lignes de doc écrites ou réécrites | **2 366** (1 530 ajoutées / 836 supprimées) |
| US livrées sur la fenêtre (`HISTORIQUE.md`) | 31 (SE-064 en a livré 0 — session hors sprint) |
| **Ratio lignes de doc / US livrée** | **76 lignes** |
| **Ratio patchs doc / US livrée** | **2,2 patchs** |
| Micro-commits (< 5 lignes) | **20 / 68 = 29,4 %** — fichier n°1 : `HISTORIQUE.md` (7) |
| Micro-commits sur tout l'historique | 57 / 282 = 20,2 % |

### 2.4 Répartition du churn documentaire (depuis SE-064)

| Document | Lignes réécrites | Part |
|---|---:|---:|
| **STATE.md** | **1 203** | **51 %** |
| **ROADMAP.md** | **668** | **28 %** |
| AUDIT.md | 253 | 11 % |
| DECISIONS.md | 77 | 3 % |
| ANTIPATTERNS.md | 69 | 3 % |
| HISTORIQUE.md | 35 | 1 % |
| AGENTS · PILOTAGE · TYPES_CONTRACT · DEC_ARCHIVE · CLAUDE | 47 | 2 % |
| **EPICS · BENCHMARK · ARCH_TECHNIQUE · ARCH_FONCTIONNELLE** | **0** | **0 %** |

**`STATE.md` + `ROADMAP.md` = 1 871 lignes sur 2 366, soit 79 % de tout le volume de patchs.**
Ce sont les deux seuls documents dont la règle impose la régénération intégrale.

`STATE.md` fait 80 lignes et a produit 1 203 lignes de churn en 10 sessions : **il a été réécrit
15 fois son propre volume.** 655 lignes de corpus (20 %) n'ont pas bougé d'une ligne en 4 sprints
et sont pourtant relues à chaque ouverture de session.

### 2.5 Dépôt de code A-Anime (49 commits, 2026-08-24 → 09-01, soit SE-066 → SE-073)

| Mesure | Valeur |
|---|---|
| Fichiers par commit | moyenne **1,78** · médiane 1 · max 4 |
| Commits dépassant la règle « max 3 fichiers » | **2 / 49 = 4 %** |
| Churn par commit | moyenne 69 lignes · médiane 38 |
| Commits de code sur la fenêtre | 49 pour 28 US ≈ **1,75 commit / US** |
| Délai ouverture → merge d'une US | **non datable** : aucun message de commit ne porte d'ID d'US. Proxy observable : le Kanban ne reporte aucune US d'une session à l'autre (`STATE.md:44` « In Progress : Aucune ») → ouverture et merge dans la même session |
| Churn code vs churn doc (même fenêtre) | code 3 380 lignes / **doc 1 891 lignes = 56 % du code** |
| Commits code vs commits doc (même fenêtre) | 49 code / **51 doc — la Knowledge est commitée plus souvent que le produit** |

### 2.6 `AGENTS.md` — synchronisation et chiffres figés

Contenu **identique** entre `Claude-V2/AGENTS.md` et `A-Anime/AGENTS.md`, à un caractère près
(saut de ligne final absent côté A-Anime). Aucune divergence de fond. En revanche, chiffres figés
désormais faux dans le document que Gemini est le seul à lire :

| Ligne | Affirmation | Réalité mesurée |
|---|---|---|
| `AGENTS.md:233` | « Dernier sweep complet : **52 / 52** verts » | `STATE.md:18` : 55/55 en SE-072 |
| `AGENTS.md:204` | « **batch3 (8)** » | `package.json:16` : 9 specs |
| `AGENTS.md:206` | « **batch5 (8)** » | `package.json:18` : 9 specs |
| `AGENTS.md:72` | « Le helper `makeAnime()` **n'existe plus** » | présent dans **5 fichiers** de `src/**/*.spec.ts` |
| `AGENTS.md:198` | « Chemin complet obligatoire : `tests/e2e/<nom>.spec.ts`, jamais le nom nu » | **43 des 45 entrées du §7 sont en nom nu** |
| `AGENTS.md:200` | « 45 specs / 45 enregistrées » | ✅ exact (vérifié sur disque et dans `package.json`) |

### 2.7 Duplications et contradictions (fichier:ligne des deux occurrences)

| Information | Occurrence 1 | Occurrence 2 | Statut |
|---|---|---|---|
| Helper `makeAnime` | `AGENTS.md:72` « n'existe plus » | `ANTIPATTERNS.md:30` « → factory `makeAnime(...)` » | **contradiction** |
| État de la bêta | `DECISIONS.md:155` (DEC-120) « en beta avec des testeurs réels » | `STATE.md:71` « Bêta non ouverte — zéro observation utilisateur » | **contradiction** |
| État de S43 | `ROADMAP.md:25` « ✅ CLOS — v0.37.0 » | `ROADMAP.md:67` « EN COURS depuis SE-070 » | **contradiction interne** |
| État d'`AUD-52` | `AUDIT.md:552` « SOLDÉ » | `ROADMAP.md:258` « 🔴 Ouvert » | **contradiction** |
| Nombre de specs | `STATE.md:19` « 45 » | `HISTORIQUE.md:55` et `:80` « 43 specs » | **contradiction** |
| Règle R4-ter | `AGENTS.md:117-121` | `DECISIONS.md:219` (DEC-186) | doublon intégral |
| Format de réponse au PO | `PILOTAGE.md:5-8` | `DECISIONS.md:222` (DEC-188) | doublon intégral |
| R7 auteur test ≠ auteur code | `AGENTS.md:127` | `PILOTAGE.md:77` · `CLAUDE.md:81` · `DECISIONS.md:126` · `ROADMAP.md:217` | **5 occurrences** |
| Clé `aanime_calendar` | `AGENTS.md:146,160,242,279` | `CLAUDE.md:72` · `ARCHITECTURE_TECHNIQUE.md:203` · `DECISIONS.md:30` | **7 occurrences** |
| Registre des batchs E2E | `AGENTS.md:202-206` | `package.json:14-18` | doublon maintenu à la main, déjà dérivé (§2.6) |
| Vocabulaire figé « Coming Soon » | `CLAUDE.md:73` | `EPICS.md:69` · `DECISIONS.md:87,88` · `ARCHITECTURE_FONCTIONNELLE.md:21` | 5 occurrences |
| Max 3 fichiers par US | `AGENTS.md:60` | `CLAUDE.md:81` | doublon |
| Zéro `any` | `AGENTS.md:69` | `PILOTAGE.md:71` · `ANTIPATTERNS.md:28` · `ARCHITECTURE_TECHNIQUE.md:13` | 4 occurrences |
| Régénération de STATE | `PILOTAGE.md:141` | `STATE.md:5` | doublon |

### 2.8 Défauts structurels vérifiés

| Constat | Preuve |
|---|---|
| `DEC-159` désigne **deux décisions différentes** | `DECISIONS.md:169` (rejet session-only) et `DECISIONS.md:171` (garde anti-écrasement) |
| `AUD-32` / `AUD-33` ont été attribués deux fois | note de réconciliation `AUDIT.md:383-401` |
| Un antipattern n'a jamais reçu de numéro | `ANTIPATTERNS.md:119` — littéralement `AP-TEST-x` |
| `AP-PROCESS-4` est écrit deux fois | `ANTIPATTERNS.md:152-157` et `ANTIPATTERNS.md:193-198` |
| Deux sections `## 9` dans le même document | `ANTIPATTERNS.md:159` et `ANTIPATTERNS.md:168` |
| Trous de numérotation AP | ni `AP-PROCESS-1`, ni `AP-TEST-1..4`, ni `AP-DIAG-1/2` n'existent |
| `AUD-53` vit hors du registre d'audit | seule occurrence : `ROADMAP.md:254` — absent d'`AUDIT.md` |
| `AUD-21` / `AUD-22` cités mais jamais écrits | `HISTORIQUE.md:36,38,40` — absents d'`AUDIT.md` |
| `SE-070` absente du journal append-only | `HISTORIQUE.md` saute de SE-069 à SE-071, alors que `HISTORIQUE.md:52` la cite |
| `AUDIT.md` est organisé par session, pas par thème | 13 sections « Campagne SE-0xx » — violation frontale de H8 (`PILOTAGE.md:175`) |
| Deux campagnes portent le même titre « SE-065 » | `AUDIT.md:336` et `AUDIT.md:361` |
| `AUDIT.md:13` « Aucun autre document ne référence celui-ci » | faux : cité par `CLAUDE.md`, `PILOTAGE.md`, `ROADMAP.md`, `HISTORIQUE.md`, `DECISIONS_ARCHIVE.md` |
| `STATE.md:30` « Documents au-dessus du plafond H7 : 0 » | faux : `ROADMAP.md` = 332 lignes, plafond H7 = 250, aucune exception nommée pour lui |
| `ROADMAP.md` absent de la carte des documents | `PILOTAGE.md:180-196` liste 14 documents, pas celui-là |
| `DECISIONS.md` mélange 4 formats d'écriture | tableau `\|**DEC-x**\|` (l.15), texte tabulé nu (l.165), liste à puces (l.183), en-têtes de tableau orphelins (l.217, l.220) |
| ESLint ne peut pas être exécuté | aucun script `lint` dans `package.json` — `DEC-02` l'exige pourtant, `STATE.md:23` le constate |

### 2.9 Classement des règles — A protège · B théorique · C coûteuse · D morte

| Règle | Cl. | Preuve |
|---|:--:|---|
| `R-SCOPE-1` fichiers listés | **A** | 5 fichiers hors US → 17/17 E2E cassés (`AGENTS.md:54`). 4 % de dépassement résiduel aujourd'hui |
| `R-CODE-7` contrat d'event | **A** | `RecCard` : 3 emits morts sur 4 consommateurs, 0 erreur console (`AGENTS.md:96`) |
| `R-LIVRAISON-2` sortie brute | **A** | récidives n°1 et n°7 (`AGENTS.md:260,266`) — « 81 passed » structurellement impossible |
| Helper de mock unique (`AGENTS §7`) | **A** | 23 specs à mock propre → 11 rouges d'un coup au changement de fournisseur (`AGENTS.md:224`) |
| Registre des batchs E2E | **A** | une spec non enregistrée ne tourne jamais, silencieusement (`ANTIPATTERNS.md:111`) |
| Remesure des faits externes (`PILOTAGE §6`) | **A** | 5 sprints gelés sur un curl jamais rejoué (`PILOTAGE.md:131`) |
| Seed 7 jours / auto-vault (`AGENTS §6`) | **A** | deux pièges gravés, reproduits (`AGENTS.md:165-166`) |
| `R1` triple preuve sur la machine du PO | **A** | build Gemini divergent de 71 kB sur 3 livraisons (`DECISIONS.md:181`) |
| `R-CODE-1` zéro `any` | **B** | jamais vérifié : aucun script `lint` dans `package.json`. Tenu par `vue-tsc` seul |
| `R-CODE-3` séparation des couches | **B** | `AUD-13` montre la couche violée en production sans que la règle l'ait empêché |
| `R-CODE-6` pas d'état sur `window` | **B** | aucun incident tracé dans le corpus |
| North Star (TTFA, adds/sem., jours-retour) | **B** | `PILOTAGE.md:104` : « leurs valeurs vivent dans `STATE.md` » — **aucune des 3 n'y figure**, 4 sprints après |
| Régénération intégrale de `STATE.md` (`DEC-146`) | **C** | **1 203 lignes de churn pour un fichier de 80 lignes** (§2.4) |
| Régénération intégrale de `ROADMAP.md` | **C** | 668 lignes de churn, et le document se contredit quand même (`:25` vs `:67`) |
| Numérotation `AP-xx` | **C** | 8 numéros manquants, un `AP-TEST-x` jamais attribué, `AP-PROCESS-4` écrit deux fois |
| `R7` auteur du test ≠ auteur du code | **C** | voir Q4 ci-dessous : 0 rouge légitime sur 26 US, 4 specs fausses côté Tech Lead |
| Chiffres figés dans `AGENTS.md` (exception H1) | **C** | 4 des 6 chiffres figés sont faux aujourd'hui (§2.6) |
| Isolation du commit `AGENTS.md` (`AP-PROCESS-3`) | **C** | impose un commit dédié par session pour un fichier qui a bougé de 21 lignes en 10 sessions |
| `H2` un backlog n'existe qu'une fois | **D** | `ROADMAP.md:120-170` re-décrit le Kanban S44 de `STATE.md` |
| `H7` plafond 250 lignes | **D** | `ROADMAP.md` = 332. `STATE.md:30` affirme pourtant « 0 document au-dessus du plafond » |
| `H8` titres thématiques, jamais chronologiques | **D** | `AUDIT.md` = 13 sections « Campagne SE-0xx » |
| `H9` purge tous les 5 sprints | **D** | dernière purge SE-063.b (S40) ; S45 est la prochaine échéance, jamais déclenchée depuis |
| `DEC-02` ESLint `no-explicit-any` en erreur | **D** | aucun script `lint` ; `STATE.md:23` : « ❌ jamais exécuté » |
| `R-CODE-8` aucun `<style scoped>` ajouté | **D** | 19 composants en portent ; le bug `DEC-185` est né dans l'un d'eux en SE-072 |
| `DEC-120` « app en beta avec testeurs réels » | **D** | contredit par `STATE.md:71` — aucun testeur n'a ouvert l'app |

**Réponses explicites aux 5 questions de la mission :**

1. **Numérotation stricte** — coût > bénéfice **sur `AP-xx` uniquement**. `AUDIT.md:399` le
   conclut lui-même : « un identifiant attribué deux fois coûte plus cher qu'un trou de
   numérotation ». Bilan tracé : 2 collisions (`DEC-159`, `AUD-32/33`), 1 identifiant jamais
   attribué, 1 entrée dupliquée, 8 trous — contre **zéro incident évité documenté**. Mais
   `DEC-xx` et `AUD-xx` sont massivement cités (US, Kanban, handoffs) : on garde ces deux-là.
2. **Régénération de `STATE.md`** — **oui, c'est la source principale du volume de patchs** :
   51 % à elle seule, 79 % avec `ROADMAP.md`. La justification d'origine (`DEC-146` : un document
   régénéré ne conserve pas d'erreur) est réelle mais se paie 1 203 lignes pour 10 sessions. Une
   régénération **par clôture de sprint** au lieu de par session en conserve 90 % du bénéfice
   pour 20 % du coût.
3. **10 documents nécessaires ?** Non — 9 suffisent au total (§4). `ANTIPATTERNS` fusionne dans
   `AGENTS` (mêmes règles, écrites deux fois : comparer `ANTIPATTERNS §1-2` et `AGENTS §4`) ;
   `EPICS` fusionne dans `ARCHITECTURE_FONCTIONNELLE` (la table des 12 EPICs y a sa place, les
   « acquis » dupliquent le §4 « LE PONT ») ; `ROADMAP`, `BENCHMARK` et `DECISIONS_ARCHIVE`
   disparaissent. Aucune information opposable n'est perdue : elle est déplacée ou déjà en double.
4. **Taux de détection du test de fidélité** — **0 bug attrapé sur les 26 dernières US.**
   `STATE.md:40` porte un streak de 26 US mergées au premier coup ; `HISTORIQUE.md:42,44,48,51`
   confirment « MERGE au 1ᵉʳ coup » sur toute la série. Sur la même période, **4 specs écrites par
   le Tech Lead se sont révélées fausses** : `US-PERSIST-P0a` verte et sans effet
   (`AUDIT.md:380`, `AP-PROCESS-5`), `US-MALIMPORT-FIX` rouge à tort (`HISTORIQUE.md:48`), et deux
   autres attrapées par Gemini en SE-067 (`HISTORIQUE.md:49`). La règle n'a rien attrapé et a
   produit 4 faux signaux. Elle reste justifiée là où un bug est invisible au vert (orchestration,
   persistance, écran) — pas ailleurs.
5. **Où le process casse face à un bug bêta** — sur la cadence. `DEC-157` (`DECISIONS.md:167`)
   interdit tout correctif hors US ; `PILOTAGE.md:141` impose la régénération de `STATE.md` à la
   clôture ; `AP-PROCESS-4` interdit d'écrire une US incomplète. Un bug bloquant exige donc une
   session complète et un slot flex — `STATE.md:54` en réserve **un seul sur dix**. Il n'existe
   aucun état « hotfix » dans le Kanban, aucune classe de risque pour un défaut observé en
   production, et `STATE.md:44` (« In Progress : Aucune ») ne peut pas coexister avec une US
   en cours interrompue.

---

## 3. LES 5 PROBLÈMES RÉELS (coût décroissant pour le PO)

### P1 — La régénération intégrale de `STATE.md` et `ROADMAP.md` (79 % des patchs)
`STATE.md` = 51 % du churn, `ROADMAP.md` = 28 %. Ce sont les deux seuls documents dont la règle
dit « régénéré entier, jamais patché » (`PILOTAGE.md:141`, `ROADMAP.md:15`). Chaque session, le PO
colle-remplace 80 à 130 lignes qui décrivent un présent déjà périmé à la session suivante. **Coût
direct : ~120 lignes collées par session pour 2 à 6 US.** C'est exactement la douleur déclarée, et
elle ne vient ni du corpus, ni des règles de code.

### P2 — Le corpus paie deux fois : 655 lignes figées relues à chaque ouverture
`EPICS.md`, `BENCHMARK.md`, `ARCHITECTURE_TECHNIQUE.md`, `ARCHITECTURE_FONCTIONNELLE.md` :
**0 ligne modifiée en 4 sprints**, 655 lignes lues à chaque session. `BENCHMARK.md` est
explicitement gelé (`ROADMAP.md:175` : « Pas avant S45 »). Ces documents ne coûtent pas de patch —
ils coûtent de la capacité de conversation, donc des US en moins par session. `DEC-188`
(`DECISIONS.md:222`) mesure déjà l'effet : **85 % de capacité consommée pour 2 US en SE-073.**

### P3 — La cérémonie documentaire produit 2 patchs par US livrée
`PILOTAGE.md:141-144` impose 3 écritures minimum par clôture (STATE + HISTORIQUE + tout document
de gouvernance touché), plus l'isolation d'`AGENTS.md` en commit seul (`PILOTAGE.md:152`).
Résultat mesuré : **6,8 commits par session, dont 29,4 % changent moins de 5 lignes.** Le fichier
n°1 des micro-commits est `HISTORIQUE.md` (7 sur 20) — un document qui coûte, de son propre aveu
(`HISTORIQUE.md:22`), « ~30 tokens par session ». Le contenu est bon marché ; c'est le **geste**
qui coûte au PO.

### P4 — Les documents se contredisent, et le process ne le détecte pas
6 contradictions actives (§2.7), dont deux graves : `DEC-120` affirme que l'app est en bêta avec
des testeurs réels alors que `STATE.md:71` dit l'inverse — et `DEC-120` est la décision qui
qualifie un P0. Et `AGENTS.md:72` interdit à Gemini un helper qui existe dans 5 fichiers du dépôt.
Aucune règle H1→H9 n'a attrapé ces cas, parce qu'elles portent sur les **chiffres**, pas sur les
**faits**. `STATE.md:30` affirme même « 0 document au-dessus du plafond » alors que `ROADMAP.md`
le dépasse de 82 lignes.

### P5 — Il n'existe aucun chemin pour un bug remonté par un testeur
`DEC-157` (`DECISIONS.md:167`) décrit la boucle bêta : retour → qualification → **slot flex du
sprint courant ou ROADMAP.md**, « jamais de correctif au vol hors US ». Le chemin le plus court
pour un bug bloquant est donc : rédaction d'une US autoportante + test de fidélité écrit par
Claude + gate 3 sorties + `STATE.md` régénéré. Sur la fenêtre mesurée, cela représente **1 session
entière**. `STATE.md:54` réserve *un* slot flex sur dix. Un testeur qui perd sa bibliothèque attend
la prochaine conversation.

---

## 4. PLAN DE COUPE

Objectif contraint : −40 %. **Résultat proposé : 3 206 → 1 580 lignes, soit −51 %, et 16 → 9
fichiers.** Aucun document nouveau n'est créé.

| Document | Avant | Après | Action | Risque accepté |
|---|---:|---:|---|---|
| **AUDIT.md** | 586 | **90** | **Alléger** — ne garder qu'un tableau des `AUD-xx` **ouverts** (`ID \| constat \| fichier:ligne \| impact utilisateur`). Les 13 sections « Campagne SE-0xx » et tous les constats soldés partent | On perd le *récit* des diagnostics clos. Un constat rouvert sera rediagnostiqué de zéro. Contrepartie : la trace reste dans `git log` et dans la ligne `HISTORIQUE.md` de la session |
| **ROADMAP.md** | 332 | **0** | **Supprimer** — §1 (11 lignes de cap S45→S50) migre dans `STATE.md` ; §2 duplique le Kanban et `HISTORIQUE` ; §3 duplique R7/`DEC-155`/`DEC-128` ; §4 duplique `AUDIT` ; §5 est un livrable, pas un document ; §6 (parking) migre dans `STATE.md §Backlog` | On perd les justifications détaillées de composition de S42/S43 — déjà closes. On perd le « parking des refusés » comme page dédiée : le risque est de re-proposer une idée déjà refusée. 6 lignes dans `STATE.md` suffisent à l'éviter |
| **ANTIPATTERNS.md** | 217 | **0** | **Fusionner** — §1 à §6 vont dans `AGENTS.md §4/§5` (ce sont les mêmes règles, écrites deux fois) ; §7 à §9 (méthode Claude, `AP-METHOD-1`, `AP-DIAG-3`, `AP-PROCESS-4/5`) vont dans `PILOTAGE.md`, 25 lignes | Les pièges deviennent lisibles par Gemini : c'est un gain, pas une perte. Vrai risque : la règle H4 « un piège s'écrit à la 2ᵉ occurrence » disparaît comme rituel — elle devient un critère d'écriture dans `AGENTS.md`, non un document |
| **EPICS.md** | 142 | **0** | **Fusionner** — la table de taxonomie des 12 EPICs (20 lignes) entre dans `ARCHITECTURE_FONCTIONNELLE.md §1` ; les « acquis » par EPIC dupliquent déjà le §4 « LE PONT » | On perd le tag `[EPIC]` documenté ailleurs qu'en une table. Aucun impact : le tag sert à nommer une US, pas à décider |
| **BENCHMARK.md** | 117 | **0** | **Supprimer** — 0 commit en 4 sprints, gelé par écrit jusqu'à S45 (`ROADMAP.md:175`) | On perd le comparatif concurrentiel. Il est daté (Q4), non rejouable, et il faudra le refaire de toute façon quand il servira |
| **DECISIONS_ARCHIVE.md** | 98 | **0** | **Supprimer** — sa seule fonction est que les renvois `DEC-xx` restent résolvables. `git log` le fait mieux | Un renvoi `DEC-64` dans un vieux document deviendra non résolvable en lecture directe. Contrepartie : chaque DEC contredit garde **une ligne** `⛔ SUPERSEDED PAR DEC-xxx` dans `DECISIONS.md` |
| **DECISIONS.md** | 222 | **130** | **Alléger** — un seul format de tableau (4 coexistent aujourd'hui), fusion des `DEC` qui ne font que répéter une règle d'`AGENTS.md` (`DEC-104`≡R7, `DEC-186`≡R4-ter, `DEC-169`≡R1), résolution de la collision `DEC-159` | On perd le « pourquoi ça casse » de ~40 décisions anciennes. Toutes concernent du code déjà stabilisé depuis ≥ 3 sprints |
| **ARCHITECTURE_TECHNIQUE.md** | 258 | **180** | **Alléger** — §5 graphe de dépendances et §9 pipeline de reco dupliquent `TYPES_CONTRACT §4` et `§9` | On perd la vue d'ensemble en un coup d'œil. Le registre des clés (§7) et la séquence de boot (§6), eux, sont intouchables |
| **PILOTAGE.md** | 206 | **140** | **Alléger** — H1→H9 se réduisent à 4 règles (chiffre unique, règle unique, plafond, purge) ; la carte des documents (§8) devient inutile à 9 documents | On perd la traçabilité fine des règles d'hygiène par numéro. Elles n'étaient de toute façon pas appliquées : H2, H7 et H8 sont violées aujourd'hui |
| **AGENTS.md** | 279 | **300** | **Absorber** ANTIPATTERNS §1-6 · **retirer** les 4 chiffres figés faux (§2.6) · remplacer le registre §7 par un renvoi à `package.json` (source unique exécutable) | Gemini perd le compte des specs par batch. C'est `package.json` qui décide réellement — le registre manuel a déjà dérivé |
| **CLAUDE.md** | 91 | **60** | **Alléger** — §4 « ce qui ne se réinvente jamais » duplique `AGENTS`/`TYPES_CONTRACT`/`ARCH_TECHNIQUE` | Aucun : §4 n'est opposable nulle part |
| **STATE.md** | 80 | **90** | **Conserver, cesser de régénérer** — absorbe le cap S45→S50 (11 l.) et le parking (6 l.). Devient **patché**, plus régénéré | Une régénération corrige les erreurs accumulées ; un patch les conserve. Contrepartie : une régénération complète, une fois par clôture de sprint |
| **HISTORIQUE.md** | 81 | **81** | **Conserver tel quel** — 1 ligne par session, 36 lignes de churn en 10 sessions. C'est le document le moins cher du corpus | — |
| **TYPES_CONTRACT.md** | 359 | **359** | **Conserver tel quel** — 8 lignes de churn en 4 sprints, et c'est le seul rempart contre l'invention de types dans une US autoportante | — |
| **ARCHITECTURE_FONCTIONNELLE.md** | 138 | **150** | **Absorber** la taxonomie EPICS | — |
| **AAA doc knowledge** | 10 | **0** | **Supprimer** — brouillon de 2026-08-20, jamais cité | Aucun |
| **TOTAL** | **3 206** | **1 580** | | **−51 %** |

---

## 5. MODE BÊTA — le chemin court

**Critère d'entrée (les deux à la fois, sinon process normal) :** (1) un testeur réel décrit un
comportement observé, et (2) le défaut empêche une action — perdre sa bibliothèque, ne pas
s'inscrire, écran blanc, ajout impossible. Un irritant visuel, une lenteur, une demande de
fonctionnalité n'entrent **jamais** : ils vont en slot flex.

**Ce qui est conservé — non négociable :**
1. **La porte verte du PO** (`type-check`, `test:run`, `build`) — c'est la seule preuve qui existe.
2. **Le périmètre déclaré** : les fichiers touchés sont listés avant, vérifiés par `git diff --name-only`.
3. **Une ligne** dans `HISTORIQUE.md` à la prochaine clôture : `HOTFIX-xx — symptôme → cause → fichier`.

**Ce qui saute :** l'US formelle autoportante · le test de fidélité écrit à l'avance · l'E2E R4 ·
la régénération de `STATE.md` · le passage par un slot de sprint · tout patch de gouvernance.

**Retour au process normal :** à la clôture de session suivante, le correctif est converti en
**une** US de consolidation qui n'apporte que le test manquant (E2E ou unitaire). Si ce test n'est
pas écrit à la session suivante, le mode bêta est suspendu jusqu'à ce qu'il le soit — c'est le
seul garde-fou contre l'usage du chemin court comme voie normale.

**Plafond :** 2 hotfix par sprint. Au troisième, le sprint s'arrête et on rouvre le Kanban.

---

## 6. DÉCISIONS POUR LE PO — questions fermées

1. **Est-ce que j'arrête de te faire coller un `STATE.md` entier à chaque fin de conversation, pour ne te faire coller que les 5-10 lignes qui ont changé ?** → **OUI.** (Économie mesurée : ~120 lignes collées par session.)
2. **Est-ce que je supprime `ROADMAP.md` et je remets le plan des prochains sprints en 11 lignes dans `STATE.md` ?** → **OUI.** (Il te fait aujourd'hui coller 668 lignes en 10 sessions, et il se contredit lui-même sur l'état de S43.)
3. **Est-ce que je vide `AUDIT.md` de tout ce qui est déjà réglé, pour ne garder que les problèmes encore ouverts ?** → **OUI.** (Tu passes de 586 à ~90 lignes ; ce qui disparaît reste retrouvable dans l'historique Git.)
4. **Est-ce que je supprime `BENCHMARK.md`, la comparaison avec les concurrents que personne n'a rouverte depuis 4 sprints ?** → **OUI.**
5. **Est-ce que je fusionne la liste des pièges (`ANTIPATTERNS.md`) dans le document que Gemini lit, pour qu'il arrête de les refaire ?** → **OUI.**
6. **Est-ce que je passe de 15 documents à 9, en fusionnant `EPICS` dans l'architecture fonctionnelle et en supprimant l'archive des décisions ?** → **OUI.**
7. **Est-ce que je m'engage à ne t'envoyer qu'UNE seule mise à jour de documentation par conversation, en toute fin, au lieu des 7 actuelles ?** → **OUI.**
8. **Est-ce que j'ouvre un chemin rapide qui te permet de corriger dans la journée un bug qui empêche un testeur d'utiliser l'app, sans écrire d'US complète ?** → **OUI.**
9. **Ce chemin rapide doit-il rester plafonné à 2 corrections par sprint, pour qu'il ne remplace pas le process normal ?** → **OUI.**
10. **Est-ce que je continue à te demander de faire relire chaque test par quelqu'un d'autre que celui qui a écrit le code ?** → **OUI, mais uniquement sur les écrans et la persistance.** (Sur les 26 dernières US, cette règle n'a jamais fait rougir une livraison de Gemini, alors qu'elle a produit 4 tests faux de mon côté — dont un vert qui ne testait rien. Elle reste justifiée là où un bug est invisible, pas partout.)
11. **Est-ce que j'arrête de numéroter les pièges `AP-xx` ?** → **OUI.** (Il en manque 8 numéros, un s'appelle littéralement `AP-TEST-x`, un autre est écrit deux fois. Ils sont cités par leur titre, jamais par leur numéro.)
12. **Est-ce que je garde la numérotation `DEC-xx` et `AUD-xx` ?** → **OUI.** (Elles servent réellement : tu les cites, les US les citent. Mais `DEC-159` désigne aujourd'hui deux décisions différentes — je le corrige.)
13. **Faut-il que je te dise, à chaque ouverture de conversation, quels documents se contredisent — au lieu de me contenter de vérifier les compteurs ?** → **OUI.** (6 contradictions actives aujourd'hui, dont une qui affirme que l'app est déjà en bêta alors qu'aucun testeur n'a encore ouvert l'app.)
14. **Est-ce que je supprime `ESLint` du discours tant qu'il n'est pas lançable ?** → **OUI.** (Il n'existe aucune commande pour le lancer dans le projet ; il est cité comme une règle depuis 4 sprints.)

---

## 7. UN POINT DE DÉSACCORD AVEC L'ÉNONCÉ DE LA MISSION

La mission pose que « l'app est EN BÊTA avec des utilisateurs réels depuis peu ».
**`STATE.md:71` dit le contraire** : « Bêta non ouverte — note aux testeurs prête, `AUD-42` reste
à revérifier. 27+ US livrées depuis S41, **zéro observation utilisateur**. Risque projet n°1. »
`DEC-120` (`DECISIONS.md:155`) dit encore autre chose : « L'app est en beta avec des testeurs
réels ».

Cela ne change rien au plan de coupe, mais cela change l'ordre des priorités : **le mode bêta
décrit en §5 n'a aujourd'hui aucun utilisateur à servir.** Le problème n°1 du projet n'est pas le
volume documentaire — c'est que 27 US ont été livrées et documentées sans qu'une seule personne
ait ouvert l'application. Réduire la documentation de 51 % fait gagner du temps ; envoyer la note
aux testeurs le fait gagner sur ce qui compte.

---

## PERSPECTIVE EXPÉRIENCE UTILISATEUR

L'utilisateur final ne voit rien de ce document. Ce qu'il voit, c'est **le débit** : sur les
10 dernières sessions, 31 US livrées et 2 430 lignes de documentation écrites. Chaque session
consacre l'équivalent de 120 lignes collées à décrire un état qui n'existera plus à la session
suivante — du temps qui ne va pas dans l'onboarding, le calendrier ou la synchronisation.

Le vrai coût UX est ailleurs, et il est chiffré ci-dessus : **zéro observation utilisateur sur
27 US livrées.** Le process est optimisé pour ne rien casser, pas pour apprendre. La coupe
proposée libère de la capacité de conversation ; le mode bêta (§5) transforme un retour de
testeur en correctif dans la journée au lieu d'une session. Ces deux changements servent le même
objectif : que le prochain sprint soit décidé sur ce que des gens font de l'application, et non
sur ce que la documentation dit d'elle.

---

*Fin du rapport. 🔻 Supprimer ce fichier après décision du PO.*
