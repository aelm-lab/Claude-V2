# BACKLOG.md — Kanban vivant

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Epics & estimations →** ROADMAP.md. **Audit UX →** AUDIT_UX_SESSION7.md. **Dual audit →** AUDIT.md §3.
> **État de référence : session 16 (dual audit).** EPIC P0 + EPIC-2 + EPIC-3 **clos**.
> **84** unit · **26 specs / 30 tests** E2E (batch1..3, `--workers=1`) · build **~717 kb**.

---

## 📋 Kanban

### ✅ Done — Migration (Phases 0→7) + EPIC-1 (remédiation post-audit)
- Voir HANDOFF_SESSION6 (archivé). Migration close, 4 bugs runtime corrigés, tag v2-stable.

### ✅ Done — Sessions 7→12 (EPIC P0 — UX)
- Sessions 7/8/9/10/12 : modal morte, dédup (Season/recherche/For You), RecCard Add/clic/dismiss, layout Month, sous-navs, login stylé, modal centrée, toasts destination visible, auto-vault toasts, snap-to-today, marquer-vu 1 tap, barre progression. **EPIC P0 → CLOS s12** (audit live PO, R6). Détail dans HANDOFF archivés + ROADMAP.

### ✅ Done — Sessions 13/14 (EPIC-2 + amont EPIC-3)
- **EPIC-2 clos** : code-split / défer Firestore / fiabilité (build ~717 kb).
- [US-118] Pool réactif suggestions calendrier *(effet de bord : BUG-10 intermittent, non prioritaire)*
- [US-119] Covers relations enrichies (modal)
- [US-120 — livré] Badge vault = « Finished » *(≠ US-120 ROADMAP d'origine, cf. § conflits)*
- [US-122 — livré] Magic-link UI in-app (input email dans `LoginPage`)
- [US-123 — livré] Renommage badges RecCards *(≠ US-123 ROADMAP d'origine, cf. § conflits)*

### ✅ Done — Session 15 (EPIC-3 — clôture)
- [US-131] Slot-fill cards : bouton **Skip** (session-only, `ref<Set<number>>` local — DEC-83) + clic carte → modal. Composant `WeekSuggestionCard.vue`. *(E2E à ajouter — voir To Do.)*
- [US-132] Cleanup groupé (DEC-84) : `episodeOverride` reseté à l'upsert · `POSTER_PLACEHOLDER` → `constants.ts` (4 copies supprimées) · `onHiatus?` supprimé du type
- [US-133] Clés localStorage `aanime_*` + migration legacy au boot (DEC-85)
- [US-134] Bug studios résolu : `normalizeAnime` produit toujours `studios: string[]` (DEC-86, résout P8-01)

**EPIC-3 → CLOS (décision PO s15).**

### ✅ Done — Session 16 (dual audit)
- [DUAL AUDIT] Claude Code + Gemini, cadre identique → synthèse convergences/divergences + backlog priorisé (DEC-87). 5 vérifs zéro-confiance tranchées par code.
- [DOCS] Régénération complète de la Knowledge (lots 7+4+3).

### 🔄 In Progress
_(rien)_

### 📝 To Do — Correctifs issus du dual audit s16 (prioritaire)

> Détail, impact user et reco dans `AUDIT.md §3`. Ordre de levier : P0 d'abord (silencieux = le plus dangereux), puis ressenti UX.

- **[US-153] 🔴 P0** — `usePersistence.saveToDatabase` : `saveSchedule` sans try/catch → échec Firestore silencieux. *Impact : on croit avoir sauvegardé, le cloud a échoué, aucun signal.* Fix : try/catch + état d'erreur + toast d'échec + test runtime R2.
- **[US-154] 🟠 P1** — `getCardStatus` mappe `'Continuing'` → `Airing` (au lieu de `Finished`). *Impact : un show en cours s'affiche « Finished ».* + test unit du mapping.
- **[US-155] 🟠 P1** — Boot non bloquant : lever l'overlay après le load local, sync en fond. *Impact : spinner de plusieurs secondes au démarrage.* + E2E R4.
- **[US-156] 🟠 P1** — Tests unit composables critiques (commencer `useEpisodeInfo` + `useSync`). *Impact : aucun visible — cause racine de US-154 non détecté.* À étaler.
- **[US-157] 🟠 P1** — `usePersistence` : passer les mutations du store par des actions (plus de `store.x =` direct), sortir les toasts de la couche persistance. *Impact : aucun visible — couplage.*
- **[US-158] 🟠 P1** — Chemin legacy : normaliser + garde runtime au lieu de `as unknown as AnimeEntry[]`. *Impact : cache corrompu → cartes incomplètes (rare).*
- **[US-159-CLEANUP] 🟢 P2** — Supprimer fichiers parasites racine (`diff.cjs`, `replace.js`, `size.cjs`, `find_usages.cjs`, `sme.json`, `*_out.txt`, `test_pid.txt`). R-SCOPE-1.

**À vérifier avant classement définitif (1 ligne de grep)** : re-save Firestore inutile au boot (`setData` au Temps 1 déclenche le watch → écriture des données qu'on vient de charger). Touche coûts/quotas Firestore. → finding P2 mono-source (Claude Code F12), non confirmé.

**P2 mono-source en backlog (à grouper par fichier)** : stubs morts `_syncAnimeUpdates`/`_startBackgroundRelationFetch` · `onNavigate` cast partiel `AnimeModal` · `status as AnimeStatus` cast direct `normalize` · `syncAnimeUpdates()` sans await dans `AnimeModal` · worker fond bloqué si throw (`backgroundWorkerRunning` jamais reset) · exports morts `fetchAnimeRecommendations`/`fetchUpcomingSeason` · duplication `negativeIds` ×4 · cache relations sans TTL · zéro test composant `.vue` · erreurs rate-limit avalées en `console.warn`.

### 📝 To Do — EPIC-4 (produit + dette glissée)

- **[US-127]** SyncIndicator : couvrir tous les fetches Jikan significatifs (sync principale + `fetchRecPool` + `fetchAnimeRelations`), exclure les enrichissements covers en fond. **Décidé option B en s15, jamais implémenté.**
- **[US-124]** Mapping MAL `Dropped` → non importé (DEC-81). **Vérifier** si `malImport` filtre déjà ; sinon 1 US.
- **[US-165]** (ex-US-123 sujet original) `fetchTopFinishedAnime` inline → extraire vers `useJikanApi`. Triviale.
- **[US-131-E2E]** Couvrir le slot-fill skip + clic→modal par un E2E R4 (manquant depuis s15).
- **[US-166-CSS]** Dette CSS groupée F18–F24 : `.test-*` mortes, doublons `.post-it`, hacks `:has()` morts, `#app-loading-overlay`, `.month-header-mobile` orpheline, jargon « Vault » empty state, `BecauseYouWatched.vue` `<style scoped>` pré-DEC-72.
- **[US-141-CSS]** Style `.rc-mark-done` (positionnement + cible tactile 44px) — bouton fonctionnel mais brut.
- **[US-140]** Onboarding 1ère visite (3 genres → 5 animes → calendrier pré-rempli). Levier rétention n°1.
- **[US-144]** États vides actionnables — jours vides muets (F10), CTA « Explorer la saison ».
- **[US-145]** Recherche enrichie : année/studio/score + bouton « + » direct (F13).
- **[US-152]** P0.8c `@more-like-this` RecCard : option A (modal, gratuit) vs B (section inline). 2 mockups s9. Décision produit.

### 🐞 Bugs / chantiers non-P0
- **BUG-10** suggestions On Air (jours vides) intermittent après US-118 — aléatoire, non prioritaire.
- **F14 skeletons** : ~6 s de blanc au boot à re-mesurer (SkeletonCard existe, inutilisé).
- Bugs résiduels : BUG-3, BUG-6, BUG-8, BUG-9.

### 🗄️ Vault fonctionnalités (post-EPIC — idées validées, reportées)
- **TTL sur le cache localStorage `aanime_*`** (expiration auto 24 h) — complexité vs gain déjà couvert par `useSync`.
- **Redirect post-login vers la route d'origine** (DEC-82).
- **Bouton « Comment marche le Rec Engine »** (explication mot-par-mot + organigramme). Pré-requis = doc Rec Engine niveau 2. *Impact : transparence → confiance.*

### 🗂️ En standby (long terme — EPIC-5)
- PWA (service worker + manifest), Firestore temps réel (`onSnapshot`), virtualisation listes longues, accessibilité (focus trap, clavier, aria).

---

## 🔢 Conflits de numéros — résolution s16

> Les numéros US-120/122/123 ont été **réutilisés** pour des sujets différents entre la table
> EPIC-3 du ROADMAP d'origine et ce qui a réellement été livré en s14. Ground-truth = ce qui
> a **shippé**. Mapping de réconciliation :

| Numéro | ROADMAP d'origine (EPIC-3) | Réalité livrée | Résolution |
|---|---|---|---|
| US-120 | Bug studios inerte | Badge vault = « Finished » (s14) | Studios = **US-134** (s15). US-120 garde « badge ». |
| US-122 | Re-saisie email magic-link | Magic-link UI in-app (s14) | Même sujet → cohérent. |
| US-123 | Extraire `fetchTopFinishedAnime` | Renommage badges RecCards (s14) | Badges gardent US-123. Extraction renumérotée **US-165** (EPIC-4). |
| US-125 | Cache `aanime_*` + TTL | Préfixe `aanime_` livré (US-133) | TTL → Vault fonctionnalités. |
| US-126 | Redirect post-login | Décidé « reste `/` » (DEC-82) | → Vault fonctionnalités. |
| US-128 | Reset `episodeOverride` | Livré dans US-132 (DEC-84) | Absorbé. |
| US-129 | `POSTER_PLACEHOLDER` unifié | Livré dans US-132 (DEC-84) | Absorbé. |
| US-130 | Nettoyer `onHiatus?` | Livré dans US-132 (DEC-84) | Absorbé. |

---

## Règles de tenue du backlog
- **Une seule US In Progress.** Pas de MERGE, pas de suite.
- US > 3 fichiers → scinder + prévenir le PO. Dépassement autorisé SI annoncé **en gras + dans le titre**.
- **Impact utilisateur + reco Claude** sur chaque US et décision (PO non-technique).
- **Zéro confiance** : code brut intégral + sortie terminale brute. Y compris diagnostics de Claude.
- **R1** triple preuve verte (vue-tsc + vitest run + build), **3 sorties brutes séparées** (jamais de paraphrase build — 4 récidives).
- **R2** test runtime sur boot/store/câblage de composables.
- **R3** un audit lit le code. Lire le code qui marche AVANT de proposer un fix.
- **R4** tout correctif UX / feature touchant l'écran → E2E Playwright geste réel + DOM visible. ROUGE→VERT sans modif. Pattern boot : attendre `#boot-loader` hidden. Seed 7 jours.
- **R5** test ciblé par US pendant l'epic, grand check complet en fin d'epic. Tests cumulatifs. Sandbox : `--workers=1`, lots `batch1..3` de ≤9 specs + `sweep`.
- **R6** audit PO live obligatoire avant clôture d'un epic.
- **Diagnostic avant spec** : grep lecture seule d'abord.
- **Démarrage Gemini** : exiger l'état des fichiers modifiés AVANT toute action (récidive s10 : 5 fichiers modifiés sans US).
- **gating ↔ E2E** : tout `v-if` ajouté sur un bouton/lien → grep des specs E2E qui cliquent cet élément, dans la même US (leçon s12).
- Livraison sans contenu intégral = review suspendue.
