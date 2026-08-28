# HISTORIQUE.md — Journal des sessions et des versions

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Créé en SE-062 par DEC-146**, par extraction des sections « Sessions » et « Versions »
> de `STATE.md`.
>
> **Rôle :** mémoire du **passé** clos. `STATE.md` porte le présent, ce fichier porte ce qui
> ne bouge plus.
>
> **Hors ordre de lecture obligatoire.** Comme `AUDIT.md` et `BENCHMARK.md`, Claude ne l'ouvre
> qu'à la demande. Il ne pollue pas l'index de la Knowledge.
>
> ## Règle de mise à jour — APPEND-ONLY
>
> | Événement | Action |
> |---|---|
> | Clôture de **session** | **+1 ligne** dans §Sessions. Rien d'autre. |
> | Clôture de **sprint** | +1 ligne dans §Sessions **et** +1 ligne dans §Versions. |
>
> **Ce fichier n'est JAMAIS régénéré, jamais réécrit, jamais réordonné.**
> C'est la parade structurelle à la perte d'information : un document qu'on n'écrase pas
> ne peut rien perdre. Coût : ~30 tokens par session.
>
> Purge H9 (tous les 5 sprints) : les lignes de plus de 5 sprints peuvent être condensées
> en une ligne de synthèse par sprint — jamais supprimées.

---

## 🕐 Sessions

