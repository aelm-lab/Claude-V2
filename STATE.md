# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état courant — sprint, session, versions,
> compteurs, Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient,
> ils ne dupliquent jamais une valeur. *Exception unique : `AGENTS.md`, lu par Gemini qui n'a
> pas accès à ce fichier — ses chiffres sont en dur et resynchronisés à chaque sprint.*
>
> **Ce qui n'est PAS ici :** les règles (→ `PILOTAGE.md`, `AGENTS.md`), le pourquoi des choix
> (→ `DECISIONS.md`), l'acquis fonctionnel (→ `EPICS.md`).

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint en cours** | **S38** — ouvert, Goal non atteint, **doit livrer du produit** |
| **Session courante** | **SE-050** — réconciliation dual audit + décision API, **0 code livré** |
| **Session suivante** | **SE-051** — rédaction des US S38 puis livraison |
| **Dernière version livrée** | v0.31.0 (S37) |
| **Dernier DEC** | **DEC-118** |
| **Commit `main`** | `c7cc60f` *(2026-08-04 17:20 — était `444d385` avant SE-050)* |

> **Rappel de modèle (détail → `PILOTAGE.md §1`) :** un **sprint** se ferme sur un Sprint Goal
> atteint ; une **session** se ferme sur la capacité de conversation et produit un handoff.
> Un sprint couvre N sessions. Une session hors sprint (chantier, audit) ne bumpe aucune version.

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-050** | S38 | Dual audit S38 réconcilié · décision API tranchée · 13 mesures de vérification | Handoff `SE-050 → SE-051`. **0 code livré** |
| SE-049 | — (hors sprint) | Refonte complète du corpus documentaire : audit, fusion, régénération | Corpus 13 → 9 docs. Handoff `SE-049 → SE-050` |
| SE-048 *(ex-« S38 »)* | S38 | Investigation Jikan + cache local + onboarding | 3 investigations P0 closes, 0 code livré |
| SE-047 *(ex-« S37 »)* | S37 | US-GRID-FIX, annulation US-MODAL-UNIFY | v0.31.0 |
| SE-046 *(ex-« S36 »)* | S36 | Resync registre E2E, overflow horizontal, centrage popups | v0.30.0 |

> Les sessions antérieures à SE-046 n'ont pas été numérotées séparément (le compteur `SXX`
> servait aux deux axes jusqu'en SE-049). Leur détail vit dans les handoffs archivés.

> ⚠️ **Trois sessions consécutives sans code livré** (SE-048, SE-049, SE-050). Le garde-fou
> `PILOTAGE §5` est en tension maximale : **SE-051 livre, ou le sprint S38 est un échec de
> pilotage, pas un échec technique.**

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.31.0** | **S37** | US-GRID-FIX (classe `.aa-card-grid` centralisée, 2 causes racines corrigées) · US-MODAL-UNIFY annulée après investigation (DEC-110) |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT (clos, DEC-108) · US-SEARCH-3 (clos sans dev). Résorbe le retard de bumps S30→S35 |
| v0.29.0 | S29→S33 | Refonte nav (header scroll iOS) · US-127 (SyncIndicator) · US-AUTH-LOGOUT |
| v0.28.0 | S28 | Epic Stats : `useStats` + `StatsPage.vue` + route `/stats` + onglet Stats |
| — | S23→S27 | ⚠️ Détail non capturé (vivait dans les handoffs archivés) |
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + consolidation `EPICS.md` |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| ≤ S16 | — | Migration vanilla → Vue 3 (Phases 0→7) · EPIC-1/2/3 clos · dual audit s16 |

> **Le numéro de version ne suit plus le numéro de sprint** depuis S29 (voir `PILOTAGE.md §1`).
> Cette table est la seule correspondance valide.

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **156 passed** (20 fichiers) | `npm run test:run`, machine PO | fin S37 |
| Type-check | **vert** | `npm run type-check`, sortie vide | fin S37 |
| Build | **vite 6.4.2 · 178 modules** | `npm run build` | fin S37 |
| E2E — fichiers | **39 sur disque** | `dir /b tests\e2e\*.spec.ts \| find /c ".spec"` | **SE-050** |
| E2E — enregistrement | **39 / 39, 0 orpheline** | diff programmatique batches 1→5 (audit Claude Code) | **SE-050** |
| E2E — tests | 46 tests, 45 verts | sweep machine PO | fin S37, **à rejouer** |
| Batchs | batch1 → batch5 | batch4/5 créés en S36 | — |

