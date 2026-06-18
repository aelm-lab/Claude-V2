HANDOFF_SESSION9.md — Reprise de projet (EPIC P0 quasi-clos)

Où mettre ce fichier : dans la Knowledge du projet Claude Chat.

Remplace HANDOFF_SESSION8.md (à archiver).

Rôle : reprendre exactement où la session 9 s'est arrêtée, sans perte.


1. Où on en est
Migration Phases 0→7 : ✅ close. EPIC-1 : ✅ clos. EPIC P0 (correctifs UX) : quasi-clos — 8 US mergées en session 9, 2 dérivés + grand check restants.
Session 9 a livré le gros du sprint UX : sous-nav On Air, layout Month, login stylé, états actifs navs, layout secondary nav, harmonisation toasts. Elle a aussi révélé un nouveau bug P0 (modal en bas de page, non vu en session 8 faute d'audit visuel) et l'a corrigé (P0.9).
Fait en session 9 (mergé)
USObjetE2EP0.5Sous-nav On Air Week/Month + fix état actif PrimaryNav (F3)onair-subnav R/VP0.6Layout Month : CSS .weekday-headers-mobile grid + suppression doublon mois (US-116 volet 1)month-layout R/VP0.6-bisSuppression doublon libellé période CalendarWeekPage (US-116 ✅ close)week-no-duplicate-period R/VP0.6-terÉtat actif visible navs primaire + secondaire — classe .active alignée sur CSS (F9 ✅)nav-active-state R/V + màj onair-subnavP0.6-quaterLayout secondary nav On Air : onglets / contrôles de date séparés (flex-direction:column)calendar-subnav-layout R/VP0.7/login stylé : AuthLayout centré + branding + explainer flow (F7 ✅)login-styled R/VP0.9🔴 Modal anime en bas de page → overlay centré fixé (.modal-backdrop CSS manquant)modal-position R/VP0.4-bisHarmonisation toasts : « Vault »→« Completed », « Radar »→« Coming Soon » (DEC-71)toast-labels R/V
État : 76 tests unit verts, build prod ~5 s, bundle 715 kb. CI active. Zéro any.
Suite E2E cumulative (16 specs) : smoke, modal-open, boot-loader, discover-season-dedup, search-dedup, reccard-add, reccard-click-dismiss, modal-add-feedback, onair-subnav, week-no-duplicate-period, month-layout, nav-active-state, calendar-subnav-layout, login-styled, modal-position, toast-labels.
⚠️ Audit UX live NON refait en session 9
Même situation que session 8 : les correctifs sont validés par code + E2E, pas par observation visuelle. Un audit live (PO pilote le navigateur) reste indispensable avant clôture formelle d'EPIC P0, notamment pour confirmer visuellement P0.5/P0.6/P0.6-ter/P0.6-quater/P0.7/P0.9 et balayer F8/F10/F11/F13/F14/F15.

2. Reste à faire — EPIC P0
USObjetRisqueUS-121Auto-vault muet au boot : usePersistence.applyLoadTransitions passe un show Finished Airing en vault sans toast (F6 volet 2)Moyen — touche le bootP0.3bDédup For You batch : getNextBatch/buildNextBatch — un item wildcard peut sortir 2× (F5 dernier chemin)Élevé — touche le moteur de recoGrand check E2ER5 : npx playwright test sans filtre (16 specs) + npx vitest run + npm run buildObligatoire avant clôtureAudit livePO pilote le navigateur sur l'app déployéeAvant clôture formelle
P0.8c sorti d'EPIC P0 → EPIC-4 (DEC-69)
Décision « more-like-this » RecCard reportée : option A (modal, gratuit) vs option B (section inline, vraie feature). 2 mockups générés en session 9. À trancher à l'attaque rétention.

3. Findings CSS nouveaux (session 9 — backlog dette EPIC-2/3)
IDSévéritéConstatF18🟡Dette « VUE TEST » : ~150 lignes .test-* mortes (vue abandonnée, aucun composant ne les utilise)F19🟡Doublons .post-it définis 2× avec valeurs contradictoires (solid Google colors vs pastel)F20🟢Hacks CSS :has([style*="none"]) morts post-migration (vanilla togglait style.display)F21🟢#app-loading-overlay { display:none !important } — à vérifier vs loader P0.2F22🟢.month-header-mobile orpheline (doublon de .weekday-headers-mobile créé en P0.6)F23🟢« your Vault » empty state LibraryCompletedPage:37 (jargon, trivial)

4. Prompt de démarrage pour la session 10
On reprend Aanime (tracker d'animes, Vue 3 + TS strict + Pinia + Vue Router 4 + Vite +
Firebase + @vueuse + Playwright). Moi = Product Owner non-dev. Code implémenté par un dev
junior via Gemini AI Studio (PAS d'accès Knowledge → US 100 % autoportantes). Doc et
communication en FRANÇAIS. Réponses concises, radical candor, perspective UX, plan +
validation avant toute implémentation.

Lis d'abord TOUTE la Knowledge : CLAUDE.md → ARCHITECTURE_TECHNIQUE.md →
ARCHITECTURE_FONCTIONNELLE.md → AUDIT.md → AUDIT_UX_SESSION7.md → PLAN_MIGRATION.md →
TYPES_CONTRACT.md → ROADMAP.md → BACKLOG.md → DECISIONS.md → ANTIPATTERNS.md → AGENTS.md →
AGENTS_E2E.md → HANDOFF_SESSION9.md. Confirme la lecture en une phrase.

ÉTAT : migration + EPIC-1 clos. EPIC P0 quasi-clos : P0.0/0.1 (s7) +
P0.2/3a/3c/4/8a/8b (s8) + P0.5/6/6-bis/6-ter/6-quater/7/9/4-bis (s9) mergées.
Restent : US-121 (auto-vault muet boot) + P0.3b (dédup For You batch, moteur reco) +
grand check E2E R5 (16 specs sans filtre) + audit live PO avant clôture formelle.
P0.8c sorti → EPIC-4/US-152.

Règles NON-NÉGOCIABLES (AGENTS.md + AGENTS_E2E.md) :
R1 triple preuve verte (vue-tsc + vitest run + build), 3 sorties BRUTES SÉPARÉES.
R2 test runtime sur boot/store/câblage.
R3 un audit lit le CODE.
R4 test E2E qui reproduit le geste + asserte le DOM visible, ROUGE→VERT sans modif, une
   preuve ROUGE = un état figé unique.
R5 test ciblé par US pendant l'epic, grand check E2E complet en fin d'epic, tests cumulatifs.
R-CODE-7 contrat d'event = le composant ; les consommateurs s'alignent.
Zéro confiance (Y COMPRIS code/diagnostic de Claude). Paraphrase de preuve = review suspendue.
DIAGNOSTIC AVANT SPEC : grep lecture seule D'ABORD.
Max 3 fichiers/US — dépassement autorisé SI annoncé EN GRAS + dans le titre.
Une seule US In Progress. Fixtures via makeAnime(Partial<AnimeEntry>).

Confirme la lecture, affiche le Kanban (BACKLOG.md), puis propose le cadrage de US-121
(auto-vault muet au boot). DIAGNOSTIC AVANT SPEC : colle d'abord
`grep -n "applyLoadTransitions\|auto.*vault\|Finished Airing\|vault" src/composables/usePersistence.ts`
avant de rédiger quoi que ce soit.

Voici le Kanban : [colle le Kanban depuis BACKLOG.md]

5. Socle de test E2E (rappel)

playwright.config.ts : webServer build && preview, baseURL :4173, env VITE_E2E_AUTH_BYPASS=true.
Bypass auth statique (mort en prod). tests/e2e/** exclu de Vitest.
Réseau mocké via page.route() (jamais l'API live). Clé localStorage réelle = 'animeCalendar'.
Sélecteurs réels documentés dans AGENTS_E2E.md §3.
Sélecteurs session 9 confirmés : .rowcard (carte semaine), .modal-backdrop, .modal-content, .secondary-tab, .tab-item, .login-brand, .toast-notification.


6. NE PAS casser

Séquence boot App.vue onMounted : load → await syncAnimeUpdates → await buildRelationMemory → reScorePool → startBackgroundRelationFetch. Couverte par App.spec.ts. Intouchée.
Contrat d'emit RecCard (add/skip/click/not-interested/more-like-this) — ne pas renommer.
Helper dedupeByMalId = source unique de dédup (P0.3b le réutilisera).
Convention classe active nav = .active (DEC-67, alignée sur CSS existant).
Règles CSS préfixées .modal-backdrop — ne pas modifier .modal vanilla (DEC-70).


7. Décisions clés session 9 (DEC-66→71)

DEC-66 : libellé période = source unique CalendarNavControls. <h2> Month (P0.6) + #calendarHeader Week (P0.6-bis) supprimés. US-116 close.
DEC-67 : convention classe active nav = .active. Markup PrimaryNav + SecondaryNav alignés sur CSS existant (.primary-nav button.active / .secondary-nav button.active).
DEC-68 : P0.7 = style pur (tokens existants, script Firebase intact). US-122 reste EPIC-3.
DEC-69 : P0.8c sorti d'EPIC P0 → EPIC-4/US-152. Option A (modal, gratuit) vs B (section inline). 2 mockups générés s9.
DEC-70 : .modal-backdrop CSS manquant (3e occurrence du pattern « markup réf classe absente »). Règles préfixées .modal-backdrop .modal-content pour éviter collision avec .modal vanilla. position:fixed; display:flex; align-items:center (flexbox moderne, compatible v-if).
DEC-71 : toasts harmonisés — destination visible exacte (onglet réel, pas jargon interne). onMarkDone « Vault »→« Completed », onRecHeart branche else « Radar »→« Coming Soon ».


8. ⚠️ Tenue documentaire
Ne JAMAIS régénérer un doc Knowledge depuis une copie ancienne : demander au PO la version courante avant réécriture.
Docs modifiés en session 9 : ce fichier (remplace SESSION8), BACKLOG.md, DECISIONS.md (DEC-66→71), ANTIPATTERNS.md (pattern classe CSS absente + E2E position + récidives Gemini s9 + findings CSS F18–F23), AUDIT_UX_SESSION7.md (F3/F4/F7/F9/F12 ✅ + F-modal-position ✅ + F17 reclassé), ROADMAP.md (US-152 EPIC-4), CLAUDE.md (état réf session 9, compteur 76 unit + 16 E2E).
Docs NON modifiés (ne pas régénérer) : ARCHITECTURE_TECHNIQUE, ARCHITECTURE_FONCTIONNELLE, PLAN_MIGRATION, TYPES_CONTRACT, AUDIT.md, AGENTS.md, AGENTS_E2E.md.
