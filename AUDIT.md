# AUDIT.md — Audit technique

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> Partie 1 = audit de l'existant vanilla (à reproduire). Partie 2 = audit
> **post-migration** mené en session 6 (le tournant qualité du projet).

---

# PARTIE 1 — Audit de l'existant (vanilla JS, référence fonctionnelle)

> Cette partie décrit l'état d'origine reproduit par la migration. Le code vanilla
> a été supprimé du dépôt en US-101 ; cette section reste la mémoire du comportement.

## 1.1 Stack d'origine
Vanilla JS (ES Modules), Vite 6 + plugin-react (mort), Firebase v12 (email-link),
Firestore (`schedules/{uid}`), Jikan v4, IndexedDB (`AnimeCaches` v2 : relations, recommendations),
localStorage/sessionStorage, routeur maison (`AppStore.currentView` + show/hide), state global
mutable `AppStore` notifiant via `CustomEvent` DOM, `style.css` (dark mode classe sur `<body>`).

## 1.2 Vues (9 + login)
`calendar:week`, `calendar:month`, `calendar:list` (stub), `incoming:explore`, `incoming:season`,
`incoming:comingup`, `library:explore`, `library:plantowatch`, `library:completed`, `login`.

## 1.3 Bus d'événements DOM (remplacé par Pinia)
`store:changed`, `ui:refresh`, `anime:add/added`, `recs:heart/remove`, `library-recs:*`,
`recs:relations:rebuild`, `nav:next/prev` → tous remplacés par actions store / `watch` / `emit`.

## 1.4 Endpoints Jikan
`/anime?q=`, `/anime/{id}`, `/anime/{id}/relations`, `/anime/{id}/recommendations`,
`/seasons/now` (3 pages, throttle 1s), `/seasons/upcoming`, top finished (`min_score=7.5`).
Contraintes : `fetchWithRetry` (3 essais, backoff, 429), batch 2/2s, worker throttle 1,1s.

## 1.5 Risques identifiés à l'origine
- `modal.js` (465 l.), `incoming.js` (580 l.) : DOM impératif, handlers sur `window`.
- `persistence.js` : injectait un bandeau dans le `<body>` → remplacé par `staleDataWarning` réactif.
- `getAnimeEpisodeInfo` : calcul multi-branches sans tests → porté à l'identique + testé.
- Rate-limiting Jikan, `parseJSTToLocal` (ancre 1970) → répliqués fidèlement.

---

# PARTIE 2 — Audit post-migration (session 6)

> **Le tournant qualité du projet.** Après la fin de la migration (Phases 0→7),
> deux audits indépendants ont été menés en parallèle. Leur confrontation a révélé
> 4 bugs runtime critiques qu'un audit complaisant avait manqués.

## 2.1 Méthode : audit croisé

| Audit | Outil | Verdict initial | Bugs critiques trouvés |
|---|---|---|---|
| Audit A | Claude Code | « migration réussie, app saine » | **0** (a raté les 4) |
| Audit B | Gemini (prompt UX+tech complet) | critique, détaillé | **4** (tous confirmés) |

**Leçon fondatrice (→ règle R3) :** l'audit A a validé sur la **forme** (type-check vert,
tests verts, build OK) sans **lire la chaîne d'orchestration runtime**. L'audit B a lu le
code. Résultat : `vue-tsc` + tests + build au vert ont créé une **fausse confiance** alors
que la logique métier centrale était sectionnée.

## 2.2 Les 4 bugs (confirmés par preuve brute, grep ligne par ligne)

