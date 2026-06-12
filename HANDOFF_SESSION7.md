# HANDOFF_SESSION7.md — Reprise de projet (EPIC P0 en cours)

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Remplace** HANDOFF_SESSION6.md (à archiver).
> **Rôle :** reprendre exactement où la session 7 s'est arrêtée, sans perte.

---

## 1. Où on en est

Migration Phases 0→7 : ✅ close. EPIC-1 (remédiation post-audit) : ✅ clos.
**Nouveauté session 7 :** un **audit UX live** (walkthrough navigateur réel) a révélé
des bugs que ni le tooling ni les audits de code n'avaient vus — dont un **P0
fonctionnel** (modal morte). Voir `AUDIT_UX_SESSION7.md` (16 findings F1→F16).

On a ouvert **EPIC P0** (correctifs UX bloquants) et posé un **socle de test E2E
Playwright** pour graver chaque correctif UX en test permanent (règle **R4**).

### Fait en session 7
| US | Objet | Statut |
|---|---|---|
| US-113a | Lazy-loading des pages (router) — EPIC-2, démarré puis EPIC-2 mis en pause | ✅ MERGE |
| P0.0 | Socle Playwright + auth de test mockée (bypass `import.meta.env`, **mort en prod**, prouvé `grep -c = 0` sur les chunks) | ✅ MERGE |
| P0.0-bis | Isolation runners : `tests/e2e` exclu de Vitest, `smoke.spec.ts` nettoyé du hack | ✅ MERGE |
| P0.1 | 🔴🔴 Modal morte réparée (désalignement event `click` vs `@open-modal`) + test E2E R4 (rouge→vert prouvé) | ✅ MERGE |

### Reste à faire — EPIC P0
| US | Objet | Finding |
|---|---|---|
| P0.2 | LoadingOverlay réellement visible au boot | F2 |
| P0.3 | Dédup des pools par `mal_id` (Discover For You/Season, suggestions recherche) | F5 |
| P0.4 | Feedback d'ajout (toast) + auto-vault annoncé | F6 / US-121 |
| P0.5 | Sous-nav On Air (Week/Month/List) → /month accessible | F3 |
| P0.6 | Layout Month cassé (en-tête grille + titres dans cellules) | F4 |
| P0.7 | /login stylé (AuthLayout + branding + explication flow) | F7 / US-122 |

Puis backlog UX : US-143 (signaux recos) → US-141 (marquer-vu 1-tap) → F8/F9/F10/F11/F12/F13/F14/F15.

---

## 2. Prompt de démarrage pour la nouvelle conversation
On reprend le projet Aanime (tracker d'animes, Vue 3 + TS strict + Pinia + Vue

Router 4 + Vite + Firebase + @vueuse + Playwright). Moi = Product Owner non-dev.

Code implémenté par un dev junior via Gemini AI Studio (PAS d'accès Knowledge → US

100 % autoportantes). Doc et communication en FRANÇAIS. Réponses concises, radical

candor, perspective UX, plan + validation avant toute implémentation.
Lis d'abord TOUTE la Knowledge dans cet ordre : CLAUDE.md → ARCHITECTURE_TECHNIQUE.md

→ ARCHITECTURE_FONCTIONNELLE.md → AUDIT.md → AUDIT_UX_SESSION7.md → PLAN_MIGRATION.md

→ TYPES_CONTRACT.md → ROADMAP.md → BACKLOG.md → DECISIONS.md → ANTIPATTERNS.md →

AGENTS.md → HANDOFF_SESSION7.md. Confirme la lecture en une phrase.
ÉTAT : migration + EPIC-1 clos. EPIC P0 (correctifs UX issus de l'audit live

session 7) EN COURS : P0.0 / P0.0-bis / P0.1 mergées. Reste P0.2→P0.7.

Socle Playwright en place (bypass auth E2E mort en prod). R4 gravée.
Règles process NON-NÉGOCIABLES (AGENTS.md) :

