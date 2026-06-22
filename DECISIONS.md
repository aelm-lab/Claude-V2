# DECISIONS.md — Journal des décisions d'architecture

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** capturer le contexte « mou » pour qu'une nouvelle instance de Claude
> ne réinvente pas les décisions passées.

---

## Format
`[ID] Décision — Raison — Impact`

---

## Décisions de session 1 (Phase 0 + Phase 1)

### Scaffold / outillage

- **[DEC-01] US-001b absorbée dans US-001.** App.vue + index.html ne rentraient pas dans la limite 3 fichiers. Gemini a tout fait en une passe (5 fichiers, signalé et autorisé). → US-001b n'existe plus.

- **[DEC-02] ESLint = flat config + `@vue/eslint-config-typescript@^14`.** Le v13 est l'ancien format eslintrc, incompatible avec la flat config. → Impact : `no-explicit-any` est en erreur (prouvé en US-002).

- **[DEC-03] `tsconfig.node.json` séparé** pour isoler `vite.config.ts` (`composite: true`, `types: ['node']`).

- **[DEC-04] `npx <outil>` ≡ `npm run <script>`.** L'environnement Gemini bloque npm direct. Accepté.

- **[DEC-05] Conteneur Gemini tourne nativement en UTC.** Pas besoin de préfixe `TZ=UTC`.

- **[DEC-06] `jsdom` installé en devDep (US-010)** pour `DOMParser` dans les tests malImport. Déclaré par fichier via `// @vitest-environment jsdom`.

### Typage

- **[DEC-07] DTO de plomberie = types locaux, pas dans le contrat.** `RawAnime` et `MalImportEntry` sont locaux non exportés. `MalImportResult` est exporté mais reste près de sa fonction.

- **[DEC-08] Fixtures de test typées via helper `Partial` ou factory complet.** Interdit : `as any`, `as unknown as T`.

- **[DEC-09] US-008-types : extension du contrat (ajouts only, rien renommé).** Ajoutés : `RecSignalKind`, `RecSignal.kind?`, `HistoryItem.completedAt?`/`recencyBucket?`, et sur `AnimeEntry` : `studios?`, `popularityScore?`, `_relevanceScore?`, `_presetScore?`.

### Fidélité fonctionnelle (rec-engine)

- **[DEC-10] Branches mortes simplifiées.** Après `normalize`, `genres`/`themes`/`studios` sont toujours `string[]` → branche `.name` injoignable. Simplifiée. Comportement identique.

- **[DEC-11] Bug `item.studios` reproduit tel quel.** `scorePool` lit `item.studios` (pluriel) que `normalize` ne produit jamais (il produit `studio` singulier) → scoring studio inerte. Réparation = P8-01 / US-120. → ✅ **RÉSOLU en US-134/DEC-86** : `normalizeAnime` produit désormais toujours `studios: string[]`.

- **[DEC-12] `decayMultiplier = 0.2` conservé** dans `buildTasteProfile`.

- **[DEC-13] `priority` du tri des signaux typé `Record<RecSignalKind, number>` avec `score: 0`.**

- **[DEC-14] `extractBecauseYouWatched` : param `profile` inutilisé → préfixé `_profile`.** Signature publique préservée.

### Découpage ICS / MAL

- **[DEC-15] `generateICSFile` scindé.** `buildICSContent` (pure) dans `utils/ics.ts`. Téléchargement + toast → `useICS.ts`.

- **[DEC-16] `openMalImport` reporté.** `parseMalXml` (pur) dans `utils/malImport.ts`. Partie impure → `useMalImport.ts`.

### UX

- **[DEC-17] Dette UX boot.** `LoadingOverlay` piloté par état réactif à faire en Phase 4.

---

## Décisions de session 2 (Phase 2)

- **[DEC-18] Upsert du store : garder `if ('state' in input)`.** Ne jamais recalculer `state` inconditionnellement en branche merge, sinon clobber. Spec de Claude fautive, corrigée par Gemini.

- **[DEC-19] `needsBroadcastSync` dans `usePersistence`.** Mutation réactive → déclenche le watchDebounced. Déviation mineure plus correcte. Conservé.

- **[DEC-20] bg worker : `fetchAnimeRelations` retourne `[]` aussi bien sur « pas en cache » que « anime sans relations ».** Idempotent.

