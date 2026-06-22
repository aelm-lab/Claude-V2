# HANDOFF S16 → S17 — Aanime

> **À coller tel quel comme PREMIER message de la conversation S17.**
> Claude (Tech Lead + PM + rédacteur specs) reprend l'état ci-dessous.
> **Remplace `HANDOFF_SESSION10.md` (archivé).**

---

## 0. DÉMARRAGE S17 — ordre impératif

1. Confirme la lecture de ce handoff.
2. **La Knowledge a été intégralement régénérée en S16** (14 docs à jour). Tu peux t'y fier.
3. Affiche le Kanban (§4) et **lance directement US-153 (P0)** — c'est la prochaine action validée.
4. Au démarrage Gemini : exiger la liste des fichiers modifiés AVANT toute action (R-SCOPE-1).

---

## 1. Identité & cadre

- **Claude** = Tech Lead + Project Manager + Rédacteur de specs. 3 casquettes simultanées.
- **Adnane (moi)** = Product Owner **non-technique**. Je priorise, transmets les retours Gemini, valide. Je ne code pas, je ne tranche pas une question technique seul.
- **Gemini AI Studio** = agent d'implémentation, **sans accès à la Knowledge** → chaque US autoportante.
- **Langue : français.** Ton : **franchise radicale**, concis, prêt au copier-coller.
- **RÈGLE PERMANENTE** : pour **chaque US et chaque décision technique**, fournir **(a) l'impact utilisateur concret** + **(b) la recommandation Claude**.
- **Une seule US In Progress.** Pas de MERGE → pas de suite. Max 3 fichiers/US (dépassement annoncé **en gras**).
- **Plan + validation avant toute implémentation.** Gemini liste les fichiers modifiés **avant** d'agir.
- **À chaque US lancée, anticiper la suivante** (poser les questions de décision de l'US d'après).

---

## 2. Règles permanentes (gouvernance) — R1 → R6 + R-SCOPE-1

- **R1 — MERGE = triple preuve verte** : `vue-tsc --noEmit` + `vitest run` + `build`, **3 sorties brutes SÉPARÉES** (jamais chaînées, jamais paraphrasées). Paraphrase de build = review suspendue (4 récidives au compteur).
- **R2 — Test runtime** sur tout ce qui touche boot / store / câblage de composables.
- **R3 — Un audit lit le CODE réel**, pas les indicateurs verts. Zéro-confiance **y compris sur les diagnostics de Claude** et sur les handoffs (cf. les 3 fantômes corrigés en s16).
- **R4 — Tout correctif UX/écran → E2E Playwright** : geste réel + DOM visible (jamais le store). ROUGE avant fix, VERT après sans modifier le test. Pattern boot : attendre `#boot-loader` hidden avant tout clic. Seed 7 jours. **R4-bis** : tout `v-if` sur un élément interactif → grep des specs E2E qui le cliquent, même US.
- **R5 — E2E ciblé par US pendant l'epic, grand check en fin d'epic, specs cumulatives.** Sandbox : `--workers=1`, lots `batch1..3` (≤9 specs) + `sweep`.
- **R6 — Audit PO live obligatoire avant clôture d'un epic.**
- **R-SCOPE-1 — Zéro fichier parasite.** Gemini liste les fichiers modifiés AVANT d'agir. Violation la plus coûteuse du projet.
- **Diagnostic-before-spec** : grep / lecture seule AVANT toute proposition de fix.
- **Zero-trust preuves Gemini** : un « X passed » peut être structurellement impossible (variable non déclarée, fonction jamais appelée). Vérifier la cohérence interne du test.

---

## 3. Stack & état technique (fin S16)

- **Stack** : Vue 3 (Composition API) + TypeScript strict + Pinia + Vue Router 4 + Vite + Firebase/Firestore + @vueuse/core. Persistence : Firestore + localStorage (cache).
- **Tests** : **84 unit** (Vitest) · **26 specs E2E / 30 tests** (Playwright, batch1..3, `--workers=1`).
- **Build** : **~717 kb** (index ~420 + firebase esm ~452), ~3.7 s. Zéro `any`.
- **Clés localStorage** : toutes préfixées **`aanime_`** depuis US-133 (migration legacy au boot dans `usePersistence.loadFromDatabase`). Registre complet dans `ARCHITECTURE_TECHNIQUE.md §7`. ⚠️ Clé persistance = **`aanime_calendar`** (plus `'animeCalendar'`).

### ⚠️ 3 faits gravés par l'audit s16 (ne pas réintroduire les fantômes)
- `setAllData` **n'existe pas** → seul `setData` (+ `clearAll`).
- `syncStatus` **n'existe pas** dans `AnimeEntry` (0 hit).
- `reconcileWithDatabase` **n'existe plus** → la réconciliation est dans `loadFromDatabase`.

---

## 4. Kanban — état fin S16

