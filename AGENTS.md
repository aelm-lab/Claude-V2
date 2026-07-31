# Custom Agent Instructions

- **Diff Requirement**: Always show the exact code diffs or the entire file content when delivering a User Story (US) or code modifications.
- Reponses to users must include the code (content or diff) alongside the command outputs.

> **État de référence : fin S33 (cleaning S34).** Ce fichier est lu automatiquement par
> Gemini AI Studio — vérifie dans ta session que `AGENTS.md` et `AGENTS_E2E.md` apparaissent
> bien sous **Environment → Sources**, sinon ils ne sont peut-être pas réellement chargés.
> Toute mise à jour de règle doit être répercutée ici, pas seulement dans la Knowledge Claude.

---

## 1. Principe

Tu génères le code des User Stories fournies par le Tech Lead. Tu ne prends **aucune
décision d'architecture seul**. Si une spec semble contredire le code réel (nom de
fonction, type, event), tu le **signales** au lieu de forcer.

---

## 2. Règles de livraison (NON-NÉGOCIABLES)

**R-LIVRAISON-1 — Contenu intégral**

Toute réponse livrant du code inclut le contenu intégral de chaque fichier créé ou
modifié, du premier au dernier caractère. Un diff seul, un « show all diff », ou un récap
sans le contenu = livraison rejetée.

Exception tolérée pour un très gros fichier (ex. `style.css`) uniquement si une preuve
de substitution complète est fournie (diff exact + `grep -c` prouvant 0 occurrence résiduelle).

**R-LIVRAISON-2 — Sortie terminale littérale**

Toute commande exécutée est prouvée par sa session terminale brute : le prompt `$`,
la commande, et la sortie réelle (même vide). Jamais de paraphrase, jamais de
`# Command completed successfully`, jamais de résumé de sortie. Format attendu, exactement :

```
$ npx vue-tsc --noEmit
$
```

**R-LIVRAISON-3 (= R1) — Triple preuve verte (CI)**

Aucune livraison n'est « prête à merger » sans les trois sorties brutes :
- `npx vue-tsc --noEmit` (type-check strict, zéro erreur)
- `npx vitest run` (tous les tests verts — coller la sortie complète)
- `npm run build` (build prod réussi)

La CI (`.github/workflows/ci.yml`) rejoue ces trois étapes à chaque push.

---

## 3. Règles de périmètre (NON-NÉGOCIABLES)

**R-SCOPE-1 — Fichiers listés uniquement.** Ne créer/modifier QUE les fichiers
explicitement listés dans l'US. Toucher un fichier non listé → STOP, le signaler au
Tech Lead avant de continuer. Au démarrage de toute session, lister les fichiers modifiés
depuis le dernier merge connu AVANT toute action. Ne jamais « préparer le terrain » de ta
propre initiative. *(Incident le plus coûteux du projet à ce jour : 5 fichiers modifiés
sans US → 17/17 E2E cassés, une session entière perdue à réparer.)*

**R-SCOPE-2 — Pas d'amélioration hors scope.** Ne pas « améliorer » au passage du code
hors périmètre de l'US. Ne pas réécrire un fichier entier quand une correction ciblée suffit.

**R-SCOPE-3 — Max 3 fichiers par US.** Si l'US en demande plus, le Tech Lead l'aura
annoncé **en gras dans le titre de l'US** (dépassement assumé). Sinon, signaler avant de continuer.

## R2 — Test obligatoire sur l'orchestration
*(anciennement mal-étiqueté « R-SCOPE-3 » dans une version antérieure de ce fichier —
corrigé au cleaning S34 pour éviter la collision avec la vraie R-SCOPE-3 ci-dessus.)*

Toute US qui touche le boot, un store, ou le câblage entre composables livre ou
met à jour un test (unitaire ou smoke) prouvant le comportement runtime. Les bugs
d'orchestration ne se voient pas à la compilation.

---

## R4 — Test E2E obligatoire sur l'UI (gravé session 7)

Tout correctif issu d'un audit UX, et toute fonctionnalité touchant l'écran, livre un
test E2E Playwright qui :
1. reproduit le geste réel de l'utilisateur (clic, saisie, navigation) ;
2. asserte le résultat VISIBLE dans le DOM (élément visible/masqué), jamais l'état
   interne d'un store ;
3. est ROUGE sur le bug actuel, VERT après le fix, SANS être modifié (preuve que le
   filet capture bien le bug — méthode US-109).

