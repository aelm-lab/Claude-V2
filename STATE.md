# STATE.md — État courant Aanime

> **Rôle :** source de vérité **UNIQUE** de l'état **courant** — sprint, session, compteurs, Kanban, faits externes, trous. Les autres documents y renvoient.
> *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à ce fichier.*

**DEC-146 — Ce fichier ne porte pas l'historique.** Sessions closes et versions livrées vivent dans `HISTORIQUE.md`. Régénération **intégrale** à chaque clôture, jamais de patch.

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint** | **S40 — ✅ CLOS.** Goal : *Jikan est débranché avant sa fermeture* — **ATTEINT** |
| **Sprint suivant** | **S41 — à composer** (triage benchmark préalable, gel levé) |
| **Session courante** | **SE-063.b — hors sprint** (chantier documentaire, aucun code) |
| **Dernière version livrée** | **v0.34.0 (S40)** — pas de bump : session hors sprint |
| **Dernier DEC** | **DEC-154** |
| **Commit `main`** | `365a6aa` — arbre propre, **inchangé cette session** |
| **Cadence** | 4 à 8 US par sprint |

**Rien ne bloque.** Aucun rouge connu sur le code.

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **265 passed** (29 fichiers) | `npm run test:run`, machine PO | **SE-063** |
| Durée suite unitaire | **4,21 s** | idem | **SE-063** |
| Type-check | **vert** | sortie vide | **SE-063** |
| Build | **vert** | `180 modules transformed`, 2,51 s | **SE-063** |
| E2E — specs sur disque | **42** (+1 helper hors batch) | inchangé | **SE-063** |
| E2E — registre `package.json` | **42 / 42** | 5 batches : 9·9·8·9·7 | **SE-063** |
| **E2E — sweep complet** | **✅ 52 / 52 · 0 rouge** | 5 batches séparés, machine PO | **SE-063** |

**Répartition du sweep :** batch1 10 · batch2 12 · batch3 9 · batch4 11 · batch5 10.

> ⚠️ Pas de mention « zéro `any` » : `vue-tsc` ne les teste pas, ESLint n'est jamais exécuté.
> Aucune métrique n'a été remesurée en SE-063.b — **aucune ligne de code n'a été touchée.**

---

## 📚 Métriques documentaires

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Corpus complet | **2 432 lignes** (14 documents) | **SE-063.b** |
| Ordre de lecture (10 documents) | **1 935 lignes** | **SE-063.b** |
| Document le plus lourd | `TYPES_CONTRACT.md` — 359 lignes | **SE-063.b** |
| Documents au-dessus du plafond H7 | **0** | **SE-063.b** |

---

## 📋 Kanban — S41 (à composer) · SE-063.b

### ✅ Done — sprint S40 (clos)

| US | Risque | Commit | Objet |
|---|---|---|---|
| `J09a` | 🟠 | `4403bd5` | Season AniList |
| `J09b` | 🟠 | `31ec1b4` | Season AniList (suite) |
| `J10a` | 🔴 | `0ed68f6` | `fetchRelations` |
| `J10b` | 🔴 | `65af6eb` | Relations sur AniList |
| `J10c` | 🔴 | — | Bande relations (504 silencieux corrigé) |
| `J10d` | 🔴 | `169f48d` | `syncAnimeUpdates` sur AniList, liste close de 9 champs |
| `J11a-1` | 🟠 | `3b87dd6` | `fetchUpcomingSeason` · `fetchTopFinished` · `resolveNextSeason` |
| `J11a-2` | 🔴 | `8234404` | `fetchRecPool` sur AniList |
| `J11a-3` | 🔴 | `70a7b91` | `getSeasonNudges` sur relations AniList |
| `J11b-1` | 🔴 | `2a24b3c` | Suppression du worker de relations en fond |
| `J11b-2` | 🟢 | `90dd38a` | Suppression `useJikanApi.ts` |
| `J11b-3` | 🟢 | `91b78eb` | Purge `helpers.ts` — dernière ligne de Jikan |
| `J12-a` | 🟠 | `1e119c4` + `8250862` | Helper `installAniListMock` + 2 specs pilotes |
| `J12-b` | 🟠 | `365a6aa` | 9 specs restantes — **sweep 52/52** |

