# AUDIT.md — Inventaire des constats de code

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
>
> **🔻 CHARGEMENT À LA DEMANDE — ce document n'est PAS lu au démarrage de session.**
> Il ne figure pas dans l'ordre de lecture de `CLAUDE.md`, volontairement. On l'ouvre dans
> deux cas seulement : **planifier un sprint**, ou **convertir un constat en US**.
>
> **Rôle :** le *contenu* des constats — ce que c'est, où c'est (`fichier:ligne`), ce que ça
> coûte à l'utilisateur. **Jamais la planification.**
> **Lien unique : `STATE.md §Backlog` cite les identifiants `AUD-xx` et décide de leur sprint.
> Ce fichier ne dit jamais quand un constat sera traité.** Aucun autre document ne référence
> celui-ci.
>
> **Append-only, par campagne.** Une campagne d'audit close n'est jamais réécrite. Un constat
> converti en US n'est pas supprimé : il est marqué `→ livré vXX` et reste comme trace.

---

# Campagne S38 — dual audit

**Commit audité :** `c7cc60f` (2026-08-04). **Réconcilié en :** SE-050. **Cadre :** DEC-117.

## Fiabilité de la campagne — à lire avant d'utiliser les constats

**Production brute :** Claude Code, 39 findings avec `fichier:ligne`, repo cloné.
Gemini AI Studio, 6 findings, sans hash de commit déclaré, fichiers collés.
**Après réconciliation : 30 constats uniques valides, dont 7 P0.**

**Gemini n'a produit aucun finding unique.** Ses 6 constats sont un sous-ensemble strict de
ceux de Claude Code, référence de ligne comprise. Une seule référence orpheline
(`usePersistence.ts:200`), non corroborée → classée « à vérifier ».

**La démonstration la plus nette — `useMalImport.ts:88`.** Les deux ont lu la même ligne.
Gemini a vu un cast → « P2, dette interne, risque théorique ». Claude Code a suivi le flux
jusqu'à `malImport.ts:56` et établi que le champ lu (`my_status`) n'est **jamais produit**
→ **P0, 100 % des entrées d'un import MAL atterrissent en `radar`.**
Preuve par grep contre preuve par flux.

**Seul apport propre de Gemini :** une meilleure calibration de gravité sur l'onboarding vide
(P0 chez lui, P1 chez Claude Code). Sur un lancement public c'est le premier écran de chaque
nouvel utilisateur — **Gemini a raison**.

**Limite assumée de la campagne :** aucune observation runtime. L'audit n'a exécuté ni
`vue-tsc`, ni `vitest`, ni `playwright`, ni le build (`node_modules` absent du conteneur).
Tous les constats sont des lectures de code.

**Auto-critique du prompt S38 :** il affirmait l'état de Jikan en préambule **sans remesure**.
`ANTIPATTERNS §7` interdit exactement ça. Le prompt d'audit a commis l'antipattern dans sa
propre prémisse, et les deux auditeurs l'ont accepté.

---

## Les constats

### 🔴 P0

| ID | Constat | Localisation | Impact utilisateur |
|---|---|---|---|
| **AUD-01** | Une entrée `state:'calendar'` peut être créée sans `day`. **13 producteurs sur 14** le font. `normalizeAnime` ne produit jamais `day` ni `airsTime` ; `syncAnimeUpdates` est le seul à le poser, et son filtre exclut le cas dominant | `useSync.ts:68-79` (filtre) · `:121-130` (seul producteur) · `onboardingFilter.ts:10` | Un anime ajouté n'apparaît **jamais** dans la semaine. Perte définitive, pas un délai : ne se répare ni au reload, ni avec une API saine |
| **AUD-02** | `saveSchedule` rattrape son propre throw et retourne normalement. Le `try/catch` ajouté dans `saveToDatabase` est donc **inopérant** | `useFirestore.ts` / `usePersistence.ts` | L'app affiche « Saved » pendant une panne Firestore. **La donnée est perdue et l'utilisateur croit l'inverse** |
| **AUD-03** | Import MAL : `my_status` est lu mais **jamais produit** → 100 % en `radar`. `episodeOverride` perdu au passage | `useMalImport.ts:88` → `malImport.ts:56` | Un import de 300 titres met tout dans « Coming Soon », avec un toast « Imported 300 animes ». Progression épisode remise à zéro |
| **AUD-04** | Coupe-circuit à portée **globale** : `lowPriorityFailures` / `circuitOpenTimestamp` sont des variables de module. 3 échecs en priorité `low` coupent **tout** `low` pendant 5 min, y compris `/seasons/now` — le seul endpoint vivant | `utils/helpers.ts` | Une panne partielle devient totale : le calendrier se vide parce qu'un autre endpoint est mort |

> ⚠️ **AUD-01 ne se corrige pas dans `normalizeAnime` (DEC-118).** J-04 réécrit cette fonction
> sur la forme AniList, J-09 supprime le parsing JST. Toute correction du mapping serait jetée
> dans deux sprints. **La réponse est une garde, pas un mapping** — donc indépendante de la
> source de données, donc migration-proof.

> ⚠️ **AUD-04 est un bloquant de la migration, pas de la dette.** Le plan AniList prévoit de
> « réutiliser le backoff existant » : on porterait le défaut dans le client AniList. Sous
> 30 req/min en mode dégradé, une rafale de frappe dans la recherche fermerait le circuit et
> couperait le calendrier — le mécanisme exact qui a coûté l'endpoint vivant chez Jikan.

### 🟠 Standard

