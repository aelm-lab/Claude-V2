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
| **Sprint** | **S38 — CLOS.** Goal atteint, Outcome Gate répondue |
| **Sprint suivant** | **S39 — à ouvrir**, Goal non défini |
| **Session courante** | **SE-053** — AUD-01 réparée en 2 US + clôture S38 |
| **Dernière version livrée** | **v0.32.0 (S38)** |
| **Dernier DEC** | **DEC-124** |
| **Commit `main`** | `2ff2753` *(2026-08-11 12:35)* |

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-053** | S38 | AUD-01 réparée (FIX + PROMOTE) · DEC-118 réouverte · clôture S38 | **v0.32.0** |
| SE-052 | S38 | AUD-01-E2E · AUD-02 · registre E2E réconcilié 41/41/41 | 164 tests |
| SE-051 | S38 | Rédaction des US S38 · AUDIT.md créé | 161 tests |
| SE-050 | S38 | Dual audit réconcilié · décision API | 0 code livré |
| SE-049 | — (hors sprint) | Refonte du corpus documentaire | 13 → 9 docs |

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
| Tests unitaires | **169 passed** (21 fichiers) | `npm run test:run`, machine PO | **SE-053** |
| Type-check | **vert** | `npm run type-check`, sortie vide | **SE-053** |
| Build | **vite 6.4.2 · 178 modules** | `npm run build` | **SE-053** |
| E2E — fichiers sur disque | **41** | `dir /b tests\e2e\*.spec.ts` | SE-052 |
| E2E — registre `package.json` | **41 / 41** | diff programmatique | SE-052 |
| E2E — registre `AGENTS.md §7` | **41 / 41** | comptage 9+9+7+9+7 | SE-052 |
| E2E — sweep complet | **45 verts / 50** | machine PO | SE-052, **à rejouer en S39** |
| E2E — batch1 | 9/10 | machine PO | **SE-053** |
| E2E — batch5 | 9/10 | machine PO | **SE-053** |

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
> 🔴 **S38 a réparé le parcours que ces métriques mesurent, sans pouvoir le prouver.**
> Candidat n°1 au Goal de S39.

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S38** | ✅ **Gain ressenti.** Un anime ajouté depuis n'importe quel chemin apparaît désormais sur son jour dans le calendrier — auparavant il disparaissait sans trace. Et l'app ne dit plus « Saved » quand Firestore est en panne. **Deux P0 fermés, dont celui qui cassait le parcours d'entrée.** |
| S37 | ✅ Gain ressenti. Densité de grille uniforme (2 colonnes) sur 4 surfaces |
| S36 | ✅ Gain ressenti. Fin du scroll latéral sur mobile, popups centrées |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.**

### Jikan v4 — dernière mesure : **SE-053 (2026-08-11)**

| Endpoint | Code | Note |
|---|---|---|
| `/seasons/now` | **200** | 152 items, 7 pages. **Renvoie `broadcast.day` + `time` + `timezone`** |
| `/anime/{id}` | **200** | testé sur 52991. Porte le `broadcast` complet |
| `/anime?q=` | **504** | inchangé depuis S33 |

**Diagnostic retenu (DEC-113) :** MyAnimeList est inaccessible depuis Jikan ; seules les URLs
déjà en cache chez Jikan répondent 200. Toute URL neuve → 504. La recherche produit par
construction des URLs neuves — d'où sa mort permanente.

🔴 **Correction de fait apportée en SE-053 :** l'affirmation « la donnée de diffusion est
absente » était fausse. Elle est présente dans les deux endpoints vivants, elle était
**ignorée au mapping**. DEC-118, bâtie sur cette prémisse, a été réouverte → **DEC-124**.

---

## 📋 Kanban — sprint S39 (à ouvrir)

### ✅ Done — S38

- **AUD-01** — CLOSE. Garde `day` + cascade `broadcast → aired.from` + marqueur
  `awaitingSchedule` + repromotion auto dans `useSync`. 2 US, 5 tests unitaires,
  1 E2E réparée
- **AUD-02** — CLOSE. `throw error.value` dans `useFirestore` + 3 tests unitaires
- **Registre E2E** — CLOSE. 41/41/41, mapping 1:1. Trou 🔴 de S37 fermé
- **AGENTS.md** — déployé à la racine, §7 à jour, en lecture seule pour Gemini (DEC-121)
- **US-ONBOARDING-REFRESH** — **absorbée par AUD-01.** La cause racine était la même ;
  le correctif de garde la rend sans objet. Reste le libellé du toast → S39

### 🔄 In Progress

*(vide)*

### 📝 To Do — S39, à composer

1. **[TEST] Sweep complet + purge des 4 specs rouges** 🟢 — dette, plafond §5 à surveiller
   - `discover-season-dedup` : sélecteur `.anime-grid` périmé depuis US-GRID-FIX (S37).
     **Requalifié en SE-053** — ce n'était pas de la dette réseau
   - `onboarding-toast` : libellé changé en SE-051, assertion non mise à jour
   - `search-hides-nav` : **non qualifié**, jamais instruit
   - `more-like-this-modal` : dette réseau réelle, `/recommendations` en 504 → `US-E2E-MLT-MOCK`
2. **[UX] Libellé du toast d'onboarding** 🟢 — doit nommer la destination réelle
3. **[MESURE] Instrumenter le TTFA** ⬆️ — S38 a réparé le parcours sans pouvoir le prouver

### 🗂️ Backlog

#### Condition de lancement public

- **[US-ANILIST-J01]** ⬆️⬆️ 🔴 — **Spike bloquant, go/no-go du lot.** (1) CORS réel depuis
  l'origine de production, preflight **et** réponse ; (2) taux d'`idMal` null sur le corpus
  réel. Seuils : `< 2 %` on avance · `2–10 %` US de rattrapage · `> 10 %` on rouvre la décision.
  **Planification non tranchée — décision PO attendue.**
- **[US-ANILIST-J02→J12]** 🔴 — lot de migration phasé. Phase 1 = coexistence (recherche +
  détail sur AniList, surfaces mortes donc zéro régression), feature flag, réversible en
  un commit. `AUD-08` est un prérequis de la phase 1.

#### Audit S38 — reste à convertir

`AUD-03` à `AUD-20`, contenu dans `AUDIT.md`. Prioritaires : `AUD-04` (coupe-circuit global,
bloquant migration) · `AUD-05` (mode dégradé, à fusionner avec `US-CACHE-STALE-WARNING`) ·
`AUD-08` (CI sans Playwright ni ESLint).
`AUD-12`, `AUD-19`, `AUD-20` : ⏸️ à vérifier avant conversion.

#### Backlog produit (captures d'onboarding)

- Titres affichés en japonais/rōmaji au lieu de l'anglais
- Cartes d'onboarding désalignées
- Lien « Clear my data » dans la modale de déconnexion

---

## 🕳️ Trous restants

| Trou | Gravité | État |
|---|---|---|
| 4 specs E2E rouges | 🟠 | 3 qualifiées, 1 non instruite. → S39 |
| CI sans Playwright ni ESLint | 🟠 | `AUD-08`, prérequis migration |
| Métriques produit non instrumentées | 🟠 | Aucune baseline depuis leur création |
| `AUD-17` — stubs `_syncAnimeUpdates` / `_startBackgroundRelationFetch` | 🟢 | Doublons morts, vérifiés inoffensifs en SE-053 |
| Détail des versions S23→S27 | 🟢 | Perdu, ne pas reconstituer |
