# ANTIPATTERNS.md — Pièges récurrents

> **Rôle :** les erreurs qui se sont **répétées**. Claude en réinjecte les entrées pertinentes
> dans la section « Anti-patterns à éviter » de chaque US.
> **Pas ici :** les règles opposables de code (→ `AGENTS.md §4`, lu par Gemini) · les récidives
> de l'agent (→ `AGENTS.md §9`) · les pièges de seed E2E (→ `AGENTS.md §6`) · la dette ouverte
> (→ `AUDIT.md`).
>
> **Règle d'entrée : un piège s'écrit à la 2ᵉ occurrence, jamais à la 1ʳᵉ.** Une occurrence unique
> vit dans `HISTORIQUE §2 Tampon`. **Organisé par thème, jamais par session.**
>
> **Mise à jour : une fois par sprint**, en append dans le `§8 Sprint courant` (`DEC-190`).
> Les identifiants `AP-xx` sont **abandonnés** (SE-075) : ces pièges sont cités par leur titre.

---

## 1. Architecture & couches

> Les interdits de base (DOM direct, bus d'événements, état sur `window`, `<style scoped>`,
> logique métier dans un `.vue`) vivent dans `AGENTS.md §4` et ne sont pas redits ici.

- ❌ **`initializeApp` / `getAuth` dans un composable** → réinitialisation. Singleton dans `lib/firebase.ts`.
- ❌ **`onAuthStateChanged` branché dans le corps de `useXxx()`** → listeners empilés à chaque appel. Le brancher **au niveau module**.
- ❌ **`watchDebounced` branché sans flag de module** → même symptôme. `usePersistence` porte `watchInitialized` pour ça.
- ❌ **`useFirebaseAuth()` appelé dans un guard Vue Router** → hors contexte `setup`. Utiliser le singleton `auth` + `await auth.authStateReady()` ; ne jamais lire `auth.currentUser` avant ce `await`.
- ❌ **Manipuler `document.body.classList`** ou injecter un bandeau dans le `<body>` → état réactif + composant.
- ❌ **Couche de persistance qui mute le store hors action et porte des toasts.**
- ❌ **Dupliquer une règle métier à deux endroits avec deux seuils** — cas fondateur : hiatus à 14 j d'un côté, 21 j de l'autre. Une seule source `computed`.
- ❌ **Écrire un champ que personne ne lit.** Code mort : supprimer.

## 2. TypeScript & contrat

- ❌ **Inventer une interface qui existe déjà** dans `TYPES_CONTRACT.md`.
- ❌ **Fixture de production écrite par cast.** Une fixture `AnimeEntry` s'écrit en **littéral complet**, tous champs obligatoires renseignés, fournie par le Tech Lead dans l'US. 🔻 *La remédiation historique « → factory `makeAnime()` » est morte : la factory était elle-même un cast (`AUD-14`). `AUD-35` suit sa réapparition dans les specs.*
- ❌ **Cast brut sans normalisation sur un chemin de chargement.** Un cache local corrompu produit des cartes incomplètes ou un écran blanc. Garde runtime + normalisation obligatoires.
- ❌ **Membre d'union jamais mappé dans un `switch` ou une chaîne de `if`.** `getCardStatus` ne gérait pas `'Continuing'` → un show en cours s'affichait « Finished ». **Quand une union gagne une valeur, grep tous ses consommateurs.**
- ❌ **Champ optionnel traité comme garanti** (oublier `?.` ou le cas `null`).
- ❌ **Export nommé depuis `<script setup>`** — impossible. Double bloc `<script>` + `<script setup>`.
- ❌ **`inject(key)` sans fallback** quand la clé est typée → `inject(isBootingKey, ref(false))`.
- ❌ **`as unknown as` posé sans qu'aucune erreur de compilation ne l'exige.** Un double cast n'est légitime qu'en réponse à une erreur `vue-tsc` **citée dans le rapport de livraison**. Sans citation : à retirer.

## 3. Gestion d'erreur

- ❌ **Renseigner un état d'erreur sans jamais l'afficher.** Le cache de saison périmé est servi et `error.value` est rempli — que rien n'affiche. Sans cache et sans réseau, la liste est vide **sans explication**.
- ❌ **Laisser remonter le `throw` de `handleFirestoreError`** → attraper localement, exposer un `error` réactif.

### Un `catch` qui renvoie un tableau vide

