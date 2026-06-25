# EPICS.md — Taxonomie produit Aanime
> MAJ : S21. Lentille : EPIC = page/surface/système. Tag US = `[EPIC][SECTION][TYPE]`.
> Une US vit dans 1 EPIC. Les TAGS TYPE sont transverses.
> Ce fichier contient l'historique complet depuis la migration (S0→S21).

## Tags TYPE (transverses)
`[FEATURE]` `[UX]` `[CSS]` `[TEST]` `[PERF]` `[DETTE]` `[A11Y]` `[CI]`

---

## ═══ ÉPICS ACTIFS ═══

### EPIC On Air — Calendrier
Sections : `[WEEK]` `[MONTH]` `[LIST]`
- US-150 [ON AIR][WEEK][UX] Snap-to-today — ✅
- US-142 [ON AIR][WEEK][UX] Barre de progression sous la carte — ✅
- US-141 [ON AIR][WEEK][UX] Bouton ✓ marquer-vu 1 tap — ✅ (style → US-141-CSS)
- US-144 [ON AIR][WEEK][UX] CTA « Explore this season » jours vides — ✅ S20
- US-141-CSS [ON AIR][WEEK][CSS] Styliser `.rc-mark-done` (cible 44px) — ⬜
- BUG-10 [ON AIR][WEEK][DETTE] Suggestions slot-fill intermittentes — ⬜

### EPIC Discover — Découvrir
Sections : `[FOR-YOU]` `[SEASON]` `[COMING-SOON]`
- US-P0.3a/b/c [DISCOVER][*][DETTE] Déduplication pools — ✅ S9
- F14 [DISCOVER][SEASON][UX] Skeleton boot — ✅ clos sans dev (loader US-155 suffit)
- F16 [DISCOVER][FOR-YOU][FEATURE] Because You Watched — ✅ déjà implémenté

### EPIC Library — Bibliothèque
Sections : `[PLAN]` `[COMPLETED]` `[UPCOMING]`
- US-120 [LIBRARY][COMPLETED][UX] Badge vault = « Finished » — ✅ S14
- US-124 [LIBRARY][DETTE] MAL `Dropped` → non importé (vérifier filtre) — ⬜
- F15 [LIBRARY][UPCOMING][CSS] Titre BYW collé, typo incohérente — ⬜

### EPIC Recherche
- US-145a [RECHERCHE][UX] Suggestions enrichies année·studio·★score — ✅ S20
- US-145b [RECHERCHE][FEATURE] Bouton « + » direct, routage statut — ✅ S20

### EPIC Modal
- US-119 [MODAL][FEATURE] Covers relations enrichies — ✅ S14

### EPIC Navigation
- US-P0.5 [NAV][UX] Sous-nav Week/Month/List — ✅ S9
- US-P0.6-ter [NAV][UX] États actifs sous-onglets (F9) — ✅ S9

### EPIC Login & Authentification
- US-P0.7 [LOGIN][UX] Page login stylée — ✅ S9
- US-122 [LOGIN][FEATURE] Magic-link UI in-app — ✅ S14
- DEC-82 [LOGIN][FEATURE] Redirect post-login route d'origine — 🗄️ Vault

### EPIC Boot & Démarrage ← EPIC-BOOT complet S21
Sections : `[BOOT]` `[PERSIST]` `[SYNC]`
- US-153 [BOOT][PERSIST][FEATURE] saveToDatabase try/catch + toast — ✅ S19
- US-155 [BOOT][UX] Boot non bloquant (loader instantané) — ✅ S19
- US-154 [BOOT][PERSIST][DETTE] getCardStatus 'Continuing' → Airing — ✅ S19
- US-156a/b [BOOT][SYNC][TEST] specs useEpisodeInfo + useSync — ✅ S19
- US-157 [BOOT][PERSIST][DETTE] Mutations store via actions — ✅ S21
- US-158 [BOOT][PERSIST][DETTE] Legacy normalisé, zéro double cast — ✅ S21

### EPIC Onboarding & Rétention
- US-140 [ONBOARDING][FEATURE] 1ʳᵉ visite : genres → animes → calendrier — ⬜ **levier n°1**

### EPIC Moteur de Recommendation
Sections : `[RECCARD]` `[ENGINE]` `[BYW]` `[MORE-LIKE-THIS]`
- US-167 [REC][ENGINE][FEATURE] Bouton ❓ + RecEngineModal — ✅ S19
- US-152 [REC][MORE-LIKE-THIS][FEATURE] more-like-this → modal (Option A) — ✅ S21
- US-131 [REC][WEEK][FEATURE] Slot-fill Skip + clic→modal — ✅ S15
- US-131-E2E [REC][WEEK][TEST] E2E slot-fill skip + clic→modal — ⬜
- US-127 [REC][SYNC][FEATURE] SyncIndicator tous fetches Jikan (option B) — ⬜
- US-165 [REC][ENGINE][DETTE] Extraire fetchTopFinishedAnime → useJikanApi — ⬜ trivial
- US-152B [REC][MORE-LIKE-THIS][FEATURE] more-like-this inline (Option B) — 🗄️ Stockée

