# EPICS.md — Taxonomie et acquis fonctionnel

> **Rôle :** le **« OÙ »** du projet — quelle surface porte quoi, et **ce qui est acquis**. Sert à situer une US : `US-XXX [EPIC][SECTION][TYPE]`.
> **Pas ici :** aucun backlog, aucune priorité, aucun bug ouvert (→ `STATE.md`). Ne figure ici que ce qui est **livré et stable**.

Ce document répond à « est-ce que ça existe déjà ? » sans confronter deux listes qui divergent.

---

## Taxonomie — 12 EPICs

| # | EPIC | Surface | État |
|---|---|---|---|
| 1 | **On Air** | Calendar Week / Month / List | ✅ Mature |
| 2 | **Discover** | This Season / For You / Coming Soon | ✅ Mature |
| 3 | **Library** | Completed / Plan to Watch / Upcoming | ✅ Mature |
| 4 | **Recherche** | `SearchInput` | ✅ Mature (AniList) |
| 5 | **Modal** | `AnimeModal` / `LogoutConfirmModal` / `RecEngineModal` | ✅ Clos |
| 6 | **Navigation** | Header / navs / routing | ✅ Refonte scroll livrée |
| 7 | **Login & Authentification** | `LoginPage` / magic-link / logout | ✅ Mature |
| 8 | **Boot & Démarrage** | Orchestration `App.vue`, persistance, sync | ✅ Durci |
| 9 | **Onboarding & Rétention** | `/welcome` → `OnboardingPage` | ✅ Livré |
| 10 | **Moteur de Recommandation** | `useRecommendations` / `recEngine` | ✅ Socle livré |
| 11 | **Évolution Majeure** | Stats / monétisation / notifications | 🟡 Stats livré, reste à venir |
| 12 | **Plateforme & Dette Technique** | Build / CI / tests / infra | 🔄 Continu |

**Classes de risque associées** (`PILOTAGE.md §3`) : EPIC 8 et le moteur d'EPIC 10 sont 🔴 CRITIQUE par défaut. Tout le reste est 🟠 ou 🟢 selon la nature de l'US.

---

## EPIC 1 — On Air
**Surface :** carte de la semaine, vue mois, vue liste. Le cœur d'usage quotidien.

