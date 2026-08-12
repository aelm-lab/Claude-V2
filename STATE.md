# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état courant — sprint, session, versions,
> compteurs, Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient.
> *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à ce fichier.*
>
> **Ce qui n'est PAS ici :** les règles (→ `PILOTAGE.md`, `AGENTS.md`) · le pourquoi des choix
> (→ `DECISIONS.md`) · l'acquis fonctionnel (→ `EPICS.md`) · le contenu des constats d'audit
> (→ `AUDIT.md`, satellite chargé à la demande).

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint** | **S39 — EN COURS.** Goal : *Aanime lit AniList, la recherche fonctionne* |
| **Session courante** | **SE-055** |
| **Dernière version livrée** | **v0.32.0 (S38)** — S39 non clos, aucun bump |
| **Dernier DEC** | **DEC-126** |
| **Commit `main`** | `27f9908` |
| **Échéance dure** | **Jikan ferme en octobre 2026** (DEC-125) — le full switch AniList n'est pas négociable |

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-055** | S39 | Sync documentaire · 5 dérives de contrat traitées · US-ANILIST-J05-E2E | — |
| SE-054 | S39 | J02→J05 AniList livrés · DEC-125 · DEC-126 | 199 tests |
| SE-053 | S38 | AUD-01 réparée (FIX + PROMOTE) · DEC-118 réouverte · clôture S38 | **v0.32.0** |
| SE-052 | S38 | AUD-01-E2E · AUD-02 · registre E2E réconcilié 41/41/41 | 164 tests |
| SE-051 | S38 | Rédaction des US S38 · AUDIT.md créé | 161 tests |

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.32.0** | **S38** | **AUD-01** (garde `day` + cascade de résolution + repromotion auto) · **AUD-02** (throw Firestore propagé) · registre E2E réconcilié · purge des fichiers de debug |
| v0.31.0 | S37 | US-GRID-FIX · US-MODAL-UNIFY annulée (DEC-110) |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT · US-SEARCH-3 |
| v0.29.0 | S29→S33 | Refonte nav · US-127 · US-AUTH-LOGOUT |
| v0.28.0 | S28 | Epic Stats |
| — | S23→S27 | ⚠️ Détail non capturé |
| v0.21.0 | S21 | US-152 · US-157 · US-158 |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154→156 · US-167 |
| ≤ S16 | — | Migration vanilla → Vue 3 |

> Le numéro de version ne suit plus le numéro de sprint depuis S29 (`PILOTAGE.md §1`).

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **199 passed** (24 fichiers) | `npm run test:run`, machine PO | **SE-054** |
| Type-check | **vert** | `npm run type-check`, sortie vide | **SE-054** |
| Build | **vert** | `npm run build` | **SE-054** |
| E2E — fichiers sur disque | **41** | `dir /b tests\e2e\*.spec.ts` | SE-052 |
| E2E — registre `package.json` | **41 / 41** | diff programmatique | SE-052 |
| E2E — registre `AGENTS.md §7` | **41 / 41** | comptage 9+9+7+9+7 | SE-052 |
| E2E — sweep complet | **45 verts / 50** | machine PO | SE-052, **à rejouer en S39** |

> ⚠️ Pas de mention « zéro `any` » : `vue-tsc` ne les teste pas, ESLint n'est jamais exécuté.
> Mesure réelle : `helpers.ts:32`. → `AUDIT.md`, `AUD-08` / `AUD-14`.

### Chaîne CI

`.github/workflows/ci.yml` : `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`.
**Ni Playwright, ni ESLint.** → `AUDIT.md`, `AUD-08`.

**Install :** `npm install` en direct. `--legacy-peer-deps` n'est plus nécessaire.

### 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> Jamais instrumentées. Définitions → `PILOTAGE.md §4`.
> 🔴 Deux sprints consécutifs (S38, S39) réparent le parcours que ces métriques mesurent,
> sans pouvoir le prouver. **Candidat n°1 au Goal de S40.**

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S39** | ⏳ en cours |
| **S38** | ✅ **Gain ressenti.** Un anime ajouté depuis n'importe quel chemin apparaît sur son jour dans le calendrier — auparavant il disparaissait sans trace. Et l'app ne dit plus « Saved » quand Firestore est en panne |
| S37 | ✅ Gain ressenti. Densité de grille uniforme (2 colonnes) sur 4 surfaces |
| S36 | ✅ Gain ressenti. Fin du scroll latéral sur mobile, popups centrées |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.**

