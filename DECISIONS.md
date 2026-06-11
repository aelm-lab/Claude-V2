# DECISIONS.md — Journal des décisions d'architecture

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** capturer le contexte « mou » pour qu'une nouvelle instance de Claude
> ne réinvente pas les décisions passées.

---

## Format
`[ID] Décision — Raison — Impact`

---

## Décisions de session 1 (Phase 0 + Phase 1)

### Scaffold / outillage

- **[DEC-01] US-001b absorbée dans US-001.** App.vue + index.html ne rentraient pas dans la limite 3 fichiers. Gemini a tout fait en une passe (5 fichiers, signalé et autorisé). → US-001b n'existe plus.

- **[DEC-02] ESLint = flat config + `@vue/eslint-config-typescript@^14`.** Le v13 est l'ancien format eslintrc, incompatible avec la flat config. Config retenue :
```js
  import pluginVue from 'eslint-plugin-vue';
  import { defineConfigWithVueTs, vueTsConfigs } from '@vue/eslint-config-typescript';
  export default defineConfigWithVueTs(
    pluginVue.configs['flat/essential'],
    vueTsConfigs.recommended,
    { rules: { '@typescript-eslint/no-explicit-any': 'error' } },
  );
```
  → Impact : `no-explicit-any` est en erreur (prouvé en US-002).

- **[DEC-03] `tsconfig.node.json` séparé** pour isoler `vite.config.ts` (`composite: true`, `types: ['node']`).

- **[DEC-04] `npx <outil>` ≡ `npm run <script>`.** L'environnement Gemini bloque npm direct. Accepté.

- **[DEC-05] Conteneur Gemini tourne nativement en UTC.** Pas besoin de préfixe `TZ=UTC`.

- **[DEC-06] `jsdom` installé en devDep (US-010)** pour `DOMParser` dans les tests malImport. Déclaré par fichier via `// @vitest-environment jsdom`.

### Typage

- **[DEC-07] DTO de plomberie = types locaux, pas dans le contrat.** `RawAnime` et `MalImportEntry` sont locaux non exportés. `MalImportResult` est exporté mais reste près de sa fonction.

- **[DEC-08] Fixtures de test typées via helper `Partial` ou factory complet.** Interdit : `as any`, `as unknown as T`.

- **[DEC-09] US-008-types : extension du contrat (ajouts only, rien renommé).** Ajoutés : `RecSignalKind`, `RecSignal.kind?`, `HistoryItem.completedAt?`/`recencyBucket?`, et sur `AnimeEntry` : `studios?`, `popularityScore?`, `_relevanceScore?`, `_presetScore?`.

### Fidélité fonctionnelle (rec-engine)

- **[DEC-10] Branches mortes simplifiées.** Après `normalize`, `genres`/`themes`/`studios` sont toujours `string[]` → branche `.name` injoignable. Simplifiée en `const display = x;`. Comportement identique.

- **[DEC-11] Bug `item.studios` reproduit tel quel.** `scorePool` lit `item.studios` (pluriel) que `normalize` ne produit jamais (il produit `studio` singulier) → scoring studio inerte. On ne corrige pas. Réparation = P8-01 / US-120.

- **[DEC-12] `decayMultiplier = 0.2` conservé** dans `buildTasteProfile`. Poids trait = `2.0 × 1.0 × 0.2 = 0.4` pour heart+recent sans completedAt.

- **[DEC-13] `priority` du tri des signaux typé `Record<RecSignalKind, number>` avec `score: 0`.** Correction de typage, pas de comportement.

- **[DEC-14] `extractBecauseYouWatched` : param `profile` inutilisé → préfixé `_profile`.** Signature publique préservée.

### Découpage ICS / MAL

- **[DEC-15] `generateICSFile` scindé.** Génération de texte (`buildICSContent`, pure) dans `utils/ics.ts`. Téléchargement + toast → `useICS.ts`.

