# CLAUDE.md — Bible du projet « Aanime »

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat (glisser-déposer).
> C'est le document de référence que Claude consulte en priorité.
>
> **État de référence : fin S33 (cleaning S34).** EPIC P0 + EPIC-2 + EPIC-3 **CLOS**.
> **Métriques (tests/E2E/build) : voir `STATE.md` — source unique, ne plus les dupliquer ici.**
> Compteurs antipatterns (historique, non rafraîchis depuis s16) : faux-vert ×5, R-SCOPE-1 ×2
> (s10+s11), classe CSS inexistante ×3, paraphrase build ×4 — **+ 2 nouvelles familles S33**
> (violation invariant auteur-test, preuve E2E sans ROUGE préalable) → détail `ANTIPATTERNS.md`.

---

## 1. L'application

**Aanime** est un tracker de calendrier d'animes. L'utilisateur :

- suit les animes en cours de diffusion sur un calendrier (semaine / mois) ;
- découvre de nouvelles séries via un **moteur de recommandations personnalisé** ;
- gère sa **bibliothèque** : terminés (vault), à voir (watchlist), à venir (radar) ;
- exporte son planning au format calendrier (`.ics`) ;
- importe sa liste depuis MyAnimeList (fichier XML) ;
- ses données sont **synchronisées dans le cloud** (Firebase) et en local ;
- peut se **déconnecter** (US-AUTH-LOGOUT, S33) — purge des clés `aanime_*` + reset store + redirect `/login`.

Les données d'animes proviennent de l'**API publique Jikan (MyAnimeList non-officielle), v4**
(⚠️ en panne depuis S33 — voir `STATE.md`).

> Vue fonctionnelle complète → **`ARCHITECTURE_FONCTIONNELLE.md`**.
> Vue technique complète → **`ARCHITECTURE_TECHNIQUE.md`**.

---

## 2. Stack ACTUELLE (cible atteinte)

