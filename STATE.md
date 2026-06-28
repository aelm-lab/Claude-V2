# STATE.md — Kanban vivant Aanime

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** source de vérité UNIQUE de l'état courant (versions, métriques, Kanban).
> Le Kanban vivant **est ici** — `BACKLOG.md` est supprimé (ne plus le recréer), les epics
> détaillés vivent dans `EPICS.md`.
>
> **État de référence : fin S29 / v0.29.0** (régénéré depuis le handoff S29).
> ⚠️ Détail S22→S27 non capturé ici (vit dans les handoffs archivés) — voir §Trous connus.

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| v0.29.0 | S29 | « Polish & confiance » : BUG-1/2/4 (vérifiés morts en prod) · US-BUG5 · US-TITLE · US-SEARCH · US-SEARCH-2+CSS |
| v0.28.0 | S28 | **Epic Stats** : `useStats` + `StatsPage.vue` (« Mon année anime ») + route `/stats` (guard auth) + onglet Stats (`PrimaryNav.vue`) |
| S23→S27 | —  | ⚠️ Non capturé dans cette régénération (détail dans handoffs archivés S23→S27) |
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + `EPICS.md` consolidé |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| <= S16  | —  | Migration 0->7 · EPIC-1/2/3 clos · dual audit S16 |

> **Prochaine release :** bump **v0.30.0** à acter en clôture S30.

---

## 🎯 Métriques actuelles (fin S29)

- **134 tests unit** (18 fichiers) · type-check vert · zéro `any` · CI GitHub verte.
- Build **vite v6.4.2 · 174 modules**.
- Commit `main` de référence : **`532ea36`**.
- **`npm install` fonctionne en direct** — la parade `--legacy-peer-deps` n'est **plus nécessaire**
  (downgrade `@pinia/testing` mergé). Garder `--legacy-peer-deps` uniquement si un futur
  `package.json` réintroduit le conflit.
- ⚠️ **E2E : aucun runner local.** Pas de script `test:e2e` → Playwright **jamais exécuté en local**.
  2 specs gelées non couvertes : `week-mark-done.spec.ts` (clé `aanime_calendar` OK) +
  `search-remove` (à écrire). **R4/R5 restent théoriques tant que US-E2E-CONFIG n'est pas livré.**

---

## 📋 Kanban

### ✅ Done — S29 (gate groupé vert sur machine PO, mergé)
- **BUG-1 / BUG-2 / BUG-4** — vérifiés **MORTS en prod** (audit live PO, R6) → rayés sans spec.
- **US-BUG5** — bouton ✓ « Mark done » masqué sur statut Airing/Hiatus (`WeekAnimeItem.vue`, `v-if`).
- **US-TITLE** — util `getAnimeTitle` (anglais primaire + romaji secondaire si différent).
- **US-SEARCH** — badge statut + nb épisodes + dual-titre dans `SearchInput.vue`.
- **US-SEARCH-2 + CSS** — ✓ Added cliquable (retrait du store) + polish (15px / ellipsis / 64px).

### 🔄 In Progress
- _(vide — S29 clôturé, S30 pas encore démarré)_

### 📝 To Do — S30 (ordre recommandé)
1. **US-E2E-CONFIG** 🔴 — script Playwright local + faire tourner la suite gelée (filet de sécurité). **Reco n°1.**
2. **US-SEARCH-3** — séparation visuelle « IN YOUR LIBRARY » / « ADD TO YOUR LIST » (cf. screenshot PO).
3. **N1** — header scroll → sous-onglets only (pattern Apple).
4. **N9** — date de semaine trop grande.
5. **Dual-titre rollout** — propager `getAnimeTitle` à la modale, RecCards, carte semaine.
6. **US-124** — mapping MAL `Dropped` → non importé (déféré ; repasse P0 si campagne import MAL).

### 🗂️ Backlog (epics — détail dans `EPICS.md`)
- **Recommendation** : Cluster B découverte (S3/S4 « Parce que vous avez aimé X », S1/C1/LB RecCard universel) · US-165 (`fetchTopFinishedAnime` → `useJikanApi`) · US-127 (SyncIndicator `startBackgroundRelationFetch` — **statut à confirmer**, cf. Trous connus).
- **Growth** : Cluster C (M4 partage, PTW4 suivi épisodes).
- **Onboarding** : US-140 (1ʳᵉ visite genres → animes → calendrier — levier rétention n°1).
- **Login** : redesign du fond de la page de connexion.
- **Stats / Vault** : STATS-5 (`topGenres` scoped completed-this-year pour la vue vault — déféré).
- **Dette** : 🟡 `.search-suggestion-added` `#10b981` en dur → `var(--airing)` · US-166-CSS (passe CSS groupée) · Option B US-127 (tous les fetches Jikan via `useJikanApi`, déféré).

---

## ❓ Trous connus / à confirmer (R3 — ne pas inventer)
- **S22→S27** : aucun détail per-sprint dans cette régénération. À reconstruire depuis les
  handoffs archivés si un audit le réclame.
- **US-127** : présent « in scope » dans la mémoire ~S28, **absent du backlog S30** du handoff S29.
  → Vérifier en début S30 s'il a été **livré** (alors le ranger en Done S2x) ou **déféré**.
- **Numérotation DEC / antipatterns** : si des entrées ont été ajoutées en S22→S27 hors de
  ma vue, vérifier le dernier numéro réel avant de coller les blocs d'append.

---

## 🔁 Démarrage de session (rappel)
1. Lire la Knowledge → confirmer (1 phrase) + signaler si STATE.md n'est plus à jour.
2. Afficher ce Kanban.
3. Questions de clarification s'il y a ambiguïté.
4. Proposer **US-E2E-CONFIG en premier** (R3 : lire `package.json` + `playwright.config.ts` s'il existe).
5. Attendre le `go` PO avant toute spec.