- **[DEC-16] `openMalImport` reporté.** `parseMalXml` (pur) dans `utils/malImport.ts`. Partie impure (FileReader, `addAnimeSilent`, toast) → `useMalImport.ts`.

### UX

- **[DEC-17] Dette UX boot.** `LoadingOverlay` piloté par état réactif à faire en Phase 4.

---

## Décisions de session 2 (Phase 2)

### Store

- **[DEC-18] Upsert du store : garder `if ('state' in input)`.** Ne jamais recalculer `state` inconditionnellement en branche merge, sinon clobber de l'état existant. Reproduit le `Object.assign` partiel du vanilla. La spec de Claude était fautive, corrigée par Gemini (2ᵉ preuve que la règle zéro-confiance paie).

- **[DEC-19] `needsBroadcastSync` dans `usePersistence`.** Le forEach qui force `status:'Continuing'` mute les items réactifs → déclenche le watchDebounced → sauvegarde différée 1 000 ms. Dans le vanilla, cette mutation ne déclenchait pas `store:changed`. Déviation mineure plus correcte. Conservé tel quel.

### Architecture composables

- **[DEC-20] bg worker : `fetchAnimeRelations` retourne `[]` aussi bien sur « pas en cache » que sur « anime sans relations ».** `shouldRescore` levé plus conservateur que le vanilla. Idempotent, sans effet de bord.

- **[DEC-21] `buildRelationMemory` provient de `recs.js`, pas de `rec-engine.js`.** Dans la migration, elle appartient à `useRecommendations` (US-017). Dans `useSync`, stubbée comme `_buildRelationMemory`. Spec de Claude fautive (3ᵉ preuve).

- **[DEC-22] `fetchTopFinishedAnime` inline dans `useRecommendations.fetchRecPool('library')`.** Mentionné dans TYPES_CONTRACT.md §7 comme méthode de useJikanApi mais absent de `api.js`. Fetch `/anime?min_score=7.5…` inline, fidèle au vanilla. Migration vers useJikanApi = P8-04 / US-123.

### Thème / styles

- **[DEC-23] `useTheme` : `useDark()` de `@vueuse/core` applique la classe `dark` sur `<html>`.** Le vanilla l'appliquait sur `<body>`. Divergence résolue en US-104 (CSS aligné sur `html.dark`).

### Import MAL

- **[DEC-24] `MalImportResult` expose `imported` (tableau d'entrées), pas `entries`.** Erreur dans la spec US-018b, corrigée par Gemini qui a lu le vrai type. 4ᵉ preuve que vue-tsc est indispensable.

---

## Décisions d'orchestration Phase 2→3

- **[DEC-25] Option 2 pour l'orchestration sync.** Les stubs `_syncAnimeUpdates`/`_startBackgroundRelationFetch` dans `usePersistence` restent des no-ops permanents. `App.vue` séquence `loadFromDatabase()` puis `syncAnimeUpdates()` directement. Pas de dépendance circulaire. *(Trou révélé en session 6 : l'orchestration des recos avait été oubliée → corrigé DEC-50.)*

- **[DEC-26] `watch → saveToDatabase` dans `usePersistence`, pas dans le store.** Le store reste sans I/O. `usePersistence` porte le `watchDebounced` avec flag `suppressPersist`. Frontière propre.

---

## Décisions de session 3 (Phase 3 + Phase 4)

### Router

- **[DEC-27] Double bloc `<script>` + `<script setup>` pour exporter un Symbol depuis App.vue.** En Vue 3.3+, `<script setup>` interdit les exports nommés. `isBootingKey: InjectionKey<Ref<boolean>>` est exporté depuis un bloc `<script lang="ts">` standard, la logique setup reste dans `<script setup lang="ts">`. Pattern validé par vue-tsc + vite build.

