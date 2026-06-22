# AGENTS.md — Gouvernance permanente Gemini

> Fichier lu automatiquement par Gemini AI Studio (racine du dépôt). Gemini n'a PAS
> accès à la Knowledge Claude → ces règles sont sa seule source de gouvernance permanente.
> Chaque US reste autoportante ; ce fichier ne se répète pas dans les US.
>
> **État de référence : session 16.** 84 unit · 26 specs / 30 tests E2E · build ~717 kb.

## 1. Principe

Tu génères le code des User Stories fournies par le Tech Lead. Tu ne prends **aucune
décision d'architecture seul**. Si une spec semble contredire le code réel (nom de
fonction, type, event), tu le **signales** au lieu de forcer — c'est arrivé plusieurs
fois et a évité des bugs (DEC-11/16/21/24, P0.1).

## 2. Règles de livraison (NON-NÉGOCIABLES)

**R-LIVRAISON-1 — Contenu intégral.** Toute réponse livrant du code inclut le contenu
intégral de chaque fichier créé ou modifié, du premier au dernier caractère. Un diff
seul, un « show all diff », ou un récap sans contenu = livraison rejetée. Exception très
gros fichier (ex. `style.css`) : diff exact + `grep -c` prouvant 0 occurrence résiduelle.

**R-LIVRAISON-2 — Sortie terminale littérale.** Toute commande est prouvée par sa
session brute : le prompt `$`, la commande, et la sortie réelle (même vide). Jamais de
paraphrase, jamais de `# Command completed successfully`, jamais de `Build succeeded`,
jamais de résumé. (4 récidives constatées — toute paraphrase = review suspendue d'office.)
Format attendu, exactement :

```
$ npx vue-tsc --noEmit
$
```

**R-LIVRAISON-3 — Triple preuve verte (CI).** Aucune livraison n'est « prête à merger »
sans les **trois sorties brutes, SÉPARÉES** (pas de chaîne `&&`) :
- `npx vue-tsc --noEmit` (type-check strict, zéro erreur)
- `npx vitest run` (tous les tests verts — sortie complète)
- `npm run build` (build prod réussi — warnings Vite réels + tableau `dist/`)

La CI (`.github/workflows/ci.yml`) rejoue ces trois étapes à chaque push.
Si le sandbox tronque la sortie build : `npm run build 2>&1 | tail -40`.

## 3. Règles de périmètre (NON-NÉGOCIABLES)

**R-SCOPE-1 — Fichiers listés uniquement.** Ne créer/modifier QUE les fichiers
explicitement listés dans l'US. Toucher un fichier non listé → STOP, le signaler au
Tech Lead avant de continuer. **Au démarrage de toute session, lister les fichiers modifiés
depuis le dernier merge connu AVANT toute action** (le sandbox interdit `git status` → à
défaut `ls -la` / liste manuelle). Ne jamais « préparer le terrain » de sa propre initiative.
*(Violation la plus coûteuse du projet : 5 fichiers modifiés sans US → 17/17 E2E cassés, une session de réparation.)*

**R-SCOPE-2 — Pas d'amélioration hors scope.** Ne pas « améliorer » au passage du code
hors périmètre. Ne pas réécrire un fichier entier quand une correction ciblée suffit.

**R-SCOPE-3 — Max 3 fichiers par US.** Si l'US en demande plus, le Tech Lead l'aura
**annoncé en gras dans le titre de l'US** (dépassement assumé). Sans cette annonce, un
dépassement = scinder + prévenir (sauf suppression pure prouvée).

**R-SCOPE-4 — Test obligatoire sur l'orchestration.** Toute US qui touche le boot, un
store, ou le câblage entre composables livre/met à jour un test (unit ou smoke) prouvant
le comportement runtime. Les bugs d'orchestration ne se voient pas à la compilation.

## 4. R4 — Test E2E obligatoire sur l'UI

Tout correctif issu d'un audit UX, et toute fonctionnalité touchant l'écran, livre un
test E2E Playwright qui :
1. reproduit le geste réel de l'utilisateur (clic, saisie, navigation) ;
2. asserte le résultat VISIBLE dans le DOM (élément visible/masqué/compté), jamais l'état
   interne d'un store ;
3. est ROUGE sur le bug actuel, VERT après le fix, SANS être modifié (méthode US-109).

Une preuve ROUGE = **un état figé unique**, jamais rejouée dans un autre état et présentée
comme la même. Fournir les DEUX sorties brutes (rouge ET vert).

Socle : `playwright.config.ts` + `tests/e2e/**` (exclu de Vitest via `vite.config.ts`
`test.exclude`). Auth des écrans protégés bypassée via `import.meta.env.VITE_E2E_AUTH_BYPASS`
(statique, mort en prod — prouver `grep -c=0`). Réseau déterministe via `page.route()`
(jamais l'API live). **La vraie clé localStorage de persistance est `aanime_calendar`**
(US-133 — toutes les clés préfixées `aanime_` ; ne pas deviner, ne pas utiliser l'ancienne
`'animeCalendar'`). **Pattern boot-dépendant** : attendre
`await expect(page.locator('#boot-loader')).toBeHidden({ timeout: 15000 })` avant tout clic
(le loader DEC-72 intercepte les pointer events). Seeder 7 jours pour garantir une carte visible.

**R4-bis — gating ↔ E2E (gravé s12).** Tout `v-if` ajouté sur un élément interactif
(bouton/lien) → grep des specs E2E qui ouvrent/cliquent cet élément, **dans la même US**,
avant merge. (US-P0-E a gaté « Mark done » → `toast-labels` est devenu rouge au sweep, pas au merge.)

## 5. R5 — Test ciblé par US, grand check en fin d'epic

Pendant un epic, chaque US ne livre qu'un E2E **ciblé** sur ce qu'elle impacte. À la **fin
de l'epic**, un **grand check E2E complet** rejoue toute la suite = régression globale. Les
tests E2E sont **cumulatifs** dans `tests/e2e/`, jamais supprimés. (Détail `AGENTS_E2E.md`.)
**Sandbox** : `--workers=1` (le parallèle provoque des `ERR_CONNECTION_REFUSED` faux-négatifs).
Suite > timeout sandbox (~60 s) → lancer en lots `test:e2e:batch1..3` (≤9 specs) + `test:e2e:sweep`.

## 6. R6 — Audit PO live avant clôture d'epic

Aucun epic n'est clos formellement sans un **audit live du PO** (walkthrough navigateur sur
l'app déployée). Le code vert + E2E vert prouvent la mécanique, pas l'expérience réelle.

## 7. Règles de code

**R-CODE-1 — Zéro `any`.** Aucun `any` (implicite/explicite), `as any`, `@ts-ignore`.
Tout type vient du contrat fourni dans l'US. `eslint-disable-next-line` ne corrige pas
`TS6133` : retirer la variable ou la préfixer `_`.

**R-CODE-2 — Fixtures de test typées.** Jamais `as unknown as T`. Helper `makeAnime(overrides: Partial<AnimeEntry>)`.
*(Audit s16 : le chemin legacy de `usePersistence` viole cette règle — `as unknown as AnimeEntry[]` → US-158.)*

**R-CODE-3 — Séparation des responsabilités.** Composant `.vue` : UI + réactivité locale.
Jamais de `fetch`/`localStorage`/IndexedDB ni logique métier lourde. Composable : n'expose
que `readonly`/`computed`. Store Pinia : aucun I/O. Utils : zéro import Vue.
*(Audit s16 : `usePersistence` mute le store hors action + porte des toasts → US-157.)*

**R-CODE-4 — Zéro DOM direct, zéro bus DOM.** Pas de `document.getElementById/querySelector/
appendChild` ni `document.dispatchEvent/CustomEvent`. Utiliser `ref`/`v-if`/`v-for`, le
store, ou `emit`. Exceptions seules autorisées : `document.createElement('a')` + `click()`
(download Blob `useICS`), `<input file>.click()` (import MAL), `DOMParser` (XML pur),
`getElementById('boot-loader').remove()` dans le `finally` du `onMounted` d'`App.vue` (DEC-72).

**R-CODE-5 — Gestion d'erreur.** Chaque fonction async a un `try/catch` explicite et expose
un état d'erreur réactif. Gérer le 429 Jikan (retry/backoff). Ne jamais avaler en silence.
*(Audit s16 : `saveToDatabase`/`saveSchedule` viole cette règle → US-153 P0. Erreurs rate-limit avalées en `console.warn` → P2.)*

**R-CODE-6 — Pas d'état sur `window`.** `onMounted`/`onUnmounted` + `@vueuse/core`.

**R-CODE-7 — Contrat d'event = le composant.** Un composant définit ses `defineEmits` ;
les consommateurs s'alignent sur ces noms. Ne jamais renommer un emit pour matcher un
listener — corriger le listener. Quand un composant à emits est réutilisé par N
consommateurs, vérifier les N alignements. (Leçons P0.1, P0.8a/b.)

## 8. Impact utilisateur + reco (rappel pour le Tech Lead)

Le PO est non-technique : chaque US et chaque décision technique énonce **l'impact
utilisateur concret** (ce que l'utilisateur voit/ressent, ou « aucun visible — dette ») et
**une recommandation**. Cette règle pilote la rédaction des specs, pas le code de Gemini,
mais elle est rappelée ici pour cohérence.

## 9. Rappel : le « zéro-confiance » vaut aussi pour l'agent

Tout snippet est faillible, y compris ceux du Tech Lead : la preuve par l'exécution
(sorties brutes) prime toujours. Si une spec contredit le code réel, le signaler. Une
sortie « X passed » peut être structurellement impossible (variable non déclarée, fonction
jamais appelée) — la cohérence interne du test prime sur le compteur affiché.
