# ANTIPATTERNS.md — Liste vivante des pièges Gemini

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Rôle :** mémoire des erreurs récurrentes de Gemini. Claude **réinjecte** la liste pertinente dans la section « Anti-patterns à éviter » de chaque nouvelle US. À chaque review qui révèle une récidive, Claude **ajoute une ligne ici**.

---

## Anti-patterns connus dès le départ (issus de l'audit + de la 1ʳᵉ tentative ratée)

### Architecture
- ❌ Mettre de la logique métier ou un `fetch` **dans un composant `.vue`** → doit être dans un composable.
- ❌ Accéder au **DOM directement** (`document.getElementById`, `querySelector` sur la page, `appendChild`) dans du code Vue → utiliser `ref` / `v-if` / `v-for`.
  - *Nuance validée : `DOMParser` + `querySelector` pour parser une **string XML** (cf. `parseMalXml`) est PUR et autorisé — ça ne mute pas la page.*
- ❌ Stocker des handlers ou de l'état sur **`window`** (ex. `window.incomingScrollHandler`) → `onMounted`/`onUnmounted` + `@vueuse/core`.
- ❌ Recréer un **bus d'événements** avec `document.dispatchEvent` / `CustomEvent` → passer par le store Pinia ou des `emit`.
- ❌ Manipuler `document.body.classList` / injecter un bandeau dans le `<body>` → état réactif + composant.

### TypeScript
- ❌ `any` (implicite ou explicite), `as any`, `@ts-ignore` → typer correctement depuis `TYPES_CONTRACT.md`.
- ❌ **Inventer** une interface alors qu'elle existe déjà dans `TYPES_CONTRACT.md`.
- ❌ Champs optionnels traités comme garantis (oublier `?.` ou les `null`).

### Gestion d'erreur
- ❌ `async` sans `try/catch` → chaque appel réseau/IO doit gérer l'échec et exposer un état d'erreur.
- ❌ Avaler une erreur en silence (`catch {}`) sans log ni état → garde explicite ou état d'erreur exposé.
- ❌ Oublier la gestion du **429** (rate limit Jikan) et du retry/backoff.

### Périmètre & process
- ❌ Toucher des fichiers **non listés** dans l'US.
- ❌ Livrer **plus de 3 fichiers** pour une US.
- ❌ « Améliorer » au passage du code hors scope (refactor non demandé).
- ❌ Réécrire un fichier entier alors qu'une correction ciblée suffisait.

### Fidélité fonctionnelle
- ❌ Simplifier une règle métier subtile (calcul d'épisode, transitions de `state`, conversion JST) → reproduire **exactement** le comportement vanilla.
- ❌ Changer les clés `localStorage` / la forme du document Firestore sans US dédiée (casse la rétro-compat des données existantes).

---

## Récidives détectées en cours de projet

> _(format : `- [US-XXX] description du piège observé`)_

- **[US-001]** ❌ Réintroduction de **dépendances parasites** non demandées (`express`, `tsx`, `dotenv`, `firebase`, `@types/node`) + `script dev = tsx server.ts` + injection `define: process.env.GEMINI_API_KEY` (plomberie AI Studio) + conservation de **tout le DOM vanilla** dans `index.html` sous le `<div id="app">`. → **Règle réinjectée** : « pas de déps hors liste, pas de `define`/`loadEnv`/variable AI Studio, `<body>` doit être nu (`#app` + `main.ts`) ».
- **[US-001]** ❌ `vite.config.ts` : alias `@` pointé sur la racine `.` au lieu de `./src`. → vérifier l'alias en review.
- **[US-002]** ⚠️ (erreur de spec côté Claude, pas Gemini) : `eslint.config.js` doit utiliser `defineConfigWithVueTs(...)` + `vueTsConfigs` et **`@vue/eslint-config-typescript@^14`** (le v13 est eslintrc, incompatible flat config). → **Règle process** : tout snippet d'outillage fourni par Claude doit être **prouvé à l'exécution** par Gemini (coller la sortie réelle).
- **[US-006a]** ❌ `as unknown as <Type>` pour fabriquer des **fixtures de test** → contourne le typage comme `as any`. **Correctif standard : helper `make<Entity>(over: Partial<Entity>): Entity`** (ou un factory complet typé, cf. `createAnime` en US-009). **Réinjecté dans TOUTES les US à fichier `.spec` à partir d'ici.**

---

## Règles process permanentes (ajoutées en cours de session 1)

- ✅ **Zéro confiance, y compris sur le code de Claude.** Tout snippet d'outillage (config, commande) est réputé faillible → Gemini doit **prouver l'exécution**.
- ✅ **Preuve par sortie réelle.** Chaque US avec critère vérifiable par commande exige la **sortie brute** (`vitest`, `vue-tsc`, `eslint`), jamais un résumé « ça passe ».
- ✅ **Gemini n'a PAS accès à la Knowledge.** Chaque US doit être **100 % autoportante** : interfaces copiées en clair, comportement décrit en clair, zéro renvoi à un fichier `.md` ou à un « §X ».
- ✅ **Fixtures de test typées** via helper `Partial` ou factory complet — **interdit** `as any` / `as unknown as T`. À mettre dans la checklist de chaque US avec test.
- ✅ `npx <outil>` accepté comme équivalent de `npm run <script>` (l'environnement Gemini bloque `npm` direct).
