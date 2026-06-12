# AUDIT_UX_SESSION7.md — Audit UX live (walkthrough navigateur)

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Origine :** audit UX mené en session 7 par Claude pilotant un vrai navigateur
> Chrome sur l'app déployée (compte PO authentifié), écran par écran, en comptant
> les taps et en capturant chaque friction.
> **Pourquoi il est unique :** ces constats ne viennent NI du code NI des docs — ils
> viennent de l'observation visuelle réelle. Ils ne sont pas re-dérivables sans
> refaire un walkthrough live. C'est la mémoire de ce que l'utilisateur VOIT.
>
> **Leçon transverse de l'audit (le fil rouge) :** l'app ne parle jamais à
> l'utilisateur. Boot muet, ajout muet, auto-vault muet, onglet actif muet, jour
> vide muet, clic carte muet. Chaque action réussie ou échouée est indistinguable
> de « rien ». Le sprint UX qui change tout n'est pas une feature : c'est **rendre
> chaque action visible**.

---

## Méthode

Navigateur Chrome piloté → app déployée (`/week`, `/month`, `/discover`,
`/discover/season`, `/library`, `/library/plan`, `/library/completed`, recherche).
Clics réels, captures à chaque écran, comptage d'interactions. Auth = compte PO
(magic link saisi par le PO, jamais par Claude).

**Constat fondateur (jumeau UX de la leçon session 6) :** « tooling vert ≠ app qui
marche » a un frère : « ça compile ≠ c'est utilisable ». Un bug P0 (modal morte) a
passé `vue-tsc` + 72 tests + 2 audits de code. Il a suffi d'UN clic pour le voir.
→ D'où la règle **R4** (test E2E qui clique et regarde l'écran).

---

## Tableau des findings (F1→F16)

