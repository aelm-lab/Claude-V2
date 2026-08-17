# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état courant — sprint, session, versions,
> compteurs, Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient.
> *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à ce fichier.*

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint** | **S40 — EN COURS.** Goal : *Jikan est débranché avant sa fermeture* |
| **Session courante** | **SE-059** (ouverture de S40) |
| **Dernière version livrée** | **v0.33.0 (S39)** — S40 non clos, aucun bump |
| **Dernier DEC** | **DEC-136** |
| **Commit `main`** | `31ec1b4` (+ 1 correctif E2E en attente de review) |
| **Échéance dure** | **Jikan ferme en octobre 2026** (DEC-125) |
| **Cadence** | **4 à 8 US par sprint**, davantage si elles servent un même sujet |

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-059** | **S40** | Planning S40 · `J08` fusionnée dans `J10` · `J09` coupée en `J09a`/`J09b` · `AUD-05` déclassée · `AUD-22` diagnostiqué · DEC-134→136 | **2 MERGE**, 218 tests, Current Season remarche |
| SE-058 | hors sprint | Tri A/B du benchmark · 3 corrections de faits · DEC-133 · création de `BENCHMARK.md` | Aucun code |
| SE-057 | S39 | `J07` synopsis · `AUD-21` faux jour · sweep E2E · **clôture S39** | v0.33.0, 205 tests, 2 MERGE |
| SE-056 | S39 | `US-SEARCH-DEDUP-ANILIST` · `J06` · DEC-127→130 | 199 tests, 2 MERGE |
| SE-055 | S39 | `J05-E2E` · `US-SEARCH-SPECS-ANILIST` · `AUD-21` ouvert | 199 tests, 2 MERGE |

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| *(en cours)* | **S40** | `J09a` `4403bd5` · `J09b` `31ec1b4` |
| **v0.33.0** | S39 | Migration AniList J02→J07 · recherche · synopsis nettoyé · AUD-21 |
| v0.32.0 | S38 | AUD-01 · AUD-02 · registre E2E · purge debug |
| v0.31.0 | S37 | US-GRID-FIX · US-MODAL-UNIFY annulée (DEC-110) |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT |
| v0.29.0 | S29→S33 | Refonte nav · US-127 · US-AUTH-LOGOUT |
| v0.28.0 | S28 | Epic Stats |
| — | S23→S27 | ⚠️ Détail non capturé |
| ≤ v0.21.0 | ≤ S21 | Migration vanilla → Vue 3, US-PINIA, US-JST, CI |

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **218 passed** (25 fichiers) | `npm run test:run`, machine PO | **SE-059** |
| Type-check | **vert** | sortie vide | **SE-059** |
| Build | **vert** | `181 modules transformed` | **SE-059** |
| E2E — fichiers sur disque | **42** | inchangé depuis SE-056 | **SE-059** |
| E2E — registre `package.json` | **42 / 42** | 5 batches : 9·9·8·9·7 | **SE-059** |
| E2E — batch 1 | **9 / 10** | rouge = `discover-season-dedup`, en correction | **SE-059** |
| E2E — batches 2→5 | **non rejoués** en SE-059 | sweep prévu en clôture de S40 | SE-057 |

> ⚠️ Pas de mention « zéro `any` » : `vue-tsc` ne les teste pas, ESLint n'est jamais exécuté.
> **CI :** `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`. **Ni Playwright, ni ESLint** (`AUD-08`).
> **Install :** `npm install` en direct, `--legacy-peer-deps` inutile.
> ⚠️ `useJikanApi.spec.ts` consomme ~1,85 s, soit ~40 % de la suite. Disparaîtra avec `J11`.
> 🔴 **Ne jamais lancer `npm run test:e2e:sweep`** : il chaîne les batches en `&&` et s'arrête au
> premier rouge. Les 5 batches se lancent **séparément**.
> 🔴 **Gemini ne peut pas exécuter les E2E** : son bac à sable n'a pas de navigateur Playwright et
> ne peut pas l'installer (constaté SE-059). Ne jamais le lui demander.

