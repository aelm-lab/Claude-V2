# STATE.md — Kanban vivant Aanime

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité UNIQUE de l'état courant (versions, métriques, Kanban).
> Ce fichier est la **source unique des compteurs** (tests / build / E2E) référencés par les
> autres docs Knowledge (`CLAUDE.md`, `ARCHITECTURE_TECHNIQUE.md`) — ne pas dupliquer ces
> chiffres ailleurs, y renvoyer.
> **Exception :** `AGENTS.md` / `AGENTS_E2E.md` (racine du dépôt, lus par Gemini qui n'a pas
> accès à ce fichier) portent leurs propres chiffres en dur, à resynchroniser à la main.
>
> **⚠️ RÉGÉNÉRATION COMPLÈTE — S38.** Ce fichier remplace intégralement la version
> précédente, qui était devenue un empilement de 3 blocs d'append aux compteurs
> contradictoires (S33 / S36 / S37). Les chiffres ci-dessous sont les seuls valides.

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.31.0** | **S37** | **US-GRID-FIX** (classe `.aa-card-grid` centralisée, 2 causes racines corrigées) · US-MODAL-UNIFY annulée après investigation (DEC-110). **Bump à acter.** |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT (clos, DEC-108) · US-SEARCH-3 (clos sans dev). Bump acté — résorbe le retard S30→S35. |
| v0.29.0 | S29→S33 | Nav refactor (header scroll iOS) · US-127 (SyncIndicator) · US-AUTH-LOGOUT. Bump non acté à l'époque. |
| v0.28.0 | S28 | Epic Stats : `useStats` + `StatsPage.vue` + route `/stats` + onglet Stats |
| — | S23→S27 | ⚠️ Détail non capturé (vit dans les handoffs archivés) |
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + `EPICS.md` consolidé |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| ≤ S16 | — | Migration vanilla → Vue 3 (Phases 0→7) · EPIC-1/2/3 clos · dual audit s16 |

---

## 🎯 Métriques de référence — fin S37, confirmées sur machine PO

| Métrique | Valeur | Preuve |
|---|---|---|
| Tests unit | **156 passed** (20 fichiers) | `npm run test:run`, machine PO, S37 |
| Type-check | **vert**, zéro `any` | `npm run type-check`, sortie vide |
| Build | **vite 6.4.2 · 178 modules** | `npm run build` |
| E2E — fichiers | **38 sur disque / 38 enregistrés** | sweep batch1→5, machine PO |
| E2E — tests | **46 tests, 45 verts** | idem |
| Batchs | batch1 → batch5 | batch4/5 créés en S36 |
| Commit `main` | `444d385` | précédents : `de64cac`, `82cd7ad`, `92d5a5a` |
| Dernier DEC réel | **DEC-114** | DEC-113/114 ajoutés en S38 |

> ✅ **Divergence de compteur E2E RÉSOLUE.** L'ancien fichier portait « 47 tests / 46 verts »
> (bloc S36) et « 46 tests / 45 verts » (bloc S37). **La valeur retenue est 46 / 45**, issue
> du sweep complet batch1→5 rejoué sur machine PO en ouverture S37 — mesure la plus récente
> et la plus complète. Le chiffre S36 était une projection non rejouée.

> Seul rouge : `more-like-this-modal` (cause externe Jikan, non-régression, dette US-E2E-MLT-MOCK).

> **`npm install` fonctionne en direct** — `--legacy-peer-deps` n'est plus nécessaire
> (downgrade `@pinia/testing` mergé). Ne le réarmer que si un futur `package.json`
> réintroduit le conflit de peer-deps.

### 📈 Métriques produit (PRODUCT_NORTHSTAR.md)
| Métrique | Valeur |
|---|---|
| TTFA (time to first add) | non instrumenté — baseline 0 |
| Adds / semaine | non instrumenté — baseline 0 |
| Jours de retour semaine 1 | non instrumenté — baseline 0 |

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S37** | ✅ **Gain ressenti.** Densité de grille uniforme (2 colonnes) sur This Season, Library/Upcoming, For You et Coming Soon. Fin de la rupture visuelle constatée par le PO sur 4 captures successives. |
| S36 | ✅ Gain ressenti. Plus de scroll latéral sur téléphone étroit, popups centrées. Clôt l'exception de dette S35. |

