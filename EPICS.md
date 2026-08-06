# EPICS.md — Taxonomie et acquis fonctionnel Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** le **« OÙ »** du projet — quelle surface porte quoi, et **ce qui est acquis** dans
> chacune. Sert à situer une US : `US-XXX [EPIC][SECTION][TYPE]`.
>
> 🔴 **Ce document ne contient aucun backlog, aucune priorité, aucun bug ouvert.** Tout ce
> qui bouge vit dans `STATE.md`. Ici ne figure que ce qui est **livré et stable** — le
> patrimoine fonctionnel. C'est ce qui permet de répondre « est-ce que ça existe déjà ? »
> sans confronter deux listes qui divergent.

---

## Taxonomie — 12 EPICs

| # | EPIC | Surface | État |
|---|---|---|---|
| 1 | **On Air** | Calendar Week / Month / List | ✅ Mature |
| 2 | **Discover** | This Season / For You / Coming Soon | ✅ Mature |
| 3 | **Library** | Completed / Plan to Watch / Upcoming | ✅ Mature |
| 4 | **Recherche** | `SearchInput` | ⚠️ **Livré mais inopérant** — dépendance externe cassée |
| 5 | **Modal** | `AnimeModal` / `LogoutConfirmModal` / `RecEngineModal` | ✅ Clos (centrage résolu S36) |
| 6 | **Navigation** | Header / navs / routing | ✅ Refonte scroll livrée |
| 7 | **Login & Authentification** | `LoginPage` / magic-link / logout | ✅ Mature |
| 8 | **Boot & Démarrage** | Orchestration `App.vue`, persistance, sync | ✅ Durci |
| 9 | **Onboarding & Rétention** | `/welcome` → `OnboardingPage` | ✅ **Livré** — atterrissage en investigation |
| 10 | **Moteur de Recommandation** | `useRecommendations` / `recEngine` | ✅ Socle livré |
| 11 | **Évolution Majeure** | Stats / monétisation / notifications | 🟡 Stats livré, reste à venir |
| 12 | **Plateforme & Dette Technique** | Build / CI / tests / infra | 🔄 Continu |

**Classes de risque associées** (`PILOTAGE.md §3`) : EPIC 8 et le moteur d'EPIC 10 sont 🔴
CRITIQUE par défaut. Tout le reste est 🟠 ou 🟢 selon la nature de l'US.

---

## EPIC 1 — On Air
**Surface :** carte de la semaine, vue mois, vue liste. Le cœur d'usage quotidien.

