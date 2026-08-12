# AGENTS.md — Contrat de l'agent d'implémentation (Gemini AI Studio)

> **Emplacement canonique :** écrit dans `aelm-lab/Claude-V2` (Knowledge Claude), **déployé à
> la racine de `aelm-lab/A-Anime`** à chaque clôture de sprint. Jamais édité à la destination.
> **Vérification côté Gemini :** ce fichier doit apparaître sous **Environment → Sources**
> dans ta session AI Studio. S'il n'y est pas, il n'est pas réellement chargé.
>
> **Fusionne et remplace `AGENTS.md` + `AGENTS_E2E.md`** (SE-049). La séparation en deux
> fichiers a produit trois dérives réelles : `R-SCOPE-1/2` et `R-CODE-7` absents du fichier
> réellement lu, et la clé localStorage legacy survivant dans l'un alors qu'elle était
> interdite dans l'autre. Un seul fichier, une seule synchro.
>
> **Tu n'as pas accès à la Knowledge du projet.** Chaque US t'est fournie autoportante :
> tout le contexte, tous les types, tous les tests sont dedans. Les chiffres de ce document
> sont en dur pour cette raison — ils sont resynchronisés à chaque sprint.

---

## 1. Principe

Tu génères le code des User Stories fournies par le Tech Lead. Tu ne prends **aucune décision
d'architecture seul**. Si une spec semble contredire le code réel (nom de fonction, type,
event), tu le **signales** au lieu de forcer — c'est arrivé plusieurs fois et a évité des bugs.

**Zéro-confiance, y compris envers le Tech Lead.** Tout snippet est faillible, même fourni
dans l'US. La preuve par l'exécution (sorties brutes) prime toujours sur l'affirmation.

---

## 2. Règles de livraison — NON NÉGOCIABLES

**R-LIVRAISON-1 — Contenu intégral.**
Toute réponse livrant du code inclut le contenu **intégral** de chaque fichier créé ou modifié,
du premier au dernier caractère. Un diff seul, un « show all diff », ou un récapitulatif sans
le contenu = livraison rejetée.
*Exception tolérée* pour un très gros fichier (ex. `style.css`) uniquement si une preuve de
substitution complète est fournie : diff exact + `grep -c` prouvant 0 occurrence résiduelle.

**R-LIVRAISON-2 — Sortie terminale littérale.**
Toute commande exécutée est prouvée par sa session terminale brute : le prompt `$`, la
commande, et la sortie réelle (même vide). Jamais de paraphrase, jamais de
`# Command completed successfully`, jamais de résumé. Format attendu, exactement :

```
$ npm run type-check
$
```

**R-LIVRAISON-3 (= R1) — Triple preuve verte.**
Aucune livraison n'est « prête à merger » sans les **trois sorties brutes séparées** :

```
npm run type-check      (vue-tsc --noEmit, zéro erreur)
npm run test:run        (vitest run, tous verts — coller la sortie complète)
npm run build           (build prod réussi)
```

- **Jamais chaînées avec `&&`.** Trois blocs distincts, auditables séparément.
- **Jamais via `npx`.** Les scripts npm embarquent des options de configuration que le bypass
  masque. Seules commandes valides : `npm run type-check`, `npm run test:run`, `npm run build`.
- La CI (`.github/workflows/ci.yml`) rejoue ces trois étapes à chaque push.
- Si le sandbox tronque la sortie de build : `npm run build 2>&1 | tail -40`.

> **Tes preuves ne sont pas recevables comme verdict.** Le PO rejoue la porte sur sa machine ;
> seule sa sortie fait foi. Un « 156 passed » collé par toi ne vaut rien s'il est
> structurellement impossible (cas déjà constaté : test livré avec une variable non déclarée
> et une fonction jamais appelée, reporté « 81 passed »).

---

## 3. Règles de périmètre — NON NÉGOCIABLES

**R-SCOPE-1 — Fichiers listés uniquement.**
Ne créer ou modifier QUE les fichiers explicitement listés dans l'US. Toucher un fichier non
listé → **STOP**, le signaler au Tech Lead avant de continuer.
**Au démarrage de toute session, lister les fichiers modifiés depuis le dernier merge connu
AVANT toute action.** Ne jamais « préparer le terrain » de ta propre initiative.
*(Incident le plus coûteux du projet : 5 fichiers modifiés sans US → 17/17 tests E2E cassés,
une session entière perdue à réparer.)*
Le PO vérifie par `git diff --name-only` — la liste que tu déclares doit correspondre au diff réel.

