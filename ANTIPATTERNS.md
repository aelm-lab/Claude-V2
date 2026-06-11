# ANTIPATTERNS.md — Liste vivante des pièges Gemini

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** mémoire des erreurs récurrentes. Claude réinjecte la liste dans
> chaque nouvelle US. À chaque review qui révèle une récidive, Claude ajoute une ligne.

---

## Anti-patterns architecturaux (issus de l'audit + tentative ratée)

### Architecture
- ❌ Mettre de la logique métier ou un `fetch` dans un composant `.vue` → composable.
- ❌ Accéder au DOM directement (`document.getElementById`, `querySelector`, `appendChild`) dans du code Vue → `ref` / `v-if` / `v-for`.
  - *Nuance validée : `DOMParser` + `querySelector` pour parser une string XML (parseMalXml) est pur et autorisé.*
  - *Nuance validée : `document.createElement('a')` + `click()` uniquement pour le download Blob dans `useICS` est l'exception acceptée.*
- ❌ Stocker des handlers ou de l'état sur `window` → `onMounted`/`onUnmounted` + `@vueuse/core`.
- ❌ Recréer un bus d'événements avec `document.dispatchEvent`/`CustomEvent` → store Pinia ou `emit`.
- ❌ Manipuler `document.body.classList` / injecter un bandeau dans le `<body>` → état réactif + composant.
- ❌ Mettre `initializeApp`/`getAuth` dans un composable → réinit à chaque appel. Singleton dans `lib/firebase.ts`.
- ❌ Brancher `onAuthStateChanged` dans la fonction `useXxx()` → listeners empilés. Au niveau module.
- ❌ Brancher `watchDebounced` dans le corps du composable sans flag → listeners empilés. Flag niveau module (`watchInitialized`).
- ❌ Appeler `useFirebaseAuth()` dans un guard Vue Router → hors contexte `setup`. Utiliser le singleton `auth` depuis `@/lib/firebase` + `await auth.authStateReady()`.
- ❌ Lire `auth.currentUser` avant `await auth.authStateReady()` → `currentUser` est `null` pendant l'init même si l'utilisateur est connecté.
- ❌ Importer un composant `.vue` de page/layout inexistant dans le router → `TS2307`. Utiliser des placeholders inline (`defineComponent`) jusqu'à Phase 5.

### TypeScript
- ❌ `any` (implicite ou explicite), `as any`, `@ts-ignore` → typer depuis TYPES_CONTRACT.md.
- ❌ Inventer une interface qui existe déjà dans TYPES_CONTRACT.md.
- ❌ Champs optionnels traités comme garantis (oublier `?.` ou les `null`).
- ❌ Fixtures de test via `as unknown as AnimeEntry` → factory `createAnime(Partial<AnimeEntry>)`.
- ❌ `inject(key)` sans fallback quand la clé est typée → TypeScript exige un fallback. Ex : `inject(isBootingKey, ref(false))`.
- ❌ Export nommé depuis `<script setup>` → impossible en Vue 3. Utiliser un double bloc `<script lang="ts">` + `<script setup lang="ts">` (DEC-27).

### Gestion d'erreur
- ❌ `async` sans `try/catch` → chaque appel réseau/IO doit gérer l'échec.
- ❌ Avaler une erreur en silence (`catch {}`) sans log ni état.
- ❌ Oublier la gestion du 429 (rate limit Jikan) et du retry/backoff.
- ❌ Laisser remonter le `throw` de `handleFirestoreError` → attraper localement, exposer `error` réactif.

### Composants
- ❌ `new Image()` pour le lazy-load image → `<img style="display:none" @load @error>` uniquement.
- ❌ `imgState` initialisé à `'loaded'` quand `cover_url` est null → toujours `'loading'`, passer à `'error'` via `@error`. Traiter `!cover_url` explicitement dans `:class` → `card-fallback-bg`.
- ❌ `setTimeout` pour les animations de dismiss → `<Transition @after-leave>` uniquement.
- ❌ Écrire `localStorage` dans un composant UI → la logique appartient au parent ou au composable.
- ❌ `router.push` au lieu de `router.replace` après auth → crée un retour arrière vers `/login`.
- ❌ `<form @submit.prevent>` → `@click` sur le bouton uniquement (règle cohérence Vue).
- ❌ `v-html` inconditionnel → toujours derrière `v-if="isHtml"` (XSS).

### Périmètre & process
- ❌ Toucher des fichiers non listés dans l'US.
- ❌ Livrer plus de 3 fichiers pour une US.
- ❌ « Améliorer » au passage du code hors scope.
- ❌ Réécrire un fichier entier quand une correction ciblée suffisait.

### Fidélité fonctionnelle
- ❌ Simplifier une règle métier subtile (calcul épisode, transitions de state, conversion JST).
- ❌ Changer les clés localStorage/Firestore sans US dédiée (casse la rétro-compat).
- ❌ Scorer ou filtrer dans `fetchUpcomingSeason` → appartient à useRecommendations.
- ❌ Utiliser `'backlog_recs_v1'` comme clé de cache brut → c'est la clé du pool scoré.
- ❌ Appeler `normalizeAnime` dans `syncAnimeUpdates` → le vanilla mutait les champs en place.

---

## Anti-patterns d'orchestration runtime (leçons de l'audit post-migration, session 6)

> Ces 4 bugs ont passé `vue-tsc` + tous les tests unitaires + le build **au vert**.
> Ils ne se voient QUE en lisant la chaîne d'appels runtime. D'où les règles R1/R2/R3.

