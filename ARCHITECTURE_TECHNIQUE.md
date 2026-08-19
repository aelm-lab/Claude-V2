# ARCHITECTURE_TECHNIQUE.md — Vue technique

> **Rôle :** l'architecture technique **réellement implémentée** — couches, modules, boot, réseau, registre des clés. Pendant fonctionnel : `ARCHITECTURE_FONCTIONNELLE.md`.
> **Pas ici :** aucun compteur, aucune dette, aucun bug ouvert (→ `STATE.md`) · aucune règle opposable (→ `AGENTS.md`).

---

## 1. Stack

| Couche | Techno |
|---|---|
| Framework | Vue 3 (`<script setup lang="ts">`) |
| Langage | TypeScript strict (zéro `any`) |
| State global | Pinia (`stores/anime.ts`, `stores/ui.ts`) |
| Routing | Vue Router 4 (11 routes, guards auth/guest) |
| Réactivité utilitaire | `@vueuse/core` — `useDark`, `useSwipe`, `useIntersectionObserver`, `watchDebounced`, `onClickOutside`, `onKeyStroke` |
| Build | Vite 6 + `@vitejs/plugin-vue` |
| Auth / DB | Firebase v12 (Auth email-link + Firestore) |
| API externe | **AniList GraphQL** (`graphql.anilist.co`), via `anilistClient` |
| Cache | IndexedDB (relations, recommandations) + localStorage (clés préfixées `aanime_`, cf. §7) |
| Styles | `style.css` **global** (variables CSS, dark mode via `html.dark`) |
| Tests / CI | Vitest + Playwright + GitHub Actions |
| Déploiement | Cloud Run (europe-west2), continu |

---

## 2. Structure des dossiers

```
src/
├── assets/
├── components/
│   ├── layout/      # AppLayout.vue, AuthLayout.vue
│   ├── pages/       # 1 composant par route
│   └── ui/          # Composants atomiques réutilisables (AppHeader.vue y compris)
├── composables/     # useXxx.ts — logique réactive réutilisable
├── stores/          # Stores Pinia
├── utils/           # Fonctions PURES (zéro import Vue)
├── router/          # index.ts
├── types/           # Interfaces TypeScript partagées
├── lib/             # firebase.ts (singleton)
└── main.ts
tests/e2e/           # Specs Playwright (cumulatives, exclues de Vitest)
    └── _helpers/    # anilistMock.ts — mock réseau unique, hors batch
```

> ⚠️ **`AppHeader.vue` vit dans `src/components/ui/`**, pas `src/components/`. **Toujours vérifier le chemin réel** (`findstr`) avant de lister un fichier dans une US.

---

## 3. Les couches et la règle de dépendance

```
COMPOSANTS (.vue)      UI + réactivité locale
   ▼
COMPOSABLES (useXxx)   orchestration, I/O, réseau
   ▼
STORES (Pinia)         état pur, zéro I/O
   ▼
UTILS (.ts purs)       calculs purs, testables
   ▼
TYPES (.ts)            contrat, zéro logique
```

**Dépendance descendante uniquement.** Les règles opposables (`R-CODE-3` à `R-CODE-8`) vivent dans `AGENTS.md` ; ce document décrit la structure, il ne la duplique pas.

Deux conséquences souvent oubliées :
- **Le store ne fait aucun I/O.** La persistance est un `watch` externe dans `usePersistence` (DEC-26).
- **Aucun `<style scoped>`.** Tous les styles vivent dans `style.css` — une classe en `<style scoped>` est invisible depuis les autres composants (DEC-111).

---

## 4. Carte des modules

### 4.1 Types (`src/types/`)

| Fichier | Contenu |
|---|---|
| `anime.ts` | `AnimeEntry`, `AnimeState`, `AnimeStatus`, `WeekDay`, `CardStatus`, `AnimeEpisodeInfo`, `JstParseResult`, `RecencyBucket` |
| `recs.ts` | `TasteProfile`, `RecBadge`, `RecSignal`, `RecPreset`, `RecContext`, `HistoryItem`, `RecAction` |
| `anilist.ts` | DTO bruts AniList + `AnimeRelation` (§8bis de `TYPES_CONTRACT.md`) |
| `persistence.ts` | `ScheduleDocument`, `IdbStoreName` |

> Aucun type n'est défini ailleurs. Tout vient de `TYPES_CONTRACT.md`.

### 4.2 Utils purs (`src/utils/`) — zéro Vue

