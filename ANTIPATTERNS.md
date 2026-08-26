# ANTIPATTERNS.md — Pièges récurrents

> **Rôle :** les erreurs qui se sont **répétées**. Claude en réinjecte les entrées pertinentes dans la section « Anti-patterns à éviter » de chaque US.
> **Pas ici :** les règles opposables (→ `AGENTS.md`), les récidives de l'agent d'implémentation (→ `AGENTS.md §9`), la dette ouverte (→ `STATE.md`), les pièges de seed E2E (→ `AGENTS.md §6`).

**Règle d'entrée : un piège s'écrit à la 2ᵉ occurrence, jamais à la 1ʳᵉ.** Une erreur unique vit dans le handoff. **Organisé par thème, jamais par session.**

---

## 1. Architecture & couches

- ❌ **Logique métier ou `fetch` dans un composant `.vue`** → composable.
- ❌ **Accès DOM direct** (`getElementById`, `querySelector`, `appendChild`) dans du code Vue → `ref` / `v-if` / `v-for`.
  *Nuances validées :* `DOMParser` + `querySelector` sur une string XML est pur ; `createElement('a')` + `.click()` pour un download Blob aussi ; `getElementById('boot-loader').remove()` agit sur un élément d'`index.html`, hors scope Vue.
- ❌ **Recréer un bus d'événements** avec `dispatchEvent` / `CustomEvent` → store Pinia ou `emit`.
- ❌ **État ou handler sur `window`** → `onMounted` / `onUnmounted` + `@vueuse/core`.
- ❌ **Manipuler `document.body.classList`** ou injecter un bandeau dans le `<body>` → état réactif + composant.
- ❌ **`initializeApp` / `getAuth` dans un composable** → réinitialisation. Singleton dans `lib/firebase.ts`.
- ❌ **`onAuthStateChanged` branché dans le corps de `useXxx()`** → listeners empilés à chaque appel. Le brancher **au niveau module**.
- ❌ **`watchDebounced` branché sans flag de module** → même symptôme.
- ❌ **`useFirebaseAuth()` appelé dans un guard Vue Router** → hors contexte `setup`. Utiliser le singleton `auth` + `await auth.authStateReady()` ; ne jamais lire `auth.currentUser` avant ce `await` (il est `null` pendant l'init).
- ❌ **Couche de persistance qui mute le store hors action et porte des toasts.**
- ❌ **Dupliquer une règle métier à deux endroits avec deux seuils** (cas fondateur : hiatus à 14 j d'un côté, 21 j de l'autre) → une seule source computed.
- ❌ **Écrire un champ que personne ne lit.** Code mort : supprimer.

## 2. TypeScript & contrat

- ❌ **`any`, `as any`, `@ts-ignore`** → typer depuis `TYPES_CONTRACT.md`. `eslint-disable-next-line` **ne silencie pas** TypeScript : `TS6133` se corrige en retirant la variable ou en la préfixant `_`.
- ❌ **Inventer une interface qui existe déjà** dans le contrat.
- ❌ **Fixtures via `as unknown as T`** → factory `makeAnime(Partial<AnimeEntry>)`.
- ❌ **Cast brut sans normalisation sur un chemin de chargement.** Un cache local corrompu produit des cartes incomplètes ou un écran blanc. Garde runtime + normalisation obligatoires.
- ❌ **Membre d'union jamais mappé dans un `switch` ou une chaîne de `if`.** `getCardStatus` ne gérait pas `'Continuing'` (pourtant dans l'union, injecté par la persistance) → un show en cours s'affichait « Finished ». **Quand une union gagne une valeur, grep tous ses consommateurs.**
- ❌ **Champ optionnel traité comme garanti** (oublier `?.` ou le cas `null`).
- ❌ **Export nommé depuis `<script setup>`** — impossible. Double bloc `<script>` + `<script setup>`.
- ❌ **`inject(key)` sans fallback** quand la clé est typée → `inject(isBootingKey, ref(false))`.

## 3. Gestion d'erreur

- ❌ **`async` sans `try/catch`**, particulièrement sur un I/O cloud. `saveToDatabase` appelait `saveSchedule` sans garde → rejet Firestore **silencieux**, l'utilisateur croyait avoir sauvegardé.
- ❌ **Avaler une erreur** (`catch {}`) sans log ni état réactif.
- ❌ **Ignorer le 429** et son backoff. ⚠️ Corollaire AniList : un **429 n'est pas une panne** et n'incrémente jamais le compteur du disjoncteur (DEC-126).
- ❌ **Laisser remonter le `throw` de `handleFirestoreError`** → attraper localement, exposer un `error` réactif.
- ❌ **Renseigner un état d'erreur sans jamais l'afficher.** Le cache de saison périmé est servi en cas d'échec et `error.value` est rempli — que rien n'affiche. Sans cache et sans réseau, la liste est vide **sans explication**.