- **[DEC-28] Guard auth utilise `auth` singleton (Option A).** `auth` importé depuis `@/lib/firebase` dans le guard `beforeEach`. `useFirebaseAuth()` non appelé dans le guard (hors contexte `setup`). `await auth.authStateReady()` obligatoire avant toute lecture de `auth.currentUser` pour éviter la race au premier load.

- **[DEC-29] Placeholders inline dans router/index.ts.** Les routes pointent vers `defineComponent` inline jusqu'à Phase 5. Évite les `TS2307` sur des fichiers inexistants.

- **[DEC-30] `lastCalendarView/RadarView/VaultView` reporté en Phase 4 (PrimaryNav).** Ces clés vivent dans `nav.js`, pas dans le router. Mettre des lectures localStorage dans les guards = concern qui fuit. Route `/` → `/week` statique jusqu'à Phase 4.

### Boot / layouts

- **[DEC-31] `isBooting` fourni via `provide/inject`, pas en prop.** Évite le prop-drilling jusqu'à `LoadingOverlay`. `isBootingKey: InjectionKey<Ref<boolean>>` exporté depuis `App.vue`. `LoadingOverlay` injecte avec fallback `ref(false)` obligatoire.

- **[DEC-32] `syncAnimeUpdates()` et `startBackgroundRelationFetch()` fire-and-forget dans `App.vue onMounted`.** *Ajusté en session 6 (DEC-50) : `syncAnimeUpdates` passe en `await` car le re-score en dépend ; seul `startBackgroundRelationFetch` reste fire-and-forget.*

- **[DEC-33] `ToastNotification` dans `AppLayout` uniquement (routes auth).** Si toast sur `/login` requis un jour → migration vers `App.vue` = dette.

### Composants atomiques

- **[DEC-34] Lazy-load image via `<img style="display:none" @load @error>`.** Remplace `new Image()` vanilla. `imgState` initialisé à `'loading'` même si `cover_url` est null → `:class` traite `!cover_url` explicitement comme `card-fallback-bg` (bug US-025 corrigé).

- **[DEC-35] `SeasonNudgeCard` dismiss via `<Transition @after-leave>`.** Remplace `addEventListener('transitionend')`. Zéro `setTimeout`.

- **[DEC-36] `ChipsStrip` : chip 'all' émet `null` (pas la string 'all').** `RecPreset` ne contient pas `'all'`. `null` = pas de preset actif.

- **[DEC-37] `WeekAnimeItem` reçoit `info: AnimeEpisodeInfo` en prop.** Le parent calcule, le composant ne recalcule pas.

- **[DEC-38] `MonthDayCell` reçoit `animes: MonthAnimeItem[]` pré-filtrés en prop.** Composant dumb.

---

## Décisions de session 4 (Phases 5+6+7)

- **[DEC-39]** Substitution des placeholders router au fil des US de page (pas de passe dédiée).
- **[DEC-40]** Pattern stub `console.warn` uniforme en Phase 5, câblage batché en US-041.
- **[DEC-41]** `stores/ui.ts` pilote tous les overlays (modal, ep-override, recency). Remplace `activeWindowClickHandler` et `modal.style.display`.
- **[DEC-42]** `modalContext` : `libraryRec` prioritaire (dernière affectation vanilla = priorité en computed).
- **[DEC-43]** `removeAnimeWithUndo` simplifié en `store.removeAnime` (undo toast = dette US-121).
- **[DEC-44]** Prefetch covers relations abandonné (contournement DOM impératif) — fallback @error suffit (US-129).
- **[DEC-45]** `useEpisodeInfo` expose `getEpisodeInfo`/`getStatus`/`checkIsOnHiatus` ; `useICS` expose `downloadICS`. Contrat §7 corrigé a posteriori.
- **[DEC-46]** CalendarNavControls = composant connecté route-aware (pas props/emits) — duplication temporaire avec les pages (résorbée US-105).
- **[DEC-47]** `synopsis?: string` ajouté à AnimeEntry (US-048c).
- **[DEC-48]** ROADMAP.md remplace PHASE8_DEBT.md ; BACKLOG.md aminci (Kanban + règles uniquement).

