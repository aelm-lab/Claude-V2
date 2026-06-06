# CLAUDE.md — Bible du projet « Aanime » (migration Vue 3)

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat (glisser-déposer). C'est le document de référence que Claude consulte en priorité.

---

## 1. L'application

**Aanime** est un tracker de calendrier d'animes. L'utilisateur :

- suit les animes en cours de diffusion sur un calendrier (semaine / mois) ;
- découvre de nouvelles séries via un **moteur de recommandations personnalisé** (basé sur ses goûts) ;
- gère sa **bibliothèque** : terminés (vault), à voir (watchlist), à venir (radar) ;
- exporte son planning au format calendrier (`.ics`) ;
- importe sa liste depuis MyAnimeList (fichier XML) ;
- ses données sont **synchronisées dans le cloud** (Firebase) et en local.

Les données d'animes proviennent de l'**API publique Jikan (MyAnimeList non-officielle), v4**.

---

## 2. Stack ACTUELLE (à remplacer entièrement)

| Couche | Techno |
|---|---|
| Langage | Vanilla JavaScript (ES Modules), pas de TypeScript |
| Build | Vite 6 + `@vitejs/plugin-react` (React installé mais **inutilisé** → à supprimer) |
| Auth | Firebase v12 — connexion par **lien email** (passwordless) |
| Base de données | Firebase Firestore — un seul document par utilisateur : `schedules/{uid}` |
| API externe | Jikan MAL API v4 (`https://api.jikan.moe/v4`), publique, sans clé |
| Cache persistant | IndexedDB (`src/idb.js`) pour relations & recommandations |
| Cache session | `localStorage` (caches TTL, préférences, positions de scroll, timestamps de sync) |
| Routing | **Routeur maison impératif** : une string `AppStore.currentView` + show/hide de `<div>` |
| State | **Objet global mutable** `AppStore` qui notifie les vues via des **`CustomEvent` DOM** |
| Styles | CSS simple (`src/style.css`), variables CSS pour le thème, dark mode par toggle de classe sur `<body>` |

> Détail complet de l'existant (vues, événements, endpoints) → voir **`AUDIT.md`**.

---

## 3. Stack CIBLE

| Couche | Techno |
|---|---|
| Framework | **Vue 3** avec `<script setup lang="ts">` |
| Langage | **TypeScript strict** (`strict: true`, aucun `any` implicite) |
| State | **Pinia** |
| Routing | **Vue Router 4** |
| Utilitaires réactifs | **@vueuse/core** (remplace swipe, dark mode, click-outside, intersection observer…) |
| Build | **Vite 6 + `@vitejs/plugin-vue`** |
| Auth / DB | Firebase v12 (**inchangé**) |
| Styles | **`style.css` existant conservé** (pas de migration Tailwind dans ce projet) |

---

## 4. Pourquoi une réécriture big-bang (et pas progressive)

Le code vanilla a été audité par des seniors → verdict : **architecture non récupérable**. Les 4 reproches précis :

1. **Pas de séparation des responsabilités** — logique métier dans les vues, manipulation DOM dans la couche données (ex : `persistence.js` qui crée un bandeau dans le `<body>`).
2. **TypeScript inexistant / superficiel** — aucune interface, aucun contrat de données.
3. **Non testable / pas de gestion d'erreur** — couplage fort, happy-path uniquement, crashes silencieux.
4. **Structure de fichiers incohérente** — pas de convention, imports en spaghetti.

Décision : **on réécrit proprement**. Le vanilla JS reste sur `main` comme **référence fonctionnelle** (on reproduit le comportement, jamais le code). La migration se fait sur `feat/vue3-migration`.

---

## 5. Qui code

Un **développeur junior IA-first** qui génère le code via **Gemini AI Studio**. Il comprend ce qu'il fait et sait corriger, mais **il a besoin de specs ultra-précises** sinon Gemini part dans tous les sens (c'est l'origine des 20 jours perdus). Il **ne prend aucune décision d'architecture seul**.