| # | Zone | Constat (observé à l'écran) | Preuve | Gravité | US cible | Statut |
|---|---|---|---|---|---|---|
| **F1** | Calendar Week + Discover | **Modal morte** : cliquer une carte n'ouvre RIEN (testé 4 fois, 2 pages). Cause : `WeekAnimeItem` émet `click`/`ep-chip-click`, la page écoutait `@open-modal`/`@open-ep-override` → emit dans le vide. Tout Phase 6 inaccessible (relations, override, recency, **suppression**). | clics live, 0 erreur console | 🔴🔴 P0 | **P0.1** | ✅ CORRIGÉ (R4) |
| **F2** | Boot | **Écran beige vide 5-9 s sans aucun spinner** au 1er chargement (`/month` à froid). LoadingOverlay invisible. = la « lenteur 1ère charge » ressentie : bundle lourd + zéro feedback = sensation de site mort. | navigation /month à froid | 🔴 | **P0.2** | ⬜ À faire |
| **F3** | On Air | **`/month` inatteignable depuis l'UI** : pas de sous-nav Week/Month/List sur On Air (Discover et Library ont leur sous-nav, On Air non). Accès uniquement par URL directe. | captures /week | 🔴 | **P0.5** | ⬜ À faire |
| **F4** | Calendar Month | **Layout cassé** : libellés Mon→Sun empilés VERTICALEMENT à gauche au lieu d'en-tête de grille ; cellules = pills d'heures **sans titre** (on ne sait pas quoi diffuse). | capture /month | 🔴 | **P0.6** | ⬜ À faire |
| **F5** | Discover (For You + Season), Recherche | **Doublons partout** : « Dr. Stone Part 3 » ×2 côte à côte (This Season), 1ʳᵉ et 3ᵉ carte identiques (For You), « Frieren Ougonkyou-hen » ×2 (suggestions). Dédup absente sur tous les pools. | 3 écrans | 🔴 | **P0.3** | ⬜ À faire |
| **F6** | Recherche / ajout | **Ajout 100 % silencieux** : clic suggestion → dropdown se ferme, rien d'autre. Anime auto-vaulté sans toast, sans highlight, sans redirection. L'utilisateur ne sait ni si ça a marché ni où c'est parti. | test add live (Frieren) | 🔴 | **P0.4** (+US-121) | ⬜ À faire |
| **F7** | Login | **`/login` = HTML brut** : input + bouton « Send magic link » collés coin haut-gauche, zéro style, zéro branding, zéro explication du flow. Toute première impression = « site cassé ». AuthLayout non appliqué/non stylé. | 2 captures | 🔴 | **P0.7** (+US-122) | ⬜ À faire |
| **F8** | Transverse (dark mode) | Sous-nav quasi **illisible** en dark (texte sombre sur fond sombre) ; logo Aanime faible contraste. | capture dark | 🟡 | nouveau | ⬜ Backlog |
| **F9** | Sous-onglets | **Aucun état actif visible** : For You / This Season / Coming Soon tous gris identiques ; idem Library. L'utilisateur ne sait pas où il est. | captures discover/library | 🟡 | nouveau | ⬜ Backlog |
| **F10** | Calendar Week | **Jours vides muets** (Tue, Thu) : pas de suggestion (WeekSuggestionCard non rendue ?), pas de CTA. Cul-de-sac. | capture /week | 🟡 | US-144 | ⬜ Backlog |
| **F11** | Calendar Week | **Pas de snap-to-today** : on atterrit sur Monday, le point • du jour courant (Friday) est hors écran. | capture /week | 🟡 | US-150 | ⬜ Backlog |
| **F12** | Calendar | **Libellé date dupliqué** : « Jun 8—14, 2026 » ×2, « June 2026 » ×2 (barre nav + libellé interne page). | captures week+month | 🟡 | US-116 | ⬜ Backlog |
| **F13** | Recherche | Suggestions sans **année ni score** pour désambiguïser (4 « Frieren » indistinguables), pas de bouton « + » direct. | capture suggestions | 🟡 | US-145 | ⬜ Backlog |
| **F14** | Discover/This Season | **~6 s de blanc total sans skeleton** au chargement (le composant SkeletonCard existe pourtant et n'est pas utilisé). | capture /discover/season | 🟡 | nouveau | ⬜ Backlog |
| **F15** | Library/Upcoming | Titre BYW interminable collé au bord, **section vide dessous** ; hiérarchie typographique incohérente (titres de sections de tailles/casses disparates). | capture /library | 🟡 | nouveau | ⬜ Backlog |
| **F16** | Discover/cartes | Chip « **Ep 11** » sans total vs « Ep 11/12 » ailleurs (incohérence) ; signaux recos génériques répétés (« Same theme: Isekai · Fantasy » ×4) ; pas de « Because you watched X » sur la carte alors que `_signals`/`_triggerTitle` existent dans le type. | captures discover | 🟡 | US-143 | ⬜ Backlog |

---

## Synthèse de priorisation

**EPIC P0 (bloquants UX) = F1→F7.** Ordre d'attaque :
1. F1 (modal) — ✅ fait
2. F2 (LoadingOverlay) — douleur boot n°1 du PO
3. F5 (dédup) — qualité perçue immédiate
4. F6 (feedback ajout) — l'app doit confirmer les actions
5. F3 (sous-nav On Air) — /month accessible
6. F6/F4 (Month layout)
7. F7 (login)

**Backlog UX post-P0 :** F16→US-143 (signaux recos, quasi gratuit) puis marquer-vu 1-tap
(US-141), puis F10/F11/F12/F13 (US-144/150/116/145) et les nouveaux F8/F9/F14/F15.

## Findings SANS US existante (à créer en session future)
- **F8** dark mode contraste sous-nav + logo
- **F9** état actif des sous-onglets
- **F14** skeletons non utilisés sur les pages async (le composant existe)
- **F15** hiérarchie typo des titres de sections + section BYW vide

## Dette de test détectée
- **[P0.1]** le test E2E `modal-open.spec.ts` seede `localStorage` avec la clé devinée
  `'animeCalendar'`. **À vérifier** : est-ce la vraie clé écrite par `usePersistence` ?
  Si non, le test passe pour une mauvaise raison. Grep à faire avant de s'appuyer
  dessus comme référence.
