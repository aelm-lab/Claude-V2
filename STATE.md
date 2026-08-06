# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état courant — sprint, session, versions,
> compteurs, Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient,
> ils ne dupliquent jamais une valeur. *Exception unique : `AGENTS.md`, lu par Gemini qui n'a
> pas accès à ce fichier — ses chiffres sont en dur et resynchronisés à chaque sprint.*
>
> **Ce qui n'est PAS ici :** les règles (→ `PILOTAGE.md`, `AGENTS.md`), le pourquoi des choix
> (→ `DECISIONS.md`), l'acquis fonctionnel (→ `EPICS.md`).

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint en cours** | **S38** — ouvert, Goal non atteint |
| **Session courante** | **SE-049** — chantier documentaire, **hors sprint** (alias historique : « S37bis ») |
| **Session suivante** | **SE-050** — reprise du sprint S38 |
| **Dernière version livrée** | v0.31.0 (S37) |
| **Dernier DEC** | **DEC-115** |
| **Commit `main`** | `444d385` |

> **Rappel de modèle (détail → `PILOTAGE.md §1`) :** un **sprint** se ferme sur un Sprint Goal
> atteint ; une **session** se ferme sur la capacité de conversation et produit un handoff.
> Un sprint couvre N sessions. Une session hors sprint (chantier, audit) ne bumpe aucune version.

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-049** | — (hors sprint) | Refonte complète du corpus documentaire : audit, fusion, régénération | Handoff `SE-049 → SE-050` |
| SE-048 *(ex-« S38 »)* | S38 | Investigation Jikan + cache local + onboarding | 3 investigations P0 closes, 0 code livré |
| SE-047 *(ex-« S37 »)* | S37 | US-GRID-FIX, annulation US-MODAL-UNIFY | v0.31.0 |
| SE-046 *(ex-« S36 »)* | S36 | Resync registre E2E, overflow horizontal, centrage popups | v0.30.0 |

> Les sessions antérieures à SE-046 n'ont pas été numérotées séparément (le compteur `SXX`
> servait aux deux axes jusqu'en SE-049). Leur détail vit dans les handoffs archivés.

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.31.0** | **S37** | US-GRID-FIX (classe `.aa-card-grid` centralisée, 2 causes racines corrigées) · US-MODAL-UNIFY annulée après investigation (DEC-110) |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT (clos, DEC-108) · US-SEARCH-3 (clos sans dev). Résorbe le retard de bumps S30→S35 |
| v0.29.0 | S29→S33 | Refonte nav (header scroll iOS) · US-127 (SyncIndicator) · US-AUTH-LOGOUT |
| v0.28.0 | S28 | Epic Stats : `useStats` + `StatsPage.vue` + route `/stats` + onglet Stats |
| — | S23→S27 | ⚠️ Détail non capturé (vivait dans les handoffs archivés) |
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + consolidation `EPICS.md` |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| ≤ S16 | — | Migration vanilla → Vue 3 (Phases 0→7) · EPIC-1/2/3 clos · dual audit s16 |

> **Le numéro de version ne suit plus le numéro de sprint** depuis S29 (voir `PILOTAGE.md §1`).
> Cette table est la seule correspondance valide.

---

## 🎯 Métriques techniques — fin S37, confirmées sur machine PO

| Métrique | Valeur | Preuve |
|---|---|---|
| Tests unitaires | **156 passed** (20 fichiers) | `npm run test:run`, machine PO |
| Type-check | **vert**, zéro `any` | `npm run type-check`, sortie vide |
| Build | **vite 6.4.2 · 178 modules** | `npm run build` |
| E2E — fichiers | **38 sur disque / 38 enregistrés** | sweep batch1→5, machine PO |
| E2E — tests | **46 tests, 45 verts** | idem |
| Batchs | batch1 → batch5 | batch4/5 créés en S36 |

**Seul rouge :** `more-like-this-modal` — cause externe (endpoint Jikan `/recommendations` en
504), **non-régression**. Dette suivie sous `US-E2E-MLT-MOCK`.

**Install :** `npm install` fonctionne en direct. `--legacy-peer-deps` n'est plus nécessaire
(downgrade `@pinia/testing` mergé). Ne le réarmer que si un futur `package.json` réintroduit
le conflit de peer-deps.

