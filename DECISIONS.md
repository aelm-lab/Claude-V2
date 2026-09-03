# DECISIONS.md — Décisions actives

> **Rôle :** les choix encore appliqués aujourd'hui, **une ligne chacun, un seul format**.
> **Pas ici :** décisions closes ou superseded (→ `DECISIONS_ARCHIVE.md`) · état courant
> (→ `STATE.md`) · règles opposables (→ `AGENTS.md`, `PILOTAGE.md`) · pièges répétés
> (→ `ANTIPATTERNS.md`).
>
> **Mise à jour : une fois par sprint**, en append dans le `§10 Sprint courant` (`DEC-190`). En
> cours de sprint, un `DEC` neuf s'écrit en une ligne dans `HISTORIQUE §2 Tampon`.
>
> **Un numéro n'est jamais supprimé ni réattribué.** Une décision contredite est marquée
> `⛔ SUPERSEDED PAR DEC-xxx` et bascule dans l'archive ; les renvois restent résolvables.

**Dernier numéro attribué : `DEC-191`.**

> 🔴 **Collisions d'identifiants résolues — à connaître avant de citer :**
> `DEC-158` désignait deux décisions ; la seconde (rechargement complet du navigateur) est
> devenue **`DEC-176`** (SE-069, `AUD-49`). `DEC-159` désignait deux décisions ; la seconde
> (garde anti-écrasement sur le watcher) est devenue **`DEC-191`** (SE-075).

---

## 1. Architecture & couches

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-26** | 🔴 `watch → saveToDatabase` vit dans `usePersistence`, jamais dans le store. **Store sans I/O** | Un store qui écrit devient intestable et sauvegarde à des moments imprévisibles |
| **DEC-41** | 🔴 `stores/ui.ts` pilote **tous** les overlays | Deux sources d'ouverture = modales concurrentes, fermeture partielle |
| **DEC-28** | 🔴 Guard auth : singleton `auth` + `await auth.authStateReady()` | `useFirebaseAuth()` hors `setup` casse ; `currentUser` est `null` avant le `await` |
| **DEC-31** | `isBooting` via `provide`/`inject` avec **fallback `ref(false)` obligatoire** | Un `inject` typé sans fallback lève au montage isolé (tests) |
| **DEC-27** | Double bloc `<script>` + `<script setup>` pour exporter un Symbol depuis `App.vue` | Un export nommé depuis `<script setup>` est impossible |
| **DEC-07** | Les DTO de plomberie restent des types locaux, hors contrat | Le contrat se dilue en types jetables |
| **DEC-55** | Deux documents d'architecture séparés : technique et fonctionnelle | Un seul document mêle « ce que fait l'app » et « comment c'est câblé » |

