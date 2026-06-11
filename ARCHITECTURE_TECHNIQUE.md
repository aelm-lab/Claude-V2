# ARCHITECTURE_TECHNIQUE.md — Vue technique du système Aanime (Vue 3)

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** décrire l'architecture **technique** cible telle qu'elle est réellement
> implémentée après la migration (Phases 0→7) et le sprint de remédiation EPIC-1.
> C'est la carte « comment c'est construit ». Le pendant fonctionnel est
> `ARCHITECTURE_FONCTIONNELLE.md` ; les deux se lisent ensemble.
>
> **État de référence :** fin de session 6 (EPIC-1 clos : P0 reco + P1 throttle +
> P1 hiatus corrigés, code vanilla supprimé, dark mode aligné, nav dédupliquée).

---

## 1. Stack & principe directeur

| Couche | Techno |
|---|---|
| Framework | Vue 3 (`<script setup lang="ts">`) |
| Langage | TypeScript strict (zéro `any`) |
| State global | Pinia (`stores/anime.ts`, `stores/ui.ts`) |
| Routing | Vue Router 4 (11 routes, guards auth/guest) |
| Réactivité utilitaire | `@vueuse/core` (useDark, useSwipe, useIntersectionObserver, watchDebounced, onClickOutside, onKeyStroke) |
| Build | Vite 6 + `@vitejs/plugin-vue` |
| Auth / DB | Firebase v12 (Auth email-link + Firestore) |
| API externe | Jikan v4 (publique, sans clé) |
| Cache | IndexedDB (relations, recommendations) + localStorage (TTL, timestamps, préférences) |
| Styles | `style.css` global (variables CSS, dark mode `html.dark`) |
| Tests | Vitest (69 tests) + CI GitHub Actions |

**Principe directeur :** séparation stricte des responsabilités en couches, avec une
règle de dépendance **descendante uniquement**. Une couche ne dépend jamais d'une
couche située au-dessus d'elle.

---

