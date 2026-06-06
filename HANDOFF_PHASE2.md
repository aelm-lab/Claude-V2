# HANDOFF_PHASE2.md — Reprise de projet pour une nouvelle conversation

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Rôle :** permettre à une **nouvelle conversation Claude Chat** de reprendre exactement où la session 1 s'est arrêtée, sans perte de contexte. À lire en premier par la nouvelle instance.

---

## 1. Où on en est

- **Phase 0 (scaffold) : ✅ close.**
- **Phase 1 (logique pure) : ✅ close à 100 %.** Tout `src/utils/` est porté + testé (~50 tests Vitest, zéro `any`).
- **Prochaine étape : Phase 2 — Store Pinia + composables services.**

Fichiers `src/utils/` déjà livrés et mergés : `jst.ts`, `normalize.ts`, `episodeInfo.ts`, `helpers.ts`, `idb.ts`, `recEngine.ts`, `ics.ts` (pur), `malImport.ts` (pur). Types : `src/types/anime.ts`, `recs.ts`, `persistence.ts` (à jour dans `TYPES_CONTRACT.md`).

---

## 2. Prompt de démarrage pour la nouvelle conversation

> Copie ce bloc comme premier message dans la nouvelle conv (les `.md` doivent être dans la Knowledge) :

```
On reprend la migration Aanime (vanilla JS → Vue 3 + TS + Pinia).
Lis d'abord la Knowledge : CLAUDE.md, AUDIT.md, PLAN_MIGRATION.md, BACKLOG.md,
TYPES_CONTRACT.md, ANTIPATTERNS.md, DECISIONS.md, HANDOFF_PHASE2.md.

État : Phases 0 et 1 closes (tout src/utils/ porté + testé, ~50 tests, zéro any).
On démarre la Phase 2 (Store Pinia + composables services).

Confirme la lecture, affiche le Kanban depuis BACKLOG.md, puis propose le
cadrage de l'US-011 (store Pinia stores/anime.ts). NE rédige pas encore l'US :
il te faut d'abord que je te colle le code vanilla source (src/store.js).
Applique toutes les règles process de DECISIONS.md et ANTIPATTERNS.md
(zéro confiance, US autoportantes car Gemini n'a pas la Knowledge, preuve par
sortie de commande, fixtures typées, max 3 fichiers/US).
```

---

## 3. Règles process à NE PAS perdre (résumé)

1. **Zéro confiance** — chaque US livrée fournit le **code (contenu/diff)** ET la **sortie brute** des commandes. Sortie verte sans code ≠ MERGE.
2. **Gemini n'a pas la Knowledge** — chaque US est **autoportante** : types copiés en clair, comportement décrit, aucun renvoi à un `.md`.
3. **Max 3 fichiers par US.** Au-delà → scinder + prévenir le PO.
4. **Une seule US `In Progress`** — pas de MERGE, pas de suite.
5. **Fixtures de test** typées via `Partial`/factory — jamais `as any`/`as unknown as T`.
6. **Format US et format Review** : voir le PROMPT (instructions perso). Toujours « À PROUVER (sortie de commande à coller) ».
7. `npx <outil>` ≡ `npm run <script>` (npm direct bloqué chez Gemini).
8. Le PO est **Product Owner non-dev** : pas de question technique à trancher seul, proposer une reco expliquée.

---

## 4. Sources vanilla à réclamer au PO (Phase 2)

Claude n'a PAS ces fichiers — les demander **une à une**, au moment d'attaquer l'US correspondante (ne pas tout demander d'un coup, économie de tokens) :

