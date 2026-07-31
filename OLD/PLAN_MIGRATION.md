# PLAN_MIGRATION.md — Plan d'action en 7 phases + architecture cible

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat. C'est le « quoi construire » et dans quel ordre.

---

## A. Vue d'ensemble des phases

| Phase | Intitulé | Durée estimée | Risque | Dépend de |
|---|---|---|---|---|
| 0 | Scaffold (setup projet) | 1–2 j | Faible | — |
| 1 | Logique pure (types + utils) | 2–3 j | Faible | 0 |
| 2 | Store Pinia + composables services | 3–4 j | Moyen | 1 |
| 3 | Router + layouts | 1 j | Faible | 2 |
| 4 | Composants atomiques (ui/) | 3–4 j | Faible | 1 |
| 5 | Pages | 5–7 j | Moyen | 2, 3, 4 |
| 6 | Modals & sheets | 3–4 j | **Élevé** | 5 |
| 7 | Branchement final + nettoyage | 2–3 j | Faible | 6 |

**Principe directeur :** on migre **verticalement** (types → logique → composant → test), jamais horizontalement. Les éléments les plus risqués (modal, incoming) passent **en dernier**, quand le store est stable.

---

## B. Détail des phases

### Phase 0 — Scaffold (1–2 j)

1. Branche `feat/vue3-migration` (le vanilla reste sur `main`).
2. Supprimer `@vitejs/plugin-react` + React.
3. Installer : `vue`, `vue-router@4`, `pinia`, `@vueuse/core`, `typescript`, `@vitejs/plugin-vue`, `@vue/eslint-config-typescript`.
4. Reconfigurer `vite.config.ts` (`plugin-vue`), garder temporairement les 2 entry points.
5. `tsconfig.json` strict.
6. ESLint + structure de dossiers vide (cf. CLAUDE.md §6).
7. **Décision CSS :** conserver `style.css` existant (pas de Tailwind).

### Phase 1 — Logique pure (2–3 j)
Porter en TypeScript, **sans aucun DOM**, les fichiers déjà purs :

| Source | Destination | Note |
|---|---|---|
| `src/utils.js` | `src/utils/helpers.ts` | typer `AnimeEpisodeInfo`, `CardStatus` |
| `src/normalize.js` | `src/utils/normalize.ts` | typer `AnimeEntry` |
| `src/rec-engine.js` | `src/utils/recEngine.ts` | typer `TasteProfile`, `RelationMemory` |
| `src/idb.js` | `src/utils/idb.ts` | typer les stores |
| `src/ics.js` | `src/utils/ics.ts` | |
| `parseMalXml` (de `mal-import.js`) | `src/utils/malImport.ts` | partie pure uniquement |

**Livrable :** dossier `src/utils/` typé + `src/types/` rempli.

### Phase 2 — Store + composables services (3–4 j)
- `stores/anime.ts` (Pinia) : `animeCalendarData`, `currentDate`, `currentView` + actions `addAnime`, `addAnimeSilent`, `removeAnime`, `setDate`. **Supprime les `dispatchEvent`.**
- Composables : `useFirebaseAuth`, `useFirestore`, `usePersistence`, `useJikanApi`, `useSync`, `useRecommendations`, `useRelationMemory`, `useToast`, `useTheme`, `useEpisodeInfo`.
- Le `watch(animeCalendarData, saveToDatabase, { deep:true, debounce })` remplace `store:changed`.
- Le bandeau « data > 1 mois » devient un état réactif `staleDataWarning`.

### Phase 3 — Router + layouts (1 j)
Routes (cf. section D). Navigation guard auth global (remplace le check `localStorage.displayName` de `index.html`). `AppLayout.vue` + `AuthLayout.vue` (coquilles).

### Phase 4 — Composants atomiques (3–4 j)
Du plus simple au plus complexe : `EmptyState`, `SkeletonCard`, `ToastNotification`, `SyncIndicator`, `LoadingOverlay`, `ChipsStrip`, `AnimeCard`, `RecCard`, `WeekAnimeItem`, `WeekSuggestionCard`, `MonthDayCell`, `SeasonNudgeCard`.

