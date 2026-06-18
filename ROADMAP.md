# ROADMAP.md — Epics post-migration

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Remplace :** PHASE8_DEBT.md (intégré ici).
> **Statut migration :** ✅ VALIDÉE par **audit croisé** (session 6).
> 69/69 tests verts, zéro `any`, build prod ~4 s, architecture conforme.
>
> ⚠️ **Note d'historique :** l'audit initial (Claude Code, session 4) avait conclu
> « migration saine, 64/66 » en **ratant 4 bugs runtime**. L'audit croisé Gemini de
> session 6 les a tous trouvés et corrigés (détail dans `AUDIT.md` Partie 2). Les tests
> sont en réalité 66/66 (+3 smoke = 69/69) ; il n'y a jamais eu de dette test timezone.

---

## EPIC-1 — Clôture migration « v2 stable » ✅ CLOS (session 6)

> Objectif atteint : chantier 100 % clos, prêt pour tag `v2-stable`.

| US | Description | Statut |
|---|---|---|
| US-109 | **Filet** : `ci.yml` (vue-tsc+vitest+build) + `App.spec.ts` (smoke test boot). *A livré la CI/CD initialement planifiée US-110.* | ✅ MERGE |
| US-102 | 🔴 P0 — Rebrancher `buildRelationMemory` + `reScorePool` au boot (App.vue orchestre, stubs morts retirés de useSync) — DEC-50 | ✅ MERGE |
| US-106 | 🟠 P1 — Throttle Jikan conditionnel réseau (`*WithMeta`/`fromNetwork`) — DEC-51 | ✅ MERGE |
| US-107 | 🟠 P1 — Hiatus source unique 14j (suppression écriture morte 21j) — DEC-52 | ✅ MERGE |
| US-101 | Suppression des 31 fichiers vanilla morts | ✅ MERGE |
| US-104 | `style.css` : `body.dark-mode` → `html.dark` (DEC-23 résiduel) | ✅ MERGE |
| US-105 | Déduplication des nav controls (Prev/Next retirés des pages) | ✅ MERGE |
| US-110 | `AGENTS.md` musclé = gouvernance permanente Gemini — DEC-53. *Réversion : ce fichier devait être supprimé, il est conservé.* | ✅ MERGE |
| ~~US-103~~ | ~~Fixer 2 tests timezone~~ → **SUPPRIMÉE** (66/66 réels, faux positif audit A) | ❌ Annulée |

**Ménage repo restant avant le tag :** supprimer `PHASE8_DEBT.md` (→ ce fichier) et `HANDOFF_SESSION5.md` (→ HANDOFF_SESSION6.md).

## EPIC-2 — Fiabilité & industrialisation 🟠 PRIORITÉ 1 (prochain sprint)

> La CI/CD (ex-US-110) est **déjà livrée** via US-109. Reste :

| US | Description | Estimation |
|---|---|---|
| US-111 | Tests E2E Playwright : 3 parcours critiques (login→ajout→calendrier, recs→heart→library, import MAL) | 2-3 US moyennes |
| US-112 | Monitoring erreurs production (Sentry ou équivalent) au lieu de `console.error` isolés | 1 US petite |
| US-113 | Code-splitting : lazy routes + chunk Firebase séparé + fix warning chunking `idb.ts` (749 kb → premier écran 2-3× plus rapide) | 1 US moyenne |
| US-114 | **File Jikan globale** (singleton p-queue) : jamais plus d'1 appel sortant/s, anti-429 sur navigation rapide *(issu audit B)* | 1 US moyenne |
| US-115 | `useScrollKeeper(refEl)` : extraire la logique `savedScrollY` dupliquée dans plusieurs pages *(issu audit B)* | 1 US petite |
| US-116 | Polish : retirer le libellé de période **interne** aux pages calendrier (doublon résiduel de `CalendarNavControls`, post-US-105) | 1 US triviale |

## EPIC-3 — Dette fonctionnelle héritée (ex-P8) 🟡 PRIORITÉ 2

