# TYPES_CONTRACT.md — Contrat TypeScript de référence

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Règle absolue :** Gemini **n'invente JAMAIS un type**. Toute interface utilisée dans une US
> provient d'ici. Si un type manque, on crée d'abord une US « types » pour l'ajouter à ce
> fichier, **puis** on l'utilise. Ces interfaces sont déduites du code vanilla existant
> (`normalize.js`, `utils.js`, `rec-engine.js`, `store.js`).
>
> **État de référence : fin S33 (cleaning S34).** Intègre les corrections S15 (DEC-84/86),
> les vérifications zéro-confiance de l'audit s16 (DEC-87), et la signature `signOut`
> ajoutée S33 (US-AUTH-LOGOUT).
>
> **Trois faits confirmés (à ne pas réintroduire par erreur) :**
> - `syncStatus` **n'existe pas** dans `AnimeEntry` (0 hit dans `src/`). Ne pas l'ajouter.
> - L'action store s'appelle **`setData`** (+ `clearAll`). Il n'existe **pas** de `setAllData`.
> - La réconciliation au load se fait dans **`loadFromDatabase`** (il n'y a pas de `reconcileWithDatabase`).

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

> ⚠️ **Bug audité s16 (US-154, P1) — `getCardStatus`** : la valeur legacy `'Continuing'`
> est dans l'union et peut être injectée par la persistance, mais `getCardStatus`
> (`utils/episodeInfo.ts`) ne la mappe pas → elle tombe sur le `return` par défaut
> `{ dot: 'done', word: 'Finished' }`. Correctif planifié : mapper `'continuing'` → `Airing`.
> Le **type** reste correct ; c'est le **mapping** qui est incomplet.

---

## 2. Entité principale : `AnimeEntry`

Forme canonique renvoyée par `normalizeAnime`, enrichie des champs runtime utilisés par les
vues et le moteur de recs.

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
  studio: string | null;          // singulier — null si inconnu, NE PAS mettre "Unknown Studio". Conservé pour l'AFFICHAGE.
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
  episodeOverride?: number;        // numéro d'épisode forcé par l'utilisateur — RESET à undefined à chaque upsert (DEC-84)
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

  // ── Ajoutés en US-008-types (moteur de recs) ──
  studios: string[];               // PLURIEL — lu par scorePool. Désormais TOUJOURS peuplé par normalizeAnime (DEC-86). Voir note ci-dessous.
  popularityScore?: number;        // fallback popularité quand historyCount < 5
  _relevanceScore?: number;        // produit par scorePool
  _presetScore?: number;           // produit par applyPreset
}
```

> ✅ **`studios: string[]` — dette P8-01 RÉSOLUE (US-134/DEC-86).** Avant s15,
> `scorePool` lisait `item.studios` (pluriel) que `normalizeAnime` ne produisait jamais
> (il produisait `studio` singulier) → dimension studio du scoring **inerte**. Depuis
> US-134, `normalizeAnime` produit **toujours** `studios: string[]` (fallback sur le
> singulier `studio`, filtre `"Unknown"`). Le champ n'est donc plus optionnel en pratique
> sur les entrées normalisées → typé **non optionnel**. `studio` (singulier) est **conservé**
> pour l'affichage. `scorePool`/`recEngine` étaient déjà corrects (lisaient `item.studios`).
>
> ❌ **`onHiatus?` SUPPRIMÉ du type (US-132/DEC-84).** Champ persisté mais plus écrit depuis
> US-107 (hiatus = calcul dérivé `isOnHiatus`, source unique). Retiré du contrat. Ne pas
> le réintroduire.

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
  completedAt?: string;            // ── ajouté US-008-types (getRecencyWeight) ──
  recencyBucket?: RecencyBucket;   // ── ajouté US-008-types (getRecencyWeight) ──
}

export interface RecBadge {
  type: string;                    // ex: "trending", "hidden-gem"
  icon: string;
  label: string;
}

// ── ajouté US-008-types ──
export type RecSignalKind = 'relation' | 'studio' | 'genre' | 'theme' | 'score';

export interface RecSignal {
  kind?: RecSignalKind;            // ── ajouté US-008-types ──
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

> ⚠️ **Chemin legacy audité s16 (US-158, P1)** : `usePersistence` charge le cache local
> via `store.setData(loadedData as unknown as AnimeEntry[])` — double cast sans normalisation
> ni garde runtime. Cache local corrompu → cartes incomplètes / écran blanc. Correctif :
> garde runtime + normalisation sur le chemin legacy. Le **type** est correct ; c'est le
> **cast non sécurisé** qui est en cause (viole R-CODE-2). ✅ Résolu — cf. `EPICS.md` EPIC 8.

---

## 6. Constantes partagées

```ts
// src/utils/constants.ts
export const POSTER_PLACEHOLDER: string;   // source UNIQUE (US-132/DEC-84) — 4 copies inline supprimées
```

---

## 7. Types locaux (NON dans le contrat — pour mémoire)

Ces types sont **locaux à un fichier util** et NE doivent PAS être déplacés ici (ce sont des
DTO de plomberie, pas des types métier) :
- `RawAnime` (+ `RawNamed`, `RawListItem`, `RawImages`) → local non exporté dans `utils/normalize.ts`.
- `MalImportEntry` → local non exporté dans `utils/malImport.ts`.
- `MalImportResult` → **exporté** depuis `utils/malImport.ts` (pas un type métier partagé, reste près de sa fonction).

---

## 8. Signatures clés des composables (contrat d'API)

> Les valeurs retournées exposées vers l'extérieur sont **`readonly`** ou des `computed`.

```ts
// useJikanApi
searchAnime(query: string): Promise<AnimeEntry[]>
fetchAnimeById(id: number): Promise<AnimeEntry | null>
fetchAnimeRelations(malId: number): Promise<unknown[]>
fetchAnimeRecommendations(malId: number): Promise<unknown[]>
fetchAnimeRelationsWithMeta(malId: number): Promise<{ data: unknown[]; fromNetwork: boolean }>
fetchAnimeRecommendationsWithMeta(malId: number): Promise<{ data: unknown[]; fromNetwork: boolean }>
fetchCurrentSeason(): Promise<AnimeEntry[]>
fetchUpcomingSeason(): Promise<AnimeEntry[]>
fetchTopFinishedAnime(): Promise<AnimeEntry[]>   // ⚠️ encore inline dans useRecommendations (extraction = US-165, ex-US-123)

