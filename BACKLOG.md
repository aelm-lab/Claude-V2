# BACKLOG.md — Kanban vivant + User Stories

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.

---

## 📋 Kanban — état actuel (fin de session 2, Phase 2 close à 100 %)

### ✅ Done — Phase 0 + Phase 1
- [US-001] Scaffold Vue 3 + TS + Vite (US-001b absorbée)
- [US-002] tsconfig strict + ESLint flat config + arborescence (9 dossiers)
- [US-003] Types de base (anime.ts, recs.ts, persistence.ts)
- [US-004] utils/jst.ts (+ test) — conversion JST→local
- [US-005] utils/normalize.ts (+ test) — normalizeAnime
- [US-006a] utils/episodeInfo.ts (+ test) — getAnimeEpisodeInfo / getCardStatus / isOnHiatus
- [US-006b] utils/helpers.ts (+ test) — escapeHTML / getWeekNumber / fetchWithRetry
- [US-007] utils/idb.ts — wrapper IndexedDB
- [US-008-types] Extension du contrat de types pour le moteur de recs
- [US-008a] utils/recEngine.ts passe 1 — RelationMemory, buildTasteProfile, assignBadge, extractBecauseYouWatched, buildNextBatch
- [US-008b] utils/recEngine.ts passe 2 — scorePool + helpers de matching
- [US-009] utils/ics.ts — buildICSContent (partie pure uniquement)
- [US-010] utils/malImport.ts — parseMalXml (partie pure uniquement)

### ✅ Done — Phase 2
- [US-011] Store Pinia stores/anime.ts — upsert, auto-vault, transitions state
- [US-012] lib/firebase.ts + useFirebaseAuth.ts — auth email link, state-only
- [US-013] useFirestore.ts — Firestore brut, sanitize, handleFirestoreError
- [US-014] usePersistence.ts — watchDebounced, staleDataWarning, fallback localStorage, transitions load
- [US-015a] useJikanApi.ts passe 1 — searchAnime, fetchAnimeById, fetchCurrentSeason, fetchUpcomingSeason
- [US-015b] useJikanApi.ts passe 2 — fetchAnimeRelations, fetchAnimeRecommendations (cache IDB)
- [US-016] useSync.ts — batch 2/2s, throttle 1,1s, isSyncing, clearSyncTimestamp
- [US-017a] useRecommendations.ts passe 1 — pools, scoring, buildRelationMemory, fetchRecPool, getNextBatch
- [US-017b] useRecommendations.ts passe 2 — BYW, slot-fill, saveRec, saveAsCompleted
- [US-018a] useEpisodeInfo.ts + useToast.ts + useTheme.ts
- [US-018b] useICS.ts (download Blob + iOS) + useMalImport.ts (FileReader + suppressPersist)
- [US-018c] Câblage stubs — remplacement _showToast par useToast dans usePersistence + useSync

> **~50 tests Vitest (Phase 1), zéro any, tous composables Phase 2 livrés.**

### 🔄 In Progress
- _(rien — handoff vers Phase 3)_

### 📝 To Do — Sprint 4 (Phase 3 : Router + layouts)
- [US-019] Router Vue Router 4 (routes, guards auth, redirections)
- [US-020] App.vue + AppLayout.vue coquille (header placeholder, nav placeholder, router-view)
- [US-021] AuthLayout.vue + LoginPage.vue coquille (câblage useFirebaseAuth)

### 🗂️ Backlog (epics par phase)
- [PHASE-4] Composants atomiques ui/
- [PHASE-5] Pages
- [PHASE-6] Modals & sheets
- [PHASE-7] Branchement final + nettoyage
- [PHASE-8] Post-migration : dette technique + features → voir PHASE8_DEBT.md

---

## Reports / dette ouverte
Voir **PHASE8_DEBT.md** pour le détail complet (P8-01 à P8-05).

Résumé rapide :
- Bug studios inerte (DEC-11) — scorePool lit item.studios que normalize ne produit pas.
- Auto-vault silencieux (P8-02) — toast informatif au premier ajout d'une série terminée.
- window.prompt() re-saisie email (P8-03) — remplacer par UI propre en LoginPage.
- fetchTopFinishedAnime inline (DEC-22 / P8-04) — à migrer dans useJikanApi.
- Mapping MAL Dropped→vault discutable (P8-05).
- Commentaire résiduel useSync.ts:177 — nettoyage Phase 7.
- DEC-23 : classe dark sur html vs body — surveiller Phase 7.

---

## Règles de tenue du backlog
- **Une seule US In Progress à la fois.** Pas de MERGE, pas de suite.
- Quand une US passe MERGE → la déplacer en Done, remonter la suivante.
- Si une US dépasse 3 fichiers → scinder + prévenir le PO.
- **Zéro confiance** : chaque US livrée fournit code brut intégral + sorties terminales brutes.
  `# Command completed successfully` n'est pas une sortie brute → review suspendue.