| ID | Constat | Localisation | Impact utilisateur |
|---|---|---|---|
| **AUD-05** | `fetchCurrentSeason` retourne `[]` sans rejeter → **tous les `catch` appelants sont morts**. `useSync` n'a aucune ref d'erreur | 5 écrans | Saison affiche « vous avez déjà tout ajouté » pendant une panne · Onboarding : grille vide muette · Library : page blanche · Discover : aucun état d'erreur |
| **AUD-06** | `ToastNotification` monté uniquement dans `AppLayout` | — | Aucun toast sur `/login` ni `/welcome` : **les erreurs de l'onboarding ne s'affichent jamais** |
| **AUD-07** | Flag d'onboarding en `localStorage` seul, effacé au logout | — | Onboarding rejoué **intégralement** à chaque connexion et sur chaque appareil, y compris avec 40 shows suivis |
| **AUD-08** | CI sans Playwright ni ESLint | `.github/workflows/ci.yml` | Aucun visible — dette. Les 39 specs ne tournent jamais avant merge, et les 17 pièges de faux-vert d'`ANTIPATTERNS §6` ne sont appliqués par aucun automate |
| **AUD-09** | Specs E2E qui n'assertent rien de significatif : toast seul ×2 · `localStorage` au lieu du DOM · fixture `episodes:null` = le seul cas qui passe · test d'échec Firestore sur branche inatteignable | `tests/e2e/**` | Aucun visible — dette. **Ce sont les filets des parcours cassés ci-dessus** |
| **AUD-10** | `show:false` masque une entrée `calendar` valide (3 cas arithmétiques), sans message | — | Un anime présent, jour renseigné, **disparaît** de la semaine et du mois |
| **AUD-11** | `EmptyState` reçoit `description` et `icon` non déclarées dans `defineProps` → fallthrough en attributs HTML, **zéro erreur console** | `EmptyState.vue` + 3 appelants | Coming Up, Plan to Watch et Discover n'affichent qu'un titre, sans la ligne d'explication |

> **AUD-08 est un prérequis de la phase 1 AniList.** Réécrire le point de contrat unique
> (`normalizeAnime`, « risque élevé » selon le benchmark) et versionner les caches (« un bug
> ici efface des comptes ») **sans aucun filet automatisé** n'est pas défendable.
> C'est la seule US de dette pure à imposer avant J-04.

> **AUD-05 est l'US « mode dégradé » du benchmark**, jusqu'ici chiffrée « ~1 US » sans contenu.
> Elle a maintenant son périmètre exact. **À fusionner avec `US-CACHE-STALE-WARNING`.**

### 🟢 Dette

| ID | Constat | Localisation | Impact utilisateur |
|---|---|---|---|
| **AUD-13** | Couches violées : `localStorage` piloté depuis `AppHeader.vue` · SDK Firebase appelé dans `LoginPage.vue` · `useSync` mute le store hors action Pinia | 3 fichiers | Aucun visible |
| **AUD-14** | Typage : `any` explicite · 10 `as any` en tests · **la factory `makeAnime` est elle-même un cast** (`over as AnimeEntry`) · cast sur merge partiel | `helpers.ts:32` + tests | Aucun visible. Effet de bord réel : **les tests ne peuvent pas détecter une entité incomplète** — soit la classe de bug d'AUD-01 |
| **AUD-15** | `getCardStatus` : `status === null` retombe sur « Finished » | `episodeInfo.ts` | Un anime au statut inconnu s'affiche « Finished », pastille grise |
| **AUD-16** | Navigation relations : `{mal_id, id, title} as AnimeEntry` | — | Modale sans jaquette ni score |
| **AUD-17** | Stubs vides `_syncAnimeUpdates` / `_startBackgroundRelationFetch` encore invoqués | — | Aucun visible — risque de double orchestration |
| **AUD-18** | `useRecommendations.ts` (549 lignes) sans aucun test unitaire. Idem `useMalImport`, `useFirestore`, `stores/ui`, `useICS`, `useToast`, `useTheme`, `utils/idb` | 8 fichiers | Aucun visible. **Les 3 P0 non liés à `day` vivent tous dans des fichiers sans test** |

---

## ⏸️ À vérifier avant conversion — ne pas rédiger d'US en l'état

| ID | Doute | Commande de levée (Windows / cmd) |
|---|---|---|
| **AUD-12** | `localStorage.setItem` hors du `try` dans `saveToDatabase` → un quota dépassé bloquerait la fin de l'onboarding sans message | `findstr /n "^" src\composables\usePersistence.ts \| more +118` |
| **AUD-19** | Famine de la file `low` par la file `high` (recherche) | `findstr /n "^" src\utils\helpers.ts \| more +50` |
| **AUD-20** | Ligne `usePersistence.ts:200` citée par Gemini, non corroborée par Claude Code | `findstr /n "^" src\composables\usePersistence.ts \| more +194` |

---

## 🚫 Findings écartés — ne pas les remettre au backlog

| Finding rejeté | Raison du rejet |
|---|---|
| « Spec E2E écrite mais non enregistrée dans un batch » | **0 orpheline vis-à-vis de `package.json`**, vérifié programmatiquement dans les deux sens. ⚠️ *La divergence avec `AGENTS.md §7` est un autre sujet, suivi dans `STATE.md §Trous restants`* |
| « `grid-two-columns` mesure du CSS sur une grille vide » | Le test **applique la remédiation codifiée** dans `ANTIPATTERNS §6` (`toHaveCount` puis `getComputedStyle`). Conforme, pas de la dette |
| « `getCardStatus` ne mappe pas `'Continuing'` » | **Faux.** `episodeInfo.ts:106` lowercase le statut, `:110` teste `'continuing'`. Faux négatif de casse `findstr` |
| « `v-html` inconditionnel » | Le code le conditionne bien à `v-if="isHtml"`, conforme à `AGENTS §4` |

---

## 🧾 Ce que l'audit dit de la documentation elle-même

Croisement des 39 findings contre `ANTIPATTERNS.md` régénéré en SE-049.

### La doc connaissait déjà, et le code viole quand même

| Antipattern déjà codifié | État réel du code |
|---|---|
| §8 — *« créer une entrée `calendar` sans se demander qui posera son `day` »* | Écrit dans la doc, **violé par 13 producteurs sur 14** |
| §4 — *« `finishWithSeed` annonce N shows sans garantir la visibilité »* | Décrit verbatim, jamais corrigé |
| §3 — *« laisser remonter le throw de `handleFirestoreError` »* | Décrit verbatim, jamais corrigé |
| §3 — *« renseigner un état d'erreur sans jamais l'afficher »* | Décrit verbatim, 5 écrans concernés |
| §6 — *« asserter l'état d'un store au lieu du DOM »* | Décrit verbatim, 2 specs actives |

> **`ANTIPATTERNS.md` n'est pas une mémoire des erreurs corrigées, c'est un inventaire de bugs
> en production.** Cinq des sept P0 y sont déjà écrits, **au passé**, comme des leçons apprises.

### Deux remédiations codifiées sont fictives