## 2. Les couches et la règle de dépendance

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSANTS (.vue)   pages / layout / ui                     │  UI + réactivité locale
│   ▼ peut importer                                            │
├─────────────────────────────────────────────────────────────┤
│  COMPOSABLES (useXxx.ts)   logique réutilisable réactive     │  orchestration, I/O, réseau
│   ▼ peut importer                                            │
├─────────────────────────────────────────────────────────────┤
│  STORES (Pinia)   état global partagé                        │  état pur, zéro I/O
│   ▼ peut importer                                            │
├─────────────────────────────────────────────────────────────┤
│  UTILS (.ts purs)   fonctions sans Vue, sans I/O             │  calculs purs, testables
│   ▼ peut importer                                            │
├─────────────────────────────────────────────────────────────┤
│  TYPES (.ts)   interfaces partagées                          │  contrat, zéro logique
└─────────────────────────────────────────────────────────────┘
```

**Règles non-négociables (rappel CLAUDE.md §7) :**
- Un composant `.vue` ne fait **jamais** de `fetch`, ni d'accès `localStorage`/IndexedDB, ni de logique métier lourde.
- Un composable n'expose vers l'extérieur que des `readonly ref` / `computed`.
- Le store Pinia ne fait **aucun I/O** (la persistance est un `watch` externe dans `usePersistence`).
- Les utils n'importent **jamais** Vue.
- Zéro manipulation DOM directe (sauf 2 exceptions documentées : download Blob dans `useICS`, `<input file>.click()` dans l'import MAL).

---

## 3. Carte des modules par couche

### 3.1 Types (`src/types/`)
| Fichier | Contenu |
|---|---|
| `anime.ts` | `AnimeEntry`, `AnimeState`, `AnimeStatus`, `WeekDay`, `CardStatus`, `AnimeEpisodeInfo`, `JstParseResult`, `RecencyBucket` |
| `recs.ts` | `TasteProfile`, `RecBadge`, `RecSignal`, `RecPreset`, `RecContext`, `HistoryItem`, `RecAction` |
| `persistence.ts` | `ScheduleDocument`, `IdbStoreName` |

> Règle absolue : aucun type n'est inventé ailleurs. Tout vient de `TYPES_CONTRACT.md`.

### 3.2 Utils purs (`src/utils/`) — zéro Vue, 100 % testés
| Fichier | Exports clés | Tests |
|---|---|---|
| `jst.ts` | `parseJSTToLocal` (ancre 1970, JST→local) | ✅ 5 |
| `normalize.ts` | `normalizeAnime` (forme canonique) | ✅ 7 |
| `episodeInfo.ts` | `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus` (**seuil hiatus 14j — source unique**) | ✅ 7 |
| `helpers.ts` | `fetchWithRetry` (backoff 429), `getWeekNumber`, `escapeHTML`, `BASE_URL` | ✅ 6 |
| `recEngine.ts` | `buildTasteProfile`, `scorePool`, `assignBadge`, `extractBecauseYouWatched`, `buildNextBatch`, `applyPreset`, `RelationMemory` | ✅ 15 |
| `ics.ts` | `buildICSContent` (génération texte pure) | ✅ 6 |
| `malImport.ts` | `parseMalXml` (DOMParser, pur) | ✅ 8 |
| `idb.ts` | `idbGet`, `idbSet` (wrapper IndexedDB) | — |
| `constants.ts` | `POSTER_PLACEHOLDER` | — |

### 3.3 Stores Pinia (`src/stores/`)
| Store | Responsabilité | I/O ? |
|---|---|---|
| `anime.ts` | `animeCalendarData[]`, `currentDate`, `currentView` + actions `addAnime` (upsert), `addAnimeSilent`, `removeAnime`, `setDate`. Transitions de state, auto-vault. | **Aucun** (pur) |
| `ui.ts` | État des overlays : modal ouvert + contexte, sheet recency, panneau ep-override. Remplace l'ancien `activeWindowClickHandler` + `modal.style.display`. | Aucun |

### 3.4 Composables services (`src/composables/`)
| Composable | Rôle | Réseau | Auth |
|---|---|---|---|
| `useFirebaseAuth.ts` | Auth email-link, `onAuthStateChanged` (niveau module) | — | initie |
| `useFirestore.ts` | `schedules/{uid}` brut + `handleFirestoreError` | — | oui |
| `usePersistence.ts` | `loadFromDatabase`, `saveToDatabase`, `watchDebounced` (remplace `store:changed`), `staleDataWarning`, fallback localStorage | — | dégrade |
| `useJikanApi.ts` | `searchAnime`, `fetchAnimeById`, `fetchCurrentSeason`, `fetchUpcomingSeason`, `fetchAnimeRelations(WithMeta)`, `fetchAnimeRecommendations(WithMeta)` | Jikan | non |
| `useSync.ts` | `syncAnimeUpdates` (batch 2/2s), `startBackgroundRelationFetch` (throttle **conditionnel réseau**), `isSyncing`, `clearSyncTimestamp` | Jikan | non |
| `useRecommendations.ts` | `fetchRecPool`, `getNextBatch`, `reScorePool`, `buildRelationMemory`, `getBecauseYouWatchedBatch`, `getSlotFillSuggestions`, `getSeasonNudges`, `saveRec`, `saveAsCompleted` | Jikan (cache) | non |
| `useEpisodeInfo.ts` | wrappe `utils/episodeInfo` : `getEpisodeInfo`, `getStatus`, `checkIsOnHiatus` | — | non |
| `useICS.ts` | `downloadICS` (Blob + iOS) | — | non |
| `useMalImport.ts` | `parseMalXml` + `importMalFile` (FileReader, `addAnimeSilent`) | — | non |
| `useToast.ts` | `showToast`, `hideToast` | — | non |
| `useTheme.ts` | `useDark()` → classe `dark` sur `<html>`, `toggleDarkMode`, `isDark` | — | non |

> **Singleton Firebase :** `src/lib/firebase.ts` initialise `initializeApp`/`getAuth`/`getFirestore`
> une seule fois. Les composables consomment ce singleton (jamais de réinit).

### 3.5 Composants (`src/components/`)
- **layout/** : `AppLayout.vue` (header + navs + `<KeepAlive><router-view>`), `AuthLayout.vue`.
- **pages/** : 1 par route (10 pages + 1 stub `CalendarListPage`).
- **ui/** : ~30 composants atomiques réutilisables (cards, navs, modals, sheets, toast…).

---

## 4. Graphe de dépendances réel (boot & flux principal)

```
main.ts
  └── App.vue ──────────────────────────── provide(isBootingKey)
        ├── useRoute (layout switch auth/guest)
        ├── usePersistence → useFirestore → lib/firebase (singleton)
        │                  → watchDebounced(animeCalendarData) → saveToDatabase
        ├── useSync → useJikanApi (fetch*WithMeta) → utils/helpers (fetchWithRetry)
        │           → stores/anime
        ├── useRecommendations → useJikanApi + utils/recEngine + utils/idb
        └── stores/anime ── stores/ui

AppLayout.vue
  ├── AppHeader (SearchInput → useJikanApi.searchAnime)
  ├── PrimaryNav / SecondaryNav (CalendarNavControls → stores/anime.setDate)
  ├── SyncIndicator (useSync.isSyncing)
  └── <KeepAlive><router-view/></KeepAlive>
        └── pages → composables (jamais de fetch direct dans la page)
