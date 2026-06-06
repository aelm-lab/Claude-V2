# DECISIONS.md — Journal des décisions d'architecture

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
>
> **Rôle :** capturer le **contexte « mou »** (décisions prises en conversation, non écrites ailleurs) pour qu'une nouvelle instance de Claude ne les réinvente pas. Chaque décision = quoi + pourquoi + impact.

---

## Format
`[ID] Décision — Raison — Impact`

---

## Décisions de session 1 (Phase 0 + Phase 1)

### Scaffold / outillage

- **[DEC-01] US-001b absorbée dans US-001.** `App.vue` + `index.html` ne rentraient pas dans la limite 3 fichiers de US-001, d'où une US-001b. Gemini a finalement tout fait en une passe (5 fichiers, signalé et autorisé). → US-001b **n'existe plus**, ne pas la relancer.

- **[DEC-02] ESLint = flat config + `@vue/eslint-config-typescript@^14`.** Le v13 est l'ancien format eslintrc, **incompatible** avec la flat config. Config retenue :
  ```js
  import pluginVue from 'eslint-plugin-vue';
  import { defineConfigWithVueTs, vueTsConfigs } from '@vue/eslint-config-typescript';
  export default defineConfigWithVueTs(
    pluginVue.configs['flat/essential'],
    vueTsConfigs.recommended,
    { rules: { '@typescript-eslint/no-explicit-any': 'error' } },
  );
  ```
  → Impact : `no-explicit-any` est en **erreur** (prouvé en US-002).

- **[DEC-03] `tsconfig.node.json` séparé** pour isoler `vite.config.ts` du projet applicatif (`composite: true`, `types: ['node']`).

- **[DEC-04] `npx <outil>` ≡ `npm run <script>`.** L'environnement d'exécution de Gemini bloque `npm` direct. Accepté.

- **[DEC-05] Conteneur Gemini tourne nativement en UTC.** L'exigence `TZ=UTC` des tests jst est satisfaite de fait (vérifié par les résultats eux-mêmes). Ne pas exiger un préfixe `TZ=UTC` explicite, il échoue dans leur shell.

- **[DEC-06] `jsdom` installé en devDep (US-010)** pour `DOMParser` dans les tests `malImport`. Déclaré par fichier via `// @vitest-environment jsdom`. ⚠️ **À vérifier** : le pin `^29.1.1` semble élevé — confirmer qu'un `npm install` propre le résout (sinon ajuster).

### Typage

- **[DEC-07] DTO de plomberie = types locaux, pas dans le contrat.** `RawAnime` (normalize) et `MalImportEntry` (malImport) sont **locaux non exportés**. `MalImportResult` est exporté mais reste près de sa fonction. Le contrat `TYPES_CONTRACT.md` est réservé aux types **métier**.

- **[DEC-08] Fixtures de test typées via helper `Partial` ou factory complet.** Interdit : `as any`, `as unknown as T`. Deux patterns acceptés et utilisés :
  - helper léger : `const anime = (o: Partial<AnimeEntry>): AnimeEntry => o as AnimeEntry;`
  - factory complet (préféré quand l'objet doit être valide) : `createAnime(overrides)` qui remplit tous les champs requis.

- **[DEC-09] US-008-types : extension du contrat (ajouts only, rien renommé).** Ajoutés : `RecSignalKind`, `RecSignal.kind?`, `HistoryItem.completedAt?`/`recencyBucket?`, et sur `AnimeEntry` : `studios?`, `popularityScore?`, `_relevanceScore?`, `_presetScore?`. Import circulaire **type-only** `recs.ts → anime.ts` (pour `RecencyBucket`) — sans danger (effacé à la compilation).

### Fidélité fonctionnelle (rec-engine)

- **[DEC-10] (Décision D) Branches mortes `typeof x === 'string' ? x : x.name` simplifiées.** Après `normalize`, `genres`/`themes`/`studios` sont **toujours `string[]`** → la branche `.name` est injoignable ET ne compile pas en TS strict. Simplifiée en `const display = x;`. **Comportement identique.**

- **[DEC-11] (Décision E) Bug `item.studios` reproduit tel quel.** `scorePool` lit `item.studios` (pluriel) que `normalize` ne produit jamais (il produit `studio` singulier) → scoring studio **inerte**. On **ne corrige pas** (règle : copier le comportement). Champ `studios?: string[]` ajouté pour typer. Réparation = US métier **après** la migration.

- **[DEC-12] `decayMultiplier = 0.2` par défaut conservé** dans `buildTasteProfile` (semble bas mais c'est le vanilla). Conséquence vérifiée en test : un item `heart` + `recencyBucket:'recent'` SANS `completedAt` donne un poids de trait = `2.0 × 1.0 × 0.2 = 0.4`.

- **[DEC-13] `priority` du tri des signaux typé `Record<RecSignalKind, number>` avec `score: 0`.** Le vanilla ne gérait pas `score` dans le map (le signal score est ajouté APRÈS le tri, donc jamais présent au tri). Ajout de `score: 0` = correction de **typage**, pas de comportement.

- **[DEC-14] `extractBecauseYouWatched` : param `profile` inutilisé → préfixé `_profile`.** Gardé pour préserver la signature publique (le vanilla ne l'utilisait pas non plus).

### Découpage ICS / MAL (séparation pur / impur)

- **[DEC-15] `generateICSFile` scindé.** Seule la **génération de texte** (`buildICSContent`, pure) est portée en `utils/ics.ts`. Le **téléchargement** (Blob, lien `<a>`, cas iOS data-URI) + toast « rien à exporter » → `useICS.ts` en **Phase 2**.

- **[DEC-16] `openMalImport` reporté.** Seul `parseMalXml` (pur) est porté en `utils/malImport.ts`. La partie impure (input fichier, FileReader, `addAnimeSilent`, `dispatchEvent`, toast) → `useMalImport.ts` en **Phase 2**.

### UX

- **[DEC-17] Dette UX boot (à traiter Phase 4).** Au démarrage, l'app charge auth + Firestore avant le premier rendu. Prévoir un `LoadingOverlay` **piloté par état réactif** (pas en dur dans `index.html`) pour éviter le flash blanc. Gemini avait bricolé un overlay inline en US-001 — retiré, à refaire proprement.
