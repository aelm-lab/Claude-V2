# TYPES_CONTRACT.md — Contrat TypeScript de référence

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Règle absolue :** Gemini **n'invente JAMAIS un type**. Toute interface utilisée dans une US provient d'ici. Si un type manque, on crée d'abord une US « types » pour l'ajouter à ce fichier, **puis** on l'utilise. Ces interfaces sont déduites du code vanilla existant (`normalize.js`, `utils.js`, `rec-engine.js`, `store.js`).

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
  // legacy à migrer vers les valeurs ci-dessus :
  | 'Continuing'
  | 'Ended'
  | 'Finished';

export type WeekDay =
  | 'monday' | 'tuesday' | 'wednesday' | 'thursday'
  | 'friday' | 'saturday' | 'sunday';

/** Pour le tri de recency dans le moteur de recs. */
export type RecencyBucket = 'recent' | 'mid' | 'old';
```

---

## 2. Entité principale : `AnimeEntry`

Forme canonique renvoyée par `normalizeAnime`, enrichie des champs runtime utilisés par les vues.

```ts
// src/types/anime.ts

export interface AnimeEntry {
  // Identité (toujours numériques après normalisation)
  mal_id: number;
  id: number;

  // Métadonnées de base
  title: string;
  cover_url: string | null;
  cover_url_hd: string | null;
  studio: string | null;          // null si inconnu — NE PAS mettre "Unknown Studio"
  genres: string[];
  themes: string[];
  demographics: string[];
  score: number | null;
  status: AnimeStatus | null;
  members: number;

  // Dates
  aired_from: string | null;       // ISO
  startDate: string | null;        // alias de aired_from (code calendrier legacy)
  lastAired?: string | null;       // ISO
  completedAt?: string;            // ISO — passage en vault
  startedAt?: string;              // ISO — passage en calendar

  // Classement & planning
  state: AnimeState | null;
  episodes: number | null;
  day?: WeekDay;                   // jour de diffusion local
  airsTime?: string | null;        // "HH:mm" local
  episodeOverride?: number;        // numéro d'épisode forcé par l'utilisateur
  onHiatus?: boolean;
  recencyBucket?: RecencyBucket;

  // Champs internes au moteur de recs (préfixe _, optionnels)
  isRec?: boolean;
  why?: string;
  _computedScore?: number;
  _recencyPenalty?: number;
  _badge?: RecBadge;
  _signals?: RecSignal[];
  _extractedByBYW?: boolean;
  _triggerTitle?: string;
}
```

---

## 3. Types calculés (UI)

```ts
// src/types/anime.ts

export interface CardStatus {
  dot: 'airing' | 'done' | 'upcoming' | 'behind';
  word: string;                    // ex: "Airing", "Finished", "Upcoming", "Hiatus"
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
  localTime: string | null;        // "HH:mm"
}
```

---

## 4. Moteur de recommandations

```ts
// src/types/recs.ts

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

export type RecAction =
  | 'heart' | 'confirm' | 'manual-add' | 'dismiss' | 'remove';

export interface HistoryItem {
  id: number;
  action: RecAction;
  genres?: string[];
  themes?: string[];
  demographics?: string[];
  studios?: string[];
  title?: string;
}

export interface RecBadge {
  type: string;                    // ex: "trending", "hidden-gem"
  icon: string;
  label: string;
}

export interface RecSignal {
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

export interface ScheduleDocument {
  timestamp: number;
  data: AnimeEntry[];
}

export type IdbStoreName = 'relations' | 'recommendations';
```

---

## 6. Signatures clés des composables (contrat d'API)

> Les valeurs retournées exposées vers l'extérieur sont **`readonly`** ou des `computed`.

```ts
// useJikanApi
searchAnime(query: string): Promise<AnimeEntry[]>
fetchAnimeById(id: number): Promise<AnimeEntry | null>
fetchAnimeRelations(malId: number): Promise<unknown[]>
fetchAnimeRecommendations(malId: number): Promise<unknown[]>
fetchCurrentSeason(): Promise<AnimeEntry[]>
fetchUpcomingSeason(): Promise<AnimeEntry[]>
fetchTopFinishedAnime(): Promise<AnimeEntry[]>

// usePersistence
loadFromDatabase(): Promise<void>
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
getBecauseYouWatchedBatch(type: RecContext): AnimeEntry[]
getSlotFillSuggestions(list: AnimeEntry[]): Partial<Record<WeekDay, AnimeEntry>>

// useEpisodeInfo (pur)
getAnimeEpisodeInfo(anime: AnimeEntry, targetDate?: Date): AnimeEpisodeInfo
getCardStatus(anime: AnimeEntry): CardStatus
isOnHiatus(anime: AnimeEntry): boolean

// useToast
showToast(message: string, isHtml?: boolean, durationMs?: number): void

// useTheme
isDark: Readonly<Ref<boolean>>
toggleDarkMode(): void
```
