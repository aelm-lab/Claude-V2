# AUDIT_PROCESS_S44.md

> 🔻 **DOCUMENT TEMPORAIRE — à supprimer après décision du PO.**
> Hors ordre de lecture, hors corpus, hors plafond H7.
> Mesures du 2026-09-02 sur `Claude-V2` @ `00f095a` et `A-Anime` @ `399892c`, historique git complet.
>
> **v2 — révisé après arbitrage du PO.** Deux fondations de la v1 sont tombées :
> **(1)** Claude Chat n'a **pas** accès à git — aucune suppression ne peut s'appuyer sur
> « la trace reste dans l'historique ». **(2)** Le PO confirme que **l'app est en bêta**.
> Les captures de la Knowledge du projet ont par ailleurs révélé le constat le plus important
> du rapport (§2.10), qui n'était pas mesurable depuis le dépôt.
>
> `OLD/` n'existe pas : supprimé par le commit `766ac65`, conformément à H6. Rien n'est inféré.

---

## 1. VERDICT

Le process n'est pas cassé. Il souffre de deux choses, dont une que je ne pouvais pas voir avant
tes captures : **la Knowledge réellement chargée ne correspond pas à celle que `CLAUDE.md`
décrit.** Deux documents déclarés obligatoires sont absents, deux documents déclarés « à la
demande » occupent 35 % de ce qui est chargé — et `HISTORIQUE.md`, le document conçu pour que
Claude Chat n'oublie pas, **n'est pas chargé**. Le reste est du volume : `STATE.md` et
`ROADMAP.md`, les deux seuls documents à régénération intégrale, produisent **79 % des patchs**
que tu colles. Les règles qui protègent le code sont peu nombreuses, tracées, et bon marché.

---

## 2. LES CHIFFRES

### 2.1 Corpus aujourd'hui

| Document | Lignes | Mots | Chargé ? |
|---|---:|---:|:--:|
| AUDIT.md | 586 | 8 005 | ✅ |
| TYPES_CONTRACT.md | 359 | 1 813 | ✅ |
| ROADMAP.md | 332 | 2 983 | ✅ |
| AGENTS.md | 279 | 3 122 | ✅ |
| ARCHITECTURE_TECHNIQUE.md | 258 | 1 819 | ✅ |
| DECISIONS.md | 222 | 6 047 | ✅ |
| ANTIPATTERNS.md | 217 | 3 330 | ✅ |
| PILOTAGE.md | 206 | 2 515 | ✅ |
| EPICS.md | 142 | 1 158 | ❌ |
| ARCHITECTURE_FONCTIONNELLE.md | 138 | 1 437 | ❌ |
| BENCHMARK.md | 117 | 1 424 | ❌ |
| DECISIONS_ARCHIVE.md | 98 | 1 393 | ❌ |
| CLAUDE.md | 91 | 938 | ✅ |
| HISTORIQUE.md | 81 | 1 603 | ❌ |
| STATE.md | 80 | 696 | ✅ |
| AAA doc knowledge | 10 | 37 | ❌ |
| **TOTAL** | **3 216** | **38 320** | **2 630 chargées (82 %)** |

### 2.2 Croissance (lignes au commit de clôture de sprint)

| Document | S38 | S40 | S42 | S44 | S40→S44 |
|---|---:|---:|---:|---:|---:|
| AUDIT.md | 194 | 280 | 499 | **586** | **×2,09** |
| ROADMAP.md | — | — | 128 | **332** | **×2,59** (S42→S44) |
| ANTIPATTERNS.md | 228 | 148 | 192 | 217 | ×1,47 |
| DECISIONS.md | 248 | 161 | 209 | 222 | ×1,38 |
| HISTORIQUE.md | — | 63 | 69 | 81 | ×1,29 |
| PILOTAGE.md | 243 | 195 | 195 | 206 | ×1,06 |
| AGENTS.md | 351 | 274 | 275 | 279 | ×1,02 |
| TYPES · ARCH_T · ARCH_F · EPICS · BENCHMARK · CLAUDE | 1 155 | 1 004 | 1 005 | 1 005 | **×1,00** |
| STATE.md | 193 | 133 | 121 | 80 | ×0,60 |
| **TOTAL** | **2 612** | **2 450** | **2 893** | **3 206** | **+31 %** |