Socle : `playwright.config.ts` + `tests/e2e/**` (exclu de Vitest). Auth des écrans
protégés bypassée en E2E via `import.meta.env.VITE_E2E_AUTH_BYPASS` (statique, mort en
prod — interdit en runtime). Fournir les sorties BRUTES rouge ET vert.

Détail complet de la stratégie de test → **`AGENTS_E2E.md`**.

---

## 4. Règles de code

**R-CODE-1 — Zéro `any`.** Aucun `any` (implicite ou explicite), aucun `as any`, aucun
`@ts-ignore`. Tout type vient du contrat de types fourni dans l'US. `eslint-disable-next-line`
ne corrige pas `TS6133` (variable inutilisée) : retirer la variable ou la préfixer `_`.

**R-CODE-2 — Fixtures de test typées.** Jamais `as unknown as T` pour une fixture. Utiliser
le helper `makeAnime(overrides: Partial<AnimeEntry>)`.

**R-CODE-3 — Séparation des responsabilités.**
- Composant `.vue` : UI + réactivité locale uniquement. Jamais de `fetch`, de
  `localStorage`/IndexedDB, ni de logique métier lourde.
- Composable : n'expose que `readonly`/`computed` vers l'extérieur.
- Store Pinia : aucun I/O.
- Utils : zéro import de Vue.

**R-CODE-4 — Zéro DOM direct, zéro bus DOM.** Pas de `document.getElementById/querySelector/
appendChild` ni `document.dispatchEvent/CustomEvent`. Utiliser `ref`/`v-if`/`v-for`, le
store Pinia, ou `emit`.

Exceptions documentées et seules autorisées : `document.createElement('a')` + `click()`
pour le download Blob (`useICS`), `<input file>.click()` pour l'import MAL, `DOMParser`
pour parser du XML (pur), et `getElementById('boot-loader').remove()` dans le `finally`
du `onMounted` d'`App.vue`.

**R-CODE-5 — Gestion d'erreur.** Chaque fonction async a un `try/catch` explicite et expose
un état d'erreur réactif. Gérer le 429 Jikan (retry/backoff). Ne jamais avaler une erreur
en silence sans log ni état.

**R-CODE-6 — Pas d'état sur `window`.** Aucun handler ni état stocké sur `window`. Utiliser
`onMounted`/`onUnmounted` + `@vueuse/core`.

**R-CODE-7 — Contrat d'event = le composant.** *(restauré au cleaning S34 — absent d'une
version antérieure de ce fichier alors que c'est la règle née des deux pires familles de
bugs du projet, P0.1 et P0.8.)* Un composant définit ses `defineEmits` ; les consommateurs
s'alignent sur ces noms. Ne jamais renommer un emit pour matcher un listener — corriger le
listener. Quand un composant à emits est réutilisé par N consommateurs, vérifier les N
alignements, pas un seul.

---

## R3 — Rappel : le « zéro-confiance » vaut aussi pour l'agent

Tout snippet est faillible, y compris ceux fournis par le Tech Lead : la preuve par
l'exécution (sorties brutes) prime toujours. Si une spec semble contredire le code réel
(ex. un nom de fonction, un type), le signaler au lieu de forcer — c'est arrivé
plusieurs fois et a évité des bugs.

---

## R6 — Audit live PO obligatoire avant clôture d'epic

Un sweep E2E vert NE SUFFIT PAS à clore un epic. Le faux-vert (test qui passe alors que
l'écran est cassé) est l'antipattern n°1 du projet. Avant toute clôture formelle d'epic
touchant l'UI, le PO réalise un audit live sur l'app déployée (viewport mobile réel) et
confirme visuellement. Le sweep prouve la non-régression ; l'œil du PO prouve
l'utilisabilité. Les deux sont requis.

---

## R7 — Invariant non-négociable : l'auteur du test ≠ l'auteur du code (NOUVEAU, gravé S33)

**Aucune exception, jamais** — même pour un test visuel jugé « simple » (ex. un test de
centrage/position). Tu ne rédiges jamais toi-même le test qui valide ton propre correctif,
ni en rouge ni en vert. Le Tech Lead fournit le test de fidélité ; tu le fais passer sans
le modifier. Une violation constatée (test E2E auto-écrit pour valider un correctif de
centrage de modale) a été écartée intégralement, sans aucune valeur de preuve, malgré un
code par ailleurs correct.
