# STATE.md — État courant Aanime

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Rôle :** source de vérité **UNIQUE** de l'état courant — sprint, session, versions,
> compteurs, Kanban, faits externes, trous ouverts.
> **Ce fichier est la seule copie des chiffres du projet.** Les autres documents y renvoient.
> *Exception unique : `AGENTS.md`, lu par Gemini qui n'a pas accès à ce fichier.*

---

## 🧭 Position courante

| | Valeur |
|---|---|
| **Sprint** | **S39 — EN COURS.** Goal : *Aanime lit AniList, la recherche fonctionne* |
| **Session courante** | **SE-056** |
| **Dernière version livrée** | **v0.32.0 (S38)** — S39 non clos, aucun bump |
| **Dernier DEC** | **DEC-130** |
| **Commit `main`** | `1b9bf3b` |
| **Échéance dure** | **Jikan ferme en octobre 2026** (DEC-125) |
| **Cadence** | **4 à 8 US par sprint**, davantage si elles servent un même sujet |

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-056** | S39 | `US-SEARCH-DEDUP-ANILIST` · `US-ANILIST-J06` · `US-MODAL-OPEN-SEED-KEY` annulée · DEC-127→130 | 199 tests, 2 MERGE |
| SE-055 | S39 | `J05-E2E` · `US-SEARCH-SPECS-ANILIST` · 5 dérives de contrat corrigées · `AUD-21` ouvert | 199 tests, 2 MERGE |
| SE-054 | S39 | J02→J05 AniList livrés · DEC-125 · DEC-126 | 199 tests |
| SE-053 | S38 | AUD-01 réparée · DEC-118 réouverte · clôture S38 | **v0.32.0** |
| SE-052 | S38 | AUD-01-E2E · AUD-02 · registre E2E réconcilié | 164 tests |

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.32.0** | **S38** | AUD-01 (garde `day` + cascade + repromotion) · AUD-02 · registre E2E · purge debug |
| v0.31.0 | S37 | US-GRID-FIX · US-MODAL-UNIFY annulée (DEC-110) |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT |
| v0.29.0 | S29→S33 | Refonte nav · US-127 · US-AUTH-LOGOUT |
| v0.28.0 | S28 | Epic Stats |
| — | S23→S27 | ⚠️ Détail non capturé |
| ≤ v0.21.0 | ≤ S21 | Migration vanilla → Vue 3, US-PINIA, US-JST, CI |

---

## 🎯 Métriques techniques

| Métrique | Valeur | Preuve | Fraîcheur |
|---|---|---|---|
| Tests unitaires | **199 passed** (24 fichiers) | `npm run test:run`, machine PO | **SE-056** |
| Type-check | **vert** | sortie vide | **SE-056** |
| Build | **vert** | `181 modules transformed` | **SE-056** |
| E2E — fichiers sur disque | **42** | `+1 modal-next-episode` (SE-056) | **SE-056** |
| E2E — registre `package.json` | **42 / 42** | batch3 passé à 8 entrées | **SE-056** |
| E2E — registre `AGENTS.md §7` | **42 / 42** | patch SE-056, redéployé racine | **SE-056** |
| E2E — rouges connus | **INCONNU** | voir Trous — aucun rouge mesuré | **SE-056** |

> ⚠️ Pas de mention « zéro `any` » : `vue-tsc` ne les teste pas, ESLint n'est jamais exécuté.
> **CI :** `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`. **Ni Playwright, ni ESLint** (`AUD-08`).
> **Install :** `npm install` en direct, `--legacy-peer-deps` inutile.
> ⚠️ `useJikanApi.spec.ts` consomme ~1,8 s à lui seul, soit ~40 % de la suite. Disparaîtra avec Jikan.
> 🔴 **Le compteur de rouges est remis à INCONNU, pas corrigé.** Les 3 rouges annoncés en SE-055
> reposaient sur une lecture de code, pas sur une exécution : `search-dedup` était réel et est
> réparé, `modal-open` n'a jamais été rouge. Voir `AP-PROCESS-2`. Seul le sweep tranchera.

### 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> 🔴 Trois sprints consécutifs réparent le parcours que ces métriques mesurent, sans pouvoir
> le prouver. **Priorité n°1 de S41.**

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S39** | ⏳ en cours |
| S38 | ✅ Gain ressenti. Un anime ajouté depuis n'importe quel chemin apparaît sur son jour |
| S37 | ✅ Gain ressenti. Densité de grille uniforme sur 4 surfaces |
| S36 | ✅ Gain ressenti. Fin du scroll latéral mobile, popups centrées |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.**
> 🔴 **Seul le `curl` du PO fait foi.** Les outils de Claude ont produit un faux 504 en SE-054.

### AniList — source primaire. Dernière mesure : SE-056 (visuel PO)

| Point | Résultat |
|---|---|
| `graphql.anilist.co` | **200.** CORS OK depuis l'origine de production |
| Rattachement `idMal` | **19 / 19** sur corpus de test (SE-054) |

⚠️ Corpus de 19 titres mainstream. Le taux d'orphelins réel sur un import MAL de 300 titres
sera > 0 → repli titre+année prévu en **J08**.

