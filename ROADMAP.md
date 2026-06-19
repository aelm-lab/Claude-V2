# ROADMAP.md — Epics post-migration

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Statut migration :** ✅ VALIDÉE (session 6). 76 tests verts, zéro `any`, build ~4.7s.
> **Statut EPIC P0 :** ✅ CLOS côté technique (session 10). Audit live PO = session 11.
>
> ⚠️ **Note d'historique :** l'audit initial (Claude Code, s4) avait conclu « saine, 64/66 »
> en ratant 4 bugs runtime. L'audit croisé Gemini (s6) les a tous trouvés et corrigés
> (détail `AUDIT.md` Partie 2). Tests réels : 66/66 (+10 smoke/unit = 76/76).

---

## EPIC-1 — Clôture migration « v2 stable » ✅ CLOS (session 6)

| US | Description | Statut |
|---|---|---|
| US-109 | Filet CI : `ci.yml` + `App.spec.ts` smoke test boot | ✅ MERGE |
| US-102 | 🔴 P0 — `buildRelationMemory` + `reScorePool` rebranchés au boot — DEC-50 | ✅ MERGE |
| US-106 | 🟠 P1 — Throttle Jikan conditionnel réseau (`*WithMeta`) — DEC-51 | ✅ MERGE |
| US-107 | 🟠 P1 — Hiatus source unique 14j — DEC-52 | ✅ MERGE |
| US-101 | Suppression 31 fichiers vanilla morts | ✅ MERGE |
| US-104 | `body.dark-mode` → `html.dark` — DEC-23 résiduel | ✅ MERGE |
| US-105 | Déduplication nav controls | ✅ MERGE |
| US-110 | `AGENTS.md` musclé — DEC-53 | ✅ MERGE |
| ~~US-103~~ | ~~Fixer 2 tests timezone~~ → faux positif, annulée | ❌ |

---

## EPIC P0 — Correctifs UX (EPIC P0) ✅ CLOS côté technique (sessions 7→10)

> 22 specs E2E cumulatives. Audit live PO planifié session 11 pour clôture formelle.

| US/Item | Description | Session | Statut |
|---|---|---|---|
| P0.0/P0.0-bis | Socle Playwright + bypass auth + isolation runners | s7 | ✅ |
| P0.1 | Modal morte réparée (event-name désaligné) — DEC-58 | s7 | ✅ |
| P0.2 | Boot loader visible (2 phases) — DEC-59 | s8 | ✅ |
| P0.3a | Dédup This Season (`dedupeByMalId`) — DEC-60 | s8 | ✅ |
| P0.3b | Dédup For You batch (batch sortant, moteur intouché) — DEC-74 | s10 | ✅ |
| P0.3c | Dédup recherche — DEC-60 | s8 | ✅ |
| P0.4 | Toast feedback ajout depuis modal — DEC-63 | s8 | ✅ |
| P0.4-bis | Harmonisation libellés toasts (→ destination visible) — DEC-71 | s9 | ✅ |
| P0.5 | Sous-nav On Air Week/Month | s9 | ✅ |
| P0.6 / P0.6-bis/ter/quater | Layout Month + doublon période + états actifs navs + layout secondary nav | s9 | ✅ |
| P0.7 | Login stylé — DEC-68 | s9 | ✅ |
| P0.8a/b | RecCard Add + clic + dismiss câblés — DEC-61 | s8 | ✅ |
| **P0.8c** | `@more-like-this` non câblé | — | → US-152 EPIC-4 (décision produit) |
| P0.9 | Modal anime centrée (`.modal-backdrop` CSS) — DEC-70 | s9 | ✅ |
| US-121 | Auto-vault muet → 2 toasts séparés « Moved to Completed » — DEC-73 | s10 | ✅ |
| DEC-72 | Dette boot-loader résorbée (loader hors `#app`, E2E réparée) | s10 | ✅ |

---

## EPIC-2 — Fiabilité & industrialisation 🟠 PRIORITÉ 1

> CI/CD déjà livrée (US-109). `savedScrollY` dans plusieurs pages : désormais résolu en
> partie par US-150 (snap-to-today dans CalendarWeekPage). US-115 reste pour les autres pages.

| US | Description | Estimation |
|---|---|---|
| US-111 | Tests E2E : parcours critiques (login→ajout→calendrier, recs→heart→library, import MAL) | 2-3 US |
| US-112 | Monitoring erreurs prod (Sentry) — remplace `console.error` isolés | 1 US petite |
| US-113 | Code-splitting : lazy routes + chunk Firebase + fix warning `idb.ts` (716kb → 2-3× plus rapide) | 1 US moyenne |
| US-114 | **File Jikan globale** (p-queue) : anti-429 sur navigation rapide | 1 US moyenne |
| US-115 | `useScrollKeeper` : extraire `savedScrollY` restant dans les autres pages | 1 US petite |
| US-117 | Défer Firestore au 1er chargement (ROI visibilité boot — F2 durée) | 1 US moyenne |

---

## EPIC-3 — Dette fonctionnelle héritée 🟡 PRIORITÉ 2