Migration terminée : code 100 % Vue 3 + TypeScript. Vanilla supprimé (US-101).
Plan de migration original archivé → `OLD/PLAN_MIGRATION.md` (pur historique, plus de rôle opérationnel).

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
| Tests / CI | Vitest + Playwright + GitHub Actions — **compteurs à jour dans `STATE.md`** |

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
R-CODE 1-7) + **`AGENTS_E2E.md`** (stratégie de test). Lus automatiquement par Gemini —
**ces deux fichiers portent leurs propres compteurs en dur** (pas d'accès à `STATE.md`),
à resynchroniser manuellement à chaque cleaning.
→ **Règle de démarrage session :** exiger de Gemini l'état des fichiers modifiés AVANT toute
action (récidive s10 : 5 fichiers modifiés sans US → cascade 80 % du temps perdu).

---

## 4-bis. Règle PO non-technique (gravée s15) — OBLIGATOIRE

Adnane est **Product Owner non-technique**. Pour **chaque US et chaque décision**, Claude
fournit systématiquement :
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
│   └── ui/          # Composants réutilisables atomiques (AppHeader.vue y compris — confirmé S33)
├── composables/     # useXxx.ts — logique réactive réutilisable
├── stores/          # Pinia stores
├── utils/           # Fonctions PURES (zéro import Vue)
├── router/          # index.ts
├── types/           # Interfaces TypeScript partagées
├── lib/             # firebase.ts (singleton)
└── main.ts
tests/e2e/           # Tests Playwright (cumulatifs, exclus de Vitest)
```

> ⚠️ **Correction de chemin (S33)** : `AppHeader.vue` vit dans `src/components/ui/`, pas
> `src/components/` comme un handoff antérieur le supposait par erreur. Toujours vérifier
> le chemin réel par `findstr` avant de spécifier un fichier dans une US (R3).

---

## 6. Règles d'architecture NON-NÉGOCIABLES

1. **Composant `.vue`** = UI + réactivité locale. Jamais de `fetch`, `localStorage`/IndexedDB, ni logique métier lourde.
2. **Composable `useXxx.ts`** = logique réutilisable, n'expose que `readonly`/`computed`.
3. **Store Pinia** = état global, **aucun I/O**. Les `watch()` remplacent les `dispatchEvent`.
4. **Utils** = fonctions pures, **zéro import Vue**.
5. **Zéro DOM direct** (sauf exceptions documentées : download Blob `useICS`, `<input>.click()` import MAL, `DOMParser`, `getElementById('boot-loader').remove()` dans `App.vue` `finally` — DEC-72).
6. **Zéro `any`**. Tout type vient de `TYPES_CONTRACT.md`.
7. **Gestion d'erreur obligatoire** : `try/catch` + état d'erreur réactif sur chaque async.
8. **Pas d'état sur `window`** → `onMounted`/`onUnmounted` + `@vueuse/core`.
9. **Migration verticale** (types → logique → composant → test).
10. **Contrat d'event = le composant** : les consommateurs s'alignent sur ses `defineEmits`, jamais l'inverse (leçons P0.1/P0.8).
11. **Aucun `<style scoped>`** dans les composants existants — tous les styles vont dans `style.css` (DEC-72). Ne pas ajouter de `<style scoped>` dans une US sans validation explicite.

---

## 7. Gouvernance & process

Type-check vert + tests verts + build OK **≠ application fonctionnelle** et **≠ application
utilisable**. D'où les règles permanentes :

- **R1 — MERGE = triple preuve verte CI** : `vue-tsc` + `vitest run` + `build`, **3 sorties
  brutes séparées**, rejouées par `ci.yml`. **Paraphrase de build = review suspendue.**
  Suggérer `npm run build 2>&1 | tail -40` si sandbox récalcitrant.
- **R2 — Test sur l'orchestration/store/câblage de composables.**
- **R3 — Un audit lit le CODE, pas les indicateurs verts.** Lire le code qui marche AVANT de
  proposer un fix. **Zéro-confiance y compris sur les diagnostics de Claude** (récidive S33 :
  un premier correctif Gemini jugé hors-sujet sur le volet centrage de `LogoutConfirmModal`,
  intercepté avant merge — R3 a fonctionné).
- **R4 — Test E2E sur tout correctif UX/écran** : geste réel + DOM visible, ROUGE→VERT sans
  modif, une preuve ROUGE = un état figé unique. Pattern boot-dépendant : attendre
  `#boot-loader` hidden avant tout clic (DEC-72). Seeder 7 jours pour garantir une carte
  visible quel que soit le jour.
- **R5 — Test ciblé par US pendant l'epic, grand check E2E complet en fin d'epic, tests
  cumulatifs.** Grand check sandbox : `--workers=1`.
- **R6 — Audit PO live obligatoire avant clôture d'un epic.**
- **Invariant non-négociable, renforcé S33 : l'auteur du test ≠ l'auteur du code.** Aucune
  exception, même pour un test visuel « simple » — Gemini a violé cette règle en S33
  (test E2E auto-écrit pour valider son propre correctif), écarté intégralement.

---

## 8. Cycle de vie d'un anime (rappel, détail complet dans `ARCHITECTURE_FONCTIONNELLE.md`)

- `radar → calendar` : diffusion commencée.
- `* → vault` : statut `Finished Airing` (auto-vault, sens unique). Toast « Moved to
  Completed » au boot (US-121, DEC-73).
- **Hiatus** : `calendar` + `Currently Airing` sans diffusion > 14 j → `isOnHiatus` (source
  unique, US-107).

> ⚠️ **Bug audité s16 (US-154, P1)** : le statut legacy `'Continuing'` (présent dans l'union
> `AnimeStatus`, injecté par la persistance) n'est pas géré par `getCardStatus` et tombe sur
> le défaut `Finished` → un show en cours de diffusion s'affiche « Finished ». Correctif planifié.

---

## 9. Trois faits gravés (ne jamais réintroduire)

- `setAllData` **n'existe pas** (seulement `setData` + `clearAll`).
- `syncStatus` **n'existe pas** dans `AnimeEntry`.
- `reconcileWithDatabase` **n'existe plus** (réconciliation faite dans `loadFromDatabase`).

---

## 10. Carte des documents du projet

> **Ce tableau est la seule copie canonique** — ne pas le dupliquer ailleurs. Les autres
> docs qui en ont besoin renvoient ici.

| Doc | Rôle | Statut |
|---|---|---|
| `CLAUDE.md` | Bible (ce fichier) | Actif |
| `ARCHITECTURE_TECHNIQUE.md` | Vue technique (couches, modules, boot, flux, registre clés `aanime_*`) | Actif |
| `ARCHITECTURE_FONCTIONNELLE.md` | Vue fonctionnelle (parcours, pont fonctionnel↔technique) | Actif |
| `AUDIT_UX_SESSION7.md` | Audit UX live session 7 (findings F1→F23) | **Archivé → `OLD/`** (lecture seule, ne pas régénérer ; items ouverts rapatriés dans `EPICS.md`) |
| `PLAN_MIGRATION.md` | Ancien plan en 7 phases + arbre des composants | **Archivé → `OLD/`** (migration terminée, pur historique) |
| `TYPES_CONTRACT.md` | Contrat TypeScript de référence | Actif |
| `STATE.md` | Kanban vivant + métriques (**source unique des compteurs**) | Actif |
| `EPICS.md` | Avancées fonctionnelles par epic | Actif |
| `DECISIONS.md` | Journal des décisions (DEC-01→106) | Actif |
| `ANTIPATTERNS.md` | Pièges récurrents (archi, runtime, UX/E2E, event-name, process) | Actif |
| `AGENTS.md` | Gouvernance permanente Gemini (racine dépôt + Knowledge) | Actif |
| `AGENTS_E2E.md` | Agent de test technique & E2E Gemini (racine dépôt + Knowledge) | Actif |
| `METHODOLOGY.md` | Méthodo agile (cérémonies, gates, versionnage) | Actif |

*Supprimés définitivement (ne plus référencer) : `AUDIT.md`, `ROADMAP.md`, `BACKLOG.md`,
`HANDOFF_SESSION16.md`, `HANDOFF_SESSION5→10`, `PHASE8_DEBT.md`.*
