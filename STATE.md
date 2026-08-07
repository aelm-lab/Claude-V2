# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état courant — sprint, session, versions,
> compteurs, Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient,
> ils ne dupliquent jamais une valeur. *Exception unique : `AGENTS.md`, lu par Gemini qui n'a
> pas accès à ce fichier — ses chiffres sont en dur et resynchronisés à chaque sprint.*
>
> **Ce qui n'est PAS ici :** les règles (→ `PILOTAGE.md`, `AGENTS.md`) · le pourquoi des choix
> (→ `DECISIONS.md`) · l'acquis fonctionnel (→ `EPICS.md`) · **le contenu des constats d'audit
> (→ `AUDIT.md`)**.
>
> 🔗 **`AUDIT.md` — doc satellite, chargé à la demande.** Il n'est pas dans l'ordre de lecture
> de `CLAUDE.md` et ne se lit qu'au moment de planifier un sprint ou de convertir un constat
> en US. Ce fichier cite les identifiants `AUD-xx` et décide de leur sprint ; `AUDIT.md`
> décrit ce qu'ils sont. **C'est le seul lien entre les deux documents, et aucun autre doc du
> corpus ne référence `AUDIT.md`.**

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
| **SE-050** | S38 | Dual audit S38 réconcilié · décision API tranchée · 13 mesures | Handoff `SE-050 → SE-051`. **0 code livré** |
| SE-049 | — (hors sprint) | Refonte complète du corpus documentaire | Corpus 13 → 9 docs |
| SE-048 *(ex-« S38 »)* | S38 | Investigation Jikan + cache local + onboarding | 3 investigations P0 closes, 0 code livré |
| SE-047 *(ex-« S37 »)* | S37 | US-GRID-FIX, annulation US-MODAL-UNIFY | v0.31.0 |
| SE-046 *(ex-« S36 »)* | S36 | Resync registre E2E, overflow horizontal, centrage popups | v0.30.0 |

> Les sessions antérieures à SE-046 n'ont pas été numérotées séparément (le compteur `SXX`
> servait aux deux axes jusqu'en SE-049). Leur détail vit dans les handoffs archivés.

> ⚠️ **Trois sessions consécutives sans code livré** (SE-048, SE-049, SE-050). Le garde-fou
> `PILOTAGE §5` est en tension maximale : **SE-051 livre, ou S38 est un échec de pilotage, pas
> un échec technique.**

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.31.0** | **S37** | US-GRID-FIX (classe `.aa-card-grid` centralisée, 2 causes racines) · US-MODAL-UNIFY annulée après investigation (DEC-110) |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT (clos, DEC-108) · US-SEARCH-3 (clos sans dev). Résorbe le retard de bumps S30→S35 |
| v0.29.0 | S29→S33 | Refonte nav (header scroll iOS) · US-127 (SyncIndicator) · US-AUTH-LOGOUT |
| v0.28.0 | S28 | Epic Stats : `useStats` + `StatsPage.vue` + route `/stats` + onglet Stats |
| — | S23→S27 | ⚠️ Détail non capturé (vivait dans les handoffs archivés) |
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + consolidation `EPICS.md` |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| ≤ S16 | — | Migration vanilla → Vue 3 (Phases 0→7) · EPIC-1/2/3 clos · dual audit s16 |

> **Le numéro de version ne suit plus le numéro de sprint** depuis S29 (`PILOTAGE.md §1`).
> Cette table est la seule correspondance valide.

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **156 passed** (20 fichiers) | `npm run test:run`, machine PO | fin S37 |
| Type-check | **vert** | `npm run type-check`, sortie vide | fin S37 |
| Build | **vite 6.4.2 · 178 modules** | `npm run build` | fin S37 |
| E2E — fichiers sur disque | **39** | `dir /b tests\e2e\*.spec.ts \| find /c ".spec"` | **SE-050** |
| E2E — registre `package.json` | **39 / 39, 0 orpheline** | diff programmatique batches 1→5 | **SE-050** |
| E2E — registre `AGENTS.md §7` | ⚠️ **38 — diverge de 1** | comptage des 5 batchs (9+9+7+9+4) | **SE-051** |
| E2E — tests | 46 tests, 45 verts | sweep machine PO | fin S37, **à rejouer** |
| Batchs | batch1 → batch5 | batch4/5 créés en S36 | — |

