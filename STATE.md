# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-071** — poursuite S43 |
| Sprint | **S43 EN COURS** — « Rien ne casse en douce » — 5 slots pleins + slot 6 aux ⅔ |
| Version | **v0.36.0** — inchangée, le sprint n'est pas clos |
| Commit main | `4ae0624` |
| Prochaine session | **SE-072** — poursuite S43 |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **331** / 38 fichiers | **SE-071 — rejoué, vert** |
| Build | **179 modules**, `index` 369,28 kB (gzip 108,39) | SE-069 — 🔻 chiffres non relevés depuis, mais build **vert** en SE-071 |
| Sweep E2E | ✅ **52 / 52** | **SE-071** — 🔻 les 5 batchs verts au cours de la session, **pas en un sweep continu** |
| Specs E2E | 42 sur disque / 42 enregistrées, mapping 1:1 | SE-071 — inchangé, aucune spec créée ni supprimée |
| Specs sur helper unique | **10** | **SE-071** |
| Specs à mock mort restant | **5** (famille B, Jikan) + 1 catch-all (famille C) | **SE-071** |
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
| Patchs documentaires en dette | ✅ **0** — les 6 hérités de SE-070 appliqués en SE-071 |
| Dernier `AUD-xx` | **AUD-54** |
| Dernier `DEC-xxx` | **DEC-186** |

---

## 📋 Kanban — S43 · SE-071

### ✅ Done — 5 slots pleins + slot 6 aux ⅔

| US | Impact utilisateur livré |
|---|---|
| `US-SWEEP-S42` 🟢 | Aucun — sweep rejoué |
| `US-SEASON-TOKENS` 🟢 | Bouton « Retry » au violet de l'app ; cartes homogènes ; **dernière rangée atteignable** |
| `US-DEMOCK-1a` 🟢 | Aucun — pilote du helper unique |
| `US-DEMOCK-HELPER` 🟠 | Aucun — le helper sait produire un studio et une date |
| `US-DEMOCK-1d` 🟠 | Aucun — la ligne `2023 · ★ 8.9 · MAPPA` est enfin protégée par un test hors réseau |
| `US-DEMOCK-1b/c` 🟢 | Aucun — famille A soldée |
| `US-DEMOCK-2a` 🟠 | Aucun — 3 specs Jikan mortes rebranchées |
| `US-DEMOCK-2b` 🟠 | Aucun — la tenue en 387 px est de nouveau mesurée pour de vrai |

**Gemini : 0 US en SE-070 et SE-071.** Streak inchangée à 23. 12 micro-patchs `DEC-128` sur deux
sessions. ⚠️ **Signal :** deux sessions pleines sans agent d'implémentation. Conforme à `DEC-183`
sur un chantier 100 % test, mais le sprint n'a encore produit **aucune ligne de code applicatif**.

### 🔄 In Progress
Aucune.

### 📝 To Do — S43
`US-DEMOCK-2c` (5 specs Jikan) · `US-HEADER-ICONS` · flex : `US-ESLINT-CI-1`, `AUD-54`,
1 slot bêta. Composition → **`ROADMAP.md §2`**.

### 🗂️ Backlog
Cap S43 → S50 : **`ROADMAP.md`**.

---

## 🌐 Faits externes

| Fait | État | Dernière mesure |
|---|---|---|
| AniList `graphql.anilist.co` | ✅ Opérationnel, ~3 s de réponse en 3G simulée | **SE-071** |
| Service worker de production | ❌ **Passe-plat, zéro cache** — reconfirmé par F5 hors ligne (`ERR_FAILED`, 0 kB servi) | **SE-071**, méthode indépendante |
| PWA hors ligne | ❌ **Inexistante** — l'app ne s'ouvre pas sans réseau. Pas une régression : jamais implémentée. Cap S49 | **SE-071** |
| Multi-appareil (`AUD-42`) | 🔻 **Non revérifié** — bande de rattrapage livrée, comportement réel jamais observé | SE-06x |

---

## 🕳️ Trous ouverts

1. **`AUD-54`** — vue Semaine qui reconstruit ses cartes en synchro. 🟠, cause non établie, composant non lu.
2. **Chiffres de build** — 179 modules / 369,28 kB datent de SE-069. Le build est vert, sa taille est inconnue.
3. **Sweep non continu** — 52/52 obtenus batch par batch au fil de SE-071, jamais en une passe.
4. **`AGENTS.md §6` — règle du seed mono-jour non vérifiée.** `modal-next-episode` seede un seul jour et sa carte **était visible**. La règle gravée dit l'inverse. 🔻 Ni confirmée ni infirmée : un seul jour de test, sur une seule vue.
5. **Bêta non ouverte** — décision PO. 25+ US livrées depuis S41, zéro observation utilisateur.
