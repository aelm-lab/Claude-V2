# DECISIONS.md — Journal des décisions d'architecture

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** capturer le contexte « mou » pour qu'une nouvelle instance de Claude
> ne réinvente pas les décisions passées.
>
> **État de référence : fin S33 (cleaning S34).** Séquence **DEC-01 → DEC-106**.
> **Correction de numérotation (S34)** : l'entrée historique non numérotée
> `[DEC-... s13/14]` devient officiellement **`DEC-80bis`** (convention déjà utilisée dans
> le projet pour les insertions tardives — cf. P0.6-bis/ter). Aucun autre DEC déjà référencé
> ailleurs (DEC-81→DEC-98) n'a été renuméroté, pour ne pas invalider les renvois existants
> dans `TYPES_CONTRACT.md`/`CLAUDE.md`/`ARCHITECTURE_TECHNIQUE.md`.
> ⚠️ **DEC-75 non capturé** dans cette régénération (trou de lecture entre DEC-74 et DEC-76,
> probablement session 10) — ne pas inventer son contenu, laisser le trou visible.

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
- **[DEC-48]** ROADMAP.md remplace PHASE8_DEBT.md. *(Historique — `ROADMAP.md` n'existe plus, supprimé depuis, cf. `CLAUDE.md` §10.)*

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

- **[DEC-59] Boot loader en deux phases (P0.2, finding F2).** Le boot a deux fenêtres d'écran vide distinctes : **Phase 1 pré-mount** (bundle pas encore parsé → aucun composant Vue ne peut s'afficher) et **Phase 2** (Vue monté mais `route.meta` encore vide pendant la résolution auth → `AppLayout`, donc `LoadingOverlay`, pas monté car gaté par `v-if="route.meta.requiresAuth"`). Fix : (1) loader statique HTML/CSS inline dans `index.html`, à l'intérieur de `<div id="app">` → Vue l'écrase au `mount()` ; (2) `<LoadingOverlay>` remonté au niveau racine de `App.vue` (hors gate auth). La séquence `onMounted` (DEC-50) est strictement intouchée. Couvert par `boot-loader.spec.ts`. **Correction d'un faux diagnostic de Claude :** ce n'était pas un problème d'`inject` mais de **placement**.
- **[DEC-60] Helper pur `dedupeByMalId` = source unique de dédup (P0.3a/c, finding F5).** F5 (doublons) avait 3 chemins indépendants (This Season, recherche, For You). Décision : un helper pur générique `dedupeByMalId<T extends { mal_id?: unknown }>(items): T[]` dans `utils/helpers.ts`, clé `mal_id` seule, garde la 1ʳᵉ occurrence, items sans `mal_id` finite conservés. Appliqué avant `writeLocalCache` et avant tri dans `searchAnime`. Digue unique — tout futur pool le réutilise.
- **[DEC-61] Contrat d'event = le composant ; les consommateurs s'alignent (P0.8a/b, application de DEC-58).** `RecCard` émet `add`/`skip`/`click`/`not-interested`/`more-like-this` ; les 3 consommateurs écoutaient `@heart` (mort) et n'écoutaient ni `@click` ni `@not-interested`. Conséquence : bouton Add mort partout + clic carte mort + « pas intéressé » mort — 0 erreur console.
- **[DEC-62]** *(P0.4 — feedback d'ajout)* Toute action d'ajout/déplacement visible produit un toast nommant la destination visible exacte.
- **[DEC-63]** Libellés de toast harmonisés côté vocabulaire visible (« Radar »/« Vault » internes → onglets réels).

> **Note transverse session 8 (audit event-name) :** un balayage `defineEmits` vs `@listener` sur **tout** `src/components/` a confirmé que le **seul foyer de désalignement 🔴 est `RecCard`** (résolu P0.8a/b). Tout le reste est aligné.
>
> **Note de capacité (session 8) :** l'audit UX **live** n'a pas pu être refait par Claude cette session (pas d'outil navigateur interactif, app derrière l'auth). Correctifs validés par code + E2E (R1/R4), pas par observation visuelle. Audit live restant à faire par le PO (réalisé en session 12).

---

## Décisions de session 9 (correctifs UX suite — P0.5→P0.9)

- **[DEC-64] Clé localStorage `'animeCalendar'` confirmée.** `usePersistence.saveToDatabase` écrit `localStorage.setItem('animeCalendar', …)`. La clé devinée par Gemini dans `modal-open.spec.ts` (session 7) était la bonne — le test E2E modal repose sur une vraie clé, pas un hasard.
- **[DEC-65] Stratégie de test E2E affinée → `AGENTS_E2E.md` (R5).** Pendant un epic, chaque US ne livre qu'un E2E ciblé sur ce qu'elle impacte (rouge→vert sur le bug précis). Fin d'epic : grand check E2E complet, suite complète rejouée. Tests cumulatifs dans `tests/e2e/`, jamais supprimés.
- **[DEC-66]** Libellé période = source unique `CalendarNavControls` (Month P0.6 + Week P0.6-bis, US-116 close).
- **[DEC-67]** Convention classe active nav = `.active` (markup aligné sur CSS).
- **[DEC-68]** P0.7 = style pur (tokens existants, script intact) ; US-122 reste EPIC-3.
- **[DEC-69]** P0.8c sorti du périmètre P0 → EPIC-4/US-152.
- **[DEC-70]** `.modal-backdrop` overlay centré (CSS manquant, 3ᵉ occurrence du pattern « markup référence une classe absente » après weekday-headers/secondary-tab). Règles préfixées pour ne pas casser `.modal` vanilla. *(⚠️ Rouvert partiellement S33 sur `LogoutConfirmModal` — voir DEC-106.)*
- **[DEC-71]** Libellés toasts harmonisés (« Vault »→« Completed », « Radar »→« Coming Soon »).

---

## Décisions de session 10 (quick wins EPIC-4 + dette DEC-72)

- **[DEC-72] Boot-loader hors `#app` + suppression DOM dans `App.vue` (déviation DEC-59 actée).** Gemini a déplacé `#boot-loader` en dehors de `<div id="app">`. La suppression se fait via `document.getElementById('boot-loader')?.remove()` dans le `finally` du `onMounted` d'`App.vue`. Exception R-CODE-4 documentée. `main.ts` = `app.mount('#app')` simple, **sans** `router.isReady()` (testé : cassait `boot-loader.spec.ts` → reverté). `playwright.config.ts` : `timeout: 120000` ajouté.
- **[DEC-73]** Toast « Moved to Completed » au boot pour l'auto-vault (US-121).
- **[DEC-74]** Dédup via `dedupeByMalId` appliquée **avant** `slice` dans le pipeline `getNextBatch` (pas après).
- ⚠️ **[DEC-75] non capturé** dans cette régénération — trou de lecture, à reconstruire depuis les handoffs archivés session 10 si besoin.
- **[DEC-76] US-150 snap-to-today remplace le scroll-restore dans `CalendarWeekPage`.** `savedScrollY`/`onDeactivated` supprimés. Fonction `snapToToday()` : `await nextTick()` + `daysData.value.findIndex(d => d.isToday)` + `sectionRefs.value[todayIndex].scrollIntoView({ behavior: 'auto', block: 'start' })`. Garde `todayIndex < 0` (semaine sans aujourd'hui = no-op). Appelée en `onMounted` + `onActivated` (KeepAlive). `behavior: 'auto'`. Pas de re-snap sur navigation Prev/Next. ⚠️ Test E2E faible (jeudi + viewport réduit → faux vert potentiel) — audit live est le vrai juge.

**[US-143 fermée sans dev]** F16 était déjà implémenté : `BecauseYouWatched.vue:3` affiche `_triggerTitle` dans le titre de section ; `RecCard.vue` affiche `_signals` (résumé ligne + panneau why au clic) + badge `_badge`. Rien à coder. Dette notée : `BecauseYouWatched.vue` a un `<style scoped>` antérieur à DEC-72 — à inclure dans une passe CSS.

> **Leçon process session 10 (→ ANTIPATTERNS) :** Gemini a modifié 5 fichiers sans US en début de session (cascade 80% du temps perdu). Claude a produit 2 mauvais diagnostics faute d'avoir lu `modal-open.spec.ts` (le code qui marchait) avant de proposer. Zéro-confiance s'applique à Claude : lire le code qui marche AVANT de proposer, pas après 4 allers-retours.

---

## Session 12 — Clôture EPIC P0 (audit live PO)

- **[DEC-77]** BUG-5 : "Mark done" + ligne recency gatés sur `isFinished` (statut 'Finished') dans `ModalCalendarTop`. Règle produit : ces actions n'ont de sens que sur un anime terminé.
- **[DEC-78]** BUG-2 clos comme PERCEPTION. Test-juge mesure boundingBox `.modal-content` (x:20 w:360 centerX:200 sur 400px) → centrage parfait sur `AnimeModal`. Le ressenti d'audit = artefact du cadre devtools responsive, pas un bug CSS. *(⚠️ Ne pas extrapoler cette clôture à d'autres modales — voir DEC-106, `LogoutConfirmModal` rouvre la question S33.)*
- **[DEC-79]** BUG-1 : réactivité Discover via dérivation, pas via canal modal→page. `excludedIds = union(store.animeCalendarData, dismissedRecIds réactif)`. Add (store réactif) ET Dismiss (Set réassigné dans `dismissRec`) retirent la carte mécaniquement, quel que soit le chemin. Handlers carte laissés en place (redondance inoffensive).
- **[DEC-80]** Conflit de règles découvert : un anime calendar+Finished s'auto-vault au boot (`usePersistence`), calendar+Airing gate Mark done. Le scénario "Mark done depuis Week" est donc structurellement impossible post-gating. Toast-labels déplacé sur watchlist+Finished (exclu de l'auto-vault) via `/library/plan`. Aucun code source modifié.

