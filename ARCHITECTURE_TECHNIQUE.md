# ARCHITECTURE_TECHNIQUE.md — Vue technique du système Aanime (Vue 3)

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** décrire l'architecture **technique** réellement implémentée.
> Le pendant fonctionnel est `ARCHITECTURE_FONCTIONNELLE.md` ; les deux se lisent ensemble.
>
> **État de référence : session 16 (dual audit).** EPIC-1 + EPIC P0 + EPIC-2 + EPIC-3 clos.
> **84** tests unit · **26 specs / 30 tests** E2E · build **~717 kb** (~3.7 s) · zéro `any`.

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
| Cache | IndexedDB (relations, recommendations) + localStorage (**clés préfixées `aanime_`**, cf. §7) |
| Styles | `style.css` global (variables CSS, dark mode `html.dark`) |
| Tests | Vitest (**84**) + Playwright (**26 specs / 30 tests**) + CI GitHub Actions |

**Principe directeur :** séparation stricte des responsabilités en couches, avec une
règle de dépendance **descendante uniquement**.

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

**Règles non-négociables (rappel CLAUDE.md §6) :**
- Un composant `.vue` ne fait **jamais** de `fetch`, ni d'accès `localStorage`/IndexedDB, ni de logique métier lourde.
- Un composable n'expose vers l'extérieur que des `readonly ref` / `computed`.
- Le store Pinia ne fait **aucun I/O** (la persistance est un `watch` externe dans `usePersistence`).
- Les utils n'importent **jamais** Vue.
- Zéro manipulation DOM directe (sauf exceptions documentées : download Blob `useICS`, `<input file>.click()` import MAL, `DOMParser`, `getElementById('boot-loader').remove()` dans `App.vue` — DEC-72).

> ⚠️ **Violations relevées par le dual audit s16** (à corriger, EPIC-5-tech) :
> `usePersistence` mute directement `store.animeCalendarData` et `store.suppressPersist`
> hors action Pinia, et porte de la logique métier + des toasts (couplage). → US-157 (P1).

---

## 3. Carte des modules par couche

### 3.1 Types (`src/types/`)
| Fichier | Contenu |
|---|---|
| `anime.ts` | `AnimeEntry`, `AnimeState`, `AnimeStatus`, `WeekDay`, `CardStatus`, `AnimeEpisodeInfo`, `JstParseResult`, `RecencyBucket` |
| `recs.ts` | `TasteProfile`, `RecBadge`, `RecSignal`, `RecPreset`, `RecContext`, `HistoryItem`, `RecAction` |
| `persistence.ts` | `ScheduleDocument`, `IdbStoreName` |

> Règle absolue : aucun type n'est inventé ailleurs. Tout vient de `TYPES_CONTRACT.md`.
> Confirmé s16 : pas de `syncStatus` dans `AnimeEntry`.

### 3.2 Utils purs (`src/utils/`) — zéro Vue, 100 % testés
| Fichier | Exports clés | Tests |
|---|---|---|
| `jst.ts` | `parseJSTToLocal` (ancre 1970, JST→local) | ✅ |
| `normalize.ts` | `normalizeAnime` (forme canonique, **produit toujours `studios: string[]`** — DEC-86) | ✅ |
| `episodeInfo.ts` | `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus` (**seuil hiatus 14j — source unique**) | ✅ ⚠️ `getCardStatus` ne gère pas `'Continuing'` → US-154 |
| `helpers.ts` | `fetchWithRetry` (backoff 429), `getWeekNumber`, `escapeHTML`, `dedupeByMalId`, `BASE_URL` | ✅ |
| `recEngine.ts` | `buildTasteProfile`, `scorePool`, `assignBadge`, `extractBecauseYouWatched`, `buildNextBatch`, `applyPreset`, `RelationMemory` | ✅ |
| `ics.ts` | `buildICSContent` (génération texte pure) | ✅ |
| `malImport.ts` | `parseMalXml` (DOMParser, pur) | ✅ |
| `idb.ts` | `idbGet`, `idbSet` (wrapper IndexedDB, import statique) | — |
| `constants.ts` | `POSTER_PLACEHOLDER` (**source unique** — US-132/DEC-84) | — |

### 3.3 Stores Pinia (`src/stores/`)
| Store | Responsabilité | I/O ? |
|---|---|---|
| `anime.ts` | `animeCalendarData[]`, `currentDate`, `currentView` + actions `addAnime` (upsert, reset `episodeOverride` — DEC-84), `addAnimeSilent`, `removeAnime`, `setDate`, **`setData`**, `clearAll`. Transitions de state, auto-vault. | **Aucun** (pur) |
| `ui.ts` | État des overlays : modal ouvert + contexte, sheet recency, panneau ep-override. | Aucun |

> Confirmé s16 : l'action de remplacement de tableau est **`setData`** (+ `clearAll`).
> Il n'existe **pas** de `setAllData`.