| # | Bug | Gravité | Preuve | Statut |
|---|---|---|---|---|
| 1 | **Moteur de reco mort** : `App.vue` n'importait pas `useRecommendations` ; `_buildRelationMemory`/`_reScorePool` étaient des stubs no-op appelés à vide dans `useSync`. Graphe de relations jamais reconstruit, recos jamais re-scorées. | 🔴 P0 | `useSync.ts:22,26` (stubs) + `App.vue` onMounted sans `buildRelationMemory` | ✅ Corrigé US-102 |
| 2 | **Throttle 1100ms inconditionnel** : `setTimeout(1100)` + `shouldRescore=true` après chaque itération du worker, **cache hit IDB inclus** → ~11 min de sleep / 300 animes en cache au boot. | 🟠 P1 | `useSync.ts:223,230` | ✅ Corrigé US-106 |
| 3 | **Hiatus 14j vs 21j** : `episodeInfo.ts:99` calcule hiatus à 14 j (lu à l'affichage) ; `useSync.ts:117` écrivait `onHiatus` à 21 j (jamais lu = code mort). | 🟠 P1 | `episodeInfo.ts` (14j) vs `useSync.ts` (21j) | ✅ Corrigé US-107 |
| 4 | **`_syncAnimeUpdates` stub** dans `usePersistence` : `needsBroadcastSync` mutait en local mais la vraie synchro n'était pas câblée → réparations perdues au reload. (même racine que #1) | 🟠 P1 | `usePersistence.ts:13` | ✅ Résolu via US-102 (orchestration App.vue) |

> **Faux positif écarté :** l'audit A rapportait 64/66 tests (2 échecs « fuseau horaire »).
> Vérification : **66/66** réels — artefact du conteneur de l'audit A. Aucune dette test.
> (Après ajout du smoke test US-109 : **69/69**.)

## 2.3 Remédiation (EPIC-1) — filet d'abord, puis correctifs

Décision PO : **filet de sécurité avant correctifs**, pour corriger sous protection.

1. **US-109 — Filet** : `ci.yml` (vue-tsc + vitest + build) + `App.spec.ts` (smoke test boot).
   Le 3ᵉ test, volontairement **rouge**, encodait le contrat du bug P0.
2. **US-102 — P0** : `App.vue` orchestre `load → await sync → await buildRelationMemory →
   reScorePool → bg fetch` (DEC-50). Le test rouge est passé **vert sans être modifié** →
   preuve que le filet fonctionne de bout en bout.
3. **US-106 — P1 throttle** : pattern `*WithMeta` exposant `fromNetwork` → throttle 1,1s
   uniquement sur appel réseau réel.
4. **US-107 — P1 hiatus** : suppression du calcul mort 21j ; `isOnHiatus` 14j = source unique.
5. **US-101** : suppression des 31 fichiers vanilla morts (grappe fermée, prouvée par inventaire).
6. **US-104** : dark mode aligné `body.dark-mode` → `html.dark`.
7. **US-105** : déduplication des contrôles de navigation date.
8. **US-110** : `AGENTS.md` musclé (gouvernance permanente Gemini).

## 2.4 Recommandations de l'audit B reportées au backlog (→ ROADMAP.md)
- Pas de **file Jikan globale** centralisée (navigation rapide = 429 ponctuel) → EPIC-2.
- **Bundle monolithique** ~749 kb + warning chunking `idb.ts` → code-split EPIC-2.
- `console.error` isolés → **store d'erreurs / monitoring** (Sentry) EPIC-2.
- **Scroll restore dupliqué** dans plusieurs pages → composable `useScrollKeeper` EPIC-2.
- Idées UX : snap-to-today, Library Completed/Plan en chips, feedback d'ajout radar, onboarding MAL.

## 2.5 Conclusion de l'audit
La migration est **structurellement saine** (séparation des couches, zéro `any`, zéro DOM
bus, store pur) mais souffrait de **4 trous d'orchestration runtime** invisibles au tooling.
Tous corrigés sous filet de test. Le process est durci (R1/R2/R3) pour rendre ce type de
régression silencieuse **impossible** à l'avenir.

---

# PARTIE 3 — Dual audit (session 16)

> **Deuxième audit croisé du projet, après celui de la session 6.** Objectif : repartir
> propre avant de charger EPIC-4, en confrontant deux audits **indépendants** menés sur le
> **même commit** avec un **cadre strictement identique** (mêmes 7 axes, même barème
> P0/P1/P2, même format de tableau). Le piège historique (disparités de bruit) vient de
> cadres différents ; ici le cadre commun rend les rapports comparables ligne à ligne.

## 3.1 Méthode

| Audit | Outil | Cadre | Rôle |
|---|---|---|---|
| Audit A | Claude Code | identique (7 axes, P0/P1/P2, tableau `ID/Axe/Finding/Gravité/fichier:ligne/Impact user`) | lecture profonde du code + raisonnement archi |
| Audit B | Gemini | identique | lecture du repo dans son sandbox |

**Axes** : 1. Bugs runtime / logique morte · 2. Sécurité des types · 3. Gestion d'erreur ·
4. Dette & duplication · 5. Performance · 6. Couverture de tests · 7. Architecture / séparation.

## 3.2 Résultat marquant — chaque audit a raté le finding n°1 de l'autre

C'est exactement ce que le dual audit devait attraper :
- **Claude Code** a trouvé le bug `getCardStatus` `'Continuing'` → `Finished` — **Gemini aveugle dessus**.
- **Gemini** a trouvé `saveSchedule` sans try/catch (son seul P0) — **Claude Code aveugle dessus**.

→ Confirmation : un **cadre commun strict** est ce qui révèle les angles morts.
Sans lui, l'un valide « sain » pendant que l'autre crie au feu (cf. s6).

## 3.3 Convergences (haute confiance)

| ID | Finding | Gravité | Preuve | Impact utilisateur |
|---|---|---|---|---|
| C1 | Boot bloquant (`await syncAnimeUpdates()` retarde la levée de l'overlay) | P1 | `App.vue` onMounted | Spinner plusieurs secondes au démarrage |
| C2 | Zéro test unit sur composables critiques | P1 | composables sans `.spec` | Régressions non détectées (cause racine de C-Continuing) |
| C3 | Cast legacy `as unknown as AnimeEntry[]` sans normalisation | P1 | `usePersistence.ts` | Cache corrompu → cartes incomplètes (rare) |
| C4 | Fichiers parasites racine | P2 | `diff.cjs`, `*_out.txt`… | Aucun — dette R-SCOPE-1 |
| C5 | Import idb « dynamique inutile » = **NON-PROBLÈME** (import statique, confirmé par les deux) | — | `useRecommendations.ts:4` | Aucun |

## 3.4 Divergences tranchées par lecture du code réel (zéro-confiance)

| # | Conflit | Verdict (code réel) | Suite |
|---|---|---|---|
| D1 | `getCardStatus` `'Continuing'` → `Finished` | **CONFIRMÉ** (tombe sur le return défaut) | US-154 (P1) |
| D2 | `saveSchedule` sans try/catch | **CONFIRMÉ** (ni `saveToDatabase` ni `saveSchedule` n'ont de garde) | US-153 (**P0**) |
| D3 | `setAllData` manquant (Gemini le traitait en bug) | **REJETÉ** — seul `setData` (+`clearAll`) existe, tous les appelants l'utilisent | — |
| D4 | `syncStatus` / `reconcileWithDatabase` | **REJETÉ / CORRIGÉ** — 0 hit ; réconciliation dans `loadFromDatabase` ; handoff S15 périmé | doc corrigée |
| D5 | `getElementById('boot-loader').remove()` (Gemini : anti-pattern P1) | **LÉGITIME** — élément d'`index.html` (loader pré-Vue, DEC-72) | reclassé P2 |

## 3.5 Backlog priorisé consolidé

| Rang | US | Finding | Gravité | Impact utilisateur | Reco Claude |
|---|---|---|---|---|---|
| 1 | US-153 | `saveToDatabase`/`saveSchedule` sans try/catch | 🔴 P0 | Perte de données silencieuse si Firestore échoue | Fix immédiat : try/catch + état erreur + toast échec + test R2 |
| 2 | US-154 | `getCardStatus` `'Continuing'` → `Airing` | 🟠 P1 | Show en cours affiché « Finished » | Mapper la valeur + test unit du mapping |
| 3 | US-155 | Boot non bloquant | 🟠 P1 | Spinner plusieurs secondes au démarrage | Lever l'overlay après load local, sync en fond + E2E R4 |
| 4 | US-156 | Tests unit composables | 🟠 P1 | Aucun — cause racine de US-154 | Commencer `useEpisodeInfo` + `useSync`, étaler |
| 5 | US-157 | Persistance mute le store hors action + toasts | 🟠 P1 | Aucun — couplage, source de US-153/154 | Mutations via actions, sortir les toasts |
| 6 | US-158 | Cast legacy non normalisé | 🟠 P1 | Cache corrompu → cartes incomplètes (rare) | Garde runtime + normalisation |
| 7 | US-159-CLEANUP | Fichiers parasites racine | 🟢 P2 | Aucun — dette | Pass cleanup groupé |

**P2 mono-source** (à grouper) : stubs morts `_syncAnimeUpdates`/`_startBackgroundRelationFetch` ·
`onNavigate` cast partiel · `status as AnimeStatus` direct · `syncAnimeUpdates()` sans await
dans `AnimeModal` · worker fond bloqué si throw · exports morts · duplication `negativeIds` ×4 ·
cache relations sans TTL · zéro test composant `.vue` · erreurs rate-limit avalées en `console.warn`.
**À vérifier (1 grep)** : re-save Firestore inutile au boot (coûts/quotas).

## 3.6 Conclusion

Aucun P0 visible non corrigé hérité des sessions précédentes ; un seul P0 **nouveau**
révélé (US-153, sauvegarde silencieuse). L'essentiel des findings est de la dette invisible
(P1/P2). Sur le vécu utilisateur, **deux** corrections comptent vraiment : US-153 (risque
silencieux) et US-154 (label mensonger) ; US-155 est la seule douleur ressentie au quotidien
(boot lent). Le process (R1→R6, zéro-confiance étendue aux handoffs) tient.