> Ordre chronologique. On ajoute en bas.

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| ≤ SE-054 | ≤ S39 | ⚠️ Détail non capturé — antérieur à la création de ce fichier | — |
| SE-055 | S39 | `J05-E2E` · `US-SEARCH-SPECS-ANILIST` · `AUD-21` ouvert | 199 tests, 2 MERGE |
| SE-056 | S39 | `US-SEARCH-DEDUP-ANILIST` · `J06` · DEC-127→130 | 199 tests, 2 MERGE |
| SE-057 | S39 | `J07` synopsis · `AUD-21` faux jour · sweep E2E · **clôture S39** | v0.33.0, 205 tests, 2 MERGE |
| SE-058 | hors sprint | Tri A/B du benchmark · 3 corrections de faits · DEC-133 · création `BENCHMARK.md` | Aucun code |
| SE-059 | S40 | Planning S40 · `J08` fusionnée dans `J10` · `J09` coupée · `AUD-05` déclassée · `AUD-22` diagnostiqué · DEC-134→136 | 2 MERGE, 218 tests |
| SE-060 | S40 | `J10a` `J10b` · DEC-137→140 · `J10` éclatée en 5 slices | 2 MERGE |
| SE-061 | S40 | `J10d` · `J11a-1/2/3` · `useRecommendations` doté de ses 2 premières specs · DEC-141→145 | **4 MERGE au 1ᵉʳ coup**, 280 tests |
| **SE-062** | **S40** | **`J11b-1/2/3` — Jikan intégralement retiré du dépôt.** Dette `AGENTS.md` E2E classée (patch déjà déployé en `4b99eeb`, handoff SE-061 erroné). Sweep complet : 12 rouges, cause unique → epic `J12`. DEC-146→149. Création de ce fichier | **3 MERGE au 1ᵉʳ coup**, 265 tests, **Sprint Goal S40 atteint, sprint non clos** |
| **SE-063** | S40 | `J12-a` (helper `installAniListMock` + 2 specs pilotes) · `J12-b` (9 specs) | **2 MERGE au 1ᵉʳ coup, 0 correction.** Sweep **52/52 verts — premier sweep intégralement vert du projet.** Diagnostic du handoff SE-062 corrigé : aucune des 11 rouges ne routait `api.jikan.moe`. Découverte `AUD-24` (12 specs démockées) et `AUD-25`. DEC-150→153. **Sprint S40 CLOS** |
| **SE-063.b** | hors sprint | **Cleaning et compression de la Knowledge.** 12 documents réécrits, création de `DECISIONS_ARCHIVE.md` (satellite hors ordre de lecture), `AUDIT.md` enrichi en append. `DECISIONS.md` était collé deux fois dans lui-même (717 → 159 lignes). 15 informations périmées corrigées, dont la séquence de boot fausse depuis `J11b-1` et deux règles opposables d'`AGENTS.md` contradictoires. DEC-154 | **Aucun code.** Corpus 3441 → 2432 lignes (−29 %), ordre de lecture −37 % |
| **SE-064** | hors sprint | **Session P0 « zéro zone d'ombre ».** 22 constats tranchés par grep : 7 soldés (`AUD-02` forme d'origine, `09`, `11`, `14`, `19`, `28`, et `AUD-26` annulé comme faux positif), 7 confirmés vivants, 4 nouveaux (`AUD-27`, `29`, `30`, `31`). Atterrissage onboarding prouvé sain (spread dans `buildSeedEntry`) — trou de SE-063.b refermé. Micro-patch DEC-128 : stub `_startBackgroundRelationFetch` supprimé. `AGENTS.md` déployé à la racine. PI Planning S41→S50, création de `ROADMAP.md`. DEC-155→159 | **Aucune US livrée, aucun bump.** 265 tests, build 180 modules. **`AUD-30` : la persistance ne fonctionne pas — bloquant de lancement bêta, US n°1 de S41** |
| **SE-065** | S41 | Ouverture S41. Bug P0 de persistance **résolu** : 2 causes (cache local empoisonné au logout + lecture jamais rejouée après connexion). 3 US mergées (`US-PERSIST-P0b`, `P0a`, `P0a2`) + 2 micro-patchs. 265 → 271 tests. Vérifié en conditions réelles par le PO. `US-PERSIST-P0a` livrée verte sans effet (erreur de spec, branche jamais atteinte). |3 MERGE + 2 micro-patchs DEC-128. 271 tests / 30 fichiers, build 180 modules, commit 27ee0ca. Sprint non clos, 3/10. Deux constats bloquants bêta ouverts : AUD-32 (écriture Firestore refusée en production) et AUD-33 (AniList 429 au boot), aucun tranché
|SE-066 |	S41	| 4 US livrées et vérifiées en conditions réelles : US-ONBOARD-PERSIST-A (b509ca0), US-MALIMPORT-FIX (b469a18), US-MONTH-COMINGSOON (e84b2aa), US-SEASON-SKIP-SESSION (80f9d11). US-MONTH-FIX ANNULÉE sur captures du PO — 42 cellules × 4-6 entrées sur 390 px n'est pas réparable en CSS ; remplacée par un « Coming Soon » assumé. La conception initiale d'US-ONBOARD-PERSIST était morte-née : router.beforeEach s'exécute avant le montage d'App.vue, le store vaut [] à cet instant. Vraie cause triviale et ailleurs : AppHeader.vue:54-56 purgeait toutes les clés aanime_, flag d'onboarding compris. Le test rouge d'US-MALIMPORT-FIX était une erreur de spec, pas de code — le store a refusé la spec du Tech Lead et il avait raison. AUD-13 : clôture antérieure fausse, soldé pour de bon. AUD-37 et AUD-38 ouverts. US-SEASON-1TAP découpée en US-ADD-EXTRACT + US-SEASON-1TAP, glissées en S42. DEC-163→165 (patchs Knowledge appliqués en SE-067)	4 MERGE au 1ᵉʳ coup, streak Gemini 12, zéro correction. 291 tests / 32 fichiers, build 178 modules / 368,64 kB, commit 80f9d11. Sprint non clos, 7/10. Aucune fonctionnalité ajoutée — l'app a cessé de mentir sur quatre points
|SE-067	| S41	| Clôture S41. 3 US 🟠 mergées (US-ANILIST-QUEUE-A/B, US-SYNC-PRIORITY) + 4 micro-patchs DEC-128 (US-CARD-ORDER-A/B, US-FIRESTORE-LIMITS, US-REC-WHY-2LINES). Patchs Knowledge de SE-066 appliqués en ouverture. AUD-32 MORT après mesure console — il gelait le bloquant n°1 depuis 3 sessions sur une erreur jamais rejouée. Plafond Firestore 100 → 500 publié en console. AUD-33 diagnostiqué : quatre causes chaînées, dont l'attente d'un 429 faite DANS la file unique (gel de 120 s, d'où le réflexe F5). AUD-37 requalifié en symptôme d'AUD-33. US-ANILIST-QUEUE-B a introduit une famine ordonnancée, rattrapée par US-SYNC-PRIORITY dans la même session. Deux specs de ma main fausses, attrapées par Gemini (test T3 antidaté, puis rappel de US-MALIMPORT-FIX). Collision d'ID AUD-32/33 et en-tête DECISIONS.md périmé détectés et arbitrés. AUD-39 à AUD-44 ouverts. DEC-163→170	10 slots S41 consommés sur 10. 291 → 295 tests / 32 fichiers, build 178 modules / 368,90 kB, commit 9e3923e. Sprint S41 CLOS, v0.35.0. Aucun bloquant technique ne subsiste pour la bêta
| SE-068 | S42 | 5 US mergées (`US-SWEEP-S41`, `US-ADD-EXTRACT`, `US-ADD-TOAST-TRUTH`, `US-ONBOARD-EMPTY`, `US-ONBOARD-TOAST`) · 295 → 315 tests · sweep E2E rejoué 52/52 · `AUD-33` fermé sur preuve, `AUD-45` requalifié non-constat, `AUD-06` soldé, `AUD-23` caduc, `AUD-42` observé · `AUD-46` ouvert · `DEC-171` → `DEC-175` · `AP-METHOD-1` créé |
| SE-069 | S42 | Clôture S42. Diagnostic AUD-46 résolu par lecture (`position: static` annulait tout z-index). 6 US Gemini mergées au premier coup + 2 micro-patchs. 3 constats neufs ouverts par lecture de code (AUD-47, AUD-49, AUD-50). 331 tests / 38 fichiers. |
| **SE-071** | S43 | Rouge `modal-next-episode` **instruit et clos** : cause = absence de mock réseau, établie par variable unique. A révélé `AUD-54` (vue Semaine qui reconstruit ses cartes en synchro, confirmé à l'œil en 3G, requalifié 🟠 — le tap reste juste). 5 US MERGE : `US-DEMOCK-HELPER` (`MediaSeed` sait produire `studioName` et `startDate`), `US-DEMOCK-1d`, `US-DEMOCK-1b/c`, `US-DEMOCK-2a`, `US-DEMOCK-2b`. **Famille A d'`AUD-52` soldée : zéro spec ne connaît plus l'URL d'AniList.** 10 specs sur le helper unique. Les 6 patchs documentaires hérités de SE-070 appliqués. Zéro sollicitation de Gemini (2ᵉ session consécutive) |

---

## 📦 Versions

> Une ligne par bump. Un bump = une clôture de sprint.

| Version | Sprint | Livré |
|---|---|---|
| ≤ v0.21.0 | ≤ S21 | Migration vanilla → Vue 3, US-PINIA, US-JST, CI |
| — | S23→S27 | ⚠️ Détail non capturé |
| v0.28.0 | S28 | Epic Stats |
| v0.29.0 | S29→S33 | Refonte nav · US-127 · US-AUTH-LOGOUT |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT |
| v0.31.0 | S37 | US-GRID-FIX · US-MODAL-UNIFY annulée (DEC-110) |
| v0.32.0 | S38 | AUD-01 · AUD-02 · registre E2E · purge debug |
| v0.33.0 | S39 | Migration AniList `J02`→`J07` · recherche · synopsis nettoyé · `AUD-21` |
| **v0.34.0** | **S40** | Migration AniList `J09`→`J11b` — **Jikan intégralement retiré** — + `J12` harnais E2E AniList |
|v0.35.0|	S41|	Ce que j'ajoute reste — la persistance cesse de mentir	9e3923e	295 tests / 32 fichiers · 178 modules · 368,90 kB	10 US + 4 micro-patchs. Gain de fiabilité visible : la bibliothèque survit au logout, revient à la connexion, tient 500 titres, et le réseau ne gèle plus l'application. Zéro fonctionnalité ajoutée|
| v0.36.0 | S42 | **On peut nous faire confiance.** L'écran cesse de mentir sur l'état réel : le message d'ajout annonce l'onglet où l'anime a réellement atterri, la recherche est lisible et range au bon endroit, un anime déjà suivi disparaît de This Season sans rechargement, un second appareil annonce le compte retrouvé au lieu de rejouer l'inscription à l'aveugle. |
