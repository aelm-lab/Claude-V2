# ARCHITECTURE_TECHNIQUE.md — Vue technique du système Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** décrire l'architecture **technique réellement implémentée** — couches, modules,
> boot, réseau, registre des clés.
> **Pendant fonctionnel :** `ARCHITECTURE_FONCTIONNELLE.md` — les deux se lisent ensemble.
>
> **Aucun compteur ici** (tests, E2E, taille de build) → `STATE.md`.
> **Aucune dette ici** (backlog, bugs ouverts) → `STATE.md`.

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
| API externe | Jikan v4 (publique, sans clé) |
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
```

> ⚠️ **`AppHeader.vue` vit dans `src/components/ui/`**, pas `src/components/` — un handoff
> antérieur le supposait par erreur. **Toujours vérifier le chemin réel** (`findstr`) avant
> de lister un fichier dans une US.

---

## 3. Les couches et la règle de dépendance

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSANTS (.vue)   pages / layout / ui                    │  UI + réactivité locale
│   ▼ peut importer                                           │
├─────────────────────────────────────────────────────────────┤
│  COMPOSABLES (useXxx.ts)                                    │  orchestration, I/O, réseau
│   ▼ peut importer                                           │
├─────────────────────────────────────────────────────────────┤
│  STORES (Pinia)                                             │  état pur, zéro I/O
│   ▼ peut importer                                           │
├─────────────────────────────────────────────────────────────┤
│  UTILS (.ts purs)                                           │  calculs purs, testables
│   ▼ peut importer                                           │
├─────────────────────────────────────────────────────────────┤
│  TYPES (.ts)                                                │  contrat, zéro logique
└─────────────────────────────────────────────────────────────┘
```

**Dépendance descendante uniquement.** Les règles opposables à Gemini (R-CODE-3 à R-CODE-8)
vivent dans `AGENTS.md` ; ce document décrit la structure, il ne la duplique pas en règles.

Deux conséquences structurantes souvent oubliées :
- **Le store ne fait aucun I/O.** La persistance est un `watch` externe, dans `usePersistence`.
- **Aucun `<style scoped>` dans les composants.** Tous les styles vivent dans `style.css`.
  Une classe définie en `<style scoped>` est invisible depuis les autres composants — cause
  racine réelle du bug de grille `.recs-grid` (DEC-111).

---

## 4. Carte des modules

### 4.1 Types (`src/types/`)

| Fichier | Contenu |
|---|---|
| `anime.ts` | `AnimeEntry`, `AnimeState`, `AnimeStatus`, `WeekDay`, `CardStatus`, `AnimeEpisodeInfo`, `JstParseResult`, `RecencyBucket` |
| `recs.ts` | `TasteProfile`, `RecBadge`, `RecSignal`, `RecPreset`, `RecContext`, `HistoryItem`, `RecAction` |
| `persistence.ts` | `ScheduleDocument`, `IdbStoreName` |

> Aucun type n'est défini ailleurs. Tout vient de `TYPES_CONTRACT.md`.

### 4.2 Utils purs (`src/utils/`) — zéro Vue

| Fichier | Exports clés |
|---|---|
| `jst.ts` | `parseJSTToLocal` (ancre 1970, JST → local) |
| `normalize.ts` | `normalizeAnime` — forme canonique, produit **toujours** `studios: string[]` (DEC-86) et pose **`day` + `airsTime`** depuis `broadcast` par cascade (DEC-124) || `episodeInfo.ts` | `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus` (**seuil hiatus 14 j — source unique**) |
| `helpers.ts` | `fetchWithRetry` (backoff 429), `getWeekNumber`, `escapeHTML`, `dedupeByMalId`, `BASE_URL` |
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
| `useFirebaseAuth.ts` | Auth email-link, `signOut`, `onAuthStateChanged` **au niveau module** (sinon listeners empilés) | — |
| `useFirestore.ts` | `schedules/{uid}` brut + `handleFirestoreError` (`loadSchedule`, `saveSchedule`) | — |
| `usePersistence.ts` | `loadFromDatabase` (migration des clés + réconciliation), `saveToDatabase`, `watchDebounced`, `staleDataWarning`, fallback localStorage | — |
| `useJikanApi.ts` | `searchAnime`, `fetchAnimeById`, `fetchCurrentSeason`, `fetchUpcomingSeason`, `fetchAnimeRelations(WithMeta)`, `fetchAnimeRecommendations(WithMeta)`, `readLocalCache`/`writeLocalCache` | Jikan |
| `useSync.ts` | `syncAnimeUpdates` (batch 2 / 2 s), `startBackgroundRelationFetch` (throttle conditionnel), `isSyncing`, `clearSyncTimestamp` | Jikan |
| `useRecommendations.ts` | `fetchRecPool`, `getNextBatch`, `reScorePool`, `buildRelationMemory`, `getBecauseYouWatchedBatch`, `getSlotFillSuggestions`, `getSeasonNudges`, `saveRec`, `saveAsCompleted`, `fetchTopFinishedAnime` (encore inline) | Jikan (cache) |
| `useEpisodeInfo.ts` | Wrappe `utils/episodeInfo` | — |
| `useStats.ts` | Agrégation « Mon année anime » — `topGenres` **scoped `completedThisYear`** (DEC-89), garde `genres ?? []` contre le cache legacy | — |
| `useICS.ts` | `downloadICS` (Blob + iOS) | — |
| `useMalImport.ts` | `parseMalXml` + `importMalFile` (FileReader, `addAnimeSilent`) | — |
| `useToast.ts` | `showToast`, `hideToast` | — |
| `useTheme.ts` | `useDark()` → classe `dark` sur `<html>`, `toggleDarkMode`, `isDark` | — |