**Bilan Gemini S40 : merge au premier coup sur toutes les US, 0 correction majeure.**

### ✅ Done — hors sprint (SE-063.b)

| Chantier | Objet |
|---|---|
| **Cleaning documentaire** | 12 documents réécrits, création de `DECISIONS_ARCHIVE.md`, `AUDIT.md` enrichi en append. **DEC-154.** Détail → `HISTORIQUE.md` |

### 🔄 In Progress
Aucune. S40 clos, S41 non composé.

### 📝 To Do — préalable à la composition de S41

| Ordre | Item | Note |
|---|---|---|
| 1 | **Déployer `AGENTS.md` à la racine de `A-Anime`** | 🔴 **Commit seul, avant toute US** (`AP-PROCESS-3`). Version en place = celle qui demande encore de mocker par URL |
| 2 | **Triage `BENCHMARK.md`** — gel PO levé | 4 remesures, ~20 min (`BENCHMARK §8`) |
| 3 | Instruire `AUD-25` (lecture composant This Season) | Avant toute US |
| 4 | Composer S41 sur un Sprint Goal produit | Après triage seulement |

### 🗂️ Backlog S41

| Item | Priorité | Note |
|---|---|---|
| **« More like this » sans backend** | **P1** | Rebrancher sur `fetchRelationsByMalIdWithMeta` (DEC-152). **Pas de masquage** |
| **`AUD-25`** — asymétrie d'action This Season / For You | **P1** | Lire le composant avant de spécifier |
| **`B-07`** — 0/13 contrôles ≥ 44 × 44 px | **P1** | Seul item « taille » mesurable en l'état |
| **`B-04`** — `.app-header` jamais rethémé | **P1** | Logo à 1,47:1 en mode sombre — l'app a l'air cassée. Effort petit |
| `AUD-05` — signal de fraîcheur visible | P2 | 🟠, DEC d'arbitrage préalable requis (DEC-151) |
| `AUD-24` — 12 specs démockées | P2 | Migration mécanique vers le helper (DEC-153) |
| `J10e-a/b/c` — repli orphelins titre+année | P2 | 3 slices (DEC-145) |
| `US-SYNOPSIS-VERSIONTOP` | P2 | Synopsis dans `ModalVersionTop.vue` |
| `US-MODAL-NEXTEP-HIERARCHY` | P3 | Hiérarchie visuelle `ModalCalendarTop.vue` |

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| AniList — Recherche | **fonctionnelle en production** | **SE-063.b** | constat PO |
| AniList — Discover This Season | **fonctionnel, contenu réel** | **SE-063** | captures PO |
| AniList — For You | **fonctionnel, badges et signaux OK** | **SE-063** | captures PO |
| AniList — Library Upcoming | **fonctionnel** | **SE-063** | captures PO |

> AniList est la seule dépendance externe de données. Le constat visuel dans l'app suffit à la remesure.

---

## 🕳️ Trous ouverts

- **ESLint jamais exécuté** — la promesse « zéro `any` » (R-CODE-1) n'est vérifiée par aucun outil de la porte verte.
- **12 specs E2E vertes tapant le réseau réel** — `AUD-24`. Le sweep n'est pas déterministe.
- **Deux signaux `stale` concurrents et morts** : `useAniListApi.ts:23,338` et `usePersistence.ts:18,192,305`. Zéro consommateur `.vue`. → `AUD-05`, DEC d'arbitrage préalable requis.
- **`usePersistence.ts:14-15,287`** — stub mort `_startBackgroundRelationFetch`. Code mort confirmé. → constat `AUDIT.md`, aucune US.
- **⚠️ Non revérifié par lecture de code (SE-063.b)** — l'atterrissage de l'onboarding a été documenté comme sain sur la seule base de DEC-124/DEC-131 et du sweep vert, **sans grep de `buildSeedEntry`**. Signature non contractualisée (`TYPES_CONTRACT.md §10`). À lever avant toute US touchant l'onboarding.
- **11 constats d'audit à l'état inconnu** — `AUD-02/03/06/07/09/10/11/13/14/15/16` n'ont pas été relus depuis leur campagne. Ils ne sont **ni soldés ni confirmés** : toute conversion en US démarre par un grep.
