# AUDIT.md — Inventaire des constats de code

> **Rôle :** le *contenu* des constats — ce que c'est, où c'est (`fichier:ligne`), ce que ça coûte
> à l'utilisateur. **Jamais la planification** : `STATE.md §Kanban` et `ROADMAP.md` décident du
> sprint, ce fichier ne dit jamais *quand*.
>
> **Organisé par état, jamais par campagne** (H7). Un constat soldé n'est **jamais supprimé** :
> il bascule au §2 en une ligne — ID, titre, verdict, preuve, session. Le raisonnement détaillé
> d'un diagnostic clos vit dans l'historique Git, pas ici.
>
> **Mise à jour : une fois par sprint**, par le lot documentaire (`DEC-190`). En cours de sprint,
> un constat neuf s'écrit **en une ligne dans `HISTORIQUE §2 Tampon`** avec son numéro attribué.
>
> 🔴 **Un `AUD-xx` neuf se prend au-dessus du max constaté**, jamais au premier numéro qui paraît
> libre. *Leçon SE-066 : un identifiant attribué deux fois coûte plus cher qu'un trou de
> numérotation (collision `AUD-32`/`AUD-33`, arbitrée en faveur de « écriture Firestore refusée »
> et « AniList 429 au boot »).*
> **Trous connus :** `AUD-21` et `AUD-22` sont cités dans `HISTORIQUE` mais n'ont jamais été
> écrits ici. Ne pas réattribuer.

**Dernier numéro attribué : `AUD-59`.**

---

## 1. Constats OUVERTS

### 🔴 P0

| ID | Constat | Localisation | Impact utilisateur |
|---|---|---|---|
| **AUD-59** | `saveToDatabase` **ne rejette jamais** sur échec Firestore : son `try/catch` interne avale l'erreur et affiche son propre toast. Le `localStorage.setItem` est **hors** du `try`. Le `catch` de `finishWithSeed` n'est donc atteignable **que** sur panne de stockage local — et il y affiche « Saved on this device » | `usePersistence.ts:116-135` (source) · `OnboardingPage.vue:109,123` (symptôme) · chaîne assertée en `OnboardingPage.persist.spec.ts:112` | **Le message de succès local s'affiche exactement quand rien n'a été sauvegardé localement.** Sur panne Firestore : deux toasts contradictoires, dont **le seul texte français de l'app**. Et `saveError` n'est relu par personne — aucun réessai n'est ordonnancé |

> ⚠️ **`AUD-59` ne se corrige pas en micro-patch.** La chaîne exacte est assertée en spec : toute
> reformulation touche 2 fichiers → US 🟠 avec test de fidélité réécrit par Claude (`R7`).
> Famille `AUD-02` — « un bug qui confirme un succès inexistant coûte la confiance ».

### 🟠 Standard