**R-SCOPE-2 — Pas d'amélioration hors scope.**
Ne pas « améliorer » au passage du code hors périmètre. Ne pas réécrire un fichier entier
quand une correction ciblée suffit.

**R-SCOPE-3 — Max 3 fichiers par US.**
Si l'US en demande plus, le Tech Lead l'aura annoncé **en gras dans le titre de l'US**
(dépassement assumé). Sinon, signaler avant de continuer.

**Fichiers jamais créés de ta propre initiative :** scripts de debug, fichiers de sortie
(`*_out.txt`, `test_pid.txt`), utilitaires racine (`diff.cjs`, `replace.js`, `size.cjs`,
`find_usages.cjs`), `.gitignore` modifié hors US, specs `debug-*.spec.ts`. Le débogage reste
local, jamais commité.

---

## 4. Règles de code

**R-CODE-1 — Zéro `any`.** Aucun `any` implicite ou explicite, aucun `as any`, aucun
`@ts-ignore`. Tout type vient du contrat fourni dans l'US.
`eslint-disable-next-line` **ne corrige pas** `TS6133` (variable inutilisée) : retirer la
variable ou la préfixer `_`.

**R-CODE-2 — Fixtures de test typées.** Jamais `as unknown as T` pour une fixture. Utiliser
le helper `makeAnime(overrides: Partial<AnimeEntry>)`.

**R-CODE-3 — Séparation des responsabilités.**
- Composant `.vue` : UI + réactivité locale uniquement. **Jamais** de `fetch`, de
  `localStorage`/IndexedDB, ni de logique métier lourde.
- Composable `useXxx.ts` : n'expose vers l'extérieur que des `readonly` / `computed`.
- Store Pinia : état global, **aucun I/O**. Les `watch()` remplacent les `dispatchEvent`.
- Utils : fonctions pures, **zéro import de Vue**.

**R-CODE-4 — Zéro DOM direct, zéro bus DOM.** Pas de
`document.getElementById/querySelector/appendChild`, pas de `document.dispatchEvent`/
`CustomEvent`. Utiliser `ref` / `v-if` / `v-for`, le store Pinia, ou `emit`.

*Exceptions documentées, les seules autorisées :*
- `document.createElement('a')` + `.click()` pour le download Blob (`useICS`) ;
- `<input file>.click()` pour l'import MyAnimeList ;
- `DOMParser` pour parser du XML (pur) ;
- `getElementById('boot-loader')?.remove()` dans le `finally` du `onMounted` d'`App.vue`
  (le loader pré-Vue vit dans `index.html`, hors du scope Vue).

**R-CODE-5 — Gestion d'erreur.** Chaque fonction `async` a un `try/catch` explicite et expose
un état d'erreur réactif. Gérer le 429 Jikan (retry / backoff). Ne jamais avaler une erreur
en silence, sans log ni état.

**R-CODE-6 — Pas d'état sur `window`.** Aucun handler ni état stocké sur `window`. Utiliser
`onMounted` / `onUnmounted` + `@vueuse/core`.

**R-CODE-7 — Contrat d'event = le composant.** Un composant définit ses `defineEmits` ; les
consommateurs s'alignent sur ces noms. **Ne jamais renommer un emit pour matcher un listener
— corriger le listener.** Quand un composant à emits est réutilisé par N consommateurs,
vérifier les **N** alignements, pas un seul.
*Origine : les deux pires familles de bugs du projet. `RecCard` émettait `add`/`click`/
`not-interested` ; aucun de ses 4 consommateurs ne les écoutait → bouton Add mort, clic carte
mort, « pas intéressé » mort sur toutes les surfaces de reco, avec 0 erreur console.*

**R-CODE-8 — Aucun `<style scoped>` ajouté.** Tous les styles vont dans `style.css` global.
Ne pas ajouter de `<style scoped>` dans une US sans validation explicite du Tech Lead.
*Un `<style scoped>` définit une classe invisible depuis les autres composants : cause racine
réelle d'un bug de grille où `.recs-grid` était référencée par une page mais définie dans le
`<style scoped>` d'une autre.*

---

## 5. Règles de test

**R2 — Test obligatoire sur l'orchestration.**
Toute US qui touche le boot, un store, ou le câblage entre composables livre ou met à jour un
test (unitaire ou smoke) prouvant le **comportement runtime**. Les bugs d'orchestration ne se
voient pas à la compilation : 4 d'entre eux ont passé `vue-tsc` + tous les tests + le build
au vert.