> ⚠️ **La mention « zéro `any` » a été retirée en SE-050.** C'était une inférence de `vue-tsc`,
> qui ne teste pas les `any`. Mesure réelle : `helpers.ts:32`. Seul ESLint le signalerait, et
> **ESLint n'est jamais exécuté**.

> 🔴 **Deux registres E2E divergents — découvert en SE-051.** Celui qui exécute
> (`package.json`, 39) et celui que Gemini lit (`AGENTS.md §7`, 38). **Une spec tourne au sweep
> sans que Gemini sache qu'elle existe.** Combiné au piège de regex de sous-chaîne documenté
> dans `AGENTS §7`, c'est exactement le scénario de duplication silencieuse que ce registre est
> censé empêcher. **À résoudre avant tout envoi d'US touchant l'E2E.**

### Chaîne CI — mesurée en SE-050

`.github/workflows/ci.yml`, 18 lignes, sur push `main` + pull request :

```
npm ci  →  npx vue-tsc --noEmit  →  npx vitest run  →  npm run build
```

**Ni Playwright, ni ESLint.** `eslint.config.js` déclare `no-explicit-any` mais `package.json`
n'a **aucun script `lint`**. Conséquences détaillées → `AUDIT.md`, constat `AUD-08`.

**Seul rouge connu :** `more-like-this-modal` — cause externe (Jikan `/recommendations` en 504),
**non-régression**. Dette suivie sous `US-E2E-MLT-MOCK`.

**Install :** `npm install` fonctionne en direct. `--legacy-peer-deps` n'est plus nécessaire
(downgrade `@pinia/testing` mergé). Ne le réarmer que si un futur `package.json` réintroduit le
conflit de peer-deps.

### 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> Jamais instrumentées depuis leur création. Elles ne deviendront mesurables qu'avec un sprint
> produit dédié. Définitions → `PILOTAGE.md §4`.

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
| Détail anime (`/anime/{id}`) | ❌ KO sur ID non caché | impacte `syncAnimeUpdates` (remplissage du champ `day`) |
| Recommandations (`/anime/{id}/recommendations`) | ❌ KO | cause du rouge `more-like-this-modal` |

**Bug amont :** `jikan-me/jikan-rest` **#610**, ouverte le 10 juillet 2026, non résolue.
Reproduit indépendamment côté agent Plex (`Fribb/MyAnimeList.bundle#49`). Corroboré par la doc
officielle Jikan (cache 24 h + rafraîchissement en tâche de fond).
**Aucun correctif possible côté Aanime.** Auto-hébergement écarté → DEC-116.

### AniList — source retenue · **DEC-116** · vérifié le 7 août 2026

> **Décision : AniList seule, source unique, sans proxy, sans seconde API au lancement.**
> Candidates écartées, pertes fonctionnelles et rationale complète → **`DECISIONS.md`, DEC-116**.
> Ne pas les recopier ici.

| Critère mesuré | État |
|---|---|
| `idMal` natif | ✅ champ de première classe |
| Auth | ✅ aucune en lecture publique (OAuth2 seulement pour les données utilisateur) |
| CORS | ⚠️ **très probable, NON PROUVÉ** — `anilist.co` et `anichart.net` sont des origines différentes de `graphql.anilist.co` et fonctionnent. Inférence solide, pas une preuve → **test bloquant J-01** |
| Rate limit | ⚠️ **30 req/min actuellement** (mode dégradé documenté), 90 nominal + burst limiter |
| Coût | Gratuit, aucune clause commerciale bloquante |
| Broadcast | ✅ **meilleur que Jikan** : `airingAt` / `nextAiringEpisode` = timestamps Unix absolus par épisode, plus de parsing de chaîne JST |

**Risque n°1, non mesuré :** le **taux d'`idMal` null sur le corpus utilisateur réel**. Aucune
source publique. Seuils décidés : `< 2 %` on avance · `2–10 %` on avance avec une US de
rattrapage · `> 10 %` **on rouvre la décision**.

**Alerte écosystème :** `manami-project/anime-offline-database` est **à l'arrêt** (dernière
release 2 avril 2026). Tous les services de mapping d'IDs en dérivent. **Ne pas bâtir la
migration sur un mapping externe.**