- **§3** dit que `saveToDatabase` appelait `saveSchedule` sans garde. Le `try/catch` a bien été
  ajouté. **Il est inopérant** (AUD-02). La leçon est classée close, le bug est intact sous une
  forme plus difficile à voir.
- **§2** dit *« fixtures via `as unknown as T` → factory `makeAnime(Partial<AnimeEntry>)` »*.
  La factory existe. Elle est écrite `(over) => over as AnimeEntry`.
  **La remédiation est le pattern qu'elle remplace** (AUD-14).

---

## 🔍 Zones jamais auditées

Aucun des deux auditeurs ne les a ouvertes, et aucun document du corpus ne les couvre.

| Zone | Enjeu |
|---|---|
| **`firestore.rules`** | 🔴 **Le risque le plus sérieux, et le seul que personne n'a jamais regardé.** Lancement public en approche, isolation de `schedules/{uid}` jamais vérifiée |
| **`utils/recEngine.ts`** | Scoring, `assignBadge`, `buildNextBatch` — le moteur de Discover et Library. Aucun audit, aucun test unitaire |
| **~20 composants `.vue`** | `AnimeCard`, `RecCard`, `WeekAnimeItem`, `ModalVersionTop`, `MonthDayCell`, `SyncIndicator`… toute la couche de rendu des cartes |
| **Accessibilité** | ARIA, ordre de tabulation, piège de focus dans les modales `Teleport`, contraste |

---

## 🎨 Lecture produit de la campagne

Le produit n'a pas un problème de bugs, il a **un problème de véracité**. Quatre confirmations
sur cinq sont fausses :

| Ce que l'app affirme | Ce qui se passe | Constat |
|---|---|---|
| « 3 shows added » | Calendrier vide | AUD-01 |
| « Saved » | Panne Firestore, donnée perdue | AUD-02 |
| « Vous avez déjà tout ajouté » | 504 en cours | AUD-05 |
| « Imported 300 animes » | 300 titres au mauvais endroit | AUD-03 |

La migration AniList répare la **capacité** — la recherche remarche, l'heure devient exacte à
la minute, une classe entière de bugs de fuseau disparaît. **Elle ne répare aucune de ces
quatre promesses fausses**, qui vivent dans du code qu'elle ne touche pas.

> Un bug qui échoue visiblement coûte un réessai. **Un bug qui confirme un succès inexistant
> coûte la confiance — et la confiance ne se réessaie pas.**

---

## 📌 Campagnes antérieures

- **s16 — dual audit.** Leçon conservée dans `ANTIPATTERNS §🎓-2`. Findings non réimportés ici
  (antérieurs à la création de ce document, corpus perdu).

## SE-061 / SE-062 — constats sans US

- **`usePersistence.ts:14-15,287` — stub mort.** `_startBackgroundRelationFetch` (`TODO US-016`
  jamais fait) n'est appelé que par lui-même. Grep complet des appelants : la vraie sync est
  câblée en 4 points vivants — `App.vue:52`, `AnimeModal.vue:134` et `:145`,
  `DiscoverExplorePage.vue:225`. **Code mort confirmé, pas un chemin concurrent.** N'a jamais
  bloqué `J10d`. Aucune US ouverte.
- **Bloc mort `if (updatedAny) { }` dans `startBackgroundRelationFetch`.** Deux commentaires,
  zéro instruction. Le worker interrogeait deux endpoints à 504 avec 1,1 s d'attente par
  requête et n'aurait rien fait même en cas de succès. **Tombé avec `J11b-1` (`2a24b3c`).**
- **`useRecommendations.ts` était sans aucun test** jusqu'en SE-061 : 550 lignes, moteur de
  Discover et de « Because you watched », zéro filet. Refermé par
  `useRecommendations.spec.ts` (12 tests) et `useRecommendations.nudges.spec.ts` (12 tests).
  Vérification faite au passage : aucun autre composable du projet n'était dans ce cas.
- **Deux signaux `stale` concurrents, tous deux morts.** `useAniListApi.ts:23,338` expose
  `stale: boolean` dans le contrat `WithMeta` ; `usePersistence.ts:18,192,305` expose
  `staleDataWarning` en readonly. **Zéro consommateur dans un `.vue`** (grep récursif
  `src\*.ts src\*.vue`, SE-062). Deux sources de vérité pour une même notion → viole DEC-52.
  **`AUD-05` requiert un DEC d'arbitrage avant rédaction**, et passe de 🟢 à 🟠 (élément
  d'écran ajouté → R4 applicable).
- **`more-like-this-modal` : feature sans backend.** Le modal appelait
  `/anime/{id}/recommendations`, mort en 504 puis sans code appelant depuis `J11b`. Un clic
  utilisateur mène à du vide. **Ce n'est plus une dette de test mais une régression
  fonctionnelle silencieuse** → P1 backlog S41.
- **ESLint n'est jamais exécuté.** La règle « zéro `any` » (R-CODE-1) n'est vérifiée par
  aucun outil de la porte verte : `vue-tsc` ne la teste pas.
## SE-063 — deux constats ouverts

- **`AUD-24` — 12 specs E2E vertes qui tapent le réseau réel.** Découvert au grep de `J12-0` :
  23 specs portaient des routes mortes sur des chemins REST Jikan, pas 11. Les 11 rouges sont
  soldées par `J12`. Les 12 autres sont **vertes pour la mauvaise raison** — elles assertent de
  la nav ou du layout que le contenu réel satisfait par accident, et frappent `graphql.anilist.co`
  en vrai à chaque sweep. Un 429 ou un déclenchement du disjoncteur les vire rouges sans qu'aucun
  code n'ait bougé. Liste : `calendar-subnav-layout` · `foryou-dedup` · `grid-two-columns` ·
  `login-styled` · `modal-status-gating` · `month-layout` · `nav-active-state` ·
  `no-horizontal-overflow` · `onair-subnav` · `onboarding-genres` · `week-no-duplicate-period`.
  *(`week-empty-day-cta` route `**/*` et reste sain.)*
  **Impact utilisateur : aucun. Impact projet : le sweep n'est pas déterministe.**
  → Migration mécanique vers `installAniListMock` en S41 (DEC-153).