- **[DEC-21] `buildRelationMemory` provient de `recs.js`, pas de `rec-engine.js`.** Dans `useSync`, stubbée `_buildRelationMemory`. Spec de Claude fautive.

- **[DEC-22] `fetchTopFinishedAnime` inline dans `useRecommendations.fetchRecPool('library')`.** Fidèle au vanilla. Migration = P8-04 / US-123.

- **[DEC-23] `useTheme` : `useDark()` applique la classe `dark` sur `<html>`.** Divergence vanilla (`<body>`) résolue en US-104.

- **[DEC-24] `MalImportResult` expose `imported`, pas `entries`.** Spec US-018b corrigée par Gemini.

## Décisions d'orchestration Phase 2→3

- **[DEC-25] Option 2 pour l'orchestration sync.** Stubs no-ops permanents dans `usePersistence`. `App.vue` séquence directement. *(Trou révélé session 6 → DEC-50.)*

- **[DEC-26] `watch → saveToDatabase` dans `usePersistence`, pas dans le store.** Store sans I/O.

---

## Décisions de session 3 (Phase 3 + Phase 4)

- **[DEC-27] Double bloc `<script>` + `<script setup>` pour exporter un Symbol depuis App.vue.**
- **[DEC-28] Guard auth utilise `auth` singleton + `await auth.authStateReady()`.**
- **[DEC-29] Placeholders inline dans router/index.ts jusqu'à Phase 5.**
- **[DEC-30] `lastCalendarView/...` reporté Phase 4.**
- **[DEC-31] `isBooting` via `provide/inject`, fallback `ref(false)` obligatoire.**
- **[DEC-32] `syncAnimeUpdates()`/`startBackgroundRelationFetch()` fire-and-forget.** *Ajusté DEC-50.*
- **[DEC-33] `ToastNotification` dans `AppLayout` uniquement.**
- **[DEC-34] Lazy-load image via `<img style="display:none">`.**
- **[DEC-35] `SeasonNudgeCard` dismiss via `<Transition @after-leave>`.**
- **[DEC-36] `ChipsStrip` : chip 'all' émet `null`.**
- **[DEC-37] `WeekAnimeItem` reçoit `info` en prop.**
- **[DEC-38] `MonthDayCell` reçoit `animes` pré-filtrés.**

---

## Décisions de session 4 (Phases 5+6+7)

- **[DEC-39]** Substitution des placeholders router au fil des US.
- **[DEC-40]** Pattern stub `console.warn` Phase 5, câblage batché US-041.
- **[DEC-41]** `stores/ui.ts` pilote tous les overlays.
- **[DEC-42]** `modalContext` : `libraryRec` prioritaire.
- **[DEC-43]** `removeAnimeWithUndo` simplifié (undo = dette US-121).
- **[DEC-44]** Prefetch covers abandonné (fallback @error).
- **[DEC-45]** `useEpisodeInfo`/`useICS` signatures corrigées a posteriori.
- **[DEC-46]** CalendarNavControls route-aware (résorbé US-105).
- **[DEC-47]** `synopsis?` ajouté à AnimeEntry.
- **[DEC-48]** ROADMAP.md remplace PHASE8_DEBT.md.

---

## Décisions de session 6 (Audit croisé + EPIC-1)

- **[DEC-49] Audit croisé : Gemini > Claude Code.** Claude Code a raté 4 bugs runtime. → règle R3.
- **[DEC-50] Orchestration boot complète dans `App.vue` (US-102, P0).** `load → await sync → await buildRelationMemory → reScorePool → bg fetch`. Couvert par `App.spec.ts`.
- **[DEC-51] Pattern `*WithMeta` pour le throttle conditionnel (US-106).** Throttle 1,1s seulement si `fromNetwork`.
- **[DEC-52] Hiatus = source unique computed 14j (US-107).** Suppression de l'écriture morte 21j.
- **[DEC-53] `AGENTS.md` conservé et musclé (US-110).** Gouvernance permanente Gemini.
- **[DEC-54] Filet de sécurité avant correctifs (US-109).** CI + smoke test ; 3ᵉ test rouge encodant le bug P0.
- **[DEC-55] Deux documents d'architecture** (TECHNIQUE + FONCTIONNELLE).