→ Conséquence : chaque US doit être autoportante, bornée (max 3 fichiers), avec les types fournis et une checkliste d'anti-patterns.

---

## 6. Architecture cible — structure des dossiers

```
src/
├── assets/
├── components/
│   ├── layout/      # AppLayout.vue, AuthLayout.vue
│   ├── pages/       # Pages routées (1 par route)
│   └── ui/          # Composants réutilisables atomiques
├── composables/     # useXxx.ts — logique réactive réutilisable
├── stores/          # Pinia stores
├── utils/           # Fonctions PURES (zéro import Vue)
├── router/          # index.ts
├── types/           # Interfaces TypeScript partagées
└── main.ts
```

---

## 7. Règles d'architecture NON-NÉGOCIABLES

Ces règles doivent apparaître (ou être référencées) dans **chaque** User Story.

1. **Composant `.vue`** = UI + réactivité locale **uniquement**. Jamais de `fetch`, jamais d'accès `localStorage`/`IndexedDB`, jamais de logique métier lourde dedans.
2. **Composable `useXxx.ts`** = logique réutilisable. Il **n'expose que des `readonly ref` ou des `computed`** vers l'extérieur — jamais l'état brut mutable.
3. **Store Pinia** = état global partagé entre plusieurs pages. **Les `watch()` du store remplacent TOUS les `document.dispatchEvent`** de l'ancien code.
4. **Utils** = fonctions pures TypeScript, **zéro import de Vue**, faciles à tester en isolation.
5. **Zéro manipulation DOM directe** dans un composant (`document.getElementById`, `querySelector`, `appendChild`… sont **interdits** — on utilise `ref`, `v-if`, `v-for`, `v-bind`).
6. **Zéro `any`** (implicite ou explicite). Toute structure de données a une interface dans `src/types/`, définie **avant** d'être utilisée. La référence est `TYPES_CONTRACT.md`.
7. **Gestion d'erreur obligatoire** : chaque fonction `async` a un `try/catch` explicite et expose un état d'erreur (`ref<Error | null>` ou équivalent).
8. **Pas de variables globales sur `window`** (l'ancien code stocke des handlers sur `window.incomingScrollHandler` etc.) → on utilise `onMounted`/`onUnmounted` + `@vueuse/core`.
9. **Une feature se migre verticalement** (types → logique → composant → test), jamais horizontalement (« tous les composants d'un coup »).

---

## 8. Definition of Done (globale, toute US confondue)

Une US est `Done` seulement si :

- [ ] Les fichiers livrés correspondent **exactement** au périmètre annoncé (≤ 3 fichiers).
- [ ] `tsc --noEmit` passe sans erreur ni `any`.
- [ ] Tous les critères d'acceptance de l'US sont ✅.
- [ ] Aucun anti-pattern de la liste de l'US n'est présent.
- [ ] Les règles d'architecture §7 sont respectées.
- [ ] Le comportement reproduit fidèlement la référence vanilla JS correspondante.

---

## 9. Glossaire métier (vocabulaire des « states »)

L'app classe chaque anime dans un **state** (champ `state`) :

| State | Sens UI | Onglet |
|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air |
| `radar` | À venir, pas encore diffusé (anciennement `upcoming`) | Discover → Coming Soon |
| `watchlist` | Prévu de regarder (« Plan to Watch ») | Library → Plan to Watch |
| `vault` | Terminé / archivé | Library → Completed |

Transitions automatiques importantes (à reproduire) :
- Un anime `radar` dont la diffusion a commencé (date de début ≤ aujourd'hui − 7 j) → passe en `calendar`.
- Un anime qui finit sa diffusion (`Finished Airing`) → passe en `vault` (sens unique).
- Statuts Jikan : `Currently Airing`, `Not yet aired`, `Finished Airing` (+ valeurs legacy `Continuing`, `Ended` à normaliser).