### Phase 5 — Pages (5–7 j)
Ordre : `LoginPage` → `AppLayout` complet → `CalendarWeekPage` → `CalendarMonthPage` → `DiscoverExplorePage` → `DiscoverSeasonPage` → `DiscoverComingUpPage` → `LibraryExplorePage` → `LibraryPlanToWatchPage` → `LibraryCompletedPage`.
Infinite scroll via `useIntersectionObserver`. Scroll position via `<KeepAlive>` + `onActivated/onDeactivated`. Swipe via `useSwipe` (@vueuse).

### Phase 6 — Modals & sheets (3–4 j) — LE PLUS RISQUÉ
- `AnimeModal.vue` : état dans un store/composable UI ; `modalContext` en `computed` ; sous-templates `ModalCalendarTop` / `ModalVersionTop` ; strips `ModalRelationsStrip` / `ModalMoreLikeThis` ; fermeture via `onClickOutside`.
- `RecencySheet.vue` + `EpOverridePanel.vue` : bottom-sheets en `<Teleport to="body">` + `v-if`.

### Phase 7 — Branchement final + nettoyage (2–3 j)
1. Un seul entry point `index.html` (supprimer `login.html` séparé).
2. Supprimer `src/store.js`, `src/main.js`, `src/ui/nav.js`, `src/ui/router.js`.
3. Harmoniser/nettoyer les clés localStorage.
4. Test du flow complet : login → load → semaine → modal → ajout → sync → export ICS.
5. Vérifier dark mode + build prod (`vite build`, taille des chunks).

---

## C. Arbre des composants cible

```
App.vue
├── AppLayout.vue                    (layout)
│   ├── LoadingOverlay.vue           (ui)
│   ├── AppHeader.vue                (ui)
│   │   ├── SearchInput.vue          (ui, réutilisable)
│   │   │   └── SuggestionList.vue   (ui, réutilisable)
│   │   ├── SyncIndicator.vue        (ui)
│   │   └── MalImportButton.vue      (ui)
│   ├── PrimaryNav.vue               (ui)
│   ├── SecondaryNav.vue             (ui)
│   │   └── CalendarNavControls.vue  (ui)
│   └── <router-view />
│       ├── CalendarWeekPage.vue     (page)
│       │   ├── WeekDayStrip.vue     (ui)
│       │   ├── WeekDayRow.vue       (ui) ×7
│       │   │   ├── WeekAnimeItem.vue       (ui, réutilisable)
│       │   │   └── WeekSuggestionCard.vue  (ui, réutilisable)
│       │   └── EmptyState.vue       (ui, réutilisable)
│       ├── CalendarMonthPage.vue    (page)
│       │   └── MonthGrid.vue → MonthDayCell.vue (ui, réutilisable)
│       ├── CalendarListPage.vue     (page, stub)
│       ├── DiscoverExplorePage.vue  (page)
│       │   ├── ChipsStrip.vue       (ui, réutilisable)
│       │   ├── SeasonNudgeStrip.vue → SeasonNudgeCard.vue (ui)
│       │   ├── BecauseYouWatched.vue → AnimeCard.vue (ui)
│       │   └── RecSection.vue → RecCard.vue (ui, réutilisable)
│       ├── DiscoverSeasonPage.vue   (page)   → AnimeCard.vue ×N
│       ├── DiscoverComingUpPage.vue (page)   → AnimeCard.vue ×N
│       ├── LibraryExplorePage.vue   (page)   → BecauseYouWatched + RecCard
│       ├── LibraryPlanToWatchPage.vue (page) → AnimeCard.vue ×N
│       └── LibraryCompletedPage.vue (page)   → AnimeCard.vue ×N
│
├── AuthLayout.vue (layout) → LoginPage.vue (page)
│
└── Overlays (Teleport to body)
    ├── AnimeModal.vue
    │   ├── ModalCalendarTop.vue / ModalVersionTop.vue
    │   ├── ModalRelationsStrip.vue / ModalMoreLikeThis.vue
    │   └── EpOverridePanel.vue
    ├── RecencySheet.vue
    └── ToastNotification.vue
```