R1 triple preuve verte (vue-tsc + vitest run + build), sorties BRUTES.
R2 test runtime sur boot/store/câblage de composables.
R3 un audit lit le CODE.
R4 (NOUVEAU) : tout correctif UX / toute feature touchant l'écran livre un test

E2E Playwright qui reproduit le geste utilisateur et asserte le DOM visible (pas

l'état interne). ROUGE sur le bug, VERT après le fix, sans modification.
Zéro confiance : code brut intégral + sortie terminale littérale. Paraphrase = review suspendue.
Max 3 fichiers/US. Une seule US In Progress. Fixtures via makeAnime(Partial<AnimeEntry>).
Diagnostic AVANT spec : grep en lecture seule, décision sur preuve (5 specs de Claude

ont déjà été fautives faute de lecture — DEC-11/16/21/24 + event-name P0.1).

Confirme la lecture, affiche le Kanban (BACKLOG.md), puis propose le cadrage de P0.2

(LoadingOverlay visible au boot — finding F2, douleur boot du PO). NE rédige aucune US

tant que je n'ai pas validé.
Voici le Kanban de fin de session 7 : [colle le Kanban depuis BACKLOG.md]

---

## 3. Socle de test E2E (à connaître)

- **Playwright** installé (`@playwright/test` + chromium). Config : `playwright.config.ts`
  (webServer `npm run build && npm run preview`, baseURL `http://localhost:4173`,
  `env: { VITE_E2E_AUTH_BYPASS: 'true' }`).
- **Bypass auth E2E** : dans `src/router/index.ts`, `beforeEach` court-circuite si
  `import.meta.env.VITE_E2E_AUTH_BYPASS === 'true'`. Lu statiquement → **éliminé du
  bundle prod** (prouvé `grep -c = 0` sur tous les chunks). Ne JAMAIS le passer en
  variable runtime.
- **Isolation runners** : `vite.config.ts` → `test.exclude` ignore `tests/e2e/**`.
  Vitest = 72 tests unit ; Playwright = E2E séparé.
- **Patron de test E2E** : voir `tests/e2e/modal-open.spec.ts` (seed localStorage +
  clic + assertion visibilité DOM). ⚠️ vérifier la vraie clé localStorage (cf. dette P0.1).
- **Méthode R4** : écrire le test, le faire échouer (ROUGE) sur le bug actuel, puis
  corriger → VERT sans toucher le test. Fournir les DEUX sorties brutes.

---

## 4. État de santé technique

- Migration 100 % Vue/TS, vanilla supprimé. Zéro `any`. Bundle 715 kb (chunk principal,
  ~2/3 = Firebase Auth+Firestore — défer Firestore = US-117 en standby).
- Boot orchestré (App.vue) : `load → await sync → await buildRelationMemory →
  reScorePool → bg fetch`. Couvert par App.spec.ts. **NE PAS casser cette séquence.**
- Tests : 72 unit (Vitest) + E2E Playwright (smoke + modal-open). CI active.

---

## 5. En standby (repris après EPIC P0)

- **EPIC-2 perf** : US-113a fait. US-117 (défer Firestore, plus gros ROI 1er chargement
  que 113b/c — diagnostiqué session 7), puis US-113b/c, US-114 (file Jikan), US-111 E2E
  (en partie absorbé par le socle Playwright), US-112 Sentry.
- **EPIC-3/4/5** : voir ROADMAP.md.

---

## 6. ⚠️ Tenue documentaire

Ne JAMAIS regénérer un doc Knowledge depuis une copie ancienne : demander au PO la
version courante avant réécriture. Docs ajoutés/modifiés en session 7 : ce fichier,
`AUDIT_UX_SESSION7.md`, BACKLOG.md (EPIC P0), DECISIONS.md (DEC-56→58), ANTIPATTERNS.md
(blocs UX/E2E), AGENTS.md (R4).