## 2. Boot & persistance

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-50** | 🔴 Orchestration du boot entièrement dans `App.vue` : `load → await sync → await buildRelationMemory → reScorePool`. Couverte par `App.spec.ts` | Un rescore avant la sync score des données vides ; le smoke test est le seul filet |
| **DEC-59** | Boot loader en **deux phases** : loader statique dans `index.html` + `<LoadingOverlay>` à la racine d'`App.vue`, **hors gate auth** | Sous la gate auth, l'overlay n'est pas monté pendant la résolution → écran vide |
| **DEC-72** | 🔴 `#boot-loader` vit **hors de `<div id="app">`**, supprimé par `.remove()` dans le `finally` du `onMounted` (exception `R-CODE-4`). `main.ts` reste un `app.mount` simple, **sans** `router.isReady()` | Le loader est `position:fixed` et intercepte tous les pointer events ; `router.isReady()` casse `boot-loader.spec.ts` |
| **DEC-85** | 🔴 Toutes les clés `localStorage` préfixées `aanime_`, migration legacy au boot dans `loadFromDatabase`. Registre → `ARCHITECTURE_TECHNIQUE §7` | Une clé hors convention = données invisibles pour l'utilisateur existant |
| **DEC-123** | Clé primaire `mal_id` conservée (vocabulaire MyAnimeList, compatibilité des données existantes) | Changer la clé primaire invalide toutes les bibliothèques déjà persistées |
| **DEC-19** | `needsBroadcastSync` est une mutation réactive dans `usePersistence` — c'est elle qui déclenche le watch | Une mutation non réactive ne déclenche pas la sauvegarde |
| **DEC-161** | 🔴 Un document de cache dont `data` est vide **ne fait jamais autorité** face à Firestore, quel que soit son timestamp (`cacheTimestamp = doc.data.length > 0 ? doc.timestamp : 0`) | Un document vide plus récent gagne la comparaison de fraîcheur et rend la bibliothèque cloud définitivement inatteignable. Cause n°1 d'`AUD-30`, et elle ne se répare jamais toute seule |
| **DEC-176** | 🔴 **On entre et on sort de la session par un rechargement complet du navigateur**, jamais par une navigation SPA. Deux points : `router/index.ts` (transition `guestOnly` → `requiresAuth`) et `AppHeader.vue` (logout) | `App.vue` charge la base dans `onMounted`, une seule fois. Sans remontage, la lecture Firestore ne rejoue jamais après connexion, et l'état mémoire survit au logout |
| **DEC-160** | Dérogation DOM explicite : `window.location.assign()` autorisé dans `AppHeader.vue` (logout) et `router/index.ts` (entrée en session). Ajouté à la liste close des exceptions `R-CODE-4` | Une SPA ne remonte pas son composant racine à la connexion ni à la déconnexion. Aucune alternative Vue ne coupe les deux à la fois de façon atomique |
| **DEC-191** | La garde anti-écrasement vit **sur le watcher** (`hasLoadedOnce`), jamais dans `saveToDatabase`. *(ex-second `DEC-159`)* | 8 appelants légitimes appellent `saveToDatabase` en direct. Une garde dans la fonction les casserait tous et ferait rougir 8 specs |
| **DEC-164** | 🔴 Aucune porte accrochée à `router.beforeEach` ne peut lire l'état applicatif : le garde s'exécute **avant** le montage d'`App.vue`, donc avant `loadFromDatabase()`. Toute redirection conditionnelle se dérive d'une source lisible sans montage | Cas fondateur : `US-ONBOARD-PERSIST` dérivait l'onboarding de la présence d'une bibliothèque. Morte-née. Ça reviendra à chaque US de redirection |
| **DEC-166** | Plafond Firestore = **500 entrées** (`data.size() <= 500`, create ET update). Une entrée pèse ~1,1 ko : 500 ≈ 550 ko, 45 % de marge sous 1 Mo. **Toute modification de `firestore.rules` doit être PUBLIÉE en console** | Un plafond à 100 refusait silencieusement les sauvegardes d'une bibliothèque moyenne. Un fichier de règles modifié sans publication ne change strictement rien |

## 3. Données, normalisation & planning

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-124** | 🔴 La normalisation pose `day` + `airsTime` par cascade : (1) valeur existante jamais écrasée, (2) diffusion JST → jour + heure locale, (3) jour de semaine de la première diffusion **sans heure**. Le résidu est marqué `awaitingSchedule` et repromu par `useSync` | Sans `day`, une entrée `state:'calendar'` est stockée mais **invisible partout** |
| **DEC-131** | 🔴 **Pas de date de diffusion prouvée, pas de créneau.** Sans `nextAiringEpisode`, une série `Currently Airing` reçoit `awaitingSchedule = true` et sort de la grille | Une case fausse discrédite **toute** la grille, pas seulement sa ligne |
| **DEC-132** | `description` AniList est du HTML : nettoyage en TypeScript pur, **sans DOM** — `<br>` → `\n`, dépouillement **puis** décodage des entités, `\n{3,}` → `\n\n`, `trim`. Vide → `synopsis: undefined` | Décoder avant de dépouiller supprime un `&lt;i&gt;` que l'auteur voulait afficher littéralement |
| **DEC-47** | `synopsis?` fait partie d'`AnimeEntry` (`TYPES_CONTRACT §2`) | Un type réinventé localement diverge du contrat |
| **DEC-52** | 🔴 **Hiatus = une seule source**, computed à 14 j | Deux seuils pour une même règle métier = deux vérités affichées |
| **DEC-86** | `normalizeAnime` produit **toujours** `studios: string[]` | Sans studios peuplés, la dimension studio du scoring est morte |
| **DEC-84** | `POSTER_PLACEHOLDER` source unique · `onHiatus?` retiré du type · `episodeOverride` reseté à chaque upsert | Une copie inline du placeholder diverge ; un `onHiatus` persisté concurrence le calcul dérivé |
| **DEC-90** | Garde null-safety `genres ?? []` | Un cache legacy sans `genres` fait crasher `topGenres` |
| **DEC-10** | Après normalisation, `genres` / `themes` / `studios` sont toujours `string[]` — branches défensives supprimées | Du code mort qui teste l'impossible masque le vrai cas |
| **DEC-81** | Une entrée MyAnimeList `Dropped` n'est **pas** importée | Un import de 300 titres remplit la bibliothèque de séries abandonnées |
| **DEC-24** | `MalImportResult` expose `imported`, pas `entries` | Signature inventée = erreur de compilation ou champ toujours vide |
| **DEC-16** | `parseMalXml` est pur dans `utils/malImport.ts` ; la partie impure vit dans `useMalImport` | Un parseur impur n'est pas testable sans navigateur |
| **DEC-15** | `buildICSContent` est pur dans `utils/ics.ts` ; download + toast dans `useICS` | Génération de texte non testable autrement |
| **DEC-18** | Upsert du store : garder `if ('state' in input)`. Ne jamais recalculer `state` inconditionnellement en branche merge | Sinon clobber du `state` choisi par l'utilisateur |

