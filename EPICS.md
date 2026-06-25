# EPICS.md — Taxonomie produit Aanime
> MAJ : S20. Lentille : EPIC = page/surface/système. Tag US = `[EPIC][SECTION][TYPE]`.
> Une US vit dans 1 EPIC (le OÙ). Les TAGS TYPE ([CSS][TEST][PERF][DETTE][A11Y][CI][FEATURE][UX])
> se posent en plus pour la vue transverse. Déplacer une US d'EPIC est permis quand le besoin évolue.

## Tags TYPE (transverses, sur n'importe quelle US)
`[FEATURE]` `[UX]` `[CSS]` `[TEST]` `[PERF]` `[DETTE]` `[A11Y]` `[CI]`

## Les 12 EPIC

### EPIC On Air — Calendrier
Sections : `[WEEK]` `[MONTH]` `[LIST]`
- US-144 [ON AIR][WEEK][UX] CTA « Explore this season » sur jours vides — ✅ MERGE S20
- US-150 [ON AIR][WEEK][UX] Snap-to-today — ✅
- US-142 [ON AIR][WEEK][UX] Barre de progression sous la carte — ✅
- US-141 [ON AIR][WEEK][UX] Bouton ✓ marquer-vu 1 tap — ✅ (style → US-141-CSS)
- US-141-CSS [ON AIR][WEEK][CSS] Styliser `.rc-mark-done` (cible 44px) — ⬜
- BUG-10 [ON AIR][WEEK][DETTE] Suggestions slot-fill intermittentes — ⬜

### EPIC Discover — Découvrir
Sections : `[FOR-YOU]` `[SEASON]` `[COMING-SOON]`
- US (P0.3a/b/c) [DISCOVER][*][DETTE] Déduplication pools — ✅
- F14 [DISCOVER][SEASON][UX] Skeleton boot — ✅ clos sans dev (loader US-155 suffit)
- F16 [DISCOVER][FOR-YOU][FEATURE] Because You Watched — ✅ déjà implémenté

### EPIC Library — Bibliothèque
Sections : `[PLAN]` `[COMPLETED]` `[UPCOMING]`
- US-120 [LIBRARY][COMPLETED][UX] Badge « Finished » — ✅
- US-124 [LIBRARY][DETTE] MAL `Dropped` → non importé (vérifier filtre) — ⬜
- F15 [LIBRARY][UPCOMING][CSS] Titre BYW collé, typo incohérente — ⬜

### EPIC Recherche
- US-145a [RECHERCHE][UX] Suggestions enrichies année·studio·★score — ✅ MERGE S20
- US-145b [RECHERCHE][FEATURE] Bouton « + » direct, routage statut — ✅ MERGE S20

### EPIC Modal
- US-119 [MODAL][FEATURE] Covers relations enrichies — ✅
- (more-like-this : voir EPIC Moteur de Recommendation / US-152)

### EPIC Navigation
- US-P0.5 [NAV][UX] Sous-nav Week/Month/List — ✅
- US-P0.6-ter / F9 [NAV][UX] États actifs sous-onglets — ✅

### EPIC Login & Authentification
- US-P0.7 [LOGIN][UX] Page login stylée — ✅
- US-122 [LOGIN][FEATURE] Magic-link UI in-app — ✅
- DEC-82 [LOGIN][FEATURE] Redirect post-login route d'origine — 🗄️ Vault

### EPIC Boot & Démarrage
Sections : `[BOOT]` `[PERSIST]` `[SYNC]`
- US-153 [BOOT][PERSIST][FEATURE] saveToDatabase try/catch + toast — ✅
- US-155 [BOOT][UX] Boot non bloquant (loader instantané) — ✅
- US-154 [BOOT][PERSIST][DETTE] getCardStatus mappe 'Continuing' — ✅
- US-156a/b [BOOT][SYNC][TEST] specs useEpisodeInfo + useSync — ✅
- US-157 [BOOT][PERSIST][DETTE] Mutations store via actions + toasts hors persistance — ⬜ P1 (pré-requis US-158)
- US-158 [BOOT][PERSIST][DETTE] Legacy normalisé, plus de double cast — ⬜ P1

### EPIC Onboarding & Rétention
- US-140 [ONBOARDING][FEATURE] 1ʳᵉ visite : 3 genres → 5 animes → calendrier — ⬜ **levier rétention n°1**

### EPIC Moteur de Recommendation
Sections : `[RECCARD]` `[ENGINE]` `[BYW]` `[MORE-LIKE-THIS]`
- US-167 [REC][ENGINE][FEATURE] Bouton ❓ + RecEngineModal (doc moteur) — ✅
- US-152 [REC][MORE-LIKE-THIS][FEATURE] more-like-this → **Option A décidée** (ouvre modal). À specs S21. B (inline) stockée. — ⬜ **PRÊTE**
- US-127 [REC][SYNC][FEATURE] SyncIndicator couvre tous fetches (option B) — ⬜
- US-165 [REC][ENGINE][DETTE] Extraire `fetchTopFinishedAnime` → useJikanApi — ⬜ trivial
- US-131-E2E [REC][WEEK][TEST] Couvrir slot-fill skip + clic→modal — ⬜

### EPIC Évolution Majeure — Horizon
- Monétisation, page stats « Mon année anime », notifs « épisode sort aujourd'hui », Library en chips — ⬜ idées

### EPIC Plateforme & Dette Technique — transverse sans page
- US-159-CLEANUP [PLATEFORME][CI][DETTE] gitignore reports + purge parasites — ✅ MERGE S20
- US-166-CSS [PLATEFORME][CSS][DETTE] Dette CSS groupée F18–F24 — ⬜
- F8 [PLATEFORME][CSS][A11Y] Dark mode lisibilité (sous-nav, logo) — ⬜
- [PLATEFORME][PERF] PWA, onSnapshot temps réel, virtualisation, file Jikan anti-429, Sentry — 🗄️ EPIC-5