### 3.4 Composables services (`src/composables/`)
| Composable | Rôle | Réseau | Auth |
|---|---|---|---|
| `useFirebaseAuth.ts` | Auth email-link, `onAuthStateChanged` (niveau module) | — | initie |
| `useFirestore.ts` | `schedules/{uid}` brut + `handleFirestoreError` (`loadSchedule`, `saveSchedule`) | — | oui |
| `usePersistence.ts` | `loadFromDatabase` (+ migration clés `aanime_*` + réconciliation), `saveToDatabase`, `watchDebounced`, `staleDataWarning`, fallback localStorage | — | dégrade |
| `useJikanApi.ts` | `searchAnime`, `fetchAnimeById`, `fetchCurrentSeason`, `fetchUpcomingSeason`, `fetchAnimeRelations(WithMeta)`, `fetchAnimeRecommendations(WithMeta)` | Jikan | non |
| `useSync.ts` | `syncAnimeUpdates` (batch 2/2s), `startBackgroundRelationFetch` (throttle conditionnel réseau), `isSyncing`, `clearSyncTimestamp` | Jikan | non |
| `useRecommendations.ts` | `fetchRecPool`, `getNextBatch`, `reScorePool`, `buildRelationMemory`, `getBecauseYouWatchedBatch`, `getSlotFillSuggestions`, `getSeasonNudges`, `saveRec`, `saveAsCompleted` ; `fetchTopFinishedAnime` encore inline (extraction = US-165) | Jikan (cache) | non |
| `useEpisodeInfo.ts` | wrappe `utils/episodeInfo` : `getEpisodeInfo`, `getStatus`, `checkIsOnHiatus` | — | non |
| `useICS.ts` | `downloadICS` (Blob + iOS) | — | non |
| `useMalImport.ts` | `parseMalXml` + `importMalFile` (FileReader, `addAnimeSilent`) | — | non |
| `useToast.ts` | `showToast`, `hideToast` | — | non |
| `useTheme.ts` | `useDark()` → classe `dark` sur `<html>`, `toggleDarkMode`, `isDark` | — | non |

> **Singleton Firebase :** `src/lib/firebase.ts` initialise `initializeApp`/`getAuth`/`getFirestore`
> une seule fois. Les composables consomment ce singleton (jamais de réinit).
> ⚠️ **Audit s16 — couverture de tests** : aucun de ces composables n'a de `.spec` adjacent
> (`usePersistence`/`firebase`/`router`/`store`/`utils` ont des specs ; pas `useRecommendations`,
> `useSync`, `useJikanApi`, `useFirebaseAuth`, `useMalImport`, `useEpisodeInfo`, etc.). → US-156 (P1).

