# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — sprint en cours, métriques, Kanban, faits externes, trous.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles
> (`PILOTAGE.md`) · le contenu des constats (`AUDIT.md`).
>
> 🔴 **Régénéré intégralement à la CLÔTURE DE SPRINT** (`DEC-190`), plus à chaque session.
> **Il peut donc dater de plusieurs sessions.** L'état réel = **ce document + `HISTORIQUE §2
> Tampon`**. Lire les deux, toujours, dans cet ordre. Plafond : 250 lignes.

---

## 📍 Où on en est

| | |
|---|---|
| **Sprint** | **S44 EN COURS** — « L'arrivée vaut le produit » — **3/10 slots** |
| **Dernière session close** | `SE-075` — chantier documentaire hors slot |
| **Version** | `v0.37.0` — pas de bump (le sprint n'est pas clos) |
| **Commit `main` (A-Anime)** | `0c8b057` |
| **Prochaine session** | `SE-076` — suite de S44, **ouverture sur le retour bêta réel non qualifié** |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **331 / 38 fichiers** | SE-074 — rejoué, vert |
| Build | 180 modules, `index.js` 372,59 kB (gzip 109,08) · `index.css` 65,36 kB (gzip 12,41) | SE-074 |
| Sweep E2E | ✅ **58/58** sur 5 batches | SE-074 — rejoué complet |
| Specs E2E | **46 sur disque / 46 enregistrées**, mapping 1:1 | SE-074 |
| Tests E2E attendus au prochain sweep | **59** (58 + 1) — `header-mobile-rows` porte 5 tests, jamais passée en sweep complet | SE-074 |
| Specs sur helper unique | 18 | SE-074 |
| Specs à mock mort | 0 + 1 catch-all (`week-empty-day-cta`) | SE-072 |
| ESLint | ❌ **jamais exécuté — aucun script `lint` n'existe** (`AUD-08`) | SE-075 |
| **North Star** — TTFA · Adds/semaine · Jours-retour S1 | ❌ **aucune valeur, 4 sprints après leur définition** | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build`, lui-même embarqué dans le `webServer`
> Playwright : **un batch E2E qui démarre vaut type-check + build verts.**
>
> ⚠️ **Aucune sortie de Gemini n'a valeur de preuve** (`AUD-43`, `DEC-169`). Seule la machine du PO mesure.
>
> 🔻 Le PO commite et pousse **avant** chaque test local. Un `git status` propre après un patch
> signifie **commité**, jamais perdu.

---

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **14 documents · ~2 250 lignes** (16 et 3 291 avant SE-075) |
| Documents au-dessus du plafond H6 (250 l.) | **1** — `TYPES_CONTRACT.md` (359), **exception nommée** |
| Patchs documentaires en dette | **0** |
| Collages PO par session | **1** — `HISTORIQUE.md`, en fin de fichier (`DEC-190`) |
| Collages PO par clôture de sprint | **5** — `STATE`, `ROADMAP` (remplacés) · `DECISIONS`, `AUDIT`, `ANTIPATTERNS` (append) |
| Dernier `AUD-xx` | **`AUD-59`** — intégré |
| Dernier `DEC-xxx` | **`DEC-191`** — intégré |

---

## 📋 Kanban — S44 · 3/10 slots

### ✅ Done

| US | Impact utilisateur livré |
|---|---|
| `US-MORELIKETHIS-FIX` 🔴 (`AUD-16`) | Cliquer une saison 2 ou une relation ouvre une vraie fiche — jaquette, note, studio, genres |
| `US-MLT-REAL` 🟠 (`AUD-58`) | « MORE LIKE THIS » propose des titres non possédés au lieu de la bibliothèque de l'utilisateur |
| `US-HEADER-MOBILE-B` 🟠 | Sur téléphone, l'en-tête passe de **181 à 128 px** — trois rangées deviennent deux. Les 5 boutons restent visibles, au même endroit, à un tap |

> Gemini : streak **27** US mergées au premier coup. 2 micro-patchs `DEC-128` en SE-074.

### 🔄 In Progress

Aucune.

### 📝 To Do — 4 slots planifiés restants