| Fichier | Exports clés |
|---|---|
| `normalizeAniList.ts` | Conversion DTO AniList → `AnimeEntry`. Rejette tout média sans `idMal` · nettoie le HTML de `description` (DEC-132) · `awaitingSchedule` sans `nextAiringEpisode` (DEC-131) |
| `normalize.ts` | `normalizeAnime` — forme canonique des entrées **legacy** (vocabulaire MyAnimeList conservé pour compatibilité) ; produit toujours `studios: string[]` (DEC-86) et pose `day` + `airsTime` par cascade (DEC-124) |
| `jst.ts` | `parseJSTToLocal` (ancre 1970, JST → local) |
| `episodeInfo.ts` | `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus` (**seuil hiatus 14 j — source unique**) |
| `helpers.ts` | `escapeHTML`, `getWeekNumber`, `dedupeByMalId`. 🔴 **Aucun export réseau** — une spec de surface (`helpers.spec.ts`) échoue si un `BASE_URL` ou un `fetchWithRetry` réapparaît |
| `recEngine.ts` | `buildTasteProfile`, `scorePool`, `assignBadge`, `extractBecauseYouWatched`, `buildNextBatch`, `applyPreset`, `RelationMemory` |
| `ics.ts` | `buildICSContent` (génération de texte pure) |
| `malImport.ts` | `parseMalXml` (DOMParser, pur) |
| `idb.ts` | `idbGet`, `idbSet` (wrapper IndexedDB, import statique) |
| `constants.ts` | `POSTER_PLACEHOLDER` (source unique, DEC-84) |
| `onboardingFilter.ts` | `buildSeedEntry`, `selectOnboardingSuggestions` (limite 8) — voir §9 |

### 4.3 Stores Pinia (`src/stores/`)

| Store | Responsabilité | I/O |
|---|---|---|
| `anime.ts` | `animeCalendarData[]`, `currentDate`, `currentView` + actions `addAnime` (upsert), `addAnimeSilent`, `removeAnime`, `setDate`, `setData`, `clearAll`. Transitions de state, auto-vault | **Aucun** |
| `ui.ts` | État des overlays : modal ouvert + contexte, sheet recency, panneau ep-override | Aucun |

### 4.4 Composables (`src/composables/`)

| Composable | Rôle | Réseau |
|---|---|---|
| `useAniListApi.ts` | Recherche, détail par `idMal`, relations, saison courante / suivante, top finished — chaque fonction en variante `WithMeta`. Exporte aussi `resolveSeason` / `resolveNextSeason` | **AniList** |
| `useSync.ts` | `syncAnimeUpdates` (liste close de 9 champs mutables), `isSyncing`, `clearSyncTimestamp`. Repromeut les entrées `awaitingSchedule` dès qu'un `day` arrive | AniList |
| `useRecommendations.ts` | `fetchRecPool`, `getNextBatch`, `reScorePool`, `buildRelationMemory`, `getBecauseYouWatchedBatch`, `getSlotFillSuggestions`, `getSeasonNudges`, `saveRec`, `saveAsCompleted` | AniList + cache |
| `useFirebaseAuth.ts` | Auth email-link, `signOut`, `onAuthStateChanged` **au niveau module** (sinon listeners empilés) | — |
| `useFirestore.ts` | `schedules/{uid}` brut + `handleFirestoreError` (`loadSchedule`, `saveSchedule`) | — |
| `usePersistence.ts` | `loadFromDatabase` (migration des clés + réconciliation), `saveToDatabase`, `watchDebounced`, fallback localStorage | — |
| `useEpisodeInfo.ts` | Wrappe `utils/episodeInfo` | — |
| `useStats.ts` | Agrégation « Mon année anime » — `topGenres` scoped `completedThisYear` (DEC-89), garde `genres ?? []` | — |
| `useICS.ts` | `downloadICS` (Blob + iOS) | — |
| `useMalImport.ts` | `parseMalXml` + `importMalFile` (FileReader, `addAnimeSilent`) | — |
| `useToast.ts` | `showToast`, `hideToast` | — |
| `useTheme.ts` | `useDark()` → classe `dark` sur `<html>`, `toggleDarkMode`, `isDark` | — |

> 🔴 **`startBackgroundRelationFetch` n'existe plus** (DEC-147). Aucun worker de relations en fond.
> **Singleton Firebase :** `src/lib/firebase.ts` initialise `initializeApp` / `getAuth` / `getFirestore` **une seule fois**. Un guard Vue Router n'appelle jamais `useFirebaseAuth()` : il utilise le singleton `auth` + `await auth.authStateReady()` (DEC-28).