## 4. Réseau AniList

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-139** | 🔴 `Page(page:1, perPage:1){ media(idMal:) }` obligatoire, **jamais** `Media(idMal:)` | `Media()` renvoie une erreur GraphQL sur une entrée introuvable ; le disjoncteur la compte comme un échec et **3 suffisent à couper AniList 60 s pour toute l'app** |
| **DEC-126** | Un **429 n'incrémente jamais** le compteur de panne du disjoncteur. `AUD-04` annulée : AniList n'expose qu'un endpoint, un breaker global y est correct | Une rafale de frappe dans la recherche fermerait le circuit et viderait le calendrier |
| **DEC-167** | L'attente sur un 429 est plafonnée à `MAX_RATELIMIT_WAIT_MS = 3000` ; `minIntervalMs` passe de 700 à **1200 ms**. L'attente reste DANS la boucle de retry — seule sa durée change | L'attente se fait dans une file unique partagée : respecter un `Retry-After: 60` gelait recherche, calendrier et fiches 60 à 120 s. 700 ms = 95 % du quota nominal, sans marge |
| **DEC-168** | La priorité de synchronisation classe par **INVISIBILITÉ** : `awaitingSchedule` (2) > `calendar` (1) > reste (0). Un anime en échec est horodaté en antidaté (réessai 15 min), sync plafonnée à 25 par démarrage | Sans ce classement, les 25 places vont aux entrées déjà visibles et un import MAL n'a jamais son tour. Sans horodatage sur échec, un anime rate-limité entretient sa propre famine |
| **DEC-141** | 🔴 Aucun throttle manuel dans `syncAnimeUpdates` : `anilistClient` sérialise déjà | Deux régulateurs empilés = deux vérités ; sync de 30 titres à ~50 s au lieu de ~21 s |
| **DEC-143** | Pool `incoming` = **2 appels** (saison courante + suivante, `perPage:50`) | Une 3ᵉ requête n'a plus d'objet et coûte du quota |
| **DEC-137** | Tri `POPULARITY_DESC` pour la requête de saison | — |
| **DEC-114** | Le cache de saison **périmé est servi** si le fetch échoue ; `error.value` renseigné mais jamais affiché | Évite une page vide, mais l'absence totale de cache donne une liste vide **silencieuse** → `AUD-05` |
| **DEC-60** | 🔴 `dedupeByMalId` = **source unique** de déduplication, clé `mal_id`, garde la 1ʳᵉ occurrence | Trois chemins de dédup indépendants divergent silencieusement |
| **DEC-127** | La déduplication de recherche s'applique **APRÈS** le tri | Appliquée avant, c'est la version terminée qui survit et l'utilisateur est renvoyé vers Completed pour une série suivable aujourd'hui |
| **DEC-74** | Déduplication appliquée **avant** le `slice` dans `getNextBatch` | Un batch de 12 rend 9 cartes sans explication |

