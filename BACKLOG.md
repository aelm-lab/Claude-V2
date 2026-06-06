# BACKLOG.md — Kanban vivant + User Stories

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat. C'est le backlog de référence. Au fil des sprints, **je (le PO) recopie ici l'état du Kanban** pour que Claude reprenne au bon endroit à chaque session.

---

## 📋 Kanban — état actuel (fin de session 1, Phase 1 close à 100 %)

### ✅ Done
- [US-001] Scaffold Vue 3 + TS + Vite (US-001b *App.vue + index.html* **absorbée** dedans)
- [US-002] tsconfig strict + ESLint flat config + arborescence (9 dossiers)
- [US-003] Types de base (`anime.ts`, `recs.ts`, `persistence.ts`)
- [US-004] `utils/jst.ts` (+ test) — conversion JST→local
- [US-005] `utils/normalize.ts` (+ test) — `normalizeAnime`
- [US-006a] `utils/episodeInfo.ts` (+ test) — `getAnimeEpisodeInfo` / `getCardStatus` / `isOnHiatus`
- [US-006b] `utils/helpers.ts` (+ test) — `escapeHTML` / `getWeekNumber` / `fetchWithRetry`
- [US-007] `utils/idb.ts` — wrapper IndexedDB
- [US-008-types] Extension du contrat de types pour le moteur de recs
- [US-008a] `utils/recEngine.ts` passe 1 — RelationMemory, buildTasteProfile, assignBadge, extractBecauseYouWatched, buildNextBatch
- [US-008b] `utils/recEngine.ts` passe 2 — scorePool + helpers de matching
- [US-009] `utils/ics.ts` — `buildICSContent` (partie **pure** uniquement)
- [US-010] `utils/malImport.ts` — `parseMalXml` (partie **pure** uniquement)

> **~50 tests Vitest, zéro `any`, comportement vanilla reproduit fidèlement (bizarreries comprises).**

### 🔄 In Progress
- _(rien — handoff vers nouvelle conversation pour Phase 2)_

### 📝 To Do — Sprint 3 (Phase 2 : Store Pinia + composables services)
Ordre recommandé (voir `HANDOFF_PHASE2.md` pour le détail) :
- [US-011] Store Pinia `stores/anime.ts`
- [US-012] `useFirebaseAuth.ts`
- [US-013] `useFirestore.ts`
- [US-014] `usePersistence.ts`
- [US-015] `useJikanApi.ts` (probablement à scinder)
- [US-016] `useSync.ts`
- [US-017] `useRecommendations.ts` (probablement à scinder)
- [US-018] petits composables : `useEpisodeInfo`, `useToast`, `useTheme`, `useICS` (download), `useMalImport`

### 🗂️ Backlog (epics par phase)
- [PHASE-3] Router + layouts
- [PHASE-4] Composants atomiques ui/
- [PHASE-5] Pages
- [PHASE-6] Modals & sheets
- [PHASE-7] Branchement final + nettoyage

---

## Reports / dette ouverte (à traiter dans une phase ultérieure)
- **`useICS.ts` (Phase 2)** : le **téléchargement** ICS (Blob, lien `<a>`, cas iOS via data-URI) + le toast « rien à exporter » NE sont PAS encore portés. `buildICSContent` (pur) existe ; le composable l'appellera puis gérera le download.
- **`useMalImport.ts` (Phase 2)** : la partie impure de `mal-import.js` (input fichier, FileReader, `addAnimeSilent`, toast) reste à porter. `parseMalXml` (pur) existe déjà.
- **Bug studios inerte (Décision E)** : `scorePool` lit `item.studios` (pluriel) que `normalize` ne produit jamais → scoring studio inerte. **Reproduit tel quel.** Réparation = US métier dédiée APRÈS la migration.
- **Dette UX boot (Phase 4)** : prévoir un `LoadingOverlay` piloté par état réactif au démarrage (auth + load Firestore) pour éviter le flash blanc.
- **`jsdom` pin à vérifier** : pin `^29.1.1` accepté en US-010 ; confirmer qu'un `npm install` propre le résout.

---

## Règles de tenue du backlog
- **Une seule US `In Progress` à la fois.** On ne démarre pas la suivante sans `MERGE`.
- Quand une US passe `MERGE` → Claude la déplace en `Done` et remonte la suivante en `In Progress`.
- Si une US dépasse 3 fichiers → Claude la **scinde** et ajoute les sous-US ici.
- Les **anti-patterns récurrents** détectés en review sont consignés dans `ANTIPATTERNS.md` et réinjectés dans les US suivantes.
- **Zéro confiance** : chaque US livrée doit fournir le **code (contenu ou diff)** ET la **sortie brute des commandes** (`vitest`, `vue-tsc`, `eslint`). Une sortie verte sans code ne suffit pas à un MERGE.