### AniList — source primaire depuis S39. Dernière mesure : **SE-054 (J01)**

| Point | Résultat |
|---|---|
| `graphql.anilist.co` | **200.** CORS OK depuis l'origine de production (preflight + réponse) |
| Rattachement `idMal` | **19 / 19** sur corpus de test — 0 % d'orphelins |

⚠️ **Réserve :** corpus de 19 titres mainstream. Le taux d'orphelins réel sur un import MAL
de 300 titres sera > 0 → repli titre+année prévu en **J-08**.

### Jikan v4 — en fin de vie. Dernière mesure : **SE-053 (2026-08-11)**

| Endpoint | Code | Note |
|---|---|---|
| `/seasons/now` | **200** | renvoie `broadcast.day` + `time` + `timezone` |
| `/anime/{id}` | **200** | porte le `broadcast` complet |
| `/anime?q=` | **504** | **mort, acté — ne plus remesurer du tout** |

🔴 **`Jikan ferme en octobre 2026` (DEC-125).** Aucune mesure ne changera cette échéance.

🔴 **Protocole de mesure :** seul le `curl` du PO fait foi. Les outils de Claude ont produit
un **faux 504** sur `/seasons/now` en SE-054 (artefact de proxy). Ne plus mesurer via Claude.

---

## 📋 Kanban — Sprint S39

**Sprint Goal :** *Aanime lit AniList, la recherche fonctionne.*

### ✅ Done
- `J02` `6976c00` · `J03` `4cc0fcd` · `J04` `0622235` (+ fix `18570e5`) · `J05` `27f9908`
- Recherche **confirmée fonctionnelle en visuel** (PO, SE-055) — modale d'anime incluse
- Sync documentaire : 5 dérives de contrat traitées · DEC-125 / DEC-126 inscrites ·
  DEC-115 marquée superseded · 2 antipatterns gravés

### 🔄 In Progress
- `US-ANILIST-J05-E2E` — verrouiller la recherche par E2E

### 📝 To Do — S39
- `US-ANILIST-J06` — détail anime sur AniList (synopsis, studio, prochain épisode + heure)
- Sweep des 3 specs E2E rouges restantes
- Libellé du toast d'onboarding
- Vérifier `useEpisodeInfo.getStatus` vs `getCardStatus` (contrat §8) — **avant J06**

### 🗂️ Backlog

#### Migration AniList
- `J-08` — repli titre+année pour les orphelins d'import MAL
- `J07` → `J12` — suite du lot de migration phasé

#### Audit S38 — reste à convertir
`AUD-03` à `AUD-20`, contenu dans `AUDIT.md`. Prioritaires : `AUD-05` (mode dégradé, à
fusionner avec `US-CACHE-STALE-WARNING`) · `AUD-08` (CI sans Playwright ni ESLint).
`AUD-12`, `AUD-19`, `AUD-20` : ⏸️ à vérifier avant conversion.
**`AUD-04` est ANNULÉE** (DEC-126).

#### Backlog produit (captures d'onboarding)
- Titres affichés en japonais/rōmaji au lieu de l'anglais
- Cartes d'onboarding désalignées
- Lien « Clear my data » dans la modale de déconnexion
- Instrumentation TTFA

#### Test
- `J05-E2E-bis` 🟢 — couvrir les états `empty` et `error` du dropdown de recherche

---

## 🕳️ Trous restants

| Trou | Gravité | État |
|---|---|---|
| 3 specs E2E rouges | 🟠 | → S39 |
| CI sans Playwright ni ESLint | 🟠 | `AUD-08` |
| Métriques produit non instrumentées | 🟠 | Aucune baseline depuis leur création |
| `studios` optionnel dans le code vs DEC-86 « toujours peuplé » | 🟢 | Contrat aligné sur le code en SE-055 ; `normalize.ts` non relu |
| `getStatus` vs `getCardStatus` | 🟢 | Détecté SE-055, non tranché |
| `AUD-17` — stubs `_syncAnimeUpdates` / `_startBackgroundRelationFetch` | 🟢 | Doublons morts, inoffensifs |
| Détail des versions S23→S27 | 🟢 | Perdu, ne pas reconstituer |
