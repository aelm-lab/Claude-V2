# AUDIT_UX_SESSION7.md — Audit UX live (walkthrough navigateur)

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.
> **Origine :** audit UX mené en session 7 par Claude pilotant un vrai navigateur
> Chrome sur l'app déployée (compte PO authentifié), écran par écran.
> **Pourquoi il est unique :** ces constats viennent de l'observation visuelle réelle ;
> ils ne sont pas re-dérivables sans refaire un walkthrough live.
>
> **Statuts mis à jour fin session 16.** Les correctifs sessions 8/9/10 ont été validés
> par code + E2E ; l'**audit live PO a eu lieu en session 12** (et non s11) et a clôturé
> formellement l'EPIC P0 (US-P0-C/D/E, DEC-77→80). Les findings backlog non traités
> (F8/F10/F13/F14/F15) restent ouverts et glissent vers EPIC-4.
>
> **Leçon transverse (fil rouge) :** l'app ne parle jamais à l'utilisateur. Le sprint UX
> qui change tout n'est pas une feature : c'est **rendre chaque action visible**.

---

## Méthode

Navigateur Chrome piloté → app déployée, clics réels, captures, comptage d'interactions.
Auth = compte PO (magic link saisi par le PO, jamais par Claude).

**Constat fondateur :** « ça compile ≠ c'est utilisable ». Un bug P0 (modal morte) a passé
`vue-tsc` + 72 tests + 2 audits de code. UN clic l'a révélé. → règle **R4**.

---

## Tableau des findings (F1→F17 + F-modal-position)

| # | Zone | Constat | Gravité | US cible | Statut |
|---|---|---|---|---|---|
| **F1** | Calendar Week + Discover | Modal morte : `WeekAnimeItem` émet `click`, page écoutait `@open-modal`. | 🔴🔴 P0 | P0.1 | ✅ CORRIGÉ s7 (R4) |
| **F2** | Boot | Écran beige 5-9 s sans spinner au 1er chargement. | 🔴 | P0.2 | ✅ CORRIGÉ s8 (code+E2E). Durée = US-117 (défer Firestore), distincte. Re-audit visuel s11. |
| **F3** | On Air | `/month` inatteignable : pas de sous-nav Week/Month/List. | 🔴 | P0.5 | ✅ CORRIGÉ s9 (R4). Re-audit visuel s11. |
| **F4** | Calendar Month | Layout cassé : libellés jours verticaux, cellules sans titre. | 🔴 | P0.6 | ✅ CORRIGÉ s9 (R4). Re-audit visuel s11. |
| **F5** | Discover/Season/Recherche/For You | Doublons partout (pools non dédupliqués). | 🔴 | P0.3a/b/c | ✅ COMPLET s8+s10 : Season (P0.3a) ✅ + Recherche (P0.3c) ✅ + For You batch (P0.3b) ✅. 3 chemins clos. |
| **F6** | Recherche / ajout / boot | Ajout silencieux (ni toast, ni redirection). Auto-vault muet. | 🔴 | P0.4 + US-121 | ✅ COMPLET s8+s10 : feedback ajout modal (P0.4) ✅ + auto-vault toast boot (US-121) ✅. 2 volets clos. |
| **F7** | Login | `/login` = HTML brut, zéro style/branding. | 🔴 | P0.7 | ✅ CORRIGÉ s9 (R4). Re-audit visuel s11. |
| **F8** | Dark mode | Sous-nav quasi illisible en dark ; logo faible contraste. | 🟡 | — | ⬜ Backlog — à observer audit s11 |
| **F9** | Sous-onglets | Aucun état actif visible (For You/Season/Coming Soon gris identiques). | 🟡 | P0.6-ter | ✅ CORRIGÉ s9 (R4). Re-audit visuel s11. |
| **F10** | Calendar Week | Jours vides muets (pas de suggestion, pas de CTA). | 🟡 | US-144 | ⬜ Backlog EPIC-4 |
| **F11** | Calendar Week | Pas de snap-to-today. | 🟡 | US-150 | ✅ LIVRÉ s10 (US-150, DEC-76). Re-audit visuel s11 (test E2E faible — voir réserves HANDOFF §4). |
| **F12** | Calendar | Libellé date dupliqué. | 🟡 | US-116 | ✅ CORRIGÉ s9 (P0.6-bis). Re-audit visuel s11. |
| **F13** | Recherche | Suggestions sans année/score, pas de bouton « + » direct. | 🟡 | US-145 | ⬜ Backlog EPIC-4 |
| **F14** | Discover/Season | ~6 s de blanc sans skeleton (SkeletonCard existe, non utilisé). | 🟡 | — | ⬜ Backlog — à observer audit s11 |
| **F15** | Library/Upcoming | Titre BYW collé au bord, section vide, hiérarchie typo incohérente. | 🟡 | — | ⬜ Backlog — à observer audit s11 |
| **F16** | Discover/cartes | Signaux recos génériques répétés ; pas de « Because you watched X ». | 🟡 | US-143 | ✅ DÉJÀ IMPLÉMENTÉ (fermé sans dev s10) : `BecauseYouWatched.vue` affiche `_triggerTitle`, `RecCard` affiche `_signals` + panneau why. |
| **F17** | Discover + Library (RecCard) | Famille event-name : Add mort, clic carte mort, « pas intéressé » mort. | 🔴🔴 P0 | P0.8a/b/c | 🟡 PARTIEL : Add (P0.8a) ✅ + clic/dismiss (P0.8b) ✅. **Reste P0.8c `more-like-this` → US-152 EPIC-4** (décision produit : option A modal vs option B section inline, toujours en attente). |
| **F-modal-position** | Calendar Week | Modal anime s'affichait en bas de page (`.modal-backdrop` CSS manquant). | 🔴 | P0.9 | ✅ CORRIGÉ s9 (R4, DEC-70). Re-audit visuel s11. |

