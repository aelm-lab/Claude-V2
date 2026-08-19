# ARCHITECTURE_FONCTIONNELLE.md — Vue fonctionnelle

> **Rôle :** ce que l'app **fait** du point de vue utilisateur, et quels fichiers portent chaque fonction. Pendant technique : `ARCHITECTURE_TECHNIQUE.md`.
> **Pas ici :** aucun état d'avancement, aucun bug ouvert (→ `STATE.md`, `EPICS.md`). Ce document décrit le **comportement attendu**, pas son état de santé.

---

## 1. Ce que fait l'application

L'utilisateur suit les animes en cours de diffusion sur un calendrier (semaine / mois), découvre de nouvelles séries via un moteur de recommandations personnalisé, gère sa bibliothèque (terminés / à voir / à venir), exporte son planning en `.ics`, importe sa liste MyAnimeList (XML), voit ses données synchronisées entre le cloud et le local, et peut se déconnecter avec purge des clés `aanime_*` et reset du store.

---

## 2. Le modèle métier : les 4 `state`

Chaque anime suivi porte un `state` qui détermine **où il apparaît**.

| State | Sens utilisateur | Onglet | Libellé visible |
|---|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air (Week / Month) | — |
| `radar` | À venir, pas encore diffusé | Discover | **« Coming Soon »** |
| `watchlist` | Prévu de regarder | Library | **« Plan to Watch »** |
| `vault` | Terminé / archivé | Library | **« Completed »** |

> 🔴 **`radar` et `vault` sont du jargon interne.** Tout toast, libellé ou message nomme **l'onglet réel** : « Coming Soon », « Completed ».

### Transitions automatiques

```
  radar ──(diffusion commencée : début ≤ aujourd'hui − 7 j)──▶ calendar
  watchlist ──(l'utilisateur commence à regarder)───────────▶ calendar
  * ──(diffusion terminée)──▶ vault        [SENS UNIQUE]
```

- `radar → calendar` : géré dans `usePersistence` (au chargement) + `useSync` (à la synchro).
- `* → vault` : **auto-vault** dès le statut terminé, **sens unique**. Un toast « Moved to Completed » signale le déplacement au boot.
- **`awaitingSchedule`** : une série en cours sans horaire de diffusion prouvé est démotée hors grille et **repromue par `useSync`** dès qu'un `day` arrive (DEC-131).
- **Hiatus** : un anime `calendar` en cours sans diffusion depuis plus de **14 j** est signalé « Hiatus » — calcul dérivé `isOnHiatus`, source unique.

> ⚠️ **Conséquence pour les tests :** un anime `calendar` + terminé s'auto-vault au boot et **disparaît** de la vue semaine. Pour tester une action sur un anime terminé, passer par `watchlist` (exclu de l'auto-vault).

---

## 3. Parcours utilisateur

### P0 — Première visite (onboarding)
`/welcome` → `OnboardingPage.vue` : choix de **3 genres** → **8 suggestions** scorées, ajout en 1 tap → atterrissage sur un calendrier pré-rempli (`/week`).

> C'est le parcours qui porte la North Star. Règle qui en découle : **un message de confirmation porte sur ce que l'utilisateur va voir, pas sur ce que le code vient d'exécuter.**

### P1 — Connexion & déconnexion (lien email, sans mot de passe)
Saisie de l'email sur `/login` (input in-app, pas de `window.prompt`) → lien Firebase → session ouverte. **Redirect post-login = `/`**, pas la route d'origine (DEC-82). **Déconnexion** : bouton header → modale de confirmation → `signOut()` + purge des clés `aanime_*` + `clearAll()` + redirection `/login`.

### P2 — Boot & chargement
Loader statique pendant le parse du bundle, puis `LoadingOverlay`. Paint immédiat depuis le cache local, réconciliation Firestore en fond. Synchro AniList → reconstruction du graphe de recommandations → re-score. L'overlay se lève, la semaine s'affiche.

### P3 — Suivre un anime
Recherche dans le header → suggestions enrichies (statut, nombre d'épisodes, année, studio, score, dual-titre) → clic → ajout avec `state` calculé **automatiquement** selon le statut de diffusion → **toast nommant la destination visible exacte**. L'état « ✓ Added » est cliquable : il **retire** l'anime d'où qu'il soit, avec un toast « Removed ». La synchro de fond récupère horaires, épisodes et score.

### P4 — Consulter le calendrier (Week / Month / List)
- **Week** : colonnes par jour, cartes-lignes avec heure locale et numéro d'épisode calculé. Snap sur aujourd'hui à l'ouverture. Barre de progression fine. Bouton ✓ « marquer vu » en 1 tap — **masqué** sur une série en cours ou en hiatus (l'action reste dans la modale).
- **Jours vides** : suggestion de remplissage, écartable **pour la session seulement** (jamais persisté), + CTA sur jour vide.
- **Month** : grille de 42 cellules, points d'anime par jour.
- **Navigation de date unifiée** : Prev / période cliquable (= Today) / Next, une seule barre, **libellé à source unique**.

