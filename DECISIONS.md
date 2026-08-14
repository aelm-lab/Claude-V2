# DECISIONS.md — Journal des décisions

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** capturer le contexte « mou » pour qu'une nouvelle session ne rejoue pas une
> décision déjà tranchée. Une entrée répond à : *« qu'est-ce que quelqu'un referait mal sans
> cette information ? »*
>
> **Séquence : DEC-01 → DEC-115.** Aucun numéro n'est jamais supprimé ni réattribué — les
> renvois existants dans les autres documents doivent rester valides. Une décision dépassée
> est **marquée** `⛔ SUPERSEDED`, pas effacée.
>
> **Ce qui n'est PAS ici :** l'état courant (→ `STATE.md`), les règles opposables
> (→ `AGENTS.md`, `PILOTAGE.md`), les pièges récurrents (→ `ANTIPATTERNS.md`).

---

## Index — les décisions structurantes encore actives

Point d'entrée par sujet. Le journal numéroté suit.

| Sujet | Décision |
|---|---|
| Clés localStorage & migration | **DEC-85** (préfixe `aanime_`) — ⛔ remplace DEC-64 |
| Orchestration du boot | **DEC-50** (séquence stricte) + **DEC-59** / **DEC-72** (loaders) |
| Contrat d'event composant → consommateurs | **DEC-58**, **DEC-61** |
| Déduplication des pools | **DEC-60** (`dedupeByMalId`), **DEC-74** (avant le slice) |
| Hiatus | **DEC-52** (source unique computed, 14 j) |
| Scoring par studio | **DEC-86** (`studios: string[]` toujours peuplé) |
| Socle E2E & bypass auth | **DEC-56** |
| Auteur du test ≠ auteur du code | **DEC-104** |
| Débordement horizontal & centrage des modales | **DEC-107** / **DEC-108** |
| Grille de cartes unifiée | **DEC-111** (`.aa-card-grid`) |
| État de Jikan | **DEC-113** (cache HIT/MISS) |
| Fallback sur cache périmé | **DEC-114** |
| Champ `day` jamais produit | **DEC-115** |
| Commandes de preuve | **AGENTS.md §2** — ⛔ remplace DEC-04 |

---

## DEC-01 → DEC-48 — Migration vanilla → Vue 3 (phases 0 à 7)

Décisions de scaffold et de portage, prises pendant la migration, **terminée**. Conservées en
index d'une ligne : leur prose détaillée n'a plus de valeur opérationnelle et reste
récupérable dans l'historique git.