| ID | Constat | Localisation | Impact utilisateur |
|---|---|---|---|
| **AUD-03** | Occurrence résiduelle de la règle « en cours de diffusion » dupliquée. `useAddAnime.resolveTargetState` est la source unique ; `utils/onboardingFilter.ts` porte encore un exemplaire divergent | `utils/onboardingFilter.ts` (`DEC-178`) | Un anime en cours ajouté depuis l'onboarding peut atterrir dans le mauvais onglet |
| **AUD-05** | Deux signaux de fraîcheur concurrents, **tous deux sans consommateur dans l'UI**. `DEC-158` (a) tranche : source unique = `WithMeta.stale` ; `staleDataWarning` est à supprimer | `useAniListApi.ts:23,338` · `usePersistence.ts:18,192,305` | Saison affiche « vous avez déjà tout ajouté » pendant une panne · Onboarding : grille vide muette · Library : page blanche. **Débloqué**, porté par `US-STALE-SIGNAL` |
| **AUD-08** | CI sans Playwright **ni ESLint**. 🔴 **Il n'existe aucun script `lint` dans `package.json`** — ESLint n'a jamais tourné une seule fois | `.github/workflows/ci.yml` · `package.json` | Aucun visible — dette. La règle « zéro `any` » (`R-CODE-1`, `DEC-02`) n'est vérifiée par **aucun** outil de la porte verte |
| **AUD-29** | `if (!auth.currentUser) return;` retourne **sans sauvegarder et sans signaler**. `saveToDatabase` affiche alors « Saved » | `useFirestore.ts:81` | L'app confirme une sauvegarde qui n'a pas eu lieu. Même famille qu'`AUD-59` |
| **AUD-35** | 🔁 **ROUVERT.** `AGENTS.md:72` affirme « le helper `makeAnime()` n'existe plus — ne pas le réintroduire ». **Mesure du 2026-09-02 : `function makeAnime` est présent dans 5 fichiers de `src/**/*.spec.ts`** | `AGENTS.md:72` vs `src/**/*.spec.ts` | Aucun — mais l'agent opère sous une règle contredite par le dépôt, dans le seul document qu'il lit |
| **AUD-37** | Un anime MAL « Watching » atterrit en Plan to Watch, pas au calendrier. **Correct techniquement** (MAL n'exporte aucun jour ; `addAnime` refuse `calendar` sans `day`). Symptôme d'`AUD-33` : 300 titres = 300 requêtes, rate limit, entrées non horodatées, `day` jamais récupéré | `useSync` · `usePersistence.ts:245` force `status='Continuing'` sans horaire | « J'ai importé mes 40 séries en cours et elles sont toutes en *à voir plus tard* ». 🟠 **DÉLÉGUÉ À LA BÊTA** (`DEC-175`) — jamais vérifié sur un vrai fichier MAL |
| **AUD-39** | `AnimeCard.vue` affiche `anime.title`, `RecCard.vue` affiche `title_english \|\| title`. **Le même anime porte deux noms selon l'écran** | 2 composants | On ne reconnaît pas une série déjà croisée. Partiellement absorbé par `US-CARD-CONVERGE-A` (livrée) ; **`-B` reste à faire** |
| **AUD-44** | `syncAnimeUpdates` pose `isSyncing = true/false` **hors de tout `try/finally`** | `useSync.ts` | Aucun aujourd'hui — spinner de `SyncIndicator` bloqué si un throw survient. ⚠️ **Ne pas corriger en marge d'une autre US** : le `try/finally` impose de réindenter ~90 lignes |
| **AUD-50** | `AnimeCard.vue` conserve 3 consommateurs aux sémantiques d'action différentes (« Add » / « Rewatch » ? / « Start watching » ?) | `DiscoverComingUpPage` · `LibraryCompletedPage` · `LibraryPlanToWatchPage` | Aucun visible. ⚠️ Requalifié **NON-CONSTAT** par `ROADMAP §Règles gravées` : la ligne de partage `RecCard`/`AnimeCard` était déjà tranchée. Reste ouvert au sens d'un chantier de convergence non fini |
| **AUD-51** | La pastille « Add » de `RecCard` en mode sans Skip mesure **36 px**, sous la cible tactile de 44 px. C'est le seul bouton d'action de la carte | `RecCard.vue` | Cible tactile sous la norme. Arbitrage PO assumé, **à revoir sur constat de testeur** |
| **AUD-54** | **La vue Semaine reconstruit ses cartes pendant la synchronisation.** Observé par Playwright (élément détaché du DOM, 29,5 s de clics) puis à l'œil en 3G. Mesures : `Load 6,88 s` en 3G vs `2,22 s` en Slow 4G, `Finish 23,21 s`, 5,8 Mo pour un écran | Composant Semaine — **non lu** | Le tap atterrit sur la bonne fiche (vérifié 2×). Ce n'est pas une perte de clic mais **~7 s d'agitation visuelle** au démarrage, sur l'écran ouvert quotidiennement. 🔻 Cause non établie · mesures prises avec `Disable cache` coché, non représentatives |
| **AUD-56** | **Audit pixel complet non fait.** Toutes les pages, toutes les modales, thèmes clair ET sombre. Aucune mesure. Sous-lot connu : aucun token de couleur hors `--accent`, les 5 teintes de l'en-tête sont en dur | `AppHeader.vue` (`nth-child(1..5)` × 2 thèmes) | Inconnu et potentiellement large. `AUD-55` prouve qu'un défaut visuel peut vivre des mois sans qu'aucun test ne le voie. **Méthode arrêtée :** par parcours, 375 × 812, un thème à la fois |

### 🟢 Dette

| ID | Constat | Localisation | Impact utilisateur |
|---|---|---|---|
| **AUD-10** | 🟡 **Périmètre réduit de 4 cas à 1.** Trois des quatre `show: false` portent `isFinished: true` (légitime). Seule la l.33 masque une entrée non finie | `episodeInfo.ts:33` | Un anime présent, `day` renseigné, **disparaît** de la semaine |
| **AUD-12** | `localStorage.setItem` est **hors du `try`** dans `saveToDatabase`. Impact réel faible depuis `US-PERSIST-P0b` (une écriture vide ne fait plus autorité) | `usePersistence.ts` | Un quota dépassé empêcherait la sauvegarde Firestore qui suit, sans message. **Confirmé vivant, toléré** |
| **AUD-15** | Fallback final `word: 'Finished'`, atteint sur statut nul ou inconnu | `episodeInfo.ts:111` | Un anime au statut inconnu s'affiche « Finished », pastille grise |
| **AUD-20** | ⏸️ **Doute jamais levé.** Ligne `usePersistence.ts:200` citée par un auditeur, non corroborée | `usePersistence.ts:200` | Inconnu. Commande de levée : `findstr /n "^" src\composables\usePersistence.ts \| more +194` |
| **AUD-40** | **CSS mort et contradictoire.** `.card-cp-why` + 3 enfants : zéro occurrence dans `src\*.vue`. `.rec-why-panel` / `-signals` / `-signal` / `-not-interested` mortes aussi (`RecCard` utilise `why-panel` / `signal-chip` / `why-act-btn`). Plus le doublon `.card-cp-title` (même spécificité, la seconde gagne). ~40 lignes | `style.css` | Aucun direct — mais quiconque éditera le mauvais bloc croira que le CSS ne s'applique pas |
| **AUD-41** | `studios: ["8-bit", "8-bit"]` observé sur une entrée Firestore réelle : doublon dans la normalisation AniList. **Non vérifié à l'écran** | `normalizeAniList.ts` | Possible « 8-bit, 8-bit » affiché |

---

## 2. Constats CLOS — ne pas rouvrir sans preuve neuve

> Une ligne par constat. Le verdict et sa preuve suffisent à ne pas refaire le chemin.

| ID | Titre | Verdict | Session |
|---|---|---|---|
| **AUD-01** | Entrée `calendar` sans `day`, 13 producteurs sur 14 | ✅ Soldé — `DEC-124` pose `day`+`airsTime` par cascade, `DEC-131` marque `awaitingSchedule`, et `addAnime` refuse toute entrée `calendar` sans `day` | S41 |
| **AUD-02** | `saveSchedule` rattrapait son propre throw → « Saved » pendant une panne | ✅ Soldé — `useFirestore.ts:89-91` relance ; chaîne d'erreur vérifiée en console de production | SE-064 / SE-065 |
| **AUD-04** | Coupe-circuit à portée globale contaminant les endpoints | ⛔ Annulé (`DEC-126`) — AniList n'expose **qu'un** endpoint, un breaker global y est correct. Leçon transposée : un `429` n'incrémente jamais le compteur | SE-063.b |
| **AUD-06** | `ToastNotification` monté uniquement dans `AppLayout` | ✅ Soldé — le volet `LoginPage` était déjà en production ; l'onboarding couvert par `US-ONBOARD-TOAST` | SE-068 |
| **AUD-07** | Flag d'onboarding en `localStorage` seul, effacé au logout | ✅ Soldé — `US-ONBOARD-PERSIST-A/B`. 🔻 Clôture dérivée de l'observation `AUD-42` (SE-074), pas d'une relecture directe | SE-074 |
| **AUD-09** | Specs E2E n'assertant rien de significatif | ✅ Quasi soldé — sur 60 usages `localStorage` en E2E, 59 sont du seed légitime ; 1 assertion sur le store subsiste (`onboarding-seed.spec.ts:56`) | SE-064 |
| **AUD-11** | Props non déclarées sur `EmptyState` → fallthrough HTML | ✅ Soldé — `EmptyStateProps` = `title` + `subtitle`, tous deux rendus | SE-064 |
| **AUD-13** | Couches violées : `localStorage` piloté depuis `AppHeader.vue` | ✅ Soldé — `b509ca0`. **Deux clôtures prématurées avant celle-ci** : un constat portant sur N éléments ne se ferme qu'après vérification des N | SE-066 |
| **AUD-14** | Typage : la factory `makeAnime` était elle-même un cast | ✅ Soldé — `as AnimeEntry` : zéro hit dans `tests\**` et `src\**\*.ts` | SE-064 |
| **AUD-16** | Navigation relations : `{mal_id, id, title} as AnimeEntry` | ✅ Soldé — `US-MORELIKETHIS-FIX`. 🔻 Le constat citait `AnimeModal.vue:179` ; la ligne réelle est **169**, dans `src/components/ui/` | SE-073 |
| **AUD-17** | Stubs vides `_syncAnimeUpdates` / `_startBackgroundRelationFetch` | ✅ Soldé pour de bon — le second en `J11b-1`, le premier en SE-065. La vraie sync vit en `useSync.ts:58`, appelée en 4 points | SE-065 |
| **AUD-18** | `useRecommendations` (549 l.) sans aucun test | ✅ Partiellement soldé — `useRecommendations.spec.ts` + `.nudges.spec.ts` (24 tests). Aucun autre composable n'était dans ce cas | SE-061 |
| **AUD-19** | Famine de la file `low` par la file `high` | ✅ Soldé — `helpers.ts` ne contient plus que `escapeHTML`, `getWeekNumber`, `dedupeByMalId`. Aucune file, aucun disjoncteur | SE-064 |
| **AUD-23** | `onboarding-toast.spec.ts` décrite comme rouge | ⛔ Caduc — verte (batch4, 2 passages). Soldée sans avoir été notée | SE-068 |
| **AUD-24** | 12 specs E2E vertes tapant le réseau réel | ✅ Absorbé par `AUD-52` famille B, migrée 10/10 vers `installAniListMock` | SE-072 |
| **AUD-25** | Asymétrie d'action This Season (2 taps) vs For You (1 tap) | ✅ Soldé — `DiscoverSeasonPage` passée sur `RecCard` (`7e0a092`), `US-CARD-CONVERGE-A` | SE-068 |
| **AUD-26** | Divergence des clés de seed E2E `animeCalendar` / `aanime_calendar` | ⛔ Annulé, faux positif — `usePersistence.ts:139-150` porte une table de migration. Fragilité réelle mais mineure | SE-064 |
| **AUD-27** | Règles Firestore : plafond 100 entrées + timestamp monotone | ✅ Soldé — plafond porté à **500** et **publié en console**. Entrée mesurée à ~1,1 ko : 500 ≈ 550 ko, 45 % de marge sous 1 Mo. Plafond dur réel vers 900 | SE-067 |
| **AUD-28** | 0 lecture Firestore en vue d'ensemble | ✅ Annulé — artefact d'agrégation ; 62 lectures réelles sur 7 jours | SE-064 |
| **AUD-30** | 🔴🔴 Aucune persistance — bloquant de lancement bêta | ✅ Soldé — **et le diagnostic de SE-064 était à moitié faux.** Le cloud n'est jamais effacé : `useFirestore.ts:81` bloque l'écriture au logout. La corruption était 100 % **locale** et agissait en **bloquant la lecture**. 2ᵉ cause : `loadFromDatabase()` ne rejouait jamais après connexion. `US-PERSIST-P0b` + `P0a2` | SE-065 |
| **AUD-31** | 8+ documents fossiles à ID pseudo dans `schedules` | ✅ Soldé — collection purgée par le PO. La capture en montrait **au moins 14**, aucun n'étant un UID Auth | SE-067 |
| **AUD-32** | Écriture Firestore refusée en production | ✅ **MORT — il n'a jamais existé sous cette forme.** L'écriture est autorisée (document vérifié en console). **A gelé le bloquant n°1 pendant 3 sessions sur une erreur observée une fois, jamais rejouée** | SE-067 |
| **AUD-33** | AniList `429` au boot, 4 causes chaînées | ✅ Fermé sur preuve — chunk prod changé seul entre 2 mesures → AI Studio redéploie depuis `main`. Combiné à `AUD-45`, ce qui est servi est ce qui est déployé. 🔻 Falsifiable par un 429 après 5+ ajouts d'affilée | SE-068 |
| **AUD-34** | `TYPES_CONTRACT §9` affirmait `addAnime(input: Partial<AnimeEntry>)` | ✅ Corrigé — le vrai type est `AddAnimeInput` (14 champs obligatoires). Une US 🔴 avait été livrée rouge sur la foi du contrat | SE-065 |
| **AUD-36** | `US-PERSIST-P0a` verte et sans effet (branche jamais atteinte) | ✅ Soldé — US supprimée après avoir traîné 4 sessions. Fonde `AP-PROCESS-5` | SE-068 |
| **AUD-38** | `dismissRec` appelait `trackNegative` sans condition | ✅ Soldé — `US-SEASON-SKIP-SESSION` (`80f9d11`). `DEC-159` était violée en production depuis sa rédaction | SE-066 |
| **AUD-42** | `aanime_sync_ts` jamais dans Firestore → appareil neuf reparti de zéro | ✅ **Fermé sur observation** — second appareil, Clear site data → login → bibliothèque revenue | SE-074 |
| **AUD-45** | Service worker de production suspecté de cacher les chunks | ⛔ Requalifié non-constat — c'est le SW d'AI Studio, qui ne proxifie que `generativelanguage.googleapis.com`. **L'API `caches` n'apparaît nulle part** | SE-068 |
| **AUD-46** | Dropdown de recherche derrière les cartes de contenu | ✅ Soldé — cause réelle : `.app-layout .app-header { position: static }` non scopée. Un `z-index` sur un élément non positionné est ignoré. Fonde `AP-CSS-1` | SE-069 |
| **AUD-47** | `SearchInput.onQuickAdd` réimplémentait la règle d'ajout | ✅ Soldé — `US-SEARCH-USE-ADDANIME`. Trois écarts corrigés dont `resolveTargetState` ignorant `Continuing` | SE-069 |
| **AUD-48** | `DiscoverSeasonPage` filtrait le pool une seule fois, sous `<KeepAlive>` | ✅ Soldé — `US-SEASON-FRESH`, filtre passé en `computed` | SE-069 |
| **AUD-49** | Deux entrées numérotées `DEC-158` | ✅ Soldé — renumérotation (b) → `DEC-176`. *Même famille : la collision `DEC-159`, résolue en `DEC-191` en SE-075* | SE-069 |
| **AUD-52** | 17 specs mockant en direct, en 3 familles | ✅ Soldé — famille A (AniList) 5/5, famille B (Jikan mort) 10/10. **Zéro spec ne connaît plus une URL d'API.** 🔻 Famille C : `week-empty-day-cta` route `**/*`, fonctionne par accident | SE-072 |
| **AUD-53** | This Season sans réserve basse, dernière rangée sous la nav | ✅ Soldé — `padding: 1.5rem` → `1.5rem 1rem 5rem` | SE-070 |
| **AUD-55** | En-tête en bandeau gris sur fond noir en thème sombre | ✅ Soldé — les règles `:deep(html.dark)` ne matchaient rien (`DEC-187`). Le thème sombre de l'en-tête était mort depuis sa création | SE-072 |
| **AUD-57** | Le helper de mock ne pouvait produire aucune relation | ✅ Soldé — `relationsBody` omettait `type: 'ANIME'`, filtré par `useAniListApi.ts:446`. Aucun faux-vert historique : aucune spec ne configurait `relations` | SE-073 |
| **AUD-58** | « MORE LIKE THIS » servait la bibliothèque de l'utilisateur | ✅ Soldé — `ModalMoreLikeThis.vue` lisait `store.animeCalendarData` sans aucun appel réseau. `US-MLT-REAL` | SE-073 |

---

## 3. Notes permanentes

**`AUD-43` — Gemini ne construit pas le même arbre.** 3 occurrences d'écart de build (439,5 kB
annoncés vs 368,7 kB réels, écart constant de 71 kB). La cause n'est pas un mensonge sur les
chiffres : les **noms de chunks diffèrent structurellement** et `index.html` diverge de 10 o.
**Conséquence dépassant le bundle : son `vue-tsc` peut tourner sur un autre TypeScript — un
type-check vert chez lui ne prouve rien.** → `DEC-169`. Aucune US, jamais.

