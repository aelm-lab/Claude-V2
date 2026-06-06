# BACKLOG.md — Kanban vivant + User Stories

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat. C'est le backlog de référence. Au fil des sprints, **je (le PO) recopie ici l'état du Kanban** pour que Claude reprenne au bon endroit à chaque session.

---

## 📋 Kanban — état initial

### ✅ Done
- _(rien encore)_

### 🔄 In Progress
- _(rien encore)_

### 📝 To Do — Sprint 1 (Phase 0 + début Phase 1)
- [US-001] Scaffold du projet Vue 3 + TS + Vite
- [US-002] Configuration ESLint + tsconfig strict + structure de dossiers
- [US-003] Définir les types de base dans `src/types/`

### 🗂️ Backlog (epics par phase)
- [PHASE-1] Logique pure : utils, normalize, recEngine, idb, ics, malImport
- [PHASE-2] Store Pinia + composables services
- [PHASE-3] Router + layouts
- [PHASE-4] Composants atomiques ui/
- [PHASE-5] Pages
- [PHASE-6] Modals & sheets
- [PHASE-7] Branchement final + nettoyage

---

## Découpage détaillé des US (sprints 1 à 3)

> Les US ci-dessous sont **pré-rédigées en version courte**. Claude doit les **développer au format complet** (template du PROMPT) au moment de les attaquer, en y injectant les interfaces depuis `TYPES_CONTRACT.md` et une section « anti-patterns ».

### PHASE 0 — Scaffold

**[US-001] Scaffold Vue 3 + TS + Vite**
- Fichiers : `package.json`, `vite.config.ts`, `index.html`
- But : retirer React, ajouter `vue`/`vue-router`/`pinia`/`@vueuse/core`/`@vitejs/plugin-vue`, monter une app Vue minimale qui affiche « Hello Vue ».
- Acceptance : `npm run dev` démarre, page blanche avec composant racine, aucune erreur console, plus aucune réf à React.

**[US-002] tsconfig strict + ESLint + arborescence**
- Fichiers : `tsconfig.json`, `.eslintrc.*`, création des dossiers vides `src/{components,composables,stores,utils,router,types}`.
- Acceptance : `tsc --noEmit` passe, ESLint configuré pour Vue+TS, `strict: true`, `noImplicitAny: true`.

### PHASE 1 — Logique pure

**[US-003] Types de base**
- Fichiers : `src/types/anime.ts`, `src/types/recs.ts`
- But : porter toutes les interfaces de `TYPES_CONTRACT.md`.
- Acceptance : `AnimeEntry`, `AnimeState`, `AnimeStatus`, `CardStatus`, `AnimeEpisodeInfo`, `TasteProfile`, `RecBadge`, `RecSignal` exportés et compilent.

**[US-004] `utils/jst.ts` (+ test)**
- Source : `parseJSTToLocal` de `src/utils.js`.
- Acceptance : reproduit la conversion JST→local à l'identique (date de référence 1970). Tests : « Mondays 23:00 » et un cas sans heure.

**[US-005] `utils/normalize.ts` (+ test)**
- Source : `src/normalize.js`.
- Acceptance : rejette « Unknown Studio », mappe genres/themes/demographics en `string[]`, gère les covers, renvoie `AnimeEntry`.

**[US-006] `utils/helpers.ts`**
- Source : `getAnimeEpisodeInfo`, `getCardStatus`, `isOnHiatus`, `getWeekNumber`, `fetchWithRetry` de `src/utils.js`.
- Acceptance : `getAnimeEpisodeInfo` couvert par ≥ 4 tests (avant diffusion, en cours, terminé, override).

**[US-007] `utils/idb.ts`**
- Source : `src/idb.js`. Acceptance : `idbGet`/`idbSet` typés, stores `relations`/`recommendations`.

**[US-008] `utils/recEngine.ts` (+ tests)**
- Source : `src/rec-engine.js`. Acceptance : `RelationMemory`, `buildTasteProfile`, `scorePool`, `assignBadge`, `buildNextBatch`, `extractBecauseYouWatched`, `applyPreset` portés et typés.

**[US-009] `utils/ics.ts`**
- Source : `src/ics.js`. Acceptance : génère un `.ics` valide pour les animes `state==='calendar'`.

**[US-010] `utils/malImport.ts` (+ test)**
- Source : `parseMalXml` de `src/ui/mal-import.js` (partie pure). Acceptance : parse un XML MAL, mappe les statuts → states, compte les skipped.

### PHASE 2 — Store + composables services

**[US-011] Store Pinia `stores/anime.ts`**
- Acceptance : state + `addAnime`/`addAnimeSilent`/`removeAnime`/`setDate`. **Aucun `dispatchEvent`.** Logique d'upsert + auto-vault reproduite.

**[US-012] `useFirebaseAuth.ts`**
- Source : `firebase.js` + `login.js`. Acceptance : `sendSignInLink`, `completeSignIn`, état réactif `currentUser`.

**[US-013] `useFirestore.ts`**
- Acceptance : `loadSchedule`/`saveSchedule` sur `schedules/{uid}`, try/catch + état d'erreur.

**[US-014] `usePersistence.ts`**
- Source : `persistence.js`. Acceptance : load Firestore + fallback localStorage ; le bandeau « > 1 mois » devient `staleDataWarning: ref<boolean>` (pas de DOM).

**[US-015] `useJikanApi.ts`**
- Source : `api.js`. Acceptance : tous les endpoints du tableau API, `fetchWithRetry`, gestion 429.

**[US-016] `useSync.ts`**
- Source : `api.js` (`syncAnimeUpdates`, `startBackgroundRelationFetch`). Acceptance : batch de 2 / 2 s, throttle 1,1 s reproduits.

**[US-017] `useRecommendations.ts`**
- Source : `recs.js`. Acceptance : pools, `getNextBatch`, `reScorePool`, BYW, slot-fill.

**[US-018] `useEpisodeInfo.ts`, `useToast.ts`, `useTheme.ts`**
- (à découper en 3 US si > 3 fichiers). Acceptance : composables réactifs sans DOM direct.

---

## Règles de tenue du backlog

- **Une seule US `In Progress` à la fois.** On ne démarre pas la suivante sans `MERGE`.
- Quand une US passe `MERGE` → Claude la déplace en `Done` et remonte la suivante en `In Progress`.
- Si une US dépasse 3 fichiers → Claude la **scinde** et ajoute les sous-US ici.
- Les **anti-patterns récurrents** détectés en review sont consignés dans `ANTIPATTERNS.md` et réinjectés dans les US suivantes.
-  Fixtures de test typées via helper Partial — interdit as any / as unknown as T.

