# DECISIONS_ARCHIVE.md — Décisions closes

> **Rôle :** garder résolvables les renvois `DEC-xx` vers des décisions **closes, périmées ou de migration**. Aucune n'a de valeur opposable aujourd'hui.
> **Pas ici :** les décisions actives (→ `DECISIONS.md`).

**🔻 Hors ordre de lecture — chargement à la demande.** Comme `AUDIT.md`, `BENCHMARK.md` et `HISTORIQUE.md`, ce fichier n'est jamais lu à l'ouverture de session. On l'ouvre pour résoudre un renvoi, jamais pour décider.

**Append-only.** Une décision entre ici, elle n'en ressort pas. Aucun numéro n'est supprimé ni réattribué.

---

## 1. Scaffold & portage initial (DEC-01 → DEC-48)

Décisions de mise en place du projet, sans valeur opérationnelle aujourd'hui. Celles de cette plage encore appliquées ont été reprises dans `DECISIONS.md` sous leur numéro d'origine.

| ID | Décision | Sort |
|---|---|---|
| DEC-01 | `US-001b` absorbée dans `US-001` (dépassement de périmètre signalé et autorisé) | clos |
| DEC-04 | `npx <outil>` ≡ `npm run <script>` | ⛔ **SUPERSEDED — `AGENTS.md §2`** : seuls `npm run type-check` / `test:run` / `build` sont recevables, les scripts npm portent des options que `npx` masque |
| DEC-09 | Extension du contrat de types en **ajouts seuls**, rien renommé | clos |
| DEC-11 | Bug `item.studios` reproduit tel quel depuis l'implémentation d'origine | ✅ résolu par **DEC-86** |
| DEC-17 | Dette UX du boot : `LoadingOverlay` piloté par état réactif | ✅ résolu |
| DEC-20 | Le worker de fond retourne `[]` aussi bien sur « pas en cache » que « anime sans relations » | sans objet — worker supprimé (**DEC-147**) |
| DEC-21 | `buildRelationMemory` provient de `recs.js`, pas de `rec-engine.js` | clos — référence à l'implémentation d'origine |
| DEC-22 | `fetchTopFinishedAnime` inline dans `useRecommendations` | sans objet depuis `J11a` (`fetchTopFinishedWithMeta` vit dans `useAniListApi`) |
| DEC-25 | Stubs de sync no-op permanents dans `usePersistence` | ⛔ SUPERSEDED par **DEC-50** |
| DEC-29 | Placeholders inline dans `router/index.ts` | clos |
| DEC-30 | `lastCalendarView` reporté | clos |
| DEC-32 | Sync et worker de fond en fire-and-forget | ⛔ ajusté par **DEC-50**, puis worker supprimé (**DEC-147**) |
| DEC-39 | Substitution progressive des placeholders de router | clos |
| DEC-40 | Pattern stub `console.warn` puis câblage batché | clos |
| DEC-45 | Signatures `useEpisodeInfo` / `useICS` corrigées a posteriori | clos |
| DEC-46 | `CalendarNavControls` route-aware | ⛔ résorbé par **DEC-66** |
| DEC-48 | `ROADMAP.md` remplace `PHASE8_DEBT.md` | ⛔ obsolète — les deux documents ont été supprimés |

## 2. Correctifs ponctuels clos (DEC-49 → DEC-112)

