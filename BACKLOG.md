# BACKLOG.md — Kanban vivant

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Epics & estimations →** voir ROADMAP.md.
> **État de référence : fin de session 6 (EPIC-1 clos, prêt pour tag `v2-stable`).**

---

## 📋 Kanban

### ✅ Done — Migration complète (Phases 0→7)
- Phases 0-4 : scaffold, utils purs (~50 tests), store+composables, router+layouts, 12 composants atomiques
- Phase 5 : 9 pages (US-028→035) + WeekDayStrip + BecauseYouWatched
- Phase 6 : stores/ui.ts, AnimeModal (shell+2 tops+2 strips), EpOverridePanel, RecencySheet, câblage 9 pages (US-036→041)
- Phase 7 : LoginPage routé, PrimaryNav/SecondaryNav/AppHeader/SearchInput/CalendarNavControls, entry point unique, nettoyage (US-042→048c)

### ✅ Done — EPIC-1 : remédiation post-audit (session 6)
- [US-109] Filet de sécurité : `ci.yml` (vue-tsc + vitest + build) + `App.spec.ts` (smoke test boot) — *absorbe la CI/CD initialement planifiée US-110*
- [US-102] 🔴 P0 — Moteur de reco rebranché au boot (App.vue orchestre, stubs morts supprimés de useSync) — DEC-50
- [US-106] 🟠 P1 — Throttle Jikan conditionnel au réseau réel (pattern `*WithMeta`/`fromNetwork`) — DEC-51
- [US-107] 🟠 P1 — Hiatus source unique 14j (suppression écriture morte 21j dans useSync) — DEC-52
- [US-101] Suppression des 31 fichiers vanilla morts (grappe fermée prouvée par inventaire)
- [US-104] Dark mode : `body.dark-mode` → `html.dark` (66 occurrences)
- [US-105] Déduplication des contrôles de navigation date (Prev/Next retirés des pages)
- [US-110] `AGENTS.md` musclé = gouvernance permanente Gemini (DEC-53) — *réversion : ce fichier devait être supprimé, il est finalement conservé*

> **US-103 SUPPRIMÉE** : pas de bug timezone. 66/66 tests réels (les « 2 échecs » de l'audit Claude Code étaient un artefact de son conteneur). Avec le smoke test US-109 : **69/69**.

### 🔄 In Progress
- _(rien — EPIC-1 clos, prêt pour tag `v2-stable`)_

### 📝 To Do — prochain sprint (EPIC-2, voir ROADMAP.md)
- [US-111] Tests E2E Playwright (3 parcours critiques)
- [US-112] Monitoring erreurs production (Sentry)
- [US-113] Code-splitting + fix warning chunking `idb.ts` (bundle 749 kb)
- [US-114] File Jikan globale (anti-429 navigation rapide) *(issu audit B)*
- [US-115] `useScrollKeeper` (dédup scroll restore) *(issu audit B)*

### 🧹 Ménage repo (avant tag)
- Supprimer `PHASE8_DEBT.md` (remplacé par ROADMAP.md)
- Supprimer `HANDOFF_SESSION5.md` (remplacé par HANDOFF_SESSION6.md)

### 🗂️ Epics
→ ROADMAP.md (EPIC-1 à EPIC-5)

---

## Règles de tenue du backlog
- **Une seule US In Progress à la fois.** Pas de MERGE, pas de suite.
- Quand une US passe MERGE → Done, remonter la suivante.
- US > 3 fichiers → scinder + prévenir le PO (sauf suppression pure prouvée).
- **Zéro confiance** : code brut intégral + sortie terminale brute obligatoires.
- **Triple preuve verte (R1)** : `vue-tsc` + `vitest run` + `build` pour tout MERGE.
- **Test obligatoire (R2)** sur toute US touchant boot / store / câblage de composables.
- Livraison sans contenu intégral = review suspendue.
