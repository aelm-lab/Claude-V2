# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état **courant** — sprint, session, compteurs,
> Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient.
> *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à ce fichier.*
>
> **DEC-146 — Ce fichier ne porte plus l'historique.** Sessions closes et versions livrées
> vivent dans `HISTORIQUE.md`. Régénération **intégrale** à chaque clôture, jamais de patch.

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint** | **S40 — ✅ CLOS.** Goal : *Jikan est débranché avant sa fermeture* — **ATTEINT** |
| **Sprint suivant** | **S41 — à composer** (triage benchmark préalable, gel levé) |
| **Session courante** | **SE-063** |
| **SE-063.b** | hors sprint | **Cleaning et compression de la Knowledge.** 12 documents réécrits, création de `DECISIONS_ARCHIVE.md` (satellite hors ordre de lecture), `AUDIT.md` enrichi en append. `DECISIONS.md` était collé deux fois dans lui-même (717 → 159 lignes). 15 informations périmées corrigées, dont la séquence de boot fausse depuis `J11b-1` et deux règles opposables d'`AGENTS.md` contradictoires. DEC-154 | **Aucun code.** Corpus 3441 → 2432 lignes (−29 %), ordre de lecture −37 % |
| **Dernière version livrée** | **v0.34.0 (S40)** |
| **Dernier DEC** | **DEC-153** |
| **Commit `main`** | `365a6aa` — arbre propre |
| **Échéance dure** | Jikan ferme en octobre 2026 — **neutralisée**, aucune ligne du dépôt ne l'appelle |
| **Cadence** | 4 à 8 US par sprint |

**Rien ne bloque.** Premier état du projet sans rouge connu.

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

---

## 📋 Kanban — S41 (à composer) · SE-063

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
| **`J12-a`** | 🟠 | **`1e119c4` + `8250862`** | Helper `installAniListMock` + 2 specs pilotes |
| **`J12-b`** | 🟠 | **`365a6aa`** | 9 specs restantes — **sweep 52/52** |

**Bilan Gemini S40 : merge au premier coup sur toutes les US, 0 correction majeure.**

### 🔄 In Progress
Aucune. S40 clos, S41 non composé.

### 📝 To Do — préalable à la composition de S41

| Ordre | Item | Note |
|---|---|---|
| 1 | **Triage `BENCHMARK.md`** — gel PO levé | 4 remesures, ~20 min (`BENCHMARK §9`) |
| 2 | Instruire `AUD-25` (lecture composant This Season) | Avant toute US |
| 3 | Composer S41 sur un Sprint Goal produit | Après triage seulement |

### 🗂️ Backlog S41

| Item | Priorité | Note |
|---|---|---|
| **« More like this » sans backend** | **P1** | Rebrancher sur `fetchRelationsByMalIdWithMeta` (DEC-152). **Pas de masquage** |
| **`AUD-25`** — asymétrie d'action This Season / For You | **P1** | Signalé PO SE-063. Lire le composant avant de spécifier |
| **`B-07`** — 0/13 contrôles ≥ 44 × 44 px | **P1** | Seul item « taille » mesurable en l'état |
| `AUD-05` — signal de fraîcheur visible | P2 | 🟠, DEC d'arbitrage préalable requis (DEC-151) |
| `AUD-24` — 12 specs démockées | P2 | Migration mécanique vers le helper (DEC-153) |
| `J10e-a/b/c` — repli orphelins titre+année | P2 | 3 slices |
| `US-SYNOPSIS-VERSIONTOP` | P2 | Synopsis dans `ModalVersionTop.vue` |
| `US-MODAL-NEXTEP-HIERARCHY` | P3 | Hiérarchie visuelle `ModalCalendarTop.vue` |

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| AniList — Discover This Season | **fonctionnel, contenu réel** | **SE-063** | captures PO |
| AniList — For You | **fonctionnel, badges et signaux OK** | **SE-063** | captures PO |
| AniList — Library Upcoming | **fonctionnel** | **SE-063** | captures PO |
| Jikan (tous endpoints) | **sans objet** | — | Aucune ligne du dépôt ne l'appelle |

> Jikan ne se remesure plus. Seul AniList compte désormais, et le constat visuel dans l'app suffit.

---

## 🕳️ Trous ouverts

- **`usePersistence.ts:14-15,287`** — stub mort `_startBackgroundRelationFetch`. Code mort
  confirmé, pas un chemin concurrent. → constat `AUDIT.md`, aucune US.
- **Deux signaux `stale` concurrents et morts** : `useAniListApi.ts:23,338` et
  `usePersistence.ts:18,192,305`. Zéro consommateur `.vue`. → `AUD-05`, DEC préalable requis.
- **ESLint jamais exécuté** — la promesse « zéro `any` » n'est vérifiée par aucun outil.
- **12 specs E2E vertes tapant le réseau réel** — `AUD-24`. Le sweep n'est pas déterministe.
