# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-067** — clôture S41 |
| Sprint | **S41 CLOS** — 10 US mergées sur 10 slots + 4 micro-patchs |
| Version | **v0.35.0** — bump de clôture de sprint |
| Commit main | `9e3923e` |
| Prochaine session | **SE-068** — ouverture S42 « On peut nous faire confiance » |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **295** / 32 fichiers | SE-067 |
| Build | **178 modules**, `index` 368,90 kB (gzip 108,20) | SE-067 |
| Sweep E2E | **52/52** | SE-063 — **non rejoué depuis 4 sessions** |
| Specs E2E | 42 enregistrées en 5 batchs | SE-063 |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build` : un build vert vaut type-check vert.
> ⚠️ **Aucune sortie de Gemini n'a valeur de preuve** (`AUD-43`) : son `node_modules` diverge
> du nôtre — noms de chunks et `index.html` différents. Écart de build reproductible de 71 kB
> sur 3 livraisons. Seule la machine du PO mesure.

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | **0** |

---

## 📋 Kanban — S41 CLOS · SE-067

### ✅ Done — Sprint S41 · 10/10 slots

| US | Commit | Impact utilisateur livré |
|---|---|---|
| `US-PERSIST-P0b` | — | Se déconnecter ne détruit plus la bibliothèque locale |
| `US-PERSIST-P0a` | — | ⚠️ **Aucun** — branche jamais atteinte (Trou n°1) |
| `US-PERSIST-P0a2` | — | La bibliothèque revient à la connexion |
| `US-ONBOARD-PERSIST-A` | `b509ca0` | L'onboarding ne rejoue plus après reconnexion |
| `US-MALIMPORT-FIX` | `b469a18` | Un import MAL atterrit dans les bons bacs |
| `US-MONTH-COMINGSOON` | `e84b2aa` | La vue Mois illisible devient un « Coming Soon » assumé |
| `US-SEASON-SKIP-SESSION` | `80f9d11` | Écarter un titre de This Season ne le bannit plus de For You |
| `US-ANILIST-QUEUE-A` | `1256d20` | Un pic AniList gèle l'app 6 s au pire, au lieu de 120 s |
| `US-ANILIST-QUEUE-B` | `c3980cc` | Un anime en échec réessaie en 15 min ; 25 max par démarrage |
| `US-SYNC-PRIORITY` | `9e3923e` | Les séries invisibles (`awaitingSchedule`) passent devant |

**Micro-patchs DEC-128 (hors slots) :** `US-CARD-ORDER-A` · `US-CARD-ORDER-B` ·
`US-FIRESTORE-LIMITS` (plafond 100 → 500, règles publiées en console) · `US-REC-WHY-2LINES`.

**Retirées de S41 :** `US-MONTH-FIX` (annulée, DEC-163) · `US-SEASON-1TAP` (découpée → S42).

### 🔄 In Progress
Aucune.

### 📝 To Do
Aucune. **S41 est clos.** Le cap S42 vit dans `ROADMAP.md`.

**Glissent en S42 :** `US-TOUCH-A` · `US-TOUCH-B` (clôture à date, DEC-155).

### 🗂️ Backlog
Cap S42 → S50 : **`ROADMAP.md`**. Aucun item de backlog n'est listé ici.

---

## 🎯 Sprint Outcome Gate — S41 « Ce que j'ajoute reste »

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait
> pas avant ce sprint ? »*

**Réponse : gain de fiabilité visible (cas n°2).** Ce qu'il ajoute part au cloud, y reste,
revient à la connexion, tient jusqu'à 500 titres, et n'est plus perdu par une file réseau
gelée. Aucune fonctionnalité ajoutée — c'est la forme attendue de ce Goal.

**Budget dette respecté :** 0 US de dette pure sur 10.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| AniList — quota | ✅ **Aucun 429 observé** après `US-ANILIST-QUEUE-A/B` | **SE-067** | constat PO en prod |
| AniList — Recherche · This Season | fonctionnels | SE-067 | constat PO |
| Firestore — écriture | ✅ **AUTORISÉE.** `title: "Chainsmoker Cat"` présent dans `schedules/<uid>`, `timestamp: 1787567729316` | **SE-067** | console Firebase |
| Firestore — plafond | **500 entrées**, règles publiées en console | **SE-067** | console → Règles |
| Firestore — poids d'une entrée | **~1,1 ko** → 500 entrées ≈ 550 ko, marge 45 % sous la limite 1 Mo | **SE-067** | échantillon réel |
| Firestore — collection `schedules` | ✅ **purgée** par le PO — un seul document vivant | **SE-067** | console Firebase |
| Firestore — base cible | `ai-studio-58fc34cb-…`, europe-west2 | SE-064 | console Firebase |

---

## 🕳️ Trous ouverts

1. **🔴 `US-PERSIST-P0a` est une US morte.** Branche jamais atteinte en connexion réelle.
   Inoffensive. **Suppression non instruite depuis 3 sessions** (SE-065 → SE-067).
2. **🟠 `AUD-33` soldé sous réserve.** Plus de 429 en prod, mais le hash du chunk servi n'a
   pas été relevé : le correctif est **plausiblement** en ligne, pas prouvé. Falsifiable par
   un 429 après 5+ ajouts d'affilée ou après un import MAL.
3. **🟠 `AUD-37` non vérifié.** Le rattrapage d'horaire d'un import MAL n'a pas été testé sur
   un vrai fichier MAL. Trois correctifs le visent — aucun n'est observé.
4. **🔴 Multi-appareil dégradé** — `aanime_sync_ts` est local (`AUD-42`), `US-ONBOARD-PERSIST-B`
   non faite. **Limite à annoncer aux bêta-testeurs, pas un bloquant.**
5. **🟠 Sweep E2E non rejoué depuis SE-063** — 4 sessions, 10 US. Le filet le plus large du
   projet n'a pas été armé de tout le sprint. → **premier geste de S42.**
6. **🟠 `AUD-44` — `isSyncing` sans `try/finally`.** Un throw laisserait le spinner à `true`
   indéfiniment. Latent. → S43 avec le lot ESLint.
7. **`AUD-39`** — `AnimeCard` affiche `title`, `RecCard` affiche `title_english || title`.
8. **`AUD-40`** — ~40 lignes de CSS mort (`.card-cp-why`, `.rec-why-*`) + doublon `.card-cp-title`.
9. **`AUD-41`** — `studios: ["8-bit", "8-bit"]`, doublon de normalisation AniList. Non vérifié.
10. **ESLint jamais exécuté** — `R-CODE-1` non vérifiée. → S43.
11. **12 specs E2E vertes tapant le réseau réel** (`AUD-24`) — sweep non déterministe. → S43.
12. **🔴 Collision d'ID `AUD-32`/`AUD-33`** dans `AUDIT.md` — arbitrée par ce fichier, note de
    réconciliation en campagne SE-066.

---

## ✅ Bêta — état du verrou

| Verrou | État |
|---|---|
| `AUD-32` écritures Firestore refusées | ✅ Mort, mesuré |
| Plafond 100 titres | ✅ Levé à 500, publié |
| Documents fossiles `schedules` | ✅ Purgés |
| Onboarding rejoué à chaque connexion | ✅ Corrigé |
| `AUD-33` AniList 429 au boot | ✅ Corrigé, sous réserve (trou n°2) |
| Multi-appareil | ⚠️ Dégradé — limite à annoncer |

> **Aucun bloquant technique ne subsiste.** L'ouverture aux bêta-testeurs est une décision
> produit, pas une attente de correctif.