## 5. Moteur de recommandations

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-144** | 🔴 `getSeasonNudges` **n'écrit ni ne lit** le store IDB `relations`, qui porte la forme MyAnimeList lue par `buildRelationMemory` | Y écrire des `AnimeRelation` AniList corromprait le scoring **en silence**, sans erreur ni test rouge |
| **DEC-140** | `AnimeRelation` normalisé ; le filtrage des types de relation vit dans le composable, **jamais** dans le `.vue` | Un filtre dans la vue est invisible aux tests unitaires |
| **DEC-12** | `decayMultiplier = 0.2` dans `buildTasteProfile` | Constante de calibrage — la changer déplace tout le scoring |
| **DEC-13** | Priorité de tri des signaux typée `Record<RecSignalKind, number>`, `score: 0` | Un membre d'union non mappé tombe silencieusement au défaut |
| **DEC-14** | `extractBecauseYouWatched` : paramètre inutilisé préfixé `_profile`, signature publique préservée | `TS6133` ne se silencie pas avec `eslint-disable` |
| **DEC-79** | Réactivité Discover par **dérivation** : `excludedIds = union(store, dismissedRecIds)` | Un canal modal → page laisse des cartes fantômes selon le chemin d'ajout |
| **DEC-83** | Le skip d'une suggestion slot-fill est **session-only** (`ref<Set<number>>` local, jamais persisté) | « Écarter pour l'instant » ≠ « bannir » |
| **DEC-159** | Depuis This Season, le rejet d'un anime est **session-only**. Skip partout sur cet écran, y compris dans la modale qui en est issue. `dismissRec()` + `trackNegative()` restent réservés à For You | Ne pas vouloir un titre cette saison n'est pas ne jamais vouloir le voir. Mélanger les deux fabrique des recommandations punies à tort |
| **DEC-180** | `RecCard` accepte une prop `showSkip` (défaut `true`). *This Season* la passe à `false` : `DEC-159` devient **sans objet** sur cet écran | Un même bouton avec deux portées invisibles est un piège : masquer vaut mieux qu'expliquer |
| **DEC-145** | `J10e` (repli des orphelins par titre + année) éclatée en 3 slices, en backlog. 🔻 **Gelée en SE-074** : spécification absente de tout document, fréquence jamais mesurée | `normalizeAniList` rejette tout média sans `idMal` — un orphelin est précisément ça. Un mauvais rattachement est **pire** que pas de rattachement |

## 6. UI, feedback & vocabulaire

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-61** | 🔴 **Le composant définit son contrat d'emit ; les consommateurs s'alignent.** Ne jamais renommer un emit pour matcher un listener | `RecCard` a eu Add, clic carte et « pas intéressé » morts sur toutes les surfaces, avec **0 erreur console** |
| **DEC-58** | Corollaire : un désalignement de nom d'event se corrige **côté page**, jamais côté composant | Renommer l'emit casse les N autres consommateurs |
| **DEC-62** | Toute action d'ajout ou de déplacement produit un **toast nommant la destination visible exacte** | Une action sans feedback est indistinguable de « rien ne s'est passé » |
| **DEC-172** | Dans `useAddAnime`, le toast dérive de l'état **APPLIQUÉ**, la synchronisation de l'état **DEMANDÉ**. ⛔ Le message s'aligne sur le store, **jamais l'inverse** | Le store peut démoter une entrée (auto-vault, `calendar` sans `day`). Brancher la sync sur l'état appliqué la désactiverait exactement quand elle est nécessaire |
| **DEC-63** · **DEC-95** | Vocabulaire visible figé : « Coming Soon », « Completed », « Finished airing » — jamais « Radar » ni « Vault » | L'utilisateur ne voit ces mots nulle part dans l'UI |
| **DEC-73** | Toast « Moved to Completed » au boot pour l'auto-vault | Une série disparaît de la semaine sans explication |
| **DEC-96** | L'état « ✓ Added » est **cliquable** : retire l'anime d'où qu'il soit + toast « Removed » | Sinon l'ajout est irréversible depuis la recherche |
| **DEC-93** · **DEC-77** | Le bouton ✓ « Mark done » et la ligne recency sont **gatés sur `isFinished`** ; masqués sur Airing / Hiatus. L'action reste dans la modale | L'action n'a de sens que sur un titre terminé |
| **DEC-129** | Ligne « prochain épisode » affichée seulement si `airsTime` existe et que la série n'est pas terminée. Pas d'`airsTime` → **aucune ligne**, jamais « time unknown » | Un repli textuel affiche une certitude que la donnée ne porte pas |
| **DEC-173** | `markOnboarded()` est appelé **AVANT** `saveToDatabase()`, et la redirection est garantie. ⛔ Ne jamais l'inverser « pour n'onboarder qu'en cas de succès » | Un échec de sauvegarde ne doit ni bloquer la fin de l'inscription, ni la faire rejouer : le tunnel complet serait rejoué à chaque échec réseau |
| **DEC-179** | Un compte retrouvé côté Firestore alors que le flag local est absent **n'interrompt pas** l'onboarding : il affiche une **bande de rattrapage** persistante, avec sortie directe vers le calendrier | Faire attendre le réseau ralentirait le démarrage de tout le monde pour un cas minoritaire |
| **DEC-163** | La vue Mois est remplacée par un **« Coming Soon » assumé** : 42 cellules × 4-6 entrées sur 390 px n'est pas réparable en CSS. `MonthDayCell.vue` et les classes `month-*` sont **CONSERVÉS** pour la refonte | Retirer une fonctionnalité peut être un gain produit. Une grille illisible fait douter rétroactivement de tout le reste de l'app |
| **DEC-66** | Le libellé de période a une **source unique** dans `CalendarNavControls` | Deux libellés divergent d'un écran à l'autre |
| **DEC-76** | Snap-to-today : `await nextTick()` + `findIndex(d => d.isToday)` + `scrollIntoView`, garde `todayIndex < 0`, en `onMounted` **et** `onActivated` (KeepAlive). Pas de re-snap sur Prev/Next | Sans `onActivated`, le retour via KeepAlive ne resnappe pas |
| **DEC-33** | `ToastNotification` monté **uniquement** dans `AppLayout` | Conséquence connue et traitée : `AUD-06` |
| **DEC-42** | `modalContext` : `libraryRec` prioritaire | Deux contextes concurrents ouvrent le mauvais gabarit |
| **DEC-36** | `ChipsStrip` : la chip `all` émet `null` | Une chaîne vide serait traitée comme un filtre |
| **DEC-37** · **DEC-38** | `WeekAnimeItem` reçoit `info` en prop ; `MonthDayCell` reçoit `animes` **déjà filtrés** | Recalculer dans la carte multiplie les calculs par ligne |
| **DEC-34** | Lazy-load d'image via `<img style="display:none">`, pas `new Image()` | `new Image()` échappe au cycle de vie Vue |
| **DEC-35** | Dismiss de `SeasonNudgeCard` via `<Transition @after-leave>`, pas `setTimeout` | Un timer désynchronisé laisse un trou visuel |
| **DEC-44** | Pas de prefetch de covers : fallback `@error` | Le prefetch doublait les requêtes image pour rien |
| **DEC-43** | `removeAnimeWithUndo` simplifié — l'undo est une dette assumée | — |
| **DEC-82** | Redirect post-login = `/`, pas la route d'origine | Choix assumé : le lien magique expire rarement, ROI faible |
| **DEC-91** | Route `/stats` derrière le guard auth | Les stats sont personnelles |
| **DEC-88** · **DEC-89** | `useStats` = composable dédié, `StatsPage.vue` = page pure ; `topGenres` scoped au **contenu terminé cette année** | Un year-in-review compte ce qui a été consommé, pas les intentions |
| **DEC-94** | Règle de titre centralisée dans `getAnimeTitle` : anglais primaire + rōmaji secondaire si différent | Deux règles de titre = deux affichages pour la même série |
| **DEC-23** | `useTheme` applique la classe `dark` sur `<html>` | Sur `<body>`, les variables CSS racine ne basculent pas |