| ID | Décision | Statut |
|---|---|---|
| DEC-01 | US-001b absorbée dans US-001 (5 fichiers, dépassement signalé et autorisé) | historique |
| DEC-02 | ESLint = flat config + `@vue/eslint-config-typescript@^14` → `no-explicit-any` en erreur | actif |
| DEC-03 | `tsconfig.node.json` séparé pour isoler `vite.config.ts` | actif |
| DEC-04 | `npx <outil>` ≡ `npm run <script>` | ⛔ **SUPERSEDED — `AGENTS.md §2`** : seuls `npm run type-check` / `test:run` / `build` sont recevables, les scripts npm portent des options que `npx` masque |
| DEC-05 | Le conteneur Gemini tourne nativement en UTC — pas de préfixe `TZ=UTC` | actif |
| DEC-06 | `jsdom` en devDep, déclaré par fichier via `// @vitest-environment jsdom` | actif |
| DEC-07 | Les DTO de plomberie restent des types locaux, hors contrat | actif |
| DEC-08 | Fixtures de test typées via helper `Partial` ou factory. Interdit : `as any`, `as unknown as T` | actif |
| DEC-09 | US-008-types : extension du contrat en **ajouts seuls**, rien renommé | historique |
| DEC-10 | Branches mortes simplifiées après `normalize` (genres/themes/studios toujours `string[]`) | actif |
| DEC-11 | Bug `item.studios` reproduit tel quel depuis le vanilla | ✅ résolu par DEC-86 |
| DEC-12 | `decayMultiplier = 0.2` conservé dans `buildTasteProfile` | actif |
| DEC-13 | Priorité de tri des signaux typée `Record<RecSignalKind, number>`, `score: 0` | actif |
| DEC-14 | `extractBecauseYouWatched` : param inutilisé préfixé `_profile`, signature publique préservée | actif |
| DEC-15 | `generateICSFile` scindé : `buildICSContent` pur dans `utils/ics.ts`, download + toast dans `useICS` | actif |
| DEC-16 | `parseMalXml` pur dans `utils/malImport.ts`, partie impure dans `useMalImport` | actif |
| DEC-17 | Dette UX du boot : `LoadingOverlay` piloté par état réactif | ✅ résolu |
| DEC-18 | Upsert du store : garder `if ('state' in input)`. Ne jamais recalculer `state` inconditionnellement en branche merge, sinon clobber | **actif — spec Claude fautive corrigée par Gemini** |
| DEC-19 | `needsBroadcastSync` dans `usePersistence` : mutation réactive qui déclenche le watch | actif |
| DEC-20 | Le bg worker retourne `[]` aussi bien sur « pas en cache » que « anime sans relations ». Idempotent | actif |
| DEC-21 | `buildRelationMemory` vient de `recs.js`, **pas** de `rec-engine.js` | **actif — spec Claude fautive** |
| DEC-22 | `fetchTopFinishedAnime` inline dans `useRecommendations` (fidèle au vanilla) | actif — extraction = US-165 |
| DEC-23 | `useTheme` applique la classe `dark` sur `<html>` (le vanilla l'appliquait sur `<body>`) | actif |
| DEC-24 | `MalImportResult` expose `imported`, pas `entries` | **actif — spec Claude corrigée par Gemini** |
| DEC-25 | Orchestration sync : stubs no-op permanents dans `usePersistence` | ⛔ SUPERSEDED par DEC-50 |
| DEC-26 | `watch → saveToDatabase` dans `usePersistence`, jamais dans le store. **Store sans I/O** | **actif — structurant** |
| DEC-27 | Double bloc `<script>` + `<script setup>` pour exporter un Symbol depuis `App.vue` | actif |
| DEC-28 | Guard auth : singleton `auth` + `await auth.authStateReady()` | **actif — structurant** |
| DEC-29 | Placeholders inline dans `router/index.ts` jusqu'à la phase 5 | historique |
| DEC-30 | `lastCalendarView` reporté phase 4 | historique |
| DEC-31 | `isBooting` via `provide`/`inject`, **fallback `ref(false)` obligatoire** | actif |
| DEC-32 | `syncAnimeUpdates` / `startBackgroundRelationFetch` en fire-and-forget | ⛔ ajusté par DEC-50 |
| DEC-33 | `ToastNotification` monté dans `AppLayout` uniquement | actif |
| DEC-34 | Lazy-load d'image via `<img style="display:none">`, pas `new Image()` | actif |
| DEC-35 | Dismiss de `SeasonNudgeCard` via `<Transition @after-leave>`, pas `setTimeout` | actif |
| DEC-36 | `ChipsStrip` : la chip `all` émet `null` | actif |
| DEC-37 | `WeekAnimeItem` reçoit `info` en prop | actif |
| DEC-38 | `MonthDayCell` reçoit `animes` déjà filtrés | actif |
| DEC-39 | Substitution des placeholders router au fil des US | historique |
| DEC-40 | Pattern stub `console.warn` en phase 5, câblage batché | historique |
| DEC-41 | `stores/ui.ts` pilote **tous** les overlays | **actif — structurant** |
| DEC-42 | `modalContext` : `libraryRec` prioritaire | actif |
| DEC-43 | `removeAnimeWithUndo` simplifié — l'undo est une dette | actif |
| DEC-44 | Prefetch des covers abandonné au profit du fallback `@error` | actif |
| DEC-45 | Signatures `useEpisodeInfo` / `useICS` corrigées a posteriori | historique |
| DEC-46 | `CalendarNavControls` route-aware | ⛔ résorbé par DEC-66 |
| DEC-47 | `synopsis?` ajouté à `AnimeEntry` | ⚠️ **absent du contrat** — voir `TYPES_CONTRACT.md §9` |
| DEC-48 | `ROADMAP.md` remplace `PHASE8_DEBT.md` | ⛔ obsolète — les deux documents ont été supprimés |

---

## DEC-49 → DEC-55 — Audit croisé & EPIC-1

- **DEC-49** Audit croisé : Gemini a vu 4 bugs runtime que Claude Code avait ratés. → naissance de la règle **R3** (un audit lit le CODE).
- **DEC-50** 🔴 **Orchestration du boot entièrement dans `App.vue`** : `load → await sync → await buildRelationMemory → reScorePool → bg fetch`. Couvert par `App.spec.ts`.
- **DEC-51** Pattern `*WithMeta` pour le throttle conditionnel : 1,1 s **seulement si `fromNetwork`**. Un throttle inconditionnel après un appel qui peut taper le cache ralentit pour rien.
- **DEC-52** 🔴 **Hiatus = source unique computed à 14 j.** Suppression de l'écriture morte à 21 j. Deux seuils pour une même règle métier = deux vérités.
- **DEC-53** `AGENTS.md` conservé et musclé comme gouvernance permanente de Gemini.
- **DEC-54** Filet de sécurité **avant** correctif : CI + smoke test + un test rouge encodant le bug.
- **DEC-55** Deux documents d'architecture séparés (technique et fonctionnelle).

## DEC-56 → DEC-63 — Audit UX live & EPIC P0

- **DEC-56** 🔴 **Socle E2E Playwright + bypass d'auth mort en production.** `import.meta.env.VITE_E2E_AUTH_BYPASS` en lecture **statique** : la branche est éliminée du bundle (prouvé `grep -c` = 0). `tests/e2e/**` exclu de Vitest.
- **DEC-57** **R4** — test E2E obligatoire sur tout correctif UX ou écran : geste réel, assertion sur le DOM visible, ROUGE puis VERT sans modifier le test.
- **DEC-58** 🔴 **Cause racine d'une modale morte = désalignement de nom d'event.** `WeekAnimeItem` émettait `click`, la page écoutait `@open-modal`. **Fix côté page** — jamais renommer l'emit du composant.
- **DEC-59** **Boot loader en deux phases.** Deux fenêtres d'écran vide distinctes : pré-mount (bundle pas encore parsé) et post-mount (`route.meta` vide pendant la résolution auth, donc `AppLayout` et son `LoadingOverlay` non montés). Fix : loader statique dans `index.html` + `<LoadingOverlay>` remonté à la racine d'`App.vue`, hors gate auth. *Correction d'un faux diagnostic de Claude : ce n'était pas un problème d'`inject` mais de **placement**.*
- **DEC-60** 🔴 **`dedupeByMalId` = source unique de déduplication.** Les doublons avaient 3 chemins indépendants. Helper pur générique, clé `mal_id` seule, garde la 1ʳᵉ occurrence. Appliqué avant `writeLocalCache` et avant tri dans `searchAnime`.
- **DEC-61** 🔴 **Contrat d'event = le composant, les consommateurs s'alignent.** `RecCard` émettait `add`/`skip`/`click`/`not-interested`/`more-like-this` ; ses 3 consommateurs écoutaient `@heart` (mort) et n'écoutaient ni `@click` ni `@not-interested`. Bouton Add mort, clic carte mort, « pas intéressé » mort — **0 erreur console**.
- **DEC-62** Toute action d'ajout ou de déplacement visible produit un **toast nommant la destination visible exacte**.
- **DEC-63** Libellés de toast harmonisés sur le vocabulaire visible : « Radar » → « Coming Soon », « Vault » → « Completed ».

## DEC-64 → DEC-71 — Correctifs UX P0.5 → P0.9

- **DEC-64** Clé localStorage `'animeCalendar'` confirmée. ⛔ **SUPERSEDED PAR DEC-85** — la clé est `aanime_calendar`. Ne plus utiliser `'animeCalendar'` dans un seed.
- **DEC-65** Stratégie E2E affinée (**R5**) : un test ciblé par US pendant l'epic, grand check complet en fin d'epic, specs cumulatives jamais supprimées.
- **DEC-66** Libellé de période = source unique dans `CalendarNavControls`.
- **DEC-67** Convention de classe active de nav = `.active` (markup aligné sur le CSS).
- **DEC-68** P0.7 = style pur, tokens existants, script intact.
- **DEC-69** P0.8c sorti du périmètre P0.
- **DEC-70** `.modal-backdrop` : overlay centré, CSS manquant ajouté — **3ᵉ occurrence du pattern « le markup référence une classe absente »**. Règles préfixées pour ne pas casser le `.modal` vanilla.
- **DEC-71** Libellés de toast harmonisés (« Vault » → « Completed », « Radar » → « Coming Soon »).

## DEC-72 → DEC-80bis — Quick wins & clôture EPIC P0

- **DEC-72** 🔴 **Boot-loader hors de `#app` + suppression DOM dans `App.vue`.** `#boot-loader` est déplacé hors de `<div id="app">` ; sa suppression se fait par `document.getElementById('boot-loader')?.remove()` dans le `finally` du `onMounted` — **exception R-CODE-4 documentée**. `main.ts` reste un `app.mount('#app')` simple, **sans** `router.isReady()` (testé : cassait `boot-loader.spec.ts`).
- **DEC-73** Toast « Moved to Completed » au boot pour l'auto-vault.
- **DEC-74** Déduplication appliquée **avant** le `slice` dans `getNextBatch`, jamais après.
- **DEC-75** ⚠️ **Non capturé.** Trou de lecture assumé entre DEC-74 et DEC-76. **Ne pas inventer son contenu.**
- **DEC-76** Snap-to-today remplace le scroll-restore dans `CalendarWeekPage` : `await nextTick()` + `findIndex(d => d.isToday)` + `scrollIntoView({ behavior:'auto', block:'start' })`, garde `todayIndex < 0`, appelé en `onMounted` et `onActivated` (KeepAlive). Pas de re-snap sur Prev/Next.
- **DEC-77** « Mark done » et la ligne recency gatés sur `isFinished` : ces actions n'ont de sens que sur un anime terminé.
- **DEC-78** Signalement de centrage clos comme **perception d'audit**, mesuré par boundingBox — **spécifiquement sur `AnimeModal`**. *(Ne pas extrapoler : la vraie cause générale est venue plus tard, DEC-107.)*
- **DEC-79** Réactivité Discover par **dérivation**, pas par canal modal → page : `excludedIds = union(store, dismissedRecIds)`. Add et Dismiss retirent la carte mécaniquement, quel que soit le chemin.
- **DEC-80** 🎓 **Conflit de règles découvert :** un anime `calendar` + `Finished` s'auto-vault au boot, tandis que `calendar` + `Airing` gate « Mark done ». Le scénario « Mark done depuis Week » est donc **structurellement impossible** après gating. Le test a été déplacé sur `watchlist` + `Finished` (exclu de l'auto-vault). Aucun code source modifié.
- **DEC-80bis** EPIC-2 clos : code-splitting, défer Firestore, fiabilité.

