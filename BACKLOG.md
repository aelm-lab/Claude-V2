# BACKLOG.md — Kanban vivant

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Epics & estimations →** ROADMAP.md. **Audit UX →** AUDIT_UX_SESSION7.md.
> **État de référence : session 10 — EPIC P0 clos côté technique. Quick wins EPIC-4 livrés.**

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
- [doc] R4 inséré proprement dans `AGENTS.md`
- [P0.2] LoadingOverlay visible au boot (F2) — DEC-59. E2E `boot-loader.spec.ts`
- [P0.3a] `dedupeByMalId` + dédup This Season. E2E `discover-season-dedup.spec.ts` — DEC-60
- [P0.3c] Dédup recherche (`searchAnime`). E2E `search-dedup.spec.ts` — DEC-60
- [P0.8a] 🔴 Bouton Add RecCard réparé (`@heart`→`@add`). E2E `reccard-add.spec.ts` — DEC-61
- [P0.8b] 🔴 `@click`/`@not-interested` RecCard câblés ; emit orphelin supprimé. E2E `reccard-click-dismiss.spec.ts` — DEC-61
- [audit] Audit event-name transverse — CLOS (1 seul foyer RecCard, résolu)
- [P0.4] Feedback toast ajout depuis modal (F6 volet 1). E2E `modal-add-feedback.spec.ts` — DEC-63

### ✅ Done — Session 9 (EPIC P0)
- [P0.5] Sous-nav On Air Week/Month/List (F3). E2E `onair-subnav.spec.ts`
- [P0.6] Layout Month cassé (F4). E2E `month-layout.spec.ts`
- [P0.6-bis] Suppression doublon libellé période CalendarWeekPage (US-116 ✅). E2E `week-no-duplicate-period.spec.ts`
- [P0.6-ter] États actifs navs primaire + secondaire (F9 ✅). E2E `nav-active-state.spec.ts`
- [P0.6-quater] Layout secondary nav On Air. E2E `calendar-subnav-layout.spec.ts`
- [P0.7] `/login` stylé : AuthLayout + branding (F7 ✅). E2E `login-styled.spec.ts` — DEC-68
- [P0.9] 🔴 Modal en bas de page → overlay centré (`.modal-backdrop` CSS). E2E `modal-position.spec.ts` — DEC-70
- [P0.4-bis] Harmonisation toasts jargon → destination visible (DEC-71). E2E `toast-labels.spec.ts`

### ✅ Done — Session 10 (EPIC P0 final + quick wins EPIC-4)
- [US-121] Auto-vault muet → 2 toasts séparés « Moved to Completed » (F6 volet 2). E2E `auto-vault-toast.spec.ts` — DEC-73
- [P0.3b] Dédup For You batch via `dedupeByMalId` sur batch sortant (F5 ✅ complet). E2E `foryou-dedup.spec.ts` — DEC-74
- [DEC-72] Dette boot-loader résorbée (loader hors `#app`, suite E2E réparée)
- [US-141] Marquer-vu 1 tap depuis calendrier (bouton ✓ WeekAnimeItem). E2E `week-mark-done.spec.ts` — DEC-75
- [US-150] Snap-to-today (remplace scroll-restore). E2E `snap-to-today.spec.ts` — DEC-76
- [US-142] Barre de progression cartes semaine (WeekAnimeItem + style.css). E2E `week-progress-bar.spec.ts`
- [US-143] **Fermée sans dev** — F16 déjà implémenté (`BecauseYouWatched` + `RecCard` ont déjà `_triggerTitle` + `_signals`)

**Suite E2E cumulative : 22 specs.** 76 tests unit verts. Build ~4.7s, 716kb.


### ✅ Done — Session 12 (clôture EPIC P0 via audit live PO)
- [US-P0-E] BUG-5 — Gating "Mark done"/recency sur statut. v-if="isFinished" ModalCalendarTop. E2E modal-status-gating
- [US-P0-C] BUG-2 — Modal centrée mobile : test-juge VERT sans fix → perception (artefact devtools). E2E modal-content-centered-mobile
- [US-P0-D] BUG-1 — Discover réactif Add/Dismiss modal. dismissedRecIds réactif + excludedIds computed. E2E modal-add-removes-from-discover
- [FIX] toast-labels réaligné post-gating (watchlist/Finished, /library/plan)

