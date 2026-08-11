# ANTIPATTERNS.md — Pièges récurrents du projet

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** la mémoire des erreurs qui se sont **répétées**. Claude en réinjecte les entrées
> pertinentes dans la section « Anti-patterns à éviter » de chaque US.
>
> **Règle d'entrée : un piège s'écrit à la 2ᵉ occurrence, jamais à la 1ʳᵉ.** Une erreur unique
> vit dans le handoff, pas ici. Sans ce filtre, ce document redevient un journal de session.
>
> **Organisé par thème, jamais par session.** Un classement chronologique produit des extraits
> qui se contredisent d'une section à l'autre — c'est ce qui est arrivé à la version
> précédente de ce fichier.
>
> **Ce qui n'est PAS ici :** les règles opposables (→ `AGENTS.md`), les récidives
> comportementales de Gemini (→ `AGENTS.md §9`, le seul fichier qu'il lit), la dette ouverte
> (→ `STATE.md`).

---

## 1. Architecture & couches

- ❌ **Logique métier ou `fetch` dans un composant `.vue`** → composable.
- ❌ **Accès DOM direct** (`getElementById`, `querySelector`, `appendChild`) dans du code Vue
  → `ref` / `v-if` / `v-for`.
  *Nuances validées :* `DOMParser` + `querySelector` sur une string XML est pur et autorisé ;
  `createElement('a')` + `.click()` pour un download Blob aussi ; `getElementById('boot-loader').remove()`
  agit sur un élément d'`index.html`, hors scope Vue — légitime.
- ❌ **Recréer un bus d'événements** avec `dispatchEvent` / `CustomEvent` → store Pinia ou `emit`.
- ❌ **État ou handler sur `window`** → `onMounted` / `onUnmounted` + `@vueuse/core`.
- ❌ **Manipuler `document.body.classList`** ou injecter un bandeau dans le `<body>` → état
  réactif + composant.
- ❌ **`initializeApp` / `getAuth` dans un composable** → réinitialisation. Singleton dans
  `lib/firebase.ts`.
- ❌ **`onAuthStateChanged` branché dans le corps de `useXxx()`** → listeners empilés à chaque
  appel. Le brancher **au niveau module**.
- ❌ **`watchDebounced` branché sans flag de module** → même symptôme, listeners empilés.
- ❌ **`useFirebaseAuth()` appelé dans un guard Vue Router** → hors contexte `setup`. Utiliser
  le singleton `auth` + `await auth.authStateReady()` ; ne jamais lire `auth.currentUser`
  avant ce `await` (il est `null` pendant l'init).
- ❌ **Couche de persistance qui mute le store hors action et porte des toasts.** Casse la
  séparation et la testabilité.
- ❌ **Dupliquer une règle métier à deux endroits avec deux seuils.** Le hiatus a vécu avec
  14 j d'un côté et 21 j de l'autre → une seule source computed.
- ❌ **Écrire un champ que personne ne lit.** Code mort : supprimer.

## 2. TypeScript & contrat

- ❌ **`any`, `as any`, `@ts-ignore`** → typer depuis `TYPES_CONTRACT.md`.
  `eslint-disable-next-line` **ne silencie pas** TypeScript : `TS6133` se corrige en retirant
  la variable ou en la préfixant `_`.
- ❌ **Inventer une interface qui existe déjà** dans le contrat.
- ❌ **Fixtures via `as unknown as T`** → factory `makeAnime(Partial<AnimeEntry>)`.
- ❌ **Cast brut sans normalisation sur un chemin de chargement**
  (`setData(loadedData as unknown as AnimeEntry[])`) : un cache local corrompu produit des
  cartes incomplètes ou un écran blanc. Garde runtime + normalisation obligatoires.
- ❌ **Membre d'union jamais mappé dans un `switch` ou une chaîne de `if`.** `getCardStatus`
  ne gérait pas `'Continuing'` (pourtant dans l'union, injecté par la persistance) → un show
  en cours s'affichait « Finished ». **Quand une union gagne une valeur, grep tous ses
  consommateurs.**
- ❌ **Champ optionnel traité comme garanti** (oublier `?.` ou le cas `null`).
- ❌ **Export nommé depuis `<script setup>`** — impossible. Double bloc `<script>` + `<script setup>`.
- ❌ **`inject(key)` sans fallback** quand la clé est typée → `inject(isBootingKey, ref(false))`.

## 3. Gestion d'erreur

- ❌ **`async` sans `try/catch`**, particulièrement sur un I/O cloud. `saveToDatabase`
  appelait `saveSchedule` sans garde → rejet Firestore **silencieux**, l'utilisateur croyait
  avoir sauvegardé. C'était le chemin le plus critique de l'app.
- ❌ **Avaler une erreur** (`catch {}`) sans log ni état réactif.
- ❌ **Oublier le 429 Jikan** et le retry / backoff.
- ❌ **Laisser remonter le `throw` de `handleFirestoreError`** → attraper localement, exposer
  un `error` réactif.
- ❌ **Renseigner un état d'erreur sans jamais l'afficher.** `fetchCurrentSeason` sert un
  cache périmé en cas d'échec et remplit `error.value` — que rien n'affiche. Sans cache et
  sans réseau, la liste est vide **sans explication**.

## 4. Composants & feedback UI

- ❌ **Émettre un event sous un nom et l'écouter sous un autre.** Le piège n°1 du projet :
  aucune erreur console, la fonctionnalité est simplement morte. **Le composant définit son
  contrat d'emit ; les consommateurs s'alignent.** Quand un composant est réutilisé par N
  consommateurs (y compris des wrappers à 2 niveaux), **vérifier les N alignements** — un
  grep `defineEmits` vs `@event` les révèle tous d'un coup.
- ❌ **Emit orphelin « sans parent ».** Une page routée sous `<router-view>` qui déclare
  `defineEmits` : vue-router ne propage pas les emits custom. Personne n'écoute.
- ❌ **Action utilisateur sans feedback visible.** Indistinguable de « rien ne s'est passé ».
  Toute action d'ajout ou de déplacement produit un toast.
- ❌ **Toast en jargon interne.** « Added to Radar », « Moved to Vault » : l'utilisateur ne
  voit ces mots nulle part dans l'UI. Nommer **l'onglet réel**.
- ❌ **Message de confirmation non conditionné à une vérification d'affichage.**
  `finishWithSeed` annonce « N shows added to your calendar » sans que rien ne garantisse que
  ces animes soient visibles dans la vue cible. **L'app certifie un succès que l'écran suivant
  dément.** Un message porte sur ce que l'utilisateur **va voir**, pas sur ce que le code
  vient d'exécuter.
- ❌ **`new Image()` pour un lazy-load** → `<img style="display:none" @load @error>`.
- ❌ **`imgState` initialisé à `'loaded'`** quand `cover_url` est `null`.
- ❌ **`setTimeout` pour une animation de dismiss** → `<Transition @after-leave>`.
- ❌ **`<form @submit.prevent>`** → `@click` sur le bouton.
- ❌ **`v-html` inconditionnel** → derrière `v-if="isHtml"` (XSS).
- ❌ **`router.push` au lieu de `router.replace`** après authentification.

## 5. CSS & layout

- ❌ **Le markup référence une classe CSS qui n'existe pas.** Déjà rencontré 4 fois
  (`weekday-headers`, `secondary-tab--active`, `.modal-backdrop`, `.toast-notification`).
  **Grep la classe dans `style.css` avant de l'utiliser.**
- ❌ **Classe définie dans un `<style scoped>` et consommée ailleurs.** Elle n'est jamais
  appliquée, silencieusement. Cause racine réelle d'une grille en 1 colonne au lieu de 2.
- ❌ **`width:100%` + `padding` sans `box-sizing:border-box`.** Produit un élément plus large
  que son conteneur (417 px dans 387 px). 🔴 **Piège aggravant : le symptôme apparaît loin de
  la cause** — ici, des modales `position:fixed` paraissant décentrées alors que leur CSS
  était correct. **Devant un décentrage, mesurer d'abord
  `document.documentElement.scrollWidth` vs `window.innerWidth`.**
- ❌ **Grille en `minmax()` sans marge de sécurité.** `minmax(160px,1fr)` + padding + gap
  réclamait 344 px sur 339 px utiles → bascule en 1 colonne **pour 5 px**.
- ❌ **Redéclarer une couleur en dur** au lieu de réutiliser un token existant.

## 6. Test E2E — la famille des faux-verts

> Ces bugs ont tous passé `vue-tsc` + tous les tests + le build **au vert**. C'est la raison
> d'être de R4.

- ❌ **Asserter l'état d'un store, ou le layout desktop, au lieu du DOM visible en viewport
  mobile.** Faux-vert n°1 du projet, récidivant. Le test passe sans rien prouver.
- ❌ **`toBeVisible()` seul quand la position est l'enjeu.** Un élément hors écran est
  « visible » au sens DOM. Asserter `boundingBox()` ou `getComputedStyle().position`.
  *Récidive constatée : un test de centrage assertait `max-height`/`overflow-y` alors que le
  placement était précisément le sujet du bug.*
- ❌ **`toBeVisible()` sur un conteneur de grille sous données mockées vides.** Un
  `display:grid` sans enfant a une hauteur de 0 px. Pour tester une propriété de conteneur,
  vérifier l'existence (`toHaveCount`) puis lire `getComputedStyle`.
- ❌ **Test de centrage sans assertion d'overflow.** Il n'a de sens qu'accompagné de
  `scrollWidth <= clientWidth` — sinon il est vert sur une modale visiblement décalée.
- ❌ **E2E réparateur livré sans sa sortie ROUGE pré-fix.** Une preuve rouge = **un état figé
  unique**, jamais rejouée dans un état différent et présentée comme la même.
- ❌ **Cliquer une carte sans attendre la fin du boot.** `#boot-loader` est `position:fixed`
  et intercepte tous les pointer events. Sans
  `await expect(page.locator('#boot-loader')).toBeHidden()`, timeout de 30 s garanti.
- ❌ **Seed mono-jour** (`day:'monday'` seul) : la carte est invisible si la semaine courante
  tombe un autre jour. **Toujours seeder les 7 jours.**
- ❌ **Seed `calendar` + `Finished Airing`** : il s'auto-vault au boot et **disparaît** de la
  vue. Pour tester une action sur un anime terminé, seeder en `watchlist`.
- ❌ **Deviner une clé localStorage dans un seed.** La clé est `aanime_calendar`.
- ❌ **Bypass de test lu en variable runtime** → survit dans le bundle de production.
  Obligatoirement `import.meta.env.*` (statique), prouvable `grep -c` = 0.
- ❌ **Mock partiel** — une seule requête qui fuit rend le test flaky.
- ❌ **Spec écrite mais non enregistrée dans un batch** → elle ne tourne **jamais** au sweep,
  sans erreur ni signal.
- ❌ **Argument de batch écrit en nom nu.** Playwright l'interprète comme une **regex de
  sous-chaîne** : `modal-position` capte aussi `logout-modal-position.spec.ts`. Chemin
  complet, slashes avant.
- ❌ **Gating ↔ E2E, l'angle mort.** Tout `v-if` ajouté sur un élément interactif casse **en
  silence** une spec qui le clique — et ça ne se voit **qu'au sweep, pas au merge**. Grep des
  specs concernées **dans la même US**.
- ❌ **US 🟠/🔴 dont le critère E2E est décrit en langage naturel** au lieu d'être fourni en
  `.spec.ts` complet et verbatim. Cela laisse une ouverture pour que Gemini écrive lui-même
  le test — violation de l'invariant auteur-test.

## 7. Diagnostic — vaut pour Claude autant que pour Gemini

> Le zéro-confiance s'applique **d'abord à soi**. Ces entrées viennent majoritairement de
> mauvais diagnostics de Claude.

- ⚠️ **Proposer un fix avant d'avoir lu le code qui fonctionne déjà.** Sur une carte absente
  après boot, 4 allers-retours ont été perdus avant d'ouvrir le test similaire qui passait —
  la solution y était visible depuis le début. **Quand un test échoue et qu'un test similaire
  passe, lire le test qui passe EN PREMIER.**
- ⚠️ **Bâtir un diagnostic sur une hypothèse jamais grep-ée.** Trois occurrences : une
  `inject` supposée ratée (c'était un problème de placement), un `syncAnimeUpdates` supposé
  écraser le store (c'était un seed mono-jour), une classe supposée absente (elle existait).
  **Grep d'abord, hypothèse ensuite.**
- ⚠️ **Théoriser une cause racine en changeant deux variables à la fois.** Sur l'investigation
  Jikan, 4 hypothèses successives ont été fausses parce que chaque mesure modifiait plusieurs
  paramètres et que l'échec était attribué à celui qui arrangeait l'hypothèse en cours.
  **Une mesure = une variable. Aucune cause racine n'est gravée avant qu'une hypothèse
  n'explique 100 % des mesures.**
- ⚠️ **Diagnostiquer une panne externe sans la remesurer.** « Jikan est en panne » a été porté
  **5 sprints** sur la foi d'un unique curl, gelant 2 items du backlog. Toute panne externe en
  standby se remesure à l'ouverture de session, **avec la requête exacte du code** — un
  endpoint testé avec d'autres paramètres est un autre endpoint.
- ⚠️ **Inférer l'état d'un service depuis des requêtes vers un autre domaine.** Les jaquettes
  viennent du CDN MyAnimeList, pas de l'API Jikan, et leurs URLs sont déjà en cache local.
  Une page pleine de posters en 200 n'indique **rien** sur la santé de l'API de données.
- ⚠️ **Confondre le cache HTTP du navigateur et le localStorage applicatif.** La case
  « Disable cache » de DevTools ne touche **jamais** au localStorage. Cette confusion a fait
  porter une session entière sur un « flag de cache désactivé en dev » qui n'existe pas.
  **Vérifier l'existence d'un flag par grep avant de raisonner dessus.**
- ⚠️ **Traiter un handoff comme une source primaire.** Un handoff a affirmé l'existence de
  `setAllData`, `syncStatus` et `reconcileWithDatabase` — **les trois inexistants**. Un
  handoff est une source secondaire faillible ; **le code réel tranche**.
- ⚠️ **Planifier depuis un backlog jamais confronté au code.** Trois US ont été planifiées
  « à faire » alors qu'elles étaient en production. **Toute US sortant du backlog démarre par
  un grep du fichier concerné avant rédaction.**
- ⚠️ **Prendre le diagnostic du PO pour une cause.** Un signalement « les modales sont
  différentes » recouvrait en réalité une incohérence de grille CSS. Le PO décrit un
  **symptôme** ; vérifier avant de spécifier.
- ⚠️ **Extrapoler une clôture d'audit à un cas voisin.** Un centrage clos comme « perception »
  sur une modale ne dit rien des deux autres qui partagent la même classe.
- ⚠️ **`findstr` sous Windows : un vide n'est une preuve d'absence que si la syntaxe est
  vérifiée.** `findstr /n "a\|b\|c"` cherche la chaîne littérale et renvoie 0 résultat
  trompeur — le OU s'écrit avec des **mots séparés par des espaces**. Deux fausses
  conclusions ont déjà été tirées de vides syntaxiques.
❌ **Fichiers de débogage commités** (`test-*.cjs`, `*.py`, `e2e_output.txt`). 4 occurrences. Vérifier `git show --name-only HEAD`, jamais `git diff` — après un `pull` l'arbre est propre et `git diff` ne prouve rien.
- ❌ **Sortie de test d'un run antérieur présentée comme fraîche.** 2 occurrences. Seule la machine PO fait foi (R1).
- ❌ **Modifier le code de production pour faire passer une spec périmée.** Cas fondateur : ajout d'une classe `anime-grid` dans un composant pour satisfaire un sélecteur E2E obsolète, glissé dans une US de normalisation sans rapport. Le symptôme est un fichier hors périmètre — d'où l'importance de vérifier le périmètre **avant** de lire le code.

## 8. Fidélité fonctionnelle

- ❌ **Simplifier une règle métier subtile** : calcul d'épisode, transitions de state,
  conversion JST. Le comportement du vanilla est reproduit, quirks inclus.
- ❌ **Changer une clé localStorage ou Firestore sans US dédiée** — avec migration.
- ❌ **Scorer ou filtrer dans `fetchUpcomingSeason`** : cela appartient à `useRecommendations`.
- ❌ **Appeler `normalizeAnime` dans `syncAnimeUpdates`.**
- ❌ **Créer une entrée `state:'calendar'` sans se demander qui posera son `day`.** Sans `day`,
  l'anime est stocké mais **invisible partout**.

- ❌ **`addInitScript()` rejoue à chaque navigation.** Un `localStorage.clear()` non gardé efface l'état entre deux `goto` d'une même spec. Remède : sentinelle `sessionStorage` + point de synchro (toast visible) avant navigation.
- ❌ **Spécifier une US sans avoir lu la spec E2E qui doit la prouver.** 2 occurrences sur AUD-01. R3 s'applique aux **tests** autant qu'au code : une spec dont le mock est volontairement dépourvu de la donnée corrigée ne peut pas passer au vert, et l'annoncer comme gate fait perdre un aller-retour complet.
- ❌ **Publier un patch de documentation dans le même message qu'une US.** L'agent applique les deux et déclenche un faux verdict R-SCOPE-1.
---

## 🎓 Les 3 leçons de méthode les plus chères du projet

1. **Le vert ne prouve rien sur l'utilisabilité.** Type-check + tests + build au vert ≠
   application fonctionnelle ≠ application utilisable. Quatre bugs runtime et toute la
   famille des events désalignés sont passés au vert intégral. D'où R2, R3, R4 et l'audit
   live du PO.
2. **Un cadre d'audit identique révèle les angles morts.** Le dual audit s16 (Claude Code +
   Gemini, mêmes axes, même barème, même format) a montré que **chacun avait raté le finding
   n°1 de l'autre**. Sans cadre commun strict, on ne compare que du bruit.
3. **Les incidents les plus coûteux ne sont pas des défauts de code, mais de fraîcheur de
   fait.** Une panne externe jamais remesurée (5 sprints gelés) et un backlog jamais confronté
   au code (3 US planifiées sur du déjà livré) ont coûté plus que tous les bugs de typage
   réunis.
