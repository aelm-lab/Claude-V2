# ARCHITECTURE_FONCTIONNELLE.md — Vue fonctionnelle du système Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** décrire ce que l'app **fait** du point de vue utilisateur, puis tracer le **pont**
> entre chaque fonctionnalité et les briques techniques qui la portent.
> **Pendant technique :** `ARCHITECTURE_TECHNIQUE.md` — les deux se lisent ensemble.
>
> **Aucun état d'avancement ici** (livré / à faire / bug ouvert) → `EPICS.md` et `STATE.md`.
> Ce document décrit le **comportement attendu**, pas son état de santé.

---

## 1. Ce que fait l'application

L'utilisateur :
- **suit** les animes en cours de diffusion sur un calendrier (semaine / mois) ;
- **découvre** de nouvelles séries via un moteur de recommandations personnalisé ;
- **gère sa bibliothèque** : terminés, à voir, à venir ;
- **exporte** son planning au format `.ics` ;
- **importe** sa liste depuis MyAnimeList (fichier XML) ;
- voit ses données **synchronisées** entre le cloud (Firebase) et le local ;
- peut se **déconnecter**, avec purge des clés `aanime_*` et reset du store.

---

## 2. Le modèle métier : les 4 `state`

Chaque anime suivi porte un `state` qui détermine **où il apparaît** dans l'interface.

| State | Sens utilisateur | Onglet d'affichage | Libellé visible |
|---|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air (Week / Month) | — |
| `radar` | À venir, pas encore diffusé | Discover | **« Coming Soon »** |
| `watchlist` | Prévu de regarder | Library | **« Plan to Watch »** |
| `vault` | Terminé / archivé | Library | **« Completed »** |

> 🔴 **`radar` et `vault` sont du jargon interne.** L'utilisateur ne les voit nulle part.
> Tout toast, libellé ou message nomme **l'onglet réel** : « Coming Soon », « Completed ».

### Transitions automatiques (reproduites du vanilla)

```
        ┌──────────┐  diffusion commencée (début ≤ aujourd'hui − 7 j)   ┌──────────┐
        │  radar   │ ──────────────────────────────────────────────────▶│ calendar │
        └──────────┘                                                     └────┬─────┘
                                                                              │
                                        diffusion terminée (Finished Airing)  │ (sens unique)
                                                                              ▼
                                                                         ┌──────────┐
                                                                         │  vault   │
                                                                         └──────────┘
   watchlist ──(l'utilisateur commence à regarder)──▶ calendar
```

- `radar → calendar` : géré dans `usePersistence` (au chargement) + `useSync` (à la synchro).
- `* → vault` : **auto-vault** dès le statut `Finished Airing`, **sens unique**. Un toast
  « Moved to Completed » signale le déplacement au boot.
- **Hiatus** : un anime `calendar` en `Currently Airing` sans diffusion depuis plus de **14 j**
  est signalé « Hiatus » à l'affichage — calcul dérivé `isOnHiatus`, **source unique**.

