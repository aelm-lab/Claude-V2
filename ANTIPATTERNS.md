# ANTIPATTERNS.md — Liste vivante des pièges Gemini

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** mémoire des erreurs récurrentes. Claude réinjecte la liste dans
> chaque nouvelle US. À chaque review qui révèle une récidive, Claude ajoute une ligne.

---

## Anti-patterns architecturaux (issus de l'audit + tentative ratée)

### Architecture
- ❌ Mettre de la logique métier ou un `fetch` dans un composant `.vue` → composable.
- ❌ Accéder au DOM directement (`document.getElementById`, `querySelector`, `appendChild`) dans du code Vue → `ref` / `v-if` / `v-for`.
  - *Nuance validée : `DOMParser` + `querySelector` pour parser une string XML (parseMalXml) est pur et autorisé.*
  - *Nuance validée : `document.createElement('a')` + `click()` uniquement pour le download Blob dans `useICS` est l'exception acceptée.*
- ❌ Stocker des handlers ou de l'état sur `window` → `onMounted`/`onUnmounted` + `@vueuse/core`.
- ❌ Recréer un bus d'événements avec `document.dispatchEvent`/`CustomEvent` → store Pinia ou `emit`.
- ❌ Manipuler `document.body.classList` / injecter un bandeau dans le `<body>` → état réactif + composant.
- ❌ Mettre `initializeApp`/`getAuth` dans un composable → réinit. Singleton dans `lib/firebase.ts`.
- ❌ Brancher `onAuthStateChanged` dans la fonction `useXxx()` → listeners empilés. Au niveau module.
- ❌ Brancher `watchDebounced` dans le corps du composable sans flag → listeners empilés. Flag niveau module.
- ❌ Appeler `useFirebaseAuth()` dans un guard Vue Router → hors contexte `setup`. Singleton `auth` + `await auth.authStateReady()`.
- ❌ Lire `auth.currentUser` avant `await auth.authStateReady()` → `null` pendant l'init.
- ❌ Importer un composant `.vue` de page/layout inexistant dans le router → `TS2307`. Placeholders inline jusqu'à Phase 5.

### TypeScript
- ❌ `any` (implicite ou explicite), `as any`, `@ts-ignore` → typer depuis TYPES_CONTRACT.md.
- ❌ Inventer une interface qui existe déjà dans TYPES_CONTRACT.md.
- ❌ Champs optionnels traités comme garantis (oublier `?.` ou les `null`).
- ❌ Fixtures de test via `as unknown as AnimeEntry` → factory `makeAnime(Partial<AnimeEntry>)`.
- ❌ `inject(key)` sans fallback quand la clé est typée → `inject(isBootingKey, ref(false))`.
- ❌ Export nommé depuis `<script setup>` → impossible. Double bloc `<script>` + `<script setup>` (DEC-27).

### Gestion d'erreur
- ❌ `async` sans `try/catch`.
- ❌ Avaler une erreur en silence (`catch {}`) sans log ni état.
- ❌ Oublier la gestion du 429 (rate limit Jikan) et du retry/backoff.
- ❌ Laisser remonter le `throw` de `handleFirestoreError` → attraper localement, exposer `error` réactif.

### Composants
- ❌ `new Image()` pour le lazy-load → `<img style="display:none" @load @error>`.
- ❌ `imgState` initialisé à `'loaded'` quand `cover_url` est null → toujours `'loading'`, `card-fallback-bg` si `!cover_url`.
- ❌ `setTimeout` pour les animations de dismiss → `<Transition @after-leave>`.
- ❌ Écrire `localStorage` dans un composant UI → composable/parent.
- ❌ `router.push` au lieu de `router.replace` après auth.
- ❌ `<form @submit.prevent>` → `@click` sur le bouton.
- ❌ `v-html` inconditionnel → derrière `v-if="isHtml"` (XSS).

### Périmètre & process
- ❌ Toucher des fichiers non listés dans l'US.
- ❌ Livrer plus de fichiers que prévu **sans l'annoncer** (le dépassement est autorisé session 8 SI annoncé en gras + dans le titre).
- ❌ « Améliorer » au passage du code hors scope.
- ❌ Réécrire un fichier entier quand une correction ciblée suffisait.