### AP-CATCH-1 — un `catch` qui renvoie un tableau vide

Il est *indistinguable* d'un résultat vide légitime : aucune erreur, aucun test rouge, et l'écran affiche « rien à voir » au lieu de « ça n'a pas marché ».
**7 occurrences sur le seul sprint S40**, dont la bande SEQUELS & RELATED, morte pour tous les utilisateurs pendant des semaines sur un échec silencieux.

**Parade structurelle : contrat `{ data, failed }` par défaut sur toute fonction réseau.** Une fonction réseau qui ne peut pas dire qu'elle a échoué est un bug en attente.
*Corollaire mesuré : un `200` servi depuis un cache expiré est plus dangereux qu'une erreur franche — l'erreur déclenche `failed`, le 200 périmé passe pour un succès.*

## 4. Composants & feedback UI

- ❌ **Émettre un event sous un nom et l'écouter sous un autre.** Le piège n°1 du projet : aucune erreur console, la fonctionnalité est simplement morte. **Le composant définit son contrat d'emit ; les consommateurs s'alignent.** Quand un composant est réutilisé par N consommateurs (y compris des wrappers à 2 niveaux), **vérifier les N alignements** — un grep `defineEmits` vs `@event` les révèle tous d'un coup.
- ❌ **Emit orphelin « sans parent ».** Une page routée sous `<router-view>` qui déclare `defineEmits` : vue-router ne propage pas les emits custom.
- ❌ **Action utilisateur sans feedback visible.** Indistinguable de « rien ne s'est passé ».
- ❌ **Toast en jargon interne.** « Added to Radar », « Moved to Vault » : ces mots n'apparaissent nulle part dans l'UI. Nommer **l'onglet réel**.
- ❌ **Message de confirmation non conditionné à une vérification d'affichage.** `finishWithSeed` annonce « N shows added to your calendar » sans que rien ne garantisse que ces animes soient visibles dans la vue cible. **L'app certifie un succès que l'écran suivant dément.** Un message porte sur ce que l'utilisateur **va voir**, pas sur ce que le code vient d'exécuter.
- ❌ **`new Image()` pour un lazy-load** → `<img style="display:none" @load @error>`.
- ❌ **`imgState` initialisé à `'loaded'`** quand `cover_url` est `null`.
- ❌ **`setTimeout` pour une animation de dismiss** → `<Transition @after-leave>`.
- ❌ **`<form @submit.prevent>`** → `@click` sur le bouton.
- ❌ **`v-html` inconditionnel** → derrière `v-if="isHtml"` (XSS).
- ❌ **`router.push` au lieu de `router.replace`** après authentification.

## 5. CSS & layout

- ❌ **Le markup référence une classe CSS qui n'existe pas.** Déjà rencontré 4 fois (`weekday-headers`, `secondary-tab--active`, `.modal-backdrop`, `.toast-notification`). **Grep la classe dans `style.css` avant de l'utiliser.**
- ❌ **Classe définie dans un `<style scoped>` et consommée ailleurs.** Elle n'est jamais appliquée, silencieusement. Cause racine réelle d'une grille en 1 colonne au lieu de 2.
- ❌ **`width:100%` + `padding` sans `box-sizing:border-box`.** Produit un élément plus large que son conteneur (417 px dans 387 px). 🔴 **Piège aggravant : le symptôme apparaît loin de la cause** — ici, des modales `position:fixed` paraissant décentrées alors que leur CSS était correct. **Devant un décentrage, mesurer d'abord `document.documentElement.scrollWidth` vs `window.innerWidth`.**
- ❌ **Grille en `minmax()` sans marge de sécurité.** `minmax(160px,1fr)` + padding + gap réclamait 344 px sur 339 px utiles → bascule en 1 colonne **pour 5 px**.
- ❌ **Redéclarer une couleur en dur** au lieu de réutiliser un token existant.

## 6. Test E2E — la famille des faux-verts

> Ces bugs ont tous passé `vue-tsc` + tous les tests + le build **au vert**. C'est la raison d'être de R4.
> Les pièges de **seed** (mono-jour, auto-vault, clé localStorage) vivent dans `AGENTS.md §6`, lu par l'agent d'implémentation.

