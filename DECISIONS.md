# DECISIONS.md — Décisions actives

> **Rôle :** les choix encore appliqués aujourd'hui, une ligne chacun. Un numéro n'est jamais supprimé ni réattribué.
> **Pas ici :** décisions closes, périmées ou de migration (→ `DECISIONS_ARCHIVE.md`, hors ordre de lecture) · état courant (→ `STATE.md`) · règles opposables (→ `AGENTS.md`, `PILOTAGE.md`) · pièges répétés (→ `ANTIPATTERNS.md`).

**Dernier numéro attribué : DEC-153.** Une décision contredite est marquée `⛔ SUPERSEDED PAR DEC-xxx` et bascule dans l'archive ; les renvois `DEC-xx` des autres documents restent résolvables.

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
| **DEC-50** | 🔴 Orchestration du boot entièrement dans `App.vue` : `load → await sync → await buildRelationMemory → reScorePool`. Couverte par `App.spec.ts` (l'étape worker de fond est tombée avec DEC-147) | Un rescore avant la sync score des données vides ; le smoke test est le seul filet |
| **DEC-59** | Boot loader en **deux phases** : loader statique dans `index.html` + `<LoadingOverlay>` à la racine d'`App.vue`, **hors gate auth** | Sous la gate auth, l'overlay n'est pas monté pendant la résolution → écran vide |
| **DEC-72** | 🔴 `#boot-loader` vit **hors de `<div id="app">`**, supprimé par `.remove()` dans le `finally` du `onMounted` (exception R-CODE-4). `main.ts` reste un `app.mount` simple, **sans** `router.isReady()` | Le loader est `position:fixed` et intercepte tous les pointer events ; `router.isReady()` casse `boot-loader.spec.ts` |
| **DEC-85** | 🔴 Toutes les clés localStorage sont préfixées `aanime_`, avec migration legacy au boot dans `loadFromDatabase`. Registre → `ARCHITECTURE_TECHNIQUE.md §7` | Une clé hors convention = données invisibles pour l'utilisateur existant |
| **DEC-123** | Clé primaire `mal_id` conservée (vocabulaire MyAnimeList, gardé pour la compatibilité des données existantes) | Changer la clé primaire invalide toutes les bibliothèques déjà persistées |
| **DEC-19** | `needsBroadcastSync` est une mutation réactive dans `usePersistence` — c'est elle qui déclenche le watch | Une mutation non réactive ne déclenche pas la sauvegarde |
| **DEC-146** | 🔴 `STATE.md` (présent, **régénéré intégralement**) + `HISTORIQUE.md` (passé, **append-only**). Tous les autres documents sont **patchés à chaque clôture de session**, jamais batchés au sprint | Un `STATE.md` trop lourd saute des sessions « faute de capacité » ; une décision différée au handoff se perd ou se déforme |

## 3. Données, normalisation & planning

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-124** | 🔴 La normalisation pose `day` + `airsTime` par cascade : (1) valeur existante jamais écrasée, (2) diffusion JST → jour + heure locale, (3) jour de semaine de la première diffusion **sans heure**. Le résidu sans source est marqué `awaitingSchedule` et repromu par `useSync` | Sans `day`, une entrée `state:'calendar'` est stockée mais **invisible partout** |
| **DEC-131** | 🔴 **Pas de date de diffusion prouvée, pas de créneau.** `normalizeAniList` ne déduit jamais un jour depuis la première diffusion. Sans `nextAiringEpisode`, une série `Currently Airing` reçoit `awaitingSchedule = true` et sort de la grille | Une case fausse ne se détecte pas à l'œil : elle discrédite **toute** la grille, pas seulement sa ligne |
| **DEC-132** | `description` AniList est du HTML : nettoyage en TypeScript pur, **sans DOM** — `<br>` → `\n`, dépouillement des balises **puis** décodage des entités, `\n{3,}` → `\n\n`, `trim`. Vide → `synopsis: undefined`, jamais `null` ni `''` | Décoder avant de dépouiller supprime un `&lt;i&gt;` que l'auteur voulait afficher littéralement |
| **DEC-47** | `synopsis?` fait partie d'`AnimeEntry` (contractualisé, `TYPES_CONTRACT.md §2`) | Un type réinventé localement diverge du contrat |
| **DEC-52** | 🔴 **Hiatus = une seule source**, computed à 14 j | Deux seuils pour une même règle métier = deux vérités affichées |
| **DEC-86** | `normalizeAnime` produit **toujours** `studios: string[]` | Sans studios peuplés, la dimension studio du scoring est morte |
| **DEC-84** | `POSTER_PLACEHOLDER` source unique · `onHiatus?` retiré du type · `episodeOverride` reseté à chaque upsert | Une copie inline du placeholder diverge ; un `onHiatus` persisté concurrence le calcul dérivé |
| **DEC-90** | Garde null-safety `genres ?? []` | Un cache legacy sans `genres` fait crasher `topGenres` |
| **DEC-10** | Après normalisation, `genres` / `themes` / `studios` sont toujours `string[]` — les branches défensives sont supprimées | Du code mort qui teste l'impossible masque le vrai cas |
| **DEC-81** | Une entrée MyAnimeList `Dropped` n'est **pas** importée | Un import de 300 titres remplit la bibliothèque de séries abandonnées |
| **DEC-24** | `MalImportResult` expose `imported`, pas `entries` | Signature inventée = erreur de compilation ou champ toujours vide |
| **DEC-16** | `parseMalXml` est pur dans `utils/malImport.ts` ; la partie impure vit dans `useMalImport` | Un parseur impur n'est pas testable sans navigateur |
| **DEC-15** | `buildICSContent` est pur dans `utils/ics.ts` ; download + toast dans `useICS` | Idem : génération de texte non testable |

## 4. Réseau AniList

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-139** | 🔴 `Page(page:1, perPage:1){ media(idMal:) }` obligatoire, **jamais** `Media(idMal:)` | `Media()` renvoie une erreur GraphQL sur une entrée introuvable ; le disjoncteur la compte comme un échec et **3 suffisent à couper AniList 60 s pour toute l'app**. La forme `Page` renvoie une liste vide |
| **DEC-126** | Un **429 n'incrémente jamais** le compteur de panne du disjoncteur (une limite de débit n'est pas une panne). `AUD-04` est annulée : AniList n'expose qu'un endpoint, un breaker global y est correct | Une rafale de frappe dans la recherche fermerait le circuit et viderait le calendrier |
| **DEC-141** | 🔴 Aucun throttle manuel dans `syncAnimeUpdates` : `anilistClient` sérialise déjà à 700 ms | Deux régulateurs empilés = deux vérités ; sync de 30 titres à ~50 s au lieu de ~21 s |
| **DEC-143** | Pool `incoming` = **2 appels** (saison courante + suivante, `perPage:50`) | Une 3ᵉ requête n'a plus d'objet et coûte du quota |
| **DEC-137** | Tri `POPULARITY_DESC` pour la requête de saison | — |
| **DEC-114** | Le cache de saison **périmé est servi** si le fetch échoue ; `error.value` est renseigné mais jamais affiché | Évite une page vide, mais l'absence totale de cache donne une liste vide **silencieuse** → dette suivie sous `AUD-05` |
| **DEC-60** | 🔴 `dedupeByMalId` = **source unique** de déduplication, clé `mal_id`, garde la 1ʳᵉ occurrence | Trois chemins de dédup indépendants divergent silencieusement |
| **DEC-127** | La déduplication de recherche s'applique **APRÈS** le tri | Appliquée avant, c'est la version terminée qui survit et l'utilisateur est renvoyé vers Completed pour une série qu'il peut suivre aujourd'hui |
| **DEC-74** | Déduplication appliquée **avant** le `slice` dans `getNextBatch`, jamais après | Un batch de 12 rend 9 cartes sans explication |

