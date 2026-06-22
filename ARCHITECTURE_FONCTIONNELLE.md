# ARCHITECTURE_FONCTIONNELLE.md — Vue fonctionnelle du système Aanime

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** décrire ce que l'app **fait** du point de vue utilisateur, puis tracer
> le **pont** entre chaque fonctionnalité et les briques techniques qui la portent.
> Se lit avec `ARCHITECTURE_TECHNIQUE.md` (le « comment c'est construit »).
>
> **État de référence :** session 16 (dual audit). EPIC-1 + EPIC P0 + EPIC-2 + EPIC-3 clos.

---

## 1. Ce que fait l'application

**Aanime** est un tracker de calendrier d'animes. L'utilisateur :
- **suit** les animes en cours de diffusion sur un calendrier (semaine / mois) ;
- **découvre** de nouvelles séries via un moteur de recommandations personnalisé ;
- **gère sa bibliothèque** : terminés (vault), à voir (watchlist), à venir (radar) ;
- **exporte** son planning au format `.ics` ;
- **importe** sa liste depuis MyAnimeList (XML) ;
- voit ses données **synchronisées** dans le cloud (Firebase) et en local.

---

## 2. Le modèle métier : les 4 « states »

Chaque anime suivi porte un `state` qui détermine où il apparaît dans l'UI.

| State | Sens utilisateur | Onglet d'affichage |
|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air (Week / Month) |
| `radar` | À venir, pas encore diffusé | Discover → Coming Soon |
| `watchlist` | Prévu de regarder | Library → Plan to Watch |
| `vault` | Terminé / archivé | Library → Completed |

### Transitions automatiques (reproduites du vanilla)
```
        ┌──────────┐  diffusion commencée (début ≤ aujourd'hui − 7j)   ┌──────────┐
        │  radar   │ ─────────────────────────────────────────────────▶│ calendar │
        └──────────┘                                                    └────┬─────┘
                                                                             │
                                       diffusion terminée (Finished Airing)  │ (sens unique)
                                                                             ▼
                                                                        ┌──────────┐
                                                                        │  vault   │
                                                                        └──────────┘
   watchlist ──(l'utilisateur commence à regarder)──▶ calendar
```
- `radar → calendar` : géré dans `usePersistence` (au load) + `useSync` (à la synchro).
- `* → vault` : auto-vault dès statut `Finished Airing` (sens unique).
- **Hiatus** : un anime `calendar` en `Currently Airing` sans diffusion depuis > 14 j est
  signalé « Hiatus » à l'affichage (calcul dérivé `isOnHiatus`, **source unique**, US-107).

> ⚠️ **Bug d'affichage audité s16 (US-154, P1)** : un show au statut legacy `'Continuing'`
> (en cours de diffusion, sans horaire) s'affiche par erreur pastille verte « Finished » —
> `getCardStatus` ne mappe pas cette valeur. *Impact user : on croit la série terminée.*

---

## 3. Parcours utilisateur principaux

### P1 — Connexion (lien email, sans mot de passe)
1. L'utilisateur saisit son email sur `/login`.
2. Firebase envoie un lien de connexion → l'utilisateur clique → retour sur l'app.
3. La session s'ouvre, le boot charge ses données.

### P2 — Boot & chargement des données
1. `LoadingOverlay` plein écran pendant l'initialisation.
2. Chargement Firestore → store → synchro Jikan → reconstruction du graphe de recos → re-score.
3. L'overlay se lève, le calendrier de la semaine s'affiche.

### P3 — Suivre un anime
1. Recherche dans le header → suggestions.
2. Clic → l'anime est ajouté ; son `state` est calculé automatiquement selon son statut
   de diffusion (en cours → `calendar`, à venir → `radar`, terminé → `vault`).
3. La synchro de fond récupère broadcast/épisodes/score et met à jour le calendrier.

### P4 — Consulter le calendrier (Week / Month)
- **Week** : colonnes par jour, cartes-lignes d'animes avec heure locale (JST→local) et
  numéro d'épisode calculé ; jours vides remplis par des suggestions.
- **Month** : grille 42 cellules, points d'anime par jour.
- Navigation date unifiée (Prev / période cliquable = Today / Next) via une seule barre.

### P5 — Découvrir (Discover)
- **For You** : recommandations personnalisées scorées, chips de presets (Hidden gems,
  Trending…), section « Because you watched », infinite scroll, nudges de séquelles.
- **This Season** : catalogue de la saison en cours.
- **Coming Soon** : le radar de l'utilisateur (à venir).

### P6 — Gérer sa bibliothèque (Library)
- **Upcoming** : recos « you might have missed ».
- **Plan to Watch** : watchlist triée.
- **Completed** : vault trié par date de complétion.

### P7 — Détail d'un anime (modal)
- Ouverture d'une modal contextuelle (calendar / version / library-rec) avec relations
  (séquelles), « more like this » (genres), override d'épisode, sheet « quand l'as-tu vu ? ».

### P8 — Exporter / Importer
- **Export** : génération d'un fichier `.ics` du planning.
- **Import MAL** : upload d'un XML MyAnimeList → ajout silencieux des séries.

---

## 4. LE PONT — fonctionnalité ▶ technique

Tableau central de ce document : pour chaque fonctionnalité, les briques qui la portent.

| Fonctionnalité (utilisateur) | Page(s) | Composables / Stores | Utils purs |
|---|---|---|---|
| Connexion email-link | `LoginPage` | `useFirebaseAuth`, `lib/firebase` | — |
| Boot & chargement | `App.vue` | `usePersistence`, `useSync`, `useRecommendations` | — |
| Recherche & ajout | `AppHeader` / `SearchInput` | `useJikanApi.searchAnime`, `stores/anime.addAnime` | `normalize` |
| Calendrier semaine | `CalendarWeekPage` | `stores/anime`, `useEpisodeInfo`, `useRecommendations.getSlotFillSuggestions` | `jst`, `episodeInfo` |
| Calendrier mois | `CalendarMonthPage` | `stores/anime`, `useEpisodeInfo` | `episodeInfo` |
| Navigation date | `CalendarNavControls` (dans `SecondaryNav`) | `stores/anime.setDate` | — |
| Découvrir / For You | `DiscoverExplorePage` | `useRecommendations` (pools, BYW, nudges, presets) | `recEngine` |
| This Season | `DiscoverSeasonPage` | `useJikanApi.fetchCurrentSeason`, `stores/anime` | `normalize` |
| Coming Soon (radar) | `DiscoverComingUpPage` | `stores/anime` (filtre radar) | — |
| Library Upcoming | `LibraryExplorePage` | `useRecommendations` (context library) | `recEngine` |
| Plan to Watch | `LibraryPlanToWatchPage` | `stores/anime` (filtre watchlist) | — |
| Completed | `LibraryCompletedPage` | `stores/anime` (filtre vault) | — |
| Détail anime (modal) | overlay `AnimeModal` | `stores/ui`, `useJikanApi` (relations/recos), `useRecommendations` | `recEngine` |
| Override d'épisode | `EpOverridePanel` | `stores/ui`, `stores/anime` | — |
| Recency (« quand vu ? ») | `RecencySheet` | `stores/ui`, `useRecommendations` | — |
| Synchro de fond | (App.vue) | `useSync` (batch + worker throttlé) | `jst`, `helpers` |
| Indicateur de synchro | `SyncIndicator` | `useSync.isSyncing` | — |
| Export ICS | `AppHeader` | `useICS.downloadICS` | `ics` |
| Import MAL | `MalImportButton` / `AppHeader` | `useMalImport`, `stores/anime.addAnimeSilent` | `malImport` |
| Thème sombre | `AppHeader` (toggle) | `useTheme` (`useDark` → `html.dark`) | — |
| Notifications | overlay `ToastNotification` | `useToast` | — |
| Persistance auto | (transparent) | `usePersistence` (`watchDebounced`) → `useFirestore` | — |

---

## 5. Cycle de vie d'un anime (de la découverte au vault)

```
DÉCOUVERTE                SUIVI                       ARCHIVAGE
──────────                ─────                       ─────────
Discover/Search           Calendar (On Air)           Completed (vault)
   │                         │                            ▲
   │ ajout                   │ diffusion en cours         │ Finished Airing
   ▼                         │ épisodes calculés          │ (auto-vault, sens unique)
[radar] ──diffusion────▶ [calendar] ──fin de diffusion──┘
   ou                       │
[watchlist] ──je regarde──▶ │
```

À chaque étape, la **synchro Jikan** (au boot et périodiquement) met à jour le statut,
les épisodes, le broadcast et déclenche les transitions de state automatiques.

---

## 6. Fraîcheur & synchronisation des données

| Donnée | Source | Fraîcheur |
|---|---|---|
| Liste de l'utilisateur | Firestore `schedules/{uid}` + cache local `aanime_calendar` | temps réel au load, sauvegarde debouncée (1s) à chaque modif |
| Statut / épisodes / broadcast | Jikan `/anime/{id}` | synchro batch au boot, TTL 6 h (`aanime_sync_ts`) |
| Relations (séquelles) | Jikan `/anime/{id}/relations` | cache IDB persistant (⚠️ sans expiration — dette audit s16) |
| Recommandations liées | Jikan `/anime/{id}/recommendations` | cache IDB TTL 7 j |
| Saison en cours / à venir | Jikan `/seasons/*` | cache localStorage TTL 24 h (`aanime_seasons_now`, `aanime_seasons_upcoming`) |

> **Clés localStorage :** toutes préfixées `aanime_` depuis US-133, avec migration
> transparente des anciennes clés au boot (registre complet dans `ARCHITECTURE_TECHNIQUE.md §7`).

**Bandeau « données anciennes » :** si les données chargées datent de plus d'un mois,
`usePersistence` expose `staleDataWarning` (état réactif) — affiché dans l'UI, et non plus
injecté dans le `<body>` comme le faisait le vanilla.

---

## 7. Lecture croisée avec la vue technique

- Chaque **parcours** (section 3) correspond à une **séquence d'appels** documentée dans
  `ARCHITECTURE_TECHNIQUE.md §4-5`.
- Chaque **transition de state** (section 2) est implémentée dans `stores/anime.ts`
  (upsert) + `usePersistence` (load) + `useSync` (synchro).
- Le **pont** (section 4) est la table de correspondance à consulter avant de toucher une
  fonctionnalité : elle dit quels fichiers techniques sont impliqués, donc quel est le
  périmètre réel d'une User Story.
