# AUDIT.md — Audit technique de l'existant (vanilla JS)

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat. Décrit l'état actuel à reproduire.

---

## 1. Stack technique détaillée

| Couche | Détail |
|---|---|
| Langage | Vanilla JavaScript (ES Modules), aucun TypeScript dans les sources |
| Build | Vite 6 + `@vitejs/plugin-react` (React = dépendance morte à supprimer) |
| Auth | Firebase v12 — Email Link (passwordless) |
| Base de données | Firestore — un document unique `schedules/{uid}` contenant `{ timestamp, data }` |
| API externe | Jikan MAL API v4 (`https://api.jikan.moe/v4`) — publique, sans clé, rate-limit ~3 req/s |
| Cache persistant | IndexedDB (base `AnimeCaches`, v2) — stores `relations` & `recommendations` |
| Cache session | `localStorage` (rec pools, scroll, sync timestamps, préférences) + `sessionStorage` (blacklist, compteurs) |
| Routing | Routeur maison : string `AppStore.currentView` + show/hide de `<div>` |
| State | Objet global mutable `AppStore` → notifie via `CustomEvent` DOM |
| Styles | `style.css` (variables CSS, dark mode par classe sur `<body>`) |
| Entrées Vite | 2 entry points : `index.html` (app) + `login.html` (auth) |

---

## 2. Inventaire des écrans / vues (9 vues + login)

| Clé de vue | Onglet → Sous-onglet | Description | Fichier source |
|---|---|---|---|
| `calendar:week` | On Air → Week | Colonnes de jours, cartes-lignes d'animes, suggestions de remplissage | `src/views/week.js` |
| `calendar:month` | On Air → Month | Grille calendrier 42 cellules | `src/views/month.js` |
| `calendar:list` | On Air → List | Stub (« coming soon ») | `src/ui/router.js` |
| `incoming:explore` | Discover → For You | Recommandations scorées + chips de filtre + infinite scroll | `src/views/incoming.js` |
| `incoming:season` | Discover → This Season | Catalogue complet de la saison en cours | `src/views/season.js` |
| `incoming:comingup` | Discover → Coming Soon | Liste « radar » de l'utilisateur (state = radar) | `src/views/incoming.js` |
| `library:explore` | Library → Upcoming | Recommandations « You might have missed » | `src/views/library.js` |
| `library:plantowatch` | Library → Plan to Watch | Watchlist triée par date de début | `src/views/library.js` |
| `library:completed` | Library → Completed | Vault trié par `completedAt` | `src/views/library.js` |
| `login` | (page séparée) | Auth par lien email | `login.html` + `src/login.js` |

---

## 3. Bus d'événements DOM actuel (à remplacer par Pinia)

| Événement (`CustomEvent`) | Émetteur → Consommateur |
|---|---|
| `store:changed` | `AppStore.addAnime/removeAnime` → `saveToDatabase`, `reScorePool`, `updateCurrentView` |
| `ui:refresh` | `loadFromDatabase`, `recency-sheet` → `updateCurrentView` |
| `anime:add` | `week.createSuggestionCard` → `actions.addAnimeToCalendar` |
| `anime:added` | `actions.addAnimeToCalendar` → `api.syncAnimeUpdates` |
| `recs:heart` | bouton Add de `rec-strip` → `incoming.handleRecHeart` |
| `recs:remove` | bouton Skip / dismiss modal → `incoming.handleRecRemove` |
| `library-recs:heart` | bouton Add de `rec-strip` → `library.handleLibraryRecHeart` |
| `library-recs:remove` | bouton Skip / dismiss → `library.handleLibraryRecRemove` |
| `recs:relations:rebuild` | `recs.trackNegative` → `api.startBackgroundRelationFetch` |
| `nav:next` / `nav:prev` | `src/ui/swipe.js` → `main.js` (handleNext/handlePrev) |

> **Implication migration :** chaque flux ci-dessus devient soit une **action de store Pinia**, soit un **`watch()`**, soit un **`emit`** de composant. Aucun `CustomEvent` ne survit.

---

## 4. Carte des appels API (Jikan v4)

