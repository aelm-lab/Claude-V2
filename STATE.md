## 📦 Versions
| Version | Sprint | Livré |
|---|---|---|
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + EPICS.md consolidé |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| ≤ S16   | —   | Migration 0→7 · EPIC-1/2/3 · dual audit S16 |

## 🎯 Métriques actuelles
102 tests unit (14 fichiers) · E2E cumulatifs (R5) : week-empty-day-cta, search-enriched,
search-quick-add, more-like-this-modal + suite existante · build vite v6.4.2 · 164 modules ·
index ~357 kb + firebase esm ~452 kb · type-check vert · zéro `any` · CI GitHub verte.

## 📋 Kanban

### ✅ Done — S21
- US-152 [REC][MORE-LIKE-THIS][FEATURE]  more-like-this → modal (Option A)
- US-157 [BOOT][PERSIST][DETTE]          Mutations store via actions (setSuppressPersist)
- US-158 [BOOT][PERSIST][DETTE]          Legacy normalisé, zéro double cast

### 🔄 In Progress
- (vide — clôture S21)

### 📝 To Do — S22 (recommandé)
- US-140 [ONBOARDING][FEATURE]   1ʳᵉ visite : genres → animes → calendrier (levier n°1, à découper)
- US-131-E2E [REC][TEST]         E2E slot-fill skip + clic→modal
- US-127 [REC][SYNC][FEATURE]    SyncIndicator tous fetches Jikan
- US-141-CSS [ON AIR][CSS]       Styliser `.rc-mark-done`
- F8 [PLATEFORME][CSS][A11Y]     Dark mode lisibilité

### 🗂️ Backlog
- Library : US-124, F15
- Recommendation : US-165, US-152B (stockée)
- Plateforme : US-166-CSS
- Évolution Majeure : monétisation, stats, notifs, Library chips