---

## 🌐 ÉTAT JIKAN — CORRECTION MAJEURE S38

> ❌ **Ancien libellé (porté à tort de S33 à S38) :** « Jikan en panne — 504, panne externe globale ».
> ✅ **Réalité mesurée (DEC-113, 9 mesures curl) :**
> **MyAnimeList est inaccessible depuis Jikan. Seules les URLs déjà présentes dans le cache
> de Jikan répondent 200. Toute URL neuve → 504.**

| Fonction | État réel | Explication |
|---|---|---|
| Saisons (`/seasons/now`, `/seasons/upcoming`) | ✅ **fonctionnelles** | URLs fixes appelées en boucle → cache chaud chez Jikan |
| Recherche (`/anime?q=`) | ❌ **structurellement KO** | chaque titre tapé = URL neuve = cache miss = 504 |
| Détail anime (`/anime/{id}`) | ❌ KO sur ID non caché | impacte `syncAnimeUpdates` (remplissage du `day`) |
| Recommandations (`/anime/{id}/recommendations`) | ❌ KO | cause du rouge `more-like-this-modal` |

**Bug amont :** issue GitHub `jikan-me/jikan-rest` **#610**, ouverte, non résolue.
Corroboré par la doc officielle Jikan (cache 24 h + rafraîchissement en tâche de fond).
**Aucun correctif possible côté Aanime.**

**Conséquences produit :**
- `US-ANILIST-SEARCH` passe de « alternative stratégique » à **condition de lancement**.
- `more-like-this-modal` et `US-NAV-A-FIX2` étaient gelés depuis S33 sur un diagnostic
  faux → **à retester**.

---

## 📋 Kanban — état S38 (session d'investigation, aucun code livré)

### ✅ Done — S38
- **[P0] Investigation cache Jikan local — CLOSE.** Cache `aanime_seasons_now` mesuré à
  22,9 h pour un TTL de 24 h → **valide**. `readLocalCache`/`writeLocalCache` conformes.
  **Aucun bug.** Le « flag de désactivation du cache en dev » évoqué n'existe pas dans le
  code (`findstr import.meta.env DEV` → 0 hit) : confusion avec la case « Disable cache »
  du panneau Network de DevTools, qui ne concerne que le cache HTTP, jamais le localStorage.
- **[P0-bis] Comportement à l'expiration — CLOS.** `fetchCurrentSeason` sert
  délibérément le cache périmé si le fetch échoue (fidèle au vanilla). `error.value` est
  renseigné mais **jamais affiché**. Sans cache et sans réseau → liste vide silencieuse.
  Comportement conservé ; dette UX enregistrée (US-CACHE-STALE-WARNING, P2). → DEC-114
- **[P0-ter] Cause racine des 504 — CLOSE, prouvée 9 mesures / 9.** → DEC-113

### 🔄 In Progress
- **[US-ONBOARDING-REFRESH] 🔴 — investigation à 90 %, non close.**
  - ✅ `OnboardingPage.vue:93` appelle bien `store.addAnime(buildSeedEntry(anime))`
  - ✅ `finishWithSeed` : `addAnime` → `markOnboarded` → `await saveToDatabase()` → toast → `router.push('/week')`
  - ✅ `CalendarWeekPage.vue:94` filtre sur `a.state === 'calendar' && a.day === dayClass`
  - ✅ `buildSeedEntry` (`onboardingFilter.ts:10`) retourne `{ ...anime, id, state }` — **ne pose jamais `day`**
  - ✅ `normalizeAnime` **ne produit jamais `day`** (`findstr day/airsTime/parseJSTToLocal/broadcast` → 0 hit)
  - ❓ **NON RÉSOLU :** le `aanime_calendar` du PO ne contient **aucune** entrée
    `state:'calendar'` (tableau vide). Trois lectures possibles, aucune tranchée :
    entrées présentes dans un autre `state` / mauvais navigateur interrogé / rien persisté.
  - ➡️ Commandes de reprise : voir handoff.