## Décisions de session 7 (audit UX live + EPIC P0)

- **[DEC-56] Socle E2E Playwright + bypass auth mort en prod (P0.0).** `import.meta.env.VITE_E2E_AUTH_BYPASS` statique, branche éliminée du bundle (prouvé `grep -c=0`). `tests/e2e/**` exclu de Vitest.
- **[DEC-57] R4 — test E2E obligatoire sur tout correctif UX / écran.** Reproduit le geste, asserte le DOM VISIBLE. ROUGE puis VERT sans modifier le test.
- **[DEC-58] Cause racine modal morte = désalignement de nom d'event (P0.1).** `WeekAnimeItem` émet `click`, la page écoutait `@open-modal`. Fix côté PAGE (aligner les listeners), jamais renommer les emits du composant.

---

## Décisions de session 8 (suite EPIC P0 — correctifs UX)

- **[DEC-59] Boot loader en deux phases (P0.2, finding F2).** Le boot a deux fenêtres d'écran vide distinctes : **Phase 1 pré-mount** (bundle 715 kb pas encore parsé → aucun composant Vue ne peut s'afficher) et **Phase 2** (Vue monté mais `route.meta` encore vide pendant la résolution auth → `AppLayout`, donc `LoadingOverlay`, pas monté car gaté par `v-if="route.meta.requiresAuth"`). Fix : (1) loader statique HTML/CSS inline dans `index.html`, **à l'intérieur de `<div id="app">`** → Vue l'écrase au `mount()`, disparaît sans JS ; (2) `<LoadingOverlay>` remonté au **niveau racine de `App.vue`** (hors gate auth) + retiré de `AppLayout`. La séquence `onMounted` (DEC-50) est **strictement intouchée**. Couvert par `boot-loader.spec.ts`. **Correction d'un faux diagnostic de Claude :** l'`inject(isBootingKey, ref(false))` n'était PAS en cause (fallback jamais touché) — c'était un problème de **placement** (overlay piégé sous le gate auth), pas d'injection. P0.2 traite la **visibilité**, pas la **vitesse** (la durée reste = US-117 défer Firestore).

- **[DEC-60] Helper pur `dedupeByMalId` = source unique de dédup (P0.3a/c, finding F5).** F5 (doublons) avait **3 chemins indépendants**, pas un seul bug : This Season (`fetchCurrentSeason` concatène les pages sans dédup), recherche (`searchAnime` sans dédup, aggravé par `:key="mal_id"` dupliquée dans `SearchInput`), et For You (voir DEC à venir P0.3b). Décision : un helper pur générique `dedupeByMalId<T extends { mal_id?: unknown }>(items): T[]` dans `utils/helpers.ts`, **clé `mal_id` seule** (pas de fallback `id` : sur le brut Jikan seul `mal_id` existe), garde la 1ʳᵉ occurrence, items sans `mal_id` finite **conservés** (ne pas masquer un trou de données). Appliqué **avant `writeLocalCache`** (la correction persiste dans le cache 24h) dans `fetchCurrentSeason`/`fetchUpcomingSeason`, et avant tri dans `searchAnime`. Le helper est la **digue unique** : tout futur pool le réutilise.

- **[DEC-61] Contrat d'event = le composant ; les consommateurs s'alignent (P0.8a/b, application de DEC-58).** `RecCard` émet `add`/`skip`/`click`/`not-interested`/`more-like-this` ; les 3 consommateurs (DiscoverExplorePage ×2, BecauseYouWatched wrapper, LibraryExplorePage) écoutaient `@heart` (mort) et n'écoutaient ni `@click` ni `@not-interested`. Conséquence : **bouton Add mort partout + clic carte mort + « pas intéressé » mort** — tout le cœur action du moteur de reco inopérant, 0 erreur console. Fix : aligner les listeners (`@add`, `@click`→`ui.openModal`, `@not-interested`→`recommendations.dismissRec`) sur le contrat de `RecCard`, **sans jamais renommer ses emits**. Emit orphelin `open-modal` de DiscoverExplorePage (vestige d'une intention abandonnée, écouté par personne) **supprimé**. `BecauseYouWatched` (wrapper de RecCard) propage le même vocabulaire (`add` pas `heart`).

