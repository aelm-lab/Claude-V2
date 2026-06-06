# ANTIPATTERNS.md — Liste vivante des pièges Gemini

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Rôle :** mémoire des erreurs récurrentes de Gemini. Claude **réinjecte** la liste pertinente dans la section « Anti-patterns à éviter » de chaque nouvelle US. À chaque review qui révèle une récidive, Claude **ajoute une ligne ici**.

---

## Anti-patterns connus dès le départ (issus de l'audit + de la 1ʳᵉ tentative ratée)

### Architecture
- ❌ Mettre de la logique métier ou un `fetch` **dans un composant `.vue`** → doit être dans un composable.
- ❌ Accéder au **DOM directement** (`document.getElementById`, `querySelector`, `appendChild`) dans du code Vue → utiliser `ref` / `v-if` / `v-for`.
- ❌ Stocker des handlers ou de l'état sur **`window`** (ex. `window.incomingScrollHandler`) → `onMounted`/`onUnmounted` + `@vueuse/core`.
- ❌ Recréer un **bus d'événements** avec `document.dispatchEvent` / `CustomEvent` → passer par le store Pinia ou des `emit`.
- ❌ Manipuler `document.body.classList` / injecter un bandeau dans le `<body>` → état réactif + composant.

### TypeScript
- ❌ `any` (implicite ou explicite), `as any`, `@ts-ignore` → typer correctement depuis `TYPES_CONTRACT.md`.
- ❌ **Inventer** une interface alors qu'elle existe déjà dans `TYPES_CONTRACT.md`.
- ❌ Champs optionnels traités comme garantis (oublier `?.` ou les `null`).
- ❌ as unknown as <Type> pour fabriquer des fixtures de test → contourne le typage comme as any. Utiliser un helper make<Entity>(over: Partial<Entity>): Entity centralisé. Réinjecté dans toutes les US à fichier .spec à partir d'ici.




### Gestion d'erreur
- ❌ `async` sans `try/catch` → chaque appel réseau/IO doit gérer l'échec et exposer un état d'erreur.
- ❌ Avaler une erreur en silence (`catch {}`) sans log ni état.
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

> _(Claude ajoute les lignes ici au fil des reviews, format : `- [US-XXX] description du piège observé`)_

- _(vide pour l'instant)_
