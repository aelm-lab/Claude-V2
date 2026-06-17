# AUDIT_UX_SESSION7.md — Audit UX live (walkthrough navigateur)

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Origine :** audit UX mené en session 7 par Claude pilotant un vrai navigateur
> Chrome sur l'app déployée (compte PO authentifié), écran par écran.
> **Pourquoi il est unique :** ces constats viennent de l'observation visuelle réelle ;
> ils ne sont pas re-dérivables sans refaire un walkthrough live.
>
> **Statuts mis à jour fin session 8** (colonne Statut). ⚠️ Les correctifs session 8 sont
> validés par **code + E2E**, PAS par un nouvel audit visuel (Claude n'a pas pu refaire le
> walkthrough en session 8 — pas de navigateur interactif + app derrière auth AI Studio).
> Un re-audit live est nécessaire pour confirmer visuellement F2/F5/F6/event-name.
>
> **Leçon transverse (fil rouge) :** l'app ne parle jamais à l'utilisateur. Le sprint UX
> qui change tout n'est pas une feature : c'est **rendre chaque action visible**.

---

## Méthode

Navigateur Chrome piloté → app déployée, clics réels, captures, comptage d'interactions.
Auth = compte PO (magic link saisi par le PO, jamais par Claude).

**Constat fondateur (jumeau de la leçon session 6) :** « ça compile ≠ c'est utilisable ».
Un bug P0 (modal morte) a passé `vue-tsc` + 72 tests + 2 audits de code. UN clic l'a révélé.
→ règle **R4**. La session 8 a confirmé que ce n'était pas isolé : la même classe de bug
(event-name désaligné) frappait aussi tout le composant `RecCard`.

---

## Tableau des findings (F1→F16)

| # | Zone | Constat | Gravité | US cible | Statut |
|---|---|---|---|---|---|
| **F1** | Calendar Week + Discover | Modal morte : cliquer une carte n'ouvre rien (`WeekAnimeItem` émet `click`, page écoutait `@open-modal`). | 🔴🔴 P0 | P0.1 | ✅ CORRIGÉ (R4) |
| **F2** | Boot | Écran beige 5-9 s sans spinner au 1er chargement. | 🔴 | P0.2 | ✅ CORRIGÉ s8 (code+E2E ; re-audit visuel à faire). Durée des 5-9 s = US-117 (défer Firestore), distincte. |
| **F3** | On Air | `/month` inatteignable : pas de sous-nav Week/Month/List sur On Air. | 🔴 | P0.5 | ⬜ À faire |
| **F4** | Calendar Month | Layout cassé : libellés jours verticaux, cellules sans titre. | 🔴 | P0.6 | ⬜ À faire (dépend de P0.5 pour être atteignable) |
| **F5** | Discover/Season/Recherche | Doublons partout (pools non dédupliqués). | 🔴 | P0.3a/b/c | 🟡 PARTIEL : Season (P0.3a) ✅ + Recherche (P0.3c) ✅. **Reste For You batch = P0.3b** (dernier chemin). |
| **F6** | Recherche / ajout | Ajout silencieux (ni toast, ni redirection). | 🔴 | P0.4 (+US-121) | 🟡 PARTIEL : feedback ajout depuis modal (P0.4) ✅. **Reste auto-vault muet au boot = US-121.** |
| **F7** | Login | `/login` = HTML brut, zéro style/branding. | 🔴 | P0.7 (+US-122) | ⬜ À faire |
| **F8** | Dark mode | Sous-nav quasi illisible en dark ; logo faible contraste. | 🟡 | nouveau | ⬜ Backlog |
| **F9** | Sous-onglets | Aucun état actif visible (For You/Season/Coming Soon gris identiques). | 🟡 | nouveau | ⬜ Backlog |
| **F10** | Calendar Week | Jours vides muets (pas de suggestion, pas de CTA). | 🟡 | US-144 | ⬜ Backlog |
| **F11** | Calendar Week | Pas de snap-to-today. | 🟡 | US-150 | ⬜ Backlog |
| **F12** | Calendar | Libellé date dupliqué. | 🟡 | US-116 | ⬜ Backlog |
| **F13** | Recherche | Suggestions sans année/score, pas de bouton « + » direct. | 🟡 | US-145 | ⬜ Backlog |
| **F14** | Discover/Season | ~6 s de blanc sans skeleton (SkeletonCard existe, non utilisé). | 🟡 | nouveau | ⬜ Backlog |
| **F15** | Library/Upcoming | Titre BYW collé au bord, section vide, hiérarchie typo incohérente. | 🟡 | nouveau | ⬜ Backlog |
| **F16** | Discover/cartes | Chip « Ep 11 » sans total ; signaux recos génériques répétés ; pas de « Because you watched X » alors que `_signals`/`_triggerTitle` existent. | 🟡 | US-143 | ⬜ Backlog |

---

## Finding ajouté en session 8 (hors walkthrough — révélé par audit de code/grep)

| # | Zone | Constat | Gravité | US cible | Statut |
|---|---|---|---|---|---|
| **F17** | Discover + Library (RecCard) | Famille event-name : bouton **Add mort** (`@heart` écouté, `add` émis), **clic carte mort** (`@click` non écouté), **« pas intéressé » mort** (`@not-interested` non écouté) sur les 4 surfaces de reco. Même classe que F1. + emit orphelin `open-modal`. | 🔴🔴 P0 | P0.8a/b/c | 🟡 PARTIEL : Add (P0.8a) ✅ + clic/dismiss (P0.8b) ✅. **Reste `more-like-this` = P0.8c** (décision produit). |

> **Audit event-name transverse (session 8) :** balayage `defineEmits` vs `@listener` sur
> tout `src/components/`. Résultat : `RecCard` = **seul** foyer 🔴 (F17). Tout le reste
> aligné. `open-recency` bien émis par `ModalCalendarTop` (pas fantôme). Dette event-name
> (F1 + F17) **cartographiée et close** (sauf `more-like-this` reporté).

---

## Synthèse de priorisation (mise à jour s8)

**EPIC P0 = F1→F7 + F17.** Restant : **P0.5 (F3) → P0.6 (F4) → P0.7 (F7)**, puis findings
dérivés (P0.4-bis, US-121, P0.8c, P0.3b). **Re-audit live recommandé** avant clôture EPIC P0.

**Backlog UX post-P0 :** F16→US-143, puis US-141, F10/F11/F12/F13 (US-144/150/116/145),
et F8/F9/F14/F15 (nouveaux).

## Findings SANS US existante (à créer)
- **F8** dark contraste sous-nav + logo
- **F9** état actif des sous-onglets
- **F14** skeletons non utilisés sur les pages async
- **F15** hiérarchie typo des titres de sections + section BYW vide

## Dette de test (résolue)
- **[P0.1]** ✅ La clé localStorage seedée dans `modal-open.spec.ts` (`'animeCalendar'`)
  est CONFIRMÉE être la vraie clé écrite par `usePersistence.saveToDatabase` (DEC-64).
  Le test repose sur une vraie clé. Dette levée.
