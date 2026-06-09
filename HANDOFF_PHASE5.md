# HANDOFF_PHASE5.md — Reprise de projet pour une nouvelle conversation

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** permettre à une nouvelle conversation Claude de reprendre exactement
> où la Phase 4 s'est arrêtée, sans perte de contexte.

---

## 1. Où on en est

- **Phase 0 (scaffold) : ✅ close.**
- **Phase 1 (logique pure) : ✅ close à 100 %.** ~50 tests Vitest, zéro any.
- **Phase 2 (Store Pinia + composables services) : ✅ close à 100 %.**
- **Phase 3 (Router + layouts) : ✅ close à 100 %.**
- **Phase 4 (Composants atomiques) : ✅ close à 100 %.**
- **Prochaine étape : Phase 5 — Pages.**

### Fichiers Phase 3 livrés et mergés
```
src/router/index.ts
src/main.ts            (ajout app.use(router))
src/App.vue            (boot orchestration + provide isBootingKey + switch layout)
src/components/layout/AppLayout.vue
src/components/layout/AuthLayout.vue
src/components/pages/LoginPage.vue
```

### Fichiers Phase 4 livrés et mergés
```
src/components/ui/EmptyState.vue
src/components/ui/SkeletonCard.vue
src/components/ui/LoadingOverlay.vue
src/components/ui/ToastNotification.vue
src/components/ui/SyncIndicator.vue
src/components/ui/ChipsStrip.vue
src/components/ui/SeasonNudgeCard.vue
src/components/ui/AnimeCard.vue
src/components/ui/RecCard.vue
src/components/ui/WeekAnimeItem.vue
src/components/ui/WeekSuggestionCard.vue
src/components/ui/MonthDayCell.vue
```

---

## 2. Prompt de démarrage pour la nouvelle conversation

> Copie ce bloc comme premier message :

```
On reprend la migration Aanime (vanilla JS → Vue 3 + TS + Pinia).
Lis d'abord toute la Knowledge : CLAUDE.md, AUDIT.md, PLAN_MIGRATION.md,
BACKLOG.md, TYPES_CONTRACT.md, ANTIPATTERNS.md, DECISIONS.md,
HANDOFF_PHASE5.md, PHASE8_DEBT.md.

État : Phases 0, 1, 2, 3 et 4 closes.
- Phase 3 : router (guards auth/guest, 11 routes, placeholders inline), App.vue
  (boot DEC-25, provide isBootingKey DEC-31), AppLayout, AuthLayout, LoginPage.
- Phase 4 : 12 composants atomiques livrés (EmptyState, SkeletonCard,
  LoadingOverlay, ToastNotification, SyncIndicator, ChipsStrip, SeasonNudgeCard,
  AnimeCard, RecCard, WeekAnimeItem, WeekSuggestionCard, MonthDayCell).

On démarre la Phase 5 (Pages).
Confirme la lecture, affiche le Kanban depuis BACKLOG.md, puis propose le
cadrage de US-027 (substitution placeholders router + premières pages).
NE rédige pas encore l'US : colle-moi d'abord les sources vanilla des vues
à migrer pour que je voie la structure d'init actuelle.
Applique toutes les règles process de DECISIONS.md et ANTIPATTERNS.md.
```

---

## 3. Règles process NON-NÉGOCIABLES (récidives Phase 2-3-4)

1. **Zéro confiance** — code brut intégral + sortie terminale brute. Résumé = review suspendue.
2. **Sortie de commande = session terminale littérale** — prompt `$` + commande + sortie réelle même vide. Aucune exception.
3. **Livraison = contenu intégral** — `show all diff` sans contenu = review suspendue.
4. **Max 3 fichiers par US** — sinon scinder + prévenir le PO.
5. **Une seule US In Progress** — pas de MERGE, pas de suite.
6. **Fixtures de test** typées via factory — jamais `as any`.
7. `npx <outil>` ≡ `npm run <script>`.
8. **`eslint-disable-next-line` ne corrige pas TS6133** — retirer l'import inutilisé.
9. PO est non-dev — ne pas lui faire trancher une question technique seul.

---

## 4. Architecture Phase 5 — points d'attention