| ID | Décision | Sort |
|---|---|---|
| DEC-49 | Origine de la règle **R3** (un audit lit le CODE) | clos — la règle vit dans `PILOTAGE.md` / `ANTIPATTERNS.md` |
| DEC-51 | Throttle conditionnel à 1,1 s **si `fromNetwork`** (pattern `*WithMeta`) | sans objet — worker supprimé (**DEC-147**), throttle manuel retiré (**DEC-141**). Le contrat `{ data, failed }` survit dans `TYPES_CONTRACT.md §0` |
| DEC-64 | Clé localStorage `'animeCalendar'` | ⛔ **SUPERSEDED PAR DEC-85** — la clé est `aanime_calendar`, jamais dans un nouveau seed |
| DEC-68 | Lot de style pur, tokens existants, script intact | clos |
| DEC-69 | Sous-lot sorti du périmètre P0 | clos |
| DEC-70 | `.modal-backdrop` : CSS manquant ajouté, règles préfixées | clos — la leçon générale vit dans `ANTIPATTERNS §5` |
| DEC-71 | Libellés de toast harmonisés | doublon strict de **DEC-63** |
| DEC-75 | ⚠️ **Non capturé.** Trou de lecture assumé entre DEC-74 et DEC-76 | **Ne pas inventer son contenu** |
| DEC-78 | Signalement de centrage clos comme perception d'audit, **spécifiquement sur `AnimeModal`** | clos — la vraie cause générale est **DEC-107**. Ne pas extrapoler |
| DEC-80bis | Clôture d'un epic : code-splitting, défer Firestore, fiabilité | clos |
| DEC-92 | Trois bugs fermés **sans spec** : invérifiables en production ou jamais reproductibles | clos — confirme qu'un ressenti d'audit se périme |
| DEC-101 | `.current-period` : taille et graisse réduites | clos, cosmétique |
| DEC-102 | `SyncIndicator` confirmé livré | clos |
| DEC-103 | US mergée malgré un volet non résolu — décision PO explicite | clos |
| DEC-105 | Healthcheck d'API : usage dev-only, verdict global OK/KO | sans objet — visait l'ancienne source de données |
| DEC-106 | Périmètre du centrage élargi à tous les popups | clos — résolu par **DEC-107** / **DEC-108** |
| DEC-108 | Audit de centrage clos **sans code dédié** : le CSS des modales était conforme depuis le début | clos |

## 3. Ancienne source de données (DEC-113 → DEC-125)

L'app ne consomme plus qu'AniList. Ces décisions portaient sur la source précédente ; elles sont conservées pour la traçabilité seulement.

| ID | Décision | Sort |
|---|---|---|
| DEC-113 | Diagnostic **cache HIT / MISS** de l'ancienne API (9 mesures curl) : le code HTTP ne dépendait d'aucun paramètre de requête mais de l'état du cache amont — URL en cache → 200, URL neuve → 504. La recherche était donc structurellement KO, les saisons restaient servies. Une formulation concurrente (« la panne venait de notre `order_by` ») a été **écartée**, falsifiée par les mesures | clos — plus aucune ligne du dépôt n'appelle cette API. **Leçon conservée** : `ANTIPATTERNS §7` (remesurer un fait externe) |
| DEC-115 | `day` n'est produit par aucun chemin de normalisation | ⛔ **SUPERSEDED PAR DEC-124.** La donnée était présente et simplement ignorée. *Leçon : une hypothèse marquée « NON close » se re-teste à sa clôture ; elle a été citée comme un fait dans trois documents* |
| DEC-116 · DEC-119 | ⚠️ **Non capturés dans le corpus.** Ne pas inventer leur contenu | — |
| DEC-117 | Cadre du dual audit — cité par `AUDIT.md` (campagne S38) | contenu non capturé ici |
| DEC-118 | Interdiction de corriger le mapping au motif qu'une US future le réécrirait | ⛔ **SUPERSEDED PAR DEC-124** — la prémisse (donnée de diffusion absente) était fausse. *Leçon : une décision « ne corrigez pas ici » doit citer la mesure qui la fonde, sinon elle survit à sa prémisse* |
| DEC-125 | Échéance de fermeture de l'ancienne API (octobre 2026), qui a transformé la réparation de la recherche en **full switch** de toutes les lectures externes | **neutralisée** — migration terminée, aucune ligne du dépôt ne l'appelle |
| DEC-130 | `US-MODAL-OPEN-SEED-KEY` **annulée — sans objet** : la migration de clés legacy au boot recopie l'ancienne clé avant lecture, la spec n'a jamais été rouge. **DEC-85 reste actif** : tout nouveau seed écrit sur `aanime_calendar` | clos — 1ʳᵉ US annulée dont la cause est une inférence de Claude inscrite en Knowledge |
| **DEC-181** | `US-CARD-CONVERGE-B` ne couvre que **Coming Soon**. *Completed* et *Plan to Watch* font l'objet d'une **US produit distincte, post-bêta** | Ces deux écrans affichent des animes **déjà en bibliothèque** : le bouton n'y est pas « Add » mais « Rewatch », « Start watching », ou rien. Ce n'est pas de la convergence technique, c'est un arbitrage produit que ni le PO ni moi ne pouvons trancher sans usage réel  | ⛔ SUPERSEDED DEC 183
## 4. Découpage de la migration AniList (DEC-134 → DEC-148)