| # | US | Risque | Impact bêta |
|---|---|---|---|
| 4 | **Retour bêta réel** — annoncé P0 par le PO, **non encore qualifié** | ? | 🔴 |
| 5 | `AUD-59` — l'onboarding confirme un succès inexistant | 🟠 | 🔴 |
| 6 | `US-STALE-SIGNAL` (`AUD-05`) — ⚠️ DEC d'arbitrage requis avant rédaction | 🟠 | 🟠 |
| 7 | Audit pixel `AUD-56` — hors slot Gemini (PO + Claude) | 🟢 | 🟠 |

**Flex (3) :** 2 slots retours bêta · 1 libre.

**Gelé ou reporté hors S44 :** `J10e-a/b/c` · chrome mobile option A · `AUD-54` · `US-ESLINT-CI-1`
· `US-DEMOCK-3` · renommage `modal-status-gating` · lot polish → `ROADMAP §3`.

### 🗂️ Backlog

Cap S45 → S50 et parking : `ROADMAP.md`. Constats ouverts : `AUDIT.md §1`.

---

## 🌐 Faits externes

| Fait | État | Dernière mesure |
|---|---|---|
| **Bêta testeurs** | ✅ **OUVERTE** — note envoyée, premier retour reçu, **non encore qualifié** | SE-074 |
| AniList `graphql.anilist.co` | ✅ Opérationnel — 200 sur la requête exacte du code | SE-073 — 🔻 **non remesuré depuis**, à refaire à l'ouverture de session |
| Multi-appareil (`AUD-42`) | ✅ Vérifié — second appareil, Clear site data → login → bibliothèque revenue | SE-074 |
| Service worker de production | ❌ Passe-plat, zéro cache — c'est celui d'AI Studio | SE-071 |
| PWA hors ligne | ❌ Inexistante — cap S49 | SE-071 |
| Déploiement | ✅ AI Studio redéploie automatiquement depuis `main` — ce qui est servi est ce qui est déployé | SE-068 |

---

## 🕳️ Trous ouverts

1. 🔴 **`AUD-59` — l'onboarding confirme un succès inexistant.** « Saved on this device » s'affiche exactement quand l'appareil n'a rien sauvegardé. Détail → `AUDIT §1`.
2. 🔴 **Aucun ordonnanceur de réessai Firestore.** `saveError` passe à `true` et personne ne le relit. « We'll sync it later » ne promet rien que le code garantisse.
3. 🔴 **Le retour bêta du PO n'est pas qualifié.** Il passe devant `AUD-59` : *un utilisateur qui a parlé bat un bug lu dans le code.*
4. **`AUD-56`** — audit pixel complet jamais fait. Méthode arrêtée : par parcours, 375 × 812, un thème à la fois, parcours ouvrir → ajouter → semaine → modale → déconnexion.
5. **Aucun token de couleur hors `--accent`** — les 5 teintes de l'en-tête sont en dur (`AppHeader.vue`, `nth-child(1..5)` × 2 thèmes).
6. **Le seul texte français de l'app** est un toast d'erreur : « Erreur de sauvegarde — données locales conservées » (`usePersistence.ts:132`). Toute l'UI est en anglais.
7. **`AGENTS.md §6`** — la règle du seed mono-jour n'a jamais été vérifiée (hérité SE-071).
8. **Famille C d'`AUD-52`** — `week-empty-day-cta` route `**/*`, fonctionne par accident.
9. **Titres mensongers dans `modal-status-gating`** — deux tests nommés « ROUGE » / « VERT » testent la même chose.
10. **`modal-open.spec.ts`** seed la clé `animeCalendar`, rattrapée par la migration `usePersistence.ts:139`. Le jour où la migration tombe, la spec devient rouge sans qu'aucun code de modale ait bougé.
11. **`AUD-35`** — `AGENTS.md:72` interdit un helper (`makeAnime`) présent dans 5 fichiers de specs. L'agent opère sous une règle contredite par le dépôt.
12. **`AUD-54`** — vue Semaine qui reconstruit ses cartes en synchro (~7 s). Cause non établie, composant non lu.
