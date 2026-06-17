# AGENTS_E2E.md — Agent de test technique & E2E (Gemini)

> **Où mettre ce fichier :** racine du dépôt (lu automatiquement par Gemini AI Studio,
> comme `AGENTS.md`) **et** dans la Knowledge du projet Claude Chat.
> **Rôle :** définir comment Gemini exécute et fait évoluer les tests technique (Vitest)
> et E2E (Playwright). Complète `AGENTS.md` ; ne le remplace pas.

---

## 1. Règle d'or (R5) : tester l'impact, pas l'univers

**Pendant un epic** — chaque US ne livre qu'un test **ciblé** sur ce qu'elle impacte :
- le geste précis du bug ou de la feature (un clic, une saisie, une navigation) ;
- l'assertion sur le **DOM visible** (R4), pas sur un état de store ;
- méthode ROUGE→VERT : le test échoue sur le code actuel (le bug), passe après le fix,
  **sans être modifié**. Fournir les DEUX sorties brutes.

**À la fin d'un epic** — un **grand check E2E complet** :
- `npx playwright test` **sans filtre** → toute la suite `tests/e2e/**` rejouée ;
- `npx vitest run` → toute la suite unit ;
- `npm run build` → build prod.
- Objectif : prouver qu'aucune US n'a régressé une autre. Fournir les 3 sorties brutes.

**Corollaire non négociable :** les tests E2E sont **cumulatifs**. On n'en supprime
jamais un pour « faire passer » le grand check. Si un test cumulé devient rouge au grand
check, c'est une **régression à corriger**, pas un test à retirer.

---

## 2. Périmètre d'un test ciblé d'US

Un test d'US couvre **uniquement** le comportement que l'US change. Exemples session 8 :
- P0.2 (LoadingOverlay) → `boot-loader.spec.ts` : un indicateur de chargement reste visible en continu au boot.
- P0.3a (dédup season) → `discover-season-dedup.spec.ts` : aucun `mal_id` affiché 2× sur `/discover/season`.
- P0.3c (dédup recherche) → `search-dedup.spec.ts` : aucune suggestion `mal_id` 2×.
- P0.8a (Add RecCard) → `reccard-add.spec.ts` : cliquer Add retire la carte.
- P0.8b (clic/dismiss RecCard) → `reccard-click-dismiss.spec.ts` : clic carte ouvre la modal ; Not interested retire la carte.
- P0.4 (feedback ajout) → `modal-add-feedback.spec.ts` : ajouter depuis la modal affiche un toast.

Ne PAS, dans une US, ajouter des assertions sur des fonctionnalités voisines non touchées
(ça transforme le test ciblé en test de régression et brouille le rouge→vert).

---

## 3. Socle technique (à connaître)

- **Playwright** : `playwright.config.ts` — `webServer: npm run build && npm run preview`,
  `baseURL: http://localhost:4173`, `env: { VITE_E2E_AUTH_BYPASS: 'true' }`.
- **Bypass auth** : `src/router/index.ts` `beforeEach` court-circuite si
  `import.meta.env.VITE_E2E_AUTH_BYPASS === 'true'` (lecture **statique** → éliminé du
  bundle prod, prouvé `grep -c=0`). **Jamais** en variable runtime.
- **Isolation runners** : `vite.config.ts` `test.exclude` ignore `tests/e2e/**`.
  Vitest = tests unit ; Playwright = E2E séparé.
- **Réseau déterministe** : mocker via `page.route('**/pattern**', route => route.fulfill(...))`.
  Ne JAMAIS taper l'API Jikan/Firebase live dans un E2E (flaky). Patterns utiles :
  `**/seasons/now**`, `**/seasons/upcoming**`, `**/anime?q=**`, `**/anime/**`.
- **localStorage** : la vraie clé de persistance est `'animeCalendar'`. Pour un état
  propre : `await page.addInitScript(() => window.localStorage.clear())` avant `goto`.
- **Sélecteurs réels (à lire dans le code, ne jamais deviner)** : carte reco/season
  `.card-cp-container` (titre `.card-cp-title`, cover `.card-cp-cover`, panneau why
  `.rec-why-clickable`) ; recherche `.search-input` / `.search-suggestion` /
  `.search-suggestion-title` ; modal `.modal-backdrop` / `.modal-content` (fermeture
  `Escape` ou clic backdrop) ; toast `.toast-notification` ; bouton add modal libellé `+ Add`.
  **Avant d'écrire un sélecteur dans un test, l'ouvrir dans le composant concerné.**

---

## 4. Preuves attendues (rappel R1/R4)

Pour une US avec test ciblé :
1. `npx vue-tsc --noEmit` (brut)
2. `npx vitest run` (brut, complet)
3. `npm run build` (brut, warnings + tableau dist inclus)
4. E2E **ROUGE** (avant le fix, état figé) — brut
5. E2E **VERT** (après le fix, test inchangé) — brut

Pour un grand check de fin d'epic :
1. `npx vitest run` (brut)
2. `npm run build` (brut)
3. `npx playwright test` **sans filtre** (brut, toute la suite)

Toute paraphrase d'une sortie = preuve rejetée, review suspendue.

---

## 5. Interdits spécifiques aux tests

- ❌ Asserter `ui.modalOpen`, `toast.visible`, ou tout état de store → asserter le **DOM**.
- ❌ Livrer un E2E « réparateur » sans sa sortie ROUGE pré-fix.
- ❌ Deviner un sélecteur ou une clé localStorage → les lire dans le code.
- ❌ Supprimer/désactiver un test cumulé pour faire passer le grand check.
- ❌ Mocker partiellement (laisser une vraie requête réseau fuir) → le test devient flaky.
- ❌ Rejouer une sortie ROUGE dans un état différent et la présenter comme la même.
