# DECISIONS.md — Décisions actives

> **Rôle :** les choix encore appliqués aujourd'hui, une ligne chacun. Un numéro n'est jamais supprimé ni réattribué.
> **Pas ici :** décisions closes, périmées ou de migration (→ `DECISIONS_ARCHIVE.md`, hors ordre de lecture) · état courant (→ `STATE.md`) · règles opposables (→ `AGENTS.md`, `PILOTAGE.md`) · pièges répétés (→ `ANTIPATTERNS.md`).

**Dernier numéro attribué : DEC-184.** 
Une décision contredite est marquée `⛔ SUPERSEDED PAR DEC-xxx` et bascule dans l'archive ; les renvois `DEC-xx` des autres documents restent résolvables.

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
| **DEC-120** | L'app est en **beta avec des testeurs réels** : tout P0 sur un parcours d'entrée est bloquant | Un défaut d'onboarding touche le premier écran de chaque nouvel utilisateur |
| **DEC-98** | `npm install` fonctionne en direct ; `--legacy-peer-deps` supprimé, à réarmer seulement si un futur `package.json` réintroduit le conflit | Une parade laissée en place masque le retour du conflit |
| **DEC-02** | ESLint = flat config + `@vue/eslint-config-typescript` avec `no-explicit-any` **en erreur** | ⚠️ La configuration existe mais ESLint **n'est jamais exécuté** dans la porte verte (`STATE.md §Trous`) |
| **DEC-03** | `tsconfig.node.json` séparé pour isoler `vite.config.ts` | Sans isolation, la config de build pollue le type-check applicatif |
| **DEC-18** | Upsert du store : garder `if ('state' in input)`. Ne jamais recalculer `state` inconditionnellement en branche merge | Sinon clobber du `state` choisi par l'utilisateur |

| **DEC-154** | 🔴 **Le corpus documentaire est compressé et scindé : `DECISIONS.md` ne porte que les décisions **actives**, en tableau `ID \| Décision \| Pourquoi ça casse` ; les décisions closes, périmées et `⛔ SUPERSEDED` basculent dans `DECISIONS_ARCHIVE.md`, satellite append-only hors ordre de lecture | Un journal chronologique de 717 lignes ne se lit plus : il se cherche. Chaque doublon évince un extrait pertinent de l'index. Les numéros `DEC-xx` ne sont **jamais** supprimés ni renumérotés — tous les renvois restent résolvables, l'archive existe précisément pour ça |