**Deux documents ont doublé : `AUDIT.md` et `ROADMAP.md` — +510 lignes sur les +756 de croissance
S40→S44, soit 67 %.** Ce sont aussi les deux que `CLAUDE.md` déclare « hors ordre de lecture ».

### 2.3 Volume de patchs (SE-064 → SE-073, 2026-08-20 → 2026-09-01)

| Mesure | Valeur |
|---|---|
| Commits Knowledge | 68 — **1,01 fichier par commit** |
| Commits Knowledge par session | **6,8 en moyenne** (min 2, max 10) |
| Lignes de doc écrites ou réécrites | **2 366** (1 530 ajoutées / 836 supprimées) |
| US livrées (`HISTORIQUE.md`) | 31 — SE-064 en a livré 0 (hors sprint) |
| **Lignes de doc par US livrée** | **76** |
| **Patchs doc par US livrée** | **2,2** |
| Micro-commits (< 5 lignes) | **20 / 68 = 29,4 %** — fichier n°1 : `HISTORIQUE.md` (7) |

### 2.4 Qui produit le volume

| Document | Lignes réécrites | Part |
|---|---:|---:|
| **STATE.md** | **1 203** | **51 %** |
| **ROADMAP.md** | **668** | **28 %** |
| AUDIT.md | 253 | 11 % |
| DECISIONS.md · ANTIPATTERNS.md | 146 | 6 % |
| HISTORIQUE · AGENTS · PILOTAGE · TYPES · DEC_ARCHIVE · CLAUDE | 82 | 3 % |
| **EPICS · BENCHMARK · ARCH_TECHNIQUE · ARCH_FONCTIONNELLE** | **0** | **0 %** |

`STATE.md` fait 80 lignes et a produit 1 203 lignes de churn : **réécrit 15 fois son volume en
10 sessions.** À deux, `STATE.md` et `ROADMAP.md` = **79 % de tout ce que tu colles.**

### 2.5 Dépôt de code A-Anime (49 commits, SE-066 → SE-073)

| Mesure | Valeur |
|---|---|
| Fichiers par commit | moyenne **1,78** · médiane 1 · max 4 |
| Dépassements de la règle « max 3 fichiers » | **2 / 49 = 4 %** |
| Churn par commit | moyenne 69 lignes · médiane 38 |
| Commits par US | 49 pour 28 US ≈ **1,75** |
| Délai ouverture → merge | **non datable** — aucun commit ne porte d'ID d'US. Proxy : le Kanban ne reporte aucune US d'une session à l'autre → même session |
| Churn code / churn doc | 3 380 / **1 891 = la doc pèse 56 % du code** |
| Commits code / commits doc | 49 / **51 — la Knowledge est commitée plus souvent que le produit** |

### 2.6 `AGENTS.md` — synchronisation et chiffres figés

Contenu **identique** entre les deux dépôts, à un saut de ligne final près. Aucune divergence de
fond. Mais les chiffres figés — l'exception assumée à H1, parce que Gemini n'a pas `STATE.md` :

| Ligne | Affirmation | Réalité mesurée |
|---|---|---|
| `AGENTS.md:233` | « Dernier sweep : **52 / 52** » | `STATE.md:18` : 55/55 en SE-072 |
| `AGENTS.md:204` · `:206` | « batch3 **(8)** » · « batch5 **(8)** » | `package.json` : **9** chacun |
| `AGENTS.md:72` | « `makeAnime()` **n'existe plus** » | présent dans **5 fichiers** de `src/**/*.spec.ts` |
| `AGENTS.md:198` | « chemin complet obligatoire, jamais le nom nu » | **43 des 45 entrées du §7 sont en nom nu** |
| `AGENTS.md:200` | « 45 specs / 45 enregistrées » | ✅ exact — vérifié sur disque et dans `package.json` |

### 2.7 Contradictions actives (fichier:ligne des deux occurrences)

| Information | Occurrence 1 | Occurrence 2 |
|---|---|---|
| Helper `makeAnime` | `AGENTS.md:72` « n'existe plus » | `ANTIPATTERNS.md:30` « → factory `makeAnime(...)` » |
| État de la bêta | `DECISIONS.md:155` « en beta avec testeurs réels » ✅ *confirmé par le PO* | `STATE.md:71` « Bêta non ouverte » ❌ **à corriger** |
| État de S43 | `ROADMAP.md:25` « ✅ CLOS — v0.37.0 » | `ROADMAP.md:67` « EN COURS depuis SE-070 » |
| État d'`AUD-52` | `AUDIT.md:552` « SOLDÉ » | `ROADMAP.md:258` « 🔴 Ouvert » |
| Nombre de specs | `STATE.md:19` « 45 » | `HISTORIQUE.md:55,80` « 43 specs » |
| Plafond H7 respecté | `STATE.md:30` « 0 document au-dessus » | `ROADMAP.md` = **332 lignes**, plafond 250, aucune exception nommée |