📌 **Mesure acceptée en SE-056 : constat visuel du PO dans l'app.** La requête du code est un
POST GraphQL multi-lignes, inéchappable proprement en `cmd` — une recherche qui rend des
résultats à l'écran est une mesure plus forte qu'un `curl` simplifié.

### Jikan v4 — en fin de vie. Dernière mesure : SE-053

| Endpoint | Code |
|---|---|
| `/seasons/now` | 200 |
| `/anime/{id}` | 200 |
| `/anime?q=` | **504 — mort, acté, ne plus remesurer** |

🔴 **Ferme en octobre 2026** (DEC-125).

---

## 📋 Kanban — Sprint S39

**Sprint Goal :** *Aanime lit AniList, la recherche fonctionne.*

### ✅ Done
- `J02` `6976c00` · `J03` `4cc0fcd` · `J04` `0622235` (+`18570e5`) · `J05` `27f9908`
- `J05-E2E` `fb3d1e3` — recherche AniList verrouillée par E2E
- `US-SEARCH-SPECS-ANILIST` `1f319cd` — 3 specs migrées, 1 preuve rouge produite
- `US-SEARCH-DEDUP-ANILIST` `0384cc8` — `dedupeByMalId` rebranché après le tri (DEC-127)
- `US-ANILIST-J06` `1b9bf3b` — `Next episode · Wednesday at 18:30` dans la modale calendrier
- ⛔ `US-MODAL-OPEN-SEED-KEY` — **ANNULÉE, sans objet** (DEC-130)
- Recherche **confirmée fonctionnelle en visuel** (PO, SE-055), modale incluse

### 🔄 In Progress
*(vide — SE-056 close)*

### 📝 To Do — S39, dans cet ordre
1. `US-ANILIST-J07` 🟠 — `description` → `synopsis` **avec nettoyage HTML** (voir Trous)
2. `AUD-21` 🔴 — jour inventé sur série sans `nextAiringEpisode`
3. `US-E2E-SWEEP` 🟢 — rejouer les **42** specs avant clôture

### 🗂️ Backlog

**S40 proposé — Goal : Jikan est débranché avant sa fermeture**
`J08` repli titre+année · `J09` saisons sur AniList 🔴 · `J10` sync/détail 🔴 ·
`J11` retrait de Jikan · `AUD-05` mode dégradé visible

**S41 proposé — Goal : mesurer et réparer le parcours d'entrée**
`US-TTFA-INSTRUMENT` · `US-ONBOARDING-REFRESH` 🔴 · `US-TOAST-ONBOARDING` ·
`US-CARDS-ALIGN` · `US-CLEAR-DATA`

**Non planifié**
`US-SYNOPSIS-VERSIONTOP` (synopsis absent de Plan to watch / Completed / Coming soon) ·
`US-DEMOGRAPHICS` (champ toujours vide sur AniList, lu par le moteur de reco) ·
`AUD-08` (CI sans Playwright ni ESLint) · `AUD-03`→`AUD-20` dans `AUDIT.md`

**Vérification avant ouverture**
Titres japonais/rōmaji — **probablement déjà résolu**, `normalizeAniList` préfère
`title.english`. Constat visuel requis avant d'ouvrir une US (cf. US-SEARCH-3).

**Benchmark concurrentiel** — livrable **déjà produit** hors projet (Claude Cowork).
À intégrer **à la clôture de S39**, pour armer le Goal de S41. N'entre pas dans le corpus
des 10 documents.

---

## 🕳️ Trous restants

| Trou | Gravité | État |
|---|---|---|
| État réel des rouges E2E **inconnu** | 🟠 | Aucun rouge mesuré depuis SE-056. Le sweep est la seule mesure |
| Specs au **seed décoratif** | 🟠 | `snap-to-today` sème un calendrier mais n'asserte qu'un en-tête de jour : il passerait avec un store vide. Couverture plus faible qu'annoncée. À qualifier au sweep, sur sorties |
| `description` AniList contient du **HTML** | 🟠 | `<br>`, `<i>`, `<b>` + entités. `{{ synopsisText }}` les afficherait littéralement → `J07` doit nettoyer, ce n'est pas un mapping d'une ligne |
| `src/types/anilist.ts` — champ `description` non vérifié | 🟢 | Bloquant pour `J07` : si absent de l'interface, US « types » d'abord |
| `AUD-21` — jour issu de la date de première diffusion | 🔴 | **Confirmé par lecture** : `normalizeAniList.ts` lignes 84-93, `getUTCDay()` sur `aired_from`. `airsTime` reste `null` → `J06` n'affiche rien dans ce cas, le faux jour reste visible sur la grille |
| `demographics` toujours vide sur AniList | 🟠 | Ligne 55 : `const demographics: string[] = []`. Dégrade le moteur de reco |
| CI sans Playwright ni ESLint | 🟠 | `AUD-08` |
| Métriques produit non instrumentées | 🟠 | Aucune baseline |
| `src/types/anilist.ts` absent de `TYPES_CONTRACT.md` | 🟢 | Ajouté aux lacunes §9 |
| `studios` optionnel dans le code vs DEC-86 | 🟢 | Contrat aligné sur le code |
| Détail des versions S23→S27 | 🟢 | Perdu, ne pas reconstituer |
