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