### Substitution des placeholders (à faire en US-027 ou début Phase 5)
Le router pointe encore sur des `defineComponent` inline pour toutes les routes.
Phase 5 substituera les placeholders par les vrais composants au fur et à mesure.
Stratégie recommandée : substituer dans `router/index.ts` à chaque US de page, pas en une seule passe.

### Pattern de pages
Chaque page Phase 5 suit ce pattern :
```ts
// Pas de fetch direct → composables
// Pas d'accès DOM → ref / v-if / v-for
// Scroll infini → useIntersectionObserver (@vueuse/core)
// Scroll position → <KeepAlive> + onActivated/onDeactivated
// Swipe → useSwipe (@vueuse/core)
```

### CalendarWeekPage (US-028) — complexité principale
- `getAnimeEpisodeInfo(anime, targetDate)` appelé pour chaque anime avec la date cible de la semaine affichée.
- Slot-fill suggestions : `useRecommendations().getSlotFillSuggestions(list)` → `WeekSuggestionCard`.
- Navigation date : prev/today/next → `useAnimeStore().setDate()`.

### CalendarMonthPage (US-029)
- Grille 42 cellules (6 semaines × 7 jours).
- Filtrage par jour + état → calcul dans la page, pas dans `MonthDayCell`.
- `MonthAnimeItem = { anime, info }` construit dans la page.

### DiscoverExplorePage (US-030) — infinite scroll
- `useIntersectionObserver` sur un sentinel div en bas de liste.
- `getNextBatch('incoming', size)` au déclenchement.
- `ChipsStrip` v-model → `applyPreset` + toast.
- Animation promotion (classe `rec-card--promoted` + setTimeout 1200ms) → reproduire fidèlement.

### Scroll position (toutes les pages)
- `<KeepAlive>` dans `AppLayout` autour de `<router-view>`.
- `onActivated` → `window.scrollTo(0, savedPos)`.
- `onDeactivated` → sauvegarder `window.scrollY` dans un `ref` local (pas localStorage — dette P8-08).

---

## 5. Sources vanilla à réclamer (Phase 5)

| US | Source à demander |
|---|---|
| US-028 CalendarWeekPage | `src/views/week.js` complet |
| US-029 CalendarMonthPage | `src/views/month.js` complet |
| US-030 DiscoverExplorePage | `src/views/incoming.js` complet |
| US-031 DiscoverSeasonPage | `src/views/season.js` complet |
| US-032 DiscoverComingUpPage | partie `incoming:comingup` de `incoming.js` |
| US-033 LibraryExplorePage | `src/views/library.js` partie explore |
| US-034 LibraryPlanToWatchPage | `src/views/library.js` partie plan-to-watch |
| US-035 LibraryCompletedPage | `src/views/library.js` partie completed |

---

## 6. État des stubs ouverts

| Stub | Fichier | État |
|---|---|---|
| `_syncAnimeUpdates` | usePersistence.ts | No-op permanent — App.vue orchestre |
| `_startBackgroundRelationFetch` | usePersistence.ts | No-op permanent — App.vue orchestre |
| `_buildRelationMemory` | useSync.ts | No-op — câblé via useRecommendations en App.vue |
| `_reScorePool` | useSync.ts | No-op — câblé via useRecommendations en App.vue |
| Commentaire résiduel | useSync.ts:177 | Texte obsolète, code correct — nettoyage Phase 7 |
| Placeholders router | router/index.ts | inline `defineComponent` — substitution Phase 5 |

---

## 7. Dette ouverte (détail dans PHASE8_DEBT.md)

- P8-01 : Bug studios inerte (DEC-11)
- P8-02 : Auto-vault silencieux → toast informatif
- P8-03 : window.prompt() re-saisie email
- P8-04 : fetchTopFinishedAnime inline → migrer dans useJikanApi (DEC-22)
- P8-05 : Mapping MAL Dropped→vault discutable
- P8-08 : Clés localStorage incohérentes — harmonisation Phase 7
- P8-09 : Redirect post-login vers route d'origine
- P8-10 : SyncIndicator — couverture complète fetches Jikan
- DEC-23 : classe dark sur `<html>` vs `<body>` → surveiller Phase 7