## 5. Moteur de recommandations

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-144** | 🔴 `getSeasonNudges` **n'écrit ni ne lit** le store IDB `relations`. Ce store porte la forme MyAnimeList (`[{ relation, entry }]`) lue par `buildRelationMemory` | Y écrire des `AnimeRelation` AniList corromprait le scoring **en silence**, sans erreur ni test rouge |
| **DEC-140** | `AnimeRelation` normalisé ; le filtrage des types de relation vit dans le composable, **jamais** dans le `.vue` | Un filtre dans la vue est invisible aux tests unitaires |
| **DEC-12** | `decayMultiplier = 0.2` dans `buildTasteProfile` | Constante de calibrage — la changer déplace tout le scoring |
| **DEC-13** | Priorité de tri des signaux typée `Record<RecSignalKind, number>`, `score: 0` | Un membre d'union non mappé tombe silencieusement au défaut |
| **DEC-14** | `extractBecauseYouWatched` : paramètre inutilisé préfixé `_profile`, signature publique préservée | `TS6133` ne se silencie pas avec `eslint-disable` |
| **DEC-79** | Réactivité Discover par **dérivation** : `excludedIds = union(store, dismissedRecIds)` | Un canal modal → page laisse des cartes fantômes selon le chemin d'ajout |
| **DEC-83** | Le skip d'une suggestion slot-fill est **session-only** (`ref<Set<number>>` local, jamais persisté) | « Écarter pour l'instant » ≠ « bannir » |
| **DEC-145** | `J10e` (repli des orphelins par titre + année) éclatée en 3 slices, en backlog | `normalizeAniList` rejette en ligne 1 tout média sans `idMal` — or un orphelin est précisément ça. Un mauvais rattachement est **pire** que pas de rattachement : l'utilisateur voit la jaquette d'une autre série |