| US | Origine | Description | Estimation |
|---|---|---|---|
| US-120 | P8-01 | Bug studios inerte (`scorePool` lit `studios`, normalize produit `studio`) | 1 US petite |
| ~~US-121~~ | P8-02 | ~~Auto-vault silencieux~~ | ✅ LIVRÉ s10 |
| US-122 | P8-03 | Re-saisie email magic link → input dans LoginPage (pas `window.prompt`) | 1 US moyenne |
| US-123 | P8-04 | `fetchTopFinishedAnime` inline → extraire vers `useJikanApi` | 1 US triviale |
| US-124 | P8-05 | Mapping MAL Dropped→vault : décision PO requise | 1 US petite |
| US-125 | P8-08 | Cache localStorage unifié : convention `aanime_*` + couche TTL | 1-2 US |
| US-126 | P8-09 | Redirect post-login vers route d'origine | 1 US petite |
| US-127 | P8-10 | SyncIndicator : couvrir tous les fetches Jikan (`jikanInFlight` counter) | 1 US petite |
| US-128 | US-039 | Vérifier reset `episodeOverride: undefined` dans upsert store | 1 US triviale |
| US-129 | US-038 | Harmoniser POSTER_PLACEHOLDER (4 copies → `constants.ts`) | 1 US petite |
| US-130 | US-107 | Nettoyer type `onHiatus?` (plus écrit depuis US-107) | 1 US triviale |
| — | s10 | **Dette CSS F18–F24** : `.test-*` mortes, doublons `.post-it`, hacks `:has()` morts, `#app-loading-overlay`, `.month-header-mobile`, jargon « Vault » empty state, `BecauseYouWatched` `<style scoped>` | 1 US CSS groupée |

---

## EPIC-4 — Expérience & rétention 🟢 BACKLOG PRODUIT

| US | Description | Impact | Statut |
|---|---|---|---|
| US-140 | Onboarding 1ère visite : 3 genres → 5 animes → calendrier pré-rempli | Levier rétention n°1 | ⬜ |
| US-141 | « Marquer vu » en 1 tap depuis le calendrier | Friction ÷3 | ✅ LIVRÉ s10 — fonctionnel, non stylé (→ US-141-CSS après audit) |
| US-141-CSS | Style `.rc-mark-done` : positionnement + cible tactile 44px | UX bouton ✓ | ⬜ Après audit s11 |
| US-142 | Barre de progression par carte (WeekAnimeItem uniquement — AnimeCard hors scope) | Lisibilité immédiate | ✅ LIVRÉ s10 |
| US-143 | Signaux recos visibles (`_signals`/`_triggerTitle`) | Confiance | ✅ DÉJÀ IMPLÉMENTÉ — fermé sans dev s10 |
| US-144 | États vides actionnables (CTA « Explorer la saison ») | Zéro cul-de-sac | ⬜ |
| US-145 | Recherche enrichie : année, studio, score + bouton « + » direct | Parcours d'ajout ÷2 | ⬜ |
| US-146 | Dark mode : `prefers-color-scheme` par défaut + transition douce | Feel natif | ⬜ |
| US-147 | Indicateur « ✓ Synchronisé / ⚠ Local » | Confiance données | ⬜ |
| US-148 | Notifications « ton épisode sort aujourd'hui » | Habitude hebdo | ⬜ |
| US-149 | Page stats « Mon année anime » (Spotify Wrapped) | Engagement + partage | ⬜ |
| US-150 | Snap-to-today (auto-ancrage jour courant à l'ouverture) | Repère immédiat | ✅ LIVRÉ s10 — test E2E faible, audit visuel s11 |
| US-151 | Library en chips : fusionner Completed/Plan to Watch via pills | Navigation fluide | ⬜ |
| US-152 | P0.8c `more-like-this` : option A (modal, gratuit) vs option B (section inline). 2 mockups générés s9. Décision PO requise. | Découverte | ⬜ Décision produit |

---

## EPIC-5 — Plateforme 🔵 LONG TERME

| US | Description |
|---|---|
| US-160 | PWA : service worker + manifest → installable + offline |
| US-161 | Firestore temps réel (`onSnapshot`) → multi-appareils sync |
| US-162 | Virtualisation listes longues (vault 500+) |
| US-163 | Accessibilité : focus trap modals, navigation clavier, aria |

---

## Priorisation recommandée (Tech Lead) — état session 10

1. **Session 11 :** audit live PO → clôture formelle EPIC P0 → specs E2E post-audit.
2. **Sprint dette :** US-141-CSS (bouton ✓ stylé) + passe CSS F18–F24 (US groupée) + US-144 (états vides).
3. **Sprint perf :** US-117 (défer Firestore, ROI 1er chargement) + US-113 (code-split, 716kb → 2-3× plus rapide).
4. **Sprint fiabilité :** US-114 (file Jikan anti-429) + US-111 (E2E parcours critiques).
5. **Sprint rétention :** US-140 (onboarding, levier n°1) + US-145 (recherche enrichie).
6. Ensuite : alternance EPIC-3 (dette fonctionnelle) / EPIC-4 (produit) selon retours audit.