| US | Origine | Description | Estimation |
|---|---|---|---|
| US-120 | P8-01 | Bug studios inerte : `scorePool` lit `studios` (pluriel), normalize produit `studio`. Corriger + mesurer impact scores | 1 US petite |
| US-121 | P8-02 | Auto-vault silencieux → toast "Added to Completed" + undo éventuel | 1 US petite |
| US-122 | P8-03 | `window.prompt()` re-saisie email → état `needsEmailConfirmation` + input dans LoginPage | 1 US moyenne |
| US-123 | P8-04 | `fetchTopFinishedAnime` inline → extraire vers useJikanApi | 1 US triviale |
| US-124 | P8-05 | Mapping MAL Dropped→vault : trancher (watchlist ? badge ?) — décision PO requise | 1 US petite |
| US-125 | P8-08 | Cache localStorage unifié : convention `aanime_*` + couche TTL centralisée | 1-2 US moyennes |
| US-126 | P8-09 | Redirect post-login vers route d'origine (query param) | 1 US petite |
| US-127 | P8-10 | SyncIndicator : couvrir tous les fetches Jikan (`jikanInFlight` counter) | 1 US petite |
| US-128 | US-039 | Vérifier reset `episodeOverride: undefined` dans l'upsert store (clobber ?) | 1 US triviale |
| US-129 | US-038 | Harmoniser POSTER_PLACEHOLDER (4 copies → constants.ts) + prefetch covers si utile | 1 US petite |
| US-130 | US-107 | Nettoyer le type `onHiatus?` (plus écrit depuis US-107, cosmétique) | 1 US triviale |

## EPIC-4 — Expérience & rétention 🟢 BACKLOG PRODUIT

| US | Description | Impact |
|---|---|---|
| US-140 | Onboarding 1ère visite : 3 genres → 5 animes → calendrier pré-rempli | Levier rétention n°1 |
| US-141 | « Marquer vu » en 1 tap depuis le calendrier (vs 3 taps via modal) | Friction ÷3 sur l'action la plus fréquente |
| US-142 | Barre de progression par carte (Ep 7/12 visuel) | Lisibilité immédiate |
| US-143 | « Parce que tu as aimé X » visible sur les recs (les `_signals` existent déjà) | Confiance → cercle vertueux |
| US-144 | États vides actionnables (bouton « Explorer la saison ») | Zéro cul-de-sac |
| US-145 | Recherche enrichie : année, studio, score + bouton « + » direct | Parcours d'ajout ÷2 |
| US-146 | Dark mode : `prefers-color-scheme` par défaut + transition douce | Feel natif |
| US-147 | Indicateur « ✓ Synchronisé / ⚠ Local » | Confiance données |
| US-148 | Notifications « ton épisode sort aujourd'hui » (badge/email/push) | Habitude hebdo |
| US-149 | Page stats « Mon année anime » (Spotify Wrapped) | Engagement + partage |
| US-150 | **Snap-to-today** : auto-ancrage sur le jour courant à l'ouverture de la semaine *(issu audit B)* | Repère immédiat |
| US-151 | **Library en chips** : fusionner Completed/Plan to Watch via pills (préserve scroll, 1 clic) *(issu audit B)* | Navigation fluide |
| US-152 | more-like-this A/B, 2 mockups réf

## EPIC-5 — Plateforme 🔵 LONG TERME

| US | Description |
|---|---|
| US-160 | PWA : service worker + manifest → installable + offline |
| US-161 | Firestore temps réel (`onSnapshot`) → multi-appareils sync |
| US-162 | Virtualisation des longues listes (vault 500+) |
| US-163 | Accessibilité : focus trap modals, navigation clavier, aria |

## Priorisation recommandée (Tech Lead)

1. **Tag `v2-stable`** maintenant (EPIC-1 clos + ménage repo).
2. **Sprint sécurisation :** EPIC-2 (US-114 file Jikan + US-113 code-split en premier, ROI immédiat), puis US-111 E2E.
3. **Sprint rétention :** US-140 (onboarding) + US-141 (marquer-vu 1-tap) — les 2 leviers produit majeurs.
4. Ensuite : alternance EPIC-3 (dette) / EPIC-4 (produit) selon retours utilisateurs.