**Piège d'ordre sans ID.** `usePersistence.guard.spec.ts` dépend de son ordre interne : le
watcher étant module-level, seul le premier test du fichier dispose d'un watcher vivant. Un
déplacement rendrait la garde faussement verte. Commentaire de garde posé dans le fichier —
c'est sa seule trace opposable.

### 🚫 Findings écartés — ne pas les remettre au backlog

| Finding rejeté | Raison |
|---|---|
| « Spec E2E écrite mais non enregistrée » | 0 orpheline vis-à-vis de `package.json`, vérifié programmatiquement dans les deux sens |
| « `grid-two-columns` mesure du CSS sur une grille vide » | Le test **applique la remédiation codifiée** (`toHaveCount` puis `getComputedStyle`). Conforme |
| « `getCardStatus` ne mappe pas `Continuing` » | **Faux** — `episodeInfo.ts:106` lowercase, `:110` teste `'continuing'`. Faux négatif de casse `findstr` |
| « `v-html` inconditionnel » | Conditionné à `v-if="isHtml"`, conforme |

### 🔍 Zones jamais auditées

| Zone | Enjeu |
|---|---|
| **`utils/recEngine.ts`** | Scoring, `assignBadge`, `buildNextBatch` — le moteur de Discover et Library. Aucun audit, aucun test unitaire |
| **~20 composants `.vue`** | `AnimeCard`, `RecCard`, `WeekAnimeItem`, `ModalVersionTop`, `MonthDayCell`, `SyncIndicator`… toute la couche de rendu |
| **Accessibilité** | ARIA, ordre de tabulation, piège de focus dans les modales `Teleport`, contraste |

