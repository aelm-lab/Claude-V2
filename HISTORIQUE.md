# HISTORIQUE.md — Journal des sessions, tampon de sprint, versions

> **Rôle :** la **mémoire**. `STATE.md` porte le présent, ce fichier porte ce qui ne bouge plus —
> plus le **tampon** de ce qui vient d'être décidé et n'a pas encore rejoint son document.
>
> 🔴 **C'est le seul document touché en cours de sprint** (`DEC-190`). À chaque clôture de
> session, le PO colle **un unique bloc à la fin de ce fichier** : 1 ligne dans `§1 Sessions`,
> plus 1 ligne par élément neuf dans `§2 Tampon`.
>
> **`§1 Sessions` et `§3 Versions` sont APPEND-ONLY.** Jamais régénérés, jamais réécrits, jamais
> réordonnés. C'est la parade structurelle à la perte d'information : un document qu'on n'écrase
> pas ne peut rien perdre. Coût mesuré : ~3,5 lignes par session.
>
> **`§2 Tampon` est la seule partie vidée** — par le lot documentaire de clôture de sprint, quand
> ses lignes ont rejoint `DECISIONS.md`, `AUDIT.md` ou `ANTIPATTERNS.md`.
>
> **Purge H8 (tous les 5 sprints) :** les lignes de plus de 5 sprints peuvent être condensées en
> une ligne de synthèse par sprint — **jamais supprimées**.

---

