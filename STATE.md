# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-070** — ouverture S43 |
| Sprint | **S43 EN COURS** — « Rien ne casse en douce » — 3/10 slots consommés |
| Version | **v0.36.0** — inchangée, le sprint n'est pas clos |
| Commit main | `a4b1d7f` + 3 commits non relevés (voir Trou 1) |
| Prochaine session | **SE-071** — poursuite S43 |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **331** / 38 fichiers | SE-069 — **non rejoué en SE-070** |
| Build | **179 modules**, `index` 369,28 kB (gzip 108,39) | SE-069 — 3 micro-patchs CSS/test depuis |
| Sweep E2E | 🔴 **51 / 52** | **SE-070** — `modal-next-episode` rouge |
| Specs E2E | 42 sur disque / 42 enregistrées, mapping 1:1 | SE-070 — inchangé |
| Specs sur helper unique | **1 / 5** (famille A, `AUD-52`) | **SE-070** |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build`, lui-même embarqué dans le `webServer`
> Playwright : un batch E2E qui démarre vaut type-check + build verts.
> ⚠️ **Aucune sortie de Gemini n'a valeur de preuve** (`AUD-43`). Seule la machine du PO mesure.
> 🔻 **Le PO commite et pousse avant chaque test local.** Un `git status` propre après un patch
> signifie **commité**, jamais **perdu**.

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | **0** |
| Patchs documentaires en dette | 🔴 **6** — reportés en clôture SE-071 (arbitrage PO, handoff §7) |

---

## 📋 Kanban — S43 · SE-070

### ✅ Done — Sprint S43 · 3/10 slots

| US | Impact utilisateur livré |
|---|---|
| `US-SWEEP-S42` 🟢 | Aucun direct — sweep rejoué, 52/52 après réparation d'une spec périmée |
| `US-SEASON-TOKENS` 🟢 | Bouton « Retry » de This Season au violet de l'app ; cartes à la même taille qu'ailleurs ; **dernière rangée atteignable** |
| `US-DEMOCK-1a` 🟢 | Aucun — `search-dedup` migrée sur le helper, pilote validé |

**Gemini : 0 US cette session.** 4 micro-patchs `DEC-128` (spec `search-quick-add`, tokens,
padding, migration `search-dedup`). Streak inchangée à 23.

### 🔄 In Progress
Aucune. **Bloquée par le Trou 2 :** aucune US ne démarre avant l'instruction du rouge.

### 📝 To Do — S43
Composition → **`ROADMAP.md §2`**. Prochaines : `US-DEMOCK-HELPER`, `US-DEMOCK-1b/c/d`,
`US-DEMOCK-2`, `US-HEADER-ICONS`.

### 🗂️ Backlog
Cap S43 → S50 : **`ROADMAP.md`**.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| Déploiement prod | ✅ **Automatique depuis `main`** | SE-068 | onglet Réseau, 2 mesures |
| Service worker prod | ✅ **Tiers (AI Studio), passe-plat, zéro cache** | SE-068 | lecture du fichier |
| AniList — quota | ✅ Aucun 429 observé | SE-067 | constat PO en prod |
| Dropdown de recherche | ✅ Au-dessus des cartes, tous écrans | SE-069 | constat PO |
| Thème sombre This Season | ✅ Visible | SE-068 | constat PO |
| **Padding This Season** | ✅ **Aligné sur Coming Up / Explore** | **SE-070** | `findstr padding` + patch. 🔻 Constat à l'œil non fourni |
| Multi-appareil | 🟠 **Bande de rattrapage livrée** — comportement non revérifié à l'œil | SE-069 | à mesurer, « Clear site data » |
| Firestore — écriture | ✅ AUTORISÉE | SE-067 | console Firebase |
| Firestore — plafond | **500 entrées** (~1,1 ko/entrée, marge 45 %) | SE-067 | console → Règles |
| Firestore — base cible | `ai-studio-58fc34cb-…`, europe-west2 | SE-064 | console Firebase |

---

## 🕳️ Trous ouverts

1. **🟠 3 hash non relevés** — micro-patchs tokens, padding, et `US-DEMOCK-1a`.
   `git log --oneline -8` en ouverture SE-071. **3ᵉ session consécutive avec ce trou.**
2. **🔴 `modal-next-episode.spec.ts` ROUGE** — `.rowcard` introuvable. Verte le matin même,
   aucun patch de la session ne touche son sujet. **Aucun mock réseau dans le fichier.**
   Hypothèse dominante non confirmée : réseau réel + circuit breaker.
   **Premier geste de SE-071, avant toute US.**
3. **🔴 `AUD-52`** — 17 specs mockent en direct, dont **11 sur des URL Jikan mortes** depuis S40
   et **1 sans aucun mock**. Le « 52/52 » couvre une trentaine de specs, pas 42 fichiers.
4. **🔴 6 patchs documentaires en dette** — `AUDIT`, `DECISIONS`, `DECISIONS_ARCHIVE`,
   `ANTIPATTERNS`, `PILOTAGE`, `HISTORIQUE`. Écrits verbatim au handoff §7, **à appliquer en
   clôture SE-071** (arbitrage PO, dérogation assumée à `DEC-146`).
5. **🟠 Métriques unitaires et build datées SE-069** — `npm run test:run` non rejoué cette session.
6. **🟠 `AUD-49`** — collision de numérotation `DEC-158`. Renumérotation en `DEC-176` à appliquer.
7. **🟠 `AUD-37`** délégué à la bêta — import MAL jamais vérifié sur un vrai fichier.
8. **🟠 `AUD-44`** — `isSyncing` sans `try/finally`. Latent. → S44, seul.
9. **🟠 Dette de vérité** — « We'll sync it later » promet une resync non garantie par le code.
   Aggravée : un anime en cours sans horaire AniList atterrit en *Plan to Watch*.
10. **🟠 Zéro retour terrain** — 28 US depuis S41, aucune observée par un utilisateur.
    **Bêta non ouverte** (décision PO SE-070) : pas derrière un filet troué.

---

## 🎯 Sprint Outcome Gate — S43 · NON RÉPONDUE

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait pas
> avant ce sprint ? »*

Sprint en cours. **Une seule US visible est planifiée** (`US-HEADER-ICONS`) ; les autres sont des
filets. `PILOTAGE.md §5` autorise un sprint « aucun gain visible » **en exception, pas en
routine** — S42 ayant répondu « gain de fiabilité visible », le garde-fou n'est pas franchi,
mais **S44 doit être un sprint produit**.

**Budget dette :** 3 US livrées, 3 de dette pure. ⚠️ **Plafond `PILOTAGE.md §5` (≤ 1 dette pour
1 gain visible) sera dépassé si `US-HEADER-ICONS` n'est pas livrée dans ce sprint.**