*(`firestore.rules`, longtemps la zone la plus risquée, a été lu et corrigé en SE-067 — `AUD-27`.)*

### 🎨 Lecture produit — la thèse qui tient depuis S38

**Le produit n'a pas un problème de bugs, il a un problème de véracité.** L'app affirme quelque
chose de faux et ne se contredit jamais : « 3 shows added » sur un calendrier vide (`AUD-01`),
« Saved » sans écriture (`AUD-29`), « Imported 300 animes » dans le mauvais bac (`AUD-03`),
« Finished » sur un statut inconnu (`AUD-15`), une liste qui s'arrête sans le dire (`AUD-27`),
une modale de relation vide (`AUD-16`), une liste effacée en silence (`AUD-30`), et aujourd'hui
« Saved on this device » quand rien n'est sauvegardé (`AUD-59`).

Aucun de ces défauts ne produit de crash — c'est précisément pour cela qu'ils ont survécu à
quarante sprints. **Un utilisateur ne signale pas un mensonge silencieux : il arrête d'utiliser
l'app.**

> Un bug qui échoue visiblement coûte un réessai. **Un bug qui confirme un succès inexistant
> coûte la confiance — et la confiance ne se réessaie pas.**

### 🎓 Leçons de méthode à conserver

1. **Une erreur observée une fois n'est pas un fait.** `AUD-32` a gelé le bloquant n°1 pendant
   trois sessions ; deux minutes de console l'ont tué. Symétriquement, `AUD-33` a été déclaré
   soldé sans que le hash servi ait été relevé — le même défaut de preuve, dans l'autre sens.
