# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-069** — clôture S42 |
| Sprint | **S42 CLOS** — « On peut nous faire confiance » — 10/10 slots |
| Version | **v0.36.0** — bump de clôture de sprint |
| Commit main | `d55952e` + 2 commits non relevés (voir Trou 1) |
| Prochaine session | **SE-070** — ouverture S43 |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **331** / 38 fichiers | **SE-069** |
| Build | **179 modules**, `index` 369,28 kB (gzip 108,39) | **SE-069** |
| Sweep E2E | **52/52** | SE-068 — **non rejoué en SE-069** |
| Specs E2E | 42 enregistrées en 5 batchs | SE-063 |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build` : un build vert vaut type-check vert.
> ⚠️ **Aucune sortie de Gemini n'a valeur de preuve** (`AUD-43`). Seule la machine du PO mesure.
> 🔻 La mesure de build ci-dessus **précède** le micro-patch `RecCard` (style seul, non rebuildé).

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | **0** |

---

## 📋 Kanban — S42 · SE-069

### ✅ Done — Sprint S42 · 10/10 slots

| US | Impact utilisateur livré |
|---|---|
| `US-SWEEP-S41` 🟢 | Aucun direct — sweep E2E rejoué, 52/52 |
| `US-ADD-EXTRACT` 🟠 | Aucun — refactor pur, règle d'ajout dans `useAddAnime.ts` |
| `US-ADD-TOAST-TRUTH` 🟠 | Le toast de la modale annonce l'onglet réel |
| `US-ONBOARD-EMPTY` 🟠 | Saison vide → état vide + « Try again » |
| `US-ONBOARD-TOAST` 🟠 | Un échec de sauvegarde ne bloque plus l'inscription |
| `AUD-46` 🟢 micro-patch | Le dropdown de recherche passe devant les cartes de contenu |
| `US-SEARCH-USE-ADDANIME` 🟠 | Ajout depuis la recherche : bon onglet, message vrai |
| `US-ONBOARD-PERSIST-B` 🔴 | Sur 2ᵉ appareil, une bande annonce le compte retrouvé |
| `US-SEASON-FRESH` 🟠 | This Season se met à jour sans rechargement |
| `US-CARD-CONVERGE-A` 🟠 | This Season : 1 tap pour ajouter, sur `RecCard` |

**Gemini : 6 US 🟠/🔴 en SE-069, 6 merges au premier coup. Streak 23.**
**2 micro-patchs hors Gemini (DEC-128) : `AppLayout.vue`, `RecCard.vue`.**

### 🔄 In Progress
Aucune.

### 📝 To Do — S43
Composition → **`ROADMAP.md §2`**.

### 🗂️ Backlog
Cap S43 → S50 : **`ROADMAP.md`**.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| Déploiement prod | ✅ **Automatique depuis `main`** | SE-068 | onglet Réseau, 2 mesures |
| Service worker prod | ✅ **Tiers (AI Studio), passe-plat, zéro cache** | SE-068 | lecture du fichier |
| AniList — quota | ✅ Aucun 429 observé | SE-067 | constat PO en prod |
| Dropdown de recherche | ✅ **Au-dessus des cartes, tous écrans** | **SE-069** | constat PO |
| Thème sombre This Season | ✅ Visible | SE-068 | constat PO |
| Multi-appareil | 🟠 **Bande de rattrapage livrée** — comportement non revérifié à l'œil | **SE-069** | à mesurer SE-070 |
| Firestore — écriture | ✅ AUTORISÉE | SE-067 | console Firebase |
| Firestore — plafond | **500 entrées** (~1,1 ko/entrée, marge 45 %) | SE-067 | console → Règles |
| Firestore — base cible | `ai-studio-58fc34cb-…`, europe-west2 | SE-064 | console Firebase |

---

## 🕳️ Trous ouverts

1. **🟠 2 hash non relevés** — `US-CARD-CONVERGE-A` et le micro-patch `RecCard`.
   `git log --oneline -8` en ouverture SE-070.
2. **🔴 Micro-patch `RecCard` non commité** — `git status` non vide en fin de SE-069.
   Bloque toute livraison (`AP-PROCESS-3`). **Premier geste SE-070.**
3. **🟠 Sweep E2E non rejoué** depuis SE-068, alors que 6 US ont touché
   recherche, onboarding et This Season. **Rejeu obligatoire en ouverture S43.**
4. **🟠 `AUD-49`** — collision de numérotation `DEC-158`. Renumérotation en `DEC-176` à appliquer.
5. **🟠 `AUD-50`** — `AnimeCard.vue` a 3 consommateurs restants (Coming Soon, Completed, Plan to Watch).
6. **🟠 `AUD-51`** — pastille « Add » à 36 px, sous la cible tactile de 44 px.
7. **🟠 `AUD-37`** délégué à la bêta — import MAL jamais vérifié sur un vrai fichier.
8. **🟠 `AUD-44`** — `isSyncing` sans `try/finally`. Latent. → S43, seul.
9. **🟠 Dette de vérité** — « We'll sync it later » promet une resync non garantie par le code.
10. **🟠 Zéro retour terrain** — 25 US depuis S41, aucune observée par un utilisateur.

---

## 🎯 Sprint Outcome Gate — S42 · RÉPONDUE

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait pas
> avant ce sprint ? »*

**Il peut faire confiance à ce que l'écran lui dit.** Le message d'ajout annonce l'onglet réel,
la recherche est lisible et range au bon endroit, un anime déjà suivi disparaît de la saison,
un second appareil annonce le compte retrouvé au lieu de faire semblant de ne pas le connaître.

**Type : gain de fiabilité visible** (réponse 2, `PILOTAGE.md §5`) — ✅ acceptable.

**Budget dette :** 1 US de dette pure (`US-ADD-EXTRACT`) sur 10 livrées. Sous plafond.