Doublons purs : R4-ter (`AGENTS.md:117` ≡ `DECISIONS.md:219`) · format de réponse au PO
(`PILOTAGE.md:5` ≡ `DECISIONS.md:222`) · R7 en **5 exemplaires** (`AGENTS.md:127`,
`PILOTAGE.md:77`, `CLAUDE.md:81`, `DECISIONS.md:126`, `ROADMAP.md:217`) · clé `aanime_calendar`
en **7 exemplaires** · registre des batchs E2E (`AGENTS.md:202` ≡ `package.json:14`, **déjà dérivé**).

### 2.8 Défauts structurels vérifiés

| Constat | Preuve |
|---|---|
| `DEC-159` désigne **deux décisions différentes** | `DECISIONS.md:169` et `:171` |
| `AUD-32` / `AUD-33` attribués deux fois | note de réconciliation `AUDIT.md:383-401` |
| Un antipattern jamais numéroté | `ANTIPATTERNS.md:119` — littéralement `AP-TEST-x` |
| `AP-PROCESS-4` écrit deux fois | `ANTIPATTERNS.md:152` et `:193` |
| Deux sections `## 9` | `ANTIPATTERNS.md:159` et `:168` |
| Trous AP | ni `AP-PROCESS-1`, ni `AP-TEST-1..4`, ni `AP-DIAG-1/2` |
| `AUD-53` hors registre · `AUD-21`/`AUD-22` cités jamais écrits | `ROADMAP.md:254` · `HISTORIQUE.md:36,38` |
| `SE-070` absente du journal append-only | `HISTORIQUE.md` saute SE-069 → SE-071, alors que `:52` la cite |
| `AUDIT.md` organisé par session, pas par thème | 13 sections « Campagne SE-0xx » — H8 violée |
| Deux campagnes titrées « SE-065 » | `AUDIT.md:336` et `:361` |
| `AUDIT.md:13` « aucun autre document ne référence celui-ci » | faux — cité par 5 documents |
| `ROADMAP.md` absent de la carte des documents | `PILOTAGE.md:180-196` en liste 14, pas lui |
| `DECISIONS.md` mélange 4 formats | tableau (l.15) · texte tabulé (l.165) · puces (l.183) · en-têtes orphelins (l.217, l.220) |
| ESLint non lançable | aucun script `lint` dans `package.json` |

### 2.9 Classement des règles — A protège · B théorique · C coûteuse · D morte

| Règle | Cl. | Preuve |
|---|:--:|---|
| `R-SCOPE-1` fichiers listés | **A** | 5 fichiers hors US → 17/17 E2E cassés (`AGENTS.md:54`) |
| `R-CODE-7` contrat d'event | **A** | `RecCard` : 3 emits morts sur 4 consommateurs, 0 erreur console |
| `R-LIVRAISON-2` sortie brute | **A** | récidives n°1 et n°7 — « 81 passed » structurellement impossible |
| Helper de mock unique · registre des batchs | **A** | 23 specs → 11 rouges d'un coup ; une spec non enregistrée ne tourne jamais |
| Remesure des faits externes (`PILOTAGE §6`) | **A** | 5 sprints gelés sur un curl jamais rejoué |
| `R1` triple preuve machine PO | **A** | build Gemini divergent de 71 kB sur 3 livraisons (`DECISIONS.md:181`) |
| `R-CODE-1` zéro `any` | **B** | jamais vérifié — pas de `lint`. Tenu par `vue-tsc` seul |
| North Star (TTFA, adds, jours-retour) | **B** | `PILOTAGE.md:104` : « leurs valeurs vivent dans `STATE.md` » — **aucune n'y figure** |
| Régénération intégrale `STATE.md` (`DEC-146`) | **C** | 1 203 lignes de churn pour 80 lignes de fichier |
| Régénération intégrale `ROADMAP.md` | **C** | 668 lignes, et il se contredit quand même (`:25` vs `:67`) |
| Numérotation `AP-xx` | **C** | 8 trous, un `AP-TEST-x`, un doublon — 0 incident évité |
| `R7` auteur test ≠ auteur code | **C** | 0 rouge légitime sur 26 US · 4 specs fausses côté Tech Lead |
| Chiffres figés `AGENTS.md` | **C** | 4 des 6 sont faux (§2.6) |
| `H2` backlog unique · `H7` plafond · `H8` titres thématiques · `H9` purge | **D** | les 4 sont violées aujourd'hui (§2.7, §2.8) |
| `DEC-02` ESLint en erreur | **D** | aucun script `lint` ; `STATE.md:23` : « jamais exécuté » |
| `R-CODE-8` aucun `<style scoped>` | **D** | 19 composants en portent ; `DEC-185` est né dans l'un d'eux |