### 3.5 Composants (`src/components/`)
- **layout/** : `AppLayout.vue` (header + navs + `<KeepAlive><router-view>`), `AuthLayout.vue`.
- **pages/** : 1 par route (10 pages + 1 stub `CalendarListPage`).
- **ui/** : ~30 composants atomiques réutilisables (cards, navs, modals, sheets, toast…).

---

## 4. Graphe de dépendances réel (boot & flux principal)

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

## 5. Séquence de boot (orchestration `App.vue`)

Cœur corrigé par **US-102 (P0)** et **DEC-49/50**. Ordre **strict** :

```ts
onMounted(async () => {
  try {
    await loadFromDatabase()          // 1. Firestore → store (+ migration aanime_*, réconciliation, fallback localStorage)
    await syncAnimeUpdates()          // 2. sync Jikan (batch 2/2s) — AWAIT (le rescore en dépend)
    await buildRelationMemory()       // 3. reconstruit le graphe de relations (IDB)
    reScorePool()                     // 4. re-score les pools de recommandations
    startBackgroundRelationFetch()    // 5. worker de fond — FIRE-AND-FORGET (throttlé)
  } catch (e) {
    console.error('[App] boot error', e)
  } finally {
    isBooting.value = false           // libère LoadingOverlay
    document.getElementById('boot-loader')?.remove()   // loader pré-Vue (DEC-72)
  }
})
```

> **Filet de régression :** `src/App.spec.ts` (smoke test, US-109) monte App et asserte
> que les 5 fonctions d'orchestration sont appelées. Ne pas casser cet ensemble.
>
> ⚠️ **Finding audit s16 (US-155, P1) — boot bloquant.** L'`await syncAnimeUpdates()`
> (batchs réseau throttlés 2 s) retarde `isBooting=false` → l'overlay plein écran reste
> affiché pendant toute la sync, proportionnellement à la taille de la liste. *Impact user :
> spinner de plusieurs secondes au démarrage.* Piste : lever l'overlay après le load local,
> garder la sync en arrière-plan. À spécifier avec un E2E R4 (overlay disparaît avant fin de sync).

---

## 6. Flux de données & réactivité (remplacement du bus DOM vanilla)

| Ancien événement DOM | Remplacement Vue |
|---|---|
| `store:changed` → save | `watchDebounced(animeCalendarData, saveToDatabase, {deep, 1000ms})` dans `usePersistence` |
| `ui:refresh` | réactivité Pinia native |
| `anime:add` / `anime:added` | actions du store + appel direct composable |
| `recs:heart` / `recs:remove` | `emit` composant → handler page → `useRecommendations` |
| `nav:next` / `nav:prev` | `useSwipe` (@vueuse) → `store.setDate` |

**Persistance :** `usePersistence` porte le `watchDebounced` (flag module `watchInitialized`).
Le store reste sans I/O (DEC-26). `suppressPersist` permet les mutations silencieuses (import MAL).

---

## 7. Couche réseau (Jikan), cache & registre des clés localStorage

| Aspect | Implémentation |
|---|---|
| Retry / 429 | `fetchWithRetry` (helpers) — 3 tentatives, backoff exponentiel |
| Batch sync | `useSync` : batch de 2, intervalle 2 s |
| Throttle worker | 1,1 s par requête, **uniquement si `fromNetwork === true`** (US-106, pattern `*WithMeta`) |
| Cache relations | IndexedDB, persistant — ⚠️ **sans expiration** (finding audit s16, P2 : relations périmées indéfiniment) |
| Cache recommandations | IndexedDB, TTL 7 j |
| Cache saisons | localStorage, TTL 24 h (`aanime_seasons_now`, `aanime_seasons_upcoming`) |
| Timestamps sync | localStorage `aanime_sync_ts`, TTL 6 h |

### Registre des clés localStorage (toutes préfixées `aanime_` — US-133/DEC-85)

| Clé actuelle | Ancienne clé (migrée au boot) | Usage |
|---|---|---|
| `aanime_calendar` | `animeCalendar` | Document de planning (cache local du store) |
| `aanime_sync_ts` | `anime_sync_ts_v1` | Timestamp dernière sync Jikan |
| `aanime_negative_removed_ids` | `negative_removed_ids` | IDs « pas intéressé » |
| `aanime_email_for_signin` | `emailForSignIn` | Email magic-link |
| `aanime_recs_incoming` | `recs_incoming_v3` | Cache recos Discover |
| `aanime_recs_library` | `recs_library_v2` | Cache recos Library |
| `aanime_season_nudges` | `season_nudges_v1` | Nudges séquelles |
| `aanime_season_nudge_dismissed` | `season_nudge_dismissed_v1` | Nudges écartés |
| `aanime_seasons_now` | `seasons_now_v1` | Cache saison en cours |
| `aanime_seasons_upcoming` | `seasons_upcoming_v1` | Cache saison à venir |

> Migration transparente dans `usePersistence.loadFromDatabase` : lecture de l'ancienne clé
> si la nouvelle est absente, ré-écriture sous le préfixe `aanime_`, suppression de l'ancienne.
> **TTL généralisé sur le cache `aanime_*`** = idée reportée au **Vault fonctionnalités** (ROADMAP).

---

## 8. Pipeline du moteur de recommandations

```
1. fetchRecPool(context)        → pools bruts (saison upcoming / top finished)
2. buildTasteProfile(history)   → profil de goûts (genres, thèmes, studios, négatifs)
3. buildRelationMemory()        → graphe relations depuis IDB (séquelles/préquelles)
4. scorePool(pool, profile)     → _relevanceScore par item
5. applyPreset(preset)          → _presetScore (chips : Hidden gems, Trending…)
6. getNextBatch(context, size)  → batch paginé (dédup via dedupeByMalId AVANT slice — DEC-74)
7. extractBecauseYouWatched()   → section « parce que vous avez vu X »
8. getSlotFillSuggestions(list) → suggestions de remplissage des jours vides (semaine)
```

> ✅ **Dette P8-01 RÉSOLUE (US-134/DEC-86).** `scorePool` lit `item.studios` ; `normalizeAnime`
> produit désormais toujours `studios: string[]` → dimension studio du scoring **active**.

---

## 9. Qualité, CI & dette technique ouverte

**Filet (US-109) :** `ci.yml` (`npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`) + `src/App.spec.ts`.

**Règles process gravées (R1–R6) :** voir `AGENTS.md` + `CLAUDE.md §7`.

**Dette technique ouverte (détail `ROADMAP.md` / `BACKLOG.md`) :**
- **Findings dual audit s16** (P0/P1) : `saveToDatabase` sans try/catch (US-153, **P0**) · `getCardStatus` `Continuing→Finished` (US-154) · boot bloquant (US-155) · zéro test unit composables (US-156) · persistance mute le store hors action (US-157) · cast legacy non normalisé (US-158).
- **Fichiers parasites racine** : `diff.cjs`, `replace.js`, `size.cjs`, `find_usages.cjs`, `sme.json`, `e2e_out.txt`, `test_out.txt`, `test_output.txt`, `test_pid.txt` → US-159-CLEANUP (R-SCOPE-1).
- **Dette CSS F18–F24** → US CSS groupée.
- ✅ **Résolu s15** : `onHiatus?` supprimé du type (US-132) ; `POSTER_PLACEHOLDER` source unique (US-132) ; clés `aanime_*` (US-133) ; studios (US-134).
