# BACKLOG.md — Kanban vivant

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Epics & estimations →** ROADMAP.md. **Audit UX →** AUDIT_UX_SESSION7.md.
> **État de référence : session 7 — EPIC P0 (correctifs UX) en cours.**

---

## 📋 Kanban

### ✅ Done — Migration (Phases 0→7) + EPIC-1 (remédiation post-audit)
- Voir HANDOFF_SESSION6.md (archivé). Migration close, 4 bugs runtime corrigés, tag v2-stable.

### ✅ Done — Session 7
- [US-113a] Lazy-loading des pages (router) — EPIC-2 (démarré puis EPIC-2 mis en pause au profit de l'UX)
- [P0.0] Socle Playwright + auth de test mockée (bypass `import.meta.env`, mort en prod, `grep -c=0` prouvé) — DEC-56
- [P0.0-bis] Isolation runners : `tests/e2e` exclu de Vitest, `smoke.spec.ts` nettoyé
- [P0.1] 🔴🔴 Modal morte réparée (event `click` vs `@open-modal` désaligné) + test E2E R4 (rouge→vert) — DEC-57

### 🔄 In Progress
- _(rien — prochaine : P0.2)_

### 📝 To Do — EPIC P0 (correctifs UX bloquants, issus de l'audit live)
- [P0.2] LoadingOverlay réellement visible au boot (F2)
- [P0.3] Dédup pools par `mal_id` — Discover For You/Season + suggestions recherche (F5)
- [P0.4] Feedback d'ajout (toast) + auto-vault annoncé (F6 / US-121)
- [P0.5] Sous-nav On Air (Week/Month/List) → /month accessible (F3)
- [P0.6] Layout Month cassé : en-tête grille + titres dans cellules (F4)
- [P0.7] /login stylé : AuthLayout + branding + explication flow (F7 / US-122)

### 📝 Backlog UX (post-P0)
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
- US > 3 fichiers → scinder + prévenir le PO (sauf suppression pure prouvée).
- **Zéro confiance** : code brut intégral + sortie terminale brute.
- **R1** triple preuve verte (vue-tsc + vitest run + build) pour tout MERGE.
- **R2** test runtime sur boot/store/câblage de composables.
- **R3** un audit lit le code.
- **R4** (session 7) : tout correctif UX / feature touchant l'écran livre un test E2E
  Playwright qui reproduit le geste utilisateur et asserte le DOM visible. ROUGE sur le
  bug, VERT après le fix, sans modification.
- **Diagnostic avant spec** : grep lecture seule, décision sur preuve.
- Livraison sans contenu intégral = review suspendue.