## DEC-81 → DEC-87 — EPIC-3 & dual audit s16

- **DEC-81** MAL `Dropped` = **non importé**. *Impact user : un anime abandonné sur MAL n'encombre pas la bibliothèque.*
- **DEC-82** Redirect post-login = reste `/`. Le redirect vers la route d'origine est déféré (le lien magique expire rarement, ROI faible).
- **DEC-83** Le skip d'une suggestion slot-fill est **session-only** : `ref<Set<number>>` local, jamais Pinia, jamais persisté. *« Écarter pour l'instant » ≠ « bannir ».*
- **DEC-84** Cleanup groupé : `POSTER_PLACEHOLDER` source unique (4 copies inline supprimées) ; `onHiatus?` retiré du type ; `episodeOverride` reseté à l'upsert.
- **DEC-85** 🔴 **Toutes les clés localStorage préfixées `aanime_` + migration legacy au boot**, dans `usePersistence.loadFromDatabase`. Registre complet → `ARCHITECTURE_TECHNIQUE.md §7`.
- **DEC-86** `normalizeAnime` produit **toujours** `studios: string[]`. *Impact user : les recommandations tiennent enfin compte du studio.*
- **DEC-87** 🎓 **Vérifications zéro-confiance de l'audit croisé s16.** Quatre points de vigilance hérités d'un handoff se sont révélés **faux** : `setAllData` n'existe pas · `syncStatus` = 0 hit · `reconcileWithDatabase` n'existe plus · l'import IDB dynamique n'était pas matérialisé. → **Un handoff est une source secondaire faillible ; le code réel tranche.** Backlog issu de l'audit (US-153 à US-159) : **tous livrés S19/S21**.