- Modal à l'ouverture au clic sur une carte.
- Vue Month + sous-navigation Week / Month / List.
- Snap-to-today à l'ouverture de la semaine.
- Barre de progression fine sous la carte.
- Bouton ✓ « marquer vu » en 1 tap, **masqué** sur Airing / Hiatus (l'action reste dans la modale).
- Slot-fill des jours vides : suggestion + skip **session-only** + CTA jour vide.
- Dual-titre via l'util `getAnimeTitle`.

## EPIC 2 — Discover
**Surface :** This Season, Coming Soon, For You, résultats enrichis.

- **Déduplication des pools** unifiée : helper pur `dedupeByMalId`, source unique.
- Feedback d'ajout visible : toast nommant la destination + toast d'auto-vault au boot.
- États actifs visibles sur les sous-onglets.
- `RecCard` : Add / clic / dismiss / « pas intéressé » tous câblés.
- Section « Because you watched X » + signaux et badge sur `RecCard`.
- Chips de filtre dans le flux normal.
- Catalogue de saison servi par AniList (saison courante + suivante).

## EPIC 3 — Library
**Surface :** Completed (vault), Plan to Watch (watchlist), Upcoming.

- Auto-vault **sens unique** sur statut terminé + badge « Finished ».
- Studios normalisés en `studios: string[]` — la dimension studio du scoring est active.

## EPIC 4 — Recherche
**Surface :** `SearchInput.vue`.

- Recherche AniList : année, studio, score, badge de statut, nombre d'épisodes, dual-titre.
- Bouton « + » d'ajout direct depuis les suggestions.
- État « ✓ Added » **cliquable** : retire l'anime d'où qu'il soit, avec toast « Removed ».
- Séparation visuelle « IN YOUR LIBRARY » / « ADD TO YOUR LIST ».
- Déduplication appliquée **après le tri** (DEC-127) : la version la mieux classée survit.
- 4 specs E2E : `search-enriched`, `search-quick-add`, `search-dedup`, `search-hides-nav`.

📌 **Vocabulaire figé :** « Coming Soon », « Finished airing ».

## EPIC 5 — Modal
**Surface :** `AnimeModal`, `LogoutConfirmModal`, `RecEngineModal` — tous partagent `.modal-backdrop`.

- Centrage : **question close**. Le CSS des modales était conforme ; le décalage venait d'un débordement du **document** (DEC-107).
- Covers de relations enrichies dans la modale.
- « More like this » → ouverture en modale.

> 🎓 **Leçon transférable :** devant un décentrage, mesurer d'abord `document.documentElement.scrollWidth` vs `window.innerWidth` **avant** de suspecter le composant qui paraît décalé.

## EPIC 6 — Navigation
**Surface :** header, navs primaire et secondaire, routing.

- Déduplication des contrôles de navigation de date.
- Onglet Stats dans `PrimaryNav.vue`.
- **Refonte du header au scroll (style iOS)** : sticky CSS pur, **sans aucune logique JS de scroll-hide**.
- Taille et graisse de `.current-period` réduites.

## EPIC 7 — Login & Authentification
**Surface :** `LoginPage`, magic-link Firebase, déconnexion.

- Page de login stylée et brandée.
- Magic-link **in-app** : saisie de l'email dans un input, pas de `window.prompt`.
- Redirect post-login = `/`.
- **Déconnexion** : `signOut()` + `LogoutConfirmModal.vue` + câblage `AppHeader.vue` — purge des clés `aanime_*`, `clearAll()`, redirection `/login`. Tests unitaires succès + erreur.

## EPIC 8 — Boot & Démarrage 🔴
**Surface :** orchestration `App.vue`, persistance, sync.

- Séquence de boot stricte + smoke test `App.spec.ts` comme filet de régression.
- **Boot en 2 phases** : paint immédiat depuis le cache local, réconciliation Firestore en fond par comparaison de timestamps.
- Loader pré-Vue statique dans `index.html` + `LoadingOverlay` remonté hors de la gate auth.
- `saveToDatabase` sous try/catch avec toast d'échec ; boot non bloquant ; mutations du store via actions ; legacy normalisé sans double cast.
- Clés localStorage toutes préfixées `aanime_`, avec migration transparente au boot.

## EPIC 9 — Onboarding & Rétention
**Surface :** parcours de 1ʳᵉ visite, `/welcome` → `OnboardingPage.vue`.

- Écran « choisis 3 genres ».
- **8 suggestions** scorées sur ces genres, ajout en 1 tap (`selectOnboardingSuggestions`).
- Atterrissage sur calendrier pré-rempli : `finishWithSeed` → `addAnime` par item coché → `markOnboarded` → `saveToDatabase` → toast → `router.push('/week')`.

C'est l'EPIC qui porte la **North Star**.

## EPIC 10 — Moteur de Recommandation 🔴
**Surface :** `useRecommendations`, `recEngine`, `TasteProfile`.

- Socle de scoring : genres, thèmes, demographics, studios, buckets de recency, presets.
- Dimension studio **active**.
- Pool réactif pour les suggestions de remplissage du calendrier.
- Graphe de relations reconstruit depuis IndexedDB au boot.
- Pools servis par AniList : 2 appels (saison courante + suivante) + top finished.
- Nudges de séquelles sur relations AniList, avec cache 24 h dédié — **sans jamais écrire dans le store IDB `relations`** (DEC-144).
- Couvert par ses premières specs unitaires (`useRecommendations.spec.ts` + `.nudges.spec.ts`).

## EPIC 11 — Évolution Majeure
**Surface :** fonctionnalités produit lourdes.

**Epic Stats livré :**
- `useStats` (composable) + `StatsPage.vue` « Mon année anime ».
- Route `/stats` derrière le guard auth + onglet dans `PrimaryNav.vue`.
- `useStats.spec.ts` — 4 cas : store vide, vault année courante, vault année passée, genres chevauchants.
- **`topGenres` scoped au contenu terminé cette année uniquement.**
- Garde null-safety `genres ?? []` contre le crash sur cache legacy.

## EPIC 12 — Plateforme & Dette Technique
**Surface :** build, CI, tests, infra.

- **Socle E2E Playwright** + bypass d'auth statique, mort en production.
- **CI GitHub Actions** : `npm ci` → `vue-tsc` → `vitest run` → `build`.
- **Registre des batchs E2E** audité et resynchronisé, mapping 1:1 entre le disque et `package.json`. Le registre vit dans `AGENTS.md §7`.
- **Harnais de mock AniList mutualisé** : `tests/e2e/_helpers/anilistMock.ts`, source unique — une spec ne connaît jamais l'URL d'une API (DEC-149/150).
- Grille de cartes unifiée : classe `.aa-card-grid` partagée dans `style.css`.
