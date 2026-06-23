# ROADMAP.md — Epics post-migration

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Statut :** migration ✅ · EPIC-1 ✅ · EPIC P0 ✅ (s12) · EPIC-2 ✅ (s13/14) · EPIC-3 ✅ (s15).
> **84** unit · **26 specs / 30 tests** E2E · build **~717 kb**, ~3.7 s, zéro `any`.
>
> ⚠️ **Note d'historique :** l'audit initial (Claude Code, s4) avait conclu « saine » en ratant
> 4 bugs runtime. L'audit croisé Gemini (s6) les a trouvés. Le **dual audit s16** (cadre identique
> imposé aux deux) a confirmé cette valeur : chacun a vu un bug que l'autre a raté (cf. AUDIT.md §3).

---

## EPIC-1 — Clôture migration « v2 stable » ✅ CLOS (s6)
Filet CI (US-109) + 4 correctifs runtime (US-102 P0 reco, US-106 throttle, US-107 hiatus) + suppression vanilla (US-101) + dark mode + nav dédupliquée + `AGENTS.md` musclé. Détail `AUDIT.md` Partie 2.

## EPIC P0 — Correctifs UX ✅ CLOS (s7→s12)
Modal morte, dédup (Season/recherche/For You), RecCard Add/clic/dismiss, layout Month, sous-navs + états actifs, login stylé, modal centrée, toasts destination visible, auto-vault toasts, snap-to-today, marquer-vu 1 tap, barre progression. **Clôture formelle s12** via audit live PO (R6). 26 specs E2E cumulatives.

## EPIC-2 — Fiabilité & industrialisation ✅ CLOS (s13/14)
Code-splitting (lazy routes + chunk Firebase), défer Firestore (ROI 1er chargement), fiabilité. Build ~717 kb (index ~420 + firebase esm ~452). *(Reste en backlog perf : file Jikan globale anti-429, monitoring Sentry, `useScrollKeeper` pour les pages hors CalendarWeek.)*

## EPIC-3 — Dette fonctionnelle héritée ✅ CLOS (s15)

| US | Description | Statut |
|---|---|---|
| US-118 | Pool réactif suggestions calendrier | ✅ *(BUG-10 intermittent en backlog)* |
| US-119 | Covers relations enrichies modal | ✅ |
| US-120 | Badge vault = « Finished » | ✅ |
| US-122 | Magic-link UI in-app (input email LoginPage) | ✅ |
| US-123 | Renommage badges RecCards | ✅ |
| US-131 | Slot-fill Skip (session-only) + clic→modal (DEC-83) | ✅ *(E2E à ajouter)* |
| US-132 | Cleanup : `episodeOverride` reset + `POSTER_PLACEHOLDER` unique + `onHiatus?` supprimé (DEC-84) | ✅ |
| US-133 | Clés `aanime_*` + migration (DEC-85) | ✅ |
| US-134 | Studios résolu (`studios: string[]`, DEC-86, résout P8-01) | ✅ |

> Conflits de numéros (US-120/122/123 réutilisés) résolus dans `BACKLOG.md § Conflits de numéros`.

---

## 🔴 SPRINT CORRECTIFS AUDIT s16 — PRIORITÉ 1

> Issu du dual audit (DEC-87 / AUDIT.md §3). Ordre : risque silencieux d'abord, puis ressenti UX.

| US | Description | Gravité | Impact utilisateur |
|---|---|---|---|
| US-153 | `saveToDatabase`/`saveSchedule` → try/catch + état d'erreur + toast échec | 🔴 P0 | Perte de données silencieuse si Firestore échoue |
| US-154 | `getCardStatus` : `'Continuing'` → `Airing` + test unit | 🟠 P1 | Show en cours affiché « Finished » (mensonge) |
| US-155 | Boot non bloquant (overlay levé après load local, sync en fond) + E2E R4 | 🟠 P1 | Spinner plusieurs secondes au démarrage |
| US-156 | Tests unit composables critiques (`useEpisodeInfo`, `useSync`…) | 🟠 P1 | Aucun visible — cause racine de US-154 |
| US-157 | `usePersistence` : mutations via actions store + toasts sortis de la couche | 🟠 P1 | Aucun visible — couplage |
| US-158 | Chemin legacy : normalisation + garde runtime (plus de double cast) | 🟠 P1 | Cache corrompu → cartes incomplètes (rare) |
| US-159-CLEANUP | Suppression fichiers parasites racine (R-SCOPE-1) | 🟢 P2 | Aucun — dette |