## 7. CSS & layout

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-107** | 🔴 `width:100%` + `padding` **sans `box-sizing:border-box`** faisait déborder le **document** de 30 px. Correctif : `box-sizing:border-box` + `flex:1 1 0; min-width:0` | Cause racine unique du débordement horizontal **et** du décentrage apparent des modales `position:fixed`. Le symptôme apparaît très loin de sa cause |
| **DEC-111** | 🔴 `.aa-card-grid` = classe partagée, colonnes fixes par breakpoint | Un `minmax(160px,1fr)` sans marge bascule en 1 colonne pour 5 px ; une classe en `<style scoped>` n'est **jamais** appliquée ailleurs |
| **DEC-177** | `.app-layout .app-header` porte `position: relative; z-index: 200` — contexte d'empilement **explicite et positionné**, entre `.secondary-nav-sticky` (100) et `.modal` (10000) | Un `z-index` sur un élément `position: static` est **ignoré**. Trois valeurs successives ont été posées sans effet |
| **DEC-185** | 🔴 Dans un `<style scoped>`, un sélecteur d'ancêtre global (`html.dark`, `body.x`) s'écrit **nu**, jamais dans `:deep()`. `:deep()` ne sert qu'à percer vers un descendant d'un composant enfant | Les 6 règles `:deep(html.dark)` d'`AppHeader.vue` ne s'appliquaient à rien. Le thème sombre de l'en-tête était mort depuis sa création — et **un sélecteur qui ne matche rien ne produit aucune erreur** |
| **DEC-99** | Header au scroll = **sticky CSS pur**, aucune logique JS de scroll-hide | `v-show`/`display:none` en cours de scroll provoque des sauts de hauteur |
| **DEC-97** | Les couleurs réutilisent les tokens existants (`var(--airing)`, `var(--upcoming)`) | Une couleur en dur ne suit pas le mode sombre |
| **DEC-67** | Convention de classe active de nav = `.active` | Le markup doit suivre le CSS, pas l'inverse |