```

---

## 5. Séquence de boot (orchestration `App.vue` — post EPIC-1)

C'est le cœur corrigé par **US-102 (P0)** et **DEC-49**. Ordre **strict** :

```ts
onMounted(async () => {
  try {
    await loadFromDatabase()          // 1. Firestore → store (+ fallback localStorage)
    await syncAnimeUpdates()          // 2. sync Jikan (batch 2/2s) — AWAIT (le rescore en dépend)
    await buildRelationMemory()       // 3. reconstruit le graphe de relations (IDB)
    reScorePool()                     // 4. re-score les pools de recommandations
    startBackgroundRelationFetch()    // 5. worker de fond — FIRE-AND-FORGET (throttlé)
  } catch (e) {
    console.error('[App] boot error', e)
  } finally {
    isBooting.value = false           // libère LoadingOverlay
  }
})
```

**Pourquoi cet ordre :** `buildRelationMemory` + `reScorePool` doivent travailler sur des
données **déjà synchronisées** → `syncAnimeUpdates` est `await` (DEC-49, ajuste DEC-32).
`startBackgroundRelationFetch` est la tâche longue (throttle réseau) → fire-and-forget pour
ne pas bloquer le rendu.

> **Filet de régression :** `src/App.spec.ts` (smoke test, US-109) monte App et asserte
> que les 5 fonctions d'orchestration sont appelées. Le 3ᵉ test garantit que
> `buildRelationMemory`/`reScorePool` sont câblés — c'est le contrat qui empêche le
> retour du bug P0.

---

## 6. Flux de données & réactivité (remplacement du bus DOM vanilla)

Le vanilla utilisait `document.dispatchEvent(CustomEvent)`. Tout passe désormais par
la réactivité Pinia :

| Ancien événement DOM | Remplacement Vue |
|---|---|
| `store:changed` → save | `watchDebounced(animeCalendarData, saveToDatabase, {deep, 1000ms})` dans `usePersistence` |
| `ui:refresh` | réactivité Pinia native (les pages lisent le store) |
| `anime:add` / `anime:added` | actions du store + appel direct composable |
| `recs:heart` / `recs:remove` | `emit` composant → handler page → `useRecommendations` |
| `nav:next` / `nav:prev` | `useSwipe` (@vueuse) → `store.setDate` |

**Persistance :** `usePersistence` porte le `watchDebounced` (flag module `watchInitialized`
pour éviter l'empilement). Le store reste sans I/O (DEC-26). `suppressPersist` permet les
mutations silencieuses (import MAL).

---

## 7. Couche réseau (Jikan) & politique de cache

| Aspect | Implémentation |
|---|---|
| Retry / 429 | `fetchWithRetry` (helpers) — 3 tentatives, backoff exponentiel |
| Batch sync | `useSync` : batch de 2, intervalle 2s |
| Throttle worker | **1,1s par requête, UNIQUEMENT si `fromNetwork === true`** (US-106) |
| Cache relations | IndexedDB, persistant (jamais expiré) |
| Cache recommandations | IndexedDB, TTL 7 j |
| Cache saisons | localStorage TTL 24 h (`seasons_now_v1`, `seasons_upcoming_v1`) |
| Timestamps sync | localStorage `anime_sync_ts_v1`, TTL 6 h |

**Pattern `*WithMeta` (US-106) :** `fetchAnimeRelationsWithMeta` / `fetchAnimeRecommendationsWithMeta`
retournent `{ data, fromNetwork }`. Les versions publiques (`fetchAnimeRelations`,
`fetchAnimeRecommendations`) délèguent et jettent le flag (signature `Promise<unknown[]>`
inchangée → appelants existants intacts). Le worker n'attend 1,1s **que** sur un vrai
appel réseau → un cache 100 % chaud = boot quasi instantané (avant : ~11 min / 300 animes).

---

## 8. Pipeline du moteur de recommandations

```
1. fetchRecPool(context)        → pools bruts (saison upcoming / top finished)
2. buildTasteProfile(history)   → profil de goûts (genres, thèmes, studios, négatifs)
3. buildRelationMemory()        → graphe relations depuis IDB (séquelles/préquelles)
4. scorePool(pool, profile)     → _relevanceScore par item
5. applyPreset(preset)          → _presetScore (chips : Hidden gems, Trending…)
6. getNextBatch(context, size)  → batch paginé pour infinite scroll
7. extractBecauseYouWatched()   → section « parce que vous avez vu X »
8. getSlotFillSuggestions(list) → suggestions de remplissage des jours vides (semaine)
```

> **Dette connue (P8-01 / EPIC-3) :** `scorePool` lit `item.studios` (pluriel) que
> `normalizeAnime` ne produit pas (il produit `studio` singulier) → dimension studio du
> scoring **inerte**. Reproduit fidèlement le vanilla, correction planifiée EPIC-3.

---

## 9. Qualité, CI & points de vigilance technique

**Filet (US-109) :**
- `.github/workflows/ci.yml` : `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`, sur push/PR.
- `src/App.spec.ts` : smoke test d'orchestration boot.
- 69 tests Vitest verts.

**Règles process gravées (R1/R2/R3) :** voir `AGENTS.md` (gouvernance Gemini).

**Dette technique ouverte (détail dans ROADMAP.md) :**
- Bundle monolithique ~749 kb + warning chunking `idb.ts` (import statique + dynamique) → code-split EPIC-2.
- Pas de file globale Jikan centralisée (navigation rapide = risque 429 ponctuel) → EPIC-2.
- `console.error` isolés au lieu d'un store d'erreurs / monitoring → EPIC-2 (Sentry).
- Scroll restore dupliqué dans plusieurs pages → composable `useScrollKeeper` (EPIC-2).
- Champ `onHiatus?` persisté mais plus écrit (cosmétique) → nettoyage type EPIC-3.
