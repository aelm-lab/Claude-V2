# ROADMAP.md — Cap produit multi-sprints

> **Où mettre ce fichier :** Knowledge du projet Claude Chat (`aelm-lab/Claude-V2`).
>
> **🔻 SATELLITE — hors ordre de lecture.** Ouvert à deux moments seulement : composition de
> sprint, et PI Planning. Créé par **DEC-156**.
>
> **Rôle :** le cap au-delà du sprint courant. **Pas ici :** le sprint en cours
> (`STATE.md §Kanban`) · le contenu des constats (`AUDIT.md`) · les règles (`PILOTAGE.md`).
>
> **Rafraîchi à chaque clôture de sprint.** PI Planning complet toutes les 4 clôtures.
> **Prochain PI complet : clôture S44**, avec des retours bêta réels.

---

## §1 — Le cap S41 → S50

| Sprint | 🎯 Sprint Goal | Confiance |
|---|---|---|
| **S41** | **Ce que j'ajoute reste** — la persistance cesse de mentir | ✅ **CLOS — v0.35.0, 10/10** |
| **S42** | **On peut nous faire confiance** — plus aucun écran vide ni muet | ✅ **CLOS — v0.36.0, 10/10** |
| **S43** | **Rien ne casse en douce** — filets automatisés, sweep déterministe, dette soldée avant bêta | 🟢 Ferme · 🔵 **PROCHAIN** |
| **S44** | **L'arrivée vaut le produit** — onboarding et premiers retours bêta | 🟡 Moyenne |
| **S45** | Consolidation bêta 1 — le top des irritants remontés | 🟠 Thématique |
| **S46** | Stats enrichies (EPIC 11) — tendances, historique de visionnage | 🟠 Thématique |
| **S47** | Notifications (EPIC 11) — « ton épisode sort ce soir » | 🟠 Thématique |
| **S48** | Composition modale & polish — sur données d'usage, pas sur esthétique | 🟠 Thématique |
| **S49** | Durcissement pré-lancement — bundle, offline, edge cases sync | 🟠 Thématique |
| **S50** | 🚀 Lancement public | 🟠 Thématique |

> **Ce tableau est un cap, pas un contrat.** À partir de S45, la composition appartient aux
> retours bêta, pas à cette page.

---

## §2 — Sprints détaillés

### S42 — « On peut nous faire confiance » · ✅ CLOS, 10/10

Livraison intégrale → `HISTORIQUE.md §Sessions` (SE-068, SE-069) et `STATE.md §Kanban`.

**Réponse à la Sprint Outcome Gate :** gain de fiabilité visible — l'écran cesse de mentir sur
l'état réel (onglet d'ajout, compte retrouvé, carte déjà suivie, dropdown lisible).

**Sorties de périmètre décidées en cours de sprint :**
- `US-SEASON-1TAP` supprimée, fusionnée dans `US-CARD-CONVERGE-A` (SE-068)
- `US-FIRESTORE-LIMITS` retirée — livrée en micro-patch en SE-067
- `US-PERSIST-P0a` supprimée — US morte
- `US-MORELIKETHIS-FIX` sortie du sprint au profit de `US-SEASON-FRESH` (SE-069, arbitrage PO)
- `US-HEADER-ICONS` non prise — glisse en S43

---

### S43 — « Rien ne casse en douce » · 7 planifiées + 3 flex

**Pourquoi ce Goal maintenant.** S42 a livré 10 US en 2 sessions sans rejouer le sweep E2E.
Le filet est daté de SE-068 alors que recherche, onboarding et This Season ont tous changé.
Avant d'ouvrir aux testeurs, on veut savoir que rien n'a cassé en silence — et on veut solder
la dette qui deviendrait embarrassante sous les yeux d'un utilisateur réel.

