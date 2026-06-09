# PHASE8_DEBT.md — Post-migration : dette technique + features

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Rôle :** recenser tout ce qui est volontairement reporté APRÈS la fin de la
> migration (branche stable, Phase 7 terminée). Rien ici ne bloque la migration.
> Chaque item sera transformé en US métier au moment voulu.
>
> **Règle d'alimentation :** dès qu'on reproduit un comportement « fidèle mais
> douteux », ou qu'on identifie une amélioration, on l'ajoute ici plutôt que
> de le corriger en douce pendant la migration.

---

## Format
`[P8-XX] Titre — Description — Origine`

---

## Dette technique (bugs reproduits fidèlement)

### [P8-01] Bug studios inerte
**Description :** `scorePool` dans `utils/recEngine.ts` lit `item.studios` (pluriel) pour le scoring studio. Or `normalizeAnime` produit `studio` (singulier). Résultat : la dimension studio du scoring est toujours inerte — les studios ne contribuent jamais au score de recommandation.
**Comportement actuel :** reproduit fidèlement (règle de migration : copier le comportement, pas le corriger).
**Correction à faire :** dans `scorePool`, lire `item.studio` (singulier) ou normaliser vers `studios[]`. Mesurer l'impact sur les scores avant/après.
**Origine :** DEC-11 (session 1)

### [P8-04] `fetchTopFinishedAnime` inline dans `useRecommendations`
**Description :** Le fetch `/anime?min_score=7.5&order_by=members&sort=desc&limit=25` (pool library) est codé inline dans `useRecommendations.fetchRecPool('library')`. `TYPES_CONTRACT.md §7` le mentionne comme méthode `fetchTopFinishedAnime` de `useJikanApi`, mais cette méthode n'a jamais été implémentée dans `useJikanApi`.
**Correction à faire :** extraire vers `useJikanApi.fetchTopFinishedAnime()`, mettre à jour `fetchRecPool` pour l'appeler.
**Origine :** DEC-22 (session 2)

---

## Dette UX (comportements fidèles mais discutables)

### [P8-02] Auto-vault silencieux au premier ajout
**Description :** Ajouter une série déjà terminée (`Finished Airing`) la fait atterrir directement dans *Completed* (vault) sans aucun feedback. L'utilisateur peut ne pas comprendre pourquoi la série n'apparaît pas dans son calendrier.
**Correction à faire :** afficher un toast informatif au moment de l'auto-vault — ex. `"Added to Completed (already finished airing)"` — avec éventuellement un bouton « Move to Watchlist » (undo).
**Composant concerné :** à implémenter dans les actions du store ou dans le composant appelant `addAnime`.
**Origine :** cadrage US-011 (session 2)