- **[DEC-62] `@more-like-this` reporté (P0.8c) — décision produit, pas technique.** `stores/ui.ts` n'a **aucun** flag `moreLikeThis` ; `openModal(anime, { info?, seasonCtx?, libraryRec? })` ne le supporte pas. Deux options réelles : modal simple (gratuit — `ModalMoreLikeThis` est déjà une section de la modal, accessible en scrollant) vs scroll-to-section (vraie feature, ajout flag store). Tranché plus tard avec le PO.

- **[DEC-63] Toasts de feedback = « destination visible exacte » (P0.4, finding F6).** Les libellés nomment l'**onglet où l'utilisateur retrouvera l'anime**, pas le jargon interne : `calendar`→« On Air », `radar`→« Coming Soon », `watchlist`→« Plan to Watch », `vault`→« Completed ». P0.4 = ajout des toasts sur `onAdd`/`onStartWatching` de `AnimeModal` (qui en manquaient ; chemin réel de la recherche : `SearchInput.onSelect`→`openModal`→bouton Add→`onAdd`). **Hors P0.4 (→ P0.4-bis) :** harmonisation des libellés jargonneux existants (`onMarkDone` « Vault », `DiscoverExplorePage` « Radar »). **Hors P0.4 (→ US-121) :** auto-vault muet au boot (`usePersistence`, chemin différent).

- **[DEC-64] Clé localStorage confirmée = `'animeCalendar'` (dette P0.1 levée).** ⚠️ **DÉPRÉCIÉE par DEC-85 (US-133)** : la clé est désormais `aanime_calendar` (toutes les clés préfixées `aanime_`, migration legacy au boot). Les tests E2E antérieurs à s15 référençant `'animeCalendar'` sont à migrer. `usePersistence.saveToDatabase` écrit `localStorage.setItem('animeCalendar', …)`. La clé devinée par Gemini dans `modal-open.spec.ts` (session 7) **était la bonne** — le test E2E modal repose sur une vraie clé, pas un hasard. Plus rien à vérifier.

- **[DEC-65] Stratégie de test E2E affinée → `AGENTS_E2E.md` (R5).** Pendant un epic, chaque US ne livre qu'un E2E **ciblé** sur ce qu'elle impacte (rouge→vert sur le bug précis). À la **fin de l'epic**, un **grand check E2E complet** rejoue toute la suite (`npx playwright test` sans filtre) = régression globale. Les tests E2E sont **cumulatifs** dans `tests/e2e/`, jamais supprimés — c'est ce qui rend le grand check possible.

**DEC-66 : libellé période = source unique CalendarNavControls (Month P0.6 + Week P0.6-bis, US-116 close)
**DEC-67 : convention classe active nav = .active (markup aligné sur CSS)
**DEC-68 : P0.7 = style pur (tokens existants, script intact) ; US-122 reste EPIC-3
**DEC-69 : P0.8c sorti P0 → EPIC-4/US-152
**DEC-70 : .modal-backdrop overlay centré (CSS manquant, pattern « markup réf une classe absente » = 3e occurrence après weekday-headers/secondary-tab). Règles préfixées pour ne pas casser .modal vanilla.
**DEC-71 : libellés toasts harmonisés (« Vault »→« Completed », « Radar »→« Coming Soon ») 


> **Note transverse session 8 (audit event-name) :** un balayage `defineEmits` vs `@listener` sur **tout** `src/components/` a confirmé que le **seul foyer de désalignement 🔴 est `RecCard`** (résolu P0.8a/b). Tout le reste (calendar, AnimeCard, chips, nudges, tous les `Modal*`) est aligné. `open-recency` est bien émis par `ModalCalendarTop` (ligne 70) et écouté par `AnimeModal` (pas un handler fantôme). La dette event-name (P0.1 + RecCard) est donc **cartographiée et close**.

> **Note de capacité (session 8) :** l'audit UX **live** (walkthrough navigateur) n'a pas pu être refait par Claude dans cette session — pas d'outil de navigateur interactif disponible, et l'app déployée est derrière l'auth AI Studio (cookie de sécurité). Les correctifs session 8 sont validés par **code + E2E** (R1/R4), pas par observation visuelle. Un audit live reste à refaire (par le PO pilotant le navigateur) avant de clore l'EPIC P0.


