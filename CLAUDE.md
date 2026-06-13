# CLAUDE.md — Bible du projet « Aanime » (migration Vue 3)

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat (glisser-déposer).
> C'est le document de référence que Claude consulte en priorité.
>
> **État de référence : fin de session 6 (EPIC-1 clos, prêt pour tag `v2-stable`).**

---

## 1. L'application

**Aanime** est un tracker de calendrier d'animes. L'utilisateur :

- suit les animes en cours de diffusion sur un calendrier (semaine / mois) ;
- découvre de nouvelles séries via un **moteur de recommandations personnalisé** ;
- gère sa **bibliothèque** : terminés (vault), à voir (watchlist), à venir (radar) ;
- exporte son planning au format calendrier (`.ics`) ;
- importe sa liste depuis MyAnimeList (fichier XML) ;
- ses données sont **synchronisées dans le cloud** (Firebase) et en local.

Les données d'animes proviennent de l'**API publique Jikan (MyAnimeList non-officielle), v4**.

> Vue fonctionnelle complète (parcours, states, pont fonctionnel→technique) → **`ARCHITECTURE_FONCTIONNELLE.md`**.
> Vue technique complète (couches, modules, boot, flux) → **`ARCHITECTURE_TECHNIQUE.md`**.

---

## 2. Stack ACTUELLE (cible atteinte)

La migration est **terminée** : le code est désormais 100 % Vue 3 + TypeScript. Le vanilla
JS de référence a été supprimé du dépôt (US-101).

| Couche | Techno |
|---|---|
| Framework | Vue 3 (`<script setup lang="ts">`) |
| Langage | TypeScript strict (zéro `any`) |
| State | Pinia (`stores/anime.ts`, `stores/ui.ts`) |
| Routing | Vue Router 4 (11 routes, guards auth/guest) |
| Réactivité | `@vueuse/core` |
| Build | Vite 6 + `@vitejs/plugin-vue` |
| Auth / DB | Firebase v12 (Auth email-link + Firestore) |
| API | Jikan v4 |
| Cache | IndexedDB + localStorage |
| Styles | `style.css` (variables CSS, dark mode `html.dark`) |
| Tests / CI | Vitest (69 tests) + GitHub Actions |

---

## 3. Pourquoi cette réécriture (rappel)

Le vanilla JS avait été audité « non récupérable » : pas de séparation des responsabilités,
TypeScript inexistant, non testable, pas de gestion d'erreur, structure incohérente. On a
réécrit proprement sur `feat/vue3-migration` en **reproduisant le comportement** (y compris
les quirks volontaires), jamais le code. Une première tentative avait coûté ~20 jours faute
de rigueur d'architecture et de review — ce workflow existe pour que ça ne se reproduise pas.

---

## 4. Qui code

Un **développeur junior IA-first** qui génère le code via **Gemini AI Studio**. Il a besoin
de specs ultra-précises et autoportantes. Il **ne prend aucune décision d'architecture seul**.

→ Chaque US est autoportante, bornée (max 3 fichiers), avec types fournis et anti-patterns.
→ Les règles de gouvernance permanentes de Gemini vivent dans **`AGENTS.md`** (racine du
dépôt, lu automatiquement par Gemini) : R-LIVRAISON 1-3, R-SCOPE 1-3, R-CODE 1-6, zéro-confiance.

---