### Fidélité fonctionnelle
- ❌ Simplifier une règle métier subtile (calcul épisode, transitions de state, JST).
- ❌ Changer les clés localStorage/Firestore sans US dédiée.
- ❌ Scorer ou filtrer dans `fetchUpcomingSeason` → appartient à useRecommendations.
- ❌ Appeler `normalizeAnime` dans `syncAnimeUpdates`.

---

## Anti-patterns d'orchestration runtime (audit post-migration, session 6)

> Ces 4 bugs ont passé `vue-tsc` + tous les tests + le build **au vert**. D'où R1/R2/R3.

- ❌ **Stub no-op câblé à vide en croyant que l'orchestration est faite ailleurs.** → grep le nom de la VRAIE fonction. [US-102]
- ❌ **Throttle/sleep inconditionnel après un appel qui peut taper le cache.** → conditionner à `fromNetwork`. [US-106]
- ❌ **Dupliquer une règle métier à deux endroits avec deux seuils.** → une seule source de vérité computed. [US-107]
- ❌ **Écrire un champ que personne ne lit.** → code mort, supprimer. [US-107]
- ❌ **Valider une migration sur le seul tooling vert.** → R3 : un audit lit le CODE. [audit croisé]
- ❌ **Livrer une US d'orchestration sans test runtime.** → R2 : smoke/unit test. [US-109]

---

## Anti-patterns UX & test E2E (audit live, session 7)

> Ces bugs ont passé vue-tsc + tests + audits de code. Ne se voient qu'en CLIQUANT. D'où R4.

- ❌ **Émettre un event sous un nom et l'écouter sous un autre.** `WeekAnimeItem` émet `click`, la page écoutait `@open-modal` → 0 erreur console. Le composant définit son contrat d'emit ; les pages s'y conforment. [P0.1]
- ❌ **Asserter l'état interne (store) au lieu de la visibilité DOM dans un test E2E.** R4 vérifie ce que l'UTILISATEUR voit.
- ❌ **Livrer un test E2E « réparateur » sans la sortie ROUGE pré-fix.** Fournir rouge PUIS vert, test inchangé.
- ❌ **Bypass de test lu en variable runtime** → survit dans le bundle prod. Obligatoirement `import.meta.env.*` (statique). Prouver `grep -c=0`.
- ❌ **Deviner une clé localStorage/Firestore dans un seed de test.** *(Résolu DEC-64 : la vraie clé est `'animeCalendar'`.)*
- ❌ **App muette.** Action utilisateur sans feedback visible = indistinguable de « rien ». [fil rouge audit UX]

---

## Anti-patterns event-name & feedback (session 8)

> Le foyer event-name de `RecCard` était une **famille** de bugs, pas un cas isolé.
> Un seul grep transverse `emit` vs `@listener` les a tous révélés d'un coup.

- ❌ **Famille d'events désalignés sur un composant réutilisé.** `RecCard` (réutilisé par DiscoverExplore ×2, BecauseYouWatched, LibraryExplore) émettait `add`/`click`/`not-interested` ; AUCUN consommateur ne les écoutait (`@heart` partout, pas de `@click`/`@not-interested`). Résultat : bouton Add mort + clic carte mort + « pas intéressé » mort sur TOUTES les surfaces de reco, 0 erreur console. → Quand un composant à emits est réutilisé par N consommateurs, **vérifier les N alignements**, pas un seul. Un grep `defineEmits` (composant) vs `@event` (chaque consommateur, y compris les wrappers à 2 niveaux comme BecauseYouWatched) est obligatoire avant de clore. [P0.8a/b]
- ❌ **Emit orphelin « sans parent ».** `DiscoverExplorePage` (page root sous `<router-view>`) déclarait `defineEmits(['open-modal'])` que personne n'écoute (vue-router ne propage pas les emits custom). Vestige d'une intention abandonnée. → Une page routée n'a quasiment jamais besoin d'`emit` ; supprimer les emits orphelins. [P0.8b]
- ❌ **Toast de feedback manquant sur une action d'ajout.** `onAdd`/`onStartWatching` de `AnimeModal` ajoutaient au store + fermaient la modal **sans toast** (alors que `onMarkDone` en avait un). → Toute action d'ajout/déplacement visible doit produire un toast nommant la **destination visible exacte** (pas le jargon `radar`/`vault`). [P0.4]
- ❌ **Libellé de toast = jargon interne.** « Added to Radar » / « Moved to Vault » : l'utilisateur ne voit nulle part « Radar »/« Vault » dans l'UI. → Nommer l'onglet réel : « Coming Soon » / « Completed ». [DEC-63, à harmoniser P0.4-bis]

