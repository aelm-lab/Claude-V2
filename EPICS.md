# EPICS.md — Vue epics & avancées fonctionnelles Aanime

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** vue d'ensemble par EPIC (le « OÙ » : page / surface / système) avec l'état
> d'avancement réel. C'est le **cœur fonctionnel** du projet. Le Kanban du sprint courant
> vit dans `STATE.md` ; le « pourquoi » des choix dans `DECISIONS.md`.
>
> **État de référence : fin S33 (cleaning S34).**
> ⚠️ Détail S22→S27 partiellement non capturé (handoffs archivés). Trous marqués ⚠️.

---

## Taxonomie — 12 EPICs

| # | EPIC | Surface | État global |
|---|---|---|---|
| 1 | On Air | Calendar Week / Month / List | ✅ Mature, polish continu |
| 2 | Discover | Season / For You / résultats | ✅ Mature |
| 3 | Library | Vault / Watchlist / Plan / Completed | ✅ Mature, 1 US déférée |
| 4 | Recherche | `SearchInput` | 🔄 Actif (S29→S33) |
| 5 | Modal | `AnimeModal` / `LogoutConfirmModal` / `RecEngineModal` | 🔄 Audit centrage rouvert S33 |
| 6 | Navigation | Header / navs / routing | ✅ Refonte scroll livrée S30-S33, dette dark mode ouverte |
| 7 | Login & Authentification | `LoginPage` / magic-link / **logout** | ✅ Logout livré S33, redesign backlog |
| 8 | Boot & Démarrage | Orchestration `App.vue` | ✅ Durci (audit S16 résolu S19) |
| 9 | Onboarding & Rétention | 1ʳᵉ visite | ⬜ Non démarré — levier n°1 |
| 10 | Moteur de Recommandation | `useRecommendations` / `recEngine` | 🔄 US-127 confirmé livré S30-S33, Clusters B/C à venir |
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
- Dual-titre rollout sur la carte semaine.
- `.rc-mark-done` / `.test-*` : dette CSS (US-166-CSS).
- Micro-fix visuel Month signalé S33, capture PO en attente.

---

## EPIC 2 — Discover
**Surface :** Season en cours, Coming Soon, For You, résultats enrichis.

✅ **Livré :**
- Déduplication des pools (Season / recherche / For You) — F5, 3 chemins clos.
- Feedback d'ajout visible (toast destination, P0.4) + auto-vault toast au boot (US-121).
- États actifs visibles sur les sous-onglets (P0.6-ter).
- RecCard : Add / clic / dismiss câblés (P0.8a/b).
- « Because you watched X » : `BecauseYouWatched.vue` + signaux RecCard (US-143, déjà implémenté).
- **S30-S33 — US-NAV-A-FIX2** : régression chips de filtre (passées sous la secondary-nav
  devenue opaque suite à la refonte scroll EPIC 6) corrigée — retrait du `sticky`, flux normal
  sous la nav. **Confirmation visuelle bloquée tant que Jikan est KO** (impossible de charger
  de vraies données pour vérifier).

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

🔄 **To Do :**
- **US-SEARCH-3** — séparation visuelle « IN YOUR LIBRARY » / « ADD TO YOUR LIST » (screenshot PO).
  Décision ouverte : cacher les en-têtes de section vides (reco Claude) ou toujours les afficher.
  Seul `SearchInput.vue` à modifier — `isAdded` existe déjà sur `enrichedSuggestions`.
- 4 specs E2E de recherche déjà écrites, jamais exécutées : `search-enriched`,
  `search-quick-add`, `search-dedup`, `search-hides-nav` — à faire tourner, ne pas modifier.
- Dette : `.search-suggestion-added` `#10b981` en dur → `var(--airing)`.
- **US-ANILIST-SEARCH** (backlog).

📌 **Vocabulaire figé (P0.4) :** « Coming Soon » (pas Upcoming), « Finished airing » (pas Finished).

---

## EPIC 5 — Modal  🔄 AUDIT CENTRAGE ROUVERT S33
**Surface :** `AnimeModal`, `LogoutConfirmModal` (nouvelle, S33), `RecEngineModal` — tous
partagent la classe `.modal-backdrop`.

✅ **Livré :**
- Centrage de la modale historique (`.modal-backdrop`, P0.9, DEC-70). Signalement de
  centrage ultérieur clos comme perception d'audit (DEC-78, session 12, sur `AnimeModal`).
- Covers de relations enrichies (US-119).
- More-like-this → modale, option A (US-152).
- **S33** : `LogoutConfirmModal.vue` créée (US-AUTH-LOGOUT).