**Test de fidélité (R7) — taux de détection réel : 0 sur 26.** `STATE.md:40` porte un streak de
26 US mergées au premier coup ; `HISTORIQUE.md:42,44,48,51` le confirment. Sur la même période,
**4 specs écrites par le Tech Lead se sont révélées fausses** : `US-PERSIST-P0a` verte et sans
effet (`AUDIT.md:380`), `US-MALIMPORT-FIX` rouge à tort (`HISTORIQUE.md:48`), deux autres
attrapées par Gemini en SE-067 (`HISTORIQUE.md:49`). La règle n'a rien attrapé et a produit
4 faux signaux. Elle reste justifiée là où un bug est invisible au vert — écrans et persistance.

### 2.10 🔴 Ce qui est réellement chargé dans la Knowledge

Mesuré sur les captures du sélecteur GitHub. **10 fichiers sur 16, 5 % de la capacité du projet.**

| | Déclaré par `CLAUDE.md §3` | Réellement chargé |
|---|---|---|
| `CLAUDE` · `STATE` · `PILOTAGE` · `TYPES_CONTRACT` · `DECISIONS` · `ANTIPATTERNS` · `AGENTS` · `ARCHITECTURE_TECHNIQUE` | ordre de lecture | ✅ |
| **`EPICS.md`** | **ordre de lecture, position 4** | ❌ **absent** |
| **`ARCHITECTURE_FONCTIONNELLE.md`** | **ordre de lecture, position 5** | ❌ **absent** |
| **`AUDIT.md` (586 l.)** | satellite, « **PAS lu au démarrage** » (`AUDIT.md:5`) | ✅ **toujours chargé** |
| **`ROADMAP.md` (332 l.)** | satellite, « ouvert à deux moments seulement » | ✅ **toujours chargé** |
| **`HISTORIQUE.md`** | satellite — **la mémoire des sessions closes** | ❌ **absent** |
| `BENCHMARK` · `DECISIONS_ARCHIVE` · `AAA doc knowledge` | satellites | ❌ absent |

Trois conséquences mesurables :
1. **Les deux documents absents de l'ordre de lecture sont exactement ceux à 0 ligne de churn en
   4 sprints** (§2.4). Ils ne sont pas stables : ils sont **invisibles**.
2. **`AUDIT` + `ROADMAP` = 918 lignes = 35 % de tout ce qui est chargé**, alors que les deux
   documents portent en en-tête qu'ils ne doivent pas l'être.
3. **Tu crains que Claude Chat oublie ce qu'on a fait — et `HISTORIQUE.md`, le document écrit
   pour ça, est le seul journal que tu ne lui donnes pas.** Il coûte 81 lignes.

La capacité n'est pas le problème : **5 % utilisés**. Ce n'est donc pas « ne pas tout donner » qui
est bien ou mal — c'est que **la sélection contredit `CLAUDE.md`, et personne ne peut le voir
depuis l'intérieur d'une session.**

---

## 3. LES 5 PROBLÈMES RÉELS (coût décroissant)

**P1 — La Knowledge chargée ne correspond pas à la Knowledge décrite (§2.10).** Claude Chat
confirme chaque session « lecture faite » sur un ordre de lecture dont 2 documents sur 10 sont
absents. C'est le seul problème du rapport qui produit des erreurs de raisonnement, pas seulement
du volume.

**P2 — La régénération intégrale de `STATE.md` et `ROADMAP.md` : 79 % des patchs.** 1 203 + 668
lignes en 10 sessions, pour deux documents qui ne portent aucune connaissance durable. C'est
exactement ta douleur déclarée, et elle ne vient ni des règles de code ni du corpus.

