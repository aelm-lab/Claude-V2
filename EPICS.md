# EPICS.md — Vue epics & avancées fonctionnelles Aanime

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** vue d'ensemble par EPIC (le « OÙ » : page / surface / système) avec l'état
> d'avancement réel. C'est le **cœur fonctionnel** du projet. Le Kanban du sprint courant
> vit dans `STATE.md` ; le « pourquoi » des choix dans `DECISIONS.md`.
>
> **État de référence : fin S29 / v0.29.0.**
> ⚠️ Détail S22→S27 partiellement non capturé (handoffs archivés). Trous marqués ⚠️.

---

## Taxonomie — 12 EPICs

| # | EPIC | Surface | État global |
|---|---|---|---|
| 1 | On Air | Calendar Week / Month / List | ✅ Mature, polish continu |
| 2 | Discover | Season / For You / résultats | ✅ Mature |
| 3 | Library | Vault / Watchlist / Plan / Completed | ✅ Mature, 1 US déférée |
| 4 | Recherche | `SearchInput` | 🔄 Actif (S29 + S30) |
| 5 | Modal | `AnimeModal` | ✅ Mature |
| 6 | Navigation | Header / navs / routing | 🔄 Polish S30 (N1, N9) |
| 7 | Login & Authentification | `LoginPage` / magic-link | ✅ Fonctionnel, redesign backlog |
| 8 | Boot & Démarrage | Orchestration `App.vue` | ✅ Durci (audit S16 résolu S19) |
| 9 | Onboarding & Rétention | 1ʳᵉ visite | ⬜ Non démarré — levier n°1 |
| 10 | Moteur de Recommandation | `useRecommendations` / `recEngine` | 🔄 Socle solide, Clusters B/C à venir |
| 11 | Évolution Majeure | Stats / monétisation / notifs | 🟡 Stats livré S28, reste backlog |
| 12 | Plateforme & Dette Technique | Build / CI / tests / infra | 🔄 Continu |

---

## EPIC 1 — On Air (Calendar)
**Surface :** carte de la semaine, vue mois, vue liste. Le cœur d'usage quotidien.

