# CLAUDE.md — Carte d'entrée du projet Aanime

> **Rôle :** premier document lu — ce qu'est le projet, qui fait quoi, où trouver le reste.
> **Aucun chiffre ici.** Tous les compteurs vivent dans `STATE.md`.

---

## 1. L'application

**Aanime** est un tracker de calendrier d'animes, en production sur Cloud Run (europe-west2),
**en bêta avec des testeurs réels**. L'utilisateur suit les séries en cours sur un calendrier
semaine/mois, découvre des titres via un moteur de recommandations personnalisé, et gère sa
bibliothèque (à venir / à voir / terminés). Ses données sont synchronisées entre Firebase et le
local. Les données d'animes viennent de l'**API GraphQL AniList**.

**Stack :** Vue 3 (`<script setup lang="ts">`) · TypeScript strict · Pinia · Vue Router 4 ·
Vite 6 · Firebase v12 · Vitest + Playwright.

**Phase de vie :** maintenance + features + qualité, **lancement public en approche**.

---

## 2. Qui fait quoi

| Rôle | Qui | Périmètre |
|---|---|---|
| **Product Owner** | Adnane — **non technique** | Priorise, transmet les retours, valide. Exécute la porte verte locale (R1). Ne code pas, ne tranche jamais une question technique seul |
| **Tech Lead / PM / Specs** | Claude Chat | Cadre les décisions techniques, tient le Kanban, rédige les US et les tests de fidélité, fait les reviews |
| **Développeur** | Gemini AI Studio | Implémente les US. **Aucune décision d'architecture seul.** Contrat → `AGENTS.md` |

### Règle PO non technique — OBLIGATOIRE

Pour **chaque US et chaque décision**, Claude fournit systématiquement :
- **(a) Impact utilisateur concret** — ce que l'utilisateur voit ou ressent, ou explicitement « aucun visible — dette ».
- **(b) Recommandation Claude** — la décision proposée, sur laquelle le PO tranche.

Pas de jargon nu. Le PO décide sur la conséquence ressentie, jamais sur la rationale technique seule.

### Invariant : Gemini n'a pas accès à cette Knowledge

Le dépôt de documentation (`aelm-lab/Claude-V2`) est **volontairement séparé** du dépôt de code
(`aelm-lab/A-Anime`). Conséquence : **chaque US est 100 % autoportante** — tout le contexte, tous
les types, tous les tests sont dans l'US. Seule exception : **`AGENTS.md`**, écrit ici et
**déployé à la racine de `A-Anime`**. C'est le seul document que Gemini lit.

---

## 3. Ordre de lecture

**Les 14 documents sont chargés dans la Knowledge du projet.** Cet ordre dit dans quel sens les
lire, pas lesquels sont disponibles.

1. `CLAUDE.md` — ce fichier : le quoi et le qui
2. `STATE.md` — **où on en est** : sprint, session, Kanban, compteurs, faits externes
3. `HISTORIQUE.md` — **le passé et le tampon de la session en cours** (§Tampon avant tout raisonnement d'état)
4. `PILOTAGE.md` — cadence, gate par classe de risque, North Star, cadence documentaire
5. `ARCHITECTURE_FONCTIONNELLE.md` — ce que l'app fait, taxonomie des EPICs, pont fonctionnel ▶ technique
6. `ARCHITECTURE_TECHNIQUE.md` — comment le code est construit
7. `TYPES_CONTRACT.md` — les types autorisés, seule source d'interfaces
8. `DECISIONS.md` — les choix encore appliqués
9. `ANTIPATTERNS.md` — les pièges déjà rencontrés **plusieurs fois**
10. `AGENTS.md` — le contrat Gemini (lu aussi par Claude pour rédiger les US)

**Ouverts à la demande :** `AUDIT.md` (constats de code) · `ROADMAP.md` (cap multi-sprints) ·
`DECISIONS_ARCHIVE.md` (décisions closes) · `BENCHMARK.md` (comparatif concurrentiel).

> 🔴 **`STATE.md` peut dater de plusieurs sessions en cours de sprint** (`DEC-190`). L'état réel =
> `STATE.md` **+** `HISTORIQUE.md §Tampon`. Lire les deux, toujours, dans cet ordre.

**Confirme la lecture en une phrase, et signale toute contradiction entre deux documents.**

---

## 4. Environnement du PO

- **Windows, terminal `cmd`.** Le PO n'est pas technique → **une commande par ligne, expliquée.** Jamais deux commandes chaînées.
- `head` n'existe pas (`git log … -10`, pas `| head`).
- 🔴 **`findstr` a deux syntaxes à ne pas confondre :**
  - **chaîne littérale** → `findstr /s /n /c:"sync it later"`. Sans `/c:`, les mots sont cherchés **séparément** (coût mesuré en SE-074 : ~800 lignes déversées dans la conversation).
  - **OU logique** → `findstr /n "a b c"`, mots séparés par des espaces. Jamais `"a\|b\|c"`, qui cherche la chaîne littérale et renvoie un vide trompeur.
- **`npm install`** fonctionne en direct ; ne réarmer `--legacy-peer-deps` que si `package.json` réintroduit le conflit.
- **`clean.cjs`** est force-recréé par AI Studio à chaque commit de Gemini → `git rm clean.cjs` avant chaque porte verte.
- **Ne pas reconfigurer** le connecteur GitHub d'AI Studio (buggé). Gemini pousse correctement sur `A-Anime/main`.