## 6. UI, feedback & vocabulaire

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-61** | 🔴 **Le composant définit son contrat d'emit ; les consommateurs s'alignent.** Ne jamais renommer un emit pour matcher un listener | `RecCard` a eu Add, clic carte et « pas intéressé » morts sur toutes les surfaces, avec **0 erreur console** |
| **DEC-58** | Corollaire : un désalignement de nom d'event se corrige **côté page**, jamais côté composant | Renommer l'emit casse les N autres consommateurs |
| **DEC-62** | Toute action d'ajout ou de déplacement produit un **toast nommant la destination visible exacte** | Une action sans feedback est indistinguable de « rien ne s'est passé » |
| **DEC-63** | Vocabulaire des toasts = vocabulaire visible : « Coming Soon », « Completed » — jamais « Radar » ni « Vault » | L'utilisateur ne voit ces mots nulle part dans l'UI |
| **DEC-95** | Vocabulaire de recherche figé : « Coming Soon », « Finished airing » | Idem |
| **DEC-73** | Toast « Moved to Completed » au boot pour l'auto-vault | Une série disparaît de la semaine sans explication |
| **DEC-96** | L'état « ✓ Added » est **cliquable** : retire l'anime d'où qu'il soit + toast « Removed » | Sinon l'ajout est irréversible depuis la recherche |
| **DEC-93** | Le bouton ✓ « Mark done » est **masqué** sur Airing / Hiatus (`v-if` présentationnel) ; l'action reste dans la modale | L'action n'a de sens que sur un titre terminé |
| **DEC-77** | « Mark done » et la ligne recency sont gatés sur `isFinished` | Idem |
| **DEC-129** | Ligne « prochain épisode » : affichée seulement si `airsTime` existe et que la série n'est pas terminée. Pas d'`airsTime` → **aucune ligne**, jamais « time unknown ». Se cale sur `airsTime`, jamais sur `day` seul | Un repli textuel affiche une certitude que la donnée ne porte pas |
| **DEC-66** | Le libellé de période a une **source unique** dans `CalendarNavControls` | Deux libellés divergent d'un écran à l'autre |
| **DEC-76** | Snap-to-today : `await nextTick()` + `findIndex(d => d.isToday)` + `scrollIntoView`, garde `todayIndex < 0`, en `onMounted` **et** `onActivated` (KeepAlive). Pas de re-snap sur Prev/Next | Sans `onActivated`, le retour sur la vue via KeepAlive ne resnappe pas |
| **DEC-33** | `ToastNotification` monté **uniquement** dans `AppLayout` | Conséquence connue : aucun toast sur `/login` ni `/welcome` (`AUD-06`) |
| **DEC-42** | `modalContext` : `libraryRec` prioritaire | Deux contextes concurrents ouvrent le mauvais gabarit |
| **DEC-36** | `ChipsStrip` : la chip `all` émet `null` | Une chaîne vide serait traitée comme un filtre |
| **DEC-37** | `WeekAnimeItem` reçoit `info` en prop | Recalculer dans la carte multiplie les calculs par ligne |
| **DEC-38** | `MonthDayCell` reçoit `animes` **déjà filtrés** | Idem |
| **DEC-34** | Lazy-load d'image via `<img style="display:none">`, pas `new Image()` | `new Image()` échappe au cycle de vie Vue |
| **DEC-35** | Dismiss de `SeasonNudgeCard` via `<Transition @after-leave>`, pas `setTimeout` | Un timer désynchronisé de l'animation laisse un trou visuel |
| **DEC-44** | Pas de prefetch de covers : fallback `@error` | Le prefetch doublait les requêtes image pour rien |
| **DEC-43** | `removeAnimeWithUndo` simplifié — l'undo est une dette assumée | — |
| **DEC-82** | Redirect post-login = `/`, pas la route d'origine | Choix assumé : le lien magique expire rarement, ROI faible |
| **DEC-91** | Route `/stats` derrière le guard auth | Les stats sont personnelles |
| **DEC-88** | `useStats` = composable dédié, `StatsPage.vue` = page pure | Séparation stricte des couches |
| **DEC-89** | `topGenres` scoped au **contenu terminé cette année** uniquement | Un year-in-review compte ce qui a été consommé, pas les intentions |
| **DEC-94** | Règle de titre centralisée dans `getAnimeTitle` : anglais primaire + rōmaji secondaire si différent | Deux règles de titre = deux affichages pour la même série |
| **DEC-23** | `useTheme` applique la classe `dark` sur `<html>` | Sur `<body>`, les variables CSS racine ne basculent pas |