**Reco séquencement** : « risque max éliminé » → US-153 → US-154 → US-157 → US-156 ; OU « ressenti max » → US-153 → US-155 → US-154. P2 mono-source groupés en fin (cf. BACKLOG).

---

## EPIC-4 — Expérience & rétention 🟢 BACKLOG PRODUIT

| US | Description | Impact | Statut |
|---|---|---|---|
| US-127 | SyncIndicator : couvrir tous les fetches Jikan significatifs (option B) | Confiance données | ⬜ Décidé, non implémenté |
| US-124 | MAL `Dropped` → non importé (DEC-81) — vérifier filtre `malImport` | Bibliothèque propre | ⬜ |
| US-165 | Extraire `fetchTopFinishedAnime` → `useJikanApi` (ex-US-123) | Aucun — dette | ⬜ Trivial |
| US-131-E2E | Couvrir slot-fill skip + clic→modal | Aucun — filet de test | ⬜ |
| US-140 | Onboarding 1ère visite (3 genres → 5 animes → calendrier) | Levier rétention n°1 | ⬜ |
| US-141-CSS | Style `.rc-mark-done` (cible tactile 44px) | UX bouton ✓ | ⬜ |
| US-144 | États vides actionnables (CTA « Explorer la saison ») | Zéro cul-de-sac | ⬜ |
| US-145 | Recherche enrichie : année, studio, score + bouton « + » direct | Parcours d'ajout ÷2 | ⬜ |
| US-152 | P0.8c `more-like-this` : option A (modal) vs B (section inline). Décision produit | Découverte | ⬜ |
| US-166-CSS | Dette CSS groupée F18–F24 | Aucun — dette | ⬜ |

*Idées EPIC-4 non priorisées : dark mode `prefers-color-scheme`, indicateur « ✓ Synchronisé / ⚠ Local », notifications « ton épisode sort aujourd'hui », page stats « Mon année anime », Library en chips.*

---

## 🗄️ Vault fonctionnalités — idées validées, reportées

- **TTL sur le cache `aanime_*`** (expiration auto 24 h). Complexité vs gain déjà couvert par `useSync`.
- **Redirect post-login vers la route d'origine** (DEC-82, ROI faible).
- **Bouton « Comment marche le Rec Engine »** : explication mot-par-mot + organigramme. *Transparence → confiance.* Pré-requis = doc Rec Engine niveau 2.

---

## EPIC-5 — Plateforme 🔵 LONG TERME

| US | Description |
|---|---|
| US-160 | PWA : service worker + manifest → installable + offline |
| US-161 | Firestore temps réel (`onSnapshot`) → multi-appareils sync |
| US-162 | Virtualisation listes longues (vault 500+) |
| US-163 | Accessibilité : focus trap modals, navigation clavier, aria |

---

## Priorisation recommandée (Tech Lead) — état s16

1. **Sprint correctifs audit s16** : US-153 (P0) en premier, puis P1 dans l'ordre risque/ressenti.
2. **Sprint dette** : US-159-CLEANUP + US-166-CSS + US-141-CSS.
3. **Sprint produit** : US-140 (onboarding, levier n°1) + US-144 + US-145.
4. **Sprint fiabilité résiduelle** : file Jikan anti-429 + monitoring Sentry.
5. Ensuite : EPIC-4 restant / Vault fonctionnalités selon retours.