**P3 — 2,2 patchs et 76 lignes de doc par US livrée.** `PILOTAGE.md:141-152` impose 3 écritures
minimum par clôture plus l'isolation d'`AGENTS.md` en commit seul. Résultat : 6,8 commits par
session, dont **29,4 % changent moins de 5 lignes**. Le contenu est bon marché ; c'est le geste
qui te coûte.

**P4 — 6 contradictions actives, aucune détectée par le process.** Les règles H1→H9 vérifient des
**compteurs**, pas des **faits**. `AGENTS.md:72` interdit à Gemini un helper présent dans
5 fichiers ; `STATE.md:30` affirme « 0 document au-dessus du plafond » alors que `ROADMAP.md` le
dépasse de 82 lignes.

**P5 — La bêta est ouverte et le process ne la voit pas.** `STATE.md:71` affirme encore « Bêta
non ouverte, zéro observation utilisateur ». Aucune classe de risque ne couvre un défaut observé
en production, et le Kanban n'a pas d'état pour un retour testeur en cours de qualification.

---

## 4. PLAN DE COUPE — arbitré

Sous tes arbitrages (rien de supprimé qui porte de la mémoire, `BENCHMARK` conservé pour S45,
`ANTIPATTERNS` non fusionné), la coupe passe de **−51 % à −33 %**. C'est le prix de tes
contraintes, et il est défendable — mais tu dois savoir ce que chacune coûte.