## 5. Architecture cible — structure des dossiers

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
├── lib/             # firebase.ts (singleton)
└── main.ts
```

---

## 6. Règles d'architecture NON-NÉGOCIABLES

Référencées dans **chaque** US (et gravées dans `AGENTS.md`) :

1. **Composant `.vue`** = UI + réactivité locale uniquement. Jamais de `fetch`, `localStorage`/IndexedDB, ni logique métier lourde.
2. **Composable `useXxx.ts`** = logique réutilisable, n'expose que `readonly`/`computed`.
3. **Store Pinia** = état global, **aucun I/O**. Les `watch()` remplacent les `dispatchEvent`.
4. **Utils** = fonctions pures, **zéro import Vue**.
5. **Zéro DOM direct** (sauf exceptions : download Blob `useICS`, `<input>.click()` import MAL, `DOMParser`).
6. **Zéro `any`**. Tout type vient de `TYPES_CONTRACT.md`.
7. **Gestion d'erreur obligatoire** : `try/catch` + état d'erreur réactif sur chaque async.
8. **Pas d'état sur `window`** → `onMounted`/`onUnmounted` + `@vueuse/core`.
9. **Migration verticale** (types → logique → composant → test).

---

## 7. Gouvernance & process (durci après l'audit de session 6)

L'audit post-migration a révélé que **type-check vert + tests verts + build OK ≠ application
fonctionnelle** : 4 bugs runtime vivaient dans l'orchestration sans casser le tooling. Trois
règles permanentes en découlent (détail dans `AGENTS.md`, historique dans `AUDIT.md`) :

- **R1 — MERGE conditionné à la triple preuve verte CI** : `vue-tsc` + `vitest run` + `build`, rejoués par `.github/workflows/ci.yml` à chaque push.
- **R2 — Toute US touchant l'orchestration/un store/un câblage de composables livre un test** (le bug ne se voit pas à la compilation).
- **R3 — Un audit lit le CODE, pas seulement les indicateurs verts.**

Plus les règles historiques : zéro-confiance (code brut + sortie terminale littérale),
max 3 fichiers, livraison = contenu intégral, fixtures via `makeAnime(Partial<AnimeEntry>)`.

---

## 8. Definition of Done (globale)

Une US est `Done` seulement si :
- [ ] Fichiers livrés = périmètre annoncé (≤ 3).
- [ ] `vue-tsc --noEmit` + `vitest run` + `build` verts (triple preuve, sorties brutes).
- [ ] Tous les critères d'acceptance ✅.
- [ ] Aucun anti-pattern de l'US présent.
- [ ] Règles d'architecture §6 respectées.
- [ ] Comportement fidèle à la référence vanilla.

---

## 9. Glossaire métier (les « states »)

| State | Sens UI | Onglet |
|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air |
| `radar` | À venir, pas encore diffusé | Discover → Coming Soon |
| `watchlist` | Prévu de regarder | Library → Plan to Watch |
| `vault` | Terminé / archivé | Library → Completed |

Transitions automatiques :
- `radar → calendar` : diffusion commencée (début ≤ aujourd'hui − 7 j).
- `* → vault` : statut `Finished Airing` (auto-vault, sens unique).
- **Hiatus** : `calendar` + `Currently Airing` sans diffusion depuis > 14 j → signalé à l'affichage (`isOnHiatus`, **source unique**, US-107).
- Statuts Jikan normalisés : `Currently Airing`, `Not yet aired`, `Finished Airing` (+ legacy `Continuing`, `Ended`).

---

## 10. Carte des documents du projet

| Doc | Rôle |
|---|---|
| `CLAUDE.md` | Bible (ce fichier) |
| `ARCHITECTURE_TECHNIQUE.md` | Vue technique (couches, modules, boot, flux) |
| `ARCHITECTURE_FONCTIONNELLE.md` | Vue fonctionnelle (parcours, pont fonctionnel↔technique) |
| `AUDIT.md` | Audit de l'existant vanilla + audit post-migration (session 6) |
| `PLAN_MIGRATION.md` | Plan en 7 phases + arbre des composants cible |
| `BACKLOG.md` | Kanban vivant + US |
| `ROADMAP.md` | Epics post-migration (remplace l'ancien PHASE8_DEBT.md) |
| `TYPES_CONTRACT.md` | Contrat TypeScript de référence |
| `DECISIONS.md` | Journal des décisions d'architecture (DEC-xx) |
| `ANTIPATTERNS.md` | Pièges Gemini récurrents |
| `AGENTS.md` | Gouvernance permanente Gemini (racine dépôt) |
| `HANDOFF_SESSION6.md` | Reprise de contexte pour la session suivante |
| `AUDIT_UX_SESSION7.md` | Audit UX live (16 findings F1→F16, walkthrough navigateur) |
| `HANDOFF_SESSION7.md` | Reprise de contexte session 8 (EPIC P0 en cours) |