## 7. CSS & layout

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-107** | 🔴 `width:100%` + `padding` **sans `box-sizing:border-box`** sur `.secondary-nav-wrapper` faisait déborder le **document** de 30 px. Correctif : `box-sizing:border-box` + `flex:1 1 0; min-width:0` sur `.secondary-tabs button` | Cause racine unique du débordement horizontal **et** du décentrage apparent des modales `position:fixed`. Le symptôme apparaît très loin de sa cause |
| **DEC-111** | 🔴 `.aa-card-grid` = classe partagée, colonnes fixes par breakpoint | Un `minmax(160px,1fr)` sans marge bascule en 1 colonne pour 5 px ; une classe définie en `<style scoped>` ailleurs n'est **jamais** appliquée |
| **DEC-99** | Header au scroll = **sticky CSS pur**, aucune logique JS de scroll-hide | `v-show`/`display:none` en cours de scroll provoque des sauts de hauteur (flicker) |
| **DEC-97** | Les couleurs réutilisent les tokens existants (`var(--airing)`, `var(--upcoming)`) | Une couleur en dur ne suit pas le mode sombre |
| **DEC-67** | Convention de classe active de nav = `.active` | Le markup doit suivre le CSS, pas l'inverse |

## 8. Tests & E2E

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-104** | 🔴 **L'auteur du test ≠ l'auteur du code. Aucune exception**, même pour un test visuel « simple » | Un test auto-écrit valide le code tel qu'il est, pas le comportement attendu — écarté intégralement, sans valeur de preuve |
| **DEC-57** | **R4** : tout correctif UX livre un E2E — geste réel, assertion sur le DOM visible, ROUGE puis VERT **sans modification du test** | Un vert obtenu en modifiant le test ne prouve rien |
| **DEC-56** | 🔴 Bypass d'auth E2E lu en **statique** (`import.meta.env`), branche éliminée du bundle (prouvable `grep -c` = 0) ; `tests/e2e/**` exclu de Vitest | Une lecture runtime laisserait le bypass vivant en production |
| **DEC-65** | **R5** : 1 test ciblé par US pendant l'epic, grand check en fin d'epic, specs cumulatives jamais supprimées | Un test cumulé rouge est une régression à corriger, pas un test à retirer |
| **DEC-109** | La suppression d'une spec **non enregistrée** au registre est autorisée | Un `debug-*.spec.ts` hors registre n'a aucune valeur de preuve |
| **DEC-112** | Pour une propriété de conteneur : `toHaveCount` puis `getComputedStyle`, pas `toBeVisible` | Un `display:grid` sans enfant a une hauteur de 0 px et paraît invisible alors que le CSS est correct |
| **DEC-80** | Un anime `calendar` + terminé s'auto-vault au boot, alors que `calendar` + en cours gate « Mark done » : le scénario est **structurellement impossible**. Les specs testent sur `watchlist` (exclu de l'auto-vault) | Un test écrit sur le cas impossible est rouge sans qu'aucun code ne soit fautif |
| **DEC-149** | 🔴 Le harnais E2E AniList passe par un **helper mutualisé** `tests/e2e/_helpers/anilistMock.ts`, jamais par N `page.route()` dupliqués | Un mock dupliqué dans N fichiers fait payer le prix fort à chaque changement de fournisseur — et casse **au sweep**, pas au merge |
| **DEC-150** | 🔴 Le mock discrimine par **corps de requête**, pas par URL (AniList n'a qu'un endpoint POST). Quatre règles ordonnées : `search` → recherche · `season` → saison · `idMal` **+ `query` contenant `relations`** → relations · `idMal` seul → détail · aucune variable → top finished. Le helper **importe** `resolveSeason` / `resolveNextSeason` depuis `useAniListApi.ts` | Détail et relations partagent `idMal` : sans le jeton `relations`, les deux se confondent. Recopier la règle mois→saison ferait passer une spec au vert le 30 juin et au rouge le 1ᵉʳ juillet |
| **DEC-153** | Les 12 specs vertes-mais-démockées (`AUD-24`) migrent **après** la gate, pas avant | Toucher 12 specs vertes juste avant une Sprint Outcome Gate en fabrique trois rouges |
| **DEC-152** | ⛔ « More like this » n'est **pas** masqué : il est **rebranché** sur `fetchRelationsByMalIdWithMeta` | Masquer ajoute un `v-if` sur un élément qu'une spec clique → rouge au sweep. Le bouton doit redevenir utile, pas disparaître |
| **DEC-122** | Une gate 🔴 peut être satisfaite par un **test unitaire dédié** quand l'E2E est structurellement impossible. Motivé au cas par cas, non généralisable | Cas fondateur `AUD-02` : le SDK Firestore met les écritures en file locale hors-ligne, aucun rejet n'est observable depuis le navigateur |
| **DEC-54** | Filet de sécurité **avant** correctif : CI + smoke test + un test rouge encodant le bug | Corriger sans filet, c'est corriger sans preuve |
| **DEC-08** | Fixtures de test typées via helper `Partial` ou factory. Interdit : `as any`, `as unknown as T` | Une fixture castée ne peut pas détecter une entité incomplète |
| **DEC-06** | `jsdom` déclaré **par fichier** via `// @vitest-environment jsdom` | Un environnement global ralentit toute la suite |
| **DEC-05** | Le conteneur de l'agent tourne nativement en UTC — pas de préfixe `TZ=UTC` | Un préfixe inutile masque un vrai problème de fuseau |