---

## 🔴 Rouges E2E — 3, tous qualifiés

| Spec | Batch | Nature | Terrain |
|---|---|---|---|
| `discover-season-dedup` | 1 | **`AUD-22` diagnostiqué et fermé sur le fond** (voir Trous). Reste un correctif de sélecteur en cours | **SE-060** |
| `more-like-this-modal` | 5 | **Pas une régression.** Dépend de `/anime/{id}/recommendations` Jikan, en 504. Tombera seul avec `J11` | S40 |
| `onboarding-toast` | 4 | `AUD-23` — non diagnostiqué | S41 |

---

## 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> 🔴 Cinq sprints consécutifs réparent le parcours que ces métriques mesurent, sans pouvoir
> le prouver. **Priorité n°1 de S41.**

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S40** | *(ouvert — réponse attendue à la clôture)* |
| S39 | ✅ Gain ressenti + fiabilité. Recherche AniList · plus de faux jour de diffusion |
| S38 | ✅ Gain ressenti. Un anime ajouté apparaît sur son jour |
| S37 | ✅ Gain ressenti. Densité de grille uniforme sur 4 surfaces |
| S36 | ✅ Gain ressenti. Fin du scroll latéral mobile, popups centrées |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.** 🔴 Seule une mesure du PO fait foi.

### AniList — source primaire. Dernière mesure : **SE-059** (visuel PO)

| Point | Résultat |
|---|---|
| `graphql.anilist.co` | **200.** CORS OK depuis l'origine de production |
| Recherche | **fonctionnelle**, revérifiée SE-059 |
| **Saison en cours** | **fonctionnelle** — Current Season peuplée, constatée SE-059 |
| Rattachement `idMal` | 19 / 19 sur corpus de test (SE-054) |

⚠️ Corpus de 19 titres mainstream. Le taux d'orphelins réel sur un import MAL de 300 titres
sera > 0 → repli titre+année prévu dans **`J10`** (DEC-134).

📌 **Mesure acceptée : constat visuel du PO dans l'app.** La requête du code est un POST
GraphQL multi-lignes, inéchappable proprement en `cmd`.

### Jikan v4 — en fin de vie. Dernière mesure : SE-053, complétée SE-058

| Endpoint | Code |
|---|---|
| `/seasons/now?limit=25` | 200 — page 1 seulement · **plus consommé depuis `J09b`** |
| `/seasons/now&page=2` | **504** — pagination morte |
| `/seasons/upcoming?limit=25` | **504** |
| `/anime/{id}` | 200 — **encore consommé**, cible de `J10` |
| `/anime?q=` | **504 — mort, acté, ne plus remesurer** |
| `/anime/{id}/recommendations` | **504** |

🔴 **Ferme en octobre 2026** (DEC-125).

---

## 📋 Kanban

### ✅ S40 — livré à ce jour
`J09a` `4403bd5` — `fetchCurrentSeason` AniList + contrat d'erreur + cache dédié
`J09b` `31ec1b4` — Discover et Onboarding branchés sur AniList + correction de clé `AUD-22`

### 🔄 In Progress
`US-E2E-SEASON-ANILIST` 🟢 — réécriture de `discover-season-dedup` sur AniList.
Correctif de sélecteur émis en fin de SE-059, **review à faire en ouverture de SE-060**.

### 📝 S40 — reste à faire
| Ordre | US | Risque | Note |
|---|---|---|---|
| 1 | `J10` — sync / détail sur AniList **+ repli titre+année** | 🔴 | Absorbe `J08` (DEC-134). Touche `useSync.ts` → **grep + découpage probable**. Débloque le synopsis de la bibliothèque persistée et le total d'épisodes |
| 2 | `J11` — retrait de Jikan | 🔴 | Fait tomber `more-like-this-modal` (attendu) |
| 3 | `AUD-05` — mode dégradé visible | 🟢 | **Déclassée de 🟠.** L'UI d'erreur existe déjà et est vivante depuis `J09b`. Reste : affichage du drapeau `stale` |
| 4 | **Sweep E2E complet** | — | 5 batches séparés, jamais `&&` |