### Tableau des composants

| Composant | Type | Responsabilité | Réutilisable |
|---|---|---|---|
| `App.vue` | root | Router outlet + providers + host Teleport | Non |
| `AppLayout.vue` | layout | Header + navs + contenu | Non |
| `AuthLayout.vue` | layout | Coquille login centrée | Non |
| `LoginPage.vue` | page | Auth lien email Firebase | Non |
| `CalendarWeekPage.vue` | page | Vue semaine On Air + nav date | Non |
| `CalendarMonthPage.vue` | page | Grille mois + nav date | Non |
| `CalendarListPage.vue` | page | Stub liste | Non |
| `DiscoverExplorePage.vue` | page | Recs perso + chips + BYW + infinite scroll | Non |
| `DiscoverSeasonPage.vue` | page | Catalogue saison | Non |
| `DiscoverComingUpPage.vue` | page | Radar utilisateur | Non |
| `LibraryExplorePage.vue` | page | Recs « missed » + BYW | Non |
| `LibraryPlanToWatchPage.vue` | page | Watchlist triée | Non |
| `LibraryCompletedPage.vue` | page | Vault trié | Non |
| `AppHeader.vue` | ui | Recherche + boutons (dark, export, recs, MAL) | Non |
| `SearchInput.vue` | ui | Input recherche débounced | Oui |
| `SuggestionList.vue` | ui | Dropdown résultats | Oui |
| `PrimaryNav.vue` | ui | Onglets On Air / Discover / Library | Non |
| `SecondaryNav.vue` | ui | Sous-onglets contextuels | Non |
| `CalendarNavControls.vue` | ui | Prev / Today / Next + libellé période | Non |
| `SyncIndicator.vue` | ui | Spinner de synchro | Oui |
| `MalImportButton.vue` | ui | Import fichier XML MAL | Non |
| `LoadingOverlay.vue` | ui | Spinner plein écran au boot | Oui |
| `AnimeCard.vue` | ui | Carte poster générique | **Oui** |
| `RecCard.vue` | ui | Carte reco + Add/Skip + panneau « why » | **Oui** |
| `WeekDayStrip.vue` | ui | Bande de navigation des jours | Non |
| `WeekDayRow.vue` | ui | Section jour (header + stack) | Non |
| `WeekAnimeItem.vue` | ui | Carte-ligne anime (thumb + titre + heure + chip ep) | Oui |
| `WeekSuggestionCard.vue` | ui | Carte suggestion jour vide | Oui |
| `MonthGrid.vue` | ui | Conteneur grille 42 cellules | Non |
| `MonthDayCell.vue` | ui | Cellule jour + points anime | Oui |
| `AnimeModal.vue` | ui | Modal détail, routage par contexte | Non |
| `ModalCalendarTop.vue` | ui | En-tête modal contexte calendar | Non |
| `ModalVersionTop.vue` | ui | En-tête modal autres contextes | Non |
| `ModalRelationsStrip.vue` | ui | Scroll horizontal séquences & related | Oui |
| `ModalMoreLikeThis.vue` | ui | Scroll horizontal similaires (genres) | Oui |
| `EpOverridePanel.vue` | ui | Panneau override numéro d'épisode | Non |
| `RecencySheet.vue` | ui | Bottom-sheet « quand l'as-tu vu ? » | Non |
| `ToastNotification.vue` | ui | Toast transient (Teleport) | Oui |
| `SeasonNudgeStrip.vue` | ui | Section « Keep Watching » (séquelles) | Non |
| `SeasonNudgeCard.vue` | ui | Carte nudge séquelle + dismiss | Oui |
| `ChipsStrip.vue` | ui | Chips de presets (actif/inactif) | Oui |
| `BecauseYouWatched.vue` | ui | Section scroll « Because you watched » | Non |
| `EmptyState.vue` | ui | État vide générique | Oui |
| `SkeletonCard.vue` | ui | Skeleton de chargement | Oui |

