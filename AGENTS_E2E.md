# AGENTS_E2E.md — Stratégie de test technique & E2E (Gemini)

> Fichier racine, lu automatiquement par Gemini comme `AGENTS.md`. Complète `AGENTS.md`, ne le remplace pas.
>
> **État de référence : fin S33 (cleaning S34).** 🔴 **§8 (registre des batchs) marqué
> À AUDITER** — au moins 3 specs écrites récemment (`search-enriched`, `search-quick-add`,
> `search-hides-nav`) ne figurent dans aucun batch ci-dessous et n'ont donc probablement
> **jamais tourné au sweep**. Ne pas faire confiance à ce registre tant qu'il n'a pas été
> confronté à un `dir tests\e2e` / `ls tests/e2e` réel.

---

## 1. R5 — Tester l'impact, pas l'univers

**Pendant un epic** : chaque US ne livre qu'UN test ciblé sur ce qu'elle change :
- le geste précis (un clic, une saisie, une navigation) ;
- assertion sur le DOM VISIBLE (R4), jamais un état de store ;
- méthode ROUGE→VERT : échoue sur le code actuel, passe après le fix, SANS être modifié. Fournir les DEUX sorties brutes.

**Fin d'epic** : grand check complet — `npx vitest run`, `npm run build`, et la suite E2E complète. Objectif : prouver qu'aucune US n'a régressé une autre.

**Cumulatif** : les specs `tests/e2e/**` ne sont JAMAIS supprimées. Un test cumulé rouge au grand check = régression à corriger, pas test à retirer.

**Renforcé S33, sans exception :** l'auteur du test ≠ l'auteur du code — même pour un test
visuel jugé « simple » (position, centrage). Tu ne rédiges jamais toi-même le test qui
valide ton propre correctif. Voir `AGENTS.md` R7.

---

## 2. Sweep sandbox : par batchs `--workers=1`

Le sweep monolithique timeout à 60s dans le sandbox. Le parallèle provoque des `ERR_CONNECTION_REFUSED`. Lancer le grand check en **batchs ≤9 specs** avec `--workers=1` (voir scripts `package.json`). Fournir chaque sortie de batch brute.

---

## 3. R4 strict — assertion mobile = position, pas visibilité

Quand le placement/centrage est l'enjeu (cas mobile), asserter `boundingBox()` ou `getComputedStyle().position` — JAMAIS `toBeVisible()` seul. Un élément hors écran peut être "visible" au sens DOM. Faux-vert n°1 du projet : ne jamais asserter `ui.modalOpen`/`toast.visible`/un état de store.

**Récidive confirmée S33** : un test de centrage de modale assertait `max-height`/`overflow-y`
au lieu de la position réelle — alors que le placement était précisément le sujet du bug
signalé. Ce test a été écarté sans valeur de preuve. Vigilance permanente sur ce point précis
à chaque test touchant du positionnement.

---

## 4. RÈGLE GATING↔E2E (durcissement, angle mort session 12)

Tout `v-if` / gating conditionnel ajouté sur un **élément interactif** (bouton, lien, carte cliquable) DÉCLENCHE, dans la MÊME US, un grep des specs E2E qui ouvrent/cliquent cet élément. Si une spec cliquait l'élément désormais gaté, elle deviendra rouge au sweep, pas au merge. Vérifier et réaligner AVANT de livrer. (US-P0-E avait gaté "Mark done" → toast-labels rouge au sweep, 3 allers-retours perdus.)

---

## 5. SEED STANDARDISÉ (durcissement, anti faux-vert seed)

🔴 **Clé localStorage réelle : `aanime_calendar`** (corrigée au cleaning S34 — une version
antérieure de ce fichier référençait encore `'animeCalendar'`, la clé legacy pré-migration
US-133). L'ancienne clé fonctionne encore *indirectement* grâce à une migration de
compatibilité au boot, mais **tout nouveau seed doit utiliser `aanime_calendar`** — ne pas
s'appuyer sur un chemin de migration legacy qui pourrait disparaître.

Pattern de seed obligatoire pour tout test calendrier/semaine :

```ts
const days = ['monday','tuesday','wednesday','thursday','friday','saturday','sunday'];
const mockData = {
  timestamp: Date.now(),
  data: days.map((day, i) => ({
    mal_id: i + 1, id: i + 1, title: `Show ${i}`,
    state: 'calendar', day, airsTime: '12:00',
    status: 'Currently Airing', startDate: '2020-01-01T00:00:00.000Z',
  })),
};
await page.addInitScript((c) => {
  window.localStorage.clear();
  window.localStorage.setItem('aanime_calendar', c);
}, JSON.stringify(mockData));
```

Deux pièges gravés :
- **Seed mono-jour** (`day:'monday'` seul) = carte invisible si la semaine courante est un autre jour → toujours seeder les 7 jours.
- **Auto-vault** : un seed `state:'calendar'` + `status:'Finished Airing'` s'auto-vault au boot (`usePersistence`) et DISPARAÎT de la vue semaine. Pour tester une action sur un anime terminé, seed en `state:'watchlist'` (exclu de l'auto-vault) sur `/library/plan`.

