# AGENTS_E2E.md — Agent de test technique & E2E (Gemini)

> **Où mettre ce fichier :** racine du dépôt (lu automatiquement par Gemini AI Studio,
> comme `AGENTS.md`) **et** dans la Knowledge du projet Claude Chat.
> **Rôle :** définir comment Gemini exécute et fait évoluer les tests technique (Vitest)
> et E2E (Playwright). Complète `AGENTS.md` ; ne le remplace pas.
>
> **État de référence : session 16.** Suite cumulative **26 specs / 30 tests** · **84** unit.

---

## 1. Règle d'or (R5) : tester l'impact, pas l'univers

**Pendant un epic** — chaque US ne livre qu'un test **ciblé** sur ce qu'elle impacte :
- le geste précis du bug ou de la feature (un clic, une saisie, une navigation) ;
- l'assertion sur le **DOM visible** (R4), pas sur un état de store ;
- méthode ROUGE→VERT : le test échoue sur le code actuel (le bug), passe après le fix,
  **sans être modifié**. Fournir les DEUX sorties brutes.

**À la fin d'un epic** — un **grand check E2E complet** :
- toute la suite `tests/e2e/**` rejouée + `npx vitest run` + `npm run build`.
- Objectif : prouver qu'aucune US n'a régressé une autre. Fournir les sorties brutes.

**Corollaire non négociable :** les tests E2E sont **cumulatifs**. On n'en supprime
jamais un pour « faire passer » le grand check. Un test cumulé rouge = **régression à
corriger**, pas un test à retirer.

---

## 2. Exécution en lots (sandbox Gemini) — OBLIGATOIRE

La suite complète dépasse le timeout sandbox (~60 s) et le mode parallèle provoque des
`ERR_CONNECTION_REFUSED` faux-négatifs. Donc :

- **`--workers=1`** toujours (jamais `fullyParallel` dans le sandbox).
- Lancer par lots via les scripts `package.json` : `test:e2e:batch1`, `test:e2e:batch2`,
  `test:e2e:batch3` (≤9 specs chacun) + `test:e2e:sweep`.
- La CI GitHub Actions n'a pas ces contraintes (elle rejoue tout d'un bloc).

---

## 3. Périmètre d'un test ciblé d'US

Un test d'US couvre **uniquement** le comportement que l'US change. Ne PAS ajouter
d'assertions sur des fonctionnalités voisines non touchées (ça transforme le test ciblé en
test de régression et brouille le rouge→vert).

---

## 4. Socle technique (à connaître)

- **Playwright** : `playwright.config.ts` — `webServer: build && preview`, `baseURL:
  http://localhost:4173`, `env: { VITE_E2E_AUTH_BYPASS: 'true' }`, **`timeout: 120000`** (sandbox lent).
- **Bypass auth** : `src/router/index.ts` `beforeEach` court-circuite si
  `import.meta.env.VITE_E2E_AUTH_BYPASS === 'true'` (lecture **statique** → éliminé du
  bundle prod, prouvé `grep -c=0`). **Jamais** en variable runtime.
- **Isolation runners** : `vite.config.ts` `test.exclude` ignore `tests/e2e/**`.
- **Réseau déterministe** : mocker via `page.route('**/pattern**', route => route.fulfill(...))`.
  Ne JAMAIS taper l'API Jikan/Firebase live (flaky). Patterns : `**/seasons/now**`,
  `**/seasons/upcoming**`, `**/anime?q=**`, `**/anime/**`. **Ne jamais mocker partiellement**
  (une vraie requête qui fuit rend le test flaky).
- **localStorage** : la vraie clé de persistance est **`aanime_calendar`** (US-133 — toutes
  les clés préfixées `aanime_`). **Ne pas utiliser l'ancienne `'animeCalendar'`.** État
  propre : `await page.addInitScript(() => window.localStorage.clear())` avant `goto`.
- **Pattern boot-dépendant** : avant tout clic après `goto`, attendre
  `await expect(page.locator('#boot-loader')).toBeHidden({ timeout: 15000 })`. Le `#boot-loader`
  (DEC-72, `position: fixed`) intercepte les pointer events tant que le boot tourne.
- **Seed 7 jours** : seeder un anime par jour (`['monday'..'sunday'].map(...)`) pour garantir
  une carte visible quel que soit le jour courant. Un seed mono-jour = carte invisible hors ce jour.
- **Auto-vault vs seed** : un seed `calendar` + `Finished` s'auto-vault au boot et disparaît.
  Pour tester une action sur un anime terminé, seeder en **`watchlist`** (exclu de l'auto-vault).

### Sélecteurs réels (à lire dans le code, ne jamais deviner)
Carte reco/season `.card-cp-container` (titre `.card-cp-title`, cover `.card-cp-cover`,
panneau why `.rec-why-clickable`) · recherche `.search-input` / `.search-suggestion` /
`.search-suggestion-title` · modal `.modal-backdrop` / `.modal-content` (fermeture `Escape`
ou clic backdrop) · toast `.toast-notification` · carte semaine `.rowcard` (bouton ✓
`.rc-mark-done`, barre progression `.rc-progress` / `.rc-progress-fill`) · jour courant
`.day-hdr.today` · navs `.secondary-tab` / `.tab-item` / `.active` · login `.login-brand`.
**Avant d'écrire un sélecteur, l'ouvrir dans le composant concerné.**

---

## 5. Preuves attendues (rappel R1/R4)

Pour une US avec test ciblé : `vue-tsc --noEmit` (brut) · `vitest run` (brut, complet) ·
`build` (brut) · E2E **ROUGE** (avant fix, état figé) · E2E **VERT** (après fix, test inchangé).
Pour un grand check fin d'epic : `vitest run` + `build` + suite E2E complète (par lots batch1..3 + sweep).
Toute paraphrase d'une sortie = preuve rejetée, review suspendue.

---

## 6. Interdits spécifiques aux tests

- ❌ Asserter `ui.modalOpen`, `toast.visible`, ou tout état de store → asserter le **DOM**.
- ❌ Asserter la visibilité sans la position quand le placement est l'enjeu → `getComputedStyle().position` / `boundingBox`.
- ❌ Livrer un E2E « réparateur » sans sa sortie ROUGE pré-fix.
- ❌ Deviner un sélecteur ou une clé localStorage → les lire dans le code (clé = `aanime_calendar`).
- ❌ Supprimer/désactiver un test cumulé pour faire passer le grand check.
- ❌ Mocker partiellement (laisser une vraie requête réseau fuir).
- ❌ Rejouer une sortie ROUGE dans un état différent et la présenter comme la même.
- ❌ Accepter un « X passed » sans vérifier la cohérence interne du test (variable déclarée ? fonction réellement appelée ?).