## Décisions de session 10 (quick wins EPIC-4 + dette DEC-72)

- **[DEC-72] Boot-loader hors `#app` + suppression DOM dans `App.vue` (déviation DEC-59 actée).** Gemini a déplacé `#boot-loader` en dehors de `<div id="app">` (contrairement à DEC-59 qui prévoyait l'intérieur). La suppression se fait via `document.getElementById('boot-loader')?.remove()` dans le `finally` du `onMounted` d'`App.vue`. Exception R-CODE-4 documentée — seule manipulation DOM directe autorisée dans `App.vue` (au même titre que `useICS`/import MAL). `main.ts` = `app.mount('#app')` simple, **sans** `router.isReady()` (testé : le `router.isReady()` introduit par Gemini cassait `boot-loader.spec.ts` → reverté). `playwright.config.ts` : ajout de `timeout: 120000` sur le webServer (sandbox lent). Ne pas « réparer » cet ensemble — il est couvert et stable.

- **[DEC-73] US-121 auto-vault muet → 2 toasts séparés On Air + Completed.** `TransitionResult` enrichi de `movedToVault: boolean`. Dans `applyLoadTransitions` : `movedToVault = true` quand passage en vault (condition `currentState !== 'vault'` préservée → pas de re-trigger sur show déjà vault). Dans `loadFromDatabase` : 2 `setTimeout` indépendants (1000ms pour « Moved X to On Air », 1500ms pour « Moved Y to Completed ») — décalage anti-collision, décision PO. `saveToDatabase` + `_syncAnimeUpdates` déclenchés si `movedToOnAirCount > 0 || movedToVaultCount > 0` (sinon auto-vault seul ne persiste pas). Toast = « Completed », jamais « Vault » (DEC-71).

- **[DEC-74] P0.3b dédup For You batch via `dedupeByMalId` sur batch sortant — moteur intouché.** Bug : `wildcards` = sous-ensemble de `remainingPool`, jamais retirés → un wildcard sort 2× (via `pIdx` + slot 5). Fix : `const fullBatch = buildNextBatch(...) as AnimeEntry[]` puis `dedupeByMalId(fullBatch).slice(0, size)` dans `getNextBatch`. Dédup **avant** slice (sinon batch à 11 au lieu de 12). `buildNextBatch` strictement intouché (fonction pure, 15 tests, cadence 1-wildcard/5 préservée). Réutilise la digue DEC-60 (`dedupeByMalId`). `mal_id === id` sur `AnimeEntry` normalisé → helper fonctionne tel quel. Preuve ROUGE : `Received: 10` (2 doublons sur 12 cartes).

- **[DEC-75] US-141 marquer-vu 1 tap depuis calendrier semaine.** `WeekAnimeItem` émet `mark-done: [anime: AnimeEntry]` avec `@click.stop` (ne déclenche pas l'ouverture modale). `CalendarWeekPage.onMarkDone` réplique la logique de `AnimeModal.onMarkDone` : `store.addAnime({ ...anime, state: 'vault', completedAt: new Date().toISOString() })` + `toast.showToast('Moved to Completed')`. Composant émet, page mute (R-CODE-3). Aucun `<style scoped>` (DEC-72). La classe `.rc-mark-done` n'a **pas** de style dans `style.css` (bouton fonctionnel mais brut) — style CSS = dette à traiter avec F18–F23 ou US dédiée après audit live.

- **[DEC-76] US-150 snap-to-today remplace le scroll-restore dans `CalendarWeekPage`.** `savedScrollY`/`onDeactivated` supprimés. Fonction `snapToToday()` : `await nextTick()` + `daysData.value.findIndex(d => d.isToday)` + `sectionRefs.value[todayIndex].scrollIntoView({ behavior: 'auto', block: 'start' })`. Garde `todayIndex < 0` (semaine sans aujourd'hui = no-op). Appelée en `onMounted` (après installation des IntersectionObservers) + `onActivated` (KeepAlive). `behavior: 'auto'` (pas de scroll animé désagréable au boot). Pas de re-snap sur navigation Prev/Next. ⚠️ Test E2E faible (jeudi + viewport réduit → faux vert potentiel) — audit live est le vrai juge.

- **[US-143 fermée sans dev]** F16 était déjà implémenté : `BecauseYouWatched.vue:3` affiche `_triggerTitle` dans le titre de section ; `RecCard.vue` affiche `_signals` (résumé ligne + panneau why au clic) + badge `_badge`. Rien à coder. Dette notée : `BecauseYouWatched.vue` a un `<style scoped>` antérieur à DEC-72 — à inclure dans la passe F18–F23.

> **Leçon process session 10 (→ ANTIPATTERNS) :** Gemini a modifié 5 fichiers sans US en début de session (cascade 80% du temps perdu). Claude a produit 2 mauvais diagnostics (mock `/anime/**` ajouté/retiré ; hypothèse Jikan écrase le store) faute d'avoir lu `modal-open.spec.ts` (le code qui marchait) avant de proposer. Zéro-confiance s'applique à Claude : lire le code qui marche AVANT de proposer, pas après 4 allers-retours.



## Session 12 — Clôture EPIC P0 (audit live PO)
- [DEC-77] BUG-5 : "Mark done" + ligne recency gatés sur isFinished (statut 'Finished') dans ModalCalendarTop. Règle produit : ces actions n'ont de sens que sur un anime terminé.
- [DEC-78] BUG-2 clos comme PERCEPTION. Test-juge mesure boundingBox .modal-content (x:20 w:360 centerX:200 sur 400px) → centrage parfait. Le ressenti d'audit = artefact du cadre devtools responsive, pas un bug CSS.
- [DEC-79] BUG-1 : réactivité Discover via dérivation, pas via canal modal→page. excludedIds = union(store.animeCalendarData, dismissedRecIds réactif). Add (store réactif) ET Dismiss (Set réassigné dans dismissRec) retirent la carte mécaniquement, quel que soit le chemin (carte ou modal). Handlers carte laissés en place (redondance inoffensive, minimise le diff/risque scope).
- [DEC-80] Conflit de règles découvert : un anime calendar+Finished s'auto-vault au boot (usePersistence), calendar+Airing gate Mark done. Le scénario "Mark done depuis Week" est donc structurellement impossible post-gating. toast-labels déplacé sur watchlist+Finished (exclu de l'auto-vault) via /library/plan. Aucun code source modifié.

---

## Décisions de sessions 13–14 (EPIC-2 + EPIC-3 amont)

> EPIC-2 (fiabilité/perf) clos en s13/s14. EPIC-3 amorcé. ⚠️ **Conflit de numéros connu**
> (US-120/122/123 réutilisés pour des sujets différents entre la table ROADMAP d'origine et
> ce qui a réellement été livré) — résolution dans `BACKLOG.md § Conflits de numéros`.

- **[DEC-... s13/14] EPIC-2 clos.** Code-splitting / défer Firestore / fiabilité livrés (détail dans les HANDOFF archivés s13/14). Build cible ~717 kb (index ~420 + firebase esm ~452).
- **[US-118] Pool réactif suggestions calendrier.** Les suggestions de remplissage des jours vides deviennent réactives. ⚠️ Effet de bord : BUG-10 (suggestions On Air intermittentes sur jours vides) — non prioritaire, backlog.
- **[US-119] Covers relations enrichies** dans la modal.
- **[US-120 — livré] Badge vault = « Finished ».** (≠ US-120 ROADMAP d'origine « bug studios », qui est devenu US-134.)
- **[US-122 — livré] Magic-link UI in-app** : re-saisie email via input dans `LoginPage` (remplace `window.prompt`).
- **[US-123 — livré] Renommage badges RecCards.** (≠ US-123 ROADMAP d'origine « extraire `fetchTopFinishedAnime` », renumérotée US-165.)

---

## Décisions de session 15 (EPIC-3 — clôture)

- **[DEC-81] MAL `Dropped` = non importé (option B).** Plus propre, évite la pollution du store. *Impact user : un anime abandonné sur MAL n'encombre pas la bibliothèque Aanime.* (Vérifier que `malImport` filtre déjà — sinon 1 US : US-124.)
- **[DEC-82] Redirect post-login = reste `/` (option B).** Le « redirect vers la route d'origine » part au Vault fonctionnalités (lien magic expire rarement → ROI faible). *Impact user : après connexion, atterrissage sur l'accueil, pas sur la page d'origine — acceptable car connexion rare.*
- **[DEC-83] US-131 skip slot-fill = session-only.** `ref<Set<number>>` local à `CalendarWeekPage`, jamais Pinia, jamais persisté. Reload → la suggestion peut revenir. *Impact user : on écarte une suggestion pour l'instant sans la bannir définitivement.*
- **[DEC-84] Cleanup groupé US-132.** `POSTER_PLACEHOLDER` = source unique `constants.ts` (4 copies inline supprimées) ; `onHiatus?` supprimé du type ; `episodeOverride` reseté à l'upsert (`input.episodeOverride ?? undefined`). *Impact user : aucun visible — fiabilité accrue.*
- **[DEC-85] Toutes les clés localStorage préfixées `aanime_` + migration legacy au boot (US-133).** Migration dans `usePersistence.loadFromDatabase`. Mapping : `animeCalendar→aanime_calendar`, `anime_sync_ts_v1→aanime_sync_ts`, `negative_removed_ids→aanime_negative_removed_ids`, `emailForSignIn→aanime_email_for_signin`, `recs_incoming_v3→aanime_recs_incoming`, `recs_library_v2→aanime_recs_library`, `season_nudges_v1→aanime_season_nudges`, `season_nudge_dismissed_v1→aanime_season_nudge_dismissed`, `seasons_now_v1→aanime_seasons_now`, `seasons_upcoming_v1→aanime_seasons_upcoming`. *Impact user : aucune perte de données existantes ; nudges dismissés conservés.* **Déprécie DEC-64.**
- **[DEC-86] `normalizeAnime` produit toujours `studios: string[]` (US-134).** Fallback singulier `studio`, filtre « Unknown » → dimension studio du scoring **active**. `studio` singulier préservé pour l'affichage. `scorePool`/`recEngine` non touchés (lisaient déjà `item.studios`). **Résout P8-01 / DEC-11.** *Impact user : recos plus pertinentes — l'app pousse d'autres animes du même studio que ceux qu'on aime.*

---

## Décision de session 16 (dual audit)

- **[DEC-87] Dual audit indépendant (Claude Code + Gemini, cadre identique) — méthode et résultats.**
  - **Méthode** : deux audits qui ne se parlent pas, **même cadre** (7 axes, barème P0/P1/P2, format tableau `ID | Axe | Finding | Gravité | fichier:ligne | Impact user`) pour rendre les rapports comparables ligne à ligne. C'est ce qui a permis de capter ce que chacun ratait : Claude Code a vu le bug `Continuing→Finished` (raté par Gemini) ; Gemini a vu `saveSchedule` sans try/catch (raté par Claude Code).
  - **5 vérifications zéro-confiance tranchées par lecture du code réel** :
    1. `getCardStatus` ne gère pas `'Continuing'` → tombe sur `Finished` par défaut. **CONFIRMÉ** → US-154 (P1).
    2. `saveToDatabase`/`saveSchedule` **sans try/catch**. **CONFIRMÉ** → US-153 (**P0**).
    3. `setAllData` **n'existe pas** (seul `setData` + `clearAll`). Point de vigilance du handoff = fantôme. **REJETÉ.**
    4. `syncStatus` = **0 hit** ; `reconcileWithDatabase` **n'existe plus** (réconciliation dans `loadFromDatabase`). Handoff S15 périmé sur ce point. **CORRIGÉ.**
    5. `getElementById('boot-loader').remove()` agit sur un élément d'`index.html` (loader pré-Vue) → pattern **légitime** (DEC-72). **REJETÉ comme bug**, reclassé P2 cosmétique.
    - `import idb dynamique inutile` : non matérialisé (import statique). **NON-PROBLÈME** (confirmé par les deux audits).
  - **Backlog priorisé issu de l'audit** : US-153 (P0 try/catch save) · US-154 (P1 Continuing→Airing) · US-155 (P1 boot non bloquant) · US-156 (P1 tests unit composables) · US-157 (P1 persistance mute le store hors action) · US-158 (P1 cast legacy normalisé) · US-159-CLEANUP (P2 fichiers parasites). Détail dans `BACKLOG.md` et `AUDIT.md §3`.
