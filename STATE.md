# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-066** — exécution S41, 4 US livrées |
| Sprint | **S41 en cours** — 7 US mergées sur 10 |
| Version | **v0.34.0** — pas de bump : le sprint n'est pas clos |
| Commit main | `80f9d11` |
| Prochaine session | **SE-067** — `US-CARD-ORDER`, puis les 3 slots restants |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **291** / 32 fichiers | SE-066 |
| Build | **178 modules**, `index` 368,64 kB | SE-066 — gzip non relevé |
| Sweep E2E | **52/52** | SE-063 — non rejoué depuis |
| Specs E2E | 42 enregistrées en 5 batchs | SE-063 |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build` : un build vert vaut type-check vert.
> ⚠️ Les sorties de build de Gemini ne font pas foi : écart de 70 kB constaté sur `80f9d11`
> (439 kB annoncés vs 368,64 kB réels). Seule la machine du PO mesure.

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | **0** |

---

## 📋 Kanban — S41 · SE-066

### ✅ Done — Sprint S41 · 7/10

| US | Commit | Impact utilisateur livré |
|---|---|---|
| `US-PERSIST-P0b` | — | Se déconnecter ne détruit plus la bibliothèque locale |
| `US-PERSIST-P0a` | — | ⚠️ **Aucun** — branche jamais atteinte (Trou n°1) |
| `US-PERSIST-P0a2` | — | La bibliothèque revient à la connexion. Vérifié en réel |
| `US-ONBOARD-PERSIST-A` | `b509ca0` | L'onboarding ne rejoue plus après déconnexion/reconnexion |
| `US-MALIMPORT-FIX` | `b469a18` | Un import MAL atterrit dans les bons bacs, zéro entrée en radar |
| `US-MONTH-COMINGSOON` | `e84b2aa` | La vue Mois illisible est remplacée par un « Coming Soon » assumé |
| `US-SEASON-SKIP-SESSION` | `80f9d11` | Écarter un titre de This Season ne le bannit plus de For You |

**Retirées de S41 :** `US-MONTH-FIX` (annulée, DEC-163) · `US-FIRESTORE-LIMITS` (au frigo,
AUD-32 non reproductible) · `US-SEASON-1TAP` (découpée, → S42).

**Streak Gemini : 12.** 4 US, 4 merges au premier coup en SE-066.

### 🔄 In Progress
Aucune.

### 📝 To Do — Sprint S41 · 🎯 **« Ce que j'ajoute reste »**

> **3 slots restants, 4 candidats. Clôture à date, pas à épuisement (DEC-155).**

| # | US | Risque | Effort | Impact utilisateur |
|---|---|---|---|---|
| 1 | `US-CARD-ORDER` | 🟢 | XS | On lit le titre **avant** de décider Skip/Add |
| 2 | `US-REC-WHY-2LINES` | 🟢 | XS | La raison d'une reco n'est plus coupée en plein mot |
| 3 | `US-TOUCH-A` | 🟢 | S | 6 contrôles deviennent tapables — zéro pixel visuel changé |
| 4 | `US-TOUCH-B` | 🟢 | S | 24 à 48 cibles remontées à 44 px sur les écrans de tri |

### 🗂️ Backlog
Cap S42 → S50 : **`ROADMAP.md`**. Aucun item de backlog n'est listé ici.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| AniList — Recherche · This Season | fonctionnels | SE-065 | constat PO |
| AniList — quota | 🔴 `429` au boot, suivi d'erreurs CORS | SE-065 | console DevTools PO |
| Firestore — écriture | 🔴 refusée : `Missing or insufficient permissions` | SE-065 | console DevTools PO |
| Firestore — base cible | `ai-studio-58fc34cb-…`, europe-west2 | SE-064 | console Firebase |
| Firestore — règles déployées | identiques au dépôt, déployées le 14/05/2026 | SE-064 | console → Sécurité |

> ⚠️ **Aucun fait externe n'a été remesuré en SE-066.** Ils ont deux sessions de retard.
> Le PO n'a pas reproduit l'erreur Firestore — **absence de reproduction ≠ absence de bug.**
> Un `channel?VER=8` en 200 ne prouve rien : les erreurs Firestore voyagent dans le flux
> WebChannel, jamais dans le statut HTTP.

---

## 🕳️ Trous ouverts

1. **🔴 `US-PERSIST-P0a` est une US morte.** Sa branche n'est jamais atteinte en connexion
   réelle. Inoffensive. **Décision de suppression toujours non instruite** — reportée SE-065
   → SE-066 → SE-067. Ne pas supprimer sans US.
2. **🔴 `AUD-32` — écritures Firestore refusées.** Au frigo, **pas soldé. Bloquant bêta.**
3. **🟠 `AUD-33` — AniList `429` au démarrage.** Non instruit.
4. **🟠 `AUD-37` — import MAL « Watching » → Plan to Watch.** Correct techniquement (MAL
   n'exporte aucun `day`). Non vérifié : `useSync` récupère-t-il le `day` ensuite ? **À
   trancher sur un vrai fichier MAL, pas sur du code.**
5. **🔴 Multi-appareil cassé** — `US-ONBOARD-PERSIST-B`, S42.
6. **🔴 Collision d'ID dans `AUDIT.md`** : `AUD-32`/`AUD-33` attribués deux fois. Arbitrage
   par ce fichier, note de réconciliation en campagne SE-066.
7. **🔴 `DECISIONS.md` §en-tête périmé** — annonçait `DEC-159`, corrigé à `DEC-165`.
8. **ESLint jamais exécuté** — `R-CODE-1` non vérifiée. → S43.
9. **12 specs E2E vertes tapant le réseau réel** (`AUD-24`) — sweep non déterministe. → S43.
10. **8 documents fossiles dans `schedules`** (`AUD-31`) — purge console avant lancement.
