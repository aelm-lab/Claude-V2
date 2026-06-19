# HANDOFF_SESSION10.md — Reprise de projet (EPIC P0 clos technique + quick wins UX)

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Remplace** HANDOFF_SESSION9.md (à archiver).
> **Rôle :** reprendre exactement où la session 10 s'est arrêtée, sans perte.
> **Prochaine session = AUDIT MANUEL du PO** (voir §4, c'est la mission n°1).

---

## 1. Où on en est

Migration Phases 0→7 : ✅ close. EPIC-1 : ✅ clos. **EPIC P0 : ✅ CLOS côté technique** (US-121 + P0.3b livrées, grand check R5 vert). Reste l'**audit live PO** avant clôture formelle.

Session 10 a (a) résorbé une lourde dette technique auto-infligée par Gemini, (b) fermé les 2 derniers items P0, (c) livré 3 quick wins UX d'EPIC-4.

### Fait en session 10 (mergé)

| US | Objet | E2E |
|---|---|---|
| US-121 | Auto-vault muet au boot → 2 toasts séparés « Moved to Completed » (F6 volet 2) | auto-vault-toast R/V |
| P0.3b | Dédup For You batch via `dedupeByMalId` sur batch sortant (F5 dernier chemin, moteur intouché) | foryou-dedup R/V |
| US-141 | Marquer-vu 1 tap depuis calendrier (bouton ✓ sur WeekAnimeItem → emit → page) | week-mark-done R/V |
| US-150 | Snap-to-today à l'arrivée sur /week (remplace scroll-restore) | snap-to-today R/V |
| US-142 | **Barre de progression sur cartes semaine** (WeekAnimeItem + style.css, 3 fichiers) | week-progress-bar R/V |
| US-143 | Signaux recos visibles (F16) → **DÉJÀ IMPLÉMENTÉ, fermée sans dev** | — |
| DEC-72 | Dette technique boot-loader résorbée (déviation actée, voir §7) | suite E2E réparée |

**État :** 76 tests unit verts, build prod ~4.7 s, bundle 716 kb. CI active. Zéro `any`.

**Suite E2E cumulative (22 specs) :** smoke, modal-open, boot-loader, discover-season-dedup, search-dedup, reccard-add, reccard-click-dismiss, modal-add-feedback, onair-subnav, week-no-duplicate-period, month-layout, nav-active-state, calendar-subnav-layout, login-styled, modal-position, toast-labels, **auto-vault-toast, foryou-dedup, week-mark-done, snap-to-today, week-progress-bar** (+ les 17 existantes).

---

## 2. ⚠️ Bilan de session — leçon process majeure

**~80 % de la session 10 a été consacrée à réparer une cascade que Gemini a déclenchée seul.** En réponse à une simple demande de diagnostic, Gemini a modifié **5 fichiers sans aucune US** (`main.ts`, `index.html`, `App.vue`, `playwright.config.ts`, `smoke.spec.ts`), cassant la suite E2E. Violation R-SCOPE-1 la plus coûteuse du projet.

**Recommandation gravée pour la session 11+ :** au démarrage, exiger de Gemini un état du dépôt propre (le sandbox AI Studio interdit `git status` — à défaut, demander la liste des fichiers modifiés depuis le dernier merge connu) **avant** de lancer toute action. Ne jamais laisser Gemini « préparer le terrain » de sa propre initiative.

**Leçon Claude (auto-critique) :** 2 mauvais diagnostics en cours de session (mock `**/anime/**` ajouté puis retiré ; hypothèse « Jikan écrase le store »). Cause : avoir proposé un fix **avant** de lire le test qui fonctionnait déjà (`modal-open.spec.ts`). Zéro-confiance s'applique à Claude : **lire le code qui marche AVANT de proposer**, pas après 4 allers-retours.

---

## 3. Reste à faire

| Item | Détail | Statut |
|---|---|---|
| **Audit live PO** | Walkthrough navigateur sur l'app déployée (voir §4) | **Mission session 11** |
| Specs E2E post-audit | À définir EN session 11, après l'audit, avec Claude | En attente |
| Clôture formelle EPIC P0 | Après audit live confirmé | En attente |

**Backlog post-P0 :** EPIC-2 (US-117 défer Firestore = ROI 1er chargement, US-113b/c, US-114, US-111/112) · EPIC-3 (US-120→130) · EPIC-4 (US-140 onboarding, US-144 états vides, US-145, US-146→152) · Dette CSS F18–F23 · P0.8c→US-152.

---

## 4. 🎯 MISSION SESSION 11 — Audit manuel du PO (flux validé)

**Le PO teste l'app déployée à sa façon, librement, sans script imposé.** Puis :

1. **PO → Claude** : le PO rapporte les gestes réellement effectués + ce qu'il a observé (ce qui marche, ce qui cloche, ce qui surprend).
2. **Claude** : analyse les retours, identifie les **trous de couverture** (fonctionnalités existantes non testées par le PO), et **décide** quelles specs E2E ajouter/adapter.
3. **⚠️ AVANT de rédiger toute spec E2E**, Claude DOIT demander au PO :
   - le fichier `AGENTS_E2E.md` courant ;
   - l'état réel des specs déjà présentes chez Gemini (lecture seule : `ls tests/e2e/` ou équivalent) ;
   pour ne PAS spécifier à l'aveugle (leçon récurrente du projet).
4. **Claude → PO** : rédige les instructions E2E prêtes à coller dans Gemini.
5. **PO → Gemini** : implémente. **Claude** : review.

> ⚠️ L'audit se fait **avec Claude**, pas avec Gemini (qui n'a aucun œil sur l'app déployée). Gemini n'intervient qu'à l'étape implémentation des specs.

### Grille des gestes à auditer (réf. — Claude la complétera par lecture du code en session 11)

Quick wins session 10 à confirmer visuellement :
- **US-141** : sur une carte du calendrier semaine, taper ✓ → l'anime passe en Completed + toast « Moved to Completed » + la carte disparaît.
- **US-150** : ouvrir On Air / Week → la vue est ancrée sur le jour courant (pas sur lundi).
- **US-142** : une carte d'anime en cours affiche une fine barre de progression sous la carte.
- **US-121** : si un anime suivi est terminé, au boot un toast « Moved to Completed » apparaît.
- **P0.3b** : la section For You ne montre jamais 2× la même carte.

Correctifs s9 jamais confirmés visuellement (à balayer aussi) :
- P0.5 sous-nav On Air, P0.6 layout Month, P0.6-ter états actifs navs, P0.6-quater layout secondary nav, P0.7 login stylé, P0.9 modal centrée, P0.2 loader au boot.

Findings UX backlog jamais traités (à observer) : F8 (dark contraste sous-nav), F10 (jours vides muets), F11 (snap — partiellement couvert par US-150), F13 (recherche enrichie), F14 (skeletons inutilisés), F15 (hiérarchie typo BYW).

### ⚠️ Réserves connues (à ne PAS confondre avec des bugs)
- **US-141 bouton ✓ non stylé** : la classe `.rc-mark-done` n'existe pas dans `style.css` (interdiction d'inventer du CSS dans l'US). Le bouton est **fonctionnel mais brut/mal positionné**. Juger la fonction, pas l'esthétique. → décision PO : faut-il une US CSS dédiée (positionnement + cible tactile 44px) ?
- **US-150 test E2E faible** : la preuve E2E a tourné un jeudi avec viewport réduit → faux vert possible (le jour courant était déjà visible sans scroll). La mécanique code est correcte ; l'audit visuel est le vrai juge.
- **US-142 progression** : barre absente si `currentEp` ou `episodes` manquant (volontaire, pas un bug).

---

## 5. Socle de test E2E (rappel)

- `playwright.config.ts` : webServer `build && preview`, baseURL :4173, env `VITE_E2E_AUTH_BYPASS=true`, **`timeout: 120000`** (ajouté s10 pour le sandbox lent).
- Bypass auth statique (mort en prod). `tests/e2e/**` exclu de Vitest.
- Réseau mocké via `page.route()` (jamais l'API live). Clé localStorage réelle = `'animeCalendar'`.
- **Grand check sandbox** : lancer `npx playwright test --workers=1` (le parallèle provoque des `ERR_CONNECTION_REFUSED` faux-négatifs dans le sandbox Gemini ; `--workers=1` est fiable). La CI GitHub Actions n'a pas ce souci.
- **Pattern E2E boot-dépendant (gravé s10)** : pour tout test qui clique une carte après le boot, attendre `await expect(page.locator('#boot-loader')).toBeHidden({ timeout: 15000 })` AVANT de localiser/cliquer (le loader DEC-72 intercepte les pointer events tant que le boot tourne). Seeder 7 jours (un anime/jour) pour garantir une carte visible quel que soit le jour courant.
- Sélecteurs confirmés s10 : `.rc-mark-done` (bouton ✓ carte semaine), `.rc-progress` / `.rc-progress-fill` (barre progression), `.day-hdr.today` (jour courant), `.card-cp-title` (titre RecCard). + sélecteurs s9 : `.rowcard`, `.modal-backdrop`, `.modal-content`, `.secondary-tab`, `.tab-item`, `.login-brand`, `.toast-notification`.

---

## 6. NE PAS casser

- Séquence boot `App.vue` onMounted : `load → await syncAnimeUpdates → await buildRelationMemory → reScorePool → startBackgroundRelationFetch`. Couverte par `App.spec.ts`. **Intouchée.**
- **boot-loader (DEC-72)** : `#boot-loader` est hors `#app` dans `index.html`, supprimé via `document.getElementById('boot-loader')?.remove()` dans le `finally` du onMounted d'`App.vue`. `main.ts` = `app.mount('#app')` simple (PAS de `router.isReady()` — testé, ça casse boot-loader.spec). Ne pas « réparer ».
- Contrat d'emit `RecCard` (add/skip/click/not-interested/more-like-this) — ne pas renommer.
- Contrat d'emit `WeekAnimeItem` : `click` / `ep-chip-click` / **`mark-done`** (ajouté s10). Le composant émet, la page (`CalendarWeekPage`) mute le store.
- `dedupeByMalId` = source unique de dédup (Season + recherche + For You). Ne pas créer de 2ᵉ logique.
- `buildNextBatch` (recEngine) = fonction pure, 15 tests, cadence wildcard 1/5 — **intouchée** (P0.3b dédup en aval, pas dedans).
- Convention classe active nav = `.active` (DEC-67).
- Règles CSS préfixées `.modal-backdrop` — ne pas modifier `.modal` vanilla (DEC-70).

---

## 7. Décisions clés session 10 (DEC-72→76)

- **DEC-72** : boot-loader hors `#app` + suppression via `getElementById('boot-loader').remove()` dans le `finally` d'`App.vue`. Exception R-CODE-4 documentée (DOM direct autorisé dans App.vue, au même titre que useICS/import MAL). Ajustement non sollicité par Gemini mais fonctionnel → acté plutôt que reverté. `main.ts` reste un `mount('#app')` simple.
- **DEC-73** : US-121 auto-vault muet → **2 toasts séparés** (« Moved to On Air » à 1000ms + « Moved to Completed » à 1500ms, décalage anti-collision). `TransitionResult` enrichi de `movedToVault`. `saveToDatabase` déclenché si `movedToOnAir OU movedToVault` (sinon auto-vault non persisté). Toast = « Completed », jamais « Vault » (DEC-71).
- **DEC-74** : P0.3b dédup For You via `dedupeByMalId(fullBatch).slice(0, size)` dans `getNextBatch` (dédup AVANT slice pour garder 12 items pleins). `recEngine.buildNextBatch` strictement intouché. Réutilise la digue DEC-60. `mal_id === id` sur AnimeEntry normalisé → helper fonctionne tel quel.
- **DEC-75** : US-141 marquer-vu 1 tap. `WeekAnimeItem` émet `mark-done` (`@click.stop` pour ne pas ouvrir la modal) ; `CalendarWeekPage.onMarkDone` réplique la logique de `AnimeModal.onMarkDone` (state vault + completedAt + toast). Composant émet, page mute (R-CODE-3).
- **DEC-76** : US-150 snap-to-today **remplace** le scroll-restore (`savedScrollY`/`onDeactivated` supprimés). `snapToToday()` appelé en `onMounted` + `onActivated` (KeepAlive), `behavior:'auto'`, garde `todayIndex < 0` (semaine sans aujourd'hui = no-op). Pas de re-snap sur navigation Prev/Next.
- **US-143 fermée sans dev** : F16 déjà implémenté — `BecauseYouWatched.vue:3` affiche `_triggerTitle`, `RecCard.vue` affiche `_signals` + panneau why + badge. Rien à coder. (Dette notée : `BecauseYouWatched.vue` a un `<style scoped>` antérieur, à traiter avec F18–F23.)

---

## 8. ⚠️ Tenue documentaire

Ne JAMAIS régénérer un doc Knowledge depuis une copie ancienne : demander au PO la version courante avant réécriture.

**Docs modifiés en session 10 :** ce fichier (remplace SESSION9), BACKLOG.md, DECISIONS.md (DEC-72→76), ANTIPATTERNS.md (récidive 5-fichiers-sans-US + 4ᵉ paraphrase build + leçon Claude diagnostic), CLAUDE.md (état réf session 10, compteur 76 unit + 22 E2E), AUDIT_UX_SESSION7.md (F5/F6/F16 ✅), ROADMAP.md (US-141/150/142 livrées EPIC-4, US-143 déjà faite).

**Docs NON modifiés (ne pas régénérer) :** ARCHITECTURE_TECHNIQUE, ARCHITECTURE_FONCTIONNELLE, PLAN_MIGRATION, TYPES_CONTRACT, AUDIT.md, AGENTS.md, **AGENTS_E2E.md** (à revoir EN session 11 après l'audit, pas avant).

---

## 9. Prompt de démarrage pour la session 11 (AUDIT)

```
On reprend Aanime (tracker d'animes, Vue 3 + TS strict + Pinia + Vue Router 4 + Vite +
Firebase + @vueuse + Playwright). Moi = Product Owner non-dev. Code implémenté par un dev
junior via Gemini AI Studio (PAS d'accès Knowledge → US 100 % autoportantes). Doc et
communication en FRANÇAIS. Réponses concises, radical candor, perspective UX, plan +
validation avant toute implémentation.

Lis d'abord TOUTE la Knowledge : CLAUDE.md → ARCHITECTURE_TECHNIQUE.md →
ARCHITECTURE_FONCTIONNELLE.md → AUDIT.md → AUDIT_UX_SESSION7.md → PLAN_MIGRATION.md →
TYPES_CONTRACT.md → ROADMAP.md → BACKLOG.md → DECISIONS.md → ANTIPATTERNS.md → AGENTS.md →
AGENTS_E2E.md → HANDOFF_SESSION10.md. Confirme la lecture en une phrase.

ÉTAT : migration + EPIC-1 + EPIC P0 (code) clos. Session 10 a livré US-121, P0.3b
(F5/F6 entièrement clos) + 3 quick wins EPIC-4 (US-141 marquer-vu 1 tap, US-150
snap-to-today, US-142 barre progression). US-143 fermée (déjà faite). 76 unit + 22 E2E
verts. Reste : AUDIT LIVE PO avant clôture formelle EPIC P0.

MISSION DE CETTE SESSION = mon audit manuel. Je vais tester l'app déployée à ma façon,
puis je te rapporte mes gestes + observations. Toi (Claude) : tu analyses, tu identifies
les trous de couverture, et tu rédiges les specs E2E pour Gemini.
⚠️ AVANT de rédiger toute spec E2E, demande-moi : (1) AGENTS_E2E.md courant,
(2) la liste des specs déjà présentes chez Gemini (tests/e2e/). Ne spécifie jamais à l'aveugle.

Règles NON-NÉGOCIABLES : R1 triple preuve verte (3 sorties brutes séparées, JAMAIS de
paraphrase build — 4 récidives au compteur). R2 test runtime boot/store/câblage.
R3 un audit lit le CODE. R4 E2E geste réel + DOM visible, ROUGE→VERT sans modif.
R5 ciblé par US + grand check fin d'epic, cumulatif. R-CODE-7 contrat d'event = le composant.
Zéro confiance (Y COMPRIS code/diagnostic de Claude — lire le code qui marche AVANT de
proposer). DIAGNOSTIC AVANT SPEC : grep lecture seule D'ABORD. Max 3 fichiers/US (dépassement
si annoncé EN GRAS + titre). Une seule US In Progress. Au démarrage Gemini : exiger l'état
des fichiers modifiés AVANT toute action (récidive s10 : 5 fichiers modifiés sans US).

Confirme la lecture, affiche le Kanban (BACKLOG.md que je te colle), puis aide-moi à
structurer mon audit manuel (grille de gestes + réserves connues du HANDOFF §4).

Voici le Kanban : [colle le Kanban depuis BACKLOG.md]
```
