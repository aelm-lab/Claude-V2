# CLAUDE.md — Bible du projet « Aanime » (migration Vue 3)

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat (glisser-déposer).
> C'est le document de référence que Claude consulte en priorité.
>
> **État de référence : session 9 ; compteur tests : 76 unit + 16 E2E..**

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

> Vue fonctionnelle complète → **`ARCHITECTURE_FONCTIONNELLE.md`**.
> Vue technique complète → **`ARCHITECTURE_TECHNIQUE.md`**.

---

## 2. Stack ACTUELLE (cible atteinte)

Migration terminée : code 100 % Vue 3 + TypeScript. Vanilla supprimé (US-101).

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
| Cache | IndexedDB + localStorage (clé persistance : `'animeCalendar'`) |
| Styles | `style.css` (variables CSS, dark mode `html.dark`) |
| Tests / CI | Vitest (76 tests) + Playwright (E2E) + GitHub Actions |

---

## 3. Pourquoi cette réécriture (rappel)

Vanilla JS jugé « non récupérable » : pas de séparation des responsabilités, TS inexistant,
non testable. Réécrit proprement sur `feat/vue3-migration` en **reproduisant le comportement**
(quirks volontaires inclus), jamais le code. Une 1ʳᵉ tentative avait coûté ~20 j faute de
rigueur — ce workflow existe pour que ça ne se reproduise pas.

---

## 4. Qui code

Un **développeur junior IA-first** via **Gemini AI Studio**. Specs ultra-précises et
autoportantes. Aucune décision d'architecture seul.

→ Chaque US autoportante, bornée (max 3 fichiers, dépassement autorisé SI annoncé en gras
dans le titre), avec types fournis et anti-patterns.
→ Gouvernance permanente Gemini dans **`AGENTS.md`** (R-LIVRAISON 1-3, R-SCOPE 1-4, R4, R5,
R-CODE 1-7) + **`AGENTS_E2E.md`** (stratégie de test). Lus automatiquement par Gemini.

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
tests/e2e/           # Tests Playwright (cumulatifs, exclus de Vitest)
```

---

## 6. Règles d'architecture NON-NÉGOCIABLES

1. **Composant `.vue`** = UI + réactivité locale. Jamais de `fetch`, `localStorage`/IndexedDB, ni logique métier lourde.
2. **Composable `useXxx.ts`** = logique réutilisable, n'expose que `readonly`/`computed`.
3. **Store Pinia** = état global, **aucun I/O**. Les `watch()` remplacent les `dispatchEvent`.
4. **Utils** = fonctions pures, **zéro import Vue**.
5. **Zéro DOM direct** (sauf exceptions : download Blob `useICS`, `<input>.click()` import MAL, `DOMParser`).
6. **Zéro `any`**. Tout type vient de `TYPES_CONTRACT.md`.
7. **Gestion d'erreur obligatoire** : `try/catch` + état d'erreur réactif sur chaque async.
8. **Pas d'état sur `window`** → `onMounted`/`onUnmounted` + `@vueuse/core`.
9. **Migration verticale** (types → logique → composant → test).
10. **Contrat d'event = le composant** : les consommateurs s'alignent sur ses `defineEmits`, jamais l'inverse (leçons P0.1/P0.8).

---

## 7. Gouvernance & process

Type-check vert + tests verts + build OK **≠ application fonctionnelle** (4 bugs runtime
session 6) et **≠ application utilisable** (modal morte + RecCard morte, sessions 7-8).
D'où les règles permanentes :

- **R1 — MERGE = triple preuve verte CI** : `vue-tsc` + `vitest run` + `build`, **3 sorties brutes séparées**, rejouées par `ci.yml`.
- **R2 — Test sur l'orchestration/store/câblage de composables.**
- **R3 — Un audit lit le CODE, pas les indicateurs verts.**
- **R4 — Test E2E sur tout correctif UX/écran** : geste réel + DOM visible, ROUGE→VERT sans modif, une preuve ROUGE = un état figé unique.
- **R5 — Test ciblé par US pendant l'epic, grand check E2E complet en fin d'epic, tests cumulatifs.**

Plus : zéro-confiance (code brut + sortie terminale littérale ; toute paraphrase de preuve =
review suspendue — 3 récidives « Build succeeded »), diagnostic avant spec (grep lecture
seule d'abord), max 3 fichiers (dépassement annoncé en gras), fixtures via `makeAnime(Partial<AnimeEntry>)`.

---

## 8. Definition of Done (globale)

- [ ] Fichiers livrés = périmètre annoncé (≤ 3, ou dépassement annoncé en gras).
- [ ] `vue-tsc --noEmit` + `vitest run` + `build` verts (triple preuve, sorties brutes séparées).
- [ ] Test E2E R4 si l'US touche l'écran (ROUGE→VERT, test inchangé).
- [ ] Tous les critères d'acceptance ✅.
- [ ] Aucun anti-pattern de l'US présent.
- [ ] Règles d'architecture §6 respectées.
- [ ] Comportement fidèle à la référence vanilla.

---

## 9. Glossaire métier (les « states ») + libellés UI

| State | Sens UI | Onglet | Libellé toast (destination visible) |
|---|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air | « Added to On Air » |
| `radar` | À venir, pas encore diffusé | Discover → Coming Soon | « Added to Coming Soon » |
| `watchlist` | Prévu de regarder | Library → Plan to Watch | « Added to Plan to Watch » |
| `vault` | Terminé / archivé | Library → Completed | « Moved to Completed » |

> ⚠️ Les libellés de toast nomment la **destination visible** (onglet réel), jamais le
> jargon interne (`radar`/`vault`). DEC-63. Harmonisation des derniers libellés jargonneux =
> P0.4-bis.

Transitions automatiques :
- `radar → calendar` : diffusion commencée (début ≤ aujourd'hui − 7 j).
- `* → vault` : statut `Finished Airing` (auto-vault, sens unique). *Auto-vault muet au boot = dette US-121.*
- **Hiatus** : `calendar` + `Currently Airing` sans diffusion > 14 j → `isOnHiatus` (source unique, US-107).

---

## 10. Carte des documents du projet

| Doc | Rôle |
|---|---|
| `CLAUDE.md` | Bible (ce fichier) |
| `ARCHITECTURE_TECHNIQUE.md` | Vue technique (couches, modules, boot, flux) |
| `ARCHITECTURE_FONCTIONNELLE.md` | Vue fonctionnelle (parcours, pont fonctionnel↔technique) |
| `AUDIT.md` | Audit existant vanilla + audit post-migration (session 6) |
| `AUDIT_UX_SESSION7.md` | Audit UX live (findings F1→F17, statuts à jour s8) |
| `PLAN_MIGRATION.md` | Plan en 7 phases + arbre des composants |
| `TYPES_CONTRACT.md` | Contrat TypeScript de référence |
| `ROADMAP.md` | Epics post-migration |
| `BACKLOG.md` | Kanban vivant + règles de tenue |
| `DECISIONS.md` | Journal des décisions (DEC-01→65) |
| `ANTIPATTERNS.md` | Pièges récurrents (archi, runtime, UX/E2E, event-name) |
| `AGENTS.md` | Gouvernance permanente Gemini (racine dépôt) |
| `AGENTS_E2E.md` | Agent de test technique & E2E Gemini — stratégie R5 (racine dépôt) |
| `HANDOFF_SESSION8.md` | Reprise de contexte pour la session 9 (EPIC P0 en cours) |

*Archivés / supprimables : HANDOFF_SESSION5/6/7, PHASE8_DEBT.md (→ ROADMAP).*