- ❌ **`AP-TEST-5` — mock réseau dupliqué, adhérent à l'URL du fournisseur.** 11 specs portaient chacune leur propre `page.route()` sur les chemins REST de l'ancienne API. Au changement de fournisseur, les 11 sont devenues muettes **en même temps** — et pas au merge, **au sweep**, trois US plus tard, quand la cause était déjà noyée dans l'historique.
  Deux symptômes, un seul bug : soit la fixture n'arrive jamais (écran vide), soit du contenu réel s'affiche à sa place (compteur inattendu). **Le second est le plus vicieux : le test échoue sur un nombre, ce qui fait chercher un bug de rendu.**
  **Parade : un helper de mock unique par fournisseur.** Une spec ne connaît jamais l'URL d'une API.
- ❌ **Asserter l'état d'un store, ou le layout desktop, au lieu du DOM visible en viewport mobile.** Faux-vert n°1 du projet, récidivant.
- ❌ **`toBeVisible()` seul quand la position est l'enjeu.** Un élément hors écran est « visible » au sens DOM. Asserter `boundingBox()` ou `getComputedStyle().position`. *Récidive : un test de centrage assertait `max-height`/`overflow-y` alors que le placement était le sujet.*
- ❌ **`toBeVisible()` sur un conteneur de grille aux données mockées vides.** Un `display:grid` sans enfant a une hauteur de 0 px → `toHaveCount` puis `getComputedStyle`.
- ❌ **Test de centrage sans assertion d'overflow** (`scrollWidth <= clientWidth`) : vert sur une modale visiblement décalée.
- ❌ **E2E réparateur livré sans sa sortie ROUGE pré-fix.** Une preuve rouge = **un état figé unique**, jamais rejouée dans un état différent.
- ❌ **Cliquer sans attendre la fin du boot.** `#boot-loader` est `position:fixed` et intercepte tous les pointer events → timeout de 30 s garanti.
- ❌ **Bypass de test lu en variable runtime** → survit dans le bundle de production. Obligatoirement `import.meta.env.*` (statique), prouvable `grep -c` = 0.
- ❌ **Mock partiel** — une seule requête qui fuit rend le test flaky.
- ❌ **Spec écrite mais non enregistrée dans un batch** → elle ne tourne **jamais** au sweep, sans erreur ni signal.
- ❌ **Argument de batch écrit en nom nu.** Playwright l'interprète comme une **regex de sous-chaîne** : `modal-position` capte aussi `logout-modal-position.spec.ts`. Chemin complet, slashes avant.
- ❌ **Gating ↔ E2E, l'angle mort.** Tout `v-if` ajouté sur un élément interactif casse **en silence** une spec qui le clique — et ça ne se voit **qu'au sweep, pas au merge**. Grep des specs concernées **dans la même US**.
- ❌ **US 🟠/🔴 dont le critère E2E est décrit en langage naturel** au lieu d'être fourni en `.spec.ts` complet et verbatim. Cela laisse une ouverture pour que l'agent écrive lui-même le test — violation de l'invariant auteur-test.
- ❌ **Modifier le code de production pour faire passer une spec périmée.** Cas fondateur : ajout d'une classe dans un composant pour satisfaire un sélecteur E2E obsolète, glissé dans une US sans rapport.
- ❌ **`addInitScript()` rejoue à chaque navigation.** Un `localStorage.clear()` non gardé efface l'état entre deux `goto` d'une même spec → sentinelle `sessionStorage` + point de synchro (toast visible) avant navigation.
- ❌ **Spécifier une US sans avoir lu la spec E2E qui doit la prouver.** Une spec dont le mock est dépourvu de la donnée corrigée ne peut pas passer au vert.

### AP-TEST-x — un test qui ne peut pas échouer pour la bonne raison

**Le motif :** une spec continue de tourner (verte ou rouge) longtemps après que son sujet a disparu. Elle n'atteste plus rien, mais sa présence au registre fait croire à une couverture.

**Occurrences :** `snap-to-today` sème un calendrier complet mais n'asserte qu'un en-tête de jour — **elle passerait avec un store vide**. `discover-season-dedup` ciblait une classe renommée deux sprints plus tôt et mockait une pagination supprimée : **rouge sans que personne le sache**, et le dédoublonnage qu'elle prétendait couvrir n'a jamais été testé.

**Règle :** toute US qui renomme une classe CSS, change une route réseau ou supprime un mécanisme (pagination, cache, endpoint) **liste les specs E2E qui en dépendent** dans sa section « fichiers », ou déclare explicitement qu'aucune n'en dépend.

**Corollaire :** un test dont on ne peut pas décrire *le scénario exact qui le ferait passer au rouge* n'est pas un filet.

## 7. Diagnostic — vaut pour Claude autant que pour l'agent

