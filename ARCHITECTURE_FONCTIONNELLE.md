# ARCHITECTURE_FONCTIONNELLE.md — Vue fonctionnelle & taxonomie des EPICs

> **Rôle :** ce que l'app **fait** du point de vue utilisateur, **où** chaque chose vit (les
> EPICs), et **quels fichiers** portent chaque fonction. Pendant technique :
> `ARCHITECTURE_TECHNIQUE.md`.
> **Pas ici :** aucun état d'avancement, aucun bug ouvert (→ `STATE.md`, `AUDIT.md`), aucune
> priorité (→ `ROADMAP.md`). Ce document décrit le **comportement attendu**, pas sa santé.
>
> *`EPICS.md` a été fusionné ici en SE-075 : les deux documents décrivaient le même axe — le
> « OÙ » fonctionnel.*

---

## 1. Taxonomie — 12 EPICs

Sert à situer une US : `US-XXX [EPIC][SECTION][TYPE]`. Répond à « est-ce que ça existe déjà ? »
sans confronter deux listes qui divergent.

| # | EPIC | Surface | État |
|---|---|---|---|
| 1 | **On Air** | Calendar Week / Month / List | ✅ Mature |
| 2 | **Discover** | This Season / For You / Coming Soon | ✅ Mature |
| 3 | **Library** | Completed / Plan to Watch / Upcoming | ✅ Mature |
| 4 | **Recherche** | `SearchInput` | ✅ Mature (AniList) |
| 5 | **Modal** | `AnimeModal` / `LogoutConfirmModal` / `RecEngineModal` | ✅ Clos |
| 6 | **Navigation** | Header / navs / routing | ✅ Refonte scroll livrée |
| 7 | **Login & Authentification** | `LoginPage` / magic-link / logout | ✅ Mature |
| 8 | **Boot & Démarrage** 🔴 | Orchestration `App.vue`, persistance, sync | ✅ Durci |
| 9 | **Onboarding & Rétention** | `/welcome` → `OnboardingPage` — **porte la North Star** | ✅ Livré |
| 10 | **Moteur de Recommandation** 🔴 | `useRecommendations` / `recEngine` | ✅ Socle livré |
| 11 | **Évolution Majeure** | Stats / monétisation / notifications | 🟡 Stats livré, reste à venir |
| 12 | **Plateforme & Dette Technique** | Build / CI / tests / infra | 🔄 Continu |

**Classes de risque** (`PILOTAGE §3`) : EPIC 8 et le moteur d'EPIC 10 sont 🔴 par défaut. Tout le
reste est 🟠 ou 🟢 selon la nature de l'US.

---

## 2. Le modèle métier : les 4 `state`

Chaque anime suivi porte un `state` qui détermine **où il apparaît**.

| State | Sens utilisateur | Onglet | Libellé visible |
|---|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air (Week / Month) | — |
| `radar` | À venir, pas encore diffusé | Discover | **« Coming Soon »** |
| `watchlist` | Prévu de regarder | Library | **« Plan to Watch »** |
| `vault` | Terminé / archivé | Library | **« Completed »** |

> 🔴 **`radar` et `vault` sont du jargon interne.** Tout toast, libellé ou message nomme
> **l'onglet réel** : « Coming Soon », « Completed », « Finished airing ».

### Transitions automatiques

```
  radar ──(diffusion commencée : début ≤ aujourd'hui − 7 j)──▶ calendar
  watchlist ──(l'utilisateur commence à regarder)───────────▶ calendar
  * ──(diffusion terminée)──▶ vault        [SENS UNIQUE]
```

- `radar → calendar` : géré dans `usePersistence` (au chargement) + `useSync` (à la synchro).
- `* → vault` : **auto-vault** dès le statut terminé, **sens unique**. Toast « Moved to Completed » au boot.
- **`awaitingSchedule`** : une série en cours sans horaire prouvé est démotée hors grille et **repromue par `useSync`** dès qu'un `day` arrive (`DEC-131`).
- **Hiatus** : un anime `calendar` en cours sans diffusion depuis plus de **14 j** est signalé « Hiatus » — calcul dérivé `isOnHiatus`, source unique.