- Modal à l'ouverture au clic sur une carte (contrat d'event corrigé, P0.1).
- Vue Month réparée + sous-navigation Week / Month / List.
- Snap-to-today à l'ouverture de la semaine.
- Barre de progression fine sous la carte.
- Bouton ✓ « marquer vu » en 1 tap, **masqué** sur Airing / Hiatus (l'action reste dans la modale).
- Slot-fill des jours vides : suggestion + skip **session-only** (jamais persisté) + CTA jour vide.
- Amorce du dual-titre via l'util `getAnimeTitle`.

## EPIC 2 — Discover
**Surface :** This Season, Coming Soon, For You, résultats enrichis.

- **Déduplication des pools** unifiée : un helper pur `dedupeByMalId` remplace 3 chemins
  indépendants (This Season / recherche / For You).
- Feedback d'ajout visible : toast nommant la destination + toast d'auto-vault au boot.
- États actifs visibles sur les sous-onglets.
- `RecCard` : Add / clic / dismiss / « pas intéressé » tous câblés (P0.8a/b).
- Section « Because you watched X » (`BecauseYouWatched.vue`) + signaux et badge sur `RecCard`.
- Chips de filtre remises dans le flux normal après la refonte scroll d'EPIC 6.

## EPIC 3 — Library
**Surface :** Completed (vault), Plan to Watch (watchlist), Upcoming.

- Auto-vault **sens unique** sur `Finished Airing` + badge « Finished ».
- Studios normalisés en `studios: string[]` — la dimension studio du scoring est active.

## EPIC 4 — Recherche ⚠️
**Surface :** `SearchInput.vue`.

**Livré :**
- Recherche enrichie : année, studio, score, badge de statut, nombre d'épisodes, dual-titre.
- Bouton « + » d'ajout direct depuis les suggestions.
- État « ✓ Added » **cliquable** : retire l'anime d'où qu'il soit, avec toast « Removed ».
- Séparation visuelle « IN YOUR LIBRARY » / « ADD TO YOUR LIST ».
- 4 specs E2E écrites : `search-enriched`, `search-quick-add`, `search-dedup`, `search-hides-nav`.

🔴 **Mais la fonctionnalité est inopérante en production.** L'endpoint `/anime?q=` de Jikan
répond 504 sur toute requête neuve : chaque titre tapé produit une URL jamais mise en cache
chez Jikan. **Aucun correctif possible côté Aanime.** La réparation passe par une migration
vers AniList. État et détail → `STATE.md §Faits externes`.

📌 **Vocabulaire figé :** « Coming Soon » (jamais « Upcoming »), « Finished airing » (jamais
« Finished »).

## EPIC 5 — Modal
**Surface :** `AnimeModal`, `LogoutConfirmModal`, `RecEngineModal` — tous partagent
`.modal-backdrop`.

- Centrage des modales : **question close**. Le CSS des modales était conforme depuis le
  début ; le décalage venait d'un `width:100%` + `padding` sans `box-sizing:border-box` sur
  `.secondary-nav-wrapper`, qui faisait déborder le **document** de 30 px. Les modales
  `position:fixed` se centraient sur le viewport pendant que le contenu s'étalait plus large.
- Covers de relations enrichies dans la modale.
- « More like this » → ouverture en modale.
- `LogoutConfirmModal.vue` (S33).

> 🎓 **Leçon transférable :** devant un décentrage, mesurer d'abord
> `document.documentElement.scrollWidth` vs `window.innerWidth` **avant** de suspecter le
> composant qui paraît décalé. Le symptôme peut apparaître très loin de sa cause.

## EPIC 6 — Navigation
**Surface :** header, navs primaire et secondaire, routing.

- Déduplication des contrôles de navigation de date.
- Onglet Stats ajouté à `PrimaryNav.vue`.
- **Refonte du header au scroll (style iOS)** : le flicker de la nav secondaire venait de
  `v-show`/`display:none` provoquant des sauts de hauteur en cours de scroll. Résolu par un
  **sticky CSS pur, sans aucune logique JS de scroll-hide**.
- Taille et graisse de `.current-period` réduites.

## EPIC 7 — Login & Authentification
**Surface :** `LoginPage`, magic-link Firebase, déconnexion.

- Page de login stylée et brandée.
- Magic-link **in-app** : saisie de l'email dans un input, plus de `window.prompt`.
- Redirect post-login = `/` (choix assumé, le lien magique expire rarement).
- **Déconnexion** : `useFirebaseAuth.signOut()` + `LogoutConfirmModal.vue` + câblage
  `AppHeader.vue` — purge des clés `aanime_*`, `clearAll()` du store, redirection `/login`.
  Tests unitaires succès + erreur.

## EPIC 8 — Boot & Démarrage 🔴
**Surface :** orchestration `App.vue`, persistance, sync.

- Séquence de boot stricte + smoke test `App.spec.ts` comme filet de régression.
- **Boot en 2 phases** : paint immédiat depuis le cache local, réconciliation Firestore en
  fond par comparaison de timestamps → suppression de l'écran blanc d'environ 6 s.
- Loader pré-Vue statique dans `index.html` + `LoadingOverlay` remonté hors de la gate auth.
- **Tous les correctifs du dual audit s16 livrés** : `saveToDatabase` sous try/catch avec
  toast d'échec ; boot non bloquant ; mutations du store via actions ; legacy normalisé sans
  double cast ; mapping du statut legacy `'Continuing'`.
- Clés localStorage toutes préfixées `aanime_`, avec migration transparente au boot.

## EPIC 9 — Onboarding & Rétention
**Surface :** parcours de 1ʳᵉ visite, `/welcome` → `OnboardingPage.vue`.

**Livré** (découvert au cleaning S34 : la doc le donnait « non démarré » alors qu'il était
en production — d'où la règle « toute US sortant du backlog démarre par un grep ») :
- Écran « choisis 3 genres ».
- **8 suggestions** scorées sur ces genres, ajout en 1 tap (`selectOnboardingSuggestions`).
- Atterrissage sur calendrier pré-rempli : `finishWithSeed` → `addAnime` par item coché →
  `markOnboarded` → `saveToDatabase` → toast → `router.push('/week')`.

⚠️ **Un défaut d'atterrissage est en investigation** : `buildSeedEntry` ne pose jamais le
champ `day`, or la vue semaine filtre dessus. Détail et état → `STATE.md`.

C'est l'EPIC qui porte la **North Star**.

## EPIC 10 — Moteur de Recommandation 🔴
**Surface :** `useRecommendations`, `recEngine`, `TasteProfile`.

- Socle de scoring : genres, thèmes, demographics, studios, buckets de recency, presets.
- Dimension studio **active** (`normalizeAnime` produit toujours `studios: string[]`).
- Pool réactif pour les suggestions de remplissage du calendrier.
- `SyncIndicator` branché sur `startBackgroundRelationFetch`.
- Graphe de relations reconstruit depuis IndexedDB au boot.

## EPIC 11 — Évolution Majeure
**Surface :** fonctionnalités produit lourdes.

**Epic Stats livré :**
- `useStats` (composable) + `StatsPage.vue` « Mon année anime ».
- Route `/stats` derrière le guard auth + onglet dans `PrimaryNav.vue`.
- `useStats.spec.ts` — 4 cas : store vide, vault année courante, vault année passée, genres
  chevauchants.
- **`topGenres` scoped au contenu terminé cette année uniquement.** Un year-in-review compte
  ce qui a été consommé, pas les intentions (benchmark AniList / Spotify Wrapped / Letterboxd).
- Garde null-safety `genres ?? []` contre le crash sur cache legacy.

## EPIC 12 — Plateforme & Dette Technique
**Surface :** build, CI, tests, infra.

- **Socle E2E Playwright** + bypass d'auth statique, mort en production.
- **CI GitHub Actions** : `npm ci` → `vue-tsc` → `vitest run` → `build`.
- **Registre des batchs E2E audité et resynchronisé** — mapping 1:1 entre le disque et
  `package.json`. Le registre vit dans `AGENTS.md §7`.
- Exécution locale de Playwright confirmée fonctionnelle.
- Grille de cartes unifiée : classe `.aa-card-grid` partagée dans `style.css`, en remplacement
  des grilles locales qui divergeaient (dont une définie en `<style scoped>`, donc jamais
  appliquée ailleurs).