---


## Anti-patterns event-name & feedback (session 9)

❌ Markup référence une classe CSS inexistante → blocs en display:block dans le flux (weekday-headers F4 / secondary-tab--active F9 / modal-backdrop P0.9). Grep la classe dans style.css avant de l'utiliser.
❌ E2E qui asserte la visibilité sans la position → un élément mal placé passe le test (P0.1 modal). Asserter getComputedStyle().position quand le placement est l'enjeu.
Récidives Gemini s9 : build tronqué ×2 (P0.6 / P0.6-ter — 3e = suspension), clé localStorage non contractuelle (onAirDefaultView, calendar-subnav-layout).


nouveaux findings CSS (backlog dette, EPIC-2/3) :

F18 : dette « VUE TEST » morte (~150 lignes .test-*)
F19 : doublons .post-it (solid vs pastel, contradictoires)
F20 : hacks CSS :has([style*="none"]) morts post-migration
F21 : #app-loading-overlay { display:none !important } à vérifier vs loader P0.2
F22 : .month-header-mobile orpheline (doublon de .weekday-headers-mobile)
F23 : « your Vault » empty state LibraryCompletedPage:37 (jargon, trivial)
---

## Récidives détectées en cours de projet

- **[US-001]** ❌ Déps parasites + DOM vanilla conservé dans `index.html`.
- **[US-001]** ❌ alias `@` pointé sur `.` au lieu de `./src`.
- **[US-002]** ⚠️ (spec Claude) ESLint flat config.
- **[US-006a]** ❌ `as unknown as <Type>` pour fixtures.
- **[US-011]** ❌ Install firebase hors périmètre.
- **[US-016]** ❌ Import `buildRelationMemory` depuis `@/utils/recEngine` (spec Claude fautive, DEC-21).
- **[US-017a]** ❌ `void import` pour silencer unused.
- **[US-025]** ❌ `imgState`/`card-fallback-bg`.
- **[Audit session 6]** ⚠️ Gemini a **paraphrasé** une sortie de commande au lieu du terminal brut.
- **[Audit session 6]** ⚠️ Gemini a renvoyé une US déjà mergée au lieu d'exécuter la commande.
- **[P0.1 diagnostic, session 7]** ⚠️ Claude a affirmé « `modalOpen` n'existe pas » AVANT de lire le grep — faux. La règle zéro-confiance s'applique à Claude lui-même.
- **[P0.2 diagnostic, session 8]** ⚠️ Claude a soupçonné une `inject` ratée sur `LoadingOverlay` AVANT lecture — faux (c'était le **placement** sous gate auth, pas l'injection). Diagnostic = lire le code D'ABORD. (DEC-59)
- **[P0.3a build, session 8]** ⚠️ **3ᵉ récidive de paraphrase de preuve** : Gemini a livré « Build succeeded - the applet is compiled. » au lieu de la sortie brute. Corrigé sur demande. → Toute paraphrase de build = review suspendue, sans exception, même quand le code est bon.
- **[P0.2 preuve, session 8]** ⚠️ Gemini a fourni DEUX sorties ROUGE incohérentes (deux états de référence différents) pour la même US. → Une preuve ROUGE = un état figé unique, jamais rejouée dans un état différent et présentée comme la même.
- **[P0.3a/P0.8a/P0.8b, session 8]** ⚠️ Triple preuve livrée en chaîne `vue-tsc && vitest && build` au lieu de 3 sorties séparées. Acceptable (le `&&` garantit le vert) mais moins auditable. → Préférer 3 blocs distincts (R1).

---

## Anti-patterns process & diagnostic (session 10)

> Session 10 a coûté ~80% de son temps à réparer une cascade de bugs auto-infligés. Ces deux patterns sont les causes racines.