> 🚫 **Aucun item du benchmark n'entre en S40** (décision PO, SE-058).

### 🗂️ S41 — Goal proposé : *mesurer et réparer le parcours d'entrée*
`US-TTFA-INSTRUMENT` **P1** · `US-SYNOPSIS-VERSIONTOP` 🟠 **P2** · `US-ONBOARDING-REFRESH` 🔴 ·
`AUD-23` · `US-TOAST-ONBOARDING` · `US-MODAL-NEXTEP-HIERARCHY` 🟢 **P3** · `US-CARDS-ALIGN` ·
`US-CLEAR-DATA` · **`US-SEARCH-ZINDEX` 🟢** *(SE-059)* · **`US-SEASON-COUNTDOWN` 🟢** *(SE-059)*

> **`US-SEARCH-ZINDEX`** : une carte de la grille hebdo passe **par-dessus** la liste de
> résultats de recherche, rendant un résultat inatteignable au clic. Constat visuel SE-059.
> **`US-SEASON-COUNTDOWN`** : `nextAiringEpisode.timeUntilAiring` est **déjà** dans la requête
> écrite en `J09a`. Le compte à rebours est devenu quasi gratuit. Répond à `B-06` du benchmark
> mais n'en est **pas issu** — ne viole pas le gel du benchmark.
> **Chantier « lisibilité des titres » :** titres tronqués à l'identique dans la recherche et
> dans Current Season (4 saisons du même anime indistinguables). À traiter avec
> `US-CARDS-ALIGN` plutôt qu'en retouches éparses.

### Non planifié
`US-DEMOGRAPHICS` · `AUD-08` (CI sans Playwright ni ESLint) · `AUD-03`→`AUD-20` dans `AUDIT.md` ·
candidats benchmark dans `BENCHMARK.md`

### Benchmark concurrentiel — ⏸️ EN ATTENTE (décision PO, SE-058)
Reprise **après S40**. Tri complet → `BENCHMARK.md`.

---

## 🕳️ Trous restants

| Trou | Gravité | État |
|---|---|---|
| **Bibliothèque existante sans synopsis** | 🟠 | Seuls les nouveaux ajouts ont un résumé. Réglé par `J10` |
| **Total d'épisodes absent** — « Episode 7 / ? » | 🟠 | AniList fournit `episodes`. Réglé par `J10` |
| **Specs au seed décoratif** | 🟠 | `snap-to-today` sème un calendrier mais n'asserte qu'un en-tête de jour |
| `AUD-23` — `onboarding-toast` rouge | 🟠 | Non diagnostiqué. Terrain S41 |
| **Benchmark partiel** | 🟠 | 3 surfaces jamais observées avec du contenu. Remesure ciblée à la clôture de S40 |
| **Tri de Current Season non arbitré** | 🟢 | AniList sert par `POPULARITY_DESC`, Jikan servait l'ordre de saison. Composition de la page et des 8 suggestions d'onboarding modifiée. Décision produit ouverte |

> ✅ **Fermé en SE-059 :** `AUD-22` — cause identifiée, ce n'était **pas** un défaut de
> dédoublonnage : la spec ciblait `.anime-grid`, classe renommée `aa-card-grid` en S37 par
> `US-GRID-FIX`, et mockait la pagination Jikan. **Elle ne vérifiait plus rien depuis S37.**
> L'inversion `id`/`mal_id` corrigée dans `J09b` était réelle mais n'était pas la cause du rouge.
> ✅ **Fermé en SE-059 :** `AUD-05` plomberie — `fetchCurrentSeason` jette désormais (DEC-136).

---

## 🛰️ Documents satellites (hors ordre de lecture)

| Doc | Contenu | Statut |
|---|---|---|
| `AUDIT.md` | Constats `AUD-03`→`AUD-20`, findings écartés | Actif |
| `BENCHMARK.md` | Tri A/B du benchmark, corrections de faits, candidats non planifiés | Actif |