---

## D. Carte des services / composables cible

### Store Pinia
| Fichier | Responsabilité |
|---|---|
| `stores/anime.ts` | `animeCalendarData[]`, `currentDate`, `currentView` + actions. Remplace `AppStore` + bus DOM. |
| `stores/ui.ts` (optionnel) | état des overlays (modal ouvert, sheet ouverte, toast). |

### Composables
| Fichier | Endpoints couverts | Méthodes | Auth |
|---|---|---|---|
| `useJikanApi.ts` | `/anime?q=`, `/anime/{id}`, `/anime/{id}/relations`, `/anime/{id}/recommendations`, `/seasons/now`, `/seasons/upcoming`, top finished | `searchAnime`, `fetchAnimeById`, `fetchAnimeRelations`, `fetchAnimeRecommendations`, `fetchCurrentSeason`, `fetchUpcomingSeason`, `fetchTopFinishedAnime` | Non |
| `useFirebaseAuth.ts` | Firebase Auth (email link) | `sendSignInLink`, `completeSignIn`, `onAuthStateChanged` | initie l'auth |
| `useFirestore.ts` | `schedules/{uid}` | `loadSchedule`, `saveSchedule` | **Oui** (UID) |
| `usePersistence.ts` | Firestore + fallback localStorage | `loadFromDatabase`, `saveToDatabase` | dégrade sans auth |
| `useSync.ts` | `/anime/{id}` (batch throttlé) | `syncAnimeUpdates`, `startBackgroundRelationFetch` | Non |
| `useRecommendations.ts` | pools Jikan (cache) | `fetchRecPool`, `getNextBatch`, `reScorePool`, `getBecauseYouWatchedBatch`, `getSlotFillSuggestions` | Non |
| `useRelationMemory.ts` | IndexedDB | `idbGet`, `idbSet`, `buildRelationMemory` | Non |
| `useEpisodeInfo.ts` | — (pur) | `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus` | Non |
| `useICS.ts` | — | `generateICSFile` | Non |
| `useMalImport.ts` | — (parse fichier) | `parseMalXml`, `importMalFile` | Non |
| `useToast.ts` | — | `showToast` | Non |
| `useTheme.ts` | — | `toggleDarkMode`, `isDark` | Non |
| `useSwipe.ts` | — (ou @vueuse) | navigation tactile | Non |
| `useSeasonNudges.ts` | IndexedDB | `getSeasonNudges` | Non |

### Utils purs (`src/utils/`)
| Fichier | Exports |
|---|---|
| `utils/jst.ts` | `parseJSTToLocal(day, time)` |
| `utils/normalize.ts` | `normalizeAnime(raw)` |
| `utils/recEngine.ts` | `RelationMemory`, `buildTasteProfile`, `scorePool`, `assignBadge`, `buildNextBatch`, `extractBecauseYouWatched`, `applyPreset` |
| `utils/idb.ts` | `idbGet`, `idbSet` |
| `utils/helpers.ts` | `escapeHTML` (si encore utile), `getWeekNumber`, `fetchWithRetry` |
| `utils/ics.ts` | génération ICS |

### Router (`src/router/index.ts`)
| Path | Layout + Page | Guard |
|---|---|---|
| `/login` | AuthLayout + LoginPage | guest |
| `/` | AppLayout → redirect `/week` | auth |
| `/week` | CalendarWeekPage | auth |
| `/month` | CalendarMonthPage | auth |
| `/list` | CalendarListPage | auth |
| `/discover` | DiscoverExplorePage | auth |
| `/discover/season` | DiscoverSeasonPage | auth |
| `/discover/coming-up` | DiscoverComingUpPage | auth |
| `/library` | LibraryExplorePage | auth |
| `/library/plan` | LibraryPlanToWatchPage | auth |
| `/library/completed` | LibraryCompletedPage | auth |
