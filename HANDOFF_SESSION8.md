# HANDOFF_SESSION8.md — Reprise de projet (EPIC P0 en cours)

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Remplace** HANDOFF_SESSION7.md (à archiver). HANDOFF_SESSION6.md peut être supprimé.
> **Rôle :** reprendre exactement où la session 8 s'est arrêtée, sans perte.

---

## 1. Où on en est

Migration Phases 0→7 : ✅ close. EPIC-1 (remédiation post-audit) : ✅ clos.
EPIC P0 (correctifs UX, issu de l'audit live session 7) : **en cours, 6 US mergées en session 8.**

La session 8 a surtout révélé et clos une **famille de bugs event-name** (le jumeau de
la modal morte P0.1) : le composant `RecCard`, réutilisé sur 4 surfaces, avait son bouton
Add, son clic-carte et son « pas intéressé » **tous morts** (events émis sous un nom,
écoutés sous un autre). Un audit `emit` vs `@listener` transverse a confirmé que c'est le
**seul foyer** restant — tout le reste est aligné.

### Fait en session 8 (mergé)
| US | Objet | Preuve |
|---|---|---|
| doc | R4 inséré dans `AGENTS.md` (insertion vérifiée, zéro perte) | diff |
| P0.2 | LoadingOverlay visible au boot (F2) — loader statique `index.html` + overlay racine App.vue hors gate auth | E2E `boot-loader` R/V |
| P0.3a | Helper `dedupeByMalId` + dédup This Season (F5 1/3) | E2E `discover-season-dedup` R/V |
| P0.3c | Dédup recherche (F5 3/3) | E2E `search-dedup` R/V |
| P0.8a | 🔴 Bouton Add mort RecCard réparé (`@heart`→`@add`, 3 conso + wrapper) | E2E `reccard-add` R/V |
| P0.8b | 🔴 Clic carte→modal + Not interested→dismiss câblés ; emit orphelin supprimé | E2E `reccard-click-dismiss` R/V |
| audit | Audit event-name transverse — CLOS (1 foyer RecCard, reste aligné, `open-recency` OK) | grep |
| P0.4 | Feedback toast ajout depuis modal (F6 volet 1) | E2E `modal-add-feedback` R/V |

État : **76 tests unit verts**, build prod ~4 s, bundle 715 kb. CI active. Zéro `any`.
Tests E2E cumulés : `smoke`, `modal-open`, `boot-loader`, `discover-season-dedup`,
`search-dedup`, `reccard-add`, `reccard-click-dismiss`, `modal-add-feedback`.

### ⚠️ Audit UX live NON refait en session 8
Claude n'a pas pu refaire le walkthrough navigateur (pas d'outil de navigateur interactif
dans la session + app déployée derrière auth AI Studio / cookie de sécurité). Les
correctifs session 8 sont validés par **code + E2E** (R1/R4), **pas par observation
visuelle**. → Un audit live (PO pilotant le navigateur, comme session 7) reste à faire
avant de clore l'EPIC P0, notamment pour confirmer visuellement P0.2/P0.4/P0.8 et balayer
les findings non traités (F8/F9/F10/F11/F12/F13/F14/F15).

---

## 2. Reste à faire — EPIC P0
| US | Objet | Finding |
|---|---|---|
| P0.5 | Sous-nav On Air (Week/Month/List) → `/month` accessible depuis l'UI | F3 |
| P0.6 | Layout Month cassé (en-tête grille + titres dans cellules) | F4 |
| P0.7 | `/login` stylé (AuthLayout + branding + explication flow) | F7 / US-122 |

### Findings dérivés (ordonnancer après ou en intercalaire)
| ID | Objet |
|---|---|
| P0.4-bis | Harmoniser libellés existants : `onMarkDone` « Vault »→« Completed », Discover « Radar »→« Coming Soon » |
| US-121 | Auto-vault muet au boot (`usePersistence`, F6 volet 2) |
| P0.8c | `@more-like-this` RecCard — décision produit (modal simple vs scroll-to-section) |
| P0.3b | Dédup For You batch (`getNextBatch`/`buildNextBatch` chevauchement wildcards) — touche le moteur, dernier chemin F5 |

Puis backlog UX : US-143, US-141, F8/F9/F14/F15, etc.

---

## 3. Prompt de démarrage pour la session 9

