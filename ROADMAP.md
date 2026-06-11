# ROADMAP.md — Epics post-migration

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Remplace :** PHASE8_DEBT.md (intégré ici).
> **Statut migration :** ✅ VALIDÉE (audit externe Claude Code, session 4).
> 64/66 tests verts, zéro any, build prod 2,5s, architecture conforme.

---

## EPIC-1 — Clôture migration « v2 stable » 🔴 PRIORITÉ 1

> Objectif : pouvoir déclarer le chantier 100 % clos. ~1 sprint.

| US | Description | Estimation |
|---|---|---|
| US-101 | Supprimer les ~30 fichiers vanilla morts (`src/views/*.js`, `src/ui/*.js`, `src/main.js`, `store.js`, `api.js`, `recs.js`, `persistence.js`…) + `/AGENTS.md` parasite | 1 US triviale |
| US-102 | Rebrancher `reScorePool` après synchro (stub `_reScorePool` dans useSync → orchestration App.vue, pattern DEC-25) | 1 US moyenne |
| US-103 | Fixer les 2 tests dépendants du fuseau (forcer TZ ou mocker la date — indépendance machine) | 1 US triviale |
| US-104 | `style.css` : migrer les sélecteurs `body.dark-mode` → `html.dark` (DEC-23 résiduel) | 1 US petite |
| US-105 | Dédupliquer les nav controls (CalendarWeek/MonthPage gardent leurs Prev/Next internes, doublons de CalendarNavControls) | 1 US petite |

## EPIC-2 — Fiabilité & industrialisation 🟠 PRIORITÉ 2

| US | Description | Estimation |
|---|---|---|
| US-110 | CI/CD GitHub Actions : vue-tsc + vitest + build à chaque commit (le filet qui aurait évité les 20 jours perdus) | 1 US petite, ROI maximal |
| US-111 | Tests E2E Playwright : 3 parcours critiques (login→ajout→calendrier, recs→heart→library, import MAL) | 2-3 US moyennes |
| US-112 | Monitoring erreurs production (Sentry ou équivalent) | 1 US petite |
| US-113 | Code-splitting : lazy loading des routes + chunk Firebase séparé (749kb → premier écran 2-3× plus rapide) | 1 US moyenne |

## EPIC-3 — Dette fonctionnelle héritée (ex-P8) 🟡 PRIORITÉ 3

| US | Origine | Description | Estimation |
|---|---|---|---|
| US-120 | P8-01 | Bug studios inerte : `scorePool` lit `studios` (pluriel), normalize produit `studio`. Corriger + mesurer impact scores | 1 US petite |
| US-121 | P8-02 | Auto-vault silencieux → toast "Added to Completed" + undo éventuel | 1 US petite |
| US-122 | P8-03 | `window.prompt()` re-saisie email → état `needsEmailConfirmation` + input dans LoginPage | 1 US moyenne |
| US-123 | P8-04 | `fetchTopFinishedAnime` inline → extraire vers useJikanApi | 1 US triviale |
| US-124 | P8-05 | Mapping MAL Dropped→vault : trancher (watchlist ? badge ?) — décision PO requise | 1 US petite |
| US-125 | P8-08 + Tech#8 | Cache localStorage unifié : convention `aanime_*` + couche TTL centralisée | 1-2 US moyennes |
| US-126 | P8-09 | Redirect post-login vers route d'origine (query param) | 1 US petite |
| US-127 | P8-10 | SyncIndicator : couvrir tous les fetches Jikan (`jikanInFlight` counter) | 1 US petite |
| US-128 | US-039 | Vérifier reset `episodeOverride: undefined` dans l'upsert store (clobber ?) | 1 US triviale |
| US-129 | US-038 | Harmoniser POSTER_PLACEHOLDER (4 copies locales → constants.ts) + restaurer prefetch covers relations si jugé utile | 1 US petite |

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

## EPIC-5 — Plateforme 🔵 LONG TERME

| US | Description |
|---|---|
| US-160 | PWA : service worker + manifest → installable + offline |
| US-161 | Firestore temps réel (`onSnapshot`) → multi-appareils sync |
| US-162 | Virtualisation des longues listes (vault 500+) |
| US-163 | Accessibilité : focus trap modals, navigation clavier, aria |

## Priorisation recommandée (Tech Lead)

1. **Sprint clôture :** EPIC-1 complet → tag « v2-stable »
2. **Sprint sécurisation :** US-110 (CI/CD) + US-140 + US-141
3. Ensuite : alternance EPIC-3 (dette) / EPIC-4 (produit) selon retours utilisateurs