---

## Findings ajoutés en session 10 (hors walkthrough — livrés comme quick wins)

| # | Zone | Constat | Gravité | US | Statut |
|---|---|---|---|---|---|
| **F18-bonus** | Calendar Week | Marquer-vu coûtait 3 taps (carte→modal→Mark done). | 🟡 | US-141 | ✅ LIVRÉ s10 — bouton ✓ direct sur la carte. Fonctionnel, **non stylé** (`.rc-mark-done` sans CSS) — à confirmer visuellement s11. |
| **F19-bonus** | Calendar Week | Pas de repère visuel sur le jour courant à l'ouverture. | 🟡 | US-150 | ✅ LIVRÉ s10 — snap-to-today. Test E2E faible (voir HANDOFF §4 réserves). |
| **F20-bonus** | Calendar Week | Avancement d'un anime non visible sans ouvrir la modal. | 🟡 | US-142 | ✅ LIVRÉ s10 — barre de progression fine sous la carte (WeekAnimeItem uniquement). |

---

## Synthèse statuts (mise à jour session 16)

**EPIC P0 — CLOS formellement en session 12** (audit live PO, R6 ; DEC-77→80).

| Catégorie | Findings | Statut |
|---|---|---|
| P0 résolus | F1, F2, F3, F4, F5, F6, F7, F9, F12, F-modal-position, F17 partiel | ✅ Code+E2E |
| P0 restant | F17 (P0.8c `more-like-this`) | → EPIC-4/US-152 |
| Quick wins s10 | F11 (snap), F18/19/20 bonus | ✅ Code+E2E |
| Backlog non traités | F8, F10, F13, F14, F15 | ⬜ À observer s11 |

**À confirmer visuellement en session 11 (audit live PO) :**
F2 (loader), F3 (sous-nav), F4 (layout month), F7 (login), F9 (états actifs), F11 (snap), F12 (doublon période), F-modal-position (modal centrée), F18-bonus (bouton ✓), F19-bonus (snap visuel), F20-bonus (barre progression).

---

## Dette de test (résolue)
- **[P0.1]** ✅ Clé localStorage `'animeCalendar'` confirmée (DEC-64). Levée.
- **[P0.8c]** Emit `more-like-this` non câblé → reporté EPIC-4/US-152, décision produit requise.
- **[US-150]** Test E2E faible (faux vert possible un jeudi) → audit live = vrai juge.
- **[US-141]** Bouton ✓ non stylé (`.rc-mark-done` sans CSS dans `style.css`) → audit live → US CSS dédiée si nécessaire.