| # | US | Risque | Effort | Impact utilisateur |
|---|---|---|---|---|
| 1 | `US-SWEEP-S42` | 🟢 | S | Aucun direct — rejeu des 5 batchs E2E après 10 US. **Ouverture de sprint, avant toute autre.** |
| 2 | `US-HEADER-ICONS` | 🟠 | M | Fin des emojis ☀️📅📥❓🚪 qui changent de forme selon le téléphone ; libellés lisibles par lecteur d'écran |
| 3 | `US-CARD-CONVERGE-B` (Coming Soon seul) | 🟠 | S | Coming Soon : 1 tap pour ajouter, carte identique à This Season. **Ne couvre pas Completed / Plan to Watch (DEC-181)** |
| 4 | `US-MORELIKETHIS-FIX` (absorbe `AUD-16`) | 🔴 | S | « More like this » revit : une relation ouvre une modale avec jaquette, score et synopsis au lieu du vide |
| 5 | `US-STALE-SIGNAL` (`AUD-05`, `DEC-158`) | 🟠 | M | L'utilisateur voit quand ses données sont périmées au lieu de croire à un calendrier faux. Supprime le signal mort `staleDataWarning` |
| 6 | `US-SYNC-FINALLY` (`AUD-44`) | 🟠 | M | Aucun direct — `isSyncing` sans `try/finally`, ~90 lignes réindentées. **Seule, jamais en marge d'une autre US** |
| 7 | `US-PERF-BASELINE` → `US-PERF-GATE` | 🟢 M / 🟠 S | Aucun direct puis : un ralentissement futur devient une spec rouge bloquée au merge. **Ordre imposé, jamais l'inverse** |

**Flex (3 slots) :** `US-CARD-DEBT` (`AUD-40`/`41` — CSS mort, doublon `.card-cp-title`, `studios: ["8-bit","8-bit"]`, titre tronqué sans recours) · `US-ONBOARD-FALLBACK` · **1 slot réservé aux retours bêta**

**Micro-patchs candidats (hors slot, DEC-128) :**
- `AUD-49` — renumérotation `DEC-158` (b) → `DEC-176`, patch documentaire pur
- `AUD-51` — pastille « Add » 36 px → 44 px, **si et seulement si** un testeur rate le bouton

**Pré-requis de sprint :**
- `git status` vide — le micro-patch `RecCard` de SE-069 doit être commité avant toute US
- Les 2 hash non relevés de SE-069 doivent être reconstitués

---

### S44 — « L'arrivée vaut le produit »

Premier sprint avec des retours bêta réels. Composition non figée : `DEC-155` impose 7 + 3 flex,
et les 3 flex appartiennent aux testeurs.

**Ce qui est déjà nommé :**
- **Note aux testeurs** — document à rédiger avant l'envoi. Doit porter au minimum :
  - `AUD-37` : « combien de titres avais-tu sur MAL, combien en retrouves-tu ? »
  - la limite multi-appareil résiduelle, si la bande de rattrapage ne suffit pas
- **US produit « cartes de bibliothèque »** (`DEC-181`) — Completed et Plan to Watch attendent
  un bouton dont la sémantique n'est pas « Add ». « Rewatch » ? « Start watching » ? Rien ?
  **Non tranchable sans usage réel.** C'est le premier arbitrage à porter aux testeurs.
- **Dette de vérité** — « We'll sync it later » promet une re-synchronisation que rien dans le
  code ne garantit. Même famille qu'`US-ADD-TOAST-TRUTH`. À traiter avant que le message ne
  soit vu par des utilisateurs réels.
- **Indicateur de sync** — le rattrapage progressif reste invisible.
- **Convergence `onboardingFilter`** (`DEC-178`) — la règle « en cours de diffusion » existe
  encore en double : `useAddAnime.resolveTargetState` et `utils/onboardingFilter.ts`.

---

### S45 → S50 — thématiques, non composées

Volontairement laissés vides de détail. `DEC-155` : la composition d'un sprint se fait à son
ouverture, sur l'état réel du produit — pas six sprints à l'avance. Les intitulés du §1 sont
des intentions, pas des engagements.

**Un point de vigilance porté depuis S42 :** `BENCHMARK.md` attend un triage. Il ne sera utile
qu'une fois les surfaces observables avec du contenu réel, donc **pas avant S45**.

---

## §3 — 🔻 Section absente de mon contexte

`ROADMAP.md` peut porter des sections au-delà du §2 que je n'ai pas lues cette session
(les extraits de recherche se sont arrêtés au §2). **Je ne les ai pas régénérées et je ne les
ai pas inventées.**

Avant de remplacer le fichier en Knowledge, vérifier s'il existe un §3 ou plus dans la version
actuelle — et, si oui, le recoller tel quel sous cette ligne.