> ⚠️ **Correction SE-050 :** la mention « zéro `any` » a été **retirée** de ce tableau. Elle
> était une inférence de `vue-tsc`, qui ne teste pas les `any`. Mesure réelle :
> `src\utils\helpers.ts:32` contient `reject: (reason?: any) => void`. Seul ESLint le
> signalerait — **et ESLint n'est jamais exécuté** (voir ci-dessous).

> ⚠️ **Désynchro documentaire détectée (comparaison de documents, pas une mesure) :**
> `AGENTS.md §7` annonce encore « **38 fichiers sur disque / 38 enregistrés** » alors que le
> disque est mesuré à **39/39** en SE-050. À resynchroniser au prochain déploiement
> d'`AGENTS.md` à la racine de `A-Anime`, sous peine que Gemini raisonne sur un registre faux.

### Chaîne CI — mesurée en SE-050

`.github/workflows/ci.yml`, 18 lignes, déclenchée sur push `main` + pull request :

```
npm ci  →  npx vue-tsc --noEmit  →  npx vitest run  →  npm run build
```

**Ni Playwright, ni ESLint.** Conséquences :
- Les **39 specs E2E ne tournent jamais avant merge** — elles ne tournent qu'au sweep manuel du PO.
- `eslint.config.js` déclare `no-explicit-any` ; `package.json` ne contient **aucun script `lint`**
  (ESLint + 2 plugins sont en `devDependencies`, jamais invocables).
- **Les 17 pièges de faux-vert de `ANTIPATTERNS §6` ne sont appliqués par aucun automate.**

**Seul rouge connu :** `more-like-this-modal` — cause externe (endpoint Jikan `/recommendations`
en 504), **non-régression**. Dette suivie sous `US-E2E-MLT-MOCK`.

**Install :** `npm install` fonctionne en direct. `--legacy-peer-deps` n'est plus nécessaire
(downgrade `@pinia/testing` mergé). Ne le réarmer que si un futur `package.json` réintroduit
le conflit de peer-deps.

### 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> Ces trois métriques n'ont jamais été instrumentées depuis leur création. Elles ne
> deviendront mesurables qu'avec un sprint produit dédié. Définitions → `PILOTAGE.md §4`.

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S38** | ⏳ **Non répondue — sprint ouvert, 3 sessions sans code.** 🔴 Le garde-fou « pas plus d'1 sprint sans gain visible consécutif » impose que la prochaine livraison soit à **gain visible**, sans exception de dette |
| **S37** | ✅ **Gain ressenti.** Densité de grille uniforme (2 colonnes) sur This Season, Library/Upcoming, For You et Coming Soon. Fin de la rupture visuelle constatée par le PO sur 4 captures successives |
| S36 | ✅ Gain ressenti. Plus de scroll latéral sur téléphone étroit, popups centrées. Clôt l'exception de dette S35 |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : à remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.** Un endpoint testé avec d'autres paramètres est un autre endpoint.

### Jikan v4 — dernière mesure : SE-048 · ⚠️ **à remesurer à l'ouverture de SE-051**

**Diagnostic retenu (DEC-113) :** ni panne globale, ni paramètre de requête fautif.
**MyAnimeList est inaccessible depuis Jikan ; seules les URLs déjà présentes dans le cache de
Jikan répondent 200.** Toute URL neuve → 504.

| Fonction | État | Explication |
|---|---|---|
| Saisons (`/seasons/now`, `/seasons/upcoming`) | ✅ **fonctionnelles** | URLs fixes appelées en boucle → cache chaud |
| Recherche (`/anime?q=`) | ❌ **structurellement KO** | chaque titre tapé = URL neuve = cache miss = 504 |
| Détail anime (`/anime/{id}`) | ❌ KO sur ID non caché | impacte `syncAnimeUpdates` (**remplissage du champ `day`**) |
| Recommandations (`/anime/{id}/recommendations`) | ❌ KO | cause du rouge `more-like-this-modal` |

**Bug amont :** issue `jikan-me/jikan-rest` **#610**, ouverte le 10 juillet 2026, non résolue.
Reproduit indépendamment côté agent Plex (`Fribb/MyAnimeList.bundle#49`). Corroboré par la
documentation officielle Jikan (cache 24 h + rafraîchissement en tâche de fond).
**Aucun correctif possible côté Aanime.** Attendre n'est pas une option de pilotage : Jikan est
un scraper d'un tiers qui l'a bloqué.