| US | Source vanilla à coller | Risque |
|---|---|---|
| US-011 | `src/store.js` | Moyen (upsert + auto-vault + transitions de state) |
| US-012 | `src/firebase.js` + `src/login.js` | Moyen (auth email link) |
| US-013 | (partie Firestore de `src/firebase.js` / `persistence.js`) | Moyen |
| US-014 | `src/persistence.js` | **Élevé** (bandeau DOM à transformer en `staleDataWarning`) |
| US-015 | `src/api.js` | **Élevé + gros** → à scinder |
| US-016 | `src/api.js` (`syncAnimeUpdates`, `startBackgroundRelationFetch`) | **Élevé** (batch 2/2s, throttle 1,1s) |
| US-017 | `src/recs.js` | **Élevé + gros** → à scinder |
| US-018 | `src/ui/toast.js`, gestion thème, + reprise download ICS & openMalImport | Faible |

---

## 5. Cadrage Phase 2 — le saut de risque

Phase 1 était du **portage pur** (copier-coller fidèle). Phase 2 est de l'**architecture réactive** — c'est là que la vigilance doit monter d'un cran :

- **Les `watch()` du store remplacent TOUT le bus DOM** (`document.dispatchEvent`/`CustomEvent`). En particulier `store:changed → saveToDatabase()` devient un `watch(animeCalendarData, ..., { deep: true })` **avec debounce**.
- **Gestion d'erreur obligatoire partout** : chaque `async` a un `try/catch` + un état d'erreur exposé (`ref<Error | null>`).
- **Les composables n'exposent que `readonly`/`computed`** vers l'extérieur, jamais l'état brut mutable.
- **Découpage plus fin qu'en Phase 1** : `useJikanApi` et `useRecommendations` dépasseront 3 fichiers → prévoir 2-3 sous-US chacun dès le cadrage.
- **Rate-limiting Jikan** (429, batch de 2 / 2 s, throttle 1,1 s) à répliquer fidèlement dans `useJikanApi`/`useSync`.
- **`usePersistence`** : le bandeau « données > 1 mois » que le vanilla injecte dans le `<body>` devient un **état réactif `staleDataWarning: Readonly<Ref<boolean>>`** — zéro DOM (cf. DEC-15/16 pour l'esprit).

### Ordre recommandé (dépendances)
```
US-011 stores/anime.ts        (socle — tout en dépend)
   ↓
US-012 useFirebaseAuth        (auth)
US-013 useFirestore           (load/save schedules/{uid})
US-014 usePersistence         (Firestore + fallback localStorage + staleDataWarning)
   ↓
US-015 useJikanApi            (à scinder : recherche/détail | saisons/top)
US-016 useSync                (batch throttlé — dépend de useJikanApi + store)
US-017 useRecommendations     (à scinder : pools/cache | batch/BYW/slot-fill)
   ↓
US-018 petits composables     useEpisodeInfo, useToast, useTheme, useICS(download), useMalImport
```

---

## 6. Dette ouverte à ne pas oublier (détail dans BACKLOG.md / DECISIONS.md)
- `useICS.ts` : download ICS (Blob + lien + iOS data-URI) + toast « rien à exporter » — pas encore porté.
- `useMalImport.ts` : partie impure de `mal-import.js` — pas encore portée.
- Bug studios inerte (DEC-11) — reproduit, à réparer post-migration via US métier.
- Dette UX boot (DEC-17) — `LoadingOverlay` réactif, à faire en Phase 4.
- Pin `jsdom ^29.1.1` (DEC-06) — vérifier résolution `npm install`.
- `getAnimeEpisodeInfo` : signature stricte `targetDate: Date` (pas de défaut) — `useEpisodeInfo` devra fournir `new Date()` lui-même si besoin d'un optionnel.

---

## 7. Faut-il rester sur la conversation actuelle ou repartir ?
**Reco : repartir sur une nouvelle conversation** dès la Phase 2. Raison : la conv 1 contient 10 US de code + reviews, chaque nouveau tour re-traite tout (coût + lenteur croissants) pour zéro valeur. La frontière Phase 1 / Phase 2 est nette, et tout le contexte utile est désormais dans la Knowledge (ce fichier + DECISIONS.md). Démarrer propre avec le prompt §2 ci-dessus.