### 📝 To Do — S38/S39
1. **[US-ONBOARDING-REFRESH]** 🔴 — clore l'investigation puis spécifier
2. **[US-ONBOARDING-i18n]** — titres d'anime en japonais/rōmaji pendant l'onboarding, incohérent avec le reste de l'app en anglais
3. **[US-ONBOARDING-LAYOUT]** — cartes d'onboarding mal centrées (probablement famille US-GRID-CENTRAL — à vérifier, ne pas supposer)
4. **[US-LOGOUT-RESET]** — lien « vider mes données / repartir de zéro » dans `LogoutConfirmModal`, effaçant compte Firestore + cache local avant déconnexion

### 🗂️ Backlog

**Condition de lancement**
- **[US-ANILIST-SEARCH]** ⬆️⬆️ — seule voie de réparation de la recherche (DEC-113). AniList GraphQL sert sa propre base et expose `idMal` → migration possible sans casser les données existantes.

**Dette**
- [US-CACHE-STALE-WARNING] (P2, nouveau S38) — avertir l'utilisateur quand les données affichées dépassent le TTL ou proviennent d'un fallback d'erreur
- [US-GRID-CENTRAL] — migrer les 4 pages saines vers `.aa-card-grid`, supprimer `.recs-grid` / `.grid` / `.plantowatch-grid` locales
- [US-E2E-MLT-MOCK] — mock réseau de `more-like-this-modal`
- [US-SEARCH-GUARD] — spec E2E sur les section headers de recherche
- [US-140d] — toast de bienvenue onboarding

**Produit**
login redesign · US-PWA · dual-titre rollout · US-124 · US-JIKAN-HEALTHCHECK

---

## ✅ Trous connus FERMÉS

| Trou | Fermé en |
|---|---|
| Compteur E2E périmé depuis session 16 | S36/S37 — sweep complet batch1→5 |
| Divergence 47/46 vs 46/45 | **S38 — arbitré à 46 / 45** |
| US-E2E-CONFIG (Playwright local) | S36 — confirmée fonctionnelle |
| Hash `main` non relevé | S36 — `444d385` |
| US-140 / US-127 / US-SEARCH-3 « à faire » | S36 — les trois étaient livrés |
| US-E2E-BATCH-AUDIT | absorbé par US-E2E-REGISTRY-RESYNC |
| Nature réelle de la panne Jikan | **S38 — DEC-113** |
| Existence d'un flag « cache désactivé » | **S38 — n'existe pas** |

## ❓ Trous restants

- **`aanime_calendar` vide côté PO** — non expliqué, bloque la clôture d'US-ONBOARDING-REFRESH
- **Détail per-sprint S22→S27** — non capturé (historique, non bloquant)
- **DEC-75** — trou de lecture assumé, ne pas inventer
- **`day` : qui le remplit ?** — `normalizeAnime` ne le produit pas ; hypothèse `syncAnimeUpdates` via `parseJSTToLocal` **non vérifiée**

---

## 🗄️ Archivage

Dossier `OLD/` dans `aelm-lab/Claude-V2` (lecture seule, **ne jamais régénérer**) :
- `OLD/PLAN_MIGRATION.md` — migration Vue 3 terminée, pur historique
- `OLD/AUDIT_UX_SESSION7.md` — audit UX session 7 ; items ouverts rapatriés dans `EPICS.md`

*Supprimés définitivement (ne plus référencer) : `AUDIT.md`, `ROADMAP.md`, `BACKLOG.md`,
`HANDOFF_SESSION16.md`, `HANDOFF_SESSION5→10`, `PHASE8_DEBT.md`.*

---

## 🔁 Démarrage de session (rappel)

1. Lire la Knowledge → confirmer en 1 phrase + signaler si `STATE.md` semble périmé.
2. Afficher ce Kanban.
3. **Remesurer l'état Jikan** avec la requête EXACTE du code (AP-PROCESS, S38) — jamais une version simplifiée.
4. Questions de clarification s'il y a ambiguïté.
5. Présenter le PLAN (ordonné, impact user + reco).
6. Attendre le `go` explicite du PO avant tout passage à Gemini.