- ❌ **Stub no-op câblé à vide en croyant que l'orchestration est faite ailleurs.** `useSync` appelait `_buildRelationMemory`/`_reScorePool` (stubs vides) ; `App.vue` n'importait même pas `useRecommendations`. Résultat : moteur de reco mort. → **Toujours vérifier que le VRAI appel existe quelque part** (grep le nom de la vraie fonction, pas du stub). [US-102]
- ❌ **Throttle / `setTimeout` / sleep inconditionnel après un appel qui peut taper le cache.** Le worker dormait 1,1 s même sur un cache hit IDB → ~11 min de boot. → **Conditionner l'attente à un flag `fromNetwork`** retourné par la fonction. Ne jamais throttler une lecture cache. [US-106]
- ❌ **Dupliquer une règle métier à deux endroits avec deux seuils différents.** Hiatus calculé à 14 j dans `episodeInfo.ts` (lu) ET à 21 j dans `useSync.ts` (écrit dans un champ jamais lu). → **Une seule source de vérité computed**, supprimer les calculs concurrents. [US-107]
- ❌ **Écrire un champ que personne ne lit.** `anime.onHiatus` était écrit par la synchro mais aucun composant/composable ne le lisait (grep : 0 lecteur vivant). → Code mort coûteux : supprimer l'écriture, dériver à l'affichage. [US-107]
- ❌ **Valider une migration sur le seul tooling vert.** `vue-tsc` + tests + build OK ≠ app fonctionnelle : 4 bugs runtime vivaient dessous. → **R3 : un audit lit le CODE**, pas seulement les indicateurs. [audit croisé]
- ❌ **Livrer une US d'orchestration/store/câblage sans test runtime.** → **R2 : smoke/unit test obligatoire** qui prouve la séquence d'appels (cf. `App.spec.ts`). [US-109]

---

## Récidives détectées en cours de projet

- **[US-001]** ❌ Réintroduction de dépendances parasites non demandées + conservation du DOM vanilla dans `index.html`. → Règle : pas de déps hors liste, `<body>` doit être nu.
- **[US-001]** ❌ `vite.config.ts` : alias `@` pointé sur `.` au lieu de `./src`.
- **[US-002]** ⚠️ (erreur de spec Claude) : ESLint flat config doit utiliser `defineConfigWithVueTs` + `@vue/eslint-config-typescript@^14`.
- **[US-006a]** ❌ `as unknown as <Type>` pour les fixtures → helper `makeAnime(Partial<AnimeEntry>)`.
- **[US-011]** ❌ Install firebase hors périmètre US. La déps firebase appartient à US-012.
- **[US-016]** ❌ Import `buildRelationMemory` depuis `@/utils/recEngine` alors qu'elle est dans `recs.js` (useRecommendations). Spec de Claude fautive — cf. DEC-21.
- **[US-017a]** ❌ `void extractBecauseYouWatched` + `void saveToDatabase` pour silencer les unused → ne pas utiliser `void` sur des imports. Retirer l'import inutilisé.
- **[US-025]** ❌ (erreur de spec Claude + Gemini) : `imgState` initialisé à `'loading'`, mais `card-fallback-bg` non appliqué quand `cover_url === null`. Fix : `'card-fallback-bg': imgState === 'error' || !anime.cover_url`.
- **[Audit session 6]** ⚠️ Gemini a parfois **paraphrasé** une sortie de commande (`grep` résumé en prose) au lieu de coller le terminal brut. → Exiger systématiquement la sortie littérale ; une paraphrase de preuve = vérification rejetée.
- **[Audit session 6]** ⚠️ Gemini a **renvoyé une US déjà mergée** au lieu d'exécuter la commande demandée. → Vérifier que la livraison répond exactement à la demande courante.

---

## Règles process permanentes (gravées dans AGENTS.md)

- ✅ **R1 — Triple preuve verte (CI).** `vue-tsc --noEmit` + `vitest run` + `build`, sorties brutes, pour tout MERGE. Rejouées par `.github/workflows/ci.yml` à chaque push.
- ✅ **R2 — Test obligatoire sur l'orchestration.** Toute US touchant le boot, un store, ou le câblage entre composables livre/maj un test runtime.
- ✅ **R3 — Un audit lit le code.** Pas seulement les indicateurs verts.
- ✅ **Zéro confiance, y compris sur le code de Claude.** Tout snippet est faillible → prouver par l'exécution.
- ✅ **Sortie de commande = session terminale littérale.** Prompt `$` + commande + sortie réelle (même vide). `# Command completed successfully` ou tout résumé = review suspendue d'office.
- ✅ **Livraison = contenu intégral des fichiers créés/modifiés.** `show all diff` ou récap sans contenu = review suspendue. (Exception très gros fichier : diff exact + `grep -c` prouvant 0 occurrence résiduelle.)
- ✅ **Gemini n'a PAS accès à la Knowledge.** Chaque US est 100 % autoportante. Sa gouvernance permanente vit dans `AGENTS.md` (racine, lu automatiquement).
- ✅ **Fixtures de test typées** via helper `Partial` ou factory complet — interdit `as any`/`as unknown as T`.
- ✅ `npx <outil>` accepté comme équivalent de `npm run <script>`.
- ✅ **`eslint-disable-next-line` ne silencie PAS TypeScript.** `TS6133` (unused) → retrait de la variable ou préfixe `_`.
- ✅ **Max 3 fichiers par US.** Si débordement → scinder + prévenir le PO (sauf suppression pure prouvée).
