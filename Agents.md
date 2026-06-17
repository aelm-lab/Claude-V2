# AGENTS.md — Gouvernance permanente Gemini

> Fichier lu automatiquement par Gemini AI Studio (racine du dépôt). Gemini n'a PAS
> accès à la Knowledge Claude → ces règles sont sa seule source de gouvernance permanente.
> Chaque US reste autoportante ; ce fichier ne se répète pas dans les US.

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
jamais de résumé. (3 récidives constatées — toute paraphrase = review suspendue d'office.)
Format attendu, exactement :

```
$ npx vue-tsc --noEmit
$
```

**R-LIVRAISON-3 — Triple preuve verte (CI).** Aucune livraison n'est « prête à merger »
sans les **trois sorties brutes, SÉPARÉES** (pas de chaîne `&&`) :
- `npx vue-tsc --noEmit` (type-check strict, zéro erreur)
- `npx vitest run` (tous les tests verts — sortie complète)
- `npm run build` (build prod réussi — y compris les warnings Vite réels et le tableau `dist/`)

La CI (`.github/workflows/ci.yml`) rejoue ces trois étapes à chaque push.

## 3. Règles de périmètre (NON-NÉGOCIABLES)

**R-SCOPE-1 — Fichiers listés uniquement.** Ne créer/modifier QUE les fichiers
explicitement listés dans l'US. Toucher un fichier non listé → STOP, le signaler au
Tech Lead avant de continuer.

**R-SCOPE-2 — Pas d'amélioration hors scope.** Ne pas « améliorer » au passage du code
hors périmètre. Ne pas réécrire un fichier entier quand une correction ciblée suffit.

**R-SCOPE-3 — Max 3 fichiers par US.** Si l'US en demande plus, c'est un signal : le
Tech Lead l'aura **annoncé en gras dans le titre de l'US** (dépassement assumé). Sans
cette annonce explicite, un dépassement = scinder + prévenir (sauf suppression pure prouvée).

**R-SCOPE-4 — Test obligatoire sur l'orchestration.** Toute US qui touche le boot, un
store, ou le câblage entre composables livre/met à jour un test (unit ou smoke) prouvant
le comportement runtime. Les bugs d'orchestration ne se voient pas à la compilation.

## 4. R4 — Test E2E obligatoire sur l'UI (gravé session 7)

Tout correctif issu d'un audit UX, et toute fonctionnalité touchant l'écran, livre un
test E2E Playwright qui :
1. reproduit le geste réel de l'utilisateur (clic, saisie, navigation) ;
2. asserte le résultat VISIBLE dans le DOM (élément visible/masqué/compté), jamais l'état
   interne d'un store ;
3. est ROUGE sur le bug actuel, VERT après le fix, SANS être modifié (méthode US-109).

Une preuve ROUGE = **un état figé unique**, jamais rejouée dans un autre état et présentée
comme la même. Fournir les DEUX sorties brutes (rouge ET vert).

Socle : `playwright.config.ts` + `tests/e2e/**` (exclu de Vitest via `vite.config.ts`
`test.exclude`). Auth des écrans protégés bypassée en E2E via
`import.meta.env.VITE_E2E_AUTH_BYPASS` (statique, mort en prod — interdit en runtime,
prouver `grep -c=0` sur les chunks). Pour un E2E déterministe, mocker le réseau via
`page.route()` (jamais l'API live). La vraie clé localStorage de persistance est
`'animeCalendar'` (ne pas deviner).

## 5. R5 — Test ciblé par US, grand check en fin d'epic (gravé session 8)

Pendant un epic, chaque US ne livre qu'un E2E **ciblé** sur ce qu'elle impacte (le geste
précis du bug). On ne re-teste pas tout l'univers à chaque US. À la **fin de l'epic**, un
**grand check E2E complet** rejoue toute la suite (`npx playwright test` sans filtre) =
régression globale. Les tests E2E sont **cumulatifs** dans `tests/e2e/`, jamais supprimés
— c'est ce qui rend le grand check possible. (Détail dans `AGENTS_E2E.md`.)

## 6. Règles de code

**R-CODE-1 — Zéro `any`.** Aucun `any` (implicite/explicite), `as any`, `@ts-ignore`.
Tout type vient du contrat fourni dans l'US. `eslint-disable-next-line` ne corrige pas
`TS6133` (variable inutilisée) : retirer la variable ou la préfixer `_`.

**R-CODE-2 — Fixtures de test typées.** Jamais `as unknown as T`. Helper `makeAnime(overrides: Partial<AnimeEntry>)`.

**R-CODE-3 — Séparation des responsabilités.** Composant `.vue` : UI + réactivité locale.
Jamais de `fetch`/`localStorage`/IndexedDB ni logique métier lourde. Composable : n'expose
que `readonly`/`computed`. Store Pinia : aucun I/O. Utils : zéro import Vue.

**R-CODE-4 — Zéro DOM direct, zéro bus DOM.** Pas de `document.getElementById/querySelector/
appendChild` ni `document.dispatchEvent/CustomEvent`. Utiliser `ref`/`v-if`/`v-for`, le
store, ou `emit`. Exceptions seules autorisées : `document.createElement('a')` + `click()`
(download Blob `useICS`), `<input file>.click()` (import MAL), `DOMParser` (XML pur).

**R-CODE-5 — Gestion d'erreur.** Chaque fonction async a un `try/catch` explicite et expose
un état d'erreur réactif. Gérer le 429 Jikan (retry/backoff). Ne jamais avaler en silence.

**R-CODE-6 — Pas d'état sur `window`.** `onMounted`/`onUnmounted` + `@vueuse/core`.

**R-CODE-7 — Contrat d'event = le composant.** Un composant définit ses `defineEmits` ;
les consommateurs (pages, wrappers) **s'alignent** sur ces noms. Ne jamais renommer un
emit pour matcher un listener — corriger le listener. Quand un composant à emits est
réutilisé par N consommateurs, vérifier les N alignements. (Leçons P0.1, P0.8a/b.)

## 7. Rappel : le « zéro-confiance » vaut aussi pour l'agent

Tout snippet est faillible, y compris ceux du Tech Lead : la preuve par l'exécution
(sorties brutes) prime toujours. Si une spec contredit le code réel, le signaler.
