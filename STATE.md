# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état **courant** — sprint, session, compteurs,
> Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient.
> *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à ce fichier.*
>
> **DEC-146 — Ce fichier ne porte plus l'historique.** Sessions closes et versions livrées
> vivent dans `HISTORIQUE.md` (hors ordre de lecture, append-only). Ici : le présent
> uniquement. Régénération **intégrale** à chaque clôture de session, jamais de patch.

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint** | **S40 — EN COURS.** Goal : *Jikan est débranché avant sa fermeture* |
| **Sprint Goal** | ✅ **ATTEINT** (`91b78eb`) — sprint **non clos**, sweep E2E rouge |
| **Session courante** | **SE-062** |
| **Dernière version livrée** | **v0.33.0 (S39)** — S40 non clos, aucun bump |
| **Dernier DEC** | **DEC-149** |
| **Commit `main`** | `91b78eb` — arbre propre |
| **Échéance dure** | Jikan ferme en octobre 2026 (DEC-125) — **neutralisée** |
| **Cadence** | 4 à 8 US par sprint |

**Ce qui bloque la clôture :** 12 specs E2E rouges, cause racine unique (mocks Jikan périmés).
→ epic `J12`, à traiter en SE-063 avant le Sprint Outcome Gate.

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **265 passed** (29 fichiers) | `npm run test:run`, machine PO | **SE-062** |
| Durée suite unitaire | **4,13 s** (était 4,34 s à 281 tests) | idem | **SE-062** |
| Type-check | **vert** | sortie vide | **SE-062** |
| Build | **vert** | `180 modules transformed` | **SE-062** |
| E2E — fichiers sur disque | **42** | inchangé depuis SE-056 | SE-056 |
| E2E — registre `package.json` | **42 / 42** | 5 batches : 9·9·8·9·7 | SE-056 |
| **E2E — sweep complet** | **40 verts / 52 · 12 rouges** | 5 batches séparés, machine PO | **SE-062** |

> ⚠️ Pas de mention « zéro `any` » : `vue-tsc` ne les teste pas, ESLint n'est jamais exécuté.

### Détail des 12 rouges E2E — **une seule cause racine**

Les 11 specs concernées installent un `page.route()` sur `api.jikan.moe/v4/…`.
L'app appelle désormais `graphql.anilist.co`. Le mock ne mord plus, le réseau réel passe.
**Aucune n'est une régression de code** — l'app est fonctionnelle (constat visuel PO, SE-062).

| Batch | Spec | Signature |
|---|---|---|
| 1 | `modal-add-appears-on-week` | A — `.card-cp-container` introuvable |
| 1 | `modal-add-feedback` | A |
| 1 | `modal-add-removes-from-discover` (2 cas) | A |
| 2 | `reccard-add` | A |
| 2 | `reccard-click-dismiss` | A |
| 3 | `toast-visible-mobile` | A |
| 4 | `onboarding-fullscreen` | B — `.suggestion-card` : 8 au lieu de 3 |
| 4 | `onboarding-seed` | B |
| 4 | `onboarding-toast` | B — **ex-AUD-23, désormais diagnostiqué** |
| 5 | `day-guard-plan-to-watch` | B |
| 5 | `onboarding-toast-destination` | B |

**Signature A** = la fixture n'arrive jamais → écran vide.
**Signature B** = du contenu réel s'affiche à la place des fixtures (8 = défaut d'onboarding).

### Rouges historiques — soldés en SE-062

| Spec | État | Cause |
|---|---|---|
| `more-like-this-modal` | ✅ **VERT** | ne tape plus Jikan, plus d'appel réseau |
| `discover-season-dedup` | ✅ **VERT** | **AUD-22 clos** par la migration AniList, sans US |
| `onboarding-toast` | 🔴 rouge | **AUD-23 diagnostiqué** : cause commune `J12` |

---

## 📋 Kanban — S40 · SE-062

### ✅ Done — sprint S40