🔴 **Ouvert (NOUVEAU, S33) :**
- **US-MODAL-CENTER-AUDIT (P1)** — un signalement de centrage sur `LogoutConfirmModal` a
  déclenché une investigation : la classe `.modal-backdrop` n'a **aucune définition de style**
  dans les 3 templates qui la référencent (`AnimeModal.vue`, `LogoutConfirmModal.vue`,
  `RecEngineModal.vue`) — le CSS réel du centrage vient d'un fichier global **non encore
  localisé**. Contrairement à DEC-78 (fermé comme perception sur `AnimeModal` spécifiquement),
  rien ne garantit que `LogoutConfirmModal` hérite du même centrage correct — à vérifier,
  pas à supposer. Le test E2E officiel (boundingBox vs centre viewport, tolérance 24px)
  n'a **jamais été exécuté**. Périmètre étendu à **tous les popups + le site en général**
  (pas seulement le logout) sur demande PO.

🔄 **Backlog :**
- Dual-titre rollout dans la modale.

---

## EPIC 6 — Navigation  ✅ REFONTE SCROLL LIVRÉE S30-S33
**Surface :** header, navs primaire/secondaire, routing (11 routes).

✅ **Livré :**
- Déduplication des contrôles de navigation date (US-105 / US-116).
- Onglet Stats ajouté à `PrimaryNav.vue` (S28).
- **Refonte header scroll asymétrique (style iOS)** : le flicker de la secondary-nav
  (cause racine : `v-show`/`display:none` provoquant des sauts de hauteur en cours de
  scroll) a été éliminé par un passage à un **sticky CSS pur, sans logique JS de
  scroll-hide** (Piste A). Régression corollaire sur les chips Discover corrigée (voir EPIC 2).
- **N9** : taille/graisse de police de `.current-period` réduite.

🔄 **Backlog :**
- **F8 (rapatrié de `OLD/AUDIT_UX_SESSION7.md` lors de son archivage, S34)** — sous-nav
  quasi illisible en dark mode, logo à faible contraste. Ouvert depuis la **session 7**,
  jamais traité. *Impact utilisateur : la navigation secondaire est difficile à lire pour
  tout utilisateur en thème sombre.* Reco Claude : à traiter dans une prochaine passe CSS
  groupée (même véhicule que la dette `.search-suggestion-added`).

---

## EPIC 7 — Login & Authentification
**Surface :** `LoginPage`, magic-link Firebase, **déconnexion (nouveau S33)**.

✅ **Livré :**
- Page login stylée/brandée (P0.7).
- Magic-link UI in-app (input email, remplace `window.prompt` — US-122).
- Redirect post-login = reste `/` (DEC-82).
- **S33 — US-AUTH-LOGOUT** : `useFirebaseAuth.signOut()` + `LogoutConfirmModal.vue` +
  câblage `AppHeader.vue` (bouton 🚪, purge `aanime_*` + `animeStore.clearAll()` + redirect
  `/login`). Tests unit T7 (succès) / T8 (erreur). **MERGE.** Le volet centrage de la
  confirmation modale est suivi séparément sous EPIC 5 (`US-MODAL-CENTER-AUDIT`).

🔄 **Backlog :**
- Redesign du fond de la page de connexion (+ logo Google officiel).
- Redirect post-login vers la route d'origine (déféré, ROI faible).
- Piste `signInWithRedirect` (Safari mobile privé) — dette mineure.

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
- **S30-S33 — US-127** : SyncIndicator sur `startBackgroundRelationFetch` — **confirmé
  livré** (résout le trou laissé ouvert fin S29).

🔄 **Backlog :**
- **US-165** — `fetchTopFinishedAnime` inline → extraire vers `useJikanApi` (trivial).
- **Cluster B découverte** : S3/S4 « Parce que vous avez aimé X », S1/C1/LB RecCard universel.
- Option B (tous les fetches Jikan via `useJikanApi`, déféré Vault).

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
- **US-PWA** (backlog).
- Monétisation, notifications.

---

## EPIC 12 — Plateforme & Dette Technique
**Surface :** build, CI, tests, infra.

🔄 **En cours / à confirmer :**
- **US-E2E-CONFIG** — statut d'exécution locale de Playwright non formellement confirmé
  entre S30 et S33 (le handoff S33 suppose que `npx playwright test` fonctionne mais rien
  ne l'a vérifié explicitement). À clarifier en priorité.
- **US-JIKAN-HEALTHCHECK (P1)** — refinement acté S33 : usage dev-only (aucun signal visible
  utilisateur), avec détail par test au-delà d'un simple verdict global OK/KO.

🔄 **Backlog :**
- `reject: any` dans `QueueTask` (dette mineure).
- Fusion `.search-loading` / `.search-message`.
- Panne Jikan externe (504, confirmée S33) — bloque plusieurs confirmations visuelles.