### 4.5 Composants (`src/components/`)

- **layout/** : `AppLayout.vue` (header + navs + `<KeepAlive><router-view>`), `AuthLayout.vue`
- **pages/** : 1 par route — `CalendarWeekPage`, `CalendarMonthPage`, `DiscoverExplorePage`, `DiscoverSeasonPage`, `DiscoverComingUpPage`, `LibraryExplorePage`, `LibraryPlanToWatchPage`, `LibraryCompletedPage`, `LoginPage`, `StatsPage`, `OnboardingPage`, + stub `CalendarListPage`
- **ui/** : composants atomiques (cards, navs, modals, sheets, toast). Trois modales partagent `.modal-backdrop` : `AnimeModal`, `LogoutConfirmModal`, `RecEngineModal`

---

## 5. Graphe de dépendances

```
main.ts
  └── App.vue ──────────────────────────── provide(isBootingKey)
        ├── usePersistence → useFirestore → lib/firebase (singleton)
        │                  → watchDebounced(animeCalendarData) → saveToDatabase
        ├── useSync → useAniListApi → anilistClient
        ├── useRecommendations → useAniListApi + utils/recEngine + utils/idb
        └── stores/anime ── stores/ui

AppLayout.vue
  ├── AppHeader (SearchInput → useAniListApi.searchAnime)
  ├── PrimaryNav / SecondaryNav (CalendarNavControls → stores/anime.setDate)
  ├── SyncIndicator (useSync.isSyncing)
  └── <KeepAlive><router-view/></KeepAlive>
```

---

## 6. Séquence de boot

**Ordre contractuel (DEC-50), couvert par `src/App.spec.ts` :**

```
1. loadFromDatabase()      // Firestore → store (+ migration aanime_*, réconciliation, fallback localStorage)
2. syncAnimeUpdates()      // sync AniList — le rescore en dépend
3. buildRelationMemory()   // graphe de relations depuis IDB
4. reScorePool()           // re-score des pools de recommandations

finally:
   isBooting.value = false                            // libère LoadingOverlay
   document.getElementById('boot-loader')?.remove()   // loader pré-Vue (DEC-72)
```

> 🔴 L'ancienne 5ᵉ étape (worker de relations en fond) est **supprimée** (DEC-147). Le smoke test `App.spec.ts` asserte les fonctions d'orchestration réellement appelées — ne pas le casser.

**Deux écrans de chargement, pour deux fenêtres distinctes :**
- **`#boot-loader`** — HTML/CSS statique inline dans `index.html`, **hors de `<div id="app">`**. Couvre la fenêtre pré-mount. Supprimé par `.remove()` dans le `finally` (exception R-CODE-4). Il est `position:fixed` et **intercepte tous les pointer events** tant qu'il est là.
- **`<LoadingOverlay>`** — composant Vue monté **à la racine d'`App.vue`**, hors de la gate `v-if="route.meta.requiresAuth"` (DEC-59).

**Boot en 2 phases :** paint immédiat depuis le cache localStorage, réconciliation Firestore en fond par comparaison de timestamps.

> ⚠️ **À confirmer par lecture du code (R3) avant toute US touchant le boot :** `isBooting` est-il libéré après l'étape 1 ou après l'étape 4 ? L'articulation entre l'ordre DEC-50 et l'architecture 2 phases n'a jamais été re-documentée. Ne pas spécifier de correctif sur le boot sans avoir tranché.

---

## 7. Réseau, cache & registre des clés

| Aspect | Implémentation |
|---|---|
| Client réseau | `anilistClient` — endpoint POST unique, **sérialisation à 700 ms**, disjoncteur global (3 échecs → blackout 60 s pour toute l'app). Un **429 n'incrémente jamais** le compteur (DEC-126) |
| Contrat de retour | `{ data, failed }` sur toute fonction réseau — les variantes `WithMeta` ne lèvent jamais (`AP-CATCH-1`) |
| Forme de requête par `idMal` | 🔴 `Page(page:1, perPage:1){ media(idMal:) }` obligatoire, jamais `Media(idMal:)` (DEC-139) |
| Sync | `useSync` — aucun throttle manuel, le client sérialise déjà (DEC-141) |
| Cache relations | IndexedDB, persistant, **sans expiration**. Porte la forme **MyAnimeList** (`[{ relation, entry }]`), lue par `buildRelationMemory` — 🔴 ne jamais y écrire d'`AnimeRelation` AniList (DEC-144) |
| Cache recommandations | IndexedDB, TTL 7 j |
| Cache saisons | localStorage, TTL 24 h. **Le cache périmé est servi si le fetch échoue** ; `error.value` est renseigné mais jamais affiché (DEC-114 → `AUD-05`) |
| Timestamps de sync | localStorage `aanime_sync_ts`, TTL 6 h. `setSyncTimestamp` utilise `anime.id`, **pas** `malId` — préserve les clés existantes |

### Registre des clés localStorage

Toutes préfixées `aanime_` (DEC-85), avec migration transparente au boot portée par `usePersistence.loadFromDatabase` : lecture de l'ancienne clé si la nouvelle est absente, réécriture sous le préfixe, suppression de l'ancienne.

| Clé actuelle | Ancienne clé (migrée) | Usage |
|---|---|---|
| `aanime_calendar` | `animeCalendar` | Document de planning (cache local du store) |
| `aanime_sync_ts` | `anime_sync_ts_v1` | Timestamp de dernière sync |
| `aanime_negative_removed_ids` | `negative_removed_ids` | IDs « pas intéressé » |
| `aanime_email_for_signin` | `emailForSignIn` | Email magic-link |
| `aanime_recs_incoming` | `recs_incoming_v3` | Cache recos Discover |
| `aanime_recs_library` | `recs_library_v2` | Cache recos Library |
| `aanime_season_nudges` | `season_nudges_v1` | Nudges de séquelles |
| `aanime_season_nudge_dismissed` | `season_nudge_dismissed_v1` | Nudges écartés |
| `aanime_seasons_now` | `seasons_now_v1` | Cache saison en cours |
| `aanime_seasons_upcoming` | `seasons_upcoming_v1` | Cache saison à venir |

> 🔴 **Changer une clé = une US dédiée, avec migration.** Jamais au fil d'une autre US.

---

## 8. Flux de données & réactivité

Aucun bus DOM : tout passe par Pinia, `emit` ou `@vueuse`.

| Besoin | Mécanisme |
|---|---|
| Sauvegarde sur mutation | `watchDebounced(animeCalendarData, saveToDatabase, { deep, 1000 })` dans `usePersistence` |
| Rafraîchissement UI | réactivité Pinia native |
| Ajout d'anime | actions du store + appel direct du composable |
| Actions de carte de reco | `emit` composant → handler page → `useRecommendations` |
| Navigation de date | `useSwipe` (@vueuse) → `store.setDate` |

`usePersistence` porte le `watchDebounced` derrière un flag de module (`watchInitialized`) — sans ce flag, les listeners s'empilent. `suppressPersist` permet les mutations silencieuses (import MyAnimeList).

---

## 9. Pipeline du moteur de recommandations

```
1. fetchRecPool(context)        → pools bruts (saison courante + suivante / top finished)
2. buildTasteProfile(history)   → profil de goûts (genres, thèmes, studios, négatifs)
3. buildRelationMemory()        → graphe de relations depuis IDB
4. scorePool(pool, profile)     → _relevanceScore par item
5. applyPreset(preset)          → _presetScore (chips : Hidden gems, Trending…)
6. getNextBatch(context, size)  → batch paginé — dédup via dedupeByMalId AVANT le slice (DEC-74)
7. extractBecauseYouWatched()   → section « parce que vous avez vu X »
8. getSlotFillSuggestions(list) → remplissage des jours vides de la semaine
```

Le pool `incoming` est construit sur **2 appels** (saison courante + suivante, `perPage:50`) — DEC-143.

**Onboarding (`utils/onboardingFilter.ts`)** : `selectOnboardingSuggestions` produit **8 suggestions** scorées sur les genres choisis ; `buildSeedEntry` retourne `{ ...anime, id, state }`. Il ne pose pas `day` lui-même : l'entrée hérite du `day` posé à la normalisation (DEC-124 / DEC-131). ⚠️ Signature exacte non contractualisée → `TYPES_CONTRACT.md §9`.

---

## 10. Qualité & CI

`ci.yml` rejoue à chaque push : `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`.
Filet de régression du boot : `src/App.spec.ts` monte `App` et asserte que les fonctions d'orchestration sont appelées.

Compteurs, état de la suite E2E et dette ouverte → `STATE.md`. Règles de test opposables → `AGENTS.md`.