### 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> Ces trois métriques n'ont jamais été instrumentées depuis leur création. Elles ne
> deviendront mesurables qu'avec un sprint produit dédié. Définitions → `PILOTAGE.md §4`.

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S37** | ✅ **Gain ressenti.** Densité de grille uniforme (2 colonnes) sur This Season, Library/Upcoming, For You et Coming Soon. Fin de la rupture visuelle constatée par le PO sur 4 captures successives |
| S36 | ✅ Gain ressenti. Plus de scroll latéral sur téléphone étroit, popups centrées. Clôt l'exception de dette S35 |
| S38 | ⏳ **Non répondue — sprint ouvert.** Aucun code livré à ce stade (session d'investigation). ⚠️ Si S38 se clôt sans gain visible, le garde-fou « pas plus d'1 sprint sans gain consécutif » impose un sprint produit ensuite |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : à remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.** Un endpoint testé avec d'autres paramètres est un autre endpoint.

### Jikan v4 — dernière mesure : S38 · ⚠️ **à remesurer à l'ouverture de SE-050**

**Diagnostic retenu (DEC-113) :** ce n'est ni une panne globale, ni un paramètre de requête
fautif. **MyAnimeList est inaccessible depuis Jikan ; seules les URLs déjà présentes dans le
cache de Jikan répondent 200.** Toute URL neuve → 504.

| Fonction | État | Explication |
|---|---|---|
| Saisons (`/seasons/now`, `/seasons/upcoming`) | ✅ **fonctionnelles** | URLs fixes appelées en boucle → cache chaud chez Jikan |
| Recherche (`/anime?q=`) | ❌ **structurellement KO** | chaque titre tapé = URL neuve = cache miss = 504 |
| Détail anime (`/anime/{id}`) | ❌ KO sur ID non caché | impacte `syncAnimeUpdates` (remplissage du champ `day`) |
| Recommandations (`/anime/{id}/recommendations`) | ❌ KO | cause du rouge `more-like-this-modal` |

**Bug amont :** issue GitHub `jikan-me/jikan-rest` **#610**, ouverte, non résolue.
Corroboré par la doc officielle Jikan (cache 24 h + rafraîchissement en tâche de fond).
**Aucun correctif possible côté Aanime.**

**Conséquence produit :** `US-ANILIST-SEARCH` est la **seule voie de réparation de la
recherche** et passe en **condition de lancement public**.

---

## 📋 Kanban — sprint S38

### ✅ Done
- **[P0] Investigation cache Jikan local — CLOSE.** Cache `aanime_seasons_now` mesuré à 22,9 h
  pour un TTL de 24 h → **valide**. `readLocalCache` / `writeLocalCache` conformes. **Aucun
  bug.** Le « flag de désactivation du cache en dev » évoqué n'existe pas dans le code
  (`findstr import.meta.env DEV` → 0 hit) : confusion avec la case « Disable cache » du
  panneau Network de DevTools, qui ne concerne que le cache HTTP, jamais le localStorage.
- **[P0-bis] Comportement à l'expiration — CLOS.** `fetchCurrentSeason` sert délibérément le
  cache périmé si le fetch échoue (fidèle au vanilla). `error.value` est renseigné mais
  **jamais affiché**. Sans cache et sans réseau → liste vide silencieuse. Comportement
  conservé ; dette UX enregistrée (`US-CACHE-STALE-WARNING`, P2). → DEC-114
- **[P0-ter] Cause racine des 504 — CLOSE**, prouvée 9 mesures / 9. → DEC-113

### 🔄 In Progress
- **[US-ONBOARDING-REFRESH] 🔴 CRITIQUE — investigation à 90 %, non close.**
  - ✅ `OnboardingPage.vue:93` appelle bien `store.addAnime(buildSeedEntry(anime))`
  - ✅ `finishWithSeed` : `addAnime` → `markOnboarded` → `await saveToDatabase()` → toast → `router.push('/week')`
  - ✅ `CalendarWeekPage.vue:94` filtre sur `a.state === 'calendar' && a.day === dayClass`
  - ✅ `buildSeedEntry` (`onboardingFilter.ts:10`) retourne `{ ...anime, id, state }` — **ne pose jamais `day`**
  - ✅ `normalizeAnime` **ne produit jamais `day`** → DEC-115
  - ❓ **NON RÉSOLU :** le `aanime_calendar` du PO ne contient **aucune** entrée
    `state:'calendar'` (tableau vide). Trois lectures possibles, aucune tranchée : entrées
    présentes dans un autre `state` / mauvais navigateur interrogé / rien persisté.
  - ➡️ Commandes de reprise : voir handoff `SE-049 → SE-050`.

### 📝 To Do — S38 / S39
1. **[US-ONBOARDING-REFRESH]** 🔴 — clore l'investigation puis spécifier
2. **[US-ONBOARDING-i18n]** 🟠 — titres d'anime en japonais / rōmaji pendant l'onboarding,
   incohérent avec le reste de l'app en anglais
3. **[US-ONBOARDING-LAYOUT]** 🟠 — cartes d'onboarding mal centrées (probablement famille
   `US-GRID-CENTRAL` — à vérifier, ne pas supposer)