## DEC-88 → DEC-98 — Epic Stats, polish & infrastructure

- **DEC-88** `useStats` = composable dédié, `StatsPage.vue` = page pure. Séparation stricte des couches.
- **DEC-89** `topGenres` scoped au **contenu terminé cette année** uniquement. Benchmark AniList / Spotify Wrapped / Letterboxd. *Impact user : les stats reflètent ce qui a été vraiment regardé, pas les intentions.*
- **DEC-90** Garde null-safety `genres ?? []` : un cache legacy sans `genres` faisait crasher `topGenres`.
- **DEC-91** Route `/stats` derrière le guard auth — les stats sont personnelles.
- **DEC-92** BUG-1 / BUG-2 / BUG-4 fermés **sans spec** : invérifiables en production, déjà corrigés ou jamais reproductibles. Aucune ligne de code touchée. *Confirmation qu'un ressenti d'audit se périme.*
- **DEC-93** Le bouton ✓ « Mark done » est **masqué** sur Airing / Hiatus — fix présentationnel (`v-if`), aucune logique touchée. L'action reste dans la modale.
- **DEC-94** Règle de titre centralisée dans `getAnimeTitle` : anglais primaire + rōmaji secondaire si différent. Rollout progressif.
- **DEC-95** Vocabulaire de recherche figé : « Coming Soon », « Finished airing ».
- **DEC-96** L'état « ✓ Added » est **cliquable** : retire l'anime d'où qu'il soit + toast « Removed ».
- **DEC-97** Les couleurs **réutilisent les tokens** existants (`var(--airing)`, `var(--upcoming)`), aucune redéclaration.
- **DEC-98** `npm install` fonctionne en direct — la parade `--legacy-peer-deps` est supprimée (downgrade `@pinia/testing` mergé). À réarmer si un futur `package.json` réintroduit le conflit.

