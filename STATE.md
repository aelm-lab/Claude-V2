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
| **Sprint** | **S39 — CLOS.** Goal atteint. **S40 à ouvrir** |
| **Session courante** | **SE-057** |
| **Dernière version livrée** | **v0.33.0 (S39)** |
| **Dernier DEC** | **DEC-132** |
| **Commit `main`** | `b381e2f` |
| **Échéance dure** | **Jikan ferme en octobre 2026** (DEC-125) |
| **Cadence** | **4 à 8 US par sprint**, davantage si elles servent un même sujet |

---

## 🕐 Sessions (5 dernières, rotatif)

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| **SE-057** | S39 | `J07` synopsis · `AUD-21` faux jour · sweep E2E complet · **clôture S39** | **v0.33.0**, 205 tests, 2 MERGE |
| SE-056 | S39 | `US-SEARCH-DEDUP-ANILIST` · `J06` · `US-MODAL-OPEN-SEED-KEY` annulée · DEC-127→130 | 199 tests, 2 MERGE |
| SE-055 | S39 | `J05-E2E` · `US-SEARCH-SPECS-ANILIST` · 5 dérives de contrat · `AUD-21` ouvert | 199 tests, 2 MERGE |
| SE-054 | S39 | J02→J05 AniList livrés · DEC-125 · DEC-126 | 199 tests |
| SE-053 | S38 | AUD-01 réparée · DEC-118 réouverte · clôture S38 | v0.32.0 |

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| **v0.33.0** | **S39** | Migration AniList : J02→J07 · recherche fonctionnelle · synopsis nettoyé · AUD-21 (fin des faux jours) |
| v0.32.0 | S38 | AUD-01 (garde `day` + cascade + repromotion) · AUD-02 · registre E2E · purge debug |
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
| Tests unitaires | **205 passed** (24 fichiers) | `npm run test:run`, machine PO | **SE-057** |
| Type-check | **vert** | sortie vide | **SE-057** |
| Build | **vert** | `181 modules transformed` | **SE-057** |
| E2E — fichiers sur disque | **42** | inchangé depuis SE-056 | **SE-057** |
| E2E — registre `package.json` | **42 / 42** | 5 batches : 9·9·8·9·7 | **SE-057** |
| E2E — registre `AGENTS.md §7` | **42 / 42** | patch SE-057 redéployé racine | **SE-057** |
| E2E — cas exécutés | **52** | sweep 5 batches séparés | **SE-057** |
| E2E — **rouges = 3, qualifiés** | voir §Rouges E2E | sweep complet, sorties PO | **SE-057** |

> ⚠️ Pas de mention « zéro `any` » : `vue-tsc` ne les teste pas, ESLint n'est jamais exécuté.
> **CI :** `npm ci` → `vue-tsc --noEmit` → `vitest run` → `build`. **Ni Playwright, ni ESLint** (`AUD-08`).
> **Install :** `npm install` en direct, `--legacy-peer-deps` inutile.
> ⚠️ `useJikanApi.spec.ts` consomme ~1,8 s, soit ~40 % de la suite. Disparaîtra avec Jikan.
> 🔴 **Ne jamais lancer `npm run test:e2e:sweep`** : il chaîne les batches en `&&` et s'arrête au
> premier rouge. Les 5 batches se lancent **séparément**, sinon on mesure une panne, pas un état.

---

## 🔴 Rouges E2E — 3, qualifiés, non diagnostiqués

| Spec | Batch | Nature | Terrain |
|---|---|---|---|
| `more-like-this-modal` | 5 | **Expliqué, pas une régression.** Dépend de `/anime/{id}/recommendations` Jikan, en 504. Tombera seul avec `J11` | S40 |
| `discover-season-dedup` | 1 | `AUD-22` — dédoublonnage saison. La page tape encore Jikan. Hypothèses concurrentes non tranchées : dédoublonnage cassé, ou cache périmé porteur de doublons | **S40, avec `J09`** |
| `onboarding-toast` | 4 | `AUD-23` — compte du toast de bienvenue avant redirection. Voisin de `US-TOAST-ONBOARDING`, déjà au backlog | **S41** |