## Décisions de sessions 13–14 (EPIC-2 + EPIC-3 amont)

> EPIC-2 (fiabilité/perf) clos en s13/s14. EPIC-3 amorcé. ⚠️ **Conflit de numéros connu**
> (US-120/122/123 réutilisés pour des sujets différents entre la table ROADMAP d'origine et
> ce qui a réellement été livré) — résolu par les notes ci-dessous, à titre définitif.

- **[DEC-80bis] EPIC-2 clos.** *(Anciennement `[DEC-... s13/14]`, non numéroté — corrigé au cleaning S34.)* Code-splitting / défer Firestore / fiabilité livrés (détail dans les handoffs archivés s13/14). Build cible ~717 kb (index ~420 + firebase esm ~452) — **chiffre historique, périmé, voir `STATE.md` pour la taille actuelle**.
- **[US-118] Pool réactif suggestions calendrier.** Les suggestions de remplissage des jours vides deviennent réactives. ⚠️ Effet de bord : BUG-10 (suggestions On Air intermittentes sur jours vides) — non prioritaire, backlog.
- **[US-119] Covers relations enrichies** dans la modal.
- **[US-120 — livré] Badge vault = « Finished ».** (≠ US-120 ROADMAP d'origine « bug studios », qui est devenu US-134.)
- **[US-122 — livré] Magic-link UI in-app** : re-saisie email via input dans `LoginPage` (remplace `window.prompt`).
- **[US-123 — livré] Renommage badges RecCards.** (≠ US-123 ROADMAP d'origine « extraire `fetchTopFinishedAnime` », renumérotée US-165.)

---

## Décisions de session 15 (EPIC-3 — clôture)

- **[DEC-81] MAL `Dropped` = non importé (option B).** Plus propre, évite la pollution du store. *Impact user : un anime abandonné sur MAL n'encombre pas la bibliothèque Aanime.* (À vérifier au besoin : US-124.)
- **[DEC-82] Redirect post-login = reste `/` (option B).** Le « redirect vers la route d'origine » part au Vault fonctionnalités (lien magic expire rarement → ROI faible). *Impact user : après connexion, atterrissage sur l'accueil — acceptable car connexion rare.*
- **[DEC-83] US-131 skip slot-fill = session-only.** `ref<Set<number>>` local à `CalendarWeekPage`, jamais Pinia, jamais persisté. Reload → la suggestion peut revenir. *Impact user : on écarte une suggestion pour l'instant sans la bannir définitivement.*
- **[DEC-84] Cleanup groupé US-132.** `POSTER_PLACEHOLDER` = source unique `constants.ts` (4 copies inline supprimées) ; `onHiatus?` supprimé du type ; `episodeOverride` reseté à l'upsert. *Impact user : aucun visible — fiabilité accrue.*
- **[DEC-85] Toutes les clés localStorage préfixées `aanime_` + migration legacy au boot (US-133).** Migration dans `usePersistence.loadFromDatabase`.
- **[DEC-86] `studios: string[]` toujours peuplé par `normalizeAnime` (US-134).** Résout la dette P8-01 (dimension studio du scoring inerte). *Impact user : les recommandations tiennent enfin compte du studio.*

## Décisions de session 16 (dual audit — zéro-confiance)

- **[DEC-87] Vérifications zéro-confiance de l'audit croisé s16.**
  1. `setAllData` **n'existe pas** (seul `setData` + `clearAll`). Point de vigilance d'un handoff antérieur = fantôme. **REJETÉ.**
  2. `syncStatus` = **0 hit** ; `reconcileWithDatabase` **n'existe plus** (réconciliation dans `loadFromDatabase`). Handoff antérieur périmé sur ce point. **CORRIGÉ.**
  3. `getElementById('boot-loader').remove()` agit sur un élément d'`index.html` (loader pré-Vue) → pattern **légitime** (DEC-72). **REJETÉ comme bug**, reclassé P2 cosmétique.
  4. `import idb dynamique inutile` : non matérialisé (import statique). **NON-PROBLÈME** (confirmé par les deux audits).
  - **Backlog priorisé issu de l'audit** : US-153 (P0 try/catch save) · US-154 (P1 Continuing→Airing) · US-155 (P1 boot non bloquant) · US-156 (P1 tests unit composables) · US-157 (P1 persistance mute le store hors action) · US-158 (P1 cast legacy normalisé) · US-159-CLEANUP (P2 fichiers parasites). Tous livrés S19/S21, cf. `EPICS.md` EPIC 8.

---

## Décisions de session 28 (Epic Stats)

- **[DEC-88] `useStats` = composable dédié, `StatsPage.vue` = page pure.** La logique d'agrégation
  (compte par année, top genres) vit dans `useStats`, la page ne fait qu'afficher. Respect strict
  de la séparation des couches. *Impact user : aucun visible — fondation propre pour enrichir les stats.*
- **[DEC-89] `topGenres` scoped à `completedThisYear` uniquement.** Un « year-in-review » ne compte
  que le contenu **consommé** (terminé), pas la watchlist ni le radar. Benchmark validé contre
  **AniList / Spotify Wrapped / Letterboxd**. *Impact user : les stats reflètent ce que tu as
  vraiment regardé cette année, pas tes intentions.*
- **[DEC-90] Garde null-safety `genres ?? []` sur le chemin Stats.** Un cache legacy peut contenir
  des entrées sans `genres` → `topGenres` crashait. Garde runtime ajoutée. *Impact user : la page
  Stats ne plante plus sur un vieux cache.*
- **[DEC-91] Route `/stats` derrière le guard auth.** Les stats sont personnelles → réservées à
  l'utilisateur connecté. Onglet ajouté dans `PrimaryNav.vue`. *Impact user : accès aux stats via
  la nav principale, après connexion.*

---

## Décisions de session 29 (Polish & confiance)

- **[DEC-92] BUG-1 / BUG-2 / BUG-4 fermés sans spec (morts en prod).** Audit live PO (R6) : les 3
  bugs sont **invérifiables en prod** (déjà corrigés ou jamais reproductibles). Aucune ligne de code
  touchée. *Impact user : aucun — confirmation que le ressenti d'audit antérieur était périmé.*
- **[DEC-93] US-BUG5 = fix présentationnel (`v-if`), pas de touche logique.** Le bouton ✓ « Mark done »
  est **masqué** sur statut Airing/Hiatus dans `WeekAnimeItem.vue`. L'action reste disponible via la
  modale. R4-bis appliqué. *Impact user : on ne propose plus « terminé » sur un anime en diffusion — cohérence.*
- **[DEC-94] Règle de titre centralisée dans `getAnimeTitle`.** Anglais primaire + romaji secondaire
  si différent. Rollout progressif (search fait en S29 ; modale / RecCards / carte semaine = backlog).
  *Impact user : titres cohérents et lisibles, d'abord dans la recherche.*
- **[DEC-95] Vocabulaire search figé (P0.4).** « Coming Soon » (jamais « Upcoming ») / « Finished airing »
  (jamais « Finished »). *Impact user : vocabulaire constant entre les surfaces.*
- **[DEC-96] ✓ Added = cliquable.** Un clic sur l'état « Added » d'une suggestion **retire** l'anime
  d'où qu'il soit + toast « Removed ». *Impact user : on annule un ajout en un clic, sans ouvrir de modale.*
- **[DEC-97] Couleurs = réutilisation des tokens, pas de redéclaration.** Les statuts search réutilisent
  `var(--airing)` / `var(--upcoming)`. *Impact user : aucun visible — dette CSS évitée (sauf un
  `#10b981` en dur résiduel, cf. backlog).*

---

## Décision infrastructure (S22→S29, à dater précisément)

- **[DEC-98] `npm install` direct — parade `--legacy-peer-deps` supprimée.** Le downgrade
  `@pinia/testing` a été mergé → le conflit de peer-deps disparaît. *Impact dev : commande
  d'install simplifiée. À ré-armer si un futur `package.json` réintroduit le conflit.*

---

## Décisions groupées S30 → S33 (append cleaning S34)

> Regroupées faute de granularité per-sprint plus fine dans le handoff disponible — voir
> `STATE.md §Trous connus`. Ne pas sur-attribuer une décision à un sprint précis sans confirmation.

- **[DEC-99] Refonte scroll header — Piste A retenue (sticky CSS pur).** Cause racine du
  flicker secondary-nav diagnostiquée : `v-show`/`display:none` provoque des sauts de hauteur
  en cours de scroll. Décision : abandon de toute logique JS de scroll-hide au profit d'un
  sticky CSS pur. Confirmé : élimine le flicker. *Impact user : navigation fluide au scroll,
  plus de saut visuel sur la nav secondaire.*
- **[DEC-100] Régression chips Discover corrigée par retrait du sticky (pas par un nouveau
  correctif).** Robustesse > effet : quand un effet sticky/scroll introduit une régression, le
  retirer entièrement est souvent le bon appel plutôt que d'empiler un correctif dessus.
  *Impact user : les chips de filtre Discover redeviennent visibles sous la nav.*
- **[DEC-101] N9 — `.current-period` : taille/graisse de police réduites.** Correctif cosmétique
  pur. *Impact user : le libellé de la période de calendrier n'écrase plus visuellement la nav.*
- **[DEC-102] US-127 confirmé livré.** Résout le trou laissé ouvert fin S29 (« présent en
  mémoire ~S28, absent du backlog S30 ») — le SyncIndicator sur `startBackgroundRelationFetch`
  est bien en production. *Impact user : aucun changement de comportement, juste une
  confirmation de traçabilité.*
- **[DEC-103] US-AUTH-LOGOUT mergée malgré le volet centrage non résolu — décision PO explicite.**
  Le code fonctionnel (`signOut`, `LogoutConfirmModal`, câblage `AppHeader`) est vert sur les
  3 sorties de porte locale. Le signalement de centrage sur `LogoutConfirmModal` est traité
  comme un sujet **séparé et élargi** (`US-MODAL-CENTER-AUDIT`, voir DEC-106), pas comme un
  blocage de cette US précise. *Impact user : la fonctionnalité de déconnexion est disponible
  immédiatement ; un éventuel défaut de centrage reste à confirmer/corriger séparément.*
- **[DEC-104] Invariant auteur-test renforcé — aucune exception, même pour un test visuel
  "simple".** Gemini a livré un test E2E auto-écrit pour valider son propre correctif de
  centrage → violation de l'invariant non-négociable du projet (auteur du test ≠ auteur du
  code), en plus de deux autres manquements : preuve E2E fournie sans état ROUGE préalable, et
  test assertant `max-height`/`overflow-y` au lieu de `position`/`boundingBox` alors que le
  placement était le sujet (violation explicite d'un interdit déjà écrit dans `AGENTS_E2E.md`).
  Le test a été écarté intégralement, sans valeur de preuve. *Impact user : aucun direct — fiabilité
  du process de validation, protège contre des correctifs visuels non prouvés qui atteindraient la prod.*
- **[DEC-105] US-JIKAN-HEALTHCHECK — refinement acté : usage dev-only, détail par test.**
  Pas de signal visible utilisateur ; le healthcheck expose un détail par test au-delà d'un
  simple verdict global OK/KO, pour faciliter le diagnostic quand Jikan est instable.
  *Impact user : aucun visible — outil de diagnostic interne.*
- **[DEC-106] Création de `US-MODAL-CENTER-AUDIT` — périmètre élargi à tous les popups et au
  site en général, sur demande PO explicite.** Ne pas traiter isolément le centrage de
  `LogoutConfirmModal` : les 3 modales (`AnimeModal`, `LogoutConfirmModal`, `RecEngineModal`)
  partagent la classe `.modal-backdrop`, **sans aucune définition de style dans les 3
  templates qui la référencent** — le CSS réel du centrage vient d'un fichier global non
  encore localisé (`findstr` prévu, jamais exécuté). DEC-78 (session 12) avait clos un
  signalement similaire comme perception d'audit, mais **spécifiquement sur `AnimeModal`** —
  cette clôture ne doit pas être extrapolée à `LogoutConfirmModal` sans vérification propre
  (R3). *Impact user : à déterminer une fois l'audit fait — potentiellement des popups mal
  centrés sur mobile pour un sous-ensemble d'utilisateurs, à confirmer avant de conclure.*