**EPIC P0 → CLOS. Suite E2E : 26 specs. 76 unit. Build 716kb.**

### 🔄 In Progress
_(rien)_

### 📝 To Do — Audit live PO (MISSION SESSION 11)
- **[Audit live]** PO pilote le navigateur sur l'app déployée → rapporte à Claude → Claude analyse + rédige specs E2E → Gemini implémente (voir HANDOFF_SESSION10.md §4 pour le flux complet)
- **[Clôture formelle EPIC P0]** Après audit live confirmé

### 📝 To Do — EPIC-4 quick wins restants (post-audit)
- [US-144] États vides actionnables — jours vides muets (F10), CTA « Explorer la saison »
- [P0.8c → US-152] `@more-like-this` RecCard — décision produit : option A (modal, gratuit) vs option B (section inline). 2 mockups générés s9. À trancher à l'attaque rétention.
- [US-141-CSS] Style `.rc-mark-done` (positionnement + cible tactile 44px) — bouton fonctionnel mais brut après audit live

### 📝 Findings CSS (dette EPIC-2/3 — issus sessions 9/10)
- F18 : ~150 lignes `.test-*` mortes dans `style.css`
- F19 : doublons `.post-it` (solid vs pastel, contradictoires)
- F20 : hacks CSS `:has([style*="none"])` morts post-migration
- F21 : `#app-loading-overlay { display:none !important }` à vérifier vs DEC-72
- F22 : `.month-header-mobile` orpheline (doublon créé P0.6)
- F23 : « your Vault » empty state `LibraryCompletedPage:37` (jargon, trivial)
- F24 : `BecauseYouWatched.vue` a un `<style scoped>` antérieur à DEC-72 (à inclure dans passe F18–F23)

### 📝 Backlog UX post-P0 (issus audit session 7 — non traités)
- [US-145] Recherche enrichie (année/score/+) (F13)
- [US-116-F12] Libellé période dupliqué (F12) — ✅ fermé P0.6-bis (côté page) ; vérifier visuellement
- Nouveaux sans US : F8 dark contraste sous-nav, F14 skeletons inutilisés, F15 hiérarchie typo + section BYW vide

### 🗂️ En standby (repris après EPIC P0 + audit)
- EPIC-2 : US-117 (défer Firestore, ROI 1er chargement = priorité), US-113b/c, US-114, US-111, US-112
- EPIC-3 : US-120 (bug studios), US-122 (email re-entry), US-123→130
- EPIC-4 : US-140 (onboarding), US-141-CSS, US-144, US-145, US-146→152
- EPIC-5 : PWA, Firestore RT, virtualisation listes

---

## Règles de tenue du backlog
- **Une seule US In Progress.** Pas de MERGE, pas de suite.
- US > 3 fichiers → scinder + prévenir le PO. Dépassement autorisé SI annoncé **en gras + dans le titre**.
- **Zéro confiance** : code brut intégral + sortie terminale brute.
- **R1** triple preuve verte (vue-tsc + vitest run + build), **3 sorties brutes séparées** (jamais de paraphrase build — 4 récidives au compteur).
- **R2** test runtime sur boot/store/câblage de composables.
- **R3** un audit lit le code. Lire le code qui marche AVANT de proposer un fix (leçon s10).
- **R4** tout correctif UX / feature touchant l'écran → E2E Playwright geste réel + DOM visible. ROUGE→VERT sans modif. Une preuve ROUGE = état figé unique.
- **R5** test ciblé par US pendant l'epic, grand check complet en fin d'epic. Tests cumulatifs dans `tests/e2e/`, jamais supprimés. Grand check sandbox : `--workers=1` (le parallèle provoque des `ERR_CONNECTION_REFUSED` dans le sandbox Gemini).
- **Diagnostic avant spec** : grep lecture seule, décision sur preuve.
- **Démarrage Gemini** : exiger l'état des fichiers modifiés AVANT toute action (récidive s10 : 5 fichiers modifiés sans US → cascade 80% du temps perdu).
- Livraison sans contenu intégral = review suspendue.