### P5 — Découvrir (Discover)
**For You** (recommandations scorées, chips de presets, section « Because you watched X », scroll infini, nudges de séquelles) · **This Season** · **Coming Soon** (le radar de l'utilisateur).

### P6 — Gérer sa bibliothèque (Library)
**Upcoming** (« you might have missed ») · **Plan to Watch** (watchlist triée) · **Completed** (vault trié par date de complétion).

### P7 — Détail d'un anime (modale)
Modale contextuelle (`calendar` / `version` / `library-rec`) : relations (séquelles), covers enrichies, « more like this » par genres, override du numéro d'épisode, sheet « quand l'as-tu vu ? » pour le scoring de recency.

### P8 — Statistiques
`/stats` (derrière le guard auth) : « Mon année anime ». Les **top genres sont calculés sur le contenu terminé cette année uniquement** — un year-in-review compte ce qui a été *consommé*, pas les intentions.

### P9 — Exporter / Importer
**Export** `.ics` du planning. **Import MyAnimeList** : upload d'un XML → ajout silencieux ; les titres `Dropped` **ne sont pas importés**.

---

## 4. LE PONT — fonctionnalité ▶ technique

**Tableau central.** À consulter **avant** de toucher une fonctionnalité : il détermine le périmètre réel d'une US, donc sa classe de risque (`PILOTAGE.md §3`).

| Fonctionnalité | Page(s) / Composant | Composables / Stores | Utils purs |
|---|---|---|---|
| Onboarding 1ʳᵉ visite | `OnboardingPage` | `stores/anime.addAnime`, `usePersistence.saveToDatabase` | `onboardingFilter` |
| Connexion email-link | `LoginPage` | `useFirebaseAuth`, `lib/firebase` | — |
| Déconnexion | `AppHeader` / `LogoutConfirmModal` | `useFirebaseAuth.signOut`, `stores/anime.clearAll` | — |
| Boot & chargement | `App.vue` | `usePersistence`, `useSync`, `useRecommendations` | — |
| Recherche & ajout | `AppHeader` / `SearchInput` | `useAniListApi.searchAnime`, `stores/anime.addAnime` | `normalizeAniList` |
| Calendrier semaine | `CalendarWeekPage` | `stores/anime`, `useEpisodeInfo`, `useRecommendations.getSlotFillSuggestions` | `jst`, `episodeInfo` |
| Calendrier mois | `CalendarMonthPage` | `stores/anime`, `useEpisodeInfo` | `episodeInfo` |
| Navigation de date | `CalendarNavControls` (dans `SecondaryNav`) | `stores/anime.setDate` | — |
| Découvrir / For You | `DiscoverExplorePage` | `useRecommendations` (pools, BYW, nudges, presets) | `recEngine` |
| This Season | `DiscoverSeasonPage` | `useAniListApi.fetchCurrentSeason`, `stores/anime` | `normalizeAniList` |
| Coming Soon | `DiscoverComingUpPage` | `stores/anime` (filtre `radar`) | — |
| Library Upcoming | `LibraryExplorePage` | `useRecommendations` (contexte `library`) | `recEngine` |
| Plan to Watch | `LibraryPlanToWatchPage` | `stores/anime` (filtre `watchlist`) | — |
| Completed | `LibraryCompletedPage` | `stores/anime` (filtre `vault`) | — |
| Statistiques | `StatsPage` | `useStats` | — |
| Détail anime | overlay `AnimeModal` | `stores/ui`, `useAniListApi`, `useRecommendations` | `recEngine` |
| Override d'épisode | `EpOverridePanel` | `stores/ui`, `stores/anime` | — |
| Recency (« quand vu ? ») | `RecencySheet` | `stores/ui`, `useRecommendations` | — |
| Synchro de fond | (`App.vue`) | `useSync` | `jst`, `helpers` |
| Indicateur de synchro | `SyncIndicator` | `useSync.isSyncing` | — |
| Export ICS | `AppHeader` | `useICS.downloadICS` | `ics` |
| Import MyAnimeList | `MalImportButton` / `AppHeader` | `useMalImport`, `stores/anime.addAnimeSilent` | `malImport` |
| Thème sombre | `AppHeader` (toggle) | `useTheme` (`useDark` → `html.dark`) | — |
| Notifications | overlay `ToastNotification` | `useToast` | — |
| Persistance automatique | (transparent) | `usePersistence` (`watchDebounced`) → `useFirestore` | — |

---

## 5. Fraîcheur & synchronisation des données

| Donnée | Source | Fraîcheur |
|---|---|---|
| Liste de l'utilisateur | Firestore `schedules/{uid}` + cache local `aanime_calendar` | Chargement au boot, sauvegarde debouncée (1 s) à chaque modification |
| Statut / épisodes / horaire | AniList (détail par `idMal`) | Synchro batch au boot, TTL 6 h |
| Relations (séquelles) | AniList | Cache IndexedDB persistant, **sans expiration**. ⚠️ Le store porte la forme MyAnimeList héritée, lue par le scoring — ne jamais y écrire une relation AniList (DEC-144) |
| Saison courante / suivante | AniList | Cache localStorage, TTL 24 h. **Le cache périmé est servi si le fetch échoue** — comportement délibéré qui évite une page vide |

**Deux avertissements de fraîcheur, à ne pas confondre :**
- **`staleDataWarning`** — les données de l'utilisateur datent de plus d'un mois. État réactif exposé par `usePersistence`.
- **Cache de saison périmé** — servi silencieusement. `error.value` est renseigné mais **jamais affiché**. Sans cache et sans réseau, la liste est vide **sans explication**.

> ⚠️ Ces deux signaux sont aujourd'hui **sans consommateur dans l'UI**. C'est une dette assumée et suivie (`AUD-05`), qui exige un arbitrage préalable sur la source unique du signal (DEC-151) — pas un bug à corriger au fil d'une autre US.

---

## 6. Lecture croisée avec la vue technique

- Chaque **parcours** (§3) correspond à une séquence documentée dans `ARCHITECTURE_TECHNIQUE.md §5-6`.
- Chaque **transition de state** (§2) est implémentée dans `stores/anime.ts` (upsert) + `usePersistence` (chargement) + `useSync` (synchro).
- Le **pont** (§4) est la table à consulter avant de rédiger une US.
