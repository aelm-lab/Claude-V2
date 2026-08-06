# TYPES_CONTRACT.md — Contrat TypeScript de référence

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** la **seule** source d'interfaces du projet. Toute US copie ses types d'ici,
> verbatim, dans son corps — Gemini n'a pas accès à ce fichier.
>
> **Règle absolue :** on n'invente **jamais** un type. Si un type manque, on crée d'abord une
> US « types » pour l'ajouter ici, **puis** on l'utilise.
>
> **Ce qui n'est PAS ici :** les bugs ouverts et la dette (→ `STATE.md`), le pourquoi des
> choix de typage (→ `DECISIONS.md`). Ce document décrit **la forme des données**, pas leur
> état de santé.

---

## 0. Trois faits gravés — ne jamais réintroduire

- **`syncStatus` n'existe pas** dans `AnimeEntry` (0 hit dans `src/`, vérifié à l'audit s16).
  Ne pas l'ajouter.
- **L'action de remplacement du store s'appelle `setData`** (+ `clearAll`). Il n'existe
  **pas** de `setAllData`.
- **`reconcileWithDatabase` n'existe plus.** La réconciliation au chargement se fait dans
  `loadFromDatabase`.

Ces trois affirmations ont été inscrites à tort dans un handoff antérieur, puis réfutées par
grep. Un handoff est une source secondaire faillible : **le code réel tranche** (R3).

---

## 1. États & statuts

```ts
// src/types/anime.ts

/** State interne de l'app (onglet de classement). */
export type AnimeState = 'calendar' | 'radar' | 'watchlist' | 'vault';

/** Statut de diffusion (vocabulaire Jikan + valeurs legacy à normaliser). */
export type AnimeStatus =
  | 'Currently Airing'
  | 'Not yet aired'
  | 'Finished Airing'
  // legacy, présentes en cache et injectées par la persistance :
  | 'Continuing'
  | 'Ended'
  | 'Finished';

export type WeekDay =
  | 'monday' | 'tuesday' | 'wednesday' | 'thursday'
  | 'friday' | 'saturday' | 'sunday';

/** Tri de recency dans le moteur de recommandations. */
export type RecencyBucket = 'recent' | 'mid' | 'old';
```