> Ces rouges peuvent préexister à S39 : le compteur était `INCONNU` depuis SE-055. Aucun n'est
> sur le chemin de la recherche. **Aucun n'a été diagnostiqué** — ne pas inscrire de cause.

---

## 📈 Métriques produit

| Métrique | Valeur |
|---|---|
| TTFA — time to first anime | **non instrumenté** — baseline 0 |
| Adds / semaine | **non instrumenté** — baseline 0 |
| Jours-retour semaine 1 | **non instrumenté** — baseline 0 |

> 🔴 Quatre sprints consécutifs réparent le parcours que ces métriques mesurent, sans pouvoir
> le prouver. **Priorité n°1 de S41.**

---

## 🏁 Sprint Outcome Gate

| Sprint | Verdict |
|---|---|
| **S39** | ✅ **Gain ressenti + gain de fiabilité.** La recherche remarche sur AniList, confirmée en visuel et verrouillée par 3 specs vertes. Le calendrier n'invente plus de jour de diffusion |
| S38 | ✅ Gain ressenti. Un anime ajouté depuis n'importe quel chemin apparaît sur son jour |
| S37 | ✅ Gain ressenti. Densité de grille uniforme sur 4 surfaces |
| S36 | ✅ Gain ressenti. Fin du scroll latéral mobile, popups centrées |

---

## 🌐 Faits externes

> **Règle (`PILOTAGE.md §6`) : remesurer à l'ouverture de chaque session, avec la requête
> EXACTE émise par le code.** 🔴 Seule une mesure du PO fait foi. Les outils de Claude ont
> produit un faux 504 en SE-054.

### AniList — source primaire. Dernière mesure : SE-057 (visuel PO)

| Point | Résultat |
|---|---|
| `graphql.anilist.co` | **200.** CORS OK depuis l'origine de production |
| Recherche + ajout + modale | **fonctionnels bout en bout**, constatés SE-057 |
| Rattachement `idMal` | **19 / 19** sur corpus de test (SE-054) |

⚠️ Corpus de 19 titres mainstream. Le taux d'orphelins réel sur un import MAL de 300 titres
sera > 0 → repli titre+année prévu en **J08**.

📌 **Mesure acceptée : constat visuel du PO dans l'app.** La requête du code est un POST
GraphQL multi-lignes, inéchappable proprement en `cmd` — une recherche qui rend des résultats
à l'écran est une mesure plus forte qu'un `curl` simplifié.

### Jikan v4 — en fin de vie. Dernière mesure : SE-053

| Endpoint | Code |
|---|---|
| `/seasons/now` | 200 |
| `/anime/{id}` | 200 |
| `/anime?q=` | **504 — mort, acté, ne plus remesurer** |
| `/anime/{id}/recommendations` | **504** — confirmé indirectement par `more-like-this-modal` (SE-057) |

🔴 **Ferme en octobre 2026** (DEC-125).

---

## 📋 Kanban

### ✅ S39 — CLOS, Goal atteint
`J02` `6976c00` · `J03` `4cc0fcd` · `J04` `0622235` · `J05` `27f9908` · `J05-E2E` `fb3d1e3` ·
`US-SEARCH-SPECS-ANILIST` `1f319cd` · `US-SEARCH-DEDUP-ANILIST` `0384cc8` · `J06` `1b9bf3b` ·
`J07` `1563957` · `AUD-21` `b381e2f` · sweep E2E complet
⛔ `US-MODAL-OPEN-SEED-KEY` annulée (DEC-130) · ⛔ `US-SYNOPSIS-LINEBREAKS` absorbée (P3 S41)

### 🔄 In Progress
*(vide — S39 clos, S40 non ouvert)*

### 📝 S40 proposé — Goal : *Jikan est débranché avant sa fermeture*
1. `J08` repli titre+année
2. `J09` saisons sur AniList 🔴 — **traiter `AUD-22` dans le même passage**
3. `J10` sync/détail sur AniList 🔴 — **débloque le synopsis de la bibliothèque existante**
4. `J11` retrait de Jikan — **fait tomber `more-like-this-modal`**
5. `AUD-05` mode dégradé visible — **valeur augmentée par AUD-21** : une série qui quitte la
   grille faute de date ne l'explique à personne