## 8. Tests & E2E

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-104** | 🔴 **L'auteur du test ≠ l'auteur du code** (`R7`), sur tout ce qui touche **un écran ou la persistance** | Un test auto-écrit valide le code tel qu'il est, pas le comportement attendu — écarté sans valeur de preuve |
| **DEC-57** | **R4** : tout correctif UX livre un E2E — geste réel, assertion sur le DOM visible, ROUGE puis VERT **sans modification du test** | Un vert obtenu en modifiant le test ne prouve rien |
| **DEC-186** | **R4-ter** — contenu ↔ E2E. Texte intégral dans `AGENTS.md §5` | `US-HEADER-ICONS` a retiré l'emoji 🚪 ; une spec le cherchait par `hasText: '🚪'`. Rouge **au sweep**, pas au merge |
| **DEC-56** | 🔴 Bypass d'auth E2E lu en **statique** (`import.meta.env`), branche éliminée du bundle (prouvable `grep -c` = 0) ; `tests/e2e/**` exclu de Vitest | Une lecture runtime laisserait le bypass vivant en production |
| **DEC-65** | **R5** : 1 test ciblé par US pendant l'epic, grand check en fin d'epic, specs cumulatives jamais supprimées | Un test cumulé rouge est une régression à corriger, pas un test à retirer |
| **DEC-109** | La suppression d'une spec **non enregistrée** au registre est autorisée | Un `debug-*.spec.ts` hors registre n'a aucune valeur de preuve |
| **DEC-112** | Pour une propriété de conteneur : `toHaveCount` puis `getComputedStyle`, pas `toBeVisible` | Un `display:grid` sans enfant a une hauteur de 0 px et paraît invisible alors que le CSS est correct |
| **DEC-80** | Un anime `calendar` + terminé s'auto-vault au boot : le scénario « Mark done » y est **structurellement impossible**. Les specs testent sur `watchlist` | Un test écrit sur le cas impossible est rouge sans qu'aucun code ne soit fautif |
| **DEC-149** · **DEC-150** | 🔴 Le harnais E2E passe par le helper mutualisé `tests/e2e/_helpers/anilistMock.ts`, qui discrimine par **corps de requête**, pas par URL. Quatre règles ordonnées : `search` · `season` · `idMal` **+ `relations`** · `idMal` seul · aucune variable → top finished. Le helper **importe** `resolveSeason`/`resolveNextSeason` | Détail et relations partagent `idMal` : sans le jeton `relations`, les deux se confondent. Un mock dupliqué casse **au sweep**, pas au merge |
| **DEC-183** | Une spec qui ne teste ni contenu ni données (layout, navigation, débordement, état vide) installe `installAniListMock(page, {})` — **mock total à vide**, aucune graine | Mesuré en SE-071 : `grid-two-columns` et `no-horizontal-overflow` passent avec une grille vide. **Un mock à vide est plus fort qu'un mock peuplé** : il prouve que le composant ne dépend pas du réseau |
| **DEC-184** | Des specs **indépendantes** migrent en lot : N fichiers, un geste identique, une seule gate. L'interdit porte sur deux changements dans un **même** fichier | Playwright nomme le fichier fautif : sur des fichiers disjoints la cause reste identifiable. Appliqué 3 fois en SE-071, zéro rouge |
| **DEC-187** | 🔴 Un mock de test doit reproduire **tous les champs que le code de production filtre**, pas seulement ceux qu'il affiche. Relire les `continue` / `if (…) return` de la fonction consommatrice avant d'écrire une fixture | `relationsBody` omettait `type: 'ANIME'` → le helper était incapable de produire une relation depuis sa création. Le champ n'est jamais affiché : aucune assertion ne pouvait le révéler |
| **DEC-162** | Avant tout test de fidélité portant sur un garde de routeur, une machine à états ou un ordre d'orchestration, la séquence réelle est rejouée dans un **bac à sable exécutable**, et l'échec AVANT correctif est prouvé | Un test de fidélité qui vise la mauvaise branche est **vert par construction**. Cas fondateur `US-PERSIST-P0a` — mergée verte, zéro effet |
| **DEC-165** | Avant d'écrire un test d'intégration traversant le store, **lire les gardes de `addAnime` / `addAnimeSilent`** (`stores/anime.ts:39-53`) : elles réécrivent le state fourni dans au moins trois cas | Le test rouge d'`US-MALIMPORT-FIX` était une erreur de spec, pas de code. **Un test faux fait échouer un code correct et coûte un tour d'escalade complet** |
| **DEC-170** | Une US de plomberie réseau (file, quota, TTL, ordre de sync) peut déroger à l'E2E d'une 🟠, sur déclaration explicite et validation PO. Gate alors exigée : R1 complète + test de fidélité. **Jamais sur un élément d'écran** | Un délai réseau n'est pas observable par un geste Playwright. Exiger un E2E produirait une spec fantoche ou un test de timing fragile — un faux vert en puissance |
| **DEC-153** | Les specs vertes-mais-démockées migrent **après** la gate, pas avant | Toucher 12 specs vertes juste avant une Sprint Outcome Gate en fabrique trois rouges |
| **DEC-152** | ⛔ « More like this » n'est **pas** masqué : il est **rebranché** sur `fetchRelationsByMalIdWithMeta` | Masquer ajoute un `v-if` sur un élément qu'une spec clique → rouge au sweep. Le bouton doit redevenir utile, pas disparaître |
| **DEC-122** | Une gate 🔴 peut être satisfaite par un **test unitaire dédié** quand l'E2E est structurellement impossible. Motivé au cas par cas | Cas fondateur `AUD-02` : le SDK Firestore met les écritures en file locale hors-ligne, aucun rejet n'est observable depuis le navigateur |
| **DEC-54** | Filet de sécurité **avant** correctif : CI + smoke test + un test rouge encodant le bug | Corriger sans filet, c'est corriger sans preuve |
| **DEC-08** | Fixtures de test typées via helper `Partial` ou factory. Interdit : `as any`, `as unknown as T` | Une fixture castée ne peut pas détecter une entité incomplète |
| **DEC-06** | `jsdom` déclaré **par fichier** via `// @vitest-environment jsdom` | Un environnement global ralentit toute la suite |
| **DEC-05** | Le conteneur de l'agent tourne nativement en UTC — pas de préfixe `TZ=UTC` | Un préfixe inutile masque un vrai problème de fuseau |