> Le zéro-confiance s'applique **d'abord à soi**. Ces entrées viennent majoritairement de mauvais diagnostics de Claude.

- ⚠️ **Proposer un fix avant d'avoir lu le code qui fonctionne déjà.** **Quand un test échoue et qu'un test similaire passe, lire le test qui passe EN PREMIER.**
- ⚠️ **Bâtir un diagnostic sur une hypothèse jamais grep-ée.** Trois occurrences : une `inject` supposée ratée (c'était un placement), un `syncAnimeUpdates` supposé écraser le store (c'était un seed mono-jour), une classe supposée absente (elle existait). **Grep d'abord, hypothèse ensuite.**
- ⚠️ **Théoriser une cause racine en changeant deux variables à la fois.** **Une mesure = une variable.** Aucune cause racine n'est gravée avant qu'une hypothèse n'explique 100 % des mesures.
- ⚠️ **Diagnostiquer une panne externe sans la remesurer.** Une panne a été portée **5 sprints** sur la foi d'un unique curl, gelant 2 items du backlog. Toute panne externe en standby se remesure à l'ouverture de session, **avec la requête exacte du code**.
- ⚠️ **Inférer l'état d'un service depuis des requêtes vers un autre domaine.** Les jaquettes viennent d'un CDN, pas de l'API de données, et leurs URLs sont en cache local : une page pleine de posters en 200 n'indique **rien** sur la santé de l'API.
- ⚠️ **Confondre le cache HTTP du navigateur et le localStorage applicatif.** « Disable cache » ne touche **jamais** au localStorage. **Vérifier l'existence d'un flag par grep avant de raisonner dessus.**
- ⚠️ **Traiter un handoff comme une source primaire.** Un handoff a affirmé l'existence de trois symboles **tous inexistants**. **Le code réel tranche.**
- ⚠️ **Planifier depuis un backlog jamais confronté au code.** Trois US ont été planifiées « à faire » alors qu'elles étaient en production. **Toute US sortant du backlog démarre par un grep.**
- ⚠️ **Prendre le diagnostic du PO pour une cause.** Le PO décrit un **symptôme** ; vérifier avant de spécifier.
- ⚠️ **Extrapoler une clôture d'audit à un cas voisin.**
- ⚠️ **`findstr` : un vide n'est une preuve d'absence que si la syntaxe est vérifiée.** Le OU s'écrit avec des **mots séparés par des espaces** (détail → `CLAUDE.md §6`). Deux fausses conclusions ont déjà été tirées de vides syntaxiques.

## 8. Process & hygiène de livraison

| ID | Piège | Gravité | Parade |
|---|---|---|---|
| **AP-HYGIENE-1** | **Fichiers de travail commités** (`wait.txt`, fichiers de debug, `*_out.txt`, `test-*.cjs`) | 🟠 | Le débogage reste local. Le PO vérifie `git status` avant la gate ; tout fichier hors périmètre déclaré = correction mineure d'office. Vérifier par `git show --name-only HEAD`, **jamais** `git diff` (après un `pull` l'arbre est propre et ne prouve rien) |
| **AP-TS-1** | **`as unknown as` posé sans qu'aucune erreur de compilation ne l'exige** | 🟠 | Un double cast n'est légitime qu'en réponse à une erreur `vue-tsc` **citée dans le rapport de livraison**. Sans citation : à retirer |
| **AP-PROCESS-2** | **Inférence promue en fait par son entrée dans un document.** Un test déclaré « rouge qualifié » sur la seule lecture de son code — il était vert. Un chemin de fichier déduit d'un handoff au lieu d'être lu | 🟠 | **Un test n'est rouge que sur sa sortie d'exécution.** Sans sortie collée par le PO : « suspecté », jamais « qualifié ». **Un chemin de fichier se lit, ne se déduit pas.** Le mot choisi dans `STATE.md` engage : il sera lu comme un fait mesuré à la session suivante |
| **AP-PROCESS-3** | **Modification documentaire non isolée → faux signal de dérive de périmètre.** Deux occurrences : patch doc envoyé dans le même message qu'une US, puis déploiement d'`AGENTS.md` non commité, embarqué par le commit suivant de l'agent | 🟠 | Le déploiement d'`AGENTS.md` se **commite seul, en clôture de session, avant toute nouvelle US**. Un `git status` vide est **condition d'ouverture** d'une livraison. Un fichier inattendu dans un diffstat se **lit** avant d'être imputé |
| — | **Sortie de test d'un run antérieur présentée comme fraîche** (2 occurrences) | 🟠 | Seule la machine PO fait foi (R1) |
**AP-PROCESS-4 — Jamais d'US non envoyable.** Un bloc `## [US-XXX]` n'est produit que s'il est
collable à Gemini immédiatement : test de fidélité inclus, types vérifiés contre le code source.
S'il manque un fichier, on demande le fichier — on ne rédige pas un squelette.
*Origine : une US livrée sans son test de fidélité a coûté un tour de conversation entier.*