**R4 — Test E2E obligatoire sur l'UI.**
Tout correctif issu d'un audit UX, et toute fonctionnalité touchant l'écran, livre un test
Playwright qui :
1. reproduit le **geste réel** de l'utilisateur (clic, saisie, navigation) ;
2. asserte le résultat **VISIBLE dans le DOM en viewport mobile** — jamais l'état interne
   d'un store, jamais le layout desktop ;
3. est **ROUGE** sur le bug actuel, **VERT** après le fix, **sans être modifié**.
Fournir les **deux** sorties brutes, rouge et verte.

**R4-bis — Gating ↔ E2E.** Tout `v-if` ou gating conditionnel ajouté sur un **élément
interactif** (bouton, lien, carte cliquable) déclenche, dans la **MÊME US**, un grep des specs
E2E qui ouvrent ou cliquent cet élément. Si une spec cliquait l'élément désormais gaté, elle
deviendra rouge **au sweep, pas au merge**. Vérifier et réaligner AVANT de livrer.

**R5 — Tester l'impact, pas l'univers.**
- *Pendant un epic :* chaque US ne livre qu'**UN** test ciblé sur ce qu'elle change.
- *Fin d'epic :* grand check complet — `npm run test:run`, `npm run build`, suite E2E entière.
- *Cumulatif :* les specs `tests/e2e/**` **enregistrées au registre §7** ne sont JAMAIS
  supprimées. Un test cumulé rouge au grand check est une régression à corriger, pas un test
  à retirer. *(Un fichier `debug-*.spec.ts` jamais enregistré n'a aucune valeur de preuve et
  doit être supprimé.)*

**R7 — L'auteur du test ≠ l'auteur du code. AUCUNE EXCEPTION.**
Tu ne rédiges **jamais** toi-même le test qui valide ton propre correctif, ni en rouge ni en
vert — même pour un test visuel jugé « simple » (position, centrage). Le Tech Lead fournit le
test de fidélité ; tu le fais passer **sans le modifier**, et tu ne mets jamais à jour un
snapshot en autonomie.
*Une violation constatée (test E2E auto-écrit pour valider un correctif de centrage de modale)
a été écartée intégralement, sans aucune valeur de preuve, malgré un code par ailleurs correct.*

---

## 6. Socle technique E2E

- **Playwright :** `webServer: build && preview`, `baseURL: http://localhost:4173`,
  `env: { VITE_E2E_AUTH_BYPASS: 'true' }`, `timeout: 120000` (sandbox lent).
- **Bypass auth :** `router/index.ts` `beforeEach` court-circuite si
  `import.meta.env.VITE_E2E_AUTH_BYPASS === 'true'`. Lecture **STATIQUE** obligatoire — la
  branche est éliminée du bundle prod (prouvable `grep -c` = 0). Jamais en runtime.
- **Isolation :** `vite.config.ts` `test.exclude` ignore `tests/e2e/**` (exclu de Vitest).
- **Sweep sandbox :** le sweep monolithique timeout à 60 s ; le parallèle provoque des
  `ERR_CONNECTION_REFUSED`. Lancer par **batchs ≤ 9 specs avec `--workers=1`**. Fournir chaque
  sortie de batch brute.
- **Boot :** attendre
  `await expect(page.locator('#boot-loader')).toBeHidden({ timeout: 15000 })`
  **AVANT tout clic**. Le loader est `position: fixed` et intercepte tous les pointer events.
  Sans cette attente : timeout 30 s garanti.
- **Réseau déterministe :** mocker via `page.route('**/pattern**', r => r.fulfill(...))`.
  Jamais l'API live. Patterns : `**/seasons/now**`, `**/seasons/upcoming**`, `**/anime?q=**`,
  `**/anime/**`. **Ne jamais mocker partiellement** — une vraie requête qui fuit rend le test
  flaky.

### Seed standardisé

🔴 **Clé localStorage réelle : `aanime_calendar`.** Toutes les clés du projet sont préfixées
`aanime_`. L'ancienne clé `'animeCalendar'` fonctionne encore *indirectement* via une
migration de compatibilité au boot, mais **tout nouveau seed doit utiliser `aanime_calendar`**
— ne jamais s'appuyer sur un chemin de migration legacy qui peut disparaître.

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
- **Seed mono-jour** (`day: 'monday'` seul) = carte invisible si la semaine courante tombe un
  autre jour → **toujours seeder les 7 jours**.
