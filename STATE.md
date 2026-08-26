# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-068** — ouverture S42 |
| Sprint | **S42 EN COURS** — « On peut nous faire confiance » — 4 US livrées / 10 slots |
| Version | **v0.35.0** — pas de bump, sprint non clos |
| Commit main | 🔻 **3 hash non relevés** — `git log --oneline -6` en ouverture SE-069 |
| Prochaine session | **SE-069** — poursuite S42 |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **315** / 35 fichiers | SE-068 |
| Build | **179 modules**, `index` 369,16 kB (gzip 108,36) | SE-068 |
| Sweep E2E | **52/52** | **SE-068 — rejoué** |
| Specs E2E | 42 enregistrées en 5 batchs | SE-063 |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build` : un build vert vaut type-check vert.
> ⚠️ **Aucune sortie de Gemini n'a valeur de preuve** (`AUD-43`). Seule la machine du PO mesure.

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | **0** |

---

## 📋 Kanban — S42 · SE-068

### ✅ Done — Sprint S42 · 4/10 slots

| US | Impact utilisateur livré |
|---|---|
| `US-SWEEP-S41` 🟢 | Aucun direct — sweep E2E rejoué, 52/52, zéro régression S41 |
| `US-ADD-EXTRACT` 🟠 | Aucun — refactor pur, la règle d'ajout vit dans `useAddAnime.ts` |
| `US-ADD-TOAST-TRUTH` 🟠 | Le message de confirmation annonce l'onglet réel, plus l'onglet demandé |
| `US-ONBOARD-EMPTY` 🟠 | Saison vide → état vide + « Try again », au lieu d'une zone blanche |
| `US-ONBOARD-TOAST` 🟠 | Un échec de sauvegarde ne bloque plus la fin de l'inscription |

**Gemini : 4 US 🟠, 4 merges au premier coup. Streak 19.**

### 🔄 In Progress
Aucune.

### 📝 To Do — S42 (6 slots restants)

| US | Risque | Note |
|---|---|---|
| `US-STALE-SIGNAL` | 🟠 M | **Bloquée** : exige l'arbitrage `DEC-151` (source unique du signal `stale`) |
| `US-ADD-EXTRACT` → `US-CARD-CONVERGE-A` | 🟠 S | This Season sur `RecCard`, 1 tap. Absorbe `US-SEASON-1TAP` |
| `US-CARD-CONVERGE-B` | 🟠 S | Coming Soon, même geste |
| `US-MORELIKETHIS-FIX` | 🔴 S | « More like this » revit |
| `US-ONBOARD-PERSIST-B` | 🔴 M | **Priorité relevée** — `AUD-42` observé en vrai |
| `US-PERF-BASELINE` → `US-PERF-GATE` | 🟢 M / 🟠 S | Ordre imposé |

**Flex :** `US-TOUCH-A` · `US-HEADER-ICONS` · `US-ONBOARD-FALLBACK`
*(slot `US-TOUCH-B` consommé par `US-ADD-TOAST-TRUTH`)*

### 🗂️ Backlog
Cap S43 → S50 : **`ROADMAP.md`**.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| Déploiement prod | ✅ **Automatique depuis `main`** — chunk changé seul, sans action manuelle | **SE-068** | onglet Réseau, 2 mesures |
| Service worker prod | ✅ **Tiers (AI Studio), passe-plat, zéro cache** — aucun risque de version périmée | **SE-068** | lecture du fichier |
| AniList — quota | ✅ Aucun 429 observé | SE-067 | constat PO en prod |
| Thème sombre This Season | ✅ **Visible** | **SE-068** | constat PO |
| Multi-appareil | 🔴 **Onboarding rejoué après purge locale** — `AUD-42` confirmé | **SE-068** | constat PO |
| Firestore — écriture | ✅ AUTORISÉE | SE-067 | console Firebase |
| Firestore — plafond | **500 entrées** (~1,1 ko/entrée, marge 45 %) | SE-067 | console → Règles |
| Firestore — base cible | `ai-studio-58fc34cb-…`, europe-west2 | SE-064 | console Firebase |

---

## 🕳️ Trous ouverts

1. **🔴 `AUD-46` — dropdown de recherche recouvert par les cartes.** Trois hypothèses réfutées
   (dont `z-index: 9000` sur `.app-header`, testé, sans effet). **Diagnostic requis avant US :
   `type src\App.vue`.** ⛔ Ne pas proposer une 4ᵉ valeur de `z-index`.
2. **🔴 Patch mort en `main`** — `AppHeader.vue:89` porte `z-index: 9000`, sans effet. **À révoquer
   en commit séparé.**
3. **🟠 3 hash de commit non relevés** (`TOAST-TRUTH`, `ONBOARD-EMPTY`, `ONBOARD-TOAST`).
4. **🟠 `DEC-151` non arbitré** — deux signaux `stale` concurrents et morts. Bloque `US-STALE-SIGNAL`.
5. **🔴 Multi-appareil dégradé** (`AUD-42`) — **observé, plus supposé.** Limite à annoncer aux testeurs.
6. **🟠 `AUD-37` délégué à la bêta** — import MAL jamais vérifié sur un vrai fichier.
7. **🟠 `AUD-44`** — `isSyncing` sans `try/finally`. Latent. → S43, seul.
8. **🟠 Dette de vérité** — « We'll sync it later » promet une resync non garantie par le code.
9. **🟠 Zéro retour terrain** — 15 US depuis S41, aucune observée par un utilisateur.

---

## 🎯 Sprint Outcome Gate — S42 (à répondre à la clôture)

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait pas
> avant ce sprint ? »*

**Budget dette :** 1 US de dette pure (`US-ADD-EXTRACT`) sur 4 livrées. Sous plafond.