- **`AUD-25` — asymétrie d'action entre This Season et For You.** Signalé par le PO sur captures
  SE-063, **absent de toute la Knowledge** (vérifié : `EPICS`, `BENCHMARK` piles A et B, `AUDIT`,
  `STATE`, `DECISIONS`).

  | Écran | Composant | Coût d'un ajout |
  |---|---|---|
  | For You | `RecCard` | **1 tap** (Skip / Add en surface) |
  | Library › Upcoming | `RecCard` | **1 tap** |
  | This Season | carte distincte | **2 taps + modale** |

  Ce n'est pas une incohérence de style mais **d'action**. Elle contredit une intention produit
  écrite (`BENCHMARK §8` : *« This Season et Coming Soon restent des sources d'ajout, pas des
  destinations »*) — une source d'ajout sans bouton d'ajout est contradictoire.
  🔴 **Ne pas rédiger d'US en l'état.** Lire le composant d'abord : `DEC-110` est né exactement
  de ce raccourci (*« le diagnostic du PO pointe un symptôme, pas une cause »*). L'absence de
  trace n'est pas une preuve d'accident — le choix peut être délibéré.
  → À instruire au triage benchmark, avec `B-07`.

---

# Réconciliation documentaire (chantier de cleaning)

> **Ajout en append, conforme à la règle du fichier :** aucune campagne antérieure n'est réécrite, aucun constat n'est supprimé. Cette section marque l'état de constats déjà présents plus haut, à la date du chantier de compression de la Knowledge.

## Constats soldés

| ID | État | Preuve |
|---|---|---|
| **AUD-04** | ⛔ **ANNULÉ, pas reporté** (DEC-126). Le constat visait la contamination du coupe-circuit entre endpoints d'un même hôte. La source actuelle n'expose **qu'un seul endpoint** : un breaker global y est le comportement correct. Leçon transposée : un `429` n'incrémente jamais le compteur de panne | DEC-126 |
| **AUD-17** | ✅ **Soldé.** Le stub `_startBackgroundRelationFetch` et le worker qu'il doublait sont supprimés du dépôt | DEC-147, slice `J11b-1` |
| **AUD-18** | ✅ **Partiellement soldé.** `useRecommendations` est doté de ses premières specs unitaires. Vérification faite au passage : aucun autre composable n'était dans le cas d'un fichier de cette taille sans aucun test | `useRecommendations.spec.ts` + `.nudges.spec.ts` |

## Constats requalifiés

| ID | Requalification |
|---|---|
| **AUD-01** | La note « ne pas corriger dans `normalizeAnime` » (DEC-118) est **caduque** : DEC-124 pose désormais `day` + `airsTime` par cascade, et DEC-131 marque `awaitingSchedule` en l'absence de date prouvée. Le constat d'origine reste lisible pour la traçabilité, **sa consigne de non-correction ne s'applique plus** |
| **AUD-05** | Toujours ouvert, mais **passe 🟢 → 🟠** et exige un **DEC d'arbitrage préalable** sur la source unique du signal `stale` — deux signaux concurrents et morts coexistent, ce qui violerait DEC-52 (DEC-151) |
| **AUD-08** | Le volet Playwright reste absent de la CI. **ESLint n'est toujours jamais exécuté** : la règle « zéro `any` » n'est vérifiée par aucun outil de la porte verte |

## Constats non réévalués

`AUD-02` · `AUD-03` · `AUD-06` · `AUD-07` · `AUD-09` · `AUD-10` · `AUD-11` · `AUD-13` · `AUD-14` · `AUD-15` · `AUD-16` — aucune lecture de code n'a été faite pendant le chantier documentaire. **Leur état est inconnu, pas « toujours vrai ».** Toute conversion en US démarre par un grep du fichier concerné.

Les doutes `AUD-12` / `AUD-19` / `AUD-20` restent à lever par les commandes indiquées plus haut.



# Campagne SE-064 — session P0 « zéro zone d'ombre »

**Méthode :** 22 constats tranchés par grep sur le dépôt local, plus deux vérifications en
console Firebase et une capture DevTools. **Aucune inférence** : tout verdict ci-dessous
s'appuie sur une sortie de commande ou une capture, conformément à `AP-PROCESS-2`.

## Constats soldés — ne pas rouvrir

| ID | Verdict | Preuve |
|---|---|---|
| **AUD-02** | ✅ **Soldé dans sa forme d'origine.** `useFirestore.ts:89-91` relance bien (`throw error.value`). Le `try/catch` inopérant est réparé. *Résidu isolé sous `AUD-29`* | `useFirestore.ts:73-94` |
| **AUD-09** | ✅ **Requalifié, quasi soldé.** Sur 60 usages `localStorage` en E2E, 59 sont du seed légitime. Une seule assertion sur le store subsiste | `onboarding-seed.spec.ts:56` |
| **AUD-11** | ✅ **Soldé.** `EmptyStateProps` = `title` + `subtitle`, tous deux rendus. Aucune prop orpheline — `vue-tsc` strict serait rouge sinon | `EmptyState.vue:1-15` |
| **AUD-14** | ✅ **Soldé.** `as AnimeEntry` : **zéro hit** dans `tests\**` et `src\**\*.ts`. La factory-cast est morte, probablement avec la purge `helpers.ts` (`J11b-3`) | grep récursif |
| **AUD-19** | ✅ **Soldé.** `helpers.ts` ne contient plus que `escapeHTML`, `getWeekNumber`, `dedupeByMalId`. Aucune file, aucun disjoncteur : la famine `low`/`high` n'a plus de code support | `helpers.ts:1-40` |
| **AUD-28** | ✅ **Annulé.** Les 0 lectures de la vue d'ensemble étaient un artefact d'agrégation : la base réelle affiche **62 lectures** et 14 lectures temps réel sur 7 jours | console → Utilisation |
| **AUD-26** | ⛔ **Annulé — faux positif que je corrige.** Les deux clés de seed E2E (`animeCalendar` / `aanime_calendar`) ne divergent pas : `usePersistence.ts:139-150` porte une table de migration (US-133). Fragilité réelle mais mineure — le jour où la migration tombe, 8 specs rougissent d'un coup. Note pour `US-DEMOCK`, pas une US | `usePersistence.ts:139-150` |
| **Atterrissage onboarding** | ✅ **Sain, prouvé.** `buildSeedEntry` fait un **spread** (`return { ...anime, id, state }`) : `day` et `airsTime` produits par `normalizeAniList` survivent. Le pattern `AUD-01` n'est pas réintroduit. Le trou honnête ouvert en SE-063.b est refermé | `onboardingFilter.ts:13` |