DEC-155	Un sprint fait 10 slots : 7 US planifiées + 3 flex (bugs sortis en route, retours bêta qualifiés). Clôture à date, jamais à épuisement — le non-fini glisse dans ROADMAP.md. Un retour bêta entre en flex, jamais en interrompant l'US In Progress. Dérogation S41 déclarée : 10 planifiées / 0 flex	Sans clôture à date, un sprint qui absorbe les retours ne finit jamais : plus aucun bump de version, plus aucun Sprint Outcome Gate
DEC-156	La vision multi-sprints vit dans ROADMAP.md, satellite hors ordre de lecture. Rafraîchi à chaque clôture de sprint, PI complet toutes les 4. STATE.md ne porte que le sprint courant et renvoie	Un cap dans STATE.md explose son plafond de 200 lignes ; un cap nulle part oblige à re-débattre les priorités à chaque session
DEC-157	Boucle bêta : retour brut du PO → qualification par Claude (impact utilisateur, effort, risque) → slot flex du sprint courant ou ROADMAP.md. Jamais de correctif « au vol » hors US	Un correctif sans US ni gate est exactement le chemin qui a produit les quatre promesses fausses de l'audit S38
DEC-158	Source unique du signal stale = WithMeta.stale (useAniListApi.ts). staleDataWarning (usePersistence.ts:18,192,305) est supprimé, ainsi que keepStaleData / clearStaleData s'ils n'ont plus d'appelant. Débloque AUD-05. Solde DEC-151	Deux sources de vérité pour une même notion violent DEC-52. Le signal doit naître là où la donnée naît, pas dans une couche de persistance qui ne parle pas au réseau
DEC-159	Depuis This Season, le rejet d'un anime est session-only. Skip partout sur cet écran, y compris dans la modale qui en est issue. dismissRec() + trackNegative() restent réservés à For You	Ne pas vouloir un titre cette saison n'est pas ne jamais vouloir le voir. Mélanger les deux fabrique des recommandations punies à tort, sans que l'utilisateur l'ait demandé ni compris
| **DEC-176** | 🔴 **On entre et on sort de la session par un rechargement complet du navigateur**, jamais par une navigation SPA. Deux points d'application : `router/index.ts` (transition `guestOnly` → `requiresAuth` en état connecté, + connecté visitant `/login`) et `AppHeader.vue` (logout). Dérogation DOM explicite à `R-CODE-4`, du même ordre que le déclencheur de fichier et `window.location.reload` anti-chunk déjà en place | `App.vue` charge la base dans `onMounted`, une seule fois. Sans remontage, la lecture Firestore ne rejoue jamais après connexion, et l'état mémoire survit au logout. C'est la cause racine d'`AUD-30` |
| **DEC-159** | La garde anti-écrasement vit **sur le watcher** (`hasLoadedOnce`), jamais dans `saveToDatabase` | 8 appelants légitimes appellent `saveToDatabase` en direct (`OnboardingPage`, `useMalImport`, `useRecommendations` ×2, `useSync`, `usePersistence` ×3). Une garde dans la fonction les casserait tous et ferait rougir 8 specs existantes |
DEC-160	Dérogation DOM explicite pour la navigation par rechargement complet : `window.location.assign()` est autorisé dans `AppHeader.vue` (logout) et `router/index.ts` (entrée en session). À ajouter à la liste close des exceptions R-CODE-4.	Une SPA ne remonte pas son composant racine à la connexion ni à la déconnexion. Sans rechargement, `loadFromDatabase()` ne rejoue jamais et une sauvegarde différée survit à la purge du logout. Aucune alternative en Vue ne coupe les deux à la fois de façon atomique.
DEC-161	Un document de cache dont `data` est vide ne fait JAMAIS autorité face à Firestore, quel que soit son timestamp (`cacheTimestamp = doc.data.length > 0 ? doc.timestamp : 0`).	Un document vide plus récent gagne la comparaison de fraîcheur et rend la bibliothèque cloud définitivement inatteignable sur l'appareil. C'est la cause n°1 d'AUD-30, et elle ne se répare jamais toute seule.
DEC-162	Avant tout test de fidélité portant sur un garde de routeur, une machine à états ou un ordre d'orchestration, la séquence réelle est rejouée dans un bac à sable exécutable (Node + la vraie librairie), et l'échec AVANT correctif est prouvé.	Un test de fidélité qui vise la mauvaise branche est vert par construction : aucune porte de qualité ne peut le détecter. Cas fondateur `US-PERSIST-P0a` — mergée verte avec 270 tests, zéro effet sur le bug.
DEC-163	La vue Mois existe, mais pas encore. `US-MONTH-FIX` est annulée : 42 cellules × 4-6 entrées sur 390 px n'est pas réparable en CSS (titres tronqués à 3 mots en sombre, invisibles en clair). Remplacée par un « Coming Soon » assumé. `MonthDayCell.vue` et les classes `month-*` de `src/style.css` sont CONSERVÉS pour la refonte, à planifier en S48 ou en sprint dédié. Tranche en avance la question ouverte de ROADMAP §3.	Retirer une fonctionnalité peut être un gain produit. Une grille fonctionnelle-sur-le-papier mais illisible fait douter rétroactivement de tout le reste de l'app ; un « Coming Soon » assumé est plus crédible qu'une demi-fonctionnalité.
DEC-164	Aucune porte accrochée à `router.beforeEach` ne peut lire l'état applicatif. Le garde s'exécute AVANT le montage d'`App.vue`, donc avant `loadFromDatabase()` : le store vaut `[]` à cet instant, toujours. Toute redirection conditionnelle se dérive d'une source lisible sans montage (localStorage, token), jamais du store.	Cas fondateur : la conception initiale d'`US-ONBOARD-PERSIST` dérivait l'onboarding de la présence d'une bibliothèque. Elle était morte-née. Même famille d'échec qu'`US-PERSIST-P0a`. Ça reviendra à chaque US de redirection.
DEC-165	Avant d'écrire un test d'intégration qui traverse le store, les gardes de `addAnime` / `addAnimeSilent` (`stores/anime.ts:39-53`) sont lues. Elles RÉÉCRIVENT le state fourni dans au moins trois cas. Une spec qui assert un `state` sans les avoir lues assert une valeur que le store n'écrira jamais.	Le test rouge de `US-MALIMPORT-FIX` était une erreur de spec, pas de code : il attendait `state:'calendar'` là où le store impose `watchlist` + `awaitingSchedule`. Un test de fidélité faux fait échouer un code correct et coûte un tour d'escalade complet.
DEC-166	Le plafond Firestore est de 500 entrées (`data.size() <= 500` dans `firestore.rules`, create ET update). Une entrée pèse ~1,1 ko mesuré : 500 ≈ 550 ko, 45 % de marge sous la limite plateforme de 1 Mo. Le plafond dur réel est vers 900. Toute modification de `firestore.rules` doit être PUBLIÉE en console — le fichier du dépôt n'est pas ce qui s'applique.	Un plafond à 100 refusait silencieusement les sauvegardes d'une bibliothèque moyenne. Un fichier de règles modifié sans publication ne change strictement rien et donne l'illusion d'un correctif livré.
DEC-167	L'attente sur un 429 AniList est plafonnée à `MAX_RATELIMIT_WAIT_MS = 3000`, et `minIntervalMs` par défaut passe de 700 à 1200 ms. L'attente reste DANS la boucle de retry — seule sa durée change.	L'attente se fait dans une file unique partagée par toute l'application : respecter un `Retry-After: 60` gelait la recherche, le calendrier et les fiches pendant 60 à 120 s. 700 ms représentait 95 % du quota nominal AniList, sans aucune marge.
DEC-168	La priorité de synchronisation classe par INVISIBILITÉ, pas par visibilité : `awaitingSchedule` (2) > `calendar` (1) > reste (0). Un anime en échec est horodaté en antidaté (réessai 15 min), et la sync est plafonnée à 25 par démarrage.	Une entrée `awaitingSchedule` est stockée mais invisible partout ; une entrée `calendar` s'affiche déjà correctement. Sans ce classement, les 25 places d'un démarrage vont aux entrées déjà visibles et un import MAL n'a jamais son tour. Sans horodatage sur échec, un anime rate-limité revient à chaque démarrage et entretient sa propre famine.
DEC-169	Aucune sortie de commande de Gemini n'a valeur de preuve — build, tests ET type-check. Son `node_modules` diverge du nôtre : noms de chunks structurellement différents, `index.html` différent, écart de build reproductible de 71 kB sur 3 livraisons.	R1 le posait déjà pour les tests. On sait maintenant POURQUOI, et que ça s'étend au type-check : un `vue-tsc` sur une autre version de TypeScript peut être vert chez lui et rouge chez nous.
DEC-170	Une US de plomberie réseau (file d'attente, quota, TTL, ordre de synchronisation) peut déroger à l'exigence E2E d'une 🟠, sur déclaration explicite dans l'US et validation du PO. Gate alors exigée : R1 complète + test de fidélité. Ne s'applique JAMAIS à un correctif touchant un élément d'écran.	Un délai réseau ou un ordre de file n'est pas observable par un geste Playwright. Exiger un E2E produirait soit une spec fantoche, soit un test de timing fragile — un faux vert en puissance, pire que l'absence de filet.
- **DEC-171 — `US-SEASON-1TAP` est supprimée, absorbée par `US-CARD-CONVERGE-A`.** Les deux
  décrivaient le même livrable (« This Season : 1 tap au lieu de 2 + modale »). `ROADMAP.md` leur
  allouait deux slots distincts. Un seul geste, un seul slot.

- **DEC-172 — dans `useAddAnime`, le toast dérive de l'état APPLIQUÉ, la synchronisation de l'état
  DEMANDÉ.** Le store peut démoter une entrée (auto-vault sur `Finished Airing`, démotion
  `calendar` sans `day`). Le message doit suivre le store — sinon l'app annonce un onglet où
  l'anime n'est pas. Mais la synchronisation doit rester sur l'état demandé : **c'est elle qui va
  chercher la date manquante et repromeut l'entrée.** La brancher sur l'état appliqué la
  désactiverait exactement dans le cas où elle est nécessaire.
  ⛔ Corollaire opposable : le message s'aligne sur le store, **jamais l'inverse**. Ne pas
  « corriger » les gardes de `stores/anime.ts` pour faire respecter l'état demandé.

- **DEC-173 — `markOnboarded()` est appelé AVANT `saveToDatabase()`, et la redirection est
  garantie.** Un échec de sauvegarde ne doit ni bloquer la fin de l'inscription, ni la faire
  rejouer. L'utilisateur arrive sur son calendrier dans tous les cas ; un message unique lui dit
  que la copie cloud a échoué. ⛔ Ne jamais déplacer `markOnboarded()` après la sauvegarde
  « pour n'onboarder qu'en cas de succès » : le tunnel complet serait rejoué à chaque échec réseau.

- **DEC-174 — le handoff se déclenche à 80 % de capacité, pas 90 %.** Un handoff coûte 8-10 % à
  produire. Déclencher à 90 % garantit un document tronqué — l'inverse de ce que la règle protège.
  Supersede le seuil de 90 % des instructions de projet.

- **DEC-175 — `AUD-37` est délégué aux bêta-testeurs, pas soldé.** Décision PO explicite de ne pas
  tester l'import MAL sur un vrai fichier. Contrepartie obligatoire : **la note aux testeurs doit
  demander « combien de titres avais-tu, combien en retrouves-tu ? »** — sans quoi un import cassé
  produirait un retour inexploitable (« ça marche pas ») au lieu d'un signal.
| **DEC-177** | `.app-layout .app-header` porte `position: relative; z-index: 200`. Le header a désormais un contexte d'empilement **explicite et positionné**, entre `.secondary-nav-sticky` (100) et `.modal` (10000) | Un `z-index` sur un élément en `position: static` est purement ignoré. Trois valeurs successives ont été posées sans effet parce que la règle qui gagnait l'arbitrage de spécificité imposait `static`. La valeur seule ne suffit jamais : il faut vérifier que l'élément est positionné |
| **DEC-178** | La règle « cet anime est en cours de diffusion » a **une seule implémentation applicable** : `useAddAnime.resolveTargetState`, qui reconnaît `Currently Airing` **et** `Continuing`. Le doublon résiduel de `utils/onboardingFilter.ts` est connu, hors périmètre S42, à converger en S44 | Trois exemplaires divergents coexistaient. `useAddAnime` ignorait `Continuing` : migrer la recherche dessus sans corriger aurait envoyé tout anime en cours dans *Plan to Watch*. Un `findstr` du PO l'a évité avant l'envoi de l'US |
| **DEC-179** | Un compte retrouvé côté Firestore alors que le flag local est absent **n'interrompt pas** l'onboarding : il affiche une **bande de rattrapage** persistante sur toutes les étapes, avec sortie directe vers le calendrier. Le boot reste non bloquant | Faire attendre le réseau avant d'afficher l'onboarding ralentirait le démarrage de tout le monde pour un cas minoritaire. La bande ne coûte rien à ceux qui ne sont pas concernés et dit la vérité à ceux qui le sont |
| **DEC-180** | `RecCard` accepte une prop `showSkip` (défaut `true`). *This Season* la passe à `false` : le bouton « Skip » n'apparaît pas hors *For You*. `DEC-159` devient **sans objet** sur cet écran | Sur *For You*, « Skip » signifie « ne me le remontre plus ». Sur *This Season*, `DEC-159` en faisait un rejet session-only. Un même bouton avec deux portées invisibles est un piège : masquer vaut mieux qu'expliquer |
| **DEC-182** | **S42 est clos en v0.36.0**, 10/10 slots, sur réponse « gain de fiabilité visible » à la Sprint Outcome Gate. `US-HEADER-ICONS` non prise glisse en S43 | `DEC-155` : clôture à date et non à épuisement. Les 10 slots sont consommés ; une version taguée avant l'envoi aux testeurs vaut mieux qu'un sprint qui déborde |
DEC-183	Une spec qui ne teste ni contenu ni données (layout, navigation, débordement, état vide) installe `installAniListMock(page, {})` — mock total à vide. Aucune graine, aucune URL	Mesuré en SE-071 : `grid-two-columns` et `no-horizontal-overflow` passent au vert avec une grille vide. Elles n'ont jamais eu besoin de données — elles routaient Jikan par mimétisme. Un mock à vide est plus fort qu'un mock peuplé : il prouve que le composant ne dépend pas du réseau
DEC-184	Des specs INDÉPENDANTES migrent en lot : N fichiers, un geste identique, une seule gate. L'interdit porte sur deux changements dans un MÊME fichier, jamais sur N fichiers disjoints	Playwright nomme le fichier fautif : sur des fichiers disjoints la cause reste identifiable, donc le lot ne coûte rien en diagnostic. Appliqué 3 fois en SE-071 (3, 3 puis 2 fichiers), zéro rouge. Le coût résiduel est la gate : des specs de batchs différents imposent autant de commandes que de batchs