✅ **Livré :**
- Modal au clic carte (P0.1, contrat event corrigé).
- Vue Month réparée (layout, P0.6) + sous-nav Week/Month/List (P0.5).
- Snap-to-today à l'ouverture (US-150, DEC-76).
- Barre de progression fine sous la carte (US-142).
- Marquer-vu 1 tap : bouton ✓ direct sur la carte (US-141).
- Slot-fill jours vides : suggestion + Skip session-only (US-131, DEC-83) + CTA jour vide (US-144).
- **S29 — US-BUG5** : bouton ✓ masqué sur Airing/Hiatus (n'a de sens que sur terminé ; reste dispo via modale).
- **S29 — US-TITLE** (amorce dual-titre, util `getAnimeTitle`).

🔄 **Backlog :**
- Dual-titre rollout sur la carte semaine (S30).
- `.rc-mark-done` / `.test-*` : dette CSS (US-166-CSS).

---

## EPIC 2 — Discover
**Surface :** Season en cours, Coming Soon, For You, résultats enrichis.

✅ **Livré :**
- Déduplication des pools (Season / recherche / For You) — F5, 3 chemins clos.
- Feedback d'ajout visible (toast destination, P0.4) + auto-vault toast au boot (US-121).
- États actifs visibles sur les sous-onglets (P0.6-ter).
- RecCard : Add / clic / dismiss câblés (P0.8a/b).
- « Because you watched X » : `BecauseYouWatched.vue` + signaux RecCard (US-143, déjà implémenté).

🔄 **Backlog :**
- Skeletons au chargement (~6 s de blanc, `SkeletonCard` existe mais inutilisé — F14).

---

## EPIC 3 — Library
**Surface :** Vault (Completed), Watchlist, Plan to Watch.

✅ **Livré :**
- Auto-vault sens unique sur `Finished Airing` + badge « Finished » (US-120).
- Studios normalisés `studios: string[]` (US-134, résout dimension scoring inerte).

🔄 **Backlog :**
- **US-124** — mapping MAL `Dropped` → non importé (déféré ; P0 si campagne import MAL).
- **F15** — hiérarchie typo Library/Upcoming, section vide.
- Library Completed/Plan en chips (idée UX).

---

## EPIC 4 — Recherche  🔄 ACTIF
**Surface :** `SearchInput.vue`.

✅ **Livré :**
- Recherche enrichie année/studio/score + bouton « + » direct (US-145).
- **S29 — US-SEARCH** : badge statut + nb épisodes + dual-titre dans les suggestions.
- **S29 — US-SEARCH-2 + CSS** : ✓ Added cliquable (retrait du store, toast « Removed ») + polish (15px / ellipsis / 64px).

🔄 **To Do (S30) :**
- **US-SEARCH-3** — séparation visuelle « IN YOUR LIBRARY » / « ADD TO YOUR LIST » (screenshot PO).
- Dette : `.search-suggestion-added` `#10b981` en dur → `var(--airing)`.

📌 **Vocabulaire figé (P0.4) :** « Coming Soon » (pas Upcoming), « Finished airing » (pas Finished).

---

## EPIC 5 — Modal
**Surface :** `AnimeModal`.

✅ **Livré :**
- Centrage de la modale (`.modal-backdrop`, P0.9, DEC-70).
- Covers de relations enrichies (US-119).
- More-like-this → modale, option A (US-152, DEC).

🔄 **Backlog :**
- Dual-titre rollout dans la modale (S30).

---

## EPIC 6 — Navigation  🔄 POLISH S30
**Surface :** header, navs primaire/secondaire, routing (11 routes).

✅ **Livré :**
- Déduplication des contrôles de navigation date (US-105 / US-116).
- Onglet Stats ajouté à `PrimaryNav.vue` (S28).

🔄 **To Do (S30) :**
- **N1** — au scroll, header → sous-onglets only (pattern Apple).
- **N9** — date de semaine trop grande.

---

## EPIC 7 — Login & Authentification
**Surface :** `LoginPage`, magic-link Firebase.

✅ **Livré :**
- Page login stylée/brandée (P0.7).
- Magic-link UI in-app (input email, remplace `window.prompt` — US-122).
- Redirect post-login = reste `/` (DEC-82).

🔄 **Backlog :**
- Redesign du fond de la page de connexion.
- Redirect post-login vers la route d'origine (déféré, ROI faible).

---

## EPIC 8 — Boot & Démarrage
**Surface :** orchestration `App.vue`, persistance, sync.

✅ **Livré (durci) :**
- Séquence de boot stricte (US-102) + smoke test `App.spec.ts` (US-109).
- **Architecture 2 phases** : paint immédiat depuis le cache localStorage (Phase 1), réconciliation
  Firestore en fond via comparaison de timestamps (Phase 2) → suppression du ~6 s d'écran blanc.
- Correctifs dual audit S16 → tous livrés S19/S21 :
  - US-153 (P0) : `saveToDatabase` try/catch + toast échec.
  - US-155 (P1) : boot non bloquant (overlay levé après load local).
  - US-157/158 (P1) : mutations store via actions ; legacy normalisé, zéro double cast.
- Clés localStorage toutes préfixées `aanime_` + migration legacy au boot (US-133, DEC-85).

---

## EPIC 9 — Onboarding & Rétention  ⬜ NON DÉMARRÉ
**Surface :** parcours 1ʳᵉ visite.

🔥 **Levier rétention n°1 — backlog :**
- **US-140** — 1ʳᵉ visite : 3 genres → 5 animes → calendrier pré-rempli. À découper.

---

## EPIC 10 — Moteur de Recommandation
**Surface :** `useRecommendations`, `recEngine`, `TasteProfile`.

✅ **Livré :**
- Socle scoring (genres/themes/demographics/studios, recency buckets, presets).
- Studios désormais peuplés → dimension studio active (US-134/DEC-86).
- Pool réactif suggestions calendrier (US-118, effet de bord BUG-10 intermittent).

🔄 **Backlog :**
- **US-165** — `fetchTopFinishedAnime` inline → extraire vers `useJikanApi` (trivial).
- **US-127** — SyncIndicator sur `startBackgroundRelationFetch` (option A) — **statut à confirmer** (cf. STATE §Trous).
- **Cluster B découverte** : S3/S4 « Parce que vous avez aimé X », S1/C1/LB RecCard universel.
- Option B US-127 : tous les fetches Jikan via `useJikanApi` (déféré Vault).

---

## EPIC 11 — Évolution Majeure
**Surface :** fonctionnalités produit lourdes.

✅ **Livré (S28 — Epic Stats) :**
- `useStats` (composable) + `StatsPage.vue` « Mon année anime ».
- Route `/stats` avec guard auth + onglet Stats dans `PrimaryNav.vue`.
- `useStats.spec.ts` (4 cas : store vide, vault année courante, vault année passée, genres chevauchants).
- **`topGenres` scoped `completedThisYear` uniquement** — benchmark AniList / Spotify Wrapped /
  Letterboxd : un year-in-review ne compte que le **contenu consommé** (terminé).
- Garde null-safety (`genres ?? []`) contre le crash legacy cache.

🔄 **Backlog :**
- **STATS-5** — `topGenres` scoped completed-this-year pour la **vue vault** (déféré Vault).
- **Cluster C growth** : M4 partage, PTW4 suivi épisodes.
- Monétisation, notifications.

---

## EPIC 12 — Plateforme & Dette Technique
**Surface :** build, CI, tests, infra, dette transverse.

✅ **Livré :**
- EPIC-2 : code-split (lazy routes + chunk Firebase), défer init Firestore, `useScrollKeeper`.
- CI GitHub Actions verte (rejoue tout d'un bloc).
- Nettoyage fichiers parasites racine (US-159).
- `npm install` direct (downgrade `@pinia/testing` mergé — parade `--legacy-peer-deps` supprimée).

🔄 **Backlog (priorité S30) :**
- 🔴 **US-E2E-CONFIG** — script Playwright local + faire tourner la suite gelée. **Reco n°1 S30.**
  (Aujourd'hui R4/R5 théoriques : aucun runner E2E local.)
- **US-166-CSS** — passe CSS groupée (`.test-*` mortes, doublons `.post-it`, hacks `:has()` morts,
  `#app-loading-overlay`, `.month-header-mobile` orpheline, jargon « Vault » empty state).
- Cache relations IDB sans TTL (relations périmées indéfiniment).
- `clean.cjs` recréé par AI Studio à chaque commit Gemini → purger (`git rm clean.cjs`) avant chaque gate.

---

## ❓ Trous connus (R3)
- **S22→S27** : avancées non capturées ici. Compléter depuis handoffs archivés au besoin.
- **US-127** : à classer (livré ou déféré) en début S30.