---

## Décisions de session 6 (Audit croisé + EPIC-1)

- **[DEC-49] Audit croisé : Gemini > Claude Code.** Deux audits indépendants menés en parallèle. Claude Code a validé « migration saine » en ratant **4 bugs runtime**. Gemini (prompt UX+tech) les a tous trouvés, confirmés par preuve brute (grep ligne par ligne). **Leçon → règle R3 :** type-check + tests + build verts ≠ application fonctionnelle ; un audit doit lire la **chaîne d'orchestration runtime**, pas seulement les indicateurs. US-103 (fix tests timezone) supprimée : 66/66 réels, faux positif de l'audit A.

- **[DEC-50] Orchestration boot complète dans `App.vue` (US-102, P0).** `App.vue` importe `useRecommendations` et séquence : `loadFromDatabase()` → `await syncAnimeUpdates()` → `await buildRelationMemory()` → `reScorePool()` → `startBackgroundRelationFetch()` (fire-and-forget). `syncAnimeUpdates` passe en **`await`** (ajuste DEC-32) car le re-score travaille sur des données synchronisées. Les stubs morts `_buildRelationMemory`/`_reScorePool` sont supprimés de `useSync`. Couvert par le smoke test `App.spec.ts`.

- **[DEC-51] Pattern `*WithMeta` pour le throttle conditionnel (US-106, P1).** `fetchAnimeRelationsWithMeta` / `fetchAnimeRecommendationsWithMeta` retournent `{ data, fromNetwork }`. Le worker n'applique le throttle 1,1s **que si `fromNetwork === true`** (vrai appel réseau) — un cache hit IDB ne déclenche plus de sleep. Les méthodes publiques (`fetchAnimeRelations`/`fetchAnimeRecommendations`) délèguent et jettent le flag → signature `Promise<unknown[]>` inchangée, appelants existants intacts. Impact : 300 animes en cache = boot quasi instantané (avant : ~11 min).

- **[DEC-52] Hiatus = source unique computed 14j (US-107, P1).** Suppression du bloc d'écriture `anime.onHiatus` (seuil 21j) dans `useSync` — un champ que **personne ne lisait** (grep : 0 lecteur vivant). `isOnHiatus()` (14j, `episodeInfo.ts`) devient l'unique source, dérivée à l'affichage. La déclaration `const broadcast` est **conservée** (réutilisée pour la mise à jour JST en aval). Le type `onHiatus?` reste (retrait cosmétique = EPIC-3).

- **[DEC-53] `AGENTS.md` conservé et musclé (US-110).** **Réversion du plan initial** (US-101 prévoyait sa suppression comme « parasite »). `AGENTS.md` est en réalité le fichier lu automatiquement par Gemini AI Studio → il devient la **gouvernance permanente** : R-LIVRAISON 1-3, R-SCOPE 1-3, R-CODE 1-6, zéro-confiance. Les règles ne sont plus répétées dans chaque US.

- **[DEC-54] Filet de sécurité avant correctifs (US-109).** Décision PO : installer le filet **avant** de corriger les bugs. `.github/workflows/ci.yml` (vue-tsc + vitest + build sur push/PR) + `src/App.spec.ts` (smoke test d'orchestration boot). Le 3ᵉ test, volontairement **rouge**, encodait le contrat du bug P0 ; il est passé **vert sans modification** après US-102 → preuve que le filet fonctionne de bout en bout. Cette US a livré la CI/CD initialement planifiée comme US-110 dans EPIC-2.

- **[DEC-55] Deux nouveaux documents d'architecture.** `ARCHITECTURE_TECHNIQUE.md` (couches, modules, boot, flux, réseau) et `ARCHITECTURE_FONCTIONNELLE.md` (parcours utilisateur + pont fonctionnel↔technique). Référence centrale pour cadrer le périmètre réel d'une US avant de toucher une fonctionnalité.