2. **Un correctif peut créer le bug suivant.** `US-ANILIST-QUEUE-B` a plafonné la sync à 25 sans
   toucher au tri : les entrées `awaitingSchedule` passaient après toute la bibliothèque visible.
   **Un plafond sans révision de la priorité change qui est servi, pas seulement combien.**
3. **Les 4 causes d'`AUD-33` — elles reviendront.** (a) File unique sans priorité : sync de fond
   et recherche utilisateur dans le même tuyau. (b) L'attente d'un 429 se faisait **dans** la
   file → `Retry-After: 60` × 3 = 120 s de gel total, d'où le réflexe F5. (c) Un anime
   rate-limité n'était jamais horodaté → il revenait à chaque démarrage, boucle auto-entretenue.
   (d) 700 ms = 85,7 req/min contre une limite de 90 — 95 % du plafond, sans marge.
4. **Un test capricieux n'est pas un test à réparer, c'est un capteur mal branché.**
   `modal-next-episode` semblait flaky ; elle voyait `AUD-54`, un comportement réel que personne
   n'avait observé.
5. **Sur un `as`, chercher ce qui n'est PAS dans l'objet.** Ni `vue-tsc`, ni les 331 tests, ni le
   build ne voyaient l'objet à 3 champs d'`AUD-16`. Seule une spec assertant un champ **absent**
   de l'objet tronqué pouvait le révéler.
6. **Un cadre d'audit identique révèle les angles morts.** Le dual audit S38 a montré que chacun
   des deux auditeurs avait raté le finding n°1 de l'autre. Le prompt de cette campagne affirmait
   par ailleurs l'état d'un service externe **sans remesure** — l'antipattern commis dans sa
   propre prémisse.

---

## 4. 🔄 Sprint courant

> Les constats du sprint en cours s'ajoutent **ici, à la fin**, une fois par sprint, depuis
> `HISTORIQUE §2 Tampon`. À la clôture suivante, ils rejoignent le §1 ou le §2.

*(vide)*