Décisions de **découpage de travaux**, sans portée technique résiduelle. Les règles techniques issues de la migration (formes de requête, disjoncteur, normalisation) vivent dans `DECISIONS.md §3-§5`.

| ID | Décision | Sort |
|---|---|---|
| DEC-134 · DEC-136 | ⚠️ **Non capturés dans le corpus** (cités par `HISTORIQUE.md`, session SE-059). Ne pas inventer leur contenu | — |
| DEC-138 | `J10` éclatée en 5 slices (`J10a` → `J10e`) | clos |
| DEC-142 | `J11a` éclatée en 3 slices — une US unique aurait touché 4 fichiers dans un composable sans aucun test ; la slice additive a fourni le socle | clos |
| DEC-148 | `J11b` éclatée en 3 slices, dont une **dérogation au plafond de 3 fichiers** : une suppression atomique, couper en deux rendait le `type-check` rouge à l'état intermédiaire | clos |

---


| **DEC-151** | `AUD-05` (signal de fraîcheur visible) exige un **DEC d'arbitrage préalable** sur la source unique du signal `stale`, et passe 🟠 | Deux signaux `stale` concurrents et morts coexistent → violerait DEC-52 | DEC-151 est superseded par DEC-158


---

## Clôtures SE-075

| ID | Décision | Sort |
|---|---|---|
| **DEC-146** | `STATE.md` régénéré **intégralement à chaque session** + tous les autres documents patchés à chaque clôture de session, jamais batchés au sprint | ⛔ **SUPERSEDED PAR `DEC-190`.** Motif chiffré : sur SE-064 → SE-073, cette cadence a produit **2 366 lignes de patchs pour 31 US**, dont 79 % sur `STATE.md` et `ROADMAP.md` seuls. `STATE.md` fait 80 lignes et a été réécrit 15 fois son volume en 10 sessions. La création d'`HISTORIQUE.md` (append-only) par cette même décision est **conservée** et étendue par `DEC-190` (§Tampon) |
| **DEC-171** | `US-SEASON-1TAP` supprimée, absorbée par `US-CARD-CONVERGE-A` — les deux décrivaient le même livrable, `ROADMAP.md` leur allouait deux slots | ✅ **Clos** — `US-CARD-CONVERGE-A` livrée en SE-068. Décision de composition, sans valeur opposable une fois le sprint clos |
| **DEC-182** | `S42` clos en `v0.36.0`, 10/10 slots, sur réponse « gain de fiabilité visible » à la Sprint Outcome Gate ; `US-HEADER-ICONS` non prise glisse en S43 | ✅ **Clos** — trace conservée dans `HISTORIQUE §3 Versions`. Une clôture de sprint est un fait historique, pas une décision active |

> 🔻 **`DEC-135` n'existe dans aucun des deux fichiers.** Numéro attribué entre `DEC-134` et
> `DEC-136` (SE-059, planning S40) mais jamais écrit. Constaté en SE-075. **Ne pas réattribuer.**

---

## Leçons extraites — vivent ailleurs, ne pas les rejouer ici

| Leçon | Document propriétaire |
|---|---|
| Remesurer un fait externe avec la **requête exacte du code** | `PILOTAGE.md §6` · `ANTIPATTERNS §7` |
| Un handoff est une source secondaire faillible | `DECISIONS.md` DEC-87 · `ANTIPATTERNS §7` |
| Le symptôme apparaît parfois très loin de sa cause | `DECISIONS.md` DEC-107 · `ANTIPATTERNS §5` |
| Une US sortant du backlog démarre par un grep | `PILOTAGE.md §6` |
