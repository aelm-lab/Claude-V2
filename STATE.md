# STATE.md — Suivi vivant du projet Aanime

> **Où vit ce fichier :** repo Knowledge `aelm-lab/Claude-V2` (branche `main`) → resync auto
> dans la Knowledge Claude. **Remplace `BACKLOG.md`** (supprimer l'ancien pour éviter 2 sources).
> **Rôle :** ground-truth de l'avancement. Macro (epics) + micro (US). Mis à jour à chaque fin de session.
>
> **État :** migration ✅ · EPIC-1 ✅ (s6) · EPIC P0 ✅ (s12) · EPIC-2 ✅ (s13/14) · EPIC-3 ✅ (s15) ·
> dual audit s16 ✅ · SGit (pont GitHub + porte locale) ✅.
> **Métriques (fin S16) :** ~84 unit (Vitest) · 26 specs / 30 tests E2E (Playwright) · build ~717 kb, ~3.7 s, zéro `any`.
> **Dernière MAJ :** S18.

---

## 📋 Kanban — vue rapide

### ✅ Done
- Migration Phases 0→7 · EPIC-1 · EPIC P0 · EPIC-2 · EPIC-3 · dual audit s16 + Knowledge régénérée (14 docs)
- ⭐ SGit : pont GitHub + porte verte locale
- S18 : instructions perso régénérées · diagnostic jst (bug DST confirmé) · version Pinia identifiée (`@pinia/testing 0.1.7`)

### 🔄 In Progress (une seule US active à la fois — ici 2 entrelacées car la 1ʳᵉ ne peut verdir sans la 2ᵉ)
- **[US-PINIA]** Downgrade `@pinia/testing` → `0.1.7` — livré par Gemini, **en attente porte locale PO** (vert global bloqué tant que jst rouge)
- **[US-JST]** `parseJSTToLocal` DST-aware + test déterministe (TZ Europe/Paris) — **à envoyer à Gemini**

### 📝 To Do — Sprint correctifs audit s16 (PRIORITÉ 1)
Voir détail ci-dessous. Ordre de levier : risque silencieux d'abord, puis ressenti UX.

---

## 🔧 Infrastructure (rappel)
- **Code :** `aelm-lab/A-Anime`, branche `main` (Gemini pousse). Clone local PO suit `main`.
- **Knowledge :** `aelm-lab/Claude-V2`, branche `main`, connecteur GitHub natif (resync auto).
- **Vanilla de référence :** `aelm-lab/aanime` — intact, jamais touché.
- **Install :** `npm install` (sans `--legacy-peer-deps`) une fois US-PINIA mergée ; sinon parade `--legacy-peer-deps`.
- **CI :** `.github/workflows/ci.yml` — à re-vérifier après Pinia+jst ; retirer la branche morte `feat/vue3-migration` du trigger.

---

## 🔴 SPRINT CORRECTIFS AUDIT s16 — PRIORITÉ 1

> Issu du dual audit (DEC-87 / AUDIT.md §3). Reco séquençage : « risque max éliminé »
> → US-153 → US-154 → US-157 → US-156 ; OU « ressenti max » → US-153 → US-155 → US-154.

| US | Gravité | Sujet | Impact utilisateur | Statut |
|---|---|---|---|---|
| **US-PINIA** | 🔧 infra | `@pinia/testing` → 0.1.7 | Aucun (débloque CI + supprime `--legacy-peer-deps`) | 🔄 livré, attente porte locale |
| **US-JST** | 🔴 bug | `parseJSTToLocal` ancré 1970 → décalage DST | Horaires faux d'1h ~7 mois/an, mauvais jour près de minuit | 🔄 à envoyer |
| **US-153** | 🔴 P0 | `saveToDatabase`/`saveSchedule` sans try/catch | **Perte de données silencieuse** si Firestore échoue | ⬜ prochaine session (spec R2 soignée) |
| **US-154** | 🟠 P1 | `getCardStatus` ne mappe pas `'Continuing'` | Show en cours affiché « Finished » (mensonge) | ⬜ |
| **US-155** | 🟠 P1 | Boot non bloquant (overlay levé après load local, sync en fond) + E2E R4 | Spinner plusieurs secondes au démarrage | ⬜ |
| **US-156** | 🟠 P1 | Tests unit composables (`useEpisodeInfo`, `useSync`…) | Aucun visible — cause racine de US-154 | ⬜ à étaler |
| **US-157** | 🟠 P1 | `usePersistence` : mutations via actions store + toasts sortis de la couche | Aucun visible — couplage | ⬜ |
| **US-158** | 🟠 P1 | Chemin legacy : normaliser + garde runtime (plus de double cast) | Cache corrompu → cartes incomplètes (rare) | ⬜ |
| **US-159-CLEANUP** | 🟢 P2 | Supprimer fichiers parasites racine + `.gitignore` (R-SCOPE-1) | Aucun — dette | ⬜ |

**À vérifier (1 grep) avant classement :** re-save Firestore inutile au boot (`setData` au Temps 1 déclenche le watch → réécriture des données chargées). Touche coûts/quotas Firestore. P2 mono-source non confirmé.

**P2 mono-source en backlog (grouper par fichier) :** stubs morts `_syncAnimeUpdates`/`_startBackgroundRelationFetch` · `onNavigate` cast partiel `AnimeModal` · `status as AnimeStatus` dans `normalize` · `syncAnimeUpdates()` sans await dans `AnimeModal` · worker fond bloqué si throw (`backgroundWorkerRunning` jamais reset) · exports morts `fetchAnimeRecommendations`/`fetchUpcomingSeason` · duplication `negativeIds` ×4 · cache relations sans TTL · zéro test composant `.vue` · erreurs rate-limit avalées en `console.warn`.

---

## 🟢 EPIC-4 — Expérience & rétention (backlog produit + dette glissée)

| US | Sujet | Impact | Statut |
|---|---|---|---|
| US-127 | SyncIndicator : couvrir tous les fetches Jikan significatifs (option B) | Confiance données | ⬜ décidé s15, non implémenté |
| US-124 | MAL `Dropped` → non importé (DEC-81) — vérifier filtre `malImport` | Bibliothèque propre | ⬜ |
| US-165 | Extraire `fetchTopFinishedAnime` → `useJikanApi` (ex-US-123) | Aucun — dette | ⬜ trivial |
| US-131-E2E | Couvrir slot-fill skip + clic→modal (R4) | Aucun — filet de test | ⬜ |
| US-140 | Onboarding 1ère visite (3 genres → 5 animes → calendrier pré-rempli) | **Levier rétention n°1** | ⬜ |
| US-141-CSS | Style `.rc-mark-done` (cible tactile 44px) | UX bouton ✓ | ⬜ |
| US-144 | États vides actionnables (CTA « Explorer la saison ») | Zéro cul-de-sac | ⬜ |
| US-145 | Recherche enrichie : année/studio/score + bouton « + » direct | Parcours d'ajout ÷2 | ⬜ |
| US-152 | `more-like-this` RecCard : option A (modal) vs B (section inline) — décision produit | Découverte | ⬜ |
| US-166-CSS | Dette CSS groupée F18–F24 | Aucun — dette | ⬜ |

*Idées non priorisées : dark mode `prefers-color-scheme`, indicateur « ✓ Synchronisé / ⚠ Local », notifs « ton épisode sort aujourd'hui », page stats « Mon année anime », Library en chips.*

---

## 🐞 Bugs / chantiers non-P0
- **BUG-10** suggestions On Air (jours vides) intermittent après US-118 — aléatoire.
- **F14 skeletons** : ~6 s de blanc au boot (SkeletonCard existe, inutilisé).
- Résiduels : BUG-3, BUG-6, BUG-8, BUG-9.

## 🗄️ Vault fonctionnalités (idées validées, reportées)
- TTL cache `aanime_*` (24 h) — gain déjà couvert par `useSync`.
- Redirect post-login vers route d'origine (DEC-82, ROI faible).
- Bouton « Comment marche le Rec Engine » (explication + organigramme). *Transparence → confiance.*

## 🗂️ EPIC-5 — Standby long terme
PWA (service worker + manifest) · Firestore temps réel (`onSnapshot`) · virtualisation listes · accessibilité (focus trap, clavier, aria) · file Jikan globale anti-429 · monitoring Sentry.

---

## 🔢 Conflits de numéros — résolution s16 (ground-truth = ce qui a shippé)

| Numéro | ROADMAP d'origine (EPIC-3) | Réalité livrée | Résolution |
|---|---|---|---|
| US-120 | Bug studios inerte | Badge vault = « Finished » (s14) | Studios = US-134 (s15). US-120 garde « badge ». |
| US-122 | Re-saisie email magic-link | Magic-link UI in-app (s14) | Cohérent. |
| US-123 | Extraire `fetchTopFinishedAnime` | Renommage badges RecCards (s14) | Badges = US-123. Extraction renumérotée US-165. |
| US-125 | Cache `aanime_*` + TTL | Préfixe `aanime_` (US-133) | TTL → Vault. |
| US-126 | Redirect post-login | « reste `/` » (DEC-82) | → Vault. |
| US-128/129/130 | reset override / placeholder / onHiatus | Livrés dans US-132 (DEC-84) | Absorbés. |

---

## Règles de tenue du suivi
- **Une seule US In Progress.** Pas de MERGE, pas de suite.
- US > 3 fichiers → scinder + prévenir le PO. Dépassement autorisé SI annoncé **en gras + dans le titre**, **avec checks supplémentaires** (diff complet, grep d'impact, test de non-régression élargi).
- **Impact utilisateur + reco Claude** sur chaque US et décision (PO non-technique).
- **R1 = porte verte LOCALE** par le PO (3 sorties brutes : type-check / test:run / build). Preuves Gemini irrecevables.
- **R2** test runtime sur boot/store/câblage composables. **R3** zéro-confiance (lire le code, y compris diagnostics Claude). **R4** E2E geste réel + DOM visible, ROUGE→VERT sans modif, `#boot-loader` hidden, seed 7 jours. **R5** specs E2E cumulatives. **R6** audit PO live avant clôture d'epic. **R-SCOPE-1** Gemini liste les fichiers AVANT d'agir.
- Canon complet des règles : `AGENTS.md` (repo code). Ne pas tout redupliquer dans chaque US.

---

## ▶️ Prochaine action
1. Envoyer **US-JST** à Gemini.
2. Après application Pinia + jst : **porte locale unique** → 3 sorties brutes → double verdict MERGE.
3. Re-vérifier CI GitHub + nettoyer le trigger de branche morte.
4. **S19 :** US-153 (P0) — spec autoportante try/catch + état d'erreur réactif + toast échec + test runtime R2.