> ⚠️ **Conséquence pour les tests :** un anime `calendar` + `Finished Airing` s'auto-vault au
> boot et **disparaît** de la vue semaine. Pour tester une action sur un anime terminé, passer
> par `watchlist` (exclu de l'auto-vault).

---

## 3. Parcours utilisateur

### P0 — Première visite (onboarding)
1. Un nouvel utilisateur atterrit sur `/welcome` → `OnboardingPage.vue`.
2. **Choix de 3 genres.**
3. **8 suggestions** scorées sur ces genres, ajout en 1 tap.
4. **Atterrissage sur un calendrier pré-rempli** : chaque item coché est ajouté au store, la
   sauvegarde part, puis redirection vers `/week`.

> C'est le parcours qui porte la North Star (« suivre son premier anime en moins de 2 min »).
> Règle qui en découle : **un message de confirmation doit porter sur ce que l'utilisateur va
> voir, pas sur ce que le code vient d'exécuter.** Un toast « N shows added » qui n'est
> conditionné à aucune vérification d'affichage certifie un succès que l'écran suivant peut
> démentir.

### P1 — Connexion & déconnexion (lien email, sans mot de passe)
1. L'utilisateur saisit son email sur `/login` (input in-app, pas de `window.prompt`).
2. Firebase envoie un lien de connexion → clic → retour sur l'app.
3. La session s'ouvre, le boot charge ses données. **Redirect post-login = `/`**, pas la
   route d'origine (choix assumé : le lien magique expire rarement, ROI faible).
4. **Déconnexion** : bouton dans le header → modale de confirmation → `signOut()` + purge des
   clés `aanime_*` + `clearAll()` du store + redirection `/login`.

### P2 — Boot & chargement des données
1. Loader statique pendant que le bundle se parse, puis `LoadingOverlay` pendant l'initialisation.
2. Paint immédiat depuis le cache local, réconciliation Firestore en arrière-plan.
3. Synchro Jikan → reconstruction du graphe de recommandations → re-score.
4. L'overlay se lève, le calendrier de la semaine s'affiche.

### P3 — Suivre un anime
1. Recherche dans le header → suggestions enrichies (statut, nombre d'épisodes, année,
   studio, score, dual-titre).
2. Clic → l'anime est ajouté ; son `state` est calculé **automatiquement** selon son statut
   de diffusion (en cours → `calendar`, à venir → `radar`, terminé → `vault`).
3. Un **toast nomme la destination visible exacte**.
4. Un état « ✓ Added » est cliquable : il **retire** l'anime d'où qu'il soit, avec un toast
   « Removed ».
5. La synchro de fond récupère broadcast / épisodes / score et met à jour le calendrier.

### P4 — Consulter le calendrier (Week / Month / List)
- **Week** : colonnes par jour, cartes-lignes avec heure locale (conversion JST → local) et
  numéro d'épisode calculé. Snap sur aujourd'hui à l'ouverture. Barre de progression fine
  sous la carte. Bouton ✓ « marquer vu » en 1 tap — **masqué** sur un anime en cours de
  diffusion ou en hiatus (il n'a de sens que sur un titre terminé ; l'action reste
  disponible via la modale).
- **Jours vides** : suggestion de remplissage + possibilité de l'écarter **pour la session
  seulement** (jamais persisté : « écarter pour l'instant » ≠ « bannir »), + CTA sur jour vide.
- **Month** : grille de 42 cellules, points d'anime par jour.
- **Navigation de date unifiée** : Prev / période cliquable (= Today) / Next, via une seule
  barre. Le libellé de période a une **source unique**.

### P5 — Découvrir (Discover)
- **For You** : recommandations personnalisées scorées, chips de presets (Hidden gems,
  Trending…), section « Because you watched X », scroll infini, nudges de séquelles.
- **This Season** : catalogue de la saison en cours.
- **Coming Soon** : le radar de l'utilisateur.

### P6 — Gérer sa bibliothèque (Library)
- **Upcoming** : recommandations « you might have missed ».
- **Plan to Watch** : watchlist triée.
- **Completed** : vault trié par date de complétion.

### P7 — Détail d'un anime (modale)
Modale contextuelle (`calendar` / `version` / `library-rec`) avec relations (séquelles),
covers enrichies, « more like this » par genres, override du numéro d'épisode, et sheet
« quand l'as-tu vu ? » pour le scoring de recency.

### P8 — Statistiques
Page `/stats` (derrière le guard auth) : « Mon année anime ». Les **top genres sont calculés
sur le contenu terminé cette année uniquement** — un year-in-review compte ce qui a été
*consommé*, pas les intentions. Benchmark : AniList, Spotify Wrapped, Letterboxd.

### P9 — Exporter / Importer
- **Export** : génération d'un fichier `.ics` du planning.
- **Import MAL** : upload d'un XML MyAnimeList → ajout silencieux des séries. Les titres
  marqués `Dropped` **ne sont pas importés** (évite de polluer la bibliothèque).

---

## 4. LE PONT — fonctionnalité ▶ technique

**Tableau central de ce document.** À consulter **avant** de toucher une fonctionnalité : il
dit quels fichiers sont impliqués, donc quel est le périmètre réel d'une User Story.

| Fonctionnalité | Page(s) / Composant | Composables / Stores | Utils purs |
|---|---|---|---|
| Onboarding 1ʳᵉ visite | `OnboardingPage` | `stores/anime.addAnime`, `usePersistence.saveToDatabase` | `onboardingFilter` |
| Connexion email-link | `LoginPage` | `useFirebaseAuth`, `lib/firebase` | — |
| Déconnexion | `AppHeader` / `LogoutConfirmModal` | `useFirebaseAuth.signOut`, `stores/anime.clearAll` | — |
| Boot & chargement | `App.vue` | `usePersistence`, `useSync`, `useRecommendations` | — |
| Recherche & ajout | `AppHeader` / `SearchInput` | `useJikanApi.searchAnime`, `stores/anime.addAnime` | `normalize` |
| Calendrier semaine | `CalendarWeekPage` | `stores/anime`, `useEpisodeInfo`, `useRecommendations.getSlotFillSuggestions` | `jst`, `episodeInfo` |
| Calendrier mois | `CalendarMonthPage` | `stores/anime`, `useEpisodeInfo` | `episodeInfo` |
| Navigation de date | `CalendarNavControls` (dans `SecondaryNav`) | `stores/anime.setDate` | — |
| Découvrir / For You | `DiscoverExplorePage` | `useRecommendations` (pools, BYW, nudges, presets) | `recEngine` |
| This Season | `DiscoverSeasonPage` | `useJikanApi.fetchCurrentSeason`, `stores/anime` | `normalize` |
| Coming Soon | `DiscoverComingUpPage` | `stores/anime` (filtre `radar`) | — |
| Library Upcoming | `LibraryExplorePage` | `useRecommendations` (contexte `library`) | `recEngine` |
| Plan to Watch | `LibraryPlanToWatchPage` | `stores/anime` (filtre `watchlist`) | — |
| Completed | `LibraryCompletedPage` | `stores/anime` (filtre `vault`) | — |
| Statistiques | `StatsPage` | `useStats` | — |
| Détail anime | overlay `AnimeModal` | `stores/ui`, `useJikanApi`, `useRecommendations` | `recEngine` |
| Override d'épisode | `EpOverridePanel` | `stores/ui`, `stores/anime` | — |
| Recency (« quand vu ? ») | `RecencySheet` | `stores/ui`, `useRecommendations` | — |
| Synchro de fond | (`App.vue`) | `useSync` (batch + worker throttlé) | `jst`, `helpers` |
| Indicateur de synchro | `SyncIndicator` | `useSync.isSyncing` | — |
| Export ICS | `AppHeader` | `useICS.downloadICS` | `ics` |
| Import MAL | `MalImportButton` / `AppHeader` | `useMalImport`, `stores/anime.addAnimeSilent` | `malImport` |
| Thème sombre | `AppHeader` (toggle) | `useTheme` (`useDark` → `html.dark`) | — |
| Notifications | overlay `ToastNotification` | `useToast` | — |
| Persistance automatique | (transparent) | `usePersistence` (`watchDebounced`) → `useFirestore` | — |

---

## 5. Fraîcheur & synchronisation des données

| Donnée | Source | Fraîcheur |
|---|---|---|
| Liste de l'utilisateur | Firestore `schedules/{uid}` + cache local `aanime_calendar` | Chargement au boot, sauvegarde debouncée (1 s) à chaque modification |
| Statut / épisodes / broadcast | Jikan `/anime/{id}` | Synchro batch au boot, TTL 6 h |
| Relations (séquelles) | Jikan `/anime/{id}/relations` | Cache IndexedDB persistant, **sans expiration** |
| Recommandations liées | Jikan `/anime/{id}/recommendations` | Cache IndexedDB, TTL 7 j |
| Saison en cours / à venir | Jikan `/seasons/*` | Cache localStorage, TTL 24 h. **Le cache périmé est servi si le fetch échoue** — comportement délibéré, hérité du vanilla, qui évite une page vide |

**Deux avertissements de fraîcheur, à ne pas confondre :**
- **`staleDataWarning`** — les données de l'utilisateur datent de plus d'un mois.
  État réactif exposé par `usePersistence`, affiché dans l'UI (le vanilla injectait un
  bandeau dans le `<body>` : plus jamais).
- **Cache de saison périmé** — servi silencieusement. `error.value` est renseigné mais
  **jamais affiché**. Sans cache et sans réseau, la liste est vide **sans explication**.
  C'est une dette UX assumée et suivie, pas un bug.

> L'état de disponibilité réel de Jikan vit dans `STATE.md §Faits externes` et **se remesure
> à chaque ouverture de session** — il ne se déduit jamais de ce document.

---

## 6. Lecture croisée avec la vue technique

- Chaque **parcours** (§3) correspond à une séquence d'appels documentée dans
  `ARCHITECTURE_TECHNIQUE.md §5-6`.
- Chaque **transition de state** (§2) est implémentée dans `stores/anime.ts` (upsert) +
  `usePersistence` (chargement) + `useSync` (synchro).
- Le **pont** (§4) est la table à consulter avant de rédiger une US : elle détermine le
  périmètre de fichiers, donc la classe de risque (`PILOTAGE.md §3`).