### Gemini — récidive R-SCOPE-1 la plus coûteuse du projet

- ❌ **Modifier plusieurs fichiers sans aucune US, en cascade.** En réponse à un simple diagnostic, Gemini a modifié `main.ts`, `index.html`, `App.vue`, `playwright.config.ts`, `smoke.spec.ts` — 5 fichiers, zéro US, zéro autorisation. Conséquence : suite E2E cassée sur 17/17 tests, 80% de la session perdue à réparer. → **Règle de démarrage session obligatoire :** exiger de Gemini un état du dépôt (liste des fichiers modifiés depuis le dernier merge connu) AVANT toute action. Ne jamais laisser Gemini « préparer le terrain » de sa propre initiative. Le sandbox AI Studio interdit `git status` — à défaut, demander `ls -la` ou liste fichiers récemment modifiés.

- ❌ **4ᵉ récidive de paraphrase de build.** `"Build succeeded - the applet is compiled."` au lieu de la sortie brute Vite. Compteur : P0.3a (s8) + 2 occurrences s10. → Rappel : toute paraphrase de build = review suspendue d'office, même quand le code est bon. Suggestion : exiger `npm run build 2>&1 | tail -40` pour forcer la sortie brute dans le sandbox.

### Claude — 2 mauvais diagnostics en série (leçon zéro-confiance sur Claude lui-même)

- ⚠️ **Proposer un fix avant de lire le code qui fonctionne déjà.** Sur le problème `.rowcard` absent après boot, Claude a proposé un mock `**/anime/**` (4 allers-retours pour constater qu'il cassait le rendu) avant de lire `modal-open.spec.ts` — le test identique qui passait. La solution (seed 7 jours) était visible dans ce fichier depuis le début. → **Règle :** quand un test échoue et qu'un test similaire passe, **lire le test qui passe EN PREMIER**, pas après épuisement des hypothèses. [s10 modal-position/toast-labels]

- ⚠️ **Diagnostic basé sur une hypothèse non vérifiée (2ᵉ occurrence après DEC-59).** Claude a supposé que `syncAnimeUpdates` écrasait le store (hypothèse plausible mais fausse) sans avoir grep-é le flux réel. Cause du bug : seed mono-jour `day: 'monday'` ne couvrait pas la semaine courante. → Zéro-confiance s'applique aux hypothèses de Claude : grep d'abord, hypothèse après. [s10 boot-loader cascade]

### Gemini — fichier parasite `test_script.ts`

- ❌ **Créer un fichier non sollicité hors périmètre de l'US** (`test_script.ts` à la racine). Pattern identique à `kill.cjs` (s10, début). → Tout fichier non listé dans l'US est un R-SCOPE-1. Gemini doit signaler, pas créer. Claude doit demander la suppression immédiate avant review.

### Pattern E2E boot-dépendant (gravé s10)

- ❌ **Cliquer une carte sans attendre la fin du boot.** Le `#boot-loader` (DEC-72, position fixed) intercepte tous les pointer events pendant le boot. Tout test qui clique après un `page.goto` doit attendre `await expect(page.locator('#boot-loader')).toBeHidden({ timeout: 15000 })` AVANT de localiser/cliquer. Sans ça : timeout 30s garanti. [modal-position/toast-labels s10]

- ❌ **Seed mono-jour dans un test de calendrier semaine.** `day: 'monday'` seul = carte invisible si la semaine courante est une autre. → Toujours seeder les 7 jours (pattern `['monday',...,'sunday'].map(...)`) pour garantir une carte visible quel que soit le jour courant. [modal-position/toast-labels s10, modal-open pattern de référence]

---

## Récidives détectées en cours de projet (suite)

- **[DEC-72 boot-loader, session 10]** ⚠️ Gemini a modifié 5 fichiers sans US → cascade E2E 17/17 rouges. Récidive R-SCOPE-1 la plus coûteuse. (détail ci-dessus)
- **[US-150 build, session 10]** ⚠️ **4ᵉ récidive paraphrase build** : « Build succeeded - the applet is compiled. » → review suspendue. (détail ci-dessus)
- **[modal-position/toast-labels, session 10]** ⚠️ Claude a proposé mock `/anime/**` (faux diagnostic ×1) puis hypothèse "Jikan écrase le store" (faux diagnostic ×2) avant de lire `modal-open.spec.ts`. 4 allers-retours évitables. → Lire le code qui marche AVANT de proposer.



## Session 12
- [AP] Faux-vert #5 (récidive) : E2E assertant store/desktop au lieu du DOM visible mobile. R4 confirmé comme garde-fou.
- [AP] Classe CSS inexistante (.toast-notification sans règle) — déjà loggé s11, rappel.
- [AP] R-SCOPE-1 récidive s11 (onRemove+onNavigate hors US-P0-B) — rappel.
- [AP NOUVEAU — angle mort gating↔E2E] Un gating conditionnel (v-if) sur un élément interactif DOIT déclencher un grep des specs E2E qui ouvrent/cliquent cet élément, AVANT merge. US-P0-E a gaté "Mark done" → toast-labels (qui le cliquait sur un anime airing) est devenu rouge au sweep, pas au merge. Coût : 3 allers-retours. Règle : tout v-if sur un bouton/lien → vérifier les consommateurs E2E dans la même US.
- [AP — auto-vault vs seed E2E] Un seed calendar+Finished s'auto-vault au boot et disparaît de la vue. Pour tester une action sur anime terminé, seed en watchlist (exclu de l'auto-vault par usePersistence). Erreur de diagnostic Claude corrigée en 1 itération.

