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
- F22 :
