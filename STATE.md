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
| Session | **SE-064** — hors sprint (chantier P0 « zéro zone d'ombre ») |
| Sprint | **S41 composé, non démarré.** Démarre en SE-065 |
| Version | **v0.34.0** — aucun bump (aucune US livrée) |
| Commit main | `365a6aa` + micro-patch P0-6 (DEC-128, non commité au moment de la clôture) |
| Prochaine session | **SE-065 = ouverture S41**, US n°1 = `US-PERSIST-P0` |

---

## 📊 Métriques

| Métrique | Valeur | Fraîcheur |
|---|---|---|
| Tests unitaires | **265** / 29 fichiers | SE-064 (micro-patch P0-6) |
| Build | **180 modules**, `index` 368,71 kB (gzip 108,15) | SE-064 |
| Sweep E2E | **52/52** | SE-063 |
| Specs E2E | 42 enregistrées en 5 batchs | SE-063 |
| ESLint | ❌ **jamais exécuté** — « zéro `any` » vérifié par aucun outil | — |

> `vue-tsc --noEmit` est embarqué dans `npm run build` : un build vert vaut type-check vert.

## 📚 Métriques documentaires

| Métrique | Valeur |
|---|---|
| Corpus | **15 documents** (10 en ordre de lecture + 5 satellites) |
| Nouveau satellite | `ROADMAP.md` (DEC-156) |
| Documents au-dessus du plafond H7 | **0** |

---

## 📋 Kanban — S41 · SE-064

### ✅ Done — hors sprint (SE-064)

| Chantier | Sortie |
|---|---|
| **P0 « zéro zone d'ombre »** | 22 constats tranchés par grep. 7 soldés, 11 confirmés vivants, 6 nouveaux (AUD-26→31) |
| **Micro-patch P0-6** | Stub `_startBackgroundRelationFetch` supprimé de `usePersistence.ts`. Porte verte ✅ |
| **Déploiement `AGENTS.md`** | Racine `A-Anime` alignée : `installAniListMock` présent, patterns REST morts absents |
| **PI Planning S41→S50** | → `ROADMAP.md`. DEC-155→159 |

### 🔄 In Progress
Aucune. S41 non démarré.

### 📝 To Do — Sprint S41 · 🎯 **« Ce que j'ajoute reste »**

> **Dérogation DEC-155 déclarée : 10 planifiées / 0 flex.** Aucun bêta-testeur n'entre avant
> `US-PERSIST-P0` — il n'y a donc pas de retour à absorber pendant ce sprint.

| # | US | Risque | Effort | Impact utilisateur |
|---|---|---|---|---|
| 1 | **`US-PERSIST-P0`** | 🔴 | M | Ce que l'utilisateur ajoute survit à une déconnexion, un changement d'appareil, un vidage de cache |
| 2 | **`US-FIRESTORE-LIMITS`** | 🔴 | M | La liste cesse de s'arrêter silencieusement à 100 titres ; un échec de sauvegarde devient visible |
| 3 | `US-ONBOARD-PERSIST` | 🟠 | S | L'onboarding n'est plus rejoué à chaque connexion ni sur chaque appareil |
| 4 | `US-MALIMPORT-FIX` | 🟠 | XS | Un import MAL de 300 titres atterrit dans les bons bacs, pas 100 % en Coming Soon |
| 5 | `US-MONTH-FIX` | 🟢 | XS | La vue Mois redevient lisible en thème clair (titres blancs sur blanc aujourd'hui) |
| 6 | `US-SEASON-1TAP` | 🟠 | S | This Season : 1 tap au lieu de 2 + modale. Skip session-only (DEC-159) |
| 7 | `US-CARD-ORDER` | 🟢 | XS | On lit le titre **avant** de décider Skip/Add |
| 8 | `US-REC-WHY-2LINES` | 🟢 | XS | La raison d'une reco n'est plus coupée en plein mot |
| 9 | `US-TOUCH-A` | 🟢 | S | 6 contrôles deviennent tapables — zéro pixel visuel changé |
| 10 | `US-TOUCH-B` | 🟢 | S | 24 à 48 cibles remontées à 44 px sur les écrans de tri |

### 🗂️ Backlog
Cap S42 → S50 : **`ROADMAP.md`**. Aucun item de backlog n'est listé ici.

---

## 🌐 Faits externes

| Fait | Mesure | Date | Méthode |
|---|---|---|---|
| AniList — Recherche · This Season · For You · Library Upcoming | **fonctionnels** | SE-063.b | constat PO |
| Firestore — base cible | `ai-studio-58fc34cb-5a66-41ef-afc5-08d725019708`, europe-west2 | **SE-064** | console Firebase |
| Firestore — règles déployées | **identiques au dépôt**, déployées le 14/05/2026 | **SE-064** | console → Sécurité |
| Firestore — moteur de règles | 176 autorisations · **0 refus · 0 erreur** sur 7 j | **SE-064** | console → Utilisation |
| Firestore — trafic app | lecture `useFirestore.ts:60` · écriture `:85`, WebChannel 200 | **SE-064** | DevTools Réseau |

> ⚠️ Un `channel?VER=8` en 200 **ne prouve pas** le succès : les erreurs Firestore voyagent
> dans le flux WebChannel, jamais dans le statut HTTP.

---

## 🕳️ Trous ouverts

1. **🔴 `US-PERSIST-P0` non livrée** — aucun accès bêta-testeur avant sa livraison **et** sa
   vérification par le PO sur deux appareils.
2. **⚠️ Un seul grep de la session P0 n'a pas été exécuté** — `_syncAnimeUpdates`
   (`usePersistence.ts:13`) est un stub vide `await`é l.284. `AUDIT.md` (SE-061/062) situe la
   vraie sync en 4 points vivants (`App.vue:52`, `AnimeModal.vue:134`/`:145`,
   `DiscoverExplorePage.vue:225`), ce qui en ferait un doublon mort — **non prouvé, AP-PROCESS-2**.
   Commande de levée : `findstr /n /s /c:"syncAnimeUpdates" src\*.ts src\*.vue`
3. **ESLint jamais exécuté** — R-CODE-1 non vérifiée. → S43.
4. **12 specs E2E vertes tapant le réseau réel** (`AUD-24`) — sweep non déterministe. → S43.
5. **8 documents fossiles dans `schedules`** (`AUD-31`) — purge console manuelle avant lancement public.
6. **`ARCHITECTURE_TECHNIQUE §6`** — la séquence de boot documente une étape « sync » qui
   pourrait ne rien faire. À corriger après levée du trou n°2.