Il est *indistinguable* d'un résultat vide légitime : aucune erreur, aucun test rouge, et l'écran
affiche « rien à voir » au lieu de « ça n'a pas marché ». **7 occurrences sur le seul sprint S40**,
dont la bande SEQUELS & RELATED, morte pour tous les utilisateurs pendant des semaines.

**Parade structurelle : contrat `{ data, failed }` par défaut sur toute fonction réseau.** Une
fonction réseau qui ne peut pas dire qu'elle a échoué est un bug en attente.
*Corollaire mesuré : un `200` servi depuis un cache expiré est plus dangereux qu'une erreur
franche — l'erreur déclenche `failed`, le 200 périmé passe pour un succès.*

## 4. Composants & feedback UI

- ❌ **Émettre un event sous un nom et l'écouter sous un autre.** Piège n°1 du projet : aucune erreur console, la fonctionnalité est simplement morte. Quand un composant est réutilisé par N consommateurs (y compris des wrappers à 2 niveaux), **vérifier les N alignements** — un grep `defineEmits` vs `@event` les révèle tous d'un coup.
- ❌ **Emit orphelin « sans parent ».** Une page routée sous `<router-view>` qui déclare `defineEmits` : vue-router ne propage pas les emits custom.
- ❌ **Action utilisateur sans feedback visible.** Indistinguable de « rien ne s'est passé ».
- ❌ **Toast en jargon interne.** « Added to Radar », « Moved to Vault » : ces mots n'apparaissent nulle part dans l'UI. Nommer **l'onglet réel**.
- ❌ **Message de confirmation non conditionné à une vérification d'affichage.** `finishWithSeed` annonce « N shows added to your calendar » sans que rien ne garantisse leur visibilité. **L'app certifie un succès que l'écran suivant dément.** Un message porte sur ce que l'utilisateur **va voir**, pas sur ce que le code vient d'exécuter.
- ❌ **`new Image()` pour un lazy-load** → `<img style="display:none" @load @error>`.
- ❌ **`imgState` initialisé à `'loaded'`** quand `cover_url` est `null`.
- ❌ **`setTimeout` pour une animation de dismiss** → `<Transition @after-leave>`.
- ❌ **`<form @submit.prevent>`** → `@click` sur le bouton.
- ❌ **`router.push` au lieu de `router.replace`** après authentification.

## 5. CSS & layout

- ❌ **Le markup référence une classe CSS qui n'existe pas.** Déjà rencontré 4 fois (`weekday-headers`, `secondary-tab--active`, `.modal-backdrop`, `.toast-notification`). **Grep la classe dans `style.css` avant de l'utiliser.**
- ❌ **Classe définie dans un `<style scoped>` et consommée ailleurs.** Jamais appliquée, silencieusement. Cause racine réelle d'une grille en 1 colonne au lieu de 2.
- ❌ **`width:100%` + `padding` sans `box-sizing:border-box`.** Produit un élément plus large que son conteneur (417 px dans 387 px). 🔴 **Le symptôme apparaît loin de la cause** — ici, des modales `position:fixed` paraissant décentrées alors que leur CSS était correct. **Devant un décentrage, mesurer d'abord `document.documentElement.scrollWidth` vs `window.innerWidth`.**
- ❌ **Grille en `minmax()` sans marge de sécurité.** `minmax(160px,1fr)` + padding + gap réclamait 344 px sur 339 px utiles → bascule en 1 colonne **pour 5 px**.
- ❌ **Redéclarer une couleur en dur** au lieu de réutiliser un token existant.

### Poser un `z-index` sans vérifier que l'élément est positionné

**Symptôme :** un élément reste derrière un autre malgré un `z-index` élevé. On augmente la
valeur. Rien ne change. On l'augmente encore.

**Cause :** `z-index` est **ignoré** sur un élément en `position: static`. Le piège est aggravé
quand la règle imposant `static` vit dans une **feuille non scopée d'un composant parent** : elle
gagne l'arbitrage de spécificité et reste invisible à qui ne lit que le composant concerné.

**Aggravant fréquent :** `backdrop-filter`, `filter`, `transform` et `opacity < 1` créent un
contexte d'empilement **même en `position: static`**. Un enfant y est alors plafonné.

**Contre-mesure :** avant de toucher un `z-index`, lire la chaîne d'ancêtres et répondre à deux
questions — (1) l'élément est-il positionné ? (2) un ancêtre crée-t-il un contexte d'empilement ?
Sans ces deux réponses, toute valeur proposée est un tir à l'aveugle.
**Coût constaté :** 3 hypothèses fausses, 1 patch mort commité puis révoqué, 2 sessions. La
lecture qui a tranché a pris 30 secondes.