> **Singleton Firebase :** `src/lib/firebase.ts` initialise `initializeApp` / `getAuth` /
> `getFirestore` **une seule fois**. Les composables consomment ce singleton, jamais de
> réinitialisation. Un guard Vue Router n'appelle jamais `useFirebaseAuth()` (hors contexte
> `setup`) : il utilise le singleton `auth` + `await auth.authStateReady()`.

### 4.5 Composants (`src/components/`)

- **layout/** : `AppLayout.vue` (header + navs + `<KeepAlive><router-view>`), `AuthLayout.vue`
- **pages/** : 1 par route, dont `CalendarWeekPage`, `CalendarMonthPage`, `DiscoverExplorePage`,
  `DiscoverSeasonPage`, `DiscoverComingUpPage`, `LibraryExplorePage`, `LibraryPlanToWatchPage`,
  `LibraryCompletedPage`, `LoginPage`, `StatsPage`, `OnboardingPage`, + stub `CalendarListPage`
- **ui/** : composants atomiques (cards, navs, modals, sheets, toast). Trois modales partagent
  la classe `.modal-backdrop` : `AnimeModal`, `LogoutConfirmModal`, `RecEngineModal`

---

## 5. Graphe de dépendances (boot & flux principal)

```
main.ts
  └── App.vue ──────────────────────────── provide(isBootingKey)
        ├── usePersistence → useFirestore → lib/firebase (singleton)
        │                  → watchDebounced(animeCalendarData) → saveToDatabase
        ├── useSync → useJikanApi (fetch*WithMeta) → utils/helpers (fetchWithRetry)
        ├── useRecommendations → useJikanApi + utils/recEngine + utils/idb
        └── stores/anime ── stores/ui

AppLayout.vue
  ├── AppHeader (SearchInput → useJikanApi.searchAnime)
  ├── PrimaryNav / SecondaryNav (CalendarNavControls → stores/anime.setDate)
  ├── SyncIndicator (useSync.isSyncing)
  └── <KeepAlive><router-view/></KeepAlive>
```

---

## 6. Séquence de boot

**Ordre d'orchestration contractuel (DEC-50), couvert par `src/App.spec.ts` :**

```
1. loadFromDatabase()          // Firestore → store (+ migration aanime_*, réconciliation, fallback localStorage)
2. syncAnimeUpdates()          // sync Jikan (batch 2 / 2 s) — le rescore en dépend
3. buildRelationMemory()       // graphe de relations depuis IDB
4. reScorePool()               // re-score des pools de recommandations
5. startBackgroundRelationFetch()   // worker de fond — FIRE-AND-FORGET (throttlé)

finally:
   isBooting.value = false                            // libère LoadingOverlay
   document.getElementById('boot-loader')?.remove()   // loader pré-Vue (DEC-72)
```

**Deux écrans de chargement distincts, pour deux fenêtres distinctes :**
- **`#boot-loader`** — HTML/CSS statique inline dans `index.html`, **hors de `<div id="app">`**.
  Couvre la fenêtre pré-mount (bundle pas encore parsé, aucun composant Vue ne peut s'afficher).
  Supprimé par `.remove()` dans le `finally` — exception R-CODE-4 documentée. Il est
  `position: fixed` et **intercepte tous les pointer events** tant qu'il est là.
- **`<LoadingOverlay>`** — composant Vue monté **au niveau racine d'`App.vue`**, hors de la
  gate `v-if="route.meta.requiresAuth"`. Ce placement est la correction de DEC-59 : le bug
  n'était pas une injection ratée mais un **placement** sous gate auth.

**Architecture 2 phases (US-155) :** le paint est immédiat depuis le cache localStorage
(phase 1), la réconciliation Firestore se fait en fond par comparaison de timestamps
(phase 2), ce qui a supprimé l'écran blanc d'environ 6 s au démarrage.

> ⚠️ **À confirmer par lecture du code (R3) avant toute US touchant le boot :** l'articulation
> exacte entre l'ordre DEC-50 ci-dessus et l'architecture 2 phases n'a jamais été
> re-documentée après US-155. La question précise : `isBooting` est-il libéré après l'étape 1
> ou après l'étape 2 ? Ne pas spécifier de correctif sur le boot sans avoir tranché.

---

## 7. Réseau, cache & registre des clés

| Aspect | Implémentation |
|---|---|
| Retry / 429 | `fetchWithRetry` — 3 tentatives, backoff exponentiel |
| Batch sync | `useSync` : batch de 2, intervalle 2 s |
| Throttle worker | 1,1 s par requête, **uniquement si `fromNetwork === true`** (pattern `*WithMeta`, DEC-51) |
| Cache relations | IndexedDB, persistant — **sans expiration** |
| Cache recommandations | IndexedDB, TTL 7 j |
| Cache saisons | localStorage, TTL 24 h. **Le cache périmé est servi si le fetch échoue** ; `error.value` est renseigné mais jamais affiché (DEC-114) |
| Timestamps de sync | localStorage `aanime_sync_ts`, TTL 6 h |

### Registre des clés localStorage

Toutes préfixées `aanime_` (DEC-85), avec migration transparente au boot : lecture de
l'ancienne clé si la nouvelle est absente, réécriture sous le préfixe, suppression de
l'ancienne. Migration portée par `usePersistence.loadFromDatabase`.

| Clé actuelle | Ancienne clé (migrée) | Usage |
|---|---|---|
| `aanime_calendar` | `animeCalendar` | Document de planning (cache local du store) |
| `aanime_sync_ts` | `anime_sync_ts_v1` | Timestamp de dernière sync Jikan |
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

Le bus DOM du vanilla a été intégralement remplacé :

| Ancien événement DOM | Remplacement Vue |
|---|---|
| `store:changed` → save | `watchDebounced(animeCalendarData, saveToDatabase, { deep, 1000 })` dans `usePersistence` |
| `ui:refresh` | réactivité Pinia native |
| `anime:add` / `anime:added` | actions du store + appel direct du composable |
| `recs:heart` / `recs:remove` | `emit` composant → handler page → `useRecommendations` |
| `nav:next` / `nav:prev` | `useSwipe` (@vueuse) → `store.setDate` |

`usePersistence` porte le `watchDebounced` derrière un flag de module (`watchInitialized`) —
sans ce flag, les listeners s'empilent. `suppressPersist` permet les mutations silencieuses
(import MyAnimeList).

---

## 9. Pipeline du moteur de recommandations

```
1. fetchRecPool(context)        → pools bruts (saison upcoming / top finished)
2. buildTasteProfile(history)   → profil de goûts (genres, thèmes, studios, négatifs)
3. buildRelationMemory()        → graphe de relations depuis IDB (séquelles / préquelles)
4. scorePool(pool, profile)     → _relevanceScore par item
5. applyPreset(preset)          → _presetScore (chips : Hidden gems, Trending…)
6. getNextBatch(context, size)  → batch paginé — dédup via dedupeByMalId AVANT le slice (DEC-74)
7. extractBecauseYouWatched()   → section « parce que vous avez vu X »
8. getSlotFillSuggestions(list) → remplissage des jours vides de la semaine
```

**Onboarding (`utils/onboardingFilter.ts`)** : `selectOnboardingSuggestions` produit
**8 suggestions** scorées sur les genres choisis ; `buildSeedEntry` retourne
`{ ...anime, id, state }` — **il ne pose jamais `day`** (DEC-115), ce qui est la piste
principale du défaut d'atterrissage sur le calendrier.

---

## 10. Qualité & CI

`ci.yml` rejoue à chaque push : `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`.
Filet de régression du boot : `src/App.spec.ts` (smoke test) monte `App` et asserte que les
5 fonctions d'orchestration sont appelées. **Ne pas casser cet ensemble.**

Les compteurs, l'état de la suite E2E et la dette technique ouverte vivent dans `STATE.md`.
Les règles de test opposables à Gemini vivent dans `AGENTS.md`.