## 9. Gouvernance, process & outillage

| ID | Décision | Pourquoi ça casse si on l'ignore |
|---|---|---|
| **DEC-53** | `AGENTS.md` est la gouvernance permanente de l'agent d'implémentation | C'est le **seul** document qu'il lit |
| **DEC-121** | `AGENTS.md` est en **lecture seule** pour l'agent (R-AGENTS-1) ; mise à jour uniquement par US dédiée avec patch verbatim (R-AGENTS-2). Circulation unique : AI Studio → repo → CMD | 3ᵉ occurrence de `R-SCOPE-1` sur ce fichier |
| **DEC-128** | Les **micro-patchs** (≤ 10 lignes, 1 fichier, entièrement dictables) ne passent pas par l'agent : Claude produit, le PO colle. **Réservé aux changements sans logique métier** (import, déplacement d'appel, renommage) | Dès qu'une décision de comportement est en jeu, la dictée verbatim rend une erreur de spec indétectable — un second lecteur doit pouvoir l'attraper |
| **DEC-87** | 🔴 **Un handoff est une source secondaire faillible ; le code réel tranche.** Quatre « faits » hérités d'un handoff se sont révélés faux | Une inférence recopiée devient un fait au bout de deux sessions |
| **DEC-110** | Le diagnostic du PO pointe un **symptôme**, pas une cause : vérifier avant de spécifier | Une US entière a été écrite pour un défaut qui n'existait pas sous cette forme |
| **DEC-100** | Devant une régression introduite par un effet visuel, **retirer** plutôt qu'empiler un correctif | Un empilement de correctifs enterre la cause |
| **DEC-133** | Le temps de démarrage de référence est **celui de la production** (2,5 s, bundle prêt à 152 ms, 15 requêtes) — pas la mesure du serveur de dev | Un chantier de réduction de bundle décidé sur le chiffre de dev vise un fantôme. **Aucun chantier de bundle n'est ouvert** |
| **DEC-147** | Le worker de relations en fond est **supprimé sans remplacement** | Il attendait 1,1 s par requête sur toute la bibliothèque, forçait `isSyncing` ~1 min et déposait son résultat dans un bloc vide. Un enrichissement de relations en fond est une US produit, pas un sous-produit |
| **DEC-151** | `AUD-05` (signal de fraîcheur visible) exige un **DEC d'arbitrage préalable** sur la source unique du signal `stale`, et passe 🟠 | Deux signaux `stale` concurrents et morts coexistent → violerait DEC-52 |
| **DEC-120** | L'app est en **beta avec des testeurs réels** : tout P0 sur un parcours d'entrée est bloquant | Un défaut d'onboarding touche le premier écran de chaque nouvel utilisateur |
| **DEC-98** | `npm install` fonctionne en direct ; `--legacy-peer-deps` supprimé, à réarmer seulement si un futur `package.json` réintroduit le conflit | Une parade laissée en place masque le retour du conflit |
| **DEC-02** | ESLint = flat config + `@vue/eslint-config-typescript` avec `no-explicit-any` **en erreur** | ⚠️ La configuration existe mais ESLint **n'est jamais exécuté** dans la porte verte (`STATE.md §Trous`) |
| **DEC-03** | `tsconfig.node.json` séparé pour isoler `vite.config.ts` | Sans isolation, la config de build pollue le type-check applicatif |
| **DEC-18** | Upsert du store : garder `if ('state' in input)`. Ne jamais recalculer `state` inconditionnellement en branche merge | Sinon clobber du `state` choisi par l'utilisateur |

| **DEC-154** | 🔴 **Le corpus documentaire est compressé et scindé : `DECISIONS.md` ne porte que les décisions **actives**, en tableau `ID \| Décision \| Pourquoi ça casse` ; les décisions closes, périmées et `⛔ SUPERSEDED` basculent dans `DECISIONS_ARCHIVE.md`, satellite append-only hors ordre de lecture | Un journal chronologique de 717 lignes ne se lit plus : il se cherche. Chaque doublon évince un extrait pertinent de l'index. Les numéros `DEC-xx` ne sont **jamais** supprimés ni renumérotés — tous les renvois restent résolvables, l'archive existe précisément pour ça |
