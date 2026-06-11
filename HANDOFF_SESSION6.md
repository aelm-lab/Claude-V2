# HANDOFF_SESSION6.md — Reprise de projet pour une nouvelle conversation

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** permettre à une nouvelle conversation Claude de reprendre exactement
> où la session 6 s'est arrêtée (EPIC-1 clos), sans perte de contexte.
> Remplace `HANDOFF_SESSION5.md` (à archiver/supprimer).

---

## 1. Où on en est

**La migration est terminée et stabilisée.**

- **Phases 0→7 : ✅ closes** (scaffold, logique pure, store+composables, router+layouts, composants atomiques, pages, modals & sheets, branchement final).
- **EPIC-1 (remédiation post-audit) : ✅ clos.** Les 4 bugs critiques de l'audit sont corrigés sous filet de test.
- **État : prêt pour le tag `v2-stable`.**
- **69 tests Vitest verts. CI active. Zéro `any`. Code 100 % Vue/TS (vanilla supprimé).**

### Ce qui a été fait en session 6 (EPIC-1)
| US | Objet | Statut |
|---|---|---|
| US-109 | Filet : CI (`ci.yml`) + smoke test boot (`App.spec.ts`) | ✅ MERGE |
| US-102 | P0 — moteur de reco rebranché au boot (DEC-50) | ✅ MERGE |
| US-106 | P1 — throttle conditionnel réseau (pattern `*WithMeta`, DEC-51) | ✅ MERGE |
| US-107 | P1 — hiatus source unique 14j (DEC-52) | ✅ MERGE |
| US-101 | Suppression 31 fichiers vanilla morts | ✅ MERGE |
| US-104 | Dark mode `body.dark-mode` → `html.dark` | ✅ MERGE |
| US-105 | Déduplication contrôles de navigation date | ✅ MERGE |
| US-110 | `AGENTS.md` gouvernance permanente Gemini (DEC-53) | ✅ MERGE |

---

## 2. Prompt de démarrage pour la nouvelle conversation

```
On reprend le projet Aanime (migration vanilla JS → Vue 3 + TS + Pinia, TERMINÉE).
Lis d'abord toute la Knowledge : CLAUDE.md, ARCHITECTURE_TECHNIQUE.md,
ARCHITECTURE_FONCTIONNELLE.md, AUDIT.md, PLAN_MIGRATION.md, BACKLOG.md, ROADMAP.md,
TYPES_CONTRACT.md, DECISIONS.md, ANTIPATTERNS.md, AGENTS.md, HANDOFF_SESSION6.md.

État : migration + EPIC-1 (remédiation post-audit) clos. 69 tests verts, CI active,
code 100 % Vue/TS, tag v2-stable. Les 4 bugs runtime de l'audit sont corrigés.

Confirme la lecture, affiche le Kanban depuis BACKLOG.md, puis propose le cadrage
du prochain epic de ROADMAP.md (priorité à valider avec moi).
Applique toutes les règles process (R1/R2/R3 + AGENTS.md + DECISIONS.md + ANTIPATTERNS.md).
NE rédige pas d'US tant que je n'ai pas validé la priorité de l'epic.
```

---

## 3. Règles process NON-NÉGOCIABLES (à réinjecter)

1. **R1 — Triple preuve verte CI** pour tout MERGE : `vue-tsc` + `vitest run` + `build` (sorties brutes).
2. **R2 — Test obligatoire** sur toute US touchant boot / store / câblage de composables.
3. **R3 — Audit lit le code**, pas seulement les indicateurs verts.
4. **Zéro confiance** : code brut intégral + sortie terminale littérale (`$` + commande + sortie réelle). Résumé = review suspendue.
5. **Max 3 fichiers / US** (sauf suppression pure prouvée).
6. **Une seule US In Progress** — pas de MERGE, pas de suite.
7. **Fixtures via `makeAnime(Partial<AnimeEntry>)`** — jamais `as any`.
8. **Gemini lit `AGENTS.md`** (gouvernance gravée) mais reste sans accès Knowledge → US autoportantes.
9. PO non-dev → ne jamais lui faire trancher une question purement technique seul (proposer + expliquer).

---

## 4. État de santé technique

- **Boot orchestré** (App.vue) : `load → await syncAnimeUpdates → await buildRelationMemory → reScorePool → startBackgroundRelationFetch` (fire-and-forget). Couvert par `App.spec.ts`.
- **Moteur de reco** : fonctionnel (graphe reconstruit + re-score au boot).
- **Throttle Jikan** : conditionnel au réseau réel (`fromNetwork`).
- **Hiatus** : source unique `isOnHiatus` 14j.
- **Dark mode** : `html.dark` partout (CSS + AppHeader + useDark cohérents).
- **Dépôt** : 100 % Vue/TS, `firebase-applet-config.json` préservé.

---

## 5. Dette ouverte & prochaines priorités (détail dans ROADMAP.md)

**Reste avant/au tag v2-stable :**
- Nettoyage repo : supprimer `PHASE8_DEBT.md` (remplacé par ROADMAP) et `HANDOFF_SESSION5.md` (remplacé par ce fichier).

**EPIC-2 — plateforme & robustesse (recommandé en premier) :**
- File Jikan globale (anti-429 sur navigation rapide).
- Code-split + fix warning chunking `idb.ts` (bundle 749 kb).
- Monitoring erreurs (store d'erreurs / Sentry) au lieu de `console.error` isolés.
- `useScrollKeeper` (dédup scroll restore).
- Polish : retirer le libellé de période interne aux pages calendrier (doublon résiduel de `CalendarNavControls`, post-US-105).

**EPIC-3 — dette fonctionnelle (ex-P8) :**
- Bug studios inerte (`scorePool` lit `studios` pluriel, `normalize` produit `studio`).
- Auto-vault silencieux → toast informatif.
- `window.prompt()` re-saisie email → UI dédiée.
- `fetchTopFinishedAnime` inline → migrer dans `useJikanApi`.
- Mapping MAL `Dropped` → discutable.
- Clés localStorage à harmoniser (`aanime_*`).
- Redirect post-login vers route d'origine.
- `SyncIndicator` : couverture complète des fetches Jikan.
- Nettoyer le type `onHiatus?` (plus écrit depuis US-107).

**EPIC-4 / 5 — produit & plateforme :** onboarding, marquer-vu 1-tap, snap-to-today, Library en chips, PWA, Firestore realtime, virtualisation listes, a11y.

---

## 6. ⚠️ Note de tenue documentaire (pour Claude session 7)

Les docs `BACKLOG.md`, `DECISIONS.md`, `ROADMAP.md`, `ANTIPATTERNS.md` ont été tenus à jour
par le PO au fil des sessions 5-6. **Ne jamais les regénérer à partir d'une copie ancienne** :
toujours demander au PO de coller la version courante avant toute réécriture, sous peine de
perdre du contexte. Les ajouts de session 6 à intégrer (si pas déjà faits) : DEC-50→DEC-55,
les US EPIC-1 en Done, les leçons d'audit dans ANTIPATTERNS.