## Constats confirmés vivants — convertis en US

| ID | Constat prouvé | Impact utilisateur | Destination |
|---|---|---|---|
| **AUD-03** | 🔴 **Vivant, et pas là où on croyait.** `malImport.ts:44-58` mappe **correctement** les statuts MAL et préserve `episodeOverride`. Mais `useMalImport.ts:86-97` (`mapMalStatusToState`) lit `my_status` sur un `MalImportEntry` qui **ne porte pas ce champ** : `s` vaut toujours `undefined` → `default: return 'radar'`. Le parser fait le bon travail, ce helper le défait. Le cast `as unknown as Record<string, unknown>` a désarmé TypeScript — `AUD-14` sous une autre forme. *Bonus : « Dropped » est silencieusement ignoré (`skipped++`)* | Un import de 300 titres atterrit intégralement en Coming Soon, avec un toast de succès | `US-MALIMPORT-FIX` — S41 |
| **AUD-06** | `ToastNotification` monté **uniquement** dans `AppLayout.vue:21` | Sur `/welcome` et `/login`, tout échec est muet | `US-ONBOARD-TOAST` — S42 |
| **AUD-07** | `useOnboarding.ts:1-15` — flag `aanime_onboarded` en **localStorage seul**, zéro Firestore | Onboarding rejoué en entier sur chaque appareil et après tout vidage de cache | `US-ONBOARD-PERSIST` — S41 |
| **AUD-10** | 🟡 **Périmètre réduit de 4 cas à 1.** Trois des quatre `show: false` d'`episodeInfo.ts` portent `isFinished: true` (légitime). Seule la l.33 masque une entrée non finie | Un anime présent, `day` renseigné, disparaît de la semaine | `US-SHOW-FALSE` — S44 |
| **AUD-13** | 🟢 **1 seule occurrence restante** : `AppHeader.vue:59-61` (purge de clés au logout) | Aucun visible — mais c'est le maillon 1 d'`AUD-30` | Absorbé par `US-PERSIST-P0` |
| **AUD-15** | `episodeInfo.ts:111` — fallback final `word: 'Finished'`, atteint sur statut nul ou inconnu | Un anime au statut inconnu s'affiche « Finished », pastille grise | `US-STATUS-UNKNOWN` — S44 |
| **AUD-16** | `AnimeModal.vue:179` — `{mal_id, id, title} as AnimeEntry`. *Les trois autres `as AnimeEntry` en `.vue` sont bénins (spreads complets, lecture de `.name`)* | Cliquer une relation ouvre une modale sans jaquette, sans score, sans synopsis | Fusionné dans `US-MORELIKETHIS-FIX` — S42 |

## Constats nouveaux — SE-064

| ID | Constat | Impact utilisateur | Destination |
|---|---|---|---|
| **AUD-27** | 🔴 Les règles Firestore déployées plafonnent le document à **100 entrées** (`data.data.size() <= 100`, l.18 et l.28) et imposent un **timestamp monotone** (`>= resource.data.timestamp`, l.26) | Au 101ᵉ anime, plus aucune sauvegarde cloud n'est acceptée — un import MAL de 300 titres est rejeté en bloc. Deux appareils aux horloges décalées : celui en retard voit toutes ses écritures refusées, définitivement | `US-FIRESTORE-LIMITS` — S41 |
| **AUD-29** | 🔴 `useFirestore.ts:81` — `if (!auth.currentUser) return;` retourne **sans sauvegarder et sans signaler**. `saveToDatabase` affiche alors « Saved » | L'app confirme une sauvegarde qui n'a pas eu lieu | `US-FIRESTORE-LIMITS` — S41 |
| **AUD-30** | 🔴🔴 **Aucune persistance. Deux causes cumulées.** **(1) Prouvée** — `AppHeader.vue:59-61` vide l'intégralité du `localStorage` au logout : `aanime_onboarded` **et** `aanime_calendar` partent ensemble. **(2) Très probable, non prouvée** — le watcher profond `usePersistence.ts:105-115` déclenche `debouncedSave()` sur un store encore vide au boot, avant que `loadFromDatabase()` ait résolu ; `saveToDatabase` écrit alors `data: []` avec un timestamp frais, et la règle serveur l'**autorise** (document vide plus récent). D'où **0 refus** côté moteur de règles : le cloud est effacé par l'app elle-même. `clearStaleData():297-302` fait la même chose délibérément | L'utilisateur ajoute des animes, se déconnecte ou change d'appareil, et retrouve un onboarding vierge. **Bloquant de lancement bêta** | `US-PERSIST-P0` — S41, US n°1 |
| **AUD-31** | 🟢 La collection `schedules` mélange **8 documents fossiles** dont l'ID est un pseudo tapé à la main (`Adn`, `Ae`, `Ael`, `ade`, `adn`, `adnanne`, `aer`, `aze`) et des documents dont l'ID est un vrai UID Auth. Les fossiles portent la forme vanilla : `name:` au lieu de `title:`, `id:` en chaîne, images `artworks.thetvdb.com`. Plus aucun compte ne peut les lire (les règles exigent `uid == documentId`) | Aucun | **Purge console manuelle** avant lancement public. Pas d'US |

## Ce que cette campagne dit du produit

Sept des constats tranchés produisent la même expérience : **l'app affirme quelque chose de
faux et ne se contredit jamais.** « Saved » sans écriture (`AUD-29`), « Imported 300 animes »
dans le mauvais bac (`AUD-03`), « Finished » sur un statut inconnu (`AUD-15`), une liste qui
s'arrête à 100 sans le dire (`AUD-27`), un anime présent qui disparaît (`AUD-10`), une modale
de relation vide (`AUD-16`), et une liste entière effacée en silence (`AUD-30`).

La campagne S38 avait déjà écrit la conclusion : *le produit n'a pas un problème de bugs, il a
un problème de véracité.* Aucun de ces défauts ne produit de crash — c'est précisément pour
cela qu'ils ont survécu à quarante sprints. Un utilisateur ne signale pas un mensonge
silencieux : il arrête d'utiliser l'app.
# Campagne SE-065 — clôture AUD-30

