# CLAUDE.md — Bible du projet « Aanime »

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat (glisser-déposer).
> C'est le document de référence que Claude consulte en priorité.
>
> **État de référence : session 16 (dual audit).** EPIC P0 + EPIC-2 + EPIC-3 **CLOS**.
> E2E : **26 specs / 30 tests** (batch1..3, `--workers=1`). Unit : **84**. Build **~717 kb** (index ~420 + firebase esm ~452), ~3.7 s, **zéro `any`**.
> Compteurs antipatterns : faux-vert ×5, R-SCOPE-1 ×2 (s10+s11), classe CSS inexistante ×3, paraphrase build ×4 (0 nouvelle depuis s10).

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
| Cache | IndexedDB + localStorage — **toutes les clés préfixées `aanime_`** (clé persistance principale : `aanime_calendar`, US-133/DEC-85) |
| Styles | `style.css` (variables CSS, dark mode `html.dark`) |
| Tests / CI | Vitest (**84** tests) + Playwright (**26 specs / 30 tests**) + GitHub Actions |

> ⚠️ **Migration de clés (US-133/DEC-85)** : toutes les clés localStorage ont été préfixées
> `aanime_` avec migration transparente des anciennes clés au boot (dans
> `usePersistence.loadFromDatabase`). L'ancienne clé `'animeCalendar'` (DEC-64) est
> **dépréciée** → `aanime_calendar`. Mapping complet dans `ARCHITECTURE_TECHNIQUE.md §7`.

---

## 3. Pourquoi cette réécriture (rappel)

Vanilla JS jugé « non récupérable » : pas de séparation des responsabilités, TS inexistant,
non testable. Réécrit proprement sur `feat/vue3-migration` en **reproduisant le comportement**
(quirks volontaires inclus), jamais le code. Une 1ʳᵉ tentative avait coûté ~20 j faute de
rigueur — ce workflow existe pour que ça ne se reproduise pas.

---

## 4. Qui code

