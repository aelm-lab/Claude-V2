# METHODOLOGY.md — Méthodo agile Aanime
> MAJ : S20. Cadre de travail PO (non technique) ↔ Claude (Tech Lead/PM/specs) ↔ Gemini (dev).

## Principe
Sprint = **un Sprint Goal atteint**, PAS « une session ». Une session peut contenir
1, 2 ou 3 sprints selon la capacité (Goal atteint + tokens OK). Claude signale
« Goal atteint, on ouvre le sprint suivant ? » plutôt que d'attendre le handoff.

## Cérémonies
| Cérémonie | Quand | Contenu |
|---|---|---|
| **Sprint Planning** | Ouverture de sprint | Lire Knowledge + Kanban → définir le **Sprint Goal** + choisir les US (déjà raffinées au sprint précédent). + vérifier le budget dette (≤1 US dette pour 1 US gain visible, PRODUCT_NORTHSTAR.md) avant de composer le sprint|
| **Backlog Refinement** | En continu + clôture | À chaque US close, Claude pose **1 question de refinement** sur une US du sprint suivant (décision produit en attente OU lecture R3 à prévoir). But : la 1ʳᵉ US du sprint suivant démarre **déjà raffinée**, zéro vas-et-vient à froid. |
| **TNR** (non-régression) | Avant chaque merge + clôture | Porte complète : type-check + test:run + build (3 sorties brutes séparées) + E2E cumulatifs (R5). Preuves Gemini IRRECEVABLES — seule la machine PO fait foi (R1). |
| **Release** | Clôture de sprint | Bump version + entrée dans STATE.md §Versions. Déploiement déjà continu (Cloud Run) → la release est un **repère versionné**, pas une mise en prod. |
**Sprint Outcome Gate** | Clôture de sprint | 1 ligne : gain ressenti / gain fiabilité / dette justifiée. Max 1 sprint "aucun gain" d'affilée. Détail → PRODUCT_NORTHSTAR.md.

## Schéma de version
`0.<sprint>.0` — ex. S20 = v0.20.0. Patch (`0.20.1`) si correctif hors sprint. 

## Règles de clôture de session

**Mise à jour minimale obligatoire.** Aucune session ne se termine sans qu'au moins
`STATE.md` soit mis à jour (Kanban + toute US mergée/débloquée dans la session). Si la
session a touché une règle de gouvernance (nouvel antipattern, nouvelle décision
d'architecture, changement de clé/contrat), le doc concerné (`ANTIPATTERNS.md` /
`DECISIONS.md` / `TYPES_CONTRACT.md`) est mis à jour dans la **même** session — pas
reporté. Un handoff qui ne touche aucun doc est un signal d'alerte, pas un raccourci.

**Handoff — double checklist obligatoire.** Chaque handoff de fin de session répond
explicitement à deux questions, même si la réponse est « rien à changer » :
1. **Côté Gemini** (`AGENTS.md`/`AGENTS_E2E.md`, racine **et** Knowledge) : une règle
   a-t-elle changé, ou une désynchronisation a-t-elle été détectée cette session ?
2. **Côté State** (`STATE.md`) : le Kanban a-t-il été mis à jour pour refléter la session ?

## Tag des US
`US-XXX [EPIC][SECTION][TYPE] Titre`
- **[EPIC]** = le OÙ (page/surface/système) — voir EPICS.md.
- **[SECTION]** = sous-zone (ex. WEEK, FOR-YOU, PLAN).
- **[TYPE]** = nature du travail, transverse : [FEATURE][UX][CSS][TEST][PERF][DETTE][A11Y][CI].
→ Vue page (priorisation PO) **et** vue type (filtrer toute la dette CSS) simultanées.

## Boucle de travail (inchangée)
Roadmap → Claude rédige US + cahier de tests → Gemini implémente → PO : git pull + porte
verte locale + colle 3 sorties brutes → Claude review (conforme ? antipatterns ? bloquant ?)
→ correction minimale si besoin → US suivante seulement après verdict MERGE.

## Règles de gate
- **R1** porte verte locale (3 commandes séparées, jamais `&&`).
- **R1-BIS** porte allégée si spec-only (`*.spec.ts`, 0 logique) : 1 run groupé / ~6 tests.
- Impact UX visible OU fin de sprint → porte complète + E2E TOUJOURS.