## DEC-99 → DEC-106 — Refonte scroll & logout

- **DEC-99** 🔴 **Refonte du header au scroll — sticky CSS pur.** Cause racine du flicker : `v-show`/`display:none` provoquant des sauts de hauteur en cours de scroll. Décision : **abandon de toute logique JS de scroll-hide**.
- **DEC-100** 🎓 **Régression corrigée par retrait, pas par empilement.** Quand un effet sticky introduit une régression, le retirer entièrement est souvent le bon appel plutôt que d'empiler un correctif dessus.
- **DEC-101** `.current-period` : taille et graisse réduites. Correctif cosmétique pur.
- **DEC-102** US-127 (`SyncIndicator`) confirmé livré — résout un trou de traçabilité laissé ouvert.
- **DEC-103** US-AUTH-LOGOUT mergée malgré le volet centrage non résolu — **décision PO explicite**. Le code fonctionnel est vert sur les 3 sorties ; le centrage est traité séparément.
- **DEC-104** 🔴 **Invariant auteur-test renforcé — aucune exception, même pour un test visuel « simple ».** Gemini a livré un test E2E auto-écrit pour valider son propre correctif, avec deux manquements supplémentaires : preuve fournie sans état ROUGE préalable, et assertion sur `max-height`/`overflow-y` au lieu de `position`/`boundingBox` alors que le placement était le sujet. **Test écarté intégralement, sans valeur de preuve**, malgré un code par ailleurs correct.
- **DEC-105** `US-JIKAN-HEALTHCHECK` : usage dev-only, détail par test au-delà d'un verdict global OK/KO.
- **DEC-106** Périmètre du centrage élargi à tous les popups, sur demande PO. *(Résolu par DEC-107/108.)*

## DEC-107 → DEC-112 — Overflow, grilles & fiabilité de test

- **DEC-107** 🔴 **Cause racine unique du débordement horizontal ET du décentrage des popups.** `.secondary-nav-wrapper` portait `width:100%` + `padding:10px 15px` **sans `box-sizing:border-box`** → document à 417 px dans un viewport de 387 px. Les modales en `position:fixed` se centraient sur le viewport pendant que le contenu s'étalait 30 px plus large → décalage visible d'environ 15 px. Correctif : `box-sizing:border-box` sur le wrapper + `flex:1 1 0; min-width:0` sur `.secondary-tabs button`. 🎓 **Le symptôme apparaissait très loin de sa cause.**
- **DEC-108** `US-MODAL-CENTER-AUDIT` close par DEC-107, **sans code dédié**. Le CSS des modales était conforme depuis le début.
- **DEC-109** Suppression **autorisée** des specs E2E non enregistrées. R5 protège les specs figurant au registre ; un `debug-*.spec.ts` jamais enregistré n'a aucune valeur de preuve.
- **DEC-110** `US-MODAL-UNIFY` **annulée** — la modale était déjà unique (un seul `AnimeModal`, monté une fois, routant par `modalContext`). Le vrai défaut signalé par le PO était une incohérence de **grille CSS** entre écrans de liste. 🎓 *Le diagnostic du PO pointe un symptôme, pas une cause : vérifier avant de spécifier.*
- **DEC-111** 🔴 **`.aa-card-grid` — classe partagée, colonnes fixes par breakpoint.** Deux causes racines : `.anime-grid` avec `minmax(160px,1fr)` + padding + gap réclamait 344 px sur 339 px utiles → bascule en 1 colonne pour 5 px ; et `.recs-grid` référençait une classe définie **uniquement dans le `<style scoped>` d'une autre page**, donc jamais appliquée.
- **DEC-112** Spec E2E corrigée en review : `toBeVisible` remplacé par `toHaveCount`. Une grille CSS vide (mock à `data:[]`) a une hauteur de 0 px → jugée invisible alors que le CSS est correct. Le nombre de colonnes est une propriété du **conteneur**, pas de son contenu rendu.