## 1. 🕐 Sessions

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
| **SE-062** | S40 | **`J11b-1/2/3` — Jikan intégralement retiré du dépôt.** Dette `AGENTS.md` E2E classée (patch déjà déployé en `4b99eeb`, handoff SE-061 erroné). Sweep complet : 12 rouges, cause unique → epic `J12`. DEC-146→149. Création de ce fichier | **3 MERGE au 1ᵉʳ coup**, 265 tests, **Sprint Goal S40 atteint, sprint non clos** |
| **SE-063** | S40 | `J12-a` (helper `installAniListMock` + 2 specs pilotes) · `J12-b` (9 specs). Diagnostic du handoff SE-062 corrigé : aucune des 11 rouges ne routait `api.jikan.moe`. Découverte `AUD-24` (12 specs démockées) et `AUD-25`. DEC-150→153 | **2 MERGE au 1ᵉʳ coup, 0 correction.** Sweep **52/52 verts — premier sweep intégralement vert du projet. Sprint S40 CLOS** |
| **SE-063.b** | hors sprint | **Cleaning et compression de la Knowledge.** 12 documents réécrits, création de `DECISIONS_ARCHIVE.md`, `AUDIT.md` enrichi en append. `DECISIONS.md` était collé deux fois dans lui-même (717 → 159 lignes). 15 informations périmées corrigées, dont la séquence de boot fausse depuis `J11b-1` et deux règles opposables d'`AGENTS.md` contradictoires. DEC-154 | **Aucun code.** Corpus 3441 → 2432 lignes (−29 %) |
| **SE-064** | hors sprint | **Session P0 « zéro zone d'ombre ».** 22 constats tranchés par grep : 7 soldés (`AUD-02` forme d'origine, `09`, `11`, `14`, `19`, `28`, `AUD-26` annulé), 7 confirmés vivants, 4 nouveaux (`AUD-27`, `29`, `30`, `31`). Atterrissage onboarding prouvé sain (spread dans `buildSeedEntry`). Micro-patch `DEC-128` : stub `_startBackgroundRelationFetch` supprimé. PI Planning S41→S50, création de `ROADMAP.md`. DEC-155→159 | **Aucune US, aucun bump.** 265 tests. **`AUD-30` : la persistance ne fonctionne pas — bloquant bêta, US n°1 de S41** |
| **SE-065** | S41 | Ouverture S41. Bug P0 de persistance **résolu** : 2 causes (cache local empoisonné au logout + lecture jamais rejouée après connexion). `US-PERSIST-P0a` livrée verte **sans effet** (erreur de spec, branche jamais atteinte) → `AP-PROCESS-5`. Vérifié en conditions réelles par le PO | 3 MERGE + 2 micro-patchs. 265 → 271 tests / 30 fichiers, `27ee0ca`. 3/10. Deux bloquants bêta ouverts : `AUD-32`, `AUD-33` |
| **SE-066** | S41 | 4 US livrées et vérifiées en réel : `US-ONBOARD-PERSIST-A` (`b509ca0`), `US-MALIMPORT-FIX` (`b469a18`), `US-MONTH-COMINGSOON` (`e84b2aa`), `US-SEASON-SKIP-SESSION` (`80f9d11`). `US-MONTH-FIX` **annulée** sur captures PO — 42 cellules × 4-6 entrées sur 390 px n'est pas réparable en CSS ; remplacée par un « Coming Soon » assumé. La conception initiale d'`US-ONBOARD-PERSIST` était morte-née : `router.beforeEach` s'exécute avant le montage d'`App.vue`, le store vaut `[]` à cet instant ; vraie cause ailleurs — `AppHeader.vue:54-56` purgeait toutes les clés `aanime_`, flag d'onboarding compris. Le test rouge d'`US-MALIMPORT-FIX` était **une erreur de spec, pas de code**. `AUD-13` : clôture antérieure fausse, soldé pour de bon. `AUD-37`/`AUD-38` ouverts. DEC-163→165 | **4 MERGE au 1ᵉʳ coup**, streak 12, zéro correction. 291 tests / 32 fichiers, `80f9d11`. 7/10. Aucune fonctionnalité ajoutée — l'app a cessé de mentir sur quatre points |
| **SE-067** | S41 | **Clôture S41.** 3 US 🟠 (`US-ANILIST-QUEUE-A/B`, `US-SYNC-PRIORITY`) + 4 micro-patchs `DEC-128`. **`AUD-32` MORT après mesure console** — il gelait le bloquant n°1 depuis 3 sessions sur une erreur jamais rejouée. Plafond Firestore 100 → 500 publié. `AUD-33` diagnostiqué : quatre causes chaînées, dont l'attente d'un 429 faite DANS la file unique (gel de 120 s, d'où le réflexe F5). `AUD-37` requalifié en symptôme d'`AUD-33`. `US-ANILIST-QUEUE-B` a introduit une famine ordonnancée, rattrapée par `US-SYNC-PRIORITY` dans la même session. **Deux specs de ma main fausses, attrapées par Gemini.** Collision d'ID `AUD-32/33` arbitrée. `AUD-39`→`AUD-44` ouverts. DEC-166→170 | 10/10 slots. 295 tests / 32 fichiers, `9e3923e`. **Sprint S41 CLOS, v0.35.0.** Aucun bloquant technique ne subsiste pour la bêta |
| SE-068 | S42 | 5 US mergées (`US-SWEEP-S41`, `US-ADD-EXTRACT`, `US-ADD-TOAST-TRUTH`, `US-ONBOARD-EMPTY`, `US-ONBOARD-TOAST`) · 295 → 315 tests · sweep E2E rejoué 52/52 · `AUD-33` fermé sur preuve, `AUD-45` requalifié non-constat, `AUD-06` soldé, `AUD-23` caduc, `AUD-42` observé · `AUD-46` ouvert · `DEC-171`→`DEC-175` · `AP-METHOD-1` créé | 5 MERGE |
| SE-069 | S42 | **Clôture S42.** Diagnostic `AUD-46` résolu par lecture (`position: static` annulait tout `z-index`). 6 US Gemini mergées au premier coup + 2 micro-patchs. 3 constats neufs ouverts par lecture de code (`AUD-47`, `AUD-49`, `AUD-50`) | 331 tests / 38 fichiers. **v0.36.0** |
| **SE-070** | S43 | Ouverture S43. `US-SWEEP-S42`, `US-SEASON-TOKENS` (+ 2 micro-patchs), `US-DEMOCK-1a` (pilote) mergées. **Le sprint change de nature :** la lecture montre que le filet lui-même est troué — sur 42 specs, 11 portent un mock mort (URL Jikan) et au moins une n'a aucun mock. `AUD-52` requalifié à 17 specs (contre 12 estimées), en 3 familles. `AUD-53` soldé (réserve basse de This Season). `modal-next-episode` rouge, cause non établie. **Décision PO : bêta NON ouverte en S43** | 3 MERGE + 2 micro-patchs |
| **SE-071** | S43 | Rouge `modal-next-episode` **instruit et clos** : cause = absence de mock réseau, établie par variable unique. A révélé `AUD-54` (vue Semaine qui reconstruit ses cartes en synchro, confirmé à l'œil en 3G, requalifié 🟠 — le tap reste juste). 5 US MERGE : `US-DEMOCK-HELPER`, `US-DEMOCK-1d`, `US-DEMOCK-1b/c`, `US-DEMOCK-2a`, `US-DEMOCK-2b`. **Famille A d'`AUD-52` soldée : zéro spec ne connaît plus l'URL d'AniList.** Zéro sollicitation de Gemini (2ᵉ session consécutive) | 5 MERGE |
| SE-072 | S43 | **Clôture S43.** `US-HEADER-ICONS` (1ʳᵉ US Gemini en 3 sessions) + `US-HEADER-TINT` + `US-DEMOCK-2c` (5 specs). `AUD-52` soldé. `AUD-55` diagnostiqué et corrigé (`DEC-187`). `DEC-188` (R4-ter) | Sweep 55/55, 43 specs. **v0.37.0** |
| SE-073 | S44 | Ouverture de S44. Backlog priorisé avec colonne impact bêta. `US-MORELIKETHIS-FIX` (`AUD-16`) + `US-MLT-REAL` (`AUD-58`) livrées par Gemini. `AUD-57` découvert et soldé (helper de mock incapable de produire une relation). `DEC-187` (fixture ↔ champs filtrés), `DEC-188` (format de réponse au PO) | 2 MERGE, 2 specs E2E créées, 45 au registre. Note aux testeurs rédigée |
| **SE-074** | S44 | `US-HEADER-MOBILE-B` mergée (en-tête mobile 181 → 128 px). Sweep complet 58/58. **`AUD-42` fermé sur observation second appareil — bêta ouverte.** `DEC-189` (maquette obligatoire). `AUD-59` ouvert (onboarding : message de succès sur échec local). `J10e` gelée faute de spécification et de mesure. Chrome mobile A reportée (casse 2 specs) | 1 MERGE, streak 27, 331 tests |
| **SE-075** | S44 | **Chantier documentaire hors slot.** Audit externe du process → refonte de la cadence : **`DEC-190` supersede `DEC-146`** (1 collage par session au lieu de 6,8 ; lot documentaire à la clôture de sprint). Corpus 3 291 → 2 250 lignes, 16 → 14 fichiers, **tous ≤ 250 lignes** sauf `TYPES_CONTRACT` (exception nommée H6). `EPICS.md` fusionné dans `ARCHITECTURE_FONCTIONNELLE.md`. `AUDIT.md` réorganisé par état (600 → 208 l., 30 constats clos condensés en 1 ligne, **aucun perdu**). Collision `DEC-159` résolue → `DEC-191`. Les 4 patchs de dette de SE-074 intégrés | **Aucun code, aucun bump.** Dette documentaire à zéro |

---

## 2. 🔄 Tampon de sprint

> **Ce que la session vient de décider et qui n'a pas encore rejoint son document.**
> 1 ligne par élément, numéro **déjà attribué**. Vidé par le lot documentaire de clôture de sprint.
>
> Format : `ID · document de destination · une phrase`.

*(vide — lot SE-075 intégralement appliqué)*

---

## 3. 📦 Versions

> Une ligne par bump. Un bump = une clôture de sprint.

| Version | Sprint | Livré |
|---|---|---|
| ≤ v0.21.0 | ≤ S21 | Migration vanilla → Vue 3, `US-PINIA`, `US-JST`, CI |
| — | S23→S27 | ⚠️ Détail non capturé |
| v0.28.0 | S28 | Epic Stats |
| v0.29.0 | S29→S33 | Refonte nav · `US-127` · `US-AUTH-LOGOUT` |
| v0.30.0 | S36 | `US-E2E-REGISTRY-RESYNC` · `US-SCROLL-387` · `US-MODAL-CENTER-AUDIT` |
| v0.31.0 | S37 | `US-GRID-FIX` · `US-MODAL-UNIFY` annulée (`DEC-110`) |
| v0.32.0 | S38 | `AUD-01` · `AUD-02` · registre E2E · purge debug |
| v0.33.0 | S39 | Migration AniList `J02`→`J07` · recherche · synopsis nettoyé · `AUD-21` |
| **v0.34.0** | **S40** | Migration AniList `J09`→`J11b` — **Jikan intégralement retiré** — + `J12` harnais E2E AniList |
| v0.35.0 | S41 | **« Ce que j'ajoute reste »** — la persistance cesse de mentir. `9e3923e` · 295 tests / 32 fichiers · 178 modules · 368,90 kB · 10 US + 4 micro-patchs. Gate : gain de fiabilité visible — la bibliothèque survit au logout, revient à la connexion, tient 500 titres, et le réseau ne gèle plus l'application. Zéro fonctionnalité ajoutée |
| v0.36.0 | S42 | **« On peut nous faire confiance »** — l'écran cesse de mentir sur l'état réel : le message d'ajout annonce l'onglet où l'anime a réellement atterri, la recherche est lisible et range au bon endroit, un anime déjà suivi disparaît de This Season sans rechargement, un second appareil annonce le compte retrouvé au lieu de rejouer l'inscription à l'aveugle. 10/10 slots |
| v0.37.0 | S43 | **« Rien ne casse en douce »** — 10/10 slots. Gate : gain ressenti — icônes SVG stables, cibles 44 px, thème sombre réparé. Filet : 0 mock mort, 43 specs, sweep 55/55 |