- **Auto-vault :** un seed `state:'calendar'` + `status:'Finished Airing'` s'auto-vault au boot
  (`usePersistence`) et **disparaît** de la vue semaine. Pour tester une action sur un anime
  terminé, seeder en `state:'watchlist'` (exclu de l'auto-vault) sur `/library/plan`.

### Assertion de position

Quand le **placement ou le centrage** est l'enjeu, asserter `boundingBox()` ou
`getComputedStyle().position` — **jamais `toBeVisible()` seul**. Un élément hors écran peut
être « visible » au sens DOM.
Deux corollaires appris à leurs dépens :
- Un test de centrage n'a de sens qu'accompagné d'une assertion
  `document.documentElement.scrollWidth <= window.innerWidth`. Une modale `position: fixed`
  correctement centrée sur le viewport paraît décalée si le **document** déborde.
- Un conteneur `display: grid` **sans enfant** a une hauteur de 0 px → Playwright le juge
  invisible même quand le CSS est correct. Pour tester une propriété de conteneur (nombre de
  colonnes), vérifier l'existence avec `toHaveCount` puis lire `getComputedStyle`.

### Sélecteurs réels

**Avant d'écrire un sélecteur, l'ouvrir dans le composant concerné.** Cette liste peut avoir
dérivé du code, elle n'est pas une garantie.

| Élément | Sélecteur |
|---|---|
| Carte reco / saison | `.card-cp-container` (titre `.card-cp-title`, cover `.card-cp-cover`, why `.rec-why-clickable`) |
| Recherche | `.search-input` / `.search-suggestion` / `.search-suggestion-title` |
| Modal | `.modal-backdrop` / `.modal-content` (fermeture Escape ou clic backdrop) |
| Toast | `.toast-notification` |
| Bouton add modal | libellé `+ Add` |
| Carte semaine | `.rowcard` (bouton ✓ `.rc-mark-done`, progression `.rc-progress` / `.rc-progress-fill`) |
| Grille de cartes | `.aa-card-grid` (classe partagée, 2/3/4 colonnes selon breakpoint) |
| Jour courant | `.day-hdr.today` |
| Navs | `.secondary-tab` / `.tab-item` / `.active` |
| Login | `.login-brand` |

---

## 7. Registre des batchs E2E

> Le découpage `test:e2e:batch1..5` est **FIGÉ EN DUR** dans `package.json`. Toute nouvelle
> spec DOIT être ajoutée à un batch **ET** listée ici, sinon elle ne tourne jamais au sweep,
> **silencieusement**. Garder chaque batch ≤ 9 fichiers.

🔴 **Chemin complet obligatoire.** Chaque entrée s'écrit `tests/e2e/<nom>.spec.ts`, jamais le
nom nu : Playwright interprète l'argument comme une **regex de sous-chaîne**, et l'entrée nue
`modal-position` captait aussi `logout-modal-position.spec.ts`, qui tournait donc hors
registre. **Slashes avant (`/`) uniquement** — sous Windows les `\` cassent le matching et
produisent « No tests found » sans erreur explicite.

**État de référence : 42 fichiers sur disque / 42 enregistrés, mapping 1:1 vérifié.**


- **batch1** (9) : `auto-vault-toast` · `boot-loader` · `calendar-subnav-layout` ·
  `discover-season-dedup` · `foryou-dedup` · `login-styled` · `modal-add-appears-on-week` ·
  `modal-add-feedback` · `modal-add-removes-from-discover`
- **batch2** (9) : `modal-content-centered-mobile` · `modal-open` · `modal-position` ·
  `modal-status-gating` · `month-layout` · `nav-active-state` · `onair-subnav` · `reccard-add` ·
  `reccard-click-dismiss`
- **batch3** (8) : `modal-next-episode` · `search-dedup` · `smoke` · `snap-to-today` ·
  `toast-labels` · `toast-visible-mobile` · `week-no-duplicate-period` · `week-progress-bar`
- **batch4** (9) : `logout-modal-position` · `nav-scroll-hide` · `onboarding-fullscreen` ·
  `onboarding-genres` · `onboarding-seed` · `onboarding-toast` · `onboarding-welcome` ·
  `search-enriched` · `search-hides-nav`
- **batch5** (7) : `search-quick-add` · `week-empty-day-cta` · `more-like-this-modal` ·
  `no-horizontal-overflow` · `grid-two-columns` · `onboarding-toast-destination` ·
  `day-guard-plan-to-watch`

**Dette connue :** `more-like-this-modal` n'a aucun `page.route()` et tape l'API Jikan live →
viole la règle du réseau déterministe. Rouge tant que l'endpoint `/anime/{id}/recommendations`
répond 504. **Ce rouge n'est pas une régression** — ne pas retirer la spec du batch pour faire
passer le sweep.
**Seed calendrier :** la clé de persistance est `aanime_calendar`. Une migration legacy au boot
recopie `animeCalendar` vers elle, donc d'anciennes specs sèment encore sur l'ancienne clé —
ce n'est pas un bug. **Toute nouvelle spec sème sur `aanime_calendar`.** Enveloppe attendue :
`{ timestamp: number, data: AnimeEntry[] }`.
---

## 8. Interdits

- ❌ Asserter un état de store → asserter le DOM visible.
- ❌ Livrer un E2E réparateur sans sa sortie **ROUGE** pré-fix. Une preuve rouge = un état figé
  unique, jamais rejouée dans un état différent et présentée comme la même.
- ❌ Deviner un sélecteur ou une clé localStorage. La clé est `aanime_calendar`, **jamais**
  `'animeCalendar'`.
- ❌ Supprimer ou désactiver une spec enregistrée pour faire passer le grand check.
- ❌ Écrire une nouvelle spec sans l'ajouter au batch **ET** au §7.
- ❌ Mock partiel (laisser une requête fuir) → flaky.
- ❌ `toBeVisible()` seul quand la position est l'enjeu.
- ❌ **Écrire soi-même le test qui valide son propre correctif — aucune exception.**
- ❌ **Injecter du code de production conscient du contexte de test** pour faire passer un test
  au vert sans corriger la vraie cause. Inadmissible, sans discussion.
- ❌ Réécrire un test rédigé par le Tech Lead. Il est figé : ROUGE → VERT sans y toucher.
- ❌ Mettre à jour des snapshots en autonomie (`vitest run -u`) sans annoncer le `.snap`
  comme fichier modifié.

---

## 9. Récidives — ce qui a déjà mal tourné

> Liste plafonnée aux comportements **répétés**. Un incident unique n'y entre pas.

| # | Récidive | Gravité | Parade |
|---|---|---|---|
| 1 | **Paraphrase de preuve de build** (« Build succeeded - the applet is compiled. ») au lieu de la sortie brute Vite | 🔴 | Toute paraphrase = review suspendue d'office, même quand le code est bon |
| 2 | **`npx` ou chaînage `&&`** dans les commandes de preuve | 🟠 | Trois `npm run` séparés, trois sorties brutes |
| 3 | **Changements hors scope poussés de sa propre initiative** — record du projet : 5 fichiers sans US → 17/17 E2E cassés | 🔴 | Lister les fichiers AVANT toute action ; le PO vérifie par `git diff --name-only` |
| 4 | **Fichiers parasites commités** — `clean.cjs`, `test_script.ts`, `debug-overflow*.spec.ts`, `*_out.txt` | 🟠 | Débogage local, jamais commité. `clean.cjs` est force-recréé par AI Studio → le PO le purge avant chaque gate |
| 5 | **Compte-rendu non conforme au diff réel** — « Edited 4 files » dont un absent du `git pull` | 🟠 | Le diff fait foi, pas la déclaration |
| 6 | **Test auto-écrit pour valider son propre correctif** | 🔴 | R7, aucune exception |
| 7 | **Compteur « X passed » structurellement impossible** — test livré avec variable non déclarée et fonction jamais appelée | 🔴 | Vérifier la cohérence interne du test avant d'annoncer un compteur |
| 8 | **Clé localStorage hors convention dans un test** (non préfixée `aanime_`) | 🟠 | Toute clé vient du registre fourni dans l'US |

---

## 10. Trois faits gravés — ne jamais réintroduire

- **`setAllData` n'existe pas.** Seulement `setData` + `clearAll`.
- **`syncStatus` n'existe pas** dans `AnimeEntry`.
- **`reconcileWithDatabase` n'existe plus** — la réconciliation se fait dans `loadFromDatabase`.

Et : **toutes les clés localStorage sont préfixées `aanime_`**, la persistance principale
étant `aanime_calendar`.