| Document | Avant | Après | Action | Risque accepté |
|---|---:|---:|---|---|
| **AUDIT.md** | 586 | **200** | **Alléger sans rien perdre** : les **27 constats clos** passent à 1 ligne chacun (`AUD-xx ✅ verdict en une phrase`) ; les **28 ouverts** gardent leur détail intégral. Réorganisé par thème, plus par campagne | Tu perds le *récit* du diagnostic des constats clos, pas leur existence ni leur verdict. Si un constat clos rouvre, il est rediagnostiqué |
| **ROADMAP.md** | 332 | **70** | **Alléger, pas supprimer** — tu as raison sur le découpage. Garder §1 (cap S45→S50) et §6 (parking). Supprimer §2 (duplique le Kanban et `HISTORIQUE`), §3 (duplique `PILOTAGE` et `DECISIONS`), §4 (duplique `AUDIT`), §5 (c'est un livrable, pas un document) | Tu perds les justifications détaillées de S42/S43, déjà closes. Bénéfice : le document repasse sous 250 lignes — l'objectif pour lequel tu l'avais découpé, aujourd'hui manqué de 82 lignes |
| **DECISIONS.md** | 222 | **150** | **Alléger** : un seul format (4 coexistent), collision `DEC-159` résolue, les `DEC` qui répètent une règle d'`AGENTS` gardent leur ID + un renvoi d'une ligne | Tu perds le « pourquoi ça casse » d'une quarantaine de décisions anciennes, toutes sur du code stable depuis ≥ 3 sprints |
| **ANTIPATTERNS.md** | 217 | **150** | **Alléger, pas fusionner** — d'accord avec toi. Retirer uniquement ce qui est **mot pour mot** dans `AGENTS §4` (§1 et §2), garder tout le reste. Numérotation `AP-xx` retirée | Aucun : ce qui part est lu par Gemini dans `AGENTS.md` |
| **EPICS.md** | 142 | **0** | **Fusionner dans `ARCHITECTURE_FONCTIONNELLE.md`** — voir §6 Q1 | Aucun : les deux documents décrivent le même axe (le « où » fonctionnel) et aucun des deux n'est chargé aujourd'hui |
| **ARCHITECTURE_FONCTIONNELLE.md** | 138 | **175** | Absorbe la taxonomie des 12 EPICs · **et à charger dans la Knowledge** | — |
| **ARCHITECTURE_TECHNIQUE.md** | 258 | **190** | **Alléger** — §5 (graphe de dépendances) et §9 (pipeline de reco) dupliquent `TYPES_CONTRACT §4` et `§9` | Tu perds la vue d'ensemble en un coup d'œil. Le registre des clés (§7) et la séquence de boot (§6) sont intouchables |
| **PILOTAGE.md** | 206 | **150** | **Alléger** — H1→H9 se réduisent à 4 règles ; la carte des documents (§8) devient inutile | Tu perds la traçabilité par numéro de règle d'hygiène. H2, H7, H8 et H9 sont de toute façon violées aujourd'hui |
| **AGENTS.md** | 279 | **265** | Retirer les **4 chiffres figés faux** (§2.6) ; le registre §7 devient un renvoi à `package.json`, seule source exécutable | Gemini perd le compte de specs par batch. C'est `package.json` qui décide réellement — le registre manuel a déjà dérivé |
| **CLAUDE.md** | 91 | **60** | **Alléger** — §4 duplique `AGENTS` / `TYPES_CONTRACT` / `ARCH_TECHNIQUE`. **L'ordre de lecture est réaligné sur ce qui est vraiment chargé** | Aucun |
| **STATE.md** | 80 | **85** | **Régénéré à la clôture de sprint, patché entre-temps.** Absorbe le cap S45→S50 si `ROADMAP` te gêne toujours | Un patch conserve les erreurs qu'une régénération corrigeait. Contrepartie : la régénération de fin de sprint les efface |
| **HISTORIQUE.md** | 81 | **81** | **Conserver et CHARGER** — c'est la mémoire, 3,5 lignes de churn par session | — |
| **TYPES_CONTRACT.md** | 359 | **359** | **Ne pas toucher** — 8 lignes de churn en 4 sprints, seul rempart contre l'invention de types | — |
| **BENCHMARK.md** | 117 | **117** | **Conserver** pour S45, comme tu l'as tranché. Non chargé → coût nul aujourd'hui | — |
| **DECISIONS_ARCHIVE.md** | 98 | **98** | **Conserver** — sans git, c'est la seule trace des décisions périmées. Non chargé → coût nul | — |
| **AAA doc knowledge** | 10 | **0** | **Supprimer** — brouillon du 2026-08-20, jamais cité | Aucun |
| **TOTAL** | **3 216** | **2 150** | | **−33 %** |
| **dont chargé en session** | **2 630** | **1 936** | 12 documents au lieu de 10 | **−26 %, et les 2 documents manquants enfin présents** |

**Ce que tes arbitrages coûtent, chiffré :** garder `ANTIPATTERNS` séparé = +150 · garder la
mémoire complète d'`AUDIT` = +110 · garder `ROADMAP` = +70 · garder `BENCHMARK` = +117 ·
garder `DECISIONS_ARCHIVE` = +98. Total **+545 lignes** par rapport au plan v1. Les trois
derniers ne sont pas chargés : ils ne te coûtent rien en session, seulement en dépôt.

---

## 5. RETOURS BÊTA — sans cérémonie nouvelle

Tu as tranché : pas de chemin hotfix, on utilise les 3 slots flex de `DEC-155`. C'est le bon
choix — mais il manque **deux lignes** pour que ça marche, parce qu'aujourd'hui rien ne définit
ce qui entre en flex.

**Critère d'entrée en flex (à graver dans `DEC-155`) :** un retour testeur entre en flex si le
défaut **empêche une action** — perdre sa bibliothèque, ne pas s'inscrire, écran blanc, ajout
impossible. Sinon il va dans `ROADMAP §1`. Un irritant visuel n'est jamais un flex.

**Qualification, en conversation, sans document :** tu colles le retour brut. Claude répond en
3 lignes — *ce que le testeur a vécu · ce qui casse dans le code · slot flex ou cap*. C'est
`DEC-157` tel qu'il est déjà écrit ; il suffit de l'appliquer.

**Ce qui ne change pas :** l'US formelle, le test, la gate `R1`. Un bug bêta prend un slot, pas un
raccourci — c'est ce que tu as décidé et c'est cohérent avec 26 US mergées au premier coup.

**Le seul vrai trou :** `STATE.md:71` dit encore « Bêta non ouverte ». Tant que cette ligne est
là, chaque nouvelle session de Claude Chat repart de l'idée qu'aucun testeur n'existe.

---

## 6. DÉCISIONS RESTANTES POUR LE PO

1. **Est-ce que je charge `HISTORIQUE.md` dans la Knowledge — le journal des sessions passées, 81 lignes, que tu ne lui donnes pas aujourd'hui alors que c'est exactement ce que tu crains de perdre ?** → **OUI, en priorité n°1.**
2. **Est-ce que je fusionne `EPICS.md` dans `ARCHITECTURE_FONCTIONNELLE.md`, sous ce nom-là, et que je charge le résultat ?** → **OUI — pas dans `ROADMAP`.** `ROADMAP` répond à *quand*, `ARCHITECTURE_FONCTIONNELLE` répond à *où dans l'app*. `EPICS` répond à *où dans l'app* : c'est le même axe. Les mettre dans `ROADMAP` mélangerait le temps et l'espace, et c'est précisément ce qui a fait gonfler `ROADMAP` à 332 lignes.
3. **Est-ce que je décharge `AUDIT.md` et `ROADMAP.md` de la Knowledge permanente, comme leurs propres en-têtes le demandent, pour ne les recoller qu'en composition de sprint ?** → **NON, pas tant qu'ils ne sont pas allégés.** Une fois `AUDIT` à 200 lignes et `ROADMAP` à 70, ils coûtent peu : laisse-les chargés. Décharger un document dont Claude a besoin coûte plus cher que de le porter.
4. **Est-ce que je corrige `STATE.md:71` pour dire que la bêta est ouverte ?** → **OUI, immédiatement** — c'est la ligne qui fait raisonner chaque nouvelle session sur un produit sans utilisateurs.
5. **Est-ce que `STATE.md` n'est régénéré entièrement qu'à la clôture de sprint, et simplement patché de 5-10 lignes entre-temps ?** → **OUI.** C'est ta réponse à la Q1 traduite en règle : ce n'est pas le collage qui te dérange, c'est sa fréquence.
6. **Combien de collages par session tu acceptes ?** → **Ma recommandation : 2 maximum en session normale, 1 seul dans la plupart des cas** — `HISTORIQUE` (+1 ligne) et, si vraiment nécessaire, un seul autre document. Tout le reste attend la clôture de sprint. Voir les trois options ci-dessous.
7. **Est-ce que j'ajoute la commande manquante pour lancer ESLint ?** → **OUI.** ESLint est un correcteur automatique de style et d'erreurs de code. Il est configuré dans le projet depuis le début, mais **il n'existe aucune commande pour le lancer** — il n'a donc jamais tourné une seule fois. C'est lui qui vérifierait la règle « zéro `any` » que rien ne vérifie aujourd'hui. Coût : une US de 10 minutes, 1 fichier.

### Les trois options pour la Q6

| Option | Ce que tu colles | Coût | Verdict |
|---|---|---|---|
| **A — Cadence sprint** | 1 ligne d'`HISTORIQUE` par session · tout le reste (STATE, DECISIONS, AUDIT…) en **un seul lot à la clôture du sprint** | Si une conversation meurt en cours de sprint, l'état de cette session tient dans la ligne d'`HISTORIQUE` et rien d'autre | ✅ **Recommandée.** Passe de 6,8 à **~1,3 collage par session** |
| **B — Un document par session** | Le document le plus urgent, un seul, jamais deux | Le second document attend, la dette documentaire revient | ⚠️ Fonctionne, mais réintroduit exactement le report de patchs que `PILOTAGE.md:144` interdit |
| **C — Passer par Claude Code** | Rien. Claude Chat produit le lot en fin de session, tu me le transmets, j'applique et je pousse dans le dépôt | Ajoute un 4ᵉ acteur et une session de plus à ouvrir | 🔵 Le plus économe pour toi (**0 collage**), le plus lourd en coordination. À garder pour les gros lots — le nettoyage du corpus lui-même, par exemple |

---

## PERSPECTIVE EXPÉRIENCE UTILISATEUR

L'utilisateur ne verra jamais ce document. Ce qu'il subit, c'est le débit : 31 US et 2 366 lignes
de documentation sur 10 sessions, dont 79 % pour décrire un présent périmé à la session suivante.

Mais le vrai coût UX n'est pas là, et il vient de changer de nature : **la bêta est ouverte.** Le
process a été conçu pour ne rien casser sur un produit sans utilisateurs ; il n'a aucun mécanisme
pour *apprendre* de ceux qui viennent d'arriver. `STATE.md` porte 3 slots flex et une ligne qui
dit encore qu'il n'y a personne. Les métriques North Star — temps pour suivre son premier anime,
jours de retour la première semaine — sont déclarées depuis 4 sprints dans `PILOTAGE.md:98-102`
et **aucune n'a jamais reçu de valeur**.

La coupe libère de la capacité. Ce qui compte davantage : que la première chose remontée par un
testeur trouve un slot le jour même, et que le sprint suivant se compose sur ce que des gens font
de l'application — pas sur ce que la documentation dit d'elle.

---

*Fin du rapport. 🔻 Supprimer ce fichier après décision du PO.*