| Endpoint | Méthode | Usage | Auth |
|---|---|---|---|
| `/anime?q={query}&sfw=true&limit=25&order_by=popularity&sort=asc` | GET | Recherche (autocomplétion) | Non |
| `/anime/{id}` | GET | Détail + synchro (statut, épisodes, broadcast, score) | Non |
| `/anime/{id}/relations` | GET | Séquences/préquelles/relations (cache IndexedDB persistant) | Non |
| `/anime/{id}/recommendations` | GET | Recommandations liées (cache IndexedDB, TTL 7 j) | Non |
| `/seasons/now?limit=25&page={1..3}` | GET | Saison en cours (jusqu'à 75 shows, throttle 1 s entre pages) | Non |
| `/seasons/upcoming?limit=25&page=1` | GET | Saison à venir / backlog recs (cache localStorage, TTL 24 h) | Non |
| `/anime?min_score=7.5&order_by=members&sort=desc&limit=25` | GET | Top finis (pool de recs Library) | Non |

**Contraintes réseau à reproduire :**
- `fetchWithRetry` : 3 tentatives, backoff exponentiel, gestion explicite du **429** (rate limit).
- Sync par **batch de 2**, intervalle **2 s** entre batches.
- Background worker relations : throttle **1,1 s** par requête.
- Spinner global déclenché en interceptant tout `fetch` vers `api.jikan.moe`.

**Firestore :**
| Opération | Chemin | Auth |
|---|---|---|
| `getDoc` | `schedules/{uid}` | Oui (UID Firebase) |
| `setDoc` | `schedules/{uid}` | Oui (UID Firebase) |

**Firebase Auth :** `sendSignInLinkToEmail`, `signInWithEmailLink`, `isSignInWithEmailLink`, `authStateReady`.

---

## 5. Logique métier & utilitaires partagés

| Fichier | Rôle | Nature |
|---|---|---|
| `src/store.js` | État global `AppStore` (data, date, vue) + add/remove | À remplacer par Pinia |
| `src/normalize.js` | `normalizeAnime(raw)` — forme canonique d'un anime | **Pur** → portable tel quel |
| `src/utils.js` | `parseJSTToLocal`, `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus`, `fetchWithRetry`, `escapeHTML`, `getWeekNumber` | Pur (sauf fetch) → portable |
| `src/rec-engine.js` | Moteur de scoring : `buildTasteProfile`, `scorePool`, `assignBadge`, `RelationMemory`, `extractBecauseYouWatched`, `buildNextBatch`, `applyPreset` | **Pur** → portable tel quel |
| `src/recs.js` | Orchestration des recs (pools, batches, BYW, slot-fill) | Devient `useRecommendations` |
| `src/api.js` | Appels Jikan + sync + background worker | Devient `useJikanApi` + `useSync` |
| `src/persistence.js` | Charge/sauvegarde Firestore + localStorage | Devient `usePersistence` |
| `src/idb.js` | Wrapper IndexedDB | Devient `utils/idb.ts` |
| `src/ics.js` | Génération fichier `.ics` | Devient `useICS` / `utils/ics.ts` |
| `src/firebase.js` | Init Firebase + gestion erreurs Firestore | Devient `useFirebaseAuth` + `useFirestore` |
| `src/ui/*.js` | toast, nav, router, search, swipe, suggestions, etc. | Deviennent composables + composants |

---

## 6. Risques identifiés

### Risque ÉLEVÉ
- **`src/views/modal.js` (465 lignes)** : DOM impératif, IIFE async dans des handlers, closures imbriquées, 5 templates selon `modalContext`. Composant le plus délicat → à migrer **en dernier**.
- **`src/views/incoming.js` (580 lignes)** : fonctions de rendu inline, handlers de scroll stockés sur `window`, IntersectionObserver géré à la main.
- **`src/persistence.js`** : insère un bandeau DOM directement (`document.body.appendChild`) → couche données qui fuit dans l'UI. À remplacer par un état réactif `staleDataWarning`.
- **Handlers globaux sur `window`** (`window.incomingScrollHandler`, `window.libraryScrollHandler`) : remplacés à chaque navigation, pas de cleanup fiable.

### Risque MOYEN
- Tous les `store:changed → saveToDatabase()` sont fire-and-forget : il faut reproduire l'équivalent en `watch` Pinia (avec debounce).
- **`getAnimeEpisodeInfo`** : logique de calcul d'épisode à branches multiples, **sans tests** → à porter à l'identique et à couvrir par des tests.
- **Rate-limiting Jikan** (429, batch, throttle 1,1 s) → à répliquer fidèlement dans la couche service.
- **`parseJSTToLocal`** : conversion JST→local subtile (date de référence fixe en 1970) → composable/util dédié, même approche exacte.

### Dette à résoudre pendant la migration
- Supprimer `@vitejs/plugin-react` et la dépendance React.
- Remplacer les `escapeHTML` manuels par l'échappement natif des templates Vue.
- Remplacer le toggle `dark-mode` sur `<body>` par un composable `useTheme`.
- Clés localStorage incohérentes (`backlog_recs_v1`, `recs_incoming_v3`, `recs_library_v2`) → harmoniser.
- `login.html` (entry point séparé) → route `/login` gardée par un navigation guard Vue Router.