### [P8-03] `window.prompt()` pour re-saisie email
**Description :** Dans `useFirebaseAuth.completeSignIn()`, si `emailForSignIn` est absent du localStorage (ex. l'utilisateur a changé de navigateur), on appelle `window.prompt()` pour demander l'email. C'est brutal (surtout mobile) et non stylisable.
**Correction à faire :** exposer un état `needsEmailConfirmation: Ref<boolean>` dans `useFirebaseAuth`, que `LoginPage.vue` utilisera pour afficher un input de re-saisie dans l'UI.
**Origine :** US-012 cadrage (session 2)

### [P8-05] Mapping MAL `Dropped` → `vault`
**Description :** Lors de l'import MAL, les animes avec statut `Dropped` sont mappés vers `vault` (Completed). Un anime abandonné n'est pas vraiment « complété » — l'utilisateur pourrait être surpris de le trouver dans Completed.
**Options à évaluer :**
  - Créer un état `dropped` dédié (changement de modèle, impact fort).
  - Mapper vers `watchlist` à la place (plus neutre).
  - Garder `vault` mais afficher un badge « Dropped » (cosmétique).
**Origine :** US-018b (session 2)

---

## Features à ajouter post-migration

### [P8-06] `LoadingOverlay` réactif au boot
**Description :** Au démarrage, l'app charge auth + Firestore avant le premier rendu → risque de flash blanc ou d'état vide visible. `App.vue` expose déjà `isBooting` via `provide/inject` (DEC-31). `LoadingOverlay` est câblé et injecte `isBootingKey`. Fonctionnel dès Phase 4.
**Statut :** implémenté (US-022 + US-023). Surveiller le comportement au boot en Phase 5.
**Origine :** DEC-17 (session 1)

### [P8-07] `hideToast()` sur clic utilisateur
**Description :** `useToast` expose `hideToast()`. `ToastNotification.vue` branche déjà `@click="hideToast"` (US-023). Amélioration disponible dès Phase 4.
**Statut :** implémenté en US-023.
**Origine :** US-018a cadrage (session 2)

### [P8-08] Clés localStorage incohérentes — harmonisation
**Description :** Les clés localStorage actuelles sont hétérogènes (`backlog_recs_v1`, `recs_incoming_v3`, `recs_library_v2`, `seasons_now_v1`, `seasons_upcoming_v1`, `anime_sync_ts_v1`, `animeCalendar`, `lastCalendarView`, etc.). À harmoniser en Phase 7 avec une convention `aanime_*`.
**Contrainte :** migration des données existantes ou reset accepté (à décider avec le PO).
**Origine :** AUDIT.md + PLAN_MIGRATION.md Phase 7

### [P8-09] Redirect post-login vers la route d'origine
**Description :** Après `completeSignIn()`, l'app redirige toujours vers `/week`. Si l'utilisateur tentait d'accéder à `/library/completed` avant d'être renvoyé sur `/login`, il perd ce contexte.
**Correction à faire :** implémenter un `redirect` query param (`/login?redirect=/library/completed`) sauvegardé dans le guard `beforeEach`, lu dans `LoginPage.vue` après auth réussie.
**Origine :** décision PO US-021 (session 3) — fidélité vanilla prioritaire sur UX.

### [P8-10] SyncIndicator — couverture complète des fetches Jikan
**Description :** `SyncIndicator` réagit uniquement à `useSync().isSyncing` (batch sync). Le vanilla monkey-patchait `window.fetch` pour détecter TOUS les fetches vers `api.jikan.moe` (search, fetchById, season…). Les fetches ponctuels ne déclenchent pas l'indicateur dans la version Vue.
**Correction à faire :** ajouter un `ref<number> jikanInFlight` dans `useJikanApi`, incrémenté/décrémenté autour de chaque fetch. Exposer `isJikanBusy: ComputedRef<boolean>`. `SyncIndicator` combinerait `isSyncing || isJikanBusy`.
**Origine :** décision PO US-023 (session 3).

---

## Notes techniques diverses

### DEC-23 — Classe dark sur `<html>` vs `<body>`
`useDark()` de `@vueuse/core` applique la classe `dark` sur `<html>`. Le vanilla appliquait `dark-mode` sur `<body>`. En Phase 7, vérifier les sélecteurs CSS de `style.css` qui ciblent `body.dark-mode` et les adapter en `html.dark` (ou configurer `useDark({ selector: 'body' })`).

### Commentaire résiduel `useSync.ts:177`
Le commentaire `// Toasts promotions (showToast stub dans usePersistence jusqu'à US-018)` est devenu obsolète après US-018c. Nettoyage en Phase 7.

### `getAnimeEpisodeInfo` — signature stricte `targetDate: Date`
L'util sous-jacent exige `targetDate` obligatoire. `useEpisodeInfo` fournit `new Date()` si omis. Ne pas modifier la signature de l'util — c'est voulu pour la testabilité.

### Toast sur `/login` — migration future
`ToastNotification` est dans `AppLayout` (routes auth uniquement). Si un toast doit apparaître sur `/login`, déplacer vers `App.vue`. Dette Phase 7 (DEC-33).