**Conséquence structurante de DEC-116 :** mono-source assumé → **le mode dégradé local
(snapshot IndexedDB + bandeau d'état) n'est pas un bonus, c'est la contrepartie obligatoire.**

---

## 📋 Kanban — sprint S38

### ✅ Done

- **[P0] Cache Jikan local** — CLOSE. Aucun bug. → *Trous fermés*
- **[P0-bis] Comportement à l'expiration** — CLOS. → DEC-114
- **[P0-ter] Cause racine des 504** — CLOSE, 9 mesures / 9. → DEC-113
- **[P0] `US-ONBOARDING-REFRESH`** — investigation CLOSE, cause racine prouvée sur données
  réelles. → *Trous fermés*
- **[P0] Dual audit S38 réconcilié** — 30 constats uniques valides. → DEC-117 · contenu dans
  `AUDIT.md`
- **[P1] Vérification héritée n°1** (mapping `'Continuing'`) — CLOSE au POSITIF
- **[P0] Décision API tranchée** — → DEC-116
- **[P0] `AGENTS.md` déployé à la racine de `A-Anime`** — CLOSE en SE-051. → *Trous fermés*

### 🔄 In Progress

*(vide — S38 doit livrer, plus rien ne doit s'ouvrir en investigation)*

### 📝 To Do — S38, à rédiger en SE-051

> **Contrainte de pilotage : ces trois items sont tous à gain visible. Zéro US de dette pure
> dans S38.** Contenu des constats → `AUDIT.md`.

1. **[AUD-01] Garde `day` centralisée** 🟠 — tue 13 producteurs d'un coup. **Migration-proof**
   (c'est une garde, pas un mapping — DEC-118).
2. **[US-ONBOARDING-REFRESH]** 🟠 — Option A : `buildSeedEntry` n'écrit `calendar` que si le
   `day` est connu, sinon `watchlist` (visible immédiatement en Library). **Le libellé du toast
   doit changer.** Périmètre pressenti : `src/utils/onboardingFilter.ts`. Spec E2E fournie
   verbatim par Claude (R7), assertion sur le **DOM visible mobile sans rechargement**.
3. **[AUD-02] Sauvegarde Firestore silencieuse** 🔴 — seul P0 totalement orthogonal à la
   migration API.

### 🗂️ Backlog

#### Condition de lancement public

- **[US-ANILIST-J01]** ⬆️⬆️ 🔴 — **Spike bloquant, go/no-go de tout le lot.** Deux mesures,
  script jetable, zéro code applicatif : (1) CORS réel depuis l'origine de production —
  preflight **et** réponse, c'est le piège classique ; (2) taux d'`idMal` null sur le corpus
  utilisateur réel. **Rien d'autre ne démarre avant. Planification non tranchée — décision PO
  attendue.**
- **[US-ANILIST-J02→J12]** 🔴 — lot de migration, phasé.
  *(Remplace et absorbe `US-ANILIST-SEARCH`, éclatée en J-01 → J-12 en SE-050.)*
  - *Phase 1 — coexistence (J-02 → J-08, J-12).* AniList devient la source de recherche et de
    détail — **les deux surfaces mortes, donc zéro régression possible**. Jikan reste sur les
    saisons. Feature flag. Le calendrier n'est pas touché : **c'est ce qui rend la phase 1
    réversible en un commit.**
  - *Phase 2 — bascule du calendrier (J-09, J-07).* `broadcast` → `airingAt` casse le contrat
    `AnimeEntry`. Exige **J-11 (versionnage des caches) livrée avant** — un bug ici efface des
    comptes. Test manuel sur ~20 titres en diffusion, dont ≥ 5 diffusés après minuit JST.
  - *Phase 3 — retrait de Jikan + mode dégradé.* Le livrable qui clôt l'incident des 5 sprints.

#### Séquençage imposé par l'audit — descriptions dans `AUDIT.md`

| Constat | Contrainte de séquence |
|---|---|
| **[AUD-04]** 🔴 | **Bloquant de J-02.** À corriger *dans* J-02, pas après |
| **[AUD-08]** 🟠 | **Prérequis de J-04.** Seule US de dette pure imposée avant la phase 1 |
| **[AUD-05]** 🟠 | Spécifie le « mode dégradé » du benchmark. **À fusionner avec `US-CACHE-STALE-WARNING`** |

#### Onboarding & compte — hérités de SE-049, non planifiés

> Déprioritisés derrière la contrainte « S38 livre 3 US à gain visible ». **Non abandonnés.**

- **[US-ONBOARDING-i18n]** 🟠 — titres d'anime en japonais / rōmaji pendant l'onboarding,
  incohérent avec le reste de l'app en anglais. ⚠️ **À réévaluer : AniList expose
  `title.english` nativement — la migration peut résorber ce défaut sans US dédiée.**
- **[US-ONBOARDING-LAYOUT]** 🟠 — cartes d'onboarding mal centrées (probablement famille
  `US-GRID-CENTRAL` — **à vérifier, ne pas supposer**).
- **[US-LOGOUT-RESET]** 🟠 — lien « vider mes données / repartir de zéro » dans
  `LogoutConfirmModal`, effaçant le compte Firestore + le cache local avant déconnexion.

#### Dette antérieure — hors audit

- `[US-CACHE-STALE-WARNING]` 🟠 — avertir quand les données dépassent le TTL ou viennent d'un
  fallback d'erreur. **À fusionner avec AUD-05.**
- `[US-GRID-CENTRAL]` 🟢 — migrer les 4 pages saines vers `.aa-card-grid`, supprimer les
  `.recs-grid` / `.grid` / `.plantowatch-grid` locales
- `[US-E2E-MLT-MOCK]` 🟠 — mock réseau de `more-like-this-modal`
- `[US-SEARCH-GUARD]` 🟠 — spec E2E sur les section headers de recherche
- `[US-140d]` 🟠 — toast de bienvenue à l'atterrissage de l'onboarding
- `[US-165]` 🟢 — extraire `fetchTopFinishedAnime` de `useRecommendations` vers `useJikanApi`.
  **⚠️ À réévaluer : la migration AniList réécrit ce code.**
- `[US-166-CSS]` 🟢 — dette CSS groupée (`.rc-mark-done`, `.test-*`, `.search-suggestion-added`
  `#10b981` en dur → `var(--airing)`, F18→F23)
- `[F8]` 🟢 — sous-nav et logo quasi illisibles en dark mode. Ouvert depuis la session 7
- `[US-JIKAN-HEALTHCHECK]` 🟠 — dev-only, détail par test au-delà d'un verdict global.
  **⚠️ Devient `US-ANILIST-HEALTHCHECK`**
- `[F14]` 🟠 — skeletons au chargement (~6 s de blanc, `SkeletonCard` existe mais inutilisé)
- `[F15]` 🟢 — hiérarchie typographique Library/Upcoming, section vide

#### Produit

`login redesign` · `US-PWA` · `dual-titre rollout` (modale / RecCards / carte semaine) ·
`US-124` (mapping MAL `Dropped`) · `Cluster B découverte` · `Cluster C growth` · `STATS-5`

**Rendus possibles par AniList — lot 2, à ne PAS mélanger au lot 1 :** compte à rebours live
(`timeUntilAiring`) · liens de streaming officiels · bannière + couleur dominante par titre ·
port `ScheduleProvider` (donne l'optionnalité AnimeSchedule sans en payer le prix aujourd'hui).

#### 🔴 Zone jamais ouverte, hors audit

**`firestore.rules`** — lancement public en approche, isolation de `schedules/{uid}` jamais
vérifiée par personne. Ce n'est pas un finding d'audit : c'est une zone que personne n'a
regardée. Autres zones aveugles → `AUDIT.md §Zones jamais auditées`.

---

## ✅ Trous fermés (ne pas réinvestiguer)

| Trou | Fermé en | Résultat |
|---|---|---|
| **`AGENTS.md` déployé à la racine de `A-Anime` ?** | **SE-051** | **OUI.** Version régénérée SE-049 (350 lignes), `R-CODE-8` présent §4. Le `findstr` vide portait sur `git show c7cc60f:AGENTS.md`, où le fichier faisait encore 164 lignes — **le déploiement est postérieur à `c7cc60f`** |
| **`aanime_calendar` vide côté PO** | **SE-050** | **Fausse mesure.** Le localStorage contient bien `{calendar: 5, watchlist: 1}` |
| **Qui remplit le champ `day` ?** | **SE-050** | **`syncAnimeUpdates` (`useSync.ts:121-130`), et lui seul** — uniquement pour les animes retenus par le filtre `:68-79` |
| **Cause racine `US-ONBOARDING-REFRESH`** | **SE-050** | **Prouvée sur données réelles.** Les 5 entrées `calendar` ont toutes `day: undefined`, `airsTime: undefined`, `status: 'Currently Airing'` et un `episodes` numérique (14/12/12/12/13) → **aucune ne satisfait une seule clause du filtre de sync. La perte est définitive, pas un délai** — y compris avec une API saine. Détail → `AUDIT.md`, AUD-01 |
| **Mapping du statut legacy `'Continuing'`** | **SE-050** | **EN PLACE.** `episodeInfo.ts:106` lowercase, `:110` teste `'continuing'` → `Airing`. Le retrait de l'avertissement en SE-049 était **correct** |
| **La régénération documentaire a-t-elle périmé l'audit ?** | **SE-050** | **Non.** Les deux auditeurs ont lu le repo **code** `A-Anime`, jamais le corpus `Claude-V2`. Aucun finding `fichier:ligne` n'est obsolète |
| **Spec E2E orpheline dans `package.json`** | **SE-050** | **0 orpheline**, vérifié dans les deux sens. *(La divergence avec `AGENTS.md §7` est un trou distinct, encore ouvert)* |
| **Cache Jikan local / flag « cache off en dev »** | **SE-048** | Cache `aanime_seasons_now` mesuré à 22,9 h pour un TTL de 24 h → **valide**. Le flag **n'existe pas** (`findstr import.meta.env DEV` → 0 hit) : confusion avec la case « Disable cache » de DevTools, qui ne touche que le cache HTTP |
| **Comportement à l'expiration du cache** | **SE-048** | `fetchCurrentSeason` sert délibérément le cache périmé si le fetch échoue. `error.value` renseigné mais **jamais affiché**. Comportement conservé, dette enregistrée. → DEC-114 |
| Nature réelle de la panne Jikan | S38 | DEC-113 |
| Compteur E2E périmé depuis la session 16 | S36/S37 | sweep complet batch1→5 |
| Divergence de compteur E2E 47/46 vs 46/45 | S38 | arbitré à **46 / 45** |
| `US-E2E-CONFIG` (Playwright local) · `US-E2E-BATCH-AUDIT` | S36 | confirmée fonctionnelle · absorbé par `US-E2E-REGISTRY-RESYNC` |
| US-140 / US-127 / US-SEARCH-3 « à faire » | S36 | les trois étaient déjà livrés |
| Centrage des modales | S36 | DEC-107/108, cause racine externe aux modales |

## ❓ Trous restants

- 🔴 **Quelle est la 39ᵉ spec E2E ?** `AGENTS.md §7` en enregistre 38 (9+9+7+9+4), le disque en
  porte 39. Une spec tourne au sweep sans que Gemini sache qu'elle existe. **Mesure de levée :**
  `dir /b tests\e2e\*.spec.ts`, puis diff contre le §7. **À corriger dans `AGENTS.md` avant tout
  envoi d'US touchant l'E2E.**
- **`isBooting` libéré après l'étape 1 ou 2 du boot ?** (vérification héritée n°2, non traitée
  par l'audit). Lire le `onMounted` de `src\App.vue`. **Ne pas spécifier de correctif sur le
  boot avant d'avoir tranché.**
- **Les 5 lacunes du contrat de types** (`TYPES_CONTRACT.md §9`) : `AnimeEntry.synopsis?`,
  `useStats`, `useOnboarding`, `buildSeedEntry`, `getAnimeTitle` existent dans le code mais
  n'ont jamais été contractualisés. **C'est le trou par lequel Gemini invente un type** — et
  `buildSeedEntry` est dans le périmètre de l'US n°2 de S38.
- **Sweep E2E non rejoué depuis S37** — le compteur 46/45 date de S37.
- **Contenu de DEC-118** — décrit dans le handoff SE-050 (non-correction de `normalizeAnime`),
  à vérifier présent dans `DECISIONS.md`. **Ne pas l'inventer, le relire.**
- **Détail par sprint S22→S27** — non capturé (historique, non bloquant)
- **DEC-75** — trou de lecture assumé, ne pas inventer son contenu