> **Règle de consommation d'une union :** quand un type union gagne une valeur, **grep tous
> les `switch` et chaînes de `if` qui le consomment**. Un membre jamais mappé tombe
> silencieusement sur le `return` par défaut. C'est exactement ce qui est arrivé à
> `'Continuing'` dans `getCardStatus` (un show en cours s'affichait « Finished »).

---

## 2. Entité principale : `AnimeEntry`

Forme canonique renvoyée par `normalizeAnime`, enrichie des champs runtime utilisés par les
vues et le moteur de recommandations.

```ts
// src/types/anime.ts
import type { RecBadge, RecSignal } from './recs';

export interface AnimeEntry {
  // Identité (toujours numériques après normalisation)
  mal_id: number;
  id: number;

  // Métadonnées de base
  title: string;
  cover_url: string | null;
  cover_url_hd: string | null;
  studio: string | null;          // SINGULIER — null si inconnu, jamais "Unknown Studio". Conservé pour l'AFFICHAGE.
  genres: string[];
  themes: string[];
  demographics: string[];
  score: number | null;
  status: AnimeStatus | null;
  members: number;

  // Dates
  aired_from: string | null;      // ISO
  startDate: string | null;       // alias de aired_from (code calendrier legacy)
  lastAired?: string | null;      // ISO
  completedAt?: string;           // ISO — passage en vault
  startedAt?: string;             // ISO — passage en calendar

  // Classement & planning
  state: AnimeState | null;
  episodes: number | null;
  day?: WeekDay;                  // jour de diffusion LOCAL — voir note ci-dessous
  airsTime?: string | null;       // "HH:mm" local
  episodeOverride?: number;       // épisode forcé par l'utilisateur — RESET à undefined à chaque upsert (DEC-84)
  recencyBucket?: RecencyBucket;

  // Champs internes au moteur de recommandations (préfixe _, optionnels)
  isRec?: boolean;
  why?: string;
  _computedScore?: number;
  _recencyPenalty?: number;
  _badge?: RecBadge;
  _signals?: RecSignal[];
  _extractedByBYW?: boolean;
  _triggerTitle?: string;

  // Moteur de recommandations
  studios: string[];              // PLURIEL — lu par scorePool. TOUJOURS peuplé par normalizeAnime (DEC-86).
  popularityScore?: number;       // fallback popularité quand historyCount < 5
  _relevanceScore?: number;       // produit par scorePool
  _presetScore?: number;          // produit par applyPreset
}
```

**`studio` vs `studios` — les deux coexistent volontairement.** `studio` (singulier) sert
l'**affichage** ; `studios: string[]` sert le **scoring** (`scorePool`). `normalizeAnime`
produit toujours `studios` (fallback sur le singulier, filtre `"Unknown"`), d'où le typage
non optionnel.

🔴 **`day` n'est produit par aucun chemin de normalisation.** `normalizeAnime` ne pose ni
`day` ni `airsTime`. Or `CalendarWeekPage` filtre sur `state === 'calendar' && day === …` :
**une entrée sans `day` est stockée mais invisible partout**. Toute fonction qui crée une
entrée `state:'calendar'` doit se demander qui posera son `day` (DEC-115).

❌ **`onHiatus?` a été SUPPRIMÉ du type** (DEC-84). Le hiatus est un calcul dérivé
(`isOnHiatus`, source unique). Ne pas le réintroduire.

---

## 3. Types calculés (UI)

```ts
// src/types/anime.ts

export interface CardStatus {
  dot: 'airing' | 'done' | 'upcoming' | 'behind';
  word: string;                   // "Airing", "Finished", "Upcoming", "Hiatus"
}

export interface AnimeEpisodeInfo {
  currentEp: number | null;
  broadcastEp: number | null;
  totalEps: number | null;
  show: boolean;
  isFinished: boolean;
  isOverride: boolean;
}

export interface JstParseResult {
  localDay: WeekDay;
  localTime: string | null;       // "HH:mm"
}
```

---

## 4. Moteur de recommandations

```ts
// src/types/recs.ts
import type { RecencyBucket } from './anime';   // import type-only (circulaire sans danger)

export interface TasteProfile {
  genres: Map<string, number>;
  themes: Map<string, number>;
  demographics: Map<string, number>;
  studios: Map<string, number>;
  negativeIds: Set<number>;
  historyTitles: Map<number, string>;
  historyCount: number;
  hasSignal?: boolean;
  genreScores?: Map<string, number>;
}

export type RecAction = 'heart' | 'confirm' | 'manual-add' | 'dismiss' | 'remove';

export interface HistoryItem {
  id: number;
  action: RecAction;
  genres?: string[];
  themes?: string[];
  demographics?: string[];
  studios?: string[];
  title?: string;
  completedAt?: string;
  recencyBucket?: RecencyBucket;
}

export interface RecBadge {
  type: string;                   // "trending", "hidden-gem"…
  icon: string;
  label: string;
}

export type RecSignalKind = 'relation' | 'studio' | 'genre' | 'theme' | 'score';

export interface RecSignal {
  kind?: RecSignalKind;
  icon?: string;
  label: string;
}

export type RecContext = 'incoming' | 'library';

export type RecPreset =
  | 'all' | 'Less sequels' | 'Short binges'
  | 'Hidden gems' | 'Trending' | 'Genre variety';
```

---

## 5. Persistance

```ts
// src/types/persistence.ts
import type { AnimeEntry } from './anime';

export interface ScheduleDocument {
  timestamp: number;
  data: AnimeEntry[];
}

export type IdbStoreName = 'relations' | 'recommendations';
```

---

## 6. Constantes partagées

```ts
// src/utils/constants.ts
export const POSTER_PLACEHOLDER: string;   // source UNIQUE (DEC-84) — 4 copies inline supprimées
```

---

## 7. Types locaux — NON contractuels

Ces types sont locaux à un fichier util et **ne doivent pas** être déplacés ici : ce sont des
DTO de plomberie, pas des types métier.

| Type | Emplacement |
|---|---|
| `RawAnime`, `RawNamed`, `RawListItem`, `RawImages` | local non exporté, `utils/normalize.ts` |
| `MalImportEntry` | local non exporté, `utils/malImport.ts` |
| `MalImportResult` | **exporté** depuis `utils/malImport.ts` (reste près de sa fonction) — expose `imported`, pas `entries` |

---

## 8. Signatures des composables & stores

> Les valeurs exposées vers l'extérieur sont **`readonly`** ou des `computed`.

```ts
// useJikanApi
searchAnime(query: string): Promise<AnimeEntry[]>
fetchAnimeById(id: number): Promise<AnimeEntry | null>
fetchAnimeRelations(malId: number): Promise<unknown[]>
fetchAnimeRecommendations(malId: number): Promise<unknown[]>
fetchAnimeRelationsWithMeta(malId: number): Promise<{ data: unknown[]; fromNetwork: boolean }>
fetchAnimeRecommendationsWithMeta(malId: number): Promise<{ data: unknown[]; fromNetwork: boolean }>
fetchCurrentSeason(): Promise<AnimeEntry[]>        // sert le cache PÉRIMÉ si le fetch échoue (DEC-114)
fetchUpcomingSeason(): Promise<AnimeEntry[]>

// usePersistence
loadFromDatabase(): Promise<void>                  // inclut migration des clés aanime_* + réconciliation
saveToDatabase(showFeedback?: boolean): Promise<void>
staleDataWarning: Readonly<Ref<boolean>>

// useSync
syncAnimeUpdates(): Promise<void>
startBackgroundRelationFetch(): Promise<void>
isSyncing: Readonly<Ref<boolean>>

// useRecommendations
fetchRecPool(type: RecContext, force?: boolean): Promise<AnimeEntry[]>
getNextBatch(type: RecContext, size: number): AnimeEntry[]
reScorePool(): void
buildRelationMemory(): Promise<void>
getBecauseYouWatchedBatch(type: RecContext): AnimeEntry[]
getSlotFillSuggestions(list: AnimeEntry[]): Partial<Record<WeekDay, AnimeEntry>>
getSeasonNudges(): AnimeEntry[]
fetchTopFinishedAnime(): Promise<AnimeEntry[]>     // ⚠️ vit ICI, inline — PAS dans useJikanApi (extraction = US-165)

// useEpisodeInfo (wrappe utils/episodeInfo.ts)
getAnimeEpisodeInfo(anime: AnimeEntry, targetDate?: Date): AnimeEpisodeInfo
getCardStatus(anime: AnimeEntry): CardStatus
isOnHiatus(anime: AnimeEntry): boolean

// stores/anime (actions)
setData(data: AnimeEntry[]): void                  // PAS de setAllData
clearAll(): void
addAnime(input: Partial<AnimeEntry>): void         // upsert — reset episodeOverride à undefined (DEC-84)
addAnimeSilent(input: Partial<AnimeEntry>): void
removeAnime(id: number): void
setDate(date: Date): void

// useToast
showToast(message: string, isHtml?: boolean, durationMs?: number): void

// useTheme
isDark: Readonly<Ref<boolean>>
toggleDarkMode(): void

// useFirebaseAuth
sendSignInLink(email: string): Promise<void>
completeSignIn(): Promise<void>
onAuthStateChanged(callback: (user: User | null) => void): Unsubscribe
signOut(): Promise<void>

// useICS / useMalImport
downloadICS(): void
importMalFile(file: File): Promise<MalImportResult>
```

> **`getAnimeEpisodeInfo`** : l'util sous-jacent impose `targetDate: Date` **obligatoire**.
> `useEpisodeInfo` fournit `new Date()` à l'appel pour exposer un `targetDate?` optionnel.

---

## 9. ⚠️ Lacunes assumées du contrat

Ces éléments **existent dans le code** d'après la documentation du projet mais n'ont jamais
été contractualisés ici. Ils sont listés pour éviter que Gemini n'invente un type en leur
absence. **Aucun ne doit être utilisé dans une US avant d'avoir été relu dans le code et
ajouté ci-dessus par une US « types ».**

| Élément | Source documentaire | Ce qui manque |
|---|---|---|
| `AnimeEntry.synopsis?` | DEC-47 — « `synopsis?` ajouté à AnimeEntry » | Le champ **n'apparaît pas** dans l'interface §2. Type exact à confirmer |
| `useStats` | DEC-88, EPIC 11 — `completedThisYear`, `topGenres` | Aucune signature contractualisée |
| `useOnboarding` / `finishWithSeed` / `markOnboarded` | EPIC 9 (livré) | Aucune signature contractualisée |
| `utils/onboardingFilter.ts` — `buildSeedEntry`, `selectOnboardingSuggestions` | EPIC 9, DEC-115 | `buildSeedEntry` retourne `{ ...anime, id, state }` et **ne pose jamais `day`**. Signature exacte à confirmer |
| `getAnimeTitle` | DEC-94 — règle de titre centralisée (anglais primaire + rōmaji si différent) | Emplacement et signature à confirmer |

> Cette section est un **filet, pas une dette permanente** : chaque ligne se ferme par une
> lecture du code (R3) et une US « types ». Elle ne doit pas grossir sans être traitée.