**Auto-hébergement de Jikan — écarté (DEC-116).** L'image est publique et déployable, mais
(1) peupler sa propre base via l'API viole les CGU MyAnimeList, risque porté par nous ;
(2) on échangerait une dépendance cassée contre une dépendance fragile qu'on opère, avec coût
d'infra permanent et risque de bannissement IP.

### AniList — source retenue · **DEC-116** · vérifications datées du 7 août 2026

> **Décision : AniList seule, source unique, sans proxy, sans seconde API au lancement.**

| Critère | État |
|---|---|
| `idMal` natif | ✅ champ de première classe |
| Auth | ✅ aucune en lecture publique (OAuth2 seulement pour les données utilisateur) |
| CORS | ⚠️ **très probable, non prouvé** — `anilist.co` et `anichart.net` sont des origines différentes de `graphql.anilist.co` et fonctionnent. **Inférence solide, pas une preuve → test bloquant J-01** |
| Rate limit | ⚠️ **30 req/min actuellement** (mode dégradé documenté), 90 nominal + burst limiter |
| Coût | Gratuit, aucune clause commerciale bloquante |
| Broadcast | ✅ **meilleur que Jikan** : `airingAt` / `nextAiringEpisode` = timestamps Unix absolus par épisode, plus de parsing de chaîne JST |

**Pertes actées :** `themes` et `demographics` n'existent pas (→ mapping `tags` + `genres` à
écrire, pas un renommage) · score sur 100 · `members` MAL → `popularity` · corpus de
recommandations différent.

**Risque n°1, non mesuré :** le **taux d'`idMal` null sur le corpus utilisateur réel**. Aucune
source publique ne le donne. Seuils décidés : `< 2 %` on avance · `2–10 %` on avance avec une US
de rattrapage · `> 10 %` **on rouvre la décision**.