## Constat corrigé

| ID | Correction |
|---|---|
| **AUD-30** | ✅ **FERMÉ — et les deux causes écrites étaient fausses.** La cause (2) affirmait « le cloud est effacé par l'app elle-même ». **Non.** L'écriture au logout n'atteint jamais Firestore : `useFirestore.ts:81` (`if (!auth.currentUser) return`) sort en silence, `currentUser` étant déjà `null` après `signOut()`. Le constat contredisait sa propre preuve — SE-064 avait vu `schedules` peuplé en console. **Les 0 refus du moteur de règles s'expliquent par l'absence totale d'écriture, pas par une écriture autorisée.** Les deux vraies causes : **(1)** `App.vue:38-41` appelle `loadFromDatabase()` dans `onMounted`, une seule fois, alors que l'utilisateur est encore sur `/login` sans `currentUser` ; la connexion étant une navigation SPA, `App.vue` ne se remonte jamais et la lecture ne rejoue pas. **(2)** `AppHeader.vue:58` `clearAll()` armait le watcher avant la purge ; la sauvegarde différée recréait `aanime_calendar = {timestamp: now, data: []}` 1 s plus tard, bloquant à jamais `firestoreData.timestamp > cacheTimestamp`. Soldé par `US-PERSIST-P0b` + `US-PERSIST-P0a2` | vérifié par le PO en parcours réel |
| **AUD-17** | 🔁 **ROUVERT puis soldé.** Le constat visait **deux** stubs. Seul `_startBackgroundRelationFetch` avait été supprimé (DEC-147) ; `_syncAnimeUpdates` a survécu jusqu'à SE-065. Fermeture prématurée à ne pas reproduire : un constat portant sur N éléments ne se ferme qu'après vérification des N | grep `syncAnimeUpdates` |
| **AUD-12** | ✅ **CONFIRMÉ VIVANT.** `localStorage.setItem` est bien **hors du `try`** dans `saveToDatabase`. Un quota dépassé empêcherait la sauvegarde Firestore qui suit, sans message. Non corrigé — à slotter | lecture complète de `usePersistence.ts` |
| **AUD-02** | ✅ **Confirmé soldé en production.** La chaîne d'erreur remonte intégralement jusqu'au toast : capture console du PO, `useFirestore.ts:88` → `usePersistence.ts:130` | console prod |

## Constats nouveaux — SE-065

