# BACKLOG.md — Kanban vivant

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Epics & estimations →** voir ROADMAP.md.

---

## 📋 Kanban — état fin de session 4 (Phases 5+6+7 closes)

### ✅ Done — Migration complète (Phases 0→7)
- Phases 0-4 : scaffold, utils purs (~50 tests), store+composables, router+layouts, 12 composants atomiques
- Phase 5 : 9 pages (US-028→035) + WeekDayStrip + BecauseYouWatched
- Phase 6 : stores/ui.ts, AnimeModal (shell+2 tops+2 strips), EpOverridePanel, RecencySheet, câblage 9 pages (US-036→041)
- Phase 7 : LoginPage routé, PrimaryNav/SecondaryNav/AppHeader/SearchInput/CalendarNavControls, entry point unique, nettoyage (US-042→048c)

### 🔄 In Progress
- _(rien — migration close, audit externe validé)_

### 📝 To Do — Sprint clôture (EPIC-1, voir ROADMAP.md)
- [US-101] Suppression fichiers vanilla morts
- [US-102] Rebranchement reScorePool post-sync
- [US-103] Fix 2 tests timezone-dépendants
- [US-104] style.css body.dark-mode → html.dark
- [US-105] Déduplication nav controls

### 🗂️ Epics
→ ROADMAP.md (EPIC-1 à EPIC-5)

---

## Règles de tenue du backlog
- **Une seule US In Progress à la fois.** Pas de MERGE, pas de suite.
- Quand une US passe MERGE → Done, remonter la suivante.
- US > 3 fichiers → scinder + prévenir le PO.
- **Zéro confiance** : code brut intégral + sortie terminale brute obligatoires.
- Sortie terminale = exactement deux lignes : `$ npx vue-tsc --noEmit` puis `$`. Tout commentaire après = violation.
- Livraison sans contenu intégral = review suspendue.