## 6. Test E2E — la famille des faux-verts

> Ces bugs ont tous passé `vue-tsc` + tous les tests + le build **au vert**. C'est la raison
> d'être de `R4`. Les pièges de **seed** vivent dans `AGENTS.md §6`.

- ❌ **Mock réseau dupliqué, adhérent à l'URL du fournisseur.** 11 specs portaient chacune leur `page.route()` sur les chemins REST de l'ancienne API. Au changement de fournisseur, les 11 sont devenues muettes **en même temps** — et pas au merge, **au sweep**, trois US plus tard. Deux symptômes, un seul bug : soit la fixture n'arrive jamais (écran vide), soit du contenu réel s'affiche à sa place (compteur inattendu). **Le second est le plus vicieux : le test échoue sur un nombre, ce qui fait chercher un bug de rendu.** Parade : un helper de mock unique par fournisseur.
- ❌ **Asserter l'état d'un store, ou le layout desktop, au lieu du DOM visible en viewport mobile.** Faux-vert n°1 du projet, récidivant.
- ❌ **`toBeVisible()` seul quand la position est l'enjeu.** Un élément hors écran est « visible » au sens DOM. Asserter `boundingBox()` ou `getComputedStyle().position`.
- ❌ **`toBeVisible()` sur un conteneur de grille aux données mockées vides.** Un `display:grid` sans enfant a une hauteur de 0 px → `toHaveCount` puis `getComputedStyle`.
- ❌ **Test de centrage sans assertion d'overflow** (`scrollWidth <= clientWidth`) : vert sur une modale visiblement décalée.
- ❌ **E2E réparateur livré sans sa sortie ROUGE pré-fix.** Une preuve rouge = **un état figé unique**, jamais rejouée dans un état différent.
- ❌ **Cliquer sans attendre la fin du boot.** `#boot-loader` est `position:fixed` et intercepte tous les pointer events → timeout de 30 s garanti.
- ❌ **Bypass de test lu en variable runtime** → survit dans le bundle de production. Obligatoirement `import.meta.env.*` (statique), prouvable `grep -c` = 0.
- ❌ **Mock partiel** — une seule requête qui fuit rend le test flaky.
- ❌ **Spec écrite mais non enregistrée dans un batch** → elle ne tourne **jamais** au sweep, sans erreur ni signal.
- ❌ **Argument de batch écrit en nom nu.** Playwright l'interprète comme une **regex de sous-chaîne** : `modal-position` capte aussi `logout-modal-position.spec.ts`. Chemin complet, slashes avant.
- ❌ **Gating ↔ E2E, l'angle mort.** Tout `v-if` ajouté sur un élément interactif casse **en silence** une spec qui le clique — et ça ne se voit **qu'au sweep, pas au merge**. Grep des specs concernées **dans la même US**.
- ❌ **US 🟠/🔴 dont le critère E2E est décrit en langage naturel** au lieu d'être fourni en `.spec.ts` complet et verbatim. Cela laisse une ouverture pour que l'agent écrive lui-même le test.
- ❌ **Modifier le code de production pour faire passer une spec périmée.** Cas fondateur : ajout d'une classe pour satisfaire un sélecteur obsolète, glissé dans une US sans rapport.
- ❌ **`addInitScript()` rejoue à chaque navigation.** Un `localStorage.clear()` non gardé efface l'état entre deux `goto` → sentinelle `sessionStorage` + point de synchro avant navigation.
- ❌ **Spécifier une US sans avoir lu la spec E2E qui doit la prouver.** Une spec dont le mock est dépourvu de la donnée corrigée ne peut pas passer au vert.
- ❌ **Envoyer une US 🟠/🔴 avant d'avoir obtenu la preuve ROUGE de son test de fidélité.** 2 occurrences (SE-073, SE-074). Rattrapable par `git checkout <commit-avant> -- <fichier>` → lancer → observer le rouge → `git checkout HEAD -- <fichier>` → relancer, **sans toucher au test entre les deux runs**. Mais une preuve rouge reconstruite vaut moins qu'une preuve rouge native. ✅ **Ordre opposable : créer la spec → lancer → constater le rouge → PUIS envoyer l'US.**

### Un test qui ne peut pas échouer pour la bonne raison

**Le motif :** une spec continue de tourner (verte ou rouge) longtemps après que son sujet a
disparu. Elle n'atteste plus rien, mais sa présence au registre fait croire à une couverture.