### 🗂️ S41 proposé — Goal : *mesurer et réparer le parcours d'entrée*
`US-TTFA-INSTRUMENT` **P1** · `US-SYNOPSIS-VERSIONTOP` 🟠 **P2** · `US-ONBOARDING-REFRESH` 🔴 ·
`AUD-23` (rouge onboarding-toast) · `US-TOAST-ONBOARDING` · `US-MODAL-NEXTEP-HIERARCHY` 🟢 **P3** ·
`US-CARDS-ALIGN` · `US-CLEAR-DATA`

> **`US-SYNOPSIS-VERSIONTOP` 🟠 promue P2** (constat visuel SE-057) : la modale ouverte depuis
> un résultat de recherche — donc **au moment de la décision d'ajout** — n'affiche aucun résumé.
> Composant `ModalVersionTop.vue`. Concerne aussi Plan to watch / Completed / Coming soon.
> ≥ 4 surfaces → **à couper en deux US**.
> **`US-MODAL-NEXTEP-HIERARCHY` 🟢 P3** : « Next episode » rendu comme une légende grise alors
> que c'est l'info la plus utile ; `Episode X/Y` et `+1` dispersés ; format verbeux ; synopsis
> tronqué **sans moyen de déplier**. 1 fichier, `ModalCalendarTop.vue`.

### Non planifié
`US-DEMOGRAPHICS` (champ absent d'AniList — devrait être **dérivé des tags**, décision produit,
lu par le moteur de reco donc 🔴 déguisé) · `AUD-08` (CI sans Playwright ni ESLint) ·
`AUD-03`→`AUD-20` dans `AUDIT.md`

### Benchmark concurrentiel
Livrable **déjà produit** hors projet (Claude Cowork). À traiter en **session dédiée hors
sprint (SE-058)**, rapport collé en premier message. N'entre pas dans le corpus des 10 documents.
🔴 **Réalisé pendant que l'API était morte** → tri obligatoire en deux piles avant tout usage :
**A exploitable** (navigation, grille, calendrier sur cache, stats, mise en page) ·
**B à jeter ou rejouer** (recherche, découverte, ajout, saisons, TTFA, premier lancement).
La pile B n'est rejouable qu'**après S40**. Aucune US S41 n'est écrite avant ce tri.

---

## 🕳️ Trous restants

| Trou | Gravité | État |
|---|---|---|
| `AUD-22` — `discover-season-dedup` rouge | 🟠 | Non diagnostiqué. À traiter **avec `J09`**, qui réécrit la page saison — un diagnostic posé avant serait jetable |
| `AUD-23` — `onboarding-toast` rouge | 🟠 | Non diagnostiqué. Terrain S41 |
| **Bibliothèque existante sans synopsis** | 🟠 | `J07` n'agit qu'à la normalisation → seuls les **nouveaux ajouts** ont un résumé. Les entrées déjà persistées n'en auront qu'après `J10` (sync/détail sur AniList) |
| Specs au **seed décoratif** | 🟠 | `snap-to-today` sème un calendrier mais n'asserte qu'un en-tête de jour : passerait avec un store vide. Couverture plus faible qu'annoncée. Non requalifié au sweep (vert ≠ probant) |
| `AUD-05` mode dégradé invisible | 🟠 | Une série sans date prouvée quitte la grille **en silence**. Planifié S40 |

> ✅ **Fermé en SE-057 :** trou « titres japonais/rōmaji » — constaté résolu en visuel,
> `normalizeAniList` préfère `title.english`. Aucune US ouverte.
> ✅ **Fermé en SE-057 :** trou « état des rouges E2E inconnu » — sweep complet, 3 rouges qualifiés.
> ✅ **Fermé en SE-057 :** `AniListMedia` absent de `TYPES_CONTRACT.md` — contractualisé.
