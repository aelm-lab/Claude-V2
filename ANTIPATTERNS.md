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

### TypeScript
- ❌ `any` (implicite ou explicite), `as any`, `@ts-ignore` → typer depuis TYPES_CONTRACT.md.
- ❌ Inventer une interface qui existe déjà dans TYPES_CONTRACT.md.
- ❌ Champs optionnels traités comme garantis (oublier `?.` ou les `null`).
- ❌ Fixtures de test via `as unknown as AnimeEntry` → factory `createAnime(Partial<AnimeEntry>)`.

### Gestion d'erreur
- ❌ `async` sans `try/catch` → chaque appel réseau/IO doit gérer l'échec.
- ❌ Avaler une erreur en silence (`catch {}`) sans log ni état.
- ❌ Oublier la gestion du 429 (rate limit Jikan) et du retry/backoff.
- ❌ Laisser remonter le `throw` de `handleFirestoreError` → attraper localement, exposer `error` réactif.

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

## Récidives détectées en cours de projet

- **[US-001]** ❌ Réintroduction de dépendances parasites non demandées + conservation du DOM vanilla dans `index.html`. → Règle : pas de déps hors liste, `<body>` doit être nu.
- **[US-001]** ❌ `vite.config.ts` : alias `@` pointé sur `.` au lieu de `./src`.
- **[US-002]** ⚠️ (erreur de spec Claude) : ESLint flat config doit utiliser `defineConfigWithVueTs` + `@vue/eslint-config-typescript@^14`.
- **[US-006a]** ❌ `as unknown as <Type>` pour les fixtures → helper `makeAnime(Partial<AnimeEntry>)`.
- **[US-011]** ❌ Install firebase hors périmètre US. La déps firebase appartient à US-012.
- **[US-016]** ❌ Import `buildRelationMemory` depuis `@/utils/recEngine` alors qu'elle est dans `recs.js` (useRecommendations). Spec de Claude fautive — cf. DEC-21.
- **[US-017a]** ❌ `void extractBecauseYouWatched` + `void saveToDatabase` pour silencer les unused → ne pas utiliser `void` sur des imports. Retirer l'import et le réintroduire dans la passe où il est utilisé.

---

## Règles process permanentes

- ✅ **Zéro confiance, y compris sur le code de Claude.** Tout snippet est faillible → prouver par l'exécution.
- ✅ **Sortie de commande = session terminale littérale.** Prompt `$` + commande + sortie réelle (même vide). `# Command completed successfully` ou tout résumé = review suspendue d'office sans exception à partir de Phase 3. Aucune tolérance supplémentaire.
- ✅ **Gemini n'a PAS accès à la Knowledge.** Chaque US est 100 % autoportante.
- ✅ **Fixtures de test typées** via helper `Partial` ou factory complet — interdit `as any`/`as unknown as T`.
- ✅ `npx <outil>` accepté comme équivalent de `npm run <script>`.
- ✅ **`eslint-disable-next-line` ne silencieux PAS TypeScript.** `TS6133` (unused variable) est un diagnostic compilateur — seul le retrait de la variable ou le préfixe `_` le résout. Ne jamais injecter un disable ESLint pour corriger une erreur `vue-tsc`. Solution : retirer l'import inutilisé, le réintroduire dans l'US où il est utilisé.
- ✅ **Livraison = code brut intégral + sorties brutes, jamais un résumé.** Un récap en prose du code ne vaut pas livraison. Si l'US crée/modifie des fichiers, coller leur contenu complet du premier au dernier caractère.
