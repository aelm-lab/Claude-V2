# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-072** — clôture de S43 |
| Sprint | **S43 CLOS** — « Rien ne casse en douce » — **10/10 slots** |
| Version | **v0.37.0** — bumpée à la clôture (code applicatif livré) |
| Commit main | 🔻 **à relever** — `db9c258` + le commit du correctif `logout-modal-position` |
| Prochaine session | **SE-073** — ouverture de **S44** |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **331** / 38 fichiers | **SE-072 — rejoué 3×, vert** |
| Build | **180 modules**, `index.js` 371,75 kB (gzip 108,84) · `index.css` **65,20 kB** (gzip 12,38) | **SE-072 — relevé** |
| Sweep E2E | ✅ **55 / 55** sur 5 batchs | **SE-072** — 🔻 batch4 vert **après correctif, sortie brute non collée** |
| Specs E2E | **43** sur disque / **43** enregistrées, mapping 1:1 | **SE-072** — `header-icons` créée et enregistrée |
| Specs sur helper unique | **15** | **SE-072** |
| Specs à mock mort restant | **0** (famille B soldée) + **1 catch-all** (famille C, `week-empty-day-cta`) | **SE-072** |
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
| Patchs documentaires en dette | 🔻 **le lot de clôture SE-072** — à appliquer en ouverture de SE-073 |
| Dernier `AUD-xx` | **AUD-56** |
| Dernier `DEC-xxx` | **DEC-188** |

---

## 📋 Kanban — S43 CLOS · SE-072

### ✅ Done — 10/10 slots

| US | Impact utilisateur livré |
|---|---|
| `US-SWEEP-S42` 🟢 | Aucun — sweep rejoué |
| `US-SEASON-TOKENS` 🟢 | Bouton « Retry » au violet de l'app ; dernière rangée de cartes atteignable |
| `US-DEMOCK-1a` 🟢 | Aucun — pilote du helper unique |
| `US-DEMOCK-HELPER` 🟠 | Aucun — le helper sait produire un studio et une date |
| `US-DEMOCK-1d` 🟠 | Aucun — la ligne `2023 · ★ 8.9 · MAPPA` protégée hors réseau |
| `US-DEMOCK-1b/c` 🟢 | Aucun — famille A soldée |
| `US-DEMOCK-2a` 🟠 | Aucun — 3 specs Jikan mortes rebranchées |
| `US-DEMOCK-2b` 🟠 | Aucun — la tenue en 387 px de nouveau mesurée |
| `US-DEMOCK-2c` (1+2+3) 🟢🟠 | Aucun — **famille B soldée, 5 dernières specs migrées, zéro bug révélé** |
| `US-HEADER-ICONS` 🟠 + `US-HEADER-TINT` 🟢 | **Icônes SVG identiques sur tous les téléphones · boutons 40 → 44 px · pastilles colorées · `aria-label` anglais · thème sombre réparé** |

**Gemini : 1 US en SE-072** (`US-HEADER-ICONS`) — fin de la traversée à zéro. Streak → 24.
4 micro-patchs `DEC-128` dans la session.

### 🔄 In Progress
Aucune.

### 📝 To Do — S44 (à composer en SE-073)
`AUD-56` audit pixel complet (demande PO, priorité 1) · `AUD-54` · `US-ESLINT-CI-1` ·
refonte chrome mobile (option A + icônes nues option B) · `US-MORELIKETHIS-FIX` ·
`US-STALE-SIGNAL` · `J10e-a/b/c` · `US-SYNC-FINALLY` · `US-CARD-CONVERGE-B` ·
`US-DEMOCK-3` (catch-all) · renommage des tests de `modal-status-gating`.
Composition → **`ROADMAP.md §2`**.

### 🗂️ Backlog
Cap S43 → S50 : **`ROADMAP.md`**.

---

## 🌐 Faits externes

| Fait | État | Dernière mesure |
|---|---|---|
| AniList `graphql.anilist.co` | ✅ Opérationnel | SE-071 — 🔻 **non remesuré en SE-072, écart validé par le PO** (aucune US du jour n'en dépendait) |
| Service worker de production | ❌ **Passe-plat, zéro cache** | SE-071 |
| PWA hors ligne | ❌ **Inexistante** — cap S49 | SE-071 |
| Multi-appareil (`AUD-42`) | 🔻 **Non revérifié** | SE-06x |

🔴 **Remesure obligatoire en ouverture de SE-073** (`PILOTAGE §6`) — deux sessions d'écart cumulé.

---

## 🕳️ Trous ouverts

1. **`AUD-56`** — audit pixel complet (Library, modales, toutes pages, clair **et** sombre). Demande PO explicite, à ouvrir en tête de S44. Aucune mesure faite.
2. **`AUD-54`** — vue Semaine qui reconstruit ses cartes en synchro (~7 s d'agitation). 🟠, cause non établie, composant non lu. **Seul constat de S43 touchant l'expérience réelle.**
3. **Batch4 non reprouvé par sortie brute** — vert sur parole du PO après le correctif `logout-modal-position`.
4. **Commit `HEAD` non relevé** après le correctif logout.
5. **`AGENTS.md §6` — règle du seed mono-jour non vérifiée** (Trou hérité SE-071). Ni confirmée ni infirmée.
6. **Famille C d'`AUD-52`** — `week-empty-day-cta` route `**/*`, fonctionne par accident.
7. **Titres mensongers dans `modal-status-gating`** — « TEST 1 ROUGE (avant fix) » et « TEST 2 VERT (après fix) » testent la même chose. À renommer en S44.
8. **Aucun token de couleur hors `--accent`** — les 5 teintes de l'en-tête sont en dur (`AUD-56` famille tokens).
9. **Bêta non ouverte** — décision PO reconduite. 27+ US livrées depuis S41, zéro observation utilisateur.