---

## 6. Socle technique

- Playwright : `webServer: build && preview`, `baseURL: http://localhost:4173`, `env:{ VITE_E2E_AUTH_BYPASS:'true' }`, `timeout: 120000` (sandbox lent).
- Bypass auth : `router/index.ts` `beforeEach` court-circuite si `import.meta.env.VITE_E2E_AUTH_BYPASS === 'true'` (lecture STATIQUE, mort en prod, prouvable `grep -c=0`). Jamais en runtime.
- Isolation : `vite.config.ts` `test.exclude` ignore `tests/e2e/**`.
- Réseau déterministe : mocker via `page.route('**/pattern**', r => r.fulfill(...))`. Jamais l'API live (flaky — Jikan est d'ailleurs en panne externe depuis S33, raison de plus). Patterns : `**/seasons/now**`, `**/seasons/upcoming**`, `**/anime?q=**`, `**/anime/**`. Ne jamais mocker partiellement (une vraie requête qui fuit rend le test flaky).
- Boot : attendre `await expect(page.locator('#boot-loader')).toBeHidden({ timeout: 15000 })` AVANT tout clic (le loader fixed intercepte les pointer events).

---

## 7. Sélecteurs réels (lire dans le code, ne jamais deviner)

- Carte reco/season : `.card-cp-container` (titre `.card-cp-title`, cover `.card-cp-cover`, why `.rec-why-clickable`)
- Recherche : `.search-input` / `.search-suggestion` / `.search-suggestion-title`
- Modal : `.modal-backdrop` / `.modal-content` (fermeture Escape ou clic backdrop)
- Toast : `.toast-notification`
- Bouton add modal : libellé `+ Add`
- Carte semaine : `.rowcard` (bouton ✓ `.rc-mark-done`, barre progression `.rc-progress` / `.rc-progress-fill`)
- Jour courant : `.day-hdr.today`
- Navs : `.secondary-tab` / `.tab-item` / `.active`
- Login : `.login-brand`

**Avant d'écrire un sélecteur, l'ouvrir dans le composant concerné** — cette liste peut
avoir dérivé du code réel, elle n'est pas une garantie.

---

## 8. Registre des batchs E2E — 🔴 À AUDITER (voir bandeau en tête de fichier)

> Le découpage en batchs (scripts `test:e2e:batch1..3`) est FIGÉ EN DUR. Toute nouvelle
> spec DOIT être ajoutée à un batch dans `package.json` ET listée ici, sinon elle ne
> tourne jamais au sweep. Garder chaque batch ≤9 fichiers.
>
> **Dernier état connu (probablement périmé, non ré-audité depuis) :**

- **batch1** (9 fichiers) : `auto-vault-toast`, `boot-loader`, `calendar-subnav-layout`, `discover-season-dedup`, `foryou-dedup`, `login-styled`, `modal-add-appears-on-week`, `modal-add-feedback`, `modal-add-removes-from-discover`
- **batch2** (9 fichiers) : `modal-content-centered-mobile`, `modal-open`, `modal-position`, `modal-status-gating`, `month-layout`, `nav-active-state`, `onair-subnav`, `reccard-add`, `reccard-click-dismiss`
- **batch3** (8 fichiers) : `search-dedup`, `smoke`, `snap-to-today`, `toast-labels`, `toast-visible-mobile`, `week-mark-done`, `week-no-duplicate-period`, `week-progress-bar`

**Connus comme absents de cette liste (à ajouter dès confirmation de leur existence) :**
`search-enriched`, `search-quick-add`, `search-hides-nav`, `logout-modal-position`.

**Action requise** : au prochain grand check E2E, lister `tests/e2e/` réellement présent
sur disque (`dir tests\e2e` / `ls tests/e2e`), comparer à cette liste, et mettre à jour les
deux (ce fichier + `package.json`) en conséquence. Tant que ce n'est pas fait, un "sweep
vert" ne prouve PAS l'absence de régression sur les specs orphelines.

---

## 9. Interdits

- ❌ Asserter un état de store → asserter le DOM.
- ❌ Livrer un E2E réparateur sans sa sortie ROUGE pré-fix.
- ❌ Deviner un sélecteur / une clé localStorage (clé = `aanime_calendar`, pas `'animeCalendar'`).
- ❌ Supprimer/désactiver une spec cumulée pour faire passer le grand check.
- ❌ Écrire une nouvelle spec sans l'ajouter au batch correspondant dans `package.json` ET au §8 de ce fichier.
- ❌ Mock partiel (laisser une requête fuir) → flaky.
- ❌ `toBeVisible()` seul quand la position est l'enjeu (cas mobile).
- ❌ **Écrire soi-même le test qui valide son propre correctif — aucune exception**, même
  pour un test visuel jugé simple. Voir `AGENTS.md` R7.