**Occurrences :** `snap-to-today` sème un calendrier complet mais n'asserte qu'un en-tête de jour
— **elle passerait avec un store vide**. `discover-season-dedup` ciblait une classe renommée deux
sprints plus tôt et mockait une pagination supprimée : **rouge sans que personne le sache**.

**Règle :** toute US qui renomme une classe CSS, change une route réseau ou supprime un mécanisme
**liste les specs E2E qui en dépendent** dans sa section « fichiers », ou déclare explicitement
qu'aucune n'en dépend.
**Corollaire :** un test dont on ne peut pas décrire *le scénario exact qui le ferait passer au
rouge* n'est pas un filet.

## 7. Méthode et diagnostic — vaut pour Claude d'abord

> Le zéro-confiance s'applique **d'abord à soi**. Ces entrées viennent majoritairement de mauvais
> diagnostics de Claude.

- ⚠️ **Énoncer une cause sur du code non lu.** Occurrences répétées sur trois sessions : `AUD-33` déclaré soldé sans mesure · comparaison de hash de chunk entre deux environnements de build, **non falsifiable par construction** · service worker déclaré bloquant avant lecture du fichier — il ne cachait rien · `backdrop-filter` désigné comme cause unique d'`AUD-46`, patch sans effet · « le message le plus vu de la semaine » affirmé avant lecture — il ne s'affiche en réalité que sur panne Firestore pendant l'onboarding · seuil de 125 px posé dans un test de fidélité par calcul mental au lieu d'une mesure (réel : 128 px, test rouge sur du code correct) · `J10e-a` planifiée 3 sessions durant sans que sa spécification existe dans aucun document.
  ✅ **Contre-mesure :** avant d'énoncer une cause, demander la lecture qui tranche. Une commande `type` coûte un aller-retour ; une hypothèse fausse en coûte trois et brûle un patch.
- ⚠️ **Écrire un chemin de fichier sans l'avoir lu.** 2 occurrences : `AUD-16` localisé dans `src/components/` au lieu de `src/components/ui/` (SE-073) ; `US-HEADER-MOBILE-B` livrée avec le même chemin faux (SE-074). L'agent corrige silencieusement, ce qui produit un **faux signal `R-SCOPE-1`** contre lui en review.
  ✅ **Contre-mesure :** tout chemin écrit dans une US provient d'un `findstr` ou d'un `type` de la même session, jamais de mémoire.
- ⚠️ **Proposer un fix avant d'avoir lu le code qui fonctionne déjà.** Quand un test échoue et qu'un test similaire passe, **lire le test qui passe EN PREMIER**.
- ⚠️ **Bâtir un diagnostic sur une hypothèse jamais grep-ée.** Trois occurrences : une `inject` supposée ratée (c'était un placement), un `syncAnimeUpdates` supposé écraser le store (c'était un seed mono-jour), une classe supposée absente (elle existait).
- ⚠️ **Théoriser une cause racine en changeant deux variables à la fois.** **Une mesure = une variable.** Aucune cause racine n'est gravée avant qu'une hypothèse n'explique 100 % des mesures.
- ⚠️ **Diagnostiquer une panne externe sans la remesurer.** Une panne portée **5 sprints** sur la foi d'un unique curl, gelant 2 items du backlog.
- ⚠️ **Inférer l'état d'un service depuis des requêtes vers un autre domaine.** Les jaquettes viennent d'un CDN, et leurs URLs sont en cache local : une page pleine de posters en 200 n'indique **rien** sur la santé de l'API.
- ⚠️ **Confondre le cache HTTP du navigateur et le `localStorage` applicatif.** « Disable cache » ne touche **jamais** au `localStorage`.
- ⚠️ **Traiter un handoff comme une source primaire.** Un handoff a affirmé l'existence de trois symboles **tous inexistants**. **Le code réel tranche.**
- ⚠️ **Planifier depuis un backlog jamais confronté au code.** Trois US planifiées « à faire » alors qu'elles étaient en production.
- ⚠️ **Prendre le diagnostic du PO pour une cause.** Le PO décrit un **symptôme** ; vérifier avant de spécifier.
- ⚠️ **Extrapoler une clôture d'audit à un cas voisin.** Deux constats (`AUD-13`, `AUD-17`) ont été fermés à tort : **un constat portant sur N éléments ne se ferme qu'après vérification des N.**
- ⚠️ **`findstr` : un vide n'est une preuve d'absence que si la syntaxe est vérifiée.** Chaîne littérale → `/c:"…"`. OU logique → mots séparés par des espaces. Détail → `CLAUDE.md §4`. Deux fausses conclusions déjà tirées de vides syntaxiques, et ~800 lignes déversées en SE-074 faute de `/c:`.

