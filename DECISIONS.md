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

- **[DEC-11] Bug `item.studios` reproduit tel quel.** `scorePool` lit `item.studios` (pluriel) que `normalize` ne produit jamais (il produit `studio` singulier) → scoring studio inerte. On ne corrige pas. Réparation = P8-01.

- **[DEC-12] `decayMultiplier = 0.2` conservé** dans `buildTasteProfile`. Poids trait = `2.0 × 1.0 × 0.2 = 0.4` pour heart+recent sans completedAt.

- **[DEC-13] `priority` du tri des signaux typé `Record<RecSignalKind, number>` avec `score: 0`.** Correction de typage, pas de comportement.

- **[DEC-14] `extractBecauseYouWatched` : param `profile` inutilisé → préfixé `_profile`.** Signature publique préservée.

### Découpage ICS / MAL

- **[DEC-15] `generateICSFile` scindé.** Génération de texte (`buildICSContent`, pure) dans `utils/ics.ts`. Téléchargement + toast → `useICS.ts` (Phase 2 / US-018b).

- **[DEC-16] `openMalImport` reporté.** `parseMalXml` (pur) dans `utils/malImport.ts`. Partie impure (FileReader, `addAnimeSilent`, toast) → `useMalImport.ts` (US-018b).

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

- **[DEC-22] `fetchTopFinishedAnime` inline dans `useRecommendations.fetchRecPool('library')`.** Mentionné dans TYPES_CONTRACT.md §7 comme méthode de useJikanApi mais absent de `api.js`. Fetch `/anime?min_score=7.5…` inline, fidèle au vanilla. Migration vers useJikanApi = P8-04.

### Thème / styles

- **[DEC-23] `useTheme` : `useDark()` de `@vueuse/core` applique la classe `dark` sur `<html>`.** Le vanilla l'appliquait sur `<body>`. Divergence mineure. Surveiller les sélecteurs CSS en Phase 7.

### Import MAL

- **[DEC-24] `MalImportResult` expose `imported` (tableau d'entrées), pas `entries`.** Erreur dans la spec US-018b, corrigée par Gemini qui a lu le vrai type. 4ᵉ preuve que vue-tsc est indispensable.

---

## Décisions d'orchestration Phase 2→3

- **[DEC-25] Option 2 pour l'orchestration sync.** Les stubs `_syncAnimeUpdates`/`_startBackgroundRelationFetch` dans `usePersistence` restent des no-ops permanents. `App.vue` séquence `loadFromDatabase()` puis `syncAnimeUpdates()` directement. Pas de dépendance circulaire.

- **[DEC-26] `watch → saveToDatabase` dans `usePersistence`, pas dans le store.** Le store reste sans I/O. `usePersistence` porte le `watchDebounced` avec flag `suppressPersist`. Frontière propre.

---

## Décisions de session 3 (Phase 3 + Phase 4)

### Router

- **[DEC-27] Double bloc `<script>` + `<script setup>` pour exporter un Symbol depuis App.vue.** En Vue 3.3+, `<script setup>` interdit les exports nommés. `isBootingKey: InjectionKey<Ref<boolean>>` est exporté depuis un bloc `<script lang="ts">` standard, la logique setup reste dans `<script setup lang="ts">`. Pattern validé par vue-tsc + vite build.

- **[DEC-28] Guard auth utilise `auth` singleton (Option A).** `auth` importé depuis `@/lib/firebase` dans le guard `beforeEach`. `useFirebaseAuth()` non appelé dans le guard (hors contexte `setup`). `await auth.authStateReady()` obligatoire avant toute lecture de `auth.currentUser` pour éviter la race au premier load.

- **[DEC-29] Placeholders inline dans router/index.ts.** Les routes pointent vers `defineComponent` inline jusqu'à Phase 5. Évite les `TS2307` sur des fichiers inexistants. Substitution de tous les placeholders en une passe Phase 5.

- **[DEC-30] `lastCalendarView/RadarView/VaultView` reporté en Phase 4 (PrimaryNav).** Ces clés vivent dans `nav.js`, pas dans le router. Mettre des lectures localStorage dans les guards = concern qui fuit. Route `/` → `/week` statique jusqu'à Phase 4.

### Boot / layouts

- **[DEC-31] `isBooting` fourni via `provide/inject`, pas en prop.** Évite le prop-drilling jusqu'à `LoadingOverlay`. `isBootingKey: InjectionKey<Ref<boolean>>` exporté depuis `App.vue`. `LoadingOverlay` injecte avec fallback `ref(false)` obligatoire.

- **[DEC-32] `syncAnimeUpdates()` et `startBackgroundRelationFetch()` fire-and-forget dans `App.vue onMounted`.** Ni `await` ni `void` — appel direct. Le commentaire `// fire-and-forget` documente l'intention. Conforme DEC-25.

- **[DEC-33] `ToastNotification` dans `AppLayout` uniquement (routes auth).** Les toasts de boot se déclenchent après `loadFromDatabase`, quand l'utilisateur est authentifié et `AppLayout` déjà monté. Si toast sur `/login` requis un jour → migration vers `App.vue` = dette Phase 7.

### Composants atomiques

- **[DEC-34] Lazy-load image via `<img style="display:none" @load @error>`.** Remplace `new Image()` vanilla. Zéro DOM direct, comportement identique. `imgState` initialisé à `'loading'` même si `cover_url` est null — passage à `'error'` impossible sans `<img>` monté → `:class` doit traiter `!cover_url` explicitement comme `card-fallback-bg` (bug initial US-025, corrigé).

- **[DEC-35] `SeasonNudgeCard` dismiss via `<Transition @after-leave>`.** Remplace `card.addEventListener('transitionend')` vanilla. Le parent écrit localStorage après réception de l'emit `dismiss`. Zéro `setTimeout`.

- **[DEC-36] `ChipsStrip` : chip 'all' émet `null` (pas la string 'all').** `RecPreset` ne contient pas `'all'`. Le parent reçoit `RecPreset | null`, `null` = pas de preset actif.

- **[DEC-37] `WeekAnimeItem` reçoit `info: AnimeEpisodeInfo` en prop.** Le parent (CalendarWeekPage, Phase 5) calcule `getAnimeEpisodeInfo(anime, targetDate)` et passe le résultat. Le composant ne recalcule pas.

- **[DEC-38] `MonthDayCell` reçoit `animes: MonthAnimeItem[]` pré-filtrés en prop.** Le filtrage par jour + état appartient à `CalendarMonthPage`. Composant dumb.
