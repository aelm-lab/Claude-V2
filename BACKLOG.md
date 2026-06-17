# BACKLOG.md — Kanban vivant

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Epics & estimations →** ROADMAP.md. **Audit UX →** AUDIT_UX_SESSION7.md.
> **État de référence : session 8 — EPIC P0 (correctifs UX) en cours, 6 US mergées.**

---

## 📋 Kanban

### ✅ Done — Migration (Phases 0→7) + EPIC-1 (remédiation post-audit)
- Voir HANDOFF_SESSION6.md (archivé). Migration close, 4 bugs runtime corrigés, tag v2-stable.

### ✅ Done — Session 7
- [US-113a] Lazy-loading des pages (router) — EPIC-2 (mis en pause au profit de l'UX)
- [P0.0] Socle Playwright + auth de test mockée (bypass `import.meta.env`, mort en prod, `grep -c=0`) — DEC-56
- [P0.0-bis] Isolation runners : `tests/e2e` exclu de Vitest, `smoke.spec.ts` nettoyé
- [P0.1] 🔴🔴 Modal morte réparée (event `click` vs `@open-modal` désaligné) + test E2E R4 — DEC-57/58

### ✅ Done — Session 8 (EPIC P0)
- [doc] R4 inséré proprement dans `AGENTS.md` (insertion chirurgicale, baseline vérifiée — aucune perte)
- [P0.2] LoadingOverlay réellement visible au boot (F2) — loader statique `index.html` (Phase 1) + `<LoadingOverlay>` remonté racine `App.vue` hors gate auth (Phase 2). Boot `onMounted` intouché. E2E `boot-loader.spec.ts` rouge→vert — DEC-59
- [P0.3a] Helper pur `dedupeByMalId` + dédup This Season (`fetchCurrentSeason`/`fetchUpcomingSeason`). E2E `discover-season-dedup.spec.ts` — DEC-60
- [P0.3c] Dédup suggestions de recherche (`searchAnime`, réutilise le helper). E2E `search-dedup.spec.ts` — DEC-60
- [P0.8a] 🔴 Bouton **Add mort** sur toutes les RecCard réparé (`@heart`→`@add` aligné sur 3 consommateurs + wrapper BYW). E2E `reccard-add.spec.ts` — DEC-61
- [P0.8b] 🔴 `@click` RecCard → `ui.openModal` + `@not-interested` → `dismissRec` câblés ; emit orphelin `open-modal` supprimé de DiscoverExplorePage. E2E `reccard-click-dismiss.spec.ts` — DEC-61
- [audit] **Audit event-name transverse** (`emit` vs `@listener`, tous composants) — CLOS : 1 seul foyer 🔴 (RecCard, résolu P0.8a/b), tout le reste aligné. `open-recency` confirmé émis par `ModalCalendarTop` (pas fantôme).
- [P0.4] Feedback toast à l'ajout depuis la modal (F6 volet 1) — `onAdd`/`onStartWatching` annoncent la destination visible. E2E `modal-add-feedback.spec.ts` — DEC-63

### 🔄 In Progress
- _(rien)_

### 📝 To Do — EPIC P0 (reste)
- [P0.5] Sous-nav On Air (Week/Month/List) → `/month` accessible depuis l'UI (F3)
- [P0.6] Layout Month cassé : en-tête grille + titres dans cellules (F4)
- [P0.7] `/login` stylé : AuthLayout + branding + explication flow (F7 / US-122)

### 📝 Findings dérivés à ordonnancer (issus de session 8)
- [P0.4-bis] Harmoniser le vocabulaire des toasts existants : `onMarkDone` « Moved to Vault »→« Moved to Completed » + `DiscoverExplorePage.onRecHeart` « Added to Radar »→« Added to Coming Soon ». (Optionnel : helper `stateToLabel` centralisé — dette légère.)
- [US-121] Auto-vault muet au boot : `usePersistence.applyLoadTransitions` passe un show `Finished Airing` en `vault` sans toast (F6 volet 2 — chemin boot, distinct de P0.4).
- [P0.8c] `@more-like-this` (RecCard, panneau why) non câblé — **décision produit requise** : modal simple (gratuit, la section ModalMoreLikeThis existe déjà) vs scroll-to-section (feature, ajout flag `stores/ui.ts`).
- [P0.3b] Dédup For You batch : `getNextBatch` passe à `buildNextBatch` `remainingPool` entier + un sous-ensemble `wildcards` → un item wildcard peut sortir 2× (« 1ʳᵉ et 3ᵉ carte identiques »). **Le seul chemin F5 restant.** Touche le moteur → examen renforcé. Approche pressentie : déduper le batch sortant par `id` (préserve la cadence 1-wildcard/5).

### 📝 Backlog UX (post-P0, issus de l'audit session 7)
- [US-143] Signaux recos visibles sur carte (`_signals`/`_triggerTitle` existent) (F16)
- [US-141] Marquer-vu en 1 tap depuis le calendrier
- [US-144] États vides actionnables (jours vides muets) (F10)
- [US-150] Snap-to-today (F11)
- [US-116] Libellé de période dupliqué (F12)
- [US-145] Recherche enrichie (année/score/+) (F13)
- Nouveaux (sans US) : F8 dark contraste sous-nav, F9 état actif onglets, F14 skeletons inutilisés, F15 hiérarchie typo + section BYW vide

### 🗂️ En standby (repris après EPIC P0)
- EPIC-2 : US-117 (défer Firestore, ROI 1er chargement), US-113b/c, US-114, US-111, US-112
- EPIC-3/4/5 : ROADMAP.md

---

## Règles de tenue du backlog
- **Une seule US In Progress.** Pas de MERGE, pas de suite.
- US > 3 fichiers → scinder + prévenir le PO (sauf suppression pure prouvée). **Exception session 8 :** le PO autorise le dépassement de 3 fichiers SANS validation préalable, à condition de l'annoncer **en gras et dans le titre de l'US**.
- **Zéro confiance** : code brut intégral + sortie terminale brute.
- **R1** triple preuve verte (vue-tsc + vitest run + build) pour tout MERGE — **3 sorties brutes séparées** (pas de chaîne `&&`).
- **R2** test runtime sur boot/store/câblage de composables.
- **R3** un audit lit le code.
- **R4** : tout correctif UX / feature touchant l'écran livre un test E2E Playwright qui reproduit le geste et asserte le DOM visible. ROUGE sur le bug, VERT après le fix, sans modification. **Une preuve ROUGE = un état figé, jamais rejouée dans un état différent.**
- **R5 (session 8, → AGENTS_E2E.md)** : pendant l'epic, chaque US ne teste QUE ce qu'elle impacte (E2E ciblé). À la fin de l'epic, grand check E2E complet (`npx playwright test` sans filtre). Les tests E2E sont **cumulatifs** dans `tests/e2e/`, jamais supprimés.
- **Diagnostic avant spec** : grep lecture seule, décision sur preuve.
- Livraison sans contenu intégral = review suspendue.