### Chercher l'état impossible, pas rejouer le parcours

Un bug qui survit des dizaines de sprints se cache dans une combinaison d'états que le
développement ne produit jamais. **Construire l'état artificiellement bat le fait de rejouer le
parcours utilisateur.** *Occurrence : `AUD-30` a résisté à trois hypothèses ; il est tombé en 30 s
avec « `localStorage` vidé à la main + `F5` sans se déconnecter » — page chargée en état connecté
avec un cache vierge. En dev, le cache est toujours chaud.*

### Un test de fidélité vert ne prouve pas que la spec vise la bonne branche

La porte verte prouve que le code fait ce que le test dit. Elle ne prouve **jamais** que le test
dit la bonne chose. Sur une US 🔴 d'orchestration, l'assertion doit porter sur la **séquence
réelle** (quelle navigation, depuis quel état, déclenchée par quoi), pas sur la condition telle
qu'on l'imagine. *Occurrence : `US-PERSIST-P0a` mergée 270/270 verte, zéro effet — spec et test
visaient tous deux une branche jamais atteinte.* **Remède : `DEC-162`.**

## 8. Process & hygiène de livraison

- ❌ **Fichiers de travail commités** (`wait.txt`, `*_out.txt`, `test-*.cjs`, fichiers de debug). Le débogage reste local. Vérifier par `git show --name-only HEAD`, **jamais** `git diff` (après un `pull` l'arbre est propre et ne prouve rien).
- ❌ **Inférence promue en fait par son entrée dans un document.** Un test déclaré « rouge qualifié » sur la seule lecture de son code — il était vert. **Un test n'est rouge que sur sa sortie d'exécution.** Sans sortie collée par le PO : « suspecté », jamais « qualifié ». Le mot choisi dans un document engage : il sera lu comme un fait mesuré à la session suivante.
- ❌ **Modification documentaire non isolée → faux signal de dérive de périmètre.** Deux occurrences : patch doc envoyé dans le même message qu'une US, puis déploiement d'`AGENTS.md` non commité, embarqué par le commit suivant de l'agent. **Le déploiement d'`AGENTS.md` se commite seul, avant toute nouvelle US.** Un `git status` vide est **condition d'ouverture** d'une livraison.
- ❌ **Sortie de test d'un run antérieur présentée comme fraîche** (2 occurrences). Seule la machine PO fait foi (`R1`).
- ❌ **Jamais d'US non envoyable.** Un bloc `## [US-XXX]` n'est produit que s'il est **immédiatement collable à Gemini** : test de fidélité inclus, types vérifiés contre le code source. Pas de brouillon, pas de « test à suivre », pas de placeholder. S'il manque un fichier : demander le fichier, ne rien écrire. *Occurrence : une US livrée sans son `.spec.ts`, une conversation entière perdue.*

---

## 🎓 Les 3 leçons de méthode les plus chères

1. **Le vert ne prouve rien sur l'utilisabilité.** Type-check + tests + build au vert ≠ application fonctionnelle ≠ application utilisable. Quatre bugs runtime et toute la famille des events désalignés sont passés au vert intégral. D'où `R2`, `R4` et l'audit live du PO.
2. **Un cadre d'audit identique révèle les angles morts.** Un dual audit (deux auditeurs, mêmes axes, même barème, même format) a montré que **chacun avait raté le finding n°1 de l'autre**. Sans cadre commun strict, on ne compare que du bruit.
3. **Les incidents les plus coûteux ne sont pas des défauts de code, mais de fraîcheur de fait.** Une panne externe jamais remesurée (5 sprints gelés) et un backlog jamais confronté au code (3 US planifiées sur du déjà livré) ont coûté plus que tous les bugs de typage réunis.

> **Application positive, SE-069.** `AUD-46` a été résolu à la première lecture demandée, sans
> aucune hypothèse préalable. Deux constats supplémentaires (`AUD-47`, `AUD-50`) sont sortis de
> ces mêmes lectures — dont un qui aurait cassé l'US suivante. **La lecture préalable n'est pas un
> coût de vérification, c'est une source de constats. Elle rapporte plus qu'elle ne coûte.**

---

## 9. 🔄 Sprint courant

> Les pièges du sprint en cours s'ajoutent **ici, à la fin**, une fois par sprint, depuis
> `HISTORIQUE §2 Tampon`. À la purge H8, ils rejoignent leur section thématique.

*(vide)*
