# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (`DEC-146`), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| **Session** | `SE-074` — clôturée |
| **Sprint** | **S44 EN COURS** — « L'arrivée vaut le produit » — **3/10 slots** |
| **Version** | `v0.37.0` — pas de bump (le sprint n'est pas clos) |
| **Commit `main`** | `0c8b057` |
| **Prochaine session** | `SE-075` — suite de S44, **ouverture sur retour bêta réel (P0 annoncé par le PO)** |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **331 / 38 fichiers** | SE-074 — rejoué, vert |
| Build | 180 modules, `index.js` 372,59 kB (gzip 109,08) · `index.css` **65,36 kB** (gzip 12,41) | SE-074 — relevé |
| Sweep E2E | ✅ **58/58** sur 5 batches | SE-074 — rejoué complet |
| Specs E2E | **46 sur disque / 46 enregistrées**, mapping 1:1 | SE-074 — `header-mobile-rows` créée et enregistrée en batch5 |
| Tests E2E attendus au prochain sweep | **59** (58 + 1) | SE-074 — `header-mobile-rows` porte 5 tests, jamais passée en sweep complet |
| Specs sur helper unique | 18 | SE-074 |
| Specs à mock mort restant | 0 + 1 catch-all (`week-empty-day-cta`) | SE-072 |
| ESLint | ❌ jamais exécuté | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build`, lui-même embarqué dans le `webServer`
> Playwright : **un batch E2E qui démarre vaut type-check + build verts.**
>
> ⚠️ **Aucune sortie de Gemini n'a valeur de preuve (`AUD-43`, `DEC-169`).** Seule la machine du PO mesure.
>
> 🔻 Le PO commite et pousse **avant** chaque test local. Un `git status` propre après un patch
> signifie **commité**, jamais perdu.

---

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | 15 documents (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | 0 |
| **Patchs documentaires en dette** | 🔻 **4 — reportés à SE-075 sur arbitrage PO.** Textes verbatim dans le handoff `SE-074 → SE-075` |
| Dernier `AUD-xx` | **`AUD-59`** — numéro **consommé**, texte non encore intégré à `AUDIT.md` |
| Dernier `DEC-xxx` | **`DEC-189`** — numéro **consommé**, texte non encore intégré à `DECISIONS.md` |

> 🔴 **Conséquence assumée :** `DECISIONS.md` et `AUDIT.md` sont **incomplets** jusqu'à
> l'application des patchs. Ne jamais réattribuer 189 ni 59.

---

## 📋 Kanban — S44 EN COURS · SE-074

### ✅ Done — 3/10 slots

| US | Impact utilisateur livré |
|---|---|
| `US-MORELIKETHIS-FIX` 🔴 (`AUD-16`) | Cliquer une saison 2 ou une relation ouvre une vraie fiche — jaquette, note, studio, genres |
| `US-MLT-REAL` 🟠 (`AUD-58`) | « MORE LIKE THIS » propose des titres non possédés au lieu de la bibliothèque de l'utilisateur |
| `US-HEADER-MOBILE-B` 🟠 | **Sur téléphone, l'en-tête passe de 181 à 128 px** — trois rangées deviennent deux. Les 5 boutons restent visibles, au même endroit, à un tap |

> Gemini : 1 US en SE-074. Streak → **27**. 2 micro-patchs `DEC-128` (seuil T3, `package.json`).

### 🔄 In Progress

Aucune.

### 📝 To Do — S44, 4 slots planifiés restants

| # | US | Risque | Impact bêta |
|---|---|---|---|
| 4 | **Retour bêta réel** — annoncé P0 par le PO, non encore qualifié | ? | 🔴 |
| 5 | `J10e-a` 🔻 **gelée** — spécification absente de la Knowledge + fréquence jamais mesurée | 🟠 | ? |
| 6 | Chrome mobile option A (menu ⋯) 🔻 **reportée** — casse `header-icons` et `logout-modal-position` | 🟠 | 🟠 |
| 7 | `US-STALE-SIGNAL` (`AUD-05`) — ⚠️ DEC d'arbitrage requis avant rédaction | 🟠 | 🟠 |

**Flex (3) :** 2 slots retours bêta · `AUD-59` (dette de vérité onboarding).

**Hors slot, côté PO :** audit pixel `AUD-56` (PO + Claude, en parallèle, ne consomme aucun slot Gemini).

**Sorti de S44 :** `AUD-54` · `US-ESLINT-CI-1` · `US-DEMOCK-3` · renommage `modal-status-gating` · lot polish ↦ S45.

### 🗂️ Backlog

Cap S44 → S50 : `ROADMAP.md`.

---

## 🌐 Faits externes

| Fait | État | Dernière mesure |
|---|---|---|
| AniList `graphql.anilist.co` | ✅ Opérationnel — 200 sur la requête exacte du code | SE-073 — 🔻 **non remesuré en SE-074**, à refaire à l'ouverture de SE-075 |
| Multi-appareil (`AUD-42`) | ✅ **Vérifié** — second appareil, Clear site data → login → bibliothèque revenue | **SE-074** |
| Bêta testeurs | ✅ **Ouverte** — note envoyée, premier retour reçu, non encore qualifié | SE-074 |
| Service worker de production | ❌ Passe-plat, zéro cache | SE-071 |
| PWA hors ligne | ❌ Inexistante — cap S49 | SE-071 |

---

## 🕳️ Trous ouverts

1. 🔴 **`AUD-59` — l'onboarding confirme un succès inexistant.** `saveToDatabase` n'échoue jamais sur Firestore (son `try/catch` avale l'erreur) : le seul `catch` atteignable dans `OnboardingPage.vue:123` est celui d'un échec **localStorage**. Le message « Saved on this device » s'affiche donc exactement quand l'appareil n'a **rien** sauvegardé. Famille `AUD-02`. Détail complet dans le handoff.
2. 🔴 **Aucun ordonnanceur de réessai Firestore.** `saveError` passe à `true` et personne ne le relit. La prochaine tentative dépend d'une action ultérieure de l'utilisateur. « We'll sync it later » ne promet rien que le code garantisse.
3. 🔻 **4 patchs documentaires en dette** (voir handoff). `DECISIONS.md` et `AUDIT.md` incomplets.
4. **`AUD-56`** — audit pixel complet (toutes pages, modales, clair et sombre). Aucune mesure faite. Méthode arrêtée : par parcours, 375 × 812, un thème à la fois.
5. **`J10e` non spécifiée.** `DEC-145` annonce 3 slices ; leur contenu n'existe dans aucun document. 5 points de rejet des orphelins recensés : `normalizeAniList.ts:42` · `useAniListApi.ts:232, 265, 331, 447`. L'identité entière d'une entrée est bâtie sur `idMal` (`mal_id` **et** `id`).
6. **`AUD-54`** — vue Semaine qui reconstruit ses cartes en synchro (~7 s). Sorti de S44, cause non établie, composant non lu.
7. **`AGENTS.md §6`** — règle du seed mono-jour non vérifiée (hérité SE-071).
8. **Famille C d'`AUD-52`** — `week-empty-day-cta` route `**/*`, fonctionne par accident.
9. **Titres mensongers dans `modal-status-gating`** — deux tests nommés « ROUGE » / « VERT » testent la même chose.
10. **Aucun token de couleur hors `--accent`** — les 5 teintes de l'en-tête sont en dur (`AppHeader.vue`, `nth-child(1..5)` × 2 thèmes).
11. **`modal-open.spec.ts`** seed la clé `animeCalendar`, rattrapée par la migration `usePersistence.ts:139`. Le jour où la migration tombe, la spec devient rouge sans qu'aucun code de modale ait bougé.
12. **Le seul texte français de l'app** est un toast d'erreur : « Erreur de sauvegarde — données locales conservées » (`usePersistence.ts:132`). Toute l'UI est en anglais.
