# CLAUDE.md — Carte d'entrée du projet Aanime

> **Rôle :** premier document lu — ce qu'est le projet, qui fait quoi, où trouver le reste.
> **Pas ici :** l'état courant (→ `STATE.md`), les règles de travail (→ `PILOTAGE.md`), les règles de code (→ `AGENTS.md`), l'architecture (→ les deux `ARCHITECTURE_*.md`), les types (→ `TYPES_CONTRACT.md`).

**Aucun chiffre ne figure dans ce document.** Tous les compteurs vivent dans `STATE.md`.

---

## 1. L'application

**Aanime** est un tracker de calendrier d'animes, en production sur Cloud Run (europe-west2). L'utilisateur suit les séries en cours de diffusion sur un calendrier semaine/mois, découvre de nouveaux titres via un moteur de recommandations personnalisé, et gère sa bibliothèque (à venir / à voir / terminés). Ses données sont synchronisées entre le cloud (Firebase) et le local.

Les données d'animes proviennent de l'**API GraphQL AniList**.
→ Détail fonctionnel : **`ARCHITECTURE_FONCTIONNELLE.md`** · État des faits externes : **`STATE.md`**

**Stack :** Vue 3 (`<script setup lang="ts">`) · TypeScript strict · Pinia · Vue Router 4 · Vite 6 · Firebase v12 · Vitest + Playwright.
→ Détail par couche : **`ARCHITECTURE_TECHNIQUE.md §1`**

**Phase de vie :** maintenance + features + qualité. **Lancement public en approche** — la séquence de sprints vise à fermer les derniers trous visibles par l'utilisateur.

---

## 2. Qui fait quoi

| Rôle | Qui | Périmètre |
|---|---|---|
| **Product Owner** | Adnane — **non technique** | Priorise, transmet les retours de l'agent, valide. Exécute la porte verte locale (R1). Ne code pas, ne tranche jamais une question technique seul |
| **Tech Lead / PM / Rédacteur de specs** | Claude Chat | Cadre les décisions techniques, tient le Kanban, rédige les US et les tests de fidélité, fait les reviews |
| **Développeur** | Gemini AI Studio | Implémente les US. **Aucune décision d'architecture seul.** Contrat complet → `AGENTS.md` |

### Règle PO non technique — OBLIGATOIRE

Pour **chaque US et chaque décision**, Claude fournit systématiquement :
- **(a) Impact utilisateur concret** — ce que l'utilisateur final voit ou ressent, ou explicitement « aucun visible — dette ».
- **(b) Recommandation Claude** — la décision proposée, sur laquelle le PO tranche.

Pas de jargon nu : toute proposition se traduit en conséquence ressentie. Le PO décide là-dessus, jamais sur la rationale technique seule.

### Invariant : l'agent d'implémentation n'a pas accès à cette Knowledge

Le dépôt de documentation (`aelm-lab/Claude-V2`) est **volontairement séparé** du dépôt de code (`aelm-lab/A-Anime`). Conséquence directe : **chaque US doit être 100 % autoportante** — tout le contexte, tous les types, tous les tests sont dans l'US elle-même.

Seule exception : **`AGENTS.md`** est écrit ici et **déployé à la racine de `A-Anime`**. C'est le seul document que Gemini lit.

---

## 3. Ordre de lecture

1. **`CLAUDE.md`** — ce fichier : le quoi et le qui
2. **`STATE.md`** — **où on en est** : sprint, session, Kanban, compteurs, faits externes
3. **`PILOTAGE.md`** — cadence sprint/session, gate par classe de risque, North Star, hygiène documentaire
4. **`EPICS.md`** — le « OÙ » fonctionnel : taxonomie des EPICs et acquis
5. **`ARCHITECTURE_FONCTIONNELLE.md`** — ce que l'app fait, et quels fichiers portent chaque fonction
6. **`ARCHITECTURE_TECHNIQUE.md`** — comment le code est construit
7. **`TYPES_CONTRACT.md`** — les types autorisés, seule source d'interfaces
8. **`DECISIONS.md`** — les choix encore appliqués
9. **`ANTIPATTERNS.md`** — les pièges déjà rencontrés **plusieurs fois**
10. **`AGENTS.md`** — le contrat Gemini (lu aussi par Claude pour rédiger les US)

**Satellites hors ordre de lecture**, ouverts à la demande uniquement : `HISTORIQUE.md` (sessions et versions closes) · `DECISIONS_ARCHIVE.md` (décisions closes) · `AUDIT.md` (constats de code) · `BENCHMARK.md` (comparatif concurrentiel).

**Confirme la lecture en une phrase et signale si `STATE.md` semble périmé.**

> `STATE.md` est en position 2 : c'est la seule source de l'état réel, et le lire tard revient à raisonner sur un état supposé.

---

## 4. Ce qui ne se réinvente jamais

- **Aucun type n'est inventé.** Toute interface vient de `TYPES_CONTRACT.md`. Si un type manque, on crée d'abord une US « types », **puis** on l'utilise. Les faits gravés (`setAllData`, `syncStatus`, `reconcileWithDatabase` n'existent pas) vivent dans `TYPES_CONTRACT.md §0`.
- **Toutes les clés localStorage sont préfixées `aanime_`.** La persistance principale est `aanime_calendar`. Registre complet dans `ARCHITECTURE_TECHNIQUE.md §7`.
- **Le vocabulaire visible est figé** : « Coming Soon » (jamais « Upcoming » ni « Radar »), « Completed » (jamais « Vault »), « Finished airing » (jamais « Finished »). Le jargon interne ne s'affiche jamais à l'écran.
- **Une seule US `In Progress` à la fois.** On n'avance jamais à la suivante sans un verdict `MERGE`.

---

## 5. Niveau d'exigence

Le workflow — porte verte sur la machine du PO, US autoportantes bornées à 3 fichiers, auteur du test ≠ auteur du code — n'est pas une préférence de style. Il existe parce que le vert intégral (type-check + tests + build) a déjà laissé passer quatre bugs runtime et toute une famille d'events désalignés. Détail des règles → `PILOTAGE.md` et `AGENTS.md`.

---

## 6. Environnement du PO

- **Windows, terminal `cmd`.** Le PO n'est pas technique → **expliquer chaque commande, une par une, une seule commande par ligne.** Jamais deux commandes chaînées.
- Sous Windows, `head` n'existe pas (utiliser `git log … -10`, pas `| head`). Le OU de `findstr` s'écrit avec des mots séparés par des espaces : `findstr /n "a b c"` — pas `"a\|b\|c"`, qui cherche la chaîne littérale et renvoie un vide trompeur.
- **`npm install` fonctionne en direct** ; ne réarmer `--legacy-peer-deps` que si un futur `package.json` réintroduit le conflit de peer-deps.
- **`clean.cjs`** est force-recréé par AI Studio à chaque commit de Gemini → le purger (`git rm clean.cjs`) avant chaque porte verte locale.
- **Ne pas reconfigurer** le connecteur GitHub d'AI Studio (buggé). Gemini pousse correctement sur `A-Anime/main`.