### ✅ Done
- **Migration Phases 0→7** + **EPIC-1** (4 bugs runtime) + **EPIC P0** (UX, clos s12 audit live) + **EPIC-2** (perf/fiabilité, s13/14) + **EPIC-3** (dette fonctionnelle, s15 : US-131/132/133/134).
- **S16 dual audit** (Claude Code + Gemini, cadre identique) → backlog priorisé + régénération complète des 14 docs Knowledge.

### 🔄 In Progress
_(rien)_

### 📝 To Do — Sprint correctifs audit s16 (PRIORITÉ 1)
| US | Gravité | Sujet |
|---|---|---|
| **US-153** | 🔴 P0 | `saveToDatabase`/`saveSchedule` sans try/catch → sauvegarde silencieuse |
| US-154 | 🟠 P1 | `getCardStatus` `'Continuing'` → `Airing` (affiché « Finished ») |
| US-155 | 🟠 P1 | Boot non bloquant (overlay levé après load local) |
| US-156 | 🟠 P1 | Tests unit composables (`useEpisodeInfo`, `useSync`…) |
| US-157 | 🟠 P1 | `usePersistence` mute le store hors action + toasts dedans |
| US-158 | 🟠 P1 | Cast legacy `as unknown as AnimeEntry[]` non normalisé |
| US-159-CLEANUP | 🟢 P2 | Fichiers parasites racine (R-SCOPE-1) |

### 📝 To Do — EPIC-4 (produit + dette glissée)
US-127 (SyncIndicator, décidé option B non implémenté) · US-124 (MAL Dropped non importé, DEC-81) · US-165 (extraire `fetchTopFinishedAnime`) · US-131-E2E · US-140 (onboarding) · US-141-CSS · US-144 (états vides) · US-145 (recherche enrichie) · US-152 (more-like-this, décision produit) · US-166-CSS (dette F18–F24).

### 🐞 Non-P0
BUG-10 (suggestions On Air intermittent) · F14 skeletons · BUG-3/6/8/9.

### 🗄️ Vault fonctionnalités
TTL cache `aanime_*` · redirect post-login route d'origine (DEC-82) · bouton « Comment marche le Rec Engine » (pré-requis doc niveau 2).

---

## 5. Synthèse dual audit s16 (détail dans AUDIT.md §3)

- **Le dual audit a prouvé sa valeur** : chaque auditeur a raté le finding n°1 de l'autre (cadre identique = ce qui révèle les angles morts).
- **5 vérifs zéro-confiance tranchées par code** : US-153 (P0) et US-154 (P1) confirmés ; `setAllData`/`syncStatus`/`reconcileWithDatabase`/idb dynamique = rejetés/inexistants ; `boot-loader.remove()` = légitime.
- **Décisions intégrées** : DEC-81→86 (s15) + DEC-87 (méthode + résolution audit).
- **Sur le vécu user, 2 corrections comptent** : US-153 (perte silencieuse) et US-154 (label mensonger). US-155 = seule douleur ressentie au boot.

---

## 6. Première action S17 attendue de Claude

> « Lecture du handoff confirmée. 84 unit / 26 E2E, build ~717 kb. Knowledge à jour (régénérée s16).
> EPIC P0/2/3 clos. On attaque le sprint correctifs audit : **US-153 (P0)** — `saveToDatabase`
> sans try/catch → sauvegarde Firestore silencieuse. Je rédige la spec autoportante (try/catch +
> état d'erreur réactif + toast d'échec + test runtime R2), avec impact user + reco. Voici le plan,
> valide avant que je passe à Gemini. »

Anticiper US-154 en parallèle (le mapping `'Continuing'` + test unit du mapping).

---

## 7. Doc Rec Engine niveau 2 (sur demande — coller 2 fichiers)

Pour la doc fonctionnelle exhaustive du moteur (chaque poids, seuil, condition) qui alimentera
le bouton « Comment marche le Rec Engine », Claude a besoin de :
1. `src/utils/recEngine.ts` (`buildNextBatch`, `scorePool`, `buildTasteProfile`, `RelationMemory`, poids).
2. `src/composables/useRecommendations.ts` (orchestration, pools, cache, `getSlotFillSuggestions`, `extractBecauseYouWatched`).

Pipeline niveau 1 (connu) : `fetchRecPool → buildTasteProfile → buildRelationMemory → scorePool → applyPreset → getNextBatch → extractBecauseYouWatched / getSlotFillSuggestions`.

---

## 8. Carte des docs (tous régénérés s16)

`CLAUDE.md` (bible) · `TYPES_CONTRACT.md` · `DECISIONS.md` (DEC-01→87) · `ARCHITECTURE_TECHNIQUE.md` ·
`ARCHITECTURE_FONCTIONNELLE.md` · `BACKLOG.md` · `ROADMAP.md` · `AGENTS.md` · `AGENTS_E2E.md` ·
`ANTIPATTERNS.md` · `AUDIT.md` (3 parties) · `AUDIT_UX_SESSION7.md` · `PLAN_MIGRATION.md` (figé, historique) ·
`HANDOFF_SESSION16.md` (ce fichier). Archivés : `HANDOFF_SESSION5→10`.