**Candidates écartées :** MAL v2 (proxy obligatoire + broadcast fragile conservé) · Kitsu (pas
d'heure de diffusion → le besoin n°4 meurt) · AnimeSchedule (excellente en planning, insuffisante
en catalogue, proxy obligatoire) · Simkl (CGU l'excluent comme source de catalogue) ·
Trakt / TMDB (**pas d'ID MAL** → critère éliminatoire n°1) · LiveChart, AniChart, Anime
Timetable, Notify.moe, Consumet (**pas d'API publique exploitable**).

**Alerte écosystème :** `manami-project/anime-offline-database` est **à l'arrêt** (dernière
release 2 avril 2026). Tous les services de mapping d'IDs en dérivent. **Ne pas bâtir la
migration sur un mapping externe.**

**Nuance assumée (DEC-116) :** on remplace une dépendance unique par une dépendance unique. Ce
n'est pas une architecture résiliente, c'est une architecture *moins mal placée* — AniList sert
sa propre base, Jikan scrapait celle d'un tiers. **Le mode dégradé local (snapshot IndexedDB +
bandeau d'état) n'est donc pas un bonus : c'est la contrepartie obligatoire du choix mono-source.**

---

## 📋 Kanban — sprint S38

### ✅ Done

- **[P0] Investigation cache Jikan local — CLOSE.** Cache `aanime_seasons_now` mesuré à 22,9 h
  pour un TTL de 24 h → **valide**. `readLocalCache` / `writeLocalCache` conformes. **Aucun
  bug.** Le « flag de désactivation du cache en dev » évoqué n'existe pas dans le code
  (`findstr import.meta.env DEV` → 0 hit) : confusion avec la case « Disable cache » du panneau
  Network de DevTools, qui ne concerne que le cache HTTP, jamais le localStorage.
- **[P0-bis] Comportement à l'expiration — CLOS.** `fetchCurrentSeason` sert délibérément le
  cache périmé si le fetch échoue (fidèle au vanilla). `error.value` est renseigné mais
  **jamais affiché**. Sans cache et sans réseau → liste vide silencieuse. Comportement
  conservé ; dette UX enregistrée (`US-CACHE-STALE-WARNING`, P2). → DEC-114
- **[P0-ter] Cause racine des 504 — CLOSE**, prouvée 9 mesures / 9. → DEC-113
- **[P0] `US-ONBOARDING-REFRESH` — investigation CLOSE en SE-050. Cause racine prouvée sur
  données réelles.** Voir « Trous fermés ». Chaîne de preuve `fichier:ligne`, conservée :
  - `OnboardingPage.vue:93` appelle bien `store.addAnime(buildSeedEntry(anime))`
  - `finishWithSeed` : `addAnime` → `markOnboarded` → `await saveToDatabase()` → toast → `router.push('/week')`
  - `CalendarWeekPage.vue:94` filtre sur `a.state === 'calendar' && a.day === dayClass`
  - `buildSeedEntry` (`onboardingFilter.ts:10`) retourne `{ ...anime, id, state }` — **ne pose jamais `day`**
  - `normalizeAnime` **ne produit jamais `day`** → DEC-115
  - `useSync.ts:121-130` est le **seul** producteur de `day`, sous le filtre `:68-79`
- **[P0] Dual audit S38 réconcilié.** 39 findings Claude Code + 6 Gemini → 30 constats uniques
  encore valides, reversés au backlog ci-dessous. → DEC-117
- **[P1] Vérification héritée n°1 (mapping `'Continuing'`) — CLOSE au POSITIF.**
- **[P0] Décision API tranchée.** → DEC-116

### 🔄 In Progress

*(vide — S38 doit livrer, plus rien ne doit s'ouvrir en investigation)*

### 📝 To Do — S38, à rédiger en SE-051

> **Contrainte de pilotage : ces trois items sont tous à gain visible. Zéro US de dette pure
> dans S38.**

1. **[AUD-01] Garde `day` centralisée** 🟠 — rendre impossible la création d'une entrée
   `state:'calendar'` sans `day`. Tue 4 producteurs d'un coup. **Migration-proof** : c'est une
   garde, pas un mapping — elle survit à la bascule AniList.
2. **[US-ONBOARDING-REFRESH]** 🟠 — Option A : `buildSeedEntry` n'écrit `calendar` que si le
   `day` est connu, sinon `watchlist` (visible immédiatement en Library). **Le libellé du toast
   doit changer.**
3. **[AUD-02] Sauvegarde Firestore silencieuse** 🔴 — `saveSchedule` rattrape son propre throw
   et retourne normalement ; l'app affiche « Saved » sur une panne. Seul P0 totalement
   orthogonal à la migration API.

### 🗂️ Backlog

#### Condition de lancement public

- **[US-ANILIST-J01]** ⬆️⬆️ 🔴 — **Spike bloquant, go/no-go de tout le lot.** Deux mesures,
  script jetable, zéro code applicatif : (1) CORS réel depuis l'origine de production —
  preflight **et** réponse, c'est le piège classique ; (2) taux d'`idMal` null sur le corpus
  utilisateur réel. **Rien d'autre ne démarre avant.**
- **[US-ANILIST-J02→J12]** 🔴 — lot de migration, phasé.
  *(Remplace et absorbe l'ancienne `US-ANILIST-SEARCH`, éclatée en J-01 → J-12 en SE-050.)*
  - *Phase 1 — coexistence (J-02 → J-08, J-12).* AniList devient la source de recherche et de
    détail — **les deux surfaces mortes, donc zéro régression possible**. Jikan reste sur les
    saisons. Feature flag. Le calendrier n'est pas touché : **c'est ce qui rend la phase 1
    réversible en un commit.**
  - *Phase 2 — bascule du calendrier (J-09, J-07).* `broadcast` → `airingAt` casse le contrat
    `AnimeEntry`. Exige **J-11 (versionnage des caches) livrée avant** — un bug ici efface des
    comptes. Test manuel sur ~20 titres en diffusion, dont ≥ 5 diffusés après minuit JST.
  - *Phase 3 — retrait de Jikan + mode dégradé.* C'est le livrable qui clôt l'incident des
    5 sprints.
- **[AUD-04] Coupe-circuit global** 🔴 — `lowPriorityFailures` / `circuitOpenTimestamp` sont des
  variables de module partagées : 3 échecs en priorité `low` coupent **tout** `low` pendant
  5 min, y compris `/seasons/now`, seul endpoint vivant. **Bloquant de J-02 : le plan prévoit de
  réutiliser le backoff existant, donc de porter le défaut dans le client AniList — sous
  30 req/min, une rafale de recherche couperait le calendrier.**
- **[AUD-08] CI sans E2E ni lint** 🟠 dette — **prérequis de J-04.** On s'apprête à réécrire le
  point de contrat unique (`normalizeAnime`, risque élevé) et à versionner les caches sans aucun
  filet automatisé.
- **[AUD-05] Mode dégradé — erreurs avalées** 🟠 — `fetchCurrentSeason` retourne `[]` sans
  rejeter : **tous les `catch` appelants sont morts.** 5 écrans concernés : Saison (« vous avez
  déjà tout ajouté » pendant une panne), Onboarding (grille vide muette), Library (page
  blanche), Discover (aucun état d'erreur), + `useSync` sans ref d'erreur. **C'est l'US « mode
  dégradé » du benchmark, désormais spécifiable.**

#### Onboarding & compte — hérités de SE-049, non planifiés S38

> Ces trois items étaient au « To Do S38/S39 » avant SE-050. Ils **ne disparaissent pas** : ils
> sont déprioritisés derrière la contrainte « S38 livre 3 US à gain visible ».

- **[US-ONBOARDING-i18n]** 🟠 — titres d'anime affichés en japonais / rōmaji pendant
  l'onboarding, incohérent avec le reste de l'app en anglais. ⚠️ **À réévaluer : AniList expose
  `title.english` nativement — la migration peut résorber ce défaut sans US dédiée.**
- **[US-ONBOARDING-LAYOUT]** 🟠 — cartes d'onboarding mal centrées (probablement famille
  `US-GRID-CENTRAL` — **à vérifier, ne pas supposer**).
- **[US-LOGOUT-RESET]** 🟠 — lien « vider mes données / repartir de zéro » dans
  `LogoutConfirmModal`, effaçant le compte Firestore + le cache local avant déconnexion.

#### Dette issue de l'audit S38 — à étaler, jamais à déverser

| Item | Risque | Impact utilisateur |
|---|---|---|
| **[AUD-03]** Import MAL → 100 % en `radar` (`my_status` lu mais jamais produit) + `episodeOverride` perdu | 🔴 | Un import de 300 titres met tout dans « Coming Soon », toast « Imported 300 animes ». Progression épisode remise à zéro |
| **[AUD-06]** `ToastNotification` monté seulement dans `AppLayout` → aucun toast sur `/login` ni `/welcome` | 🟠 | Les erreurs de l'onboarding ne s'affichent jamais |
| **[AUD-07]** Flag d'onboarding en localStorage seul, effacé au logout | 🟠 | Onboarding rejoué intégralement à chaque connexion / appareil, y compris avec 40 shows suivis |
| **[AUD-09]** Specs E2E qui n'assertent rien de significatif (toast seul ×2, `localStorage` au lieu du DOM, fixture `episodes:null` = le seul cas qui passe, test d'échec Firestore sur branche inatteignable) | 🟠 | aucun visible — dette. **Ce sont les filets des parcours cassés** |
| **[AUD-10]** `show:false` masque une entrée `calendar` valide (3 cas arithmétiques) sans message | 🟠 | Un anime présent, jour renseigné, disparaît de la semaine et du mois |
| **[AUD-11]** `EmptyState` reçoit `description` et `icon` non déclarées → fallthrough en attributs HTML | 🟠 | Coming Up, Plan to Watch et Discover n'affichent qu'un titre, sans la ligne d'explication |
| **[AUD-13]** Couches : `localStorage` piloté depuis `AppHeader.vue`, SDK Firebase appelé dans `LoginPage.vue`, `useSync` mute le store hors action Pinia | 🟢 | aucun visible — dette |
| **[AUD-14]** Typage : `any` à `helpers.ts:32` · 10 `as any` en tests · **la factory `makeAnime` est elle-même un cast** (`over as AnimeEntry`) · cast sur merge partiel | 🟢 | aucun visible — dette. Effet de bord : les tests ne peuvent pas détecter une entité incomplète, classe de bug d'AUD-01 |
| **[AUD-15]** `getCardStatus` : `status === null` retombe sur « Finished » | 🟢 | Un anime au statut inconnu s'affiche « Finished », pastille grise |
| **[AUD-16]** Navigation relations : `{mal_id, id, title} as AnimeEntry` | 🟢 | Modale sans jaquette ni score |
| **[AUD-17]** Stubs vides `_syncAnimeUpdates` / `_startBackgroundRelationFetch` encore invoqués | 🟢 | aucun visible — risque de double orchestration |
| **[AUD-18]** `useRecommendations.ts` (549 lignes) sans aucun test unitaire ; idem `useMalImport`, `useFirestore`, `stores/ui`, `useICS`, `useToast`, `useTheme`, `utils/idb` | 🟢 | aucun visible — **les 3 P0 non liés à `day` vivent tous dans des fichiers sans test** |

#### À vérifier avant conversion — ne pas rédiger d'US en l'état

| Item | Commande / mesure |
|---|---|
| **[AUD-12]** `localStorage.setItem` hors du `try` dans `saveToDatabase` → un quota dépassé bloquerait la fin de l'onboarding sans message | `findstr /n "^" src\composables\usePersistence.ts \| more +118` |
| **[AUD-19]** Famine de la file `low` par la file `high` (recherche) | `findstr /n "^" src\utils\helpers.ts \| more +50` |
| **[AUD-20]** Ligne `usePersistence.ts:200` citée par Gemini, non corroborée par Claude Code | `findstr /n "^" src\composables\usePersistence.ts \| more +194` |

#### Dette antérieure (inchangée)

- `[US-CACHE-STALE-WARNING]` 🟠 (P2) — avertir l'utilisateur quand les données affichées
  dépassent le TTL ou proviennent d'un fallback d'erreur. **À fusionner avec AUD-05.**
- `[US-GRID-CENTRAL]` 🟢 — migrer les 4 pages saines vers `.aa-card-grid`, supprimer les
  `.recs-grid` / `.grid` / `.plantowatch-grid` locales
- `[US-E2E-MLT-MOCK]` 🟠 — mock réseau de `more-like-this-modal`
- `[US-SEARCH-GUARD]` 🟠 — spec E2E sur les section headers de recherche
- `[US-140d]` 🟠 — toast de bienvenue à l'atterrissage de l'onboarding
- `[US-165]` 🟢 — extraire `fetchTopFinishedAnime` de `useRecommendations` vers `useJikanApi`.
  **⚠️ À réévaluer : la migration AniList réécrit ce code.**
- `[US-166-CSS]` 🟢 — dette CSS groupée (`.rc-mark-done`, `.test-*`, `.search-suggestion-added`
  `#10b981` en dur → `var(--airing)`, F18→F23)
- `[F8]` 🟢 — sous-nav et logo quasi illisibles en dark mode. Ouvert depuis la session 7,
  jamais traité
- `[US-JIKAN-HEALTHCHECK]` 🟠 (P1) — dev-only, détail par test au-delà d'un verdict global
  OK/KO. **⚠️ À réévaluer : devient `US-ANILIST-HEALTHCHECK`**
- `[F14]` 🟠 — skeletons au chargement (~6 s de blanc, `SkeletonCard` existe mais inutilisé)
- `[F15]` 🟢 — hiérarchie typographique Library/Upcoming, section vide

#### Produit

`login redesign` · `US-PWA` · `dual-titre rollout` (modale / RecCards / carte semaine) ·
`US-124` (mapping MAL `Dropped`) · `Cluster B découverte` · `Cluster C growth` · `STATS-5`

**Rendus possibles par AniList — lot 2, à ne PAS mélanger au lot 1 :** compte à rebours live
(`timeUntilAiring`) · liens de streaming officiels · bannière + couleur dominante par titre ·
port `ScheduleProvider` (donne l'optionnalité AnimeSchedule sans en payer le prix aujourd'hui).

---

## 🔍 Zones jamais auditées — reconnu en SE-050

Aucun des deux auditeurs ne les a ouvertes, et aucun document ne les couvre.

| Zone | Enjeu |
|---|---|
| **`firestore.rules`** | 🔴 **Le risque le plus sérieux et le seul jamais regardé.** Lancement public en approche, isolation de `schedules/{uid}` jamais vérifiée |
| **`utils/recEngine.ts`** | Scoring, `assignBadge`, `buildNextBatch` — le moteur de Discover et Library. Aucun audit, aucun test unitaire |
| **20 composants `.vue`** | `AnimeCard`, `RecCard`, `WeekAnimeItem`, `ModalVersionTop`, `MonthDayCell`, `SyncIndicator`… toute la couche de rendu des cartes |
| **Accessibilité** | ARIA, ordre de tabulation, piège de focus dans les modales `Teleport`, contraste |
| **Aucune observation runtime** | L'audit n'a exécuté ni `vue-tsc`, ni `vitest`, ni `playwright`, ni le build (`node_modules` absent du conteneur) |

---

## ✅ Trous fermés (ne pas réinvestiguer)

| Trou | Fermé en | Résultat |
|---|---|---|
| **`aanime_calendar` vide côté PO** | **SE-050** | **Fausse mesure.** Le localStorage contient bien `{calendar: 5, watchlist: 1}` |
| **Qui remplit le champ `day` ?** | **SE-050** | **`syncAnimeUpdates` (`useSync.ts:121-130`), et lui seul** — mais uniquement pour les animes retenus par le filtre `:68-79` |
| **Cause racine `US-ONBOARDING-REFRESH`** | **SE-050** | **Prouvée sur données réelles.** Les 5 entrées `calendar` ont toutes `day: undefined`, `airsTime: undefined`, `status: 'Currently Airing'` et un `episodes` numérique (14/12/12/12/13) → **aucune ne satisfait une seule clause du filtre de sync. La perte est définitive, pas un délai** — y compris avec une API saine |
| **Mapping du statut legacy `'Continuing'`** | **SE-050** | **EN PLACE.** `episodeInfo.ts:106` lowercase le statut, `:110` teste `'continuing'` → `Airing`. Le retrait de l'avertissement des 3 documents en SE-049 était **correct** |
| **La régénération documentaire a-t-elle périmé l'audit ?** | **SE-050** | **Non.** Les deux auditeurs ont lu le repo **code** `A-Anime`, jamais le corpus `Claude-V2`. Aucun finding avec `fichier:ligne` n'est obsolète |
| **Spec E2E non enregistrée dans un batch** | **SE-050** | **0 orpheline**, vérifié programmatiquement dans les deux sens |
| Compteur E2E périmé depuis la session 16 | S36/S37 | sweep complet batch1→5 |
| Divergence de compteur E2E 47/46 vs 46/45 | S38 | arbitré à **46 / 45** |
| `US-E2E-CONFIG` (Playwright local) | S36 | confirmée fonctionnelle |
| `US-E2E-BATCH-AUDIT` | — | absorbé par `US-E2E-REGISTRY-RESYNC` |
| Hash `main` non relevé | S36 | `444d385` — puis `c7cc60f` en SE-050 |
| US-140 / US-127 / US-SEARCH-3 « à faire » | S36 | les trois étaient déjà livrés |
| Nature réelle de la panne Jikan | S38 | DEC-113 |
| Existence d'un flag « cache désactivé en dev » | S38 | **n'existe pas** |
| Centrage des modales | S36 | DEC-107/108, cause racine externe aux modales |

## ❓ Trous restants

- **`AGENTS.md` déployé à la racine de `A-Anime` ?** À `c7cc60f` le fichier existe (164 lignes)
  mais `findstr "R-CODE-8"` est vide. ⚠️ **Ce grep était sensible à la casse — à refaire en
  `findstr /i "r-code-8"` avant de conclure.** Tant que ce n'est pas tranché, **Gemini lit
  peut-être une source périmée : aucune US ne part chez lui.**
- **`isBooting` libéré après l'étape 1 ou 2 du boot ?** (vérification héritée n°2, non traitée
  par l'audit). Lire le `onMounted` de `src\App.vue`. **Ne pas spécifier de correctif sur le
  boot avant d'avoir tranché.**
- **Les 5 lacunes du contrat de types** (`TYPES_CONTRACT.md §9`) : `AnimeEntry.synopsis?`,
  `useStats`, `useOnboarding`, `buildSeedEntry`, `getAnimeTitle` existent dans le code mais
  n'ont jamais été contractualisés. **C'est le trou par lequel Gemini invente un type.**
- **Sweep E2E non rejoué depuis S37** — 39 fichiers sur disque, le compteur 46/45 date de S37.
- **Registre `AGENTS.md §7` à 38 fichiers vs 39 mesurés** — à resynchroniser au prochain
  déploiement d'`AGENTS.md`.
- **Contenu de DEC-118** — le compteur « Dernier DEC » est à 118, mais seuls DEC-116 (décision
  API) et DEC-117 (réconciliation du dual audit) sont décrits dans cet état. **Ne pas inventer
  le contenu de DEC-118 : le relire dans `DECISIONS.md`.**
- **Détail par sprint S22→S27** — non capturé (historique, non bloquant)
- **DEC-75** — trou de lecture assumé, ne pas inventer son contenu