### EPIC Plateforme & Dette Technique
- US-159 [PLATEFORME][CI][DETTE] gitignore reports + purge parasites — ✅ S20
- US-166-CSS [PLATEFORME][CSS][DETTE] Dette CSS groupée F18–F24 — ⬜
- F8 [PLATEFORME][CSS][A11Y] Dark mode lisibilité (sous-nav, logo) — ⬜

### EPIC Évolution Majeure — Horizon
- Monétisation, stats « Mon année », notifs épisode, Library en chips — ⬜ idées

### EPIC Plateforme Long Terme (ex EPIC-5)
- US-160 PWA service worker + manifest — 🗄️
- US-161 Firestore onSnapshot temps réel — 🗄️
- US-162 Virtualisation listes longues — 🗄️
- US-163 Accessibilité focus trap, clavier, aria — 🗄️
- File Jikan anti-429 · Monitoring Sentry · TTL cache aanime_ · Redirect post-login — 🗄️

---

## ═══ ÉPICS CLOS — Historique ═══

### EPIC-1 — Stabilisation migration ✅ CLOS S6
> Filet CI + 4 correctifs runtime + suppression vanilla + dark mode + nav dédupliquée.
- US-101 Suppression code vanilla du repo — ✅
- US-102 [P0] Correctifs reco runtime (Add mort, clic carte mort) — ✅
- US-106 Throttle Jikan — ✅
- US-107 Hiatus detection (`isOnHiatus`) — ✅
- US-109 CI GitHub Actions — ✅

### EPIC P0 — Correctifs UX ✅ CLOS S12 (audit live PO, R6, DEC-77→80)
> Modal morte, dédup, RecCard actions, layout Month, sous-navs, login,
> toasts destination visible, auto-vault toasts, snap, marquer-vu, barre progression.
> 26 specs E2E cumulatives au moment de la clôture.
- US-P0.1 [BOOT] Clé localStorage `aanime_calendar` — ✅ (DEC-64, puis DEC-85)
- US-P0.2 [MODAL] Modal morte réparée — ✅
- US-P0.3a/b/c [DISCOVER] Déduplication pools Season/Reco/For You — ✅
- US-P0.4 [MODAL] Toasts destination visible (Add → « On Air / Plan to Watch / … ») — ✅ (DEC-63)
- US-P0.4-bis Harmonisation libellés toasts existants — ✅
- US-P0.5 [NAV] Sous-nav Week/Month/List — ✅
- US-P0.6 [CALENDAR] Layout Month — ✅
- US-P0.6-bis [CALENDAR] Libellé date dupliqué — ✅
- US-P0.6-ter [NAV] États actifs sous-onglets (F9) — ✅
- US-P0.7 [LOGIN] Page login stylée — ✅
- US-P0.8a [REC] RecCard `@add` câblé — ✅ (DEC-61)
- US-P0.8b [REC] RecCard `@click`→modal + `@not-interested`→dismiss — ✅ (DEC-61)
- US-P0.8c [REC] `@more-like-this` → reporté US-152, livré S21
- US-P0.9 [MODAL] Modal centrée (`.modal-backdrop` CSS) — ✅ (DEC-70)
- US-121 [BOOT] Auto-vault toast au boot — ✅
- US-141 [ON AIR] Bouton ✓ marquer-vu 1 tap — ✅
- US-142 [ON AIR] Barre progression — ✅
- US-150 [ON AIR] Snap-to-today — ✅ (DEC-76)

### EPIC-2 — Fiabilité & industrialisation ✅ CLOS S13/S14
> Code-splitting lazy routes + chunk Firebase, défer Firestore, build ~717 kb.
- Lazy routes + chunk Firebase — ✅
- Défer init Firestore (ROI 1er chargement) — ✅
- `useScrollKeeper` CalendarWeek — ✅

### EPIC-3 — Dette fonctionnelle héritée ✅ CLOS S15
- US-118 Pool réactif suggestions calendrier — ✅ *(BUG-10 intermittent)*
- US-119 Covers relations enrichies modal — ✅
- US-120 Badge vault = « Finished » — ✅
- US-122 Magic-link UI in-app — ✅
- US-123 Renommage badges RecCards — ✅
- US-131 Slot-fill Skip session-only + clic→modal (DEC-83) — ✅
- US-132 episodeOverride reset + POSTER_PLACEHOLDER unique + onHiatus? supprimé (DEC-84) — ✅
- US-133 Clés `aanime_*` + migration legacy (DEC-85) — ✅
- US-134 Studios normalisés `studios: string[]` (DEC-86) — ✅

### EPIC Boot — Sprint audit S16 ✅ CLOS S19/S21
> Dual audit S16 (DEC-87) → 7 US correctifs. Toutes livrées S19 (US-153→156) + S21 (US-157→158).
> Voir détail dans section EPIC Boot & Démarrage ci-dessus.