## 9. Gouvernance, process & outillage

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-190** | 🔴 **Cadence documentaire.** En cours de sprint : **1 seul geste par session**, un bloc collé à la fin d'`HISTORIQUE.md` (1 ligne de session + le tampon). À la clôture de sprint : **le lot** — `STATE` et `ROADMAP` remplacés, `DECISIONS`/`AUDIT`/`ANTIPATTERNS` en append de fin. Détail → `PILOTAGE §6`. ⛔ **Supersede `DEC-146`** | Mesuré sur SE-064→SE-073 : la régénération par session produisait 2 366 lignes de patchs pour 31 US, dont **79 % sur `STATE.md` et `ROADMAP.md` seuls**. `STATE.md` fait 80 lignes et a été réécrit 15 fois son volume en 10 sessions |
| **DEC-189** | 🔴 **Toute US touchant une surface visible est précédée d'une maquette validée par le PO**, montrant l'état actuel, les options, et le coût de chacune (dont les specs E2E cassées). Pas de maquette, pas d'US | Une description en prose d'un changement visuel n'est pas vérifiable : le PO valide un texte et découvre un écran. Cas fondateur SE-074 — l'option « une seule rangée » semblait supérieure en prose ; la maquette a montré qu'elle enterrait 5 actions derrière un menu, la semaine de l'ouverture bêta |
| **DEC-155** | Un sprint fait **10 slots : 7 planifiées + 3 flex** (bugs sortis en route, retours bêta qualifiés). **Clôture à date, jamais à épuisement** — le non-fini glisse dans `ROADMAP.md`. Un retour bêta entre en flex, jamais en interrompant l'US `In Progress` | Sans clôture à date, un sprint qui absorbe les retours ne finit jamais : plus aucun bump, plus aucune Sprint Outcome Gate |
| **DEC-157** | **Boucle bêta :** retour brut du PO → qualification par Claude (impact utilisateur, effort, risque) → slot flex du sprint courant ou `ROADMAP.md`. **Jamais de correctif « au vol » hors US.** Critère d'entrée en flex → `PILOTAGE §4` | Un correctif sans US ni gate est exactement le chemin qui a produit les quatre promesses fausses de l'audit S38 |
| **DEC-120** | L'app est en **bêta avec des testeurs réels** : tout P0 sur un parcours d'entrée est bloquant | Un défaut d'onboarding touche le premier écran de chaque nouvel utilisateur |
| **DEC-156** | La vision multi-sprints vit dans `ROADMAP.md`. `STATE.md` ne porte que le sprint courant et renvoie | Un cap dans `STATE.md` explose son plafond ; un cap nulle part oblige à re-débattre les priorités à chaque session |
| **DEC-154** | `DECISIONS.md` ne porte que les décisions **actives**, en tableau `ID \| Décision \| Pourquoi ça casse`. Les closes basculent dans `DECISIONS_ARCHIVE.md`. **Les numéros ne sont jamais supprimés ni renumérotés** | Un journal chronologique de 717 lignes ne se lit plus : il se cherche. Chaque doublon évince un extrait pertinent de l'index |
| **DEC-53** · **DEC-121** | `AGENTS.md` est la gouvernance permanente de l'agent, en **lecture seule** pour lui. Mise à jour par patch verbatim uniquement. Circulation unique : Knowledge → repo `A-Anime` | C'est le **seul** document qu'il lit. 3ᵉ occurrence de `R-SCOPE-1` sur ce fichier |
| **DEC-128** | Les **micro-patchs** (≤ 10 lignes, 1 fichier, entièrement dictables) ne passent pas par l'agent : Claude produit, le PO colle. **Réservé aux changements sans logique métier** | Dès qu'une décision de comportement est en jeu, la dictée verbatim rend une erreur de spec indétectable |
| **DEC-169** | 🔴 **Aucune sortie de commande de Gemini n'a valeur de preuve** — build, tests ET type-check. Son `node_modules` diverge : noms de chunks structurellement différents, écart de build reproductible de 71 kB sur 3 livraisons | `R1` le posait déjà pour les tests. On sait maintenant pourquoi, et que ça s'étend au type-check : un `vue-tsc` sur une autre version de TypeScript peut être vert chez lui et rouge chez nous |
| **DEC-87** | 🔴 **Un handoff est une source secondaire faillible ; le code réel tranche.** Quatre « faits » hérités d'un handoff se sont révélés faux | Une inférence recopiée devient un fait au bout de deux sessions |
| **DEC-174** | Le handoff se déclenche à **80 %** de capacité, pas 90 % | Un handoff coûte 8-10 % à produire ; déclencher à 90 % garantit un document tronqué |
| **DEC-188** | 🔴 **Format de réponse au PO — 4 blocs, pas plus.** Texte intégral dans `PILOTAGE`, en-tête | Coût mesuré en SE-073 : 85 % de capacité consommée pour 2 US, en majorité en démonstrations que le PO ne lisait pas. Une explication non demandée n'est pas de la pédagogie, c'est de la dette de contexte |
| **DEC-110** | Le diagnostic du PO pointe un **symptôme**, pas une cause : vérifier avant de spécifier | Une US entière a été écrite pour un défaut qui n'existait pas sous cette forme |
| **DEC-175** | `AUD-37` est **délégué aux bêta-testeurs, pas soldé**. Contrepartie obligatoire : la note aux testeurs demande « combien de titres avais-tu, combien en retrouves-tu ? » | Sans la question, un import cassé produirait un retour inexploitable (« ça marche pas ») au lieu d'un signal |
| **DEC-100** | Devant une régression introduite par un effet visuel, **retirer** plutôt qu'empiler un correctif | Un empilement de correctifs enterre la cause |
| **DEC-133** | Le temps de démarrage de référence est celui de la **production** (2,5 s, bundle prêt à 152 ms, 15 requêtes) — pas la mesure du serveur de dev. **Aucun chantier de bundle n'est ouvert** | Un chantier décidé sur le chiffre de dev vise un fantôme |
| **DEC-147** | Le worker de relations en fond est **supprimé sans remplacement** | Il attendait 1,1 s par requête sur toute la bibliothèque, forçait `isSyncing` ~1 min et déposait son résultat dans un bloc vide |
| **DEC-98** | `npm install` fonctionne en direct ; `--legacy-peer-deps` supprimé | Une parade laissée en place masque le retour du conflit |
| **DEC-02** | ESLint = flat config + `@vue/eslint-config-typescript` avec `no-explicit-any` **en erreur** | ⚠️ La configuration existe mais **aucun script `lint` n'existe dans `package.json`** : ESLint n'a jamais tourné (`AUD-08`) |
| **DEC-03** | `tsconfig.node.json` séparé pour isoler `vite.config.ts` | Sans isolation, la config de build pollue le type-check applicatif |
| **DEC-178** | La règle « cet anime est en cours de diffusion » a **une seule implémentation applicable** : `useAddAnime.resolveTargetState`, qui reconnaît `Currently Airing` **et** `Continuing`. Doublon résiduel connu : `utils/onboardingFilter.ts` (`AUD-03`) | Trois exemplaires divergents coexistaient. Migrer la recherche sans corriger aurait envoyé tout anime en cours dans *Plan to Watch* |

---

## 10. 🔄 Sprint courant

> Les `DEC` du sprint en cours s'ajoutent **ici, à la fin**, une fois par sprint, depuis
> `HISTORIQUE §2 Tampon`. À la purge H8, ils rejoignent leur section thématique.

*(vide)*