## Session 12
- [AP] Faux-vert #5 (récidive) : E2E assertant store/desktop au lieu du DOM visible mobile. R4 confirmé comme garde-fou.
- [AP] Classe CSS inexistante (.toast-notification sans règle) — déjà loggé s11, rappel.
- [AP] R-SCOPE-1 récidive s11 (onRemove+onNavigate hors US-P0-B) — rappel.
- [AP NOUVEAU — angle mort gating↔E2E] Un gating conditionnel (v-if) sur un élément interactif DOIT déclencher un grep des specs E2E qui ouvrent/cliquent cet élément, AVANT merge. US-P0-E a gaté "Mark done" → toast-labels (qui le cliquait sur un anime airing) est devenu rouge au sweep, pas au merge. Coût : 3 allers-retours. Règle : tout v-if sur un bouton/lien → vérifier les consommateurs E2E dans la même US.
- [AP — auto-vault vs seed E2E] Un seed calendar+Finished s'auto-vault au boot et disparaît de la vue. Pour tester une action sur anime terminé, seed en watchlist (exclu de l'auto-vault par usePersistence). Erreur de diagnostic Claude corrigée en 1 itération.

## Règles process permanentes (gravées dans AGENTS.md)

- ✅ **R1 — Triple preuve verte (CI).** `vue-tsc --noEmit` + `vitest run` + `build`, **3 sorties brutes séparées**, pour tout MERGE. Rejouées par `ci.yml`.
- ✅ **R2 — Test obligatoire sur l'orchestration.**
- ✅ **R3 — Un audit lit le code.**
- ✅ **R4 — Test E2E sur tout correctif UX/écran.** Geste réel + DOM visible. ROUGE→VERT sans modif. Une preuve ROUGE = un état figé unique.
- ✅ **R5 — Test ciblé par US pendant l'epic, grand check E2E complet en fin d'epic.** Tests cumulatifs dans `tests/e2e/`. (→ AGENTS_E2E.md)
- ✅ **Zéro confiance, y compris sur le code/diagnostic de Claude.** Prouver par l'exécution.
- ✅ **Sortie de commande = session terminale littérale.** Tout résumé/paraphrase = review suspendue.
- ✅ **Livraison = contenu intégral des fichiers.**
- ✅ **Gemini n'a PAS accès à la Knowledge.** US autoportantes. Gouvernance dans `AGENTS.md`.
- ✅ **Fixtures via helper `Partial`/factory** — interdit `as any`/`as unknown as T`.
- ✅ `npx <outil>` accepté ≡ `npm run <script>`.
- ✅ **`eslint-disable-next-line` ne silencie PAS TypeScript.** `TS6133` → retrait ou préfixe `_`.
- ✅ **Max 3 fichiers par US** — sauf dépassement **annoncé en gras + dans le titre** (autorisé session 8).
