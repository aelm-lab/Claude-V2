# STATE.md — État vivant Aanime (Kanban + versions)
> MAJ : S20. Taxonomie : voir EPICS.md. Méthodo : voir METHODOLOGY.md.
> Source de vérité du Kanban + historique des versions. Lu en ouverture de chaque sprint.

## 📦 Versions
| Version | Sprint | Livré |
|---|---|---|
| v0.20.0 | S20 | US-144 [ON AIR][WEEK] · US-145a/b [RECHERCHE] · US-159 [PLATEFORME] + refonte taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI GitHub · US-154 · US-155 · US-156a/b · US-167 |
| v0.18.0 | S18 | Instructions régénérées · diagnostic jst · downgrade Pinia |
| ≤ s16 | — | Migration 0→7 · EPIC-1/2/3 · dual audit s16 |

## 🎯 Métriques actuelles
102 tests unit (14 fichiers) · E2E cumulatifs (R5) : week-empty-day-cta, search-enriched,
search-quick-add + suite existante · build vite v6.4.2 · 164 modules · index ~357 kb +
firebase esm ~452 kb · type-check vert · zéro `any` · CI GitHub verte.

## 📋 Kanban

### ✅ Done — S20
- US-144  [ON AIR][WEEK][UX]        CTA « Explore this season » jours vides (F10)
- US-145a [RECHERCHE][UX]           Suggestions enrichies année·studio·★score (F13)
- US-145b [RECHERCHE][FEATURE]      Bouton « + » direct, routage statut (F13)
- US-159  [PLATEFORME][CI][DETTE]   gitignore reports + purge 9 parasites racine

### 🔄 In Progress
- (vide — clôture S20)

### 📝 To Do — prochain sprint (S21)
- US-152  [REC][MORE-LIKE-THIS][FEATURE]  Option A décidée (ouvre modal) — PRÊTE, 1ʳᵉ US S21
- US-157  [BOOT][PERSIST][DETTE]          Mutations via actions + toasts hors persistance (P1, pré-requis 158)
- US-158  [BOOT][PERSIST][DETTE]          Legacy normalisé, plus de double cast (P1)

### 🗂️ Backlog par EPIC
- **On Air** : US-141-CSS, BUG-10
- **Library** : US-124 (MAL Dropped), F15 [UPCOMING][CSS]
- **Recommendation** : US-127 (SyncIndicator), US-165 (extraire fetchTopFinishedAnime), US-131-E2E, US-152(B) inline (stockée)
- **Onboarding & Rétention** : US-140 (levier n°1, à découper)
- **Plateforme & Dette** : US-166-CSS, F8 [CSS][A11Y] dark mode
- **Évolution Majeure** : monétisation, stats « Mon année », notifs épisode, Library en chips

### 🗄️ Vault / EPIC-5 (long terme)
PWA · onSnapshot temps réel · virtualisation listes · TTL cache aanime_ · redirect post-login ·
file Jikan globale anti-429 · monitoring Sentry · a11y (focus trap, clavier, aria)

## 🐞 Bugs non-P0
BUG-10 (slot-fill intermittent) · BUG-3/6/8/9 résiduels.

## ⚠️ Dette technique active (relevée dual audit s16, non bloquante)
- usePersistence mute store hors action + porte logique/toasts → US-157.
- Chemin legacy `setData(... as unknown as ...)` sans garde runtime → US-158.
- 3 faits gravés (ne JAMAIS réintroduire) : setAllData ❌ · syncStatus ❌ · reconcileWithDatabase ❌.