Un **développeur junior IA-first** via **Gemini AI Studio**. Specs ultra-précises et
autoportantes (**Gemini n'a PAS accès à la Knowledge**). Aucune décision d'architecture seul.

→ Chaque US autoportante, bornée (max 3 fichiers, dépassement autorisé SI annoncé en gras
dans le titre), avec types fournis et anti-patterns.
→ Gouvernance permanente Gemini dans **`AGENTS.md`** (R-LIVRAISON 1-3, R-SCOPE 1-4, R4, R5,
R-CODE 1-7) + **`AGENTS_E2E.md`** (stratégie de test). Lus automatiquement par Gemini.
→ **Règle de démarrage session :** exiger de Gemini l'état des fichiers modifiés AVANT toute
action (récidive s10 : 5 fichiers modifiés sans US → cascade 80 % du temps perdu).

---

## 4-bis. Règle PO non-technique (gravée s15) — OBLIGATOIRE

Adnane est **Product Owner non-technique**. Pour **chaque US et chaque décision **,
Claude fournit systématiquement :
- **(a) Impact utilisateur concret** — ce que l'utilisateur final voit/ressent, ou « aucun visible — dette ».
- **(b) Recommandation Claude** — la décision proposée, sur laquelle le PO tranche.

Pas de jargon nu : toute proposition se traduit en conséquence ressentie.

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
5. **Zéro DOM direct** (sauf exceptions documentées : download Blob `useICS`, `<input>.click()` import MAL, `DOMParser`, `getElementById('boot-loader').remove()` dans `App.vue` `finally` — DEC-72).
6. **Zéro `any`**. Tout type vient de `TYPES_CONTRACT.md`.
7. **Gestion d'erreur obligatoire** : `try/catch` + état d'erreur réactif sur chaque async. ⚠️ **Trou audité s16** : `usePersistence.saveToDatabase` viole cette règle (`saveSchedule` sans try/catch) → US-153 P0.
8. **Pas d'état sur `window`** → `onMounted`/`onUnmounted` + `@vueuse/core`.
9. **Migration verticale** (types → logique → composant → test).
10. **Contrat d'event = le composant** : les consommateurs s'alignent sur ses `defineEmits`, jamais l'inverse (leçons P0.1/P0.8).
11. **Aucun `<style scoped>`** dans les composants existants — tous les styles vont dans `style.css` (DEC-72). Ne pas ajouter de `<style scoped>` dans une US sans validation explicite.

---

### Règles d' NON-NÉGOCIABLES
**R1 — Porte verte locale (preuve par le PO, jamais par l'implémenteur) 
Aucune US ne reçoit le verdict MERGE sans TROIS sorties brutes,
produites par le PO sur sa machine, jamais par Gemini, jamais paraphrasées :
  1. npm run type-check   (vue-tsc --noEmit)
  2. npm run test:run     (vitest run)
  3. npm run build
Les trois doivent être vertes. Une seule sortie rouge = pas de MERGE.
Toute preuve fournie par l'implémenteur ("82 passed" collé par Gemini)
est IRRECEVABLE : seule la machine du PO fait foi.





---

## 7. Gouvernance & process

Type-check vert + tests verts + build OK **≠ application fonctionnelle** (4 bugs runtime
session 6) et **≠ application utilisable** (modal morte + RecCard morte, sessions 7-8).
D'où les règles permanentes :

- **R1 — MERGE = triple preuve verte CI** : `vue-tsc` + `vitest run` + `build`, **3 sorties brutes séparées**, rejouées par `ci.yml`. **Paraphrase de build = review suspendue — 4 récidives au compteur.** Suggérer `npm run build 2>&1 | tail -40` si sandbox récalcitrant.
- **R2 — Test sur l'orchestration/store/câblage de composables.**
- **R3 — Un audit lit le CODE, pas les indicateurs verts.** Lire le code qui marche AVANT de proposer un fix (leçon s10). **Zéro-confiance y compris sur les diagnostics de Claude.**
- **R4 — Test E2E sur tout correctif UX/écran** : geste réel + DOM visible, ROUGE→VERT sans modif, une preuve ROUGE = un état figé unique. Pattern boot-dépendant : attendre `#boot-loader` hidden avant tout clic (DEC-72). Seeder 7 jours pour garantir une carte visible quel que soit le jour.
- **R5 — Test ciblé par US pendant l'epic, grand check E2E complet en fin d'epic, tests cumulatifs.** Grand check sandbox : `--workers=1` (`fullyParallel` provoque des `ERR_CONNECTION_REFUSED` dans le sandbox Gemini). Suite > timeout sandbox (~60 s) → lots `batch1..3` de ≤9 specs + `sweep`.
- **R6 — Audit PO live obligatoire avant clôture d'un epic** (gravé s12, formalisé s15).

Plus : zéro-confiance y compris sur le diagnostic de Claude (grep d'abord, hypothèse après),
diagnostic avant spec (grep lecture seule d'abord), max 3 fichiers (dépassement annoncé en
gras), fixtures via `makeAnime(Partial<AnimeEntry>)`.

---

## 8. Definition of Done (globale)

- [ ] Fichiers livrés = périmètre annoncé (≤ 3, ou dépassement annoncé en gras).
- [ ] `vue-tsc --noEmit` + `vitest run` + `build` verts (triple preuve, sorties brutes séparées, jamais de paraphrase).
- [ ] Test E2E R4 si l'US touche l'écran (ROUGE→VERT, test inchangé).
- [ ] Tous les critères d'acceptance ✅.
- [ ] Aucun anti-pattern de l'US présent.
- [ ] Règles d'architecture §6 respectées.
- [ ] Comportement fidèle à la référence vanilla.
- [ ] **Impact utilisateur + reco Claude énoncés** (règle §4-bis).

---

## 9. Glossaire métier (les « states ») + libellés UI

| State | Sens UI | Onglet | Libellé toast (destination visible) |
|---|---|---|---|
| `calendar` | En cours de visionnage / diffusion suivie | On Air | « Added to On Air » |
| `radar` | À venir, pas encore diffusé | Discover → Coming Soon | « Added to Coming Soon » |
| `watchlist` | Prévu de regarder | Library → Plan to Watch | « Added to Plan to Watch » |
| `vault` | Terminé / archivé | Library → Completed | « Moved to Completed » |

> ⚠️ Les libellés de toast nomment la **destination visible** (onglet réel), jamais le
> jargon interne (`radar`/`vault`). DEC-63/71.

Transitions automatiques :
- `radar → calendar` : diffusion commencée (début ≤ aujourd'hui − 7 j).
- `* → vault` : statut `Finished Airing` (auto-vault, sens unique). Toast « Moved to Completed » au boot (US-121, DEC-73).
- **Hiatus** : `calendar` + `Currently Airing` sans diffusion > 14 j → `isOnHiatus` (source unique, US-107).

> ⚠️ **Bug audité s16 (US-154, P1)** : le statut legacy `'Continuing'` (présent dans l'union
> `AnimeStatus`, injecté par la persistance) n'est pas géré par `getCardStatus` et tombe sur
> le défaut `Finished` → un show en cours de diffusion s'affiche « Finished ». Correctif planifié.

---

## 10. Carte des documents du projet

| Doc | Rôle |
|---|---|
| `CLAUDE.md` | Bible (ce fichier) |
| `ARCHITECTURE_TECHNIQUE.md` | Vue technique (couches, modules, boot, flux, registre clés `aanime_*`) |
| `ARCHITECTURE_FONCTIONNELLE.md` | Vue fonctionnelle (parcours, pont fonctionnel↔technique) |
| `AUDIT.md` | Audit existant vanilla + audit post-migration (s6) + dual audit (s16) |
| `AUDIT_UX_SESSION7.md` | Audit UX live (findings F1→F20, statuts à jour) |
| `PLAN_MIGRATION.md` | Plan en 7 phases + arbre des composants |
| `TYPES_CONTRACT.md` | Contrat TypeScript de référence |
| `ROADMAP.md` | Epics post-migration |
| `BACKLOG.md` | Kanban vivant + règles de tenue |
| `DECISIONS.md` | Journal des décisions (DEC-01→87) |
| `ANTIPATTERNS.md` | Pièges récurrents (archi, runtime, UX/E2E, event-name, process) |
| `AGENTS.md` | Gouvernance permanente Gemini (racine dépôt) |
| `AGENTS_E2E.md` | Agent de test technique & E2E Gemini — stratégie R5 (racine dépôt) |
| `HANDOFF_SESSION16.md` | Reprise de contexte pour la session 17 |

*Archivés / supprimables : HANDOFF_SESSION5→10, PHASE8_DEBT.md (→ ROADMAP).*