| ID | Constat | Impact utilisateur | Destination |
|---|---|---|---|
| **AUD-32** | 🟠 `POST graphql.anilist.co` répond **429 Too Many Requests** en usage réel (ajout de plusieurs animes d'affilée). Le message `blocked by CORS policy` qui suit est un **artefact** : une réponse 429 ne porte pas d'en-tête CORS. **Ne pas ouvrir d'US CORS** | Les animes ajoutés n'apparaissent pas dans Week sans actualiser la page | `US-ANILIST-429` — S41 |
| **AUD-33** | 🟢 `usePersistence.guard.spec.ts` dépend de son **ordre interne** : le watcher étant module-level, seul le premier test du fichier dispose d'un watcher vivant. Un déplacement du test rendrait la garde faussement verte. Commentaire de garde posé dans le fichier | Aucun — piège de test | Aucune US, note permanente |

## Rappel de méthode confirmé

`AUD-29` (`useFirestore.ts:81` retourne sans sauvegarder pendant que l'app affiche « Saved ») et
`AUD-27` (plafond 100 entrées + timestamp monotone) sont **tous deux observés en production**
sur le compte du PO. Ils passent en tête de S41.


# Campagne SE-065 — exécution S41 (persistance)

## Constats soldés

| ID | Verdict | Preuve |
|---|---|---|
| **AUD-13** | ✅ **Soldé.** `AppHeader.vue` ne pilote plus l'état mémoire : `clearAll()` supprimé, remplacé par un rechargement complet | `US-PERSIST-P0b` |
| **AUD-17** | ✅ **Soldé pour de bon.** La clôture antérieure était partielle : seul `_startBackgroundRelationFetch` avait été supprimé. `_syncAnimeUpdates` l'est depuis SE-065. La vraie sync vit en `useSync.ts:58`, appelée en 4 points vivants | micro-patch DEC-128 |
| **AUD-30** | ✅ **Soldé, et le diagnostic de SE-064 était à moitié faux.** Cause (1) confirmée. Cause (2) **requalifiée** : le cloud n'est jamais effacé — `useFirestore.ts:81` (`if (!auth.currentUser) return`) bloque l'écriture au logout. La corruption est **100 % locale** et agit en **bloquant la lecture**, pas en effaçant l'écriture. D'où les 0 refus côté règles : il n'y a jamais eu d'écriture à refuser. **Deuxième cause découverte en SE-065** : `loadFromDatabase()` ne rejoue jamais après connexion | `US-PERSIST-P0b` + `P0a2` |
| **AUD-12** | ✅ **Confirmé vivant, puis toléré.** `localStorage.setItem` est bien hors du `try` dans `saveToDatabase`. Impact réel faible depuis `P0b` : une écriture vide ne fait plus autorité. **Reste ouvert, non prioritaire** | lecture complète du fichier |

## Constats nouveaux — SE-065

| ID | Constat | Impact utilisateur | Destination |
|---|---|---|---|
| **AUD-32** | 🔴 **Écriture Firestore refusée en production.** `Firestore Error: {"error":"Missing or insufficient permissions","operationType":"write","path":"schedules/<uid>"}`, remontée par `useFirestore.ts:39` puis `usePersistence.ts:130`. Le document ID est un UID Auth valide : la règle `uid == documentId` n'est donc pas en cause. Reste `data.size() <= 100` ou le timestamp monotone (`AUD-27`) — **non tranché** | Une sauvegarde échoue ; le toast d'erreur s'affiche bien, mais la donnée cloud diverge du local | `US-FIRESTORE-LIMITS` — S41. 🔴 **Lire `firestore.rules` et compter les entrées du document AVANT de rédiger** |
| **AUD-33** | 🟠 **AniList `429 Too Many Requests` au démarrage**, suivi d'erreurs CORS (`Access-Control-Allow-Origin` absent sur la réponse d'erreur). Plusieurs occurrences par chargement, depuis `anilistClient.ts:116` | Les animes n'apparaissent pas dans Week au premier chargement ; il faut actualiser | À instruire en SE-066 |
| **AUD-34** | 🔴 **`TYPES_CONTRACT.md §9` affirmait `addAnime(input: Partial<AnimeEntry>)`.** Le vrai type est `AddAnimeInput` (14 champs obligatoires). Une US 🔴 a été livrée rouge sur la foi du contrat | Aucun — mais un aller-retour Gemini perdu | Corrigé en SE-065 |
| **AUD-35** | 🔴 **`AGENTS.md R-CODE-2` impose le helper `makeAnime()`, supprimé du dépôt** (purge `helpers.ts`, `J11b-3`). Gemini a proposé de l'utiliser pour contourner un blocage | Aucun — mais l'agent opère sous une règle sans objet | Corrigé en SE-065 |
| **AUD-36** | 🔴 **`US-PERSIST-P0a` est verte et sans effet.** Le correctif visait `to.meta.guestOnly && isLoggedIn`, branche jamais atteinte lors d'une connexion réelle. Le test de fidélité testait la même mauvaise branche : **vert par construction**. Aucune porte de qualité ne peut détecter ce défaut — c'est une erreur de spec | Aucun — code inoffensif, mais mort | Suppression à instruire en SE-066 |
# Campagne SE-066 — exécution S41 (onboarding, import MAL, saison)

## 🔴 Note de réconciliation d'identifiants — à lire avant d'utiliser AUD-32 / AUD-33

Deux campagnes de ce fichier portent le titre « SE-065 » et attribuent **les mêmes numéros à
des constats différents**. `AUDIT.md` étant append-only, rien n'est réécrit. Arbitrage retenu,
aligné sur `STATE.md` :

| ID | Constat qui garde l'ID | Constat évincé |
|---|---|---|
| **AUD-32** | Écriture Firestore refusée en production 🔴 | AniList 429 |
| **AUD-33** | AniList 429 au boot 🟠 | Piège d'ordre `usePersistence.guard.spec.ts` |

Le piège d'ordre de `usePersistence.guard.spec.ts` **reste vrai et sans ID** : le watcher étant
module-level, seul le premier test du fichier dispose d'un watcher vivant. Le commentaire de
garde est posé dans le fichier — c'est sa seule trace opposable. Aucune US.

**Leçon de méthode :** un identifiant attribué deux fois coûte plus cher qu'un trou de
numérotation. Un `AUD-xx` neuf se prend **au-dessus du max constaté**, jamais au premier
numéro qui paraît libre.

## Constat requalifié

| ID | Correction |
|---|---|
| **AUD-13** | 🔁 **Marqué ✅ Soldé à tort en campagne « SE-065 — exécution S41 ».** `US-PERSIST-P0b` n'avait supprimé que `clearAll()`. La purge `localStorage` de toutes les clés `aanime_` a survécu dans `AppHeader.vue:54-56` jusqu'à `US-ONBOARD-PERSIST-A`. **Soldé pour de bon depuis `b509ca0`.** Deuxième clôture prématurée du fichier après `AUD-17` : un constat portant sur N éléments ne se ferme qu'après vérification des N |

## Constats nouveaux — SE-066

| ID | Constat | Impact utilisateur | Destination |
|---|---|---|---|
| **AUD-37** | 🟠 Un anime MAL au statut « Watching » atterrit en Plan to Watch, pas au calendrier. **Correct techniquement** : MAL n'exporte aucun jour de diffusion, et `addAnime` refuse toute entrée `state:'calendar'` sans `day` (garde anti-`AUD-01`). **Non vérifié :** `useSync` récupère-t-il ensuite le `day` chez AniList et fait-il remonter l'entrée au calendrier ? | « J'ai importé mes 40 séries en cours et elles sont toutes en *à voir plus tard* » | **À trancher sur un vrai fichier MAL, pas sur du code** |
| **AUD-38** | ✅ **Corrigé par `US-SEASON-SKIP-SESSION` (`80f9d11`).** `dismissRec` appelait `trackNegative` sans condition — DEC-159 était violée en production depuis sa rédaction | Écarter un titre de This Season le bannissait définitivement de For You. `localStorage` vérifié `null` après correctif | Soldé |

## 🎓 Contre-mesure de campagne

Le test rouge de `US-MALIMPORT-FIX` était **une erreur de spec, pas de code**. Il attendait
`state:'calendar'` ; le store requalifie en `watchlist` + `awaitingSchedule: true`. **Le store
a refusé la spec du Tech Lead et il avait raison.** Corrigé par micro-patch DEC-128, escalade
Gemini exemplaire (test rouge signalé sans toucher au test ni au store).

→ Généralisé en **DEC-165**.
# Campagne SE-067 — clôture S41 (réseau, quotas, cartes)

## Constats soldés

| ID | Verdict | Preuve |
|---|---|---|
| **AUD-32** | ✅ **MORT — et il n'a jamais existé sous cette forme.** L'écriture Firestore est autorisée : `title: "Chainsmoker Cat"` présent dans `schedules/<uid>`, `timestamp: 1787567729316`, console Firebase. Le constat a gelé `US-FIRESTORE-LIMITS` et l'accès bêta pendant **trois sessions** sur une erreur observée une seule fois et jamais rejouée. `PILOTAGE §6` interdit exactement ça | console Firebase, SE-067 |
| **AUD-27** | ✅ **Soldé.** Plafond `data.size()` porté de 100 à 500 dans `firestore.rules` **et publié en console**. Poids réel d'une entrée mesuré à **~1,1 ko** sur échantillon : 500 entrées ≈ 550 ko, soit 45 % de marge sous la limite plateforme de 1 Mo. Le plafond dur réel se situe vers 900 titres | échantillon réel + console → Règles |
| **AUD-31** | ✅ **Soldé.** Collection `schedules` purgée par le PO. Un seul document vivant. Le constat annonçait 8 fossiles ; la capture console en montrait **au moins 14** (`Adn`, `Ae`, `Ael`, `ade`, `adn`, `adnanne`, `aer`, `aze`…), aucun n'étant un UID Auth valide | console Firebase |
| **AUD-33** | ⚠️ **Soldé sous réserve.** Quatre causes chaînées établies par lecture de code, trois corrigées (`US-ANILIST-QUEUE-A/B`, `US-SYNC-PRIORITY`). Plus aucun 429 observé en production. **Mais le hash du chunk servi n'a pas été relevé** : le correctif est plausiblement en ligne, pas prouvé. Falsifiable par un 429 après 5+ ajouts d'affilée | constat PO en prod |
| **AUD-38** | ✅ Soldé en SE-066 par `US-SEASON-SKIP-SESSION` | `80f9d11` |

## Les quatre causes d'AUD-33 — à conserver, elles reviendront

1. **File unique sans priorité.** `anilistClient.ts` n'a qu'une `requestQueue`. La sync de fond et la recherche de l'utilisateur passent par le même tuyau, dans l'ordre d'arrivée. `AUD-19` notait que l'ancienne file `low`/`high` avait été supprimée — **elle n'a jamais été remplacée.**
2. **L'attente d'un 429 se faisait DANS la file.** `Retry-After: 60` × `retries: 3` = jusqu'à **120 s de gel total**. `F5` vidait la file (`requestQueue = []` au rechargement du module) — d'où « il faut actualiser pour voir ses animes ». Corrigé : plafond `MAX_RATELIMIT_WAIT_MS = 3000`.
3. **Un anime rate-limité n'était jamais horodaté.** `if (result.failed) continue;` sans `setSyncTimestamp` → il revenait à **chaque** démarrage. Boucle auto-entretenue : plus d'échecs → plus d'entrées non horodatées → plus de requêtes au boot → plus d'échecs. Corrigé par horodatage antidaté (`RETRY_TTL_MS`).
4. **700 ms = 85,7 req/min contre une limite nominale de 90/min.** Le régulateur tournait à 95 % du plafond, sans marge. Porté à 1 200 ms (50 req/min, 55 %).

## 🔗 AUD-37 est un symptôme d'AUD-33, pas un constat indépendant

Un import MAL pose `awaitingSchedule: true` sur toutes ses entrées — exactement ce que capte le filtre de `useSync`. **300 titres = 300 requêtes d'affilée**, rate limit garanti, entrées non horodatées (cause 3), donc jamais de `day` récupéré, donc blocage définitif en « Plan to Watch ».

`usePersistence.ts:245` aggrave en forçant `status = 'Continuing'` sur les shows sans horaire, ce qui les remet dans le filtre à chaque boot.

**Non vérifié sur un vrai fichier MAL.** Trois correctifs le visent ; aucun n'est observé.

## Constats nouveaux — SE-067

| ID | Constat | Impact utilisateur | Destination |
|---|---|---|---|
| **AUD-39** | 🟢 `AnimeCard.vue` affiche `anime.title`, `RecCard.vue` affiche `anime.title_english \|\| anime.title`. **Le même anime porte deux noms selon l'écran** | On ne reconnaît pas une série déjà croisée, donc on ne la retrouve pas | Absorbé par `US-CARD-CONVERGE-A/B` — S42 |
| **AUD-40** | 🟢 **CSS mort et contradictoire.** `.card-cp-why` + ses 3 enfants : zéro occurrence dans `src\*.vue`. `.rec-why-panel`, `.rec-why-signals`, `.rec-why-signal`, `.rec-why-not-interested` : mortes aussi — `RecCard` utilise `why-panel` / `signal-chip` / `why-act-btn`. Plus le doublon `.card-cp-title` (l.931 et l.1936, même spécificité, la seconde gagne). ~40 lignes | Aucun direct — mais quiconque éditera le mauvais bloc croira que le CSS ne s'applique pas | Lot dette CSS — S43 |
| **AUD-41** | 🟢 `studios: ["8-bit", "8-bit"]` observé sur une entrée Firestore réelle : doublon dans la normalisation AniList. **Non vérifié à l'écran** | Possible « 8-bit, 8-bit » affiché | À instruire, sans US |
| **AUD-42** | 🟠 `aanime_sync_ts` vit en `localStorage`, **jamais dans Firestore**. Chaque nouvel appareil repart d'une carte vierge et relance une sync complète au premier démarrage — 100 titres = 2 min de rafale sur un appareil neuf | Un appareil neuf est lent et se rate-limite au premier lancement | Avec `US-ONBOARD-PERSIST-B` — S42 |
| **AUD-43** | 🟠 **Gemini ne construit pas le même arbre.** 3ᵉ occurrence d'écart de build (439,5 kB annoncés vs 368,7 kB réels, écart constant de 71 kB). La vraie cause n'est pas un mensonge sur les chiffres : les **noms de chunks diffèrent structurellement** (`AnimeCard-*.js` vs `AnimeCard.vue_vue_type_script_setup_true_lang-*.js`) et `index.html` lui-même diverge de 10 o. Plugin Vue ou version Vite différents. **Conséquence dépassant le bundle : son `vue-tsc` peut tourner sur un autre TypeScript — un type-check vert chez lui ne prouve rien** | Aucun | Note permanente, aucune US |
| **AUD-44** | 🟢 `syncAnimeUpdates` pose `isSyncing.value = true` en entrée et `= false` en sortie, **hors de tout `try/finally`**. Un throw sur `saveToDatabase` laisserait le spinner de `SyncIndicator.vue` tourner indéfiniment. Latent : `saveToDatabase` capture et ne relance pas | Aucun aujourd'hui — spinner bloqué si la condition se réalise | S43, lot ESLint. **Ne pas corriger en marge d'une autre US** : le `try/finally` impose de réindenter ~90 lignes, terrain idéal pour une réécriture silencieuse |

## 🎓 Deux leçons de méthode de la campagne

**1. Une erreur observée une fois n'est pas un fait.** `AUD-32` a gelé le bloquant n°1 du projet pendant trois sessions. Deux minutes de console Firebase l'ont tué. Symétriquement, `AUD-33` est déclaré soldé sans que le hash servi ait été relevé : le même défaut de preuve, dans l'autre sens.

**2. Un correctif peut créer le bug suivant.** `US-ANILIST-QUEUE-B` a plafonné la sync à 25 sans toucher au tri : les entrées `awaitingSchedule` étant `state: 'watchlist'`, elles passaient **après** toute la bibliothèque déjà visible. Famine ordonnancée introduite par le correctif lui-même, rattrapée par `US-SYNC-PRIORITY` dans la même session. **Un plafond sans révision de la priorité change qui est servi, pas seulement combien.**