> ⚠️ **Conséquence pour les tests :** un anime `calendar` + terminé s'auto-vault au boot et
> **disparaît** de la vue semaine. Pour tester une action sur un anime terminé, passer par
> `watchlist` (exclu de l'auto-vault).

---

## 3. Parcours utilisateur

### P0 — Première visite (onboarding) · EPIC 9
`/welcome` → `OnboardingPage.vue` : choix de **3 genres** → **8 suggestions** scorées
(`selectOnboardingSuggestions`), ajout en 1 tap → atterrissage sur un calendrier pré-rempli.
Séquence exacte : `finishWithSeed` → `addAnime` par item coché → `markOnboarded` →
`saveToDatabase` → toast → `router.push('/week')`.

> C'est le parcours qui porte la North Star. Règle qui en découle : **un message de confirmation
> porte sur ce que l'utilisateur va voir, pas sur ce que le code vient d'exécuter.**

### P1 — Connexion & déconnexion · EPIC 7
Saisie de l'email sur `/login` (input in-app, **pas de `window.prompt`**) → lien Firebase →
session ouverte. **Redirect post-login = `/`** (`DEC-82`). **Déconnexion :** bouton header →
`LogoutConfirmModal` → `signOut()` + purge des clés `aanime_*` + redirection `/login`.
Entrée et sortie de session passent par un **rechargement complet** (`DEC-176`).

### P2 — Boot & chargement · EPIC 8
Loader statique pendant le parse du bundle, puis `LoadingOverlay`. **Boot en 2 phases :** paint
immédiat depuis le cache local, réconciliation Firestore en fond par comparaison de timestamps.
Synchro AniList → reconstruction du graphe de recommandations → re-score. Couvert par le smoke
test `App.spec.ts`.

### P3 — Suivre un anime · EPIC 4
Recherche dans le header → suggestions enrichies (statut, épisodes, année, studio, score,
dual-titre) → clic → ajout avec `state` calculé **automatiquement** → **toast nommant la
destination visible exacte**. Séparation visuelle « IN YOUR LIBRARY » / « ADD TO YOUR LIST ».
L'état « ✓ Added » est **cliquable** : il retire l'anime d'où qu'il soit, avec un toast
« Removed ». Un bouton « + » permet l'ajout direct depuis les suggestions.

### P4 — Consulter le calendrier · EPIC 1
- **Week** : colonnes par jour, cartes-lignes avec heure locale et numéro d'épisode calculé. Snap sur aujourd'hui à l'ouverture. Barre de progression fine. Bouton ✓ « marquer vu » en 1 tap — **masqué** sur Airing / Hiatus (l'action reste dans la modale).
- **Jours vides** : suggestion de remplissage, écartable **pour la session seulement**, + CTA sur jour vide.
- **Month** : grille de 42 cellules. 🔻 Remplacée par un **« Coming Soon » assumé** (`DEC-163`) — `MonthDayCell.vue` et les classes `month-*` sont conservés pour la refonte.
- **List** : stub `CalendarListPage`.
- **Navigation de date unifiée** : Prev / période cliquable (= Today) / Next, une seule barre, **libellé à source unique**.

### P5 — Découvrir · EPIC 2
**For You** (recommandations scorées, chips de presets, section « Because you watched X », scroll
infini, nudges de séquelles) · **This Season** (catalogue AniList, saison courante + suivante) ·
**Coming Soon** (le radar de l'utilisateur). Déduplication des pools unifiée par `dedupeByMalId`.

### P6 — Gérer sa bibliothèque · EPIC 3
**Upcoming** (« you might have missed ») · **Plan to Watch** (watchlist triée) · **Completed**
(vault trié par date de complétion). Studios normalisés en `studios: string[]` — la dimension
studio du scoring est active.

### P7 — Détail d'un anime · EPIC 5
Modale contextuelle (`calendar` / `version` / `library-rec`) : relations (séquelles) avec covers
enrichies, « more like this », override du numéro d'épisode, sheet « quand l'as-tu vu ? » pour le
scoring de recency. Les trois modales partagent `.modal-backdrop`.

### P8 — Statistiques · EPIC 11
`/stats` (derrière le guard auth) : « Mon année anime » (`useStats` + `StatsPage.vue`, 4 cas
couverts en spec). Les **top genres sont calculés sur le contenu terminé cette année uniquement**
— un year-in-review compte ce qui a été *consommé*, pas les intentions.

### P9 — Exporter / Importer
**Export** `.ics` du planning. **Import MyAnimeList** : upload d'un XML → ajout silencieux ; les
titres `Dropped` **ne sont pas importés**.

---

## 4. LE PONT — fonctionnalité ▶ technique

**Tableau central.** À consulter **avant** de toucher une fonctionnalité : il détermine le
périmètre réel d'une US, donc sa classe de risque (`PILOTAGE §3`).

| Fonctionnalité | Page(s) / Composant | Composables / Stores | Utils purs |
|---|---|---|---|
| Onboarding 1ʳᵉ visite | `OnboardingPage` | `stores/anime.addAnime`, `usePersistence.saveToDatabase` | `onboardingFilter` |
| Connexion email-link | `LoginPage` | `useFirebaseAuth`, `lib/firebase` | — |
| Déconnexion | `AppHeader` / `LogoutConfirmModal` | `useFirebaseAuth.signOut`, `stores/anime.clearAll` | — |
| Boot & chargement | `App.vue` | `usePersistence`, `useSync`, `useRecommendations` | — |
| Recherche & ajout | `AppHeader` / `SearchInput` | `useAniListApi.searchAnime`, `useAddAnime` | `normalizeAniList` |
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
| Relations (séquelles) | AniList | Cache IndexedDB persistant, **sans expiration**. ⚠️ Le store porte la forme MyAnimeList héritée, lue par le scoring — ne jamais y écrire une relation AniList (`DEC-144`) |
| Saison courante / suivante | AniList | Cache localStorage, TTL 24 h. **Le cache périmé est servi si le fetch échoue** — comportement délibéré qui évite une page vide |

**Deux avertissements de fraîcheur, à ne pas confondre :**
- **`staleDataWarning`** — les données de l'utilisateur datent de plus d'un mois (`usePersistence`).
- **Cache de saison périmé** — servi silencieusement ; `error.value` renseigné mais **jamais affiché**.

> ⚠️ Ces deux signaux sont **sans consommateur dans l'UI**. Dette assumée et suivie (`AUD-05`).
> `DEC-158` tranche la source unique : `WithMeta.stale`. Ce n'est pas un bug à corriger au fil
> d'une autre US.

---

## 6. Lecture croisée

- Chaque **parcours** (§3) correspond à une séquence documentée dans `ARCHITECTURE_TECHNIQUE §5-6`.
- Chaque **transition de state** (§2) est implémentée dans `stores/anime.ts` (upsert) + `usePersistence` (chargement) + `useSync` (synchro).
- Le **pont** (§4) est la table à consulter avant de rédiger une US.
