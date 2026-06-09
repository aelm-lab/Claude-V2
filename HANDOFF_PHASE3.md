markdown# HANDOFF_PHASE3.md — Reprise de projet pour une nouvelle conversation

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** permettre à une nouvelle conversation Claude de reprendre exactement
> où la Phase 2 s'est arrêtée, sans perte de contexte.

---

## 1. Où on en est

- **Phase 0 (scaffold) : ✅ close.**
- **Phase 1 (logique pure) : ✅ close à 100 %.** ~50 tests Vitest, zéro any.
- **Phase 2 (Store Pinia + composables services) : ✅ close à 100 %.**
- **Prochaine étape : Phase 3 — Router + layouts.**

### Fichiers Phase 2 livrés et mergés (dans l'ordre)
src/stores/anime.ts
src/lib/firebase.ts
src/composables/useFirebaseAuth.ts
src/composables/useFirestore.ts
src/composables/usePersistence.ts
src/composables/useJikanApi.ts
src/composables/useSync.ts
src/composables/useRecommendations.ts
src/composables/useEpisodeInfo.ts
src/composables/useToast.ts
src/composables/useTheme.ts
src/composables/useICS.ts
src/composables/useMalImport.ts
src/utils/helpers.ts  (ajout BASE_URL)

---

## 2. Prompt de démarrage pour la nouvelle conversation

> Copie ce bloc comme premier message :
On reprend la migration Aanime (vanilla JS → Vue 3 + TS + Pinia).
Lis d'abord toute la Knowledge : CLAUDE.md, AUDIT.md, PLAN_MIGRATION.md,
BACKLOG.md, TYPES_CONTRACT.md, ANTIPATTERNS.md, DECISIONS.md, HANDOFF_PHASE3.md,
PHASE8_DEBT.md.
État : Phases 0, 1 et 2 closes (tout src/utils/, src/stores/, src/lib/ et
src/composables/ livré, ~50 tests Vitest, zéro any).
On démarre la Phase 3 (Router + layouts).
Confirme la lecture, affiche le Kanban depuis BACKLOG.md, puis propose le
cadrage de US-019 (Router Vue Router 4 : routes, guards, redirections).
NE rédige pas encore l'US : colle-moi d'abord src/main.js (entry point vanilla)
pour que je voie la structure d'init actuelle.
Applique toutes les règles process de DECISIONS.md et ANTIPATTERNS.md.

---

## 3. Règles process NON-NÉGOCIABLES (récidives Phase 2)

1. **Zéro confiance** — code brut intégral + sortie terminale brute. Résumé = review suspendue.
2. **Sortie de commande = session terminale littérale** — prompt `$` + commande + sortie réelle même vide. `# Command completed successfully` = review suspendue sans exception, sans appel, sans tolérance.
3. **Max 3 fichiers par US** — sinon scinder + prévenir le PO.
4. **Une seule US In Progress** — pas de MERGE, pas de suite.
5. **Fixtures de test** typées via factory — jamais `as any`.
6. `npx <outil>` ≡ `npm run <script>`.
7. **`eslint-disable-next-line` ne corrige pas TS6133** — retirer l'import inutilisé.
8. PO est non-dev — ne pas lui faire trancher une question technique seul.

---

## 4. Sources vanilla à réclamer (Phase 3)

| US | Source à demander | Note |
|---|---|---|
| US-019 Router | `src/main.js` (entry point) | Voir comment Firebase auth est initialisée |
| US-019 Router | `src/ui/router.js` (routeur maison) | Voir les vues et la logique show/hide |
| US-020 App.vue + AppLayout | `src/main.js` | Structure d'init + mount |
| US-021 AuthLayout + LoginPage | Déjà disponible (`src/login.js` vu en US-012) | Réutiliser le contexte |

---

## 5. Points d'attention Phase 3

### Router (US-019)
- **Navigation guard** : remplace le check `localStorage.displayName` de `index.html`. Utiliser `useFirebaseAuth().isAuthenticated` + `auth.authStateReady()`.
- **`lastCalendarView` / `lastRadarView` / `lastVaultView`** (clés localStorage vues dans `nav.js`) : mémorisation du dernier sous-onglet. À porter dans le guard ou dans `App.vue` comme `initialRoute`. Décision à prendre au cadrage selon le code vanilla de `main.js`.
- **Route `/login`** : garde `guest` (redirige `/` si déjà auth).
- **Route `/`** : redirige vers `/week` (vue par défaut).
- **Toutes les routes protégées** : garde `auth` (redirige `/login` si non auth).

### App.vue + AppLayout (US-020)
- **Orchestration boot** dans `App.vue onMounted` :
```ts
  await loadFromDatabase();
  await syncAnimeUpdates();
  startBackgroundRelationFetch(); // fire-and-forget
```
  C'est ici que les stubs `_syncAnimeUpdates` de `usePersistence` sont contournés (Option 2, DEC-25).
- **`AppLayout.vue` Phase 3 = coquille** : `<slot>` ou `<router-view>`, header et nav en placeholder. Les vrais composants arrivent en Phase 4.
- **`LoadingOverlay`** (DEC-17 / P8 dette UX boot) : prévoir un `ref<boolean> isBooting` dans `App.vue`, posé à `false` après `loadFromDatabase`. `AppLayout` l'affichera.

### LoginPage (US-021)
- **`completeSignIn()`** appelé au mount de `LoginPage` si l'URL contient un lien Firebase.
- **Redirection post-login** : `router.replace('/')` après `completeSignIn()` réussi ou auth déjà présente.
- **`displayName`** calculé depuis `useFirebaseAuth().displayName` (computed, déjà exposé).

---

## 6. État des stubs ouverts

| Stub | Fichier | État |
|---|---|---|
| `_syncAnimeUpdates` | usePersistence.ts | No-op permanent — App.vue orchestre |
| `_startBackgroundRelationFetch` | usePersistence.ts | No-op permanent — App.vue orchestre |
| `_buildRelationMemory` | useSync.ts | No-op — câblé via useRecommendations en App.vue |
| `_reScorePool` | useSync.ts | No-op — câblé via useRecommendations en App.vue |
| Commentaire résiduel | useSync.ts:177 | Texte obsolète, code correct — nettoyage Phase 7 |

---

## 7. Dette ouverte (détail dans PHASE8_DEBT.md)

- P8-01 : Bug studios inerte (DEC-11)
- P8-02 : Auto-vault silencieux → toast informatif
- P8-03 : window.prompt() re-saisie email
- P8-04 : fetchTopFinishedAnime inline → migrer dans useJikanApi (DEC-22)
- P8-05 : Mapping MAL Dropped→vault discutable
- DEC-23 : classe dark sur `<html>` vs `<body>` → surveiller Phase 7