## 9. Fidélité fonctionnelle

- ❌ **Simplifier une règle métier subtile** : calcul d'épisode, transitions de state, conversion de fuseau. Le comportement de référence est reproduit, quirks inclus.
- ❌ **Changer une clé localStorage ou Firestore sans US dédiée** — avec migration.
- ❌ **Scorer ou filtrer dans une fonction de fetch de saison** : cela appartient à `useRecommendations`.
- ❌ **Créer une entrée `state:'calendar'` sans se demander qui posera son `day`.** Sans `day`, l'anime est stocké mais **invisible partout**.

---

## 9. Méthode d'analyse (Claude)

- ❌ **`AP-METHOD-1` — énoncer une cause sur du code non lu.** Trois occurrences sur deux sessions
  consécutives : `AUD-33` déclaré soldé sans mesure (SE-067) ; comparaison de hash de chunk entre
  deux environnements de build, **non falsifiable par construction** (SE-068) ; service worker
  déclaré bloquant bêta avant lecture du fichier — il ne cachait rien (SE-068) ; `backdrop-filter`
  désigné comme cause unique d'`AUD-46`, patch sans effet (SE-068).
  **Coût mesuré :** deux des trois se sont réglées par un `type` de fichier qui aurait pu être
  demandé d'emblée ; la troisième a produit un patch mort commité en `main`.
  ✅ **Contre-mesure :** avant d'énoncer une cause, demander la lecture qui tranche. Une commande
  `type` coûte un aller-retour ; une hypothèse fausse en coûte trois et brûle un patch.
  Corollaire de `PILOTAGE §6` (« toute US sortant du backlog démarre par un grep »), étendu au
  **diagnostic**, pas seulement à la rédaction d'US.
----

## 🎓 Les 3 leçons de méthode les plus chères

1. **Le vert ne prouve rien sur l'utilisabilité.** Type-check + tests + build au vert ≠ application fonctionnelle ≠ application utilisable. Quatre bugs runtime et toute la famille des events désalignés sont passés au vert intégral. D'où R2, R3, R4 et l'audit live du PO.
2. **Un cadre d'audit identique révèle les angles morts.** Un dual audit (deux auditeurs, mêmes axes, même barème, même format) a montré que **chacun avait raté le finding n°1 de l'autre**. Sans cadre commun strict, on ne compare que du bruit.
3. **Les incidents les plus coûteux ne sont pas des défauts de code, mais de fraîcheur de fait.** Une panne externe jamais remesurée (5 sprints gelés) et un backlog jamais confronté au code (3 US planifiées sur du déjà livré) ont coûté plus que tous les bugs de typage réunis.
### AP-PROCESS-4 — Une US incomplète ne s'écrit pas

Un bloc `## [US-XXX]` n'est produit que s'il est **immédiatement envoyable à Gemini**. Pas de
brouillon, pas de « test de fidélité à suivre », pas de placeholder. S'il manque un fichier
pour rédiger : demander le fichier, ne rien écrire.
*Occurrence : SE-065, une US livrée sans son `.spec.ts`, capacité de conversation perdue.*

### AP-PROCESS-5 — Un test de fidélité vert ne prouve pas que la spec vise la bonne branche

La porte verte prouve que le code fait ce que le test dit. Elle ne prouve **jamais** que le
test dit la bonne chose. Sur une US 🔴 d'orchestration, l'assertion doit porter sur la
**séquence réelle** (quelle navigation, depuis quel état, déclenchée par quoi), pas sur la
condition telle qu'on l'imagine.
*Occurrence : SE-065, `US-PERSIST-P0a` mergée 270/270 verte, zéro effet — spec et test visaient
tous deux `to.meta.guestOnly && isLoggedIn`, branche jamais atteinte pendant une connexion.*
**Remède : DEC-161.**

### AP-DIAG-3 — Chercher l'état impossible, pas rejouer le parcours

Un bug qui survit des dizaines de sprints se cache dans une combinaison d'états que le
développement ne produit jamais. Construire l'état artificiellement bat le fait de rejouer le
parcours utilisateur.
*Occurrence : `AUD-30` a résisté à trois hypothèses ; il est tombé en 30 s avec
« `localStorage` vidé à la main + `F5` sans se déconnecter » — page chargée en état connecté
avec un cache vierge. En dev, le cache est toujours chaud.*
