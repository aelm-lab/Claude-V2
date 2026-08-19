# HISTORIQUE.md — Journal des sessions et des versions

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
> **Créé en SE-062 par DEC-146**, par extraction des sections « Sessions » et « Versions »
> de `STATE.md`.
>
> **Rôle :** mémoire du **passé** clos. `STATE.md` porte le présent, ce fichier porte ce qui
> ne bouge plus.
>
> **Hors ordre de lecture obligatoire.** Comme `AUDIT.md` et `BENCHMARK.md`, Claude ne l'ouvre
> qu'à la demande. Il ne pollue pas l'index de la Knowledge.
>
> ## Règle de mise à jour — APPEND-ONLY
>
> | Événement | Action |
> |---|---|
> | Clôture de **session** | **+1 ligne** dans §Sessions. Rien d'autre. |
> | Clôture de **sprint** | +1 ligne dans §Sessions **et** +1 ligne dans §Versions. |
>
> **Ce fichier n'est JAMAIS régénéré, jamais réécrit, jamais réordonné.**
> C'est la parade structurelle à la perte d'information : un document qu'on n'écrase pas
> ne peut rien perdre. Coût : ~30 tokens par session.
>
> Purge H9 (tous les 5 sprints) : les lignes de plus de 5 sprints peuvent être condensées
> en une ligne de synthèse par sprint — jamais supprimées.

---

## 🕐 Sessions

> Ordre chronologique. On ajoute en bas.

| Session | Sprint | Objet | Sortie |
|---|---|---|---|
| ≤ SE-054 | ≤ S39 | ⚠️ Détail non capturé — antérieur à la création de ce fichier | — |
| SE-055 | S39 | `J05-E2E` · `US-SEARCH-SPECS-ANILIST` · `AUD-21` ouvert | 199 tests, 2 MERGE |
| SE-056 | S39 | `US-SEARCH-DEDUP-ANILIST` · `J06` · DEC-127→130 | 199 tests, 2 MERGE |
| SE-057 | S39 | `J07` synopsis · `AUD-21` faux jour · sweep E2E · **clôture S39** | v0.33.0, 205 tests, 2 MERGE |
| SE-058 | hors sprint | Tri A/B du benchmark · 3 corrections de faits · DEC-133 · création `BENCHMARK.md` | Aucun code |
| SE-059 | S40 | Planning S40 · `J08` fusionnée dans `J10` · `J09` coupée · `AUD-05` déclassée · `AUD-22` diagnostiqué · DEC-134→136 | 2 MERGE, 218 tests |
| SE-060 | S40 | `J10a` `J10b` · DEC-137→140 · `J10` éclatée en 5 slices | 2 MERGE |
| SE-061 | S40 | `J10d` · `J11a-1/2/3` · `useRecommendations` doté de ses 2 premières specs · DEC-141→145 | **4 MERGE au 1ᵉʳ coup**, 280 tests |
| **SE-062** | **S40** | **`J11b-1/2/3` — Jikan intégralement retiré du dépôt.** Dette `AGENTS.md` E2E classée (patch déjà déployé en `4b99eeb`, handoff SE-061 erroné). Sweep complet : 12 rouges, cause unique → epic `J12`. DEC-146→149. Création de ce fichier | **3 MERGE au 1ᵉʳ coup**, 265 tests, **Sprint Goal S40 atteint, sprint non clos** |
| **SE-063** | S40 | `J12-a` (helper `installAniListMock` + 2 specs pilotes) · `J12-b` (9 specs) | **2 MERGE au 1ᵉʳ coup, 0 correction.** Sweep **52/52 verts — premier sweep intégralement vert du projet.** Diagnostic du handoff SE-062 corrigé : aucune des 11 rouges ne routait `api.jikan.moe`. Découverte `AUD-24` (12 specs démockées) et `AUD-25`. DEC-150→153. **Sprint S40 CLOS** |
| **SE-063.b** | hors sprint | **Cleaning et compression de la Knowledge.** 12 documents réécrits, création de `DECISIONS_ARCHIVE.md` (satellite hors ordre de lecture), `AUDIT.md` enrichi en append. `DECISIONS.md` était collé deux fois dans lui-même (717 → 159 lignes). 15 informations périmées corrigées, dont la séquence de boot fausse depuis `J11b-1` et deux règles opposables d'`AGENTS.md` contradictoires. DEC-154 | **Aucun code.** Corpus 3441 → 2432 lignes (−29 %), ordre de lecture −37 % |

---

## 📦 Versions

> Une ligne par bump. Un bump = une clôture de sprint.

| Version | Sprint | Livré |
|---|---|---|
| ≤ v0.21.0 | ≤ S21 | Migration vanilla → Vue 3, US-PINIA, US-JST, CI |
| — | S23→S27 | ⚠️ Détail non capturé |
| v0.28.0 | S28 | Epic Stats |
| v0.29.0 | S29→S33 | Refonte nav · US-127 · US-AUTH-LOGOUT |
| v0.30.0 | S36 | US-E2E-REGISTRY-RESYNC · US-SCROLL-387 · US-MODAL-CENTER-AUDIT |
| v0.31.0 | S37 | US-GRID-FIX · US-MODAL-UNIFY annulée (DEC-110) |
| v0.32.0 | S38 | AUD-01 · AUD-02 · registre E2E · purge debug |
| v0.33.0 | S39 | Migration AniList `J02`→`J07` · recherche · synopsis nettoyé · `AUD-21` |
| **v0.34.0** | **S40** | Migration AniList `J09`→`J11b` — **Jikan intégralement retiré** — + `J12` harnais E2E AniList |
