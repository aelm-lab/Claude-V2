# STATE.md — État courant du projet Aanime

> **Rôle :** le présent uniquement — session en cours, métriques, Kanban, trous ouverts.
> **Pas ici :** le passé (`HISTORIQUE.md`) · le cap multi-sprints (`ROADMAP.md`) · les règles (`PILOTAGE.md`).
>
> **Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.**
- si un document est absent de la Knowledge et n'a pas été collé dans le chat, je ne l'invente ni ne l'infère depuis un extrait de recherche partiel. Je dis explicitement lequel me manque et j'attends qu'il soit collé avant de trancher dessus.

---

## 📍 Session courante

| | |
|---|---|
| Session | **SE-065** — ouverture et exécution de S41 |
| Sprint | **S41 démarré** — 3 US mergées sur 10 |
| Version | **v0.34.0** — pas de bump : le sprint n'est pas clos |
| Commit main | `27ee0ca` |
| Prochaine session | **SE-066** — `US-ONBOARD-PERSIST` puis `US-FIRESTORE-LIMITS` |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **271** / 30 fichiers | SE-065 |
| Build | **180 modules**, `index` 368,76 kB (gzip 108,15) | SE-065 |
| Sweep E2E | **52/52** | SE-063 — non rejoué en SE-065 |
| Specs E2E | 42 enregistrées en 5 batchs | SE-063 |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build` : un build vert vaut type-check vert.

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Documents au-dessus du plafond H7 | **0** |

---

## 📋 Kanban — S41 · SE-065

### ✅ Done — Sprint S41

| US | Sortie | Impact utilisateur livré |
|---|---|---|
| **`US-PERSIST-P0b`** | 3 fichiers · 268 tests | Se déconnecter ne détruit plus la bibliothèque locale ; tout appareil déjà cassé se répare seul au chargement suivant |
| **`US-PERSIST-P0a`** | 2 fichiers · 270 tests | ⚠️ **Aucun** — branche jamais atteinte (voir Trou n°1) |
| **`US-PERSIST-P0a2`** | 2 fichiers · 271 tests | **La bibliothèque revient à la connexion.** Vérifié par le PO en conditions réelles |

**Micro-patchs DEC-128 appliqués :** suppression du stub mort `_syncAnimeUpdates`
(`usePersistence.ts`) · correction de la fixture `addAnime` dans `usePersistence.guard.spec.ts`.

### 🔄 In Progress
Aucune.

### 📝 To Do — Sprint S41 · 🎯 **« Ce que j'ajoute reste »**

> **Dérogation DEC-155 déclarée : 10 planifiées / 0 flex.** Aucun bêta-testeur n'entre avant
> que les trous n°2 et n°3 soient fermés.

| # | US | Risque | Effort | Impact utilisateur |
|---|---|---|---|---|
| 1 | **`US-ONBOARD-PERSIST`** | 🟠 | S | L'onboarding cesse d'être rejoué à chaque connexion et sur chaque appareil |
| 2 | **`US-FIRESTORE-LIMITS`** | 🔴 | M | Les sauvegardes cessent d'être refusées en silence ; la liste ne s'arrête plus à 100 titres |
| 3 | `US-MALIMPORT-FIX` | 🟠 | XS | Un import MAL de 300 titres atterrit dans les bons bacs |
| 4 | `US-MONTH-FIX` | 🟢 | XS | La vue Mois redevient lisible en thème clair |
| 5 | `US-SEASON-1TAP` | 🟠 | S | This Season : 1 tap au lieu de 2 + modale. Skip session-only (DEC-159) |
| 6 | `US-CARD-ORDER` | 🟢 | XS | On lit le titre **avant** de décider Skip/Add |
| 7 | `US-REC-WHY-2LINES` | 🟢 | XS | La raison d'une reco n'est plus coupée en plein mot |
| 8 | `US-TOUCH-A` | 🟢 | S | 6 contrôles deviennent tapables — zéro pixel visuel changé |
| 9 | `US-TOUCH-B` | 🟢 | S | 24 à 48 cibles remontées à 44 px sur les écrans de tri |

### 🗂️ Backlog
Cap S42 → S50 : **`ROADMAP.md`**. Aucun item de backlog n'est listé ici.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| AniList — Recherche · This Season | **fonctionnels** | **SE-065** | constat PO |
| AniList — quota | 🔴 **`429 Too Many Requests` au boot**, suivi d'erreurs CORS | **SE-065** | console DevTools PO |
| Firestore — écriture | 🔴 **refusée** : `Missing or insufficient permissions` sur `schedules/<uid>` | **SE-065** | console DevTools PO |
| Firestore — base cible | `ai-studio-58fc34cb-…`, europe-west2 | SE-064 | console Firebase |
| Firestore — règles déployées | identiques au dépôt, déployées le 14/05/2026 | SE-064 | console → Sécurité |

> ⚠️ Un `channel?VER=8` en 200 **ne prouve pas** le succès : les erreurs Firestore voyagent
> dans le flux WebChannel, jamais dans le statut HTTP.

---

## 🕳️ Trous ouverts

1. **🔴 `US-PERSIST-P0a` est une US morte.** Sa branche (`to.meta.guestOnly && isLoggedIn`)
   n'est jamais atteinte lors d'une connexion réelle. Elle est **inoffensive** et couvre son
   propre cas (utilisateur connecté visitant `/login`). `P0a2` porte le correctif réel.
   **Décision de suppression à instruire en SE-066** — ne pas supprimer sans US.
2. **🔴 `AUD-32` — écritures Firestore refusées.** Une sauvegarde réelle a été rejetée par le
   serveur. Cause non établie. **Bloquant bêta.** → `US-FIRESTORE-LIMITS`.
3. **🟠 `AUD-33` — AniList `429` au démarrage.** Le PO doit actualiser pour voir ses animes
   dans Week. Cause non établie. → à instruire en SE-066.
4. **🟠 `AUD-17` rouvert** — le stub `_startBackgroundRelationFetch` avait été supprimé, pas
   `_syncAnimeUpdates`. Ce dernier l'est depuis SE-065 : **le constat est maintenant soldé
   pour de bon.**
5. **ESLint jamais exécuté** — `R-CODE-1` non vérifiée. → S43.
6. **12 specs E2E vertes tapant le réseau réel** (`AUD-24`) — sweep non déterministe. → S43.
7. **8 documents fossiles dans `schedules`** (`AUD-31`) — purge console avant lancement public.