// usePersistence
loadFromDatabase(): Promise<void>                // inclut migration clés aanime_* + réconciliation
saveToDatabase(showFeedback?: boolean): Promise<void>   // ⚠️ saveSchedule sans try/catch → US-153 P0 (✅ résolu, cf. EPICS.md EPIC 8)
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

// useEpisodeInfo (pur — wrappe utils/episodeInfo.ts)
getAnimeEpisodeInfo(anime: AnimeEntry, targetDate?: Date): AnimeEpisodeInfo
getCardStatus(anime: AnimeEntry): CardStatus     // ⚠️ ne gère pas 'Continuing' → US-154 P1
isOnHiatus(anime: AnimeEntry): boolean

// stores/anime (actions)
setData(data: AnimeEntry[]): void                // PAS de setAllData (confirmé audit s16)
clearAll(): void
addAnime(input: Partial<AnimeEntry>): void       // upsert — reset episodeOverride à undefined (DEC-84)
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
signOut(): Promise<void>                          // ── ajouté S33 (US-AUTH-LOGOUT) ── try/catch conforme R-CODE-5, testé (T7 succès / T8 erreur)
```

> ⚠️ **Note signature `getAnimeEpisodeInfo`** : l'util porté (US-006a) impose `targetDate: Date`
> **obligatoire**. `useEpisodeInfo` fournit `new Date()` au moment de l'appel pour exposer
> un `targetDate?` optionnel.