| US | Risque | Commit | Objet |
|---|---|---|---|
| `J09a` | 🟠 | `4403bd5` | Season AniList |
| `J09b` | 🟠 | `31ec1b4` | Season AniList (suite) |
| `US-E2E-SEASON-ANILIST` | — | — | Spec season |
| `J10a` | 🔴 | `0ed68f6` | `fetchRelations` |
| `J10b` | 🔴 | `65af6eb` | Relations sur AniList |
| `J10c` | 🔴 | — | Bande relations (504 silencieux corrigé) |
| `J10d` | 🔴 | `169f48d` | `syncAnimeUpdates` sur AniList, liste close de 9 champs |
| *micro-patch* | — | `9714df9` | JSDoc `syncAnimeUpdates` |
| `J11a-1` | 🟠 | `3b87dd6` | `fetchUpcomingSeasonWithMeta` · `fetchTopFinishedWithMeta` · `resolveNextSeason` |
| `J11a-2` | 🔴 | `8234404` | `fetchRecPool` sur AniList (2 appels au lieu de 3) |
| *micro-patch* | — | `d5e12a9` | mock `assignBadge` cloné |
| `J11a-3` | 🔴 | `70a7b91` | `getSeasonNudges` sur relations AniList |
| **`J11b-1`** | 🔴 | **`2a24b3c`** | Suppression du worker de relations en fond |
| **`J11b-2`** | 🟢 | **`90dd38a`** | Suppression `useJikanApi.ts` + spec |
| **`J11b-3`** | 🟢 | **`91b78eb`** | Purge `helpers.ts` — **dernière ligne de Jikan** |

**Bilan Gemini S40 : 0 correction majeure, merge au premier coup sur toutes les US.**

### 🔄 In Progress
Aucune. Session SE-062 close sur la capacité.

### 📝 To Do — clôture S40 (SE-063)

| Ordre | Item | Risque | Note |
|---|---|---|---|
| 1 | **`J12`** — migration du harnais E2E vers AniList | 🟠 | Nouvel epic. Helper mutualisé, 11 specs. **Gemini non impliqué** |
| 2 | `AUD-05` — signal de fraîcheur visible | 🟠 | Requalifiée 🟢→🟠 (élément d'écran → R4). DEC préalable requis |
| 3 | Re-sweep E2E — 5 batches séparés | — | |
| 4 | Sprint Outcome Gate + bump **v0.34.0** | — | |

### 🗂️ Backlog S41

| Item | Priorité | Note |
|---|---|---|
| **« More like this » sans backend** | **P1** | Feature morte : le modal appelait `/anime/{id}/recommendations`. Plus de source. Rebrancher sur `fetchRelationsByMalIdWithMeta` **ou** masquer le point d'entrée |
| `J10e-a/b/c` — repli orphelins titre+année | P2 | 3 slices. Population d'orphelins jamais mesurée |
| `US-SYNOPSIS-VERSIONTOP` | P2 | Synopsis dans `ModalVersionTop.vue` |
| `US-MODAL-NEXTEP-HIERARCHY` | P3 | Hiérarchie visuelle `ModalCalendarTop.vue` |
| Triage `BENCHMARK.md` | — | Gel PO levé une fois S40 clos |

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| Jikan `/v4/anime/1` | **200** + `X-Cache-Status: STALE` | **SE-062** | `curl -s -D -` PO |
| Jikan `/v4/anime/1/relations` | **504** | **SE-062** | idem |
| Jikan `/v4/anime/1/recommendations` | **504** | **SE-062** | idem — 1ʳᵉ mesure |
| AniList — Discover This Season | **fonctionnel** | **SE-062** | constat visuel PO |
| AniList — For You | **fonctionnel** | **SE-062** | constat visuel PO |

> **Le `200 STALE` de Jikan est un piège**, pas un signe de vie : réponse servie depuis un cache
> expiré, origine muette. Un 200 périmé est plus dangereux qu'un 504 — il passe pour un succès.
> **Ces faits n'ont plus d'impact sur le code** : aucune ligne du dépôt n'appelle Jikan.

---

## 🕳️ Trous ouverts

- **`usePersistence.ts:14-15,287`** — stub mort `_startBackgroundRelationFetch`, `TODO US-016`
  jamais fait, appelé uniquement par lui-même. Code mort confirmé, pas un chemin concurrent.
  → constat `AUDIT.md`, aucune US.
- **Deux signaux `stale` concurrents et tous deux morts** :
  `useAniListApi.ts:23,338` (`stale: boolean` dans `WithMeta`) et
  `usePersistence.ts:18,192,305` (`staleDataWarning`). **Zéro consommateur `.vue`**
  (grep récursif `src\*.ts src\*.vue`, SE-062). → objet d'`AUD-05`, DEC préalable requis.
- **ESLint jamais exécuté** — la promesse « zéro `any` » n'est vérifiée par aucun outil.