```
On reprend Aanime (tracker d'animes, Vue 3 + TS strict + Pinia + Vue Router 4 + Vite +
Firebase + @vueuse + Playwright). Moi = Product Owner non-dev. Code implémenté par un dev
junior via Gemini AI Studio (PAS d'accès Knowledge → US 100 % autoportantes). Doc et
communication en FRANÇAIS. Réponses concises, radical candor, perspective UX, plan +
validation avant toute implémentation.

Lis d'abord TOUTE la Knowledge : CLAUDE.md → ARCHITECTURE_TECHNIQUE.md →
ARCHITECTURE_FONCTIONNELLE.md → AUDIT.md → AUDIT_UX_SESSION7.md → PLAN_MIGRATION.md →
TYPES_CONTRACT.md → ROADMAP.md → BACKLOG.md → DECISIONS.md → ANTIPATTERNS.md → AGENTS.md →
AGENTS_E2E.md → HANDOFF_SESSION8.md. Confirme la lecture en une phrase.

ÉTAT : migration + EPIC-1 clos. EPIC P0 en cours : P0.0/0.1 (s7) + P0.2/3a/3c/4/8a/8b (s8)
mergées. Reste P0.5/P0.6/P0.7 + findings dérivés (P0.4-bis, US-121, P0.8c, P0.3b). Audit UX
LIVE à refaire (PO pilote le navigateur) avant clôture EPIC P0.

Règles NON-NÉGOCIABLES (AGENTS.md + AGENTS_E2E.md) :
R1 triple preuve verte (vue-tsc + vitest run + build), 3 sorties BRUTES SÉPARÉES.
R2 test runtime sur boot/store/câblage.
R3 un audit lit le CODE.
R4 test E2E qui reproduit le geste + asserte le DOM visible, ROUGE→VERT sans modif, une
   preuve ROUGE = un état figé unique.
R5 test ciblé par US pendant l'epic, grand check E2E complet en fin d'epic, tests cumulatifs.
R-CODE-7 contrat d'event = le composant ; les consommateurs s'alignent.
Zéro confiance (Y COMPRIS code/diagnostic de Claude). Paraphrase de preuve = review suspendue
(3 récidives « Build succeeded » déjà). DIAGNOSTIC AVANT SPEC : grep lecture seule D'ABORD.
Max 3 fichiers/US — dépassement autorisé SI annoncé EN GRAS + dans le titre. Une seule US
In Progress. Fixtures via makeAnime(Partial<AnimeEntry>).

Confirme la lecture, affiche le Kanban (BACKLOG.md), puis propose le cadrage de P0.5
(sous-nav On Air — finding F3). NE rédige aucune US tant que je n'ai pas validé le
diagnostic et la cible.

Voici le Kanban de fin de session 8 : [colle le Kanban depuis BACKLOG.md]
```

---

## 4. Socle de test E2E (rappel)
- `playwright.config.ts` : webServer `build && preview`, baseURL :4173, env `VITE_E2E_AUTH_BYPASS=true`.
- Bypass auth statique (mort en prod, `grep -c=0`). `tests/e2e/**` exclu de Vitest.
- Réseau mocké via `page.route()` (jamais l'API live). Clé localStorage réelle = `'animeCalendar'`.
- Sélecteurs réels documentés dans `AGENTS_E2E.md` §3 (les lire, ne jamais deviner).

---

## 5. NE PAS casser
- Séquence boot `App.vue onMounted` : `load → await syncAnimeUpdates → await
  buildRelationMemory → reScorePool → startBackgroundRelationFetch` (fire-and-forget).
  Couverte par `App.spec.ts`. **Intouchée par P0.2.**
- Contrat d'emit de `RecCard` (`add`/`skip`/`click`/`not-interested`/`more-like-this`) —
  ne pas renommer ; les pages s'alignent dessus.
- Helper `dedupeByMalId` = source unique de dédup (P0.3b le réutilisera).

---

## 6. ⚠️ Tenue documentaire
Ne JAMAIS régénérer un doc Knowledge depuis une copie ancienne : demander au PO la version
courante avant réécriture. Docs ajoutés/modifiés en session 8 : ce fichier (remplace
SESSION7), `AGENTS_E2E.md` (nouveau), `AGENTS.md` (R4 intégré + R5 + R-SCOPE consolidés +
R-CODE-7), `BACKLOG.md`, `DECISIONS.md` (DEC-59→65), `ANTIPATTERNS.md` (foyer event-name +
récidives s8), `AUDIT_UX_SESSION7.md` (statuts mis à jour), `CLAUDE.md` (carte des docs).
Docs NON modifiés (ne pas régénérer) : ARCHITECTURE_TECHNIQUE, ARCHITECTURE_FONCTIONNELLE,
PLAN_MIGRATION, TYPES_CONTRACT, ROADMAP, AUDIT.md.