4. **[US-LOGOUT-RESET]** 🟠 — lien « vider mes données / repartir de zéro » dans
   `LogoutConfirmModal`, effaçant le compte Firestore + le cache local avant déconnexion

### 🗂️ Backlog

**Condition de lancement**
- **[US-ANILIST-SEARCH]** ⬆️⬆️ 🔴 — seule voie de réparation de la recherche (DEC-113).
  AniList GraphQL sert sa propre base et expose `idMal` → migration possible sans casser les
  données existantes.

**Dette**
- `[US-CACHE-STALE-WARNING]` 🟠 (P2) — avertir l'utilisateur quand les données affichées
  dépassent le TTL ou proviennent d'un fallback d'erreur
- `[US-GRID-CENTRAL]` 🟢 — migrer les 4 pages saines vers `.aa-card-grid`, supprimer les
  `.recs-grid` / `.grid` / `.plantowatch-grid` locales
- `[US-E2E-MLT-MOCK]` 🟠 — mock réseau de `more-like-this-modal`
- `[US-SEARCH-GUARD]` 🟠 — spec E2E sur les section headers de recherche
- `[US-140d]` 🟠 — toast de bienvenue à l'atterrissage de l'onboarding
- `[US-165]` 🟢 — extraire `fetchTopFinishedAnime` de `useRecommendations` vers `useJikanApi`
- `[US-166-CSS]` 🟢 — dette CSS groupée (`.rc-mark-done`, `.test-*`, `.search-suggestion-added`
  `#10b981` en dur → `var(--airing)`, F18→F23)
- `[F8]` 🟢 — sous-nav et logo quasi illisibles en dark mode. Ouvert depuis la session 7,
  jamais traité
- `[US-JIKAN-HEALTHCHECK]` 🟠 (P1) — dev-only, détail par test au-delà d'un verdict global
- `[F14]` 🟠 — skeletons au chargement (~6 s de blanc, `SkeletonCard` existe mais inutilisé)
- `[F15]` 🟢 — hiérarchie typographique Library/Upcoming, section vide

**Produit**
`login redesign` · `US-PWA` · `dual-titre rollout` (modale / RecCards / carte semaine) ·
`US-124` (mapping MAL `Dropped`) · `Cluster B découverte` · `Cluster C growth` · `STATS-5`

---

## ✅ Trous fermés (ne pas réinvestiguer)

| Trou | Fermé en |
|---|---|
| Compteur E2E périmé depuis la session 16 | S36/S37 — sweep complet batch1→5 |
| Divergence de compteur E2E 47/46 vs 46/45 | S38 — arbitré à **46 / 45** |
| `US-E2E-CONFIG` (Playwright local) | S36 — confirmée fonctionnelle |
| Hash `main` non relevé | S36 — `444d385` |
| US-140 / US-127 / US-SEARCH-3 « à faire » | S36 — les trois étaient déjà livrés |
| `US-E2E-BATCH-AUDIT` | absorbé par `US-E2E-REGISTRY-RESYNC` |
| Nature réelle de la panne Jikan | S38 — DEC-113 |
| Existence d'un flag « cache désactivé en dev » | S38 — **n'existe pas** |
| Centrage des modales | S36 — DEC-107/108, cause racine externe aux modales |

## ❓ Trous restants

- **`aanime_calendar` vide côté PO** — non expliqué, bloque la clôture d'`US-ONBOARDING-REFRESH`
- **Qui remplit le champ `day` ?** — `normalizeAnime` ne le produit pas (DEC-115) ; hypothèse
  `syncAnimeUpdates` via `parseJSTToLocal` depuis le `broadcast` Jikan **non vérifiée**. Si
  confirmée, le défaut d'onboarding et la panne Jikan ont la même cause aval
- **Détail par sprint S22→S27** — non capturé (historique, non bloquant)
- **DEC-75** — trou de lecture assumé, ne pas inventer son contenu