## DEC-113 → DEC-115 — Jikan, cache et champ `day`

### DEC-113 — Jikan : cache HIT / MISS, ni panne globale ni paramètre fautif

**9 mesures curl successives.** Le code HTTP ne dépend d'**aucun paramètre de requête** mais
de l'état du cache de Jikan : URL déjà en cache → 200 ; URL neuve → Jikan interroge
MyAnimeList → échec → 504.

| URL | Code |
|---|---|
| `anime?q=naruto&limit=1` | 200 (×2) — URL de test banale, en cache mondial |
| `anime?q=naruto&limit=25` | 504 |
| `anime?q=naruto&sfw=true&limit=1` | 504 |
| `anime?q=naruto&sfw=true&limit=25` | 504 |
| `anime?q=naruto&sfw=true&limit=25&order_by=popularity&sort=asc` | 504 |
| `anime?q=zzqxwv&limit=1` (chaîne absurde) | 504 |
| `seasons/now?limit=25` (URL réelle de l'app) | 200 (×2) |
| `seasons/now?limit=1` | 504 |
| `anime/1/recommendations` | 504 |

Corroboré par la documentation officielle Jikan (cache 24 h + rafraîchissement en tâche de
fond) et par l'issue GitHub `jikan-me/jikan-rest` **#610**, ouverte et non résolue.

**Conséquences**
- La **recherche** est structurellement KO : chaque titre tapé = URL neuve = miss = 504.
  **Aucun correctif possible côté Aanime.**
- Les **saisons** restent fonctionnelles (URLs fixes, cache chaud).
- `US-ANILIST-SEARCH` devient la **seule** voie de réparation → **condition de lancement**.
- Le standby « panne Jikan globale », porté de S33 à S38, était **faux**.

**Hypothèses écartées — ne pas réessayer :** cache local périmé · TTL non vérifié · flag dev
non câblé · `order_by=popularity` · intermittence aléatoire.

> ⛔ **Une formulation concurrente de DEC-113 a circulé** (« la panne venait de notre
> `order_by=popularity` »). Elle est **écartée** : elle est falsifiée par les mesures
> ci-dessus — `anime?q=naruto&limit=25` sans `order_by` répond 504, et
> `seasons/now?limit=1` répond 504 alors que `limit=25` répond 200. Aucune théorie
> « paramètre fautif » ne survit à ces deux mesures. *Arbitrage acté en SE-049.*

### DEC-114 — Fallback sur cache périmé : comportement voulu, non signalé

`readLocalCache` expose un booléen `stale` ; `fetchCurrentSeason` sert le cache périmé si le
fetch échoue — comportement **délibéré**, documenté en commentaire, fidèle au vanilla (il
évite une page vide en cas de panne). `error.value` est renseigné mais **jamais affiché**.
Sans cache et sans réseau → liste vide **silencieuse**.
**Décision : comportement conservé.** Dette UX enregistrée sous `US-CACHE-STALE-WARNING`.

### DEC-115 — Le champ `day` n'est produit par aucun chemin de normalisation⛔ **SUPERSEDED PAR DEC-124.** La cascade de résolution livrée en S38 pose `day` et `airsTime`
dans `normalizeAnime`. La piste « dépendant de `/anime/{id}` en 504 » était fausse : la donnée
était présente et ignorée. *Leçon : une hypothèse marquée « NON close » doit être re-testée à
sa clôture, pas citée comme un fait dans les autres documents — elle l'a été 3 fois.*
`normalizeAnime` ne produit **jamais** `day` ni `airsTime`. Or `CalendarWeekPage` filtre sur
`state === 'calendar' && day === dayClass` : **sans `day`, un anime en `state:'calendar'` est
stocké mais invisible partout** — ni dans la semaine, ni en Library.
`buildSeedEntry` pose `state` mais pas `day` ; le même bug avait déjà été corrigé une fois
sur `state` seul, la moitié `day` est restée.

⚠️ **Statut : hypothèse forte, NON close.** Le `aanime_calendar` du PO ne contient aucune
entrée `state:'calendar'`, ce qui empêche la vérification sur donnée réelle. **Ne pas
spécifier de correctif avant d'avoir tranché.**
**Piste non vérifiée :** `day` serait rempli par `syncAnimeUpdates` via `parseJSTToLocal`
depuis le `broadcast` Jikan — donc dépendant de `/anime/{id}`, endpoint en 504 (DEC-113). Si
confirmé, le défaut d'onboarding et la panne Jikan ont **la même cause aval**.


## DEC-120 → DEC-124 — Beta réelle, gouvernance Gemini, garde `day`

- **DEC-120** L'app est en **beta avec des testeurs réels**. Tout P0 sur un parcours d'entrée devient bloquant, plus seulement gênant.
- **DEC-121** **`AGENTS.md` en lecture seule pour l'agent implémenteur** (R-AGENTS-1). Mise à jour uniquement par US dédiée avec patch verbatim (R-AGENTS-2). Sens de circulation unique : AI Studio → repo → CMD. *Motif : 3ᵉ occurrence de R-SCOPE-1 sur ce fichier.*
- **DEC-122** **La gate 🔴 peut être satisfaite par un test unitaire dédié quand l'E2E est structurellement impossible.** Motivé au cas par cas, non généralisable. Cas fondateur : AUD-02 — le SDK Firestore met les écritures en file locale hors-ligne, `setDoc` résout, aucun rejet n'est observable depuis le navigateur.
- **DEC-123** Clé primaire `mal_id` conservée. `US-BETA-DATA-MIGRATION` annulée.
- **DEC-124** 🔴 **`normalizeAnime` pose `day` et `airsTime` depuis `broadcast`.** ⛔ **SUPERSEDE DEC-118.** DEC-118 interdisait de corriger le mapping au motif que J-04 le réécrirait. Elle reposait sur une prémisse fausse : la donnée de diffusion était supposée absente, alors qu'elle est présente dans `/seasons/now` **et** `/anime/{id}` (mesure SE-053) et simplement ignorée. Cascade retenue : (1) valeur existante jamais écrasée, (2) `broadcast` JST → jour + heure locale, (3) jour de semaine de `aired.from`, **sans heure** — l'absence d'heure signale honnêtement l'approximation. Le résidu (aucune source) est marqué `awaitingSchedule` et repromu par `useSync`. *Leçon : une décision « ne corrigez pas ici » doit citer la mesure qui la fonde, sinon elle survit à sa prémisse.*

## DEC-125 → DEC-126 — Fin de vie Jikan & périmètre du breaker

- **DEC-125** 🔴 **Jikan ferme en octobre 2026.** Fait apporté par le PO, absent du corpus.
  La migration AniList cesse d'être une réparation de la recherche pour devenir un **full
  switch** de toutes les lectures externes, avec échéance dure. C'est ce qui a défini le
  Sprint Goal S39. *Impact utilisateur : sans ce switch, l'app cesse purement de fonctionner
  en octobre.*
- **DEC-126** **`AUD-04` est ANNULÉE, pas reportée.** Le constat visait la contamination du
  circuit-breaker entre endpoints d'un même hôte. AniList n'expose **qu'un seul endpoint**
  (`graphql.anilist.co`) : un breaker global y est le comportement correct, pas un défaut.
  La seule leçon transposable a été gravée dans `anilistClient` : **un 429 n'incrémente
  jamais le compteur de panne** (une limite de débit n'est pas une panne).
## DEC-127 → DEC-130 — Recherche AniList, modale calendrier & gouvernance d'agent (SE-056)

- **DEC-127** **La déduplication de recherche s'applique APRÈS le tri.** Précise `DEC-60` sans la contredire : clé `mal_id` et helper unique `dedupeByMalId` inchangés, seul le point d'application se déplace. `dedupeByMalId` garde la 1ʳᵉ occurrence — appliqué après le tri, c'est la version la mieux classée qui survit (série en cours de diffusion > terminée, puis popularité). Impact utilisateur : on ne renvoie plus l'utilisateur vers Completed pour une série qu'il peut suivre aujourd'hui. Implémenté dans `useAniListApi.searchAnimeWithMeta` (`0384cc8`).
- **DEC-128** **Les micro-patchs ne passent plus par Gemini.** Tout changement ≤ 10 lignes sur 1 fichier, entièrement dictable verbatim, est produit par Claude et collé par le PO. Corollaire de gouvernance : **la dictée verbatim est réservée aux changements sans logique métier** (import, déplacement d'appel, renommage). Dès qu'une US porte une décision de comportement, la spec décrit le comportement et Gemini implémente — pour qu'un second lecteur puisse attraper une erreur de spec. Origine : la garde `day` mal placée en SE-053 était une erreur de Claude qu'une dictée verbatim aurait rendue indétectable.
- **DEC-129** **Ligne « prochain épisode » : pas de repli textuel.** La modale calendrier affiche `Next episode · Wednesday at 18:30` uniquement si `airsTime` existe et que la série n'est pas terminée. `airsTime` sans `day` → `Next episode · 18:30`. Pas d'`airsTime` → **aucune ligne**, jamais « time unknown ». Se cale sur `airsTime`, **jamais sur `day` seul**, qui peut venir du repli fautif d'`AUD-21`. Aligné sur le bloc synopsis (`v-if="synopsisText"`) : le composant se tait quand il ne sait pas. Revient sur la note SE-055 qui prévoyait « time unknown ».
- **DEC-130** ⛔ **`US-MODAL-OPEN-SEED-KEY` annulée — sans objet.** `usePersistence.loadFromDatabase` exécute au boot une **migration de clés legacy (US-133)** qui recopie `animeCalendar` → `aanime_calendar` avant lecture. Un seed E2E sur l'ancienne clé atteint donc le store, et l'enveloppe `{timestamp, data}` est exactement `ScheduleDocument`. `modal-open.spec.ts` n'a jamais été rouge. **`DEC-85` reste actif** : toute nouvelle spec sème sur `aanime_calendar`. 3ᵉ US annulée par lecture du code après `US-MODAL-UNIFY` (DEC-110) et `US-SEARCH-3` — mais la 1ʳᵉ dont la cause est une inférence de Claude inscrite en Knowledge, pas un backlog vieilli.
## DEC-131 → DEC-132 — S39, clôture

- **DEC-131** 🔴 **Pas de date de diffusion prouvée, pas de créneau.** `normalizeAniList` ne
  déduit plus jamais un jour de semaine depuis `aired_from` (première diffusion). Sans
  `nextAiringEpisode`, une série `Currently Airing` (donc `RELEASING` ou `HIATUS`) reçoit
  `awaitingSchedule = true` et sort de la grille ; `useSync` la repromeut dès qu'un `day`
  arrive. Tout autre statut : ni jour, ni attente.
  *Origine :* `AUD-21`. Une série en pause entre deux cours occupait un créneau tiré d'une date
  parfois vieille de plusieurs années. **Une case fausse ne se détecte pas à l'œil : elle
  discrédite toute la grille, pas seulement sa ligne.** Option écartée : conserver le repli pour
  les séries en cours — elle ne corrigeait pas le cas d'origine, elle le déguisait.
  ⚠️ Arme enfin le mécanisme de DEC-124, jusque-là jamais posé sur le chemin AniList
  (`awaitingSchedule` était retourné `undefined` en dur).

- **DEC-132** **`description` AniList est du HTML, jamais affichable brut.** Règle de nettoyage
  gravée dans `normalizeAniList`, en TypeScript pur, **sans DOM** : `<br>` → `\n`, dépouillement
  des autres balises, **puis** décodage des entités, `\n{3,}` → `\n\n`, `trim`. Résultat vide →
  `synopsis: undefined`, jamais `null` ni `''`.
  *L'ordre dépouillement → décodage est structurant :* une description contenant `&lt;i&gt;` veut
  afficher `<i>` littéralement. Décoder d'abord supprimerait ce texte voulu par l'auteur.
  ⚠️ N'agit qu'à la normalisation : les entrées déjà persistées restent sans synopsis jusqu'à `J10`.
### DEC-133 — Le chiffre de boot de référence est celui de la production (SE-058)

**Décision :** le temps de démarrage de référence d'Aanime est **2,5 s en production**, avec
un bundle prêt à **152 ms** et 15 requêtes. Le chiffre de **8 045 ms / 77 requêtes** porté par
les notes internes est une mesure du **serveur Vite de développement** hébergé à Singapour,
servant 59 fichiers `/src/*` un par un. Rapport 3,2 : 1.

**Conséquence :** **aucun chantier de réduction de bundle n'est ouvert.** Le découpage par
route existe et fonctionne déjà en production. La perte de 2,3 s est entièrement dans
l'orchestration du boot (910 ms mortes entre la réponse d'authentification et le chargement du
chunk de route), pas dans le poids du code.

**Pourquoi c'est inscrit ici :** c'est le même défaut que « Jikan est en panne » porté 5 sprints
sur un curl jamais rejoué — un défaut de **fraîcheur de fait**, pas de code. Un chantier de
bundle décidé sur le chiffre de dev aurait visé un fantôme.
