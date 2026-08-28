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
> **Prochain PI complet : clôture S44** — avec les premiers retours bêta réels.
>
> 🔻 **Régénéré intégralement en SE-070.** La version SE-069 avait perdu les §3 à §6 :
> une régénération partielle est une amputation. **Ce document se régénère entier ou pas du tout.**

---

## §1 — Le cap S41 → S50

| Sprint | 🎯 Sprint Goal | Confiance |
|---|---|---|
| **S41** | **Ce que j'ajoute reste** — la persistance cesse de mentir | ✅ **CLOS — v0.35.0, 10/10** |
| **S42** | **On peut nous faire confiance** — plus aucun écran vide ni muet | ✅ **CLOS — v0.36.0, 10/10** |
| **S43** | **Rien ne casse en douce** — filets automatisés et sweep déterministe | ✅ CLOS — v0.37.0, 10/10 |
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

### S42 — « On peut nous faire confiance » · ✅ CLOS, v0.36.0, 10/10

Livraison intégrale → `STATE.md §Kanban` et `HISTORIQUE.md`.

**Réponse à la Sprint Outcome Gate :** gain de fiabilité visible — l'écran cesse de mentir sur
l'état réel (onglet d'ajout, compte retrouvé, carte déjà suivie, dropdown lisible).

**Sorties de périmètre décidées en cours de sprint :**

| Item | Sort |
|---|---|
| `US-SEASON-1TAP` | ❌ Supprimée — même geste qu'`US-CARD-CONVERGE-A`. Fusionnée (SE-068) |
| `US-FIRESTORE-LIMITS` (`AUD-32`) | ✅ Livrée en micro-patch en SE-067. Retirée du sprint (SE-068) |
| `US-PERSIST-P0a` | ❌ Supprimée — US morte, traînée 4 sessions (SE-068) |
| `US-TOUCH-B` | Slot consommé par `US-ADD-TOAST-TRUTH` (arbitrage PO, SE-068) |
| `US-MORELIKETHIS-FIX` | ↦ S44 — sortie au profit d'`US-SEASON-FRESH` (SE-069) |
| `US-HEADER-ICONS` | ↦ S43 |
| `US-STALE-SIGNAL` | ↦ S44 — débloquée en SE-069, plus de slot |
| `US-PERF-BASELINE` / `US-PERF-GATE` | ↦ S44 |
| `US-TOUCH-A`, `US-ONBOARD-FALLBACK` | ↦ S44 flex |

**US non prévues, entrées en cours de sprint :** `US-SEARCH-USE-ADDANIME` (`AUD-47`),
`US-SEASON-FRESH` (`AUD-48`), micro-patchs `AUD-46` et `RecCard`.

---

### S43 — « Rien ne casse en douce » · EN COURS depuis SE-070

**Pourquoi ce Goal.** S42 a livré 10 US en 2 sessions sans rejouer le sweep. Avant d'ouvrir aux
testeurs, on veut savoir que rien n'a cassé en silence.

🔴 **Le sprint a changé de nature en SE-070.** Il était prévu comme un lot d'outillage confortable.
La lecture a montré que **le filet lui-même est troué** : sur 42 specs, 11 portent un mock mort
(URL Jikan, abandonnées depuis S40) et au moins une n'a aucun mock du tout. Le « 52/52 » couvre en
réalité une trentaine de specs. Ce n'est plus du confort, c'est le Goal.

#### Composition arrêtée (7 planifiées + 3 flex, `DEC-155`)

| # | US | Risque | État | Impact utilisateur |
|---|---|---|---|---|
| 1 | `US-SWEEP-S42` | 🟢 | ✅ **MERGE** (SE-070) | Aucun direct. A révélé `search-quick-add` rouge : spec périmée, pas régression |
| 2 | `US-SEASON-TOKENS` | 🟢 | ✅ **MERGE** — 2 micro-patchs (SE-070) | Bouton « Retry » de This Season au violet de l'app, plus au bleu Bootstrap. Cartes à la même taille qu'ailleurs, dernière rangée atteignable |
| 3 | `US-DEMOCK-1a` (`search-dedup`) | 🟢 | ✅ **MERGE** — pilote (SE-070) | Aucun. **A prouvé que le helper tient et que `startDate: null` est toléré** |
| 4 | `US-DEMOCK-HELPER` | 🟠 | 📝 À faire | Aucun. Étend `MediaSeed` (`studioName`, `startDate`) — **sans quoi `search-enriched` est non migrable** |
| 5 | `US-DEMOCK-1b/c/d` | 🟢 ×2 / 🟠 ×1 | 📝 À faire | Aucun. Migre `search-quick-add`, `search-hides-nav`, `discover-season-dedup`. `search-enriched` attend le HELPER |
| 6 | `US-DEMOCK-2` (`AUD-52`, 5 specs Jikan mortes) | 🟠 | 📝 À faire | Aucun **si vert**. Un rouge ici = vrai bug de production découvert, à budgéter |
| 7 | `US-HEADER-ICONS` (+ `AUD-51`) | 🟠 M | 📝 À faire | **La seule US visible du sprint.** Fin des emojis ☀️📅📥❓🚪 qui changent de forme selon le téléphone ; boutons 40 → 44 px ; `aria-label` tout anglais |

**Flex (3) :** `US-DEMOCK-3` (6 specs restantes + le catch-all `**/*`) · `US-ESLINT-CI-1` ·
**1 slot réservé aux retours bêta**

**Micro-patchs candidats (hors slot, `DEC-128`) :**
- `AUD-49` — renumérotation `DEC-158` (b) → `DEC-176`, patch documentaire pur

#### 🔴 Point ouvert à l'écriture de ce document

`modal-next-episode.spec.ts` est **rouge** en fin de SE-070, sans qu'aucun patch de la session
ne touche son sujet (`.rowcard`). Elle ne figure dans aucun `page.route` : elle n'a **aucun mock
réseau**. Hypothèse dominante non confirmée : elle tape AniList en vrai et subit le circuit
breaker. **Cause à établir par lecture en SE-071, avant toute autre US.**

#### 🔴 Le vrai arbitrage reste la date de la bêta

**25+ US livrées depuis S41. Zéro observée par un utilisateur réel.**

S43 ne produit qu'**une** US visible (`US-HEADER-ICONS`). `PILOTAGE.md §5` autorise un sprint
sans gain visible **en exception, pas en routine** — le compteur est armé, S44 doit être produit.

**Décision PO (SE-070) : bêta NON ouverte en S43.** Motif : on n'invite pas de testeurs derrière
un filet dont on vient de découvrir les trous. Arbitrage assumé, coût nommé : un sprint de plus
sans signal terrain.

**Prérequis avant envoi, inchangés :**
- Sweep vert **et** `AUD-52` soldé — un sweep vert sur des mocks morts ne prouve rien
- La **note aux testeurs** rédigée (§5)
- `AUD-42` revérifié à l'œil sur second appareil (« Clear site data »)

---

### S44 — « L'arrivée vaut le produit » · 7 planifiées + 3 flex

**Composition non figée** : `DEC-155` impose 7 + 3, et les 3 flex appartiennent aux testeurs.

**Déjà planifié :**

| US | Risque | Effort | Impact utilisateur |
|---|---|---|---|
| `J10e-a/b/c` (repli orphelins MAL titre+année, `DEC-145`) | 🟠 | M ×3 | Les animes sans `idMal` cessent de disparaître silencieusement |
| `US-MORELIKETHIS-FIX` (absorbe `AUD-16`) | 🔴 | S | « More like this » revit : une relation ouvre une modale avec jaquette, score et synopsis au lieu du vide |
| `US-STALE-SIGNAL` (`AUD-05`, `DEC-158`) | 🟠 | M | L'utilisateur voit quand ses données sont périmées au lieu de croire à un calendrier faux |
| `US-SYNC-FINALLY` (`AUD-44`) | 🟠 | M | Aucun direct — `isSyncing` sans `try/finally`, ~90 lignes réindentées. **Seule, jamais en marge** |
| `US-PERF-BASELINE` → `US-PERF-GATE` | 🟢 M / 🟠 S | Aucun puis : un ralentissement futur devient une spec rouge bloquée au merge. **Ordre imposé** |
| `US-CARD-CONVERGE-B` (Coming Soon seul, `DEC-181` annulé — voir §3) | 🟠 | S | Coming Soon : 1 tap pour ajouter, carte identique à This Season |
| `US-STATUS-UNKNOWN` (`AUD-15`) | 🟢 | XS | Un anime au statut inconnu ne s'affiche plus « Finished » |
| `US-ADD-DIRECT` | 🟠 | XS | Fin des ~10 s où « Added to On Air » ment sur For You |
| `US-SHOW-FALSE` (`AUD-10`) | 🟠 | S | Un anime présent avec un `day` renseigné cesse de disparaître de la semaine |
| `US-LOGO-INTERNAL` (`BM-09` partiel) | 🟢 | S | ~3 cartes visibles à l'ouverture de Library au lieu de 2,5 |
| `US-SYNOPSIS-VERSIONTOP` | 🟢 | S | Le synopsis apparaît en recherche — on sait ce qu'on ajoute |
| `US-MODAL-NEXTEP-HIERARCHY` | 🟢 | S | La modale hiérarchise prochain épisode / compteur / +1 |
| `US-REMOVE-DANGER` | 🟢 | XS | « Remove from list » cesse de ressembler à un lien anodin |
| `US-DARK-HEADER` (`B-04`) | 🟢 | XS | Fin du bandeau incohérent en mode sombre (logo à 1,47:1) |
| `US-ESLINT-CI-2` | 🟢 | M | Aucun — purge des violations de typage |
| `US-TOUCH-A`, `US-ONBOARD-FALLBACK` | — | — | Flex |

> 🔴 **S44 est sur-souscrit à 16+.** Ce n'est pas tenable. La recomposition sur retours bêta
> n'est pas une option de confort, c'est la seule sortie : elle donnera un critère objectif pour
> couper. **À trancher au PI Planning de clôture S44.**

**Chantiers nommés, sans slot :**
- **Dette de vérité** — « We'll sync it later » promet une re-synchronisation que rien dans le
  code ne garantit : elle dépend d'une sauvegarde ultérieure déclenchée par une autre action.
  Un testeur qui ne touche plus à rien garde ses choix en local indéfiniment. Même famille
  qu'`US-ADD-TOAST-TRUTH`. **À traiter avant que le message ne soit vu en masse.**
  🔻 **Aggravé par un constat SE-070 :** un anime *en cours de diffusion* sans horaire AniList
  atterrit en *Plan to Watch*. Le message est honnête, mais l'utilisateur voit une app qui range
  mal. C'est le premier retour bêta attendu.
- **Convergence `onboardingFilter`** (`DEC-178`) — la règle « en cours de diffusion » existe
  encore en double : `useAddAnime.resolveTargetState` et `utils/onboardingFilter.ts`.
- **Indicateur de sync** — le rattrapage progressif reste invisible.
- **Bouton « See my calendar (0) »** — trois sorties concurrentes sur un écran vide d'onboarding.
- **Titre tronqué sans recours** — « Smoking Behind the Supermarket… » sur This Season. Le titre
  complet n'est accessible nulle part depuis la grille.
- **`<style scoped>` sur `AppHeader.vue`** — viole `R-CODE-8`. Constaté SE-070.

**PI Planning complet à la clôture de S44. S45 → S50 recomposés à cette occasion.**

---

### S45 → S50 — thématiques, non composées

Volontairement laissés sans détail. `DEC-155` : la composition se fait à l'ouverture du sprint,
sur l'état réel du produit. Les intitulés du §1 sont des intentions, pas des engagements.

**`BENCHMARK.md`** attend un triage — utile seulement une fois les surfaces observables avec du
contenu réel. **Pas avant S45.**

---

## §3 — Règles de composition gravées

### 🔴 La ligne de partage `RecCard` / `AnimeCard`

**Ce n'est pas le composant, c'est *découverte vs bibliothèque*.**

| Composant | Écrans | Action de surface |
|---|---|---|
| **`RecCard`** | For You, This Season, **Coming Soon**, Library › Upcoming | Add (+ Skip sur For You seulement) |
| **`AnimeCard`** | Library › Plan to Watch, Library › Completed | Aucune |

> **Un bouton « Add » sur Completed n'a aucun sens.** L'anime y est déjà.
>
> ⛔ **`AnimeCard.vue` n'est pas destiné à disparaître.** `US-CARD-CONVERGE-B` migre **Coming Soon
> seul**. Ses deux autres consommateurs sont conformes à cette règle et doivent le rester.
>
> 🔻 **Correction SE-069, confirmée SE-070 :** `AUD-50` (« 3 consommateurs restants ») est
> **requalifié NON-CONSTAT** et `DEC-181` est **⛔ SUPERSEDED** — la question était déjà tranchée
> ici. Motif de l'erreur : `ROADMAP.md` n'a pas été ouvert avant rédaction de l'US.
> **Le parking et les règles gravées se lisent avant toute US de convergence.**

### Règle d'ajout — source unique

✅ **`US-ADD-EXTRACT` est LIVRÉE** (SE-068, `a2bf796`). La logique vit dans `useAddAnime.ts`
(`addToLibrary`, `resolveTargetState`). Toute page qui ajoute un anime l'**appelle**.

- ⛔ **Ne jamais réécrire une logique d'ajout simplifiée dans une page** — motif exact d'`AUD-03`.
  Deux occurrences soldées : `AnimeModal` (SE-068), `SearchInput` (SE-069, `AUD-47`).
- ⛔ **`DEC-172`** : toast sur l'état **appliqué**, sync sur l'état **demandé**. Ne pas unifier —
  la sync est ce qui repromeut une entrée démotée.
- ⛔ **`DEC-178`** : `resolveTargetState` reconnaît `Currently Airing` **et** `Continuing`.
  Doublon résiduel connu : `utils/onboardingFilter.ts`, à converger en S44.
- ⛔ **`DEC-180`** : `RecCard.showSkip` (défaut `true`). Masqué hors For You. `DEC-159` sans objet
  sur This Season.

### 🔴 Un chantier 100 % test ne passe pas par Gemini

`R7` : l'auteur du test ≠ l'auteur du code. Une US qui ne livre **que** des fichiers de test ne
peut donc pas être confiée à Gemini — il écrirait le test validant son propre travail, et il ne
peut pas l'exécuter (pas de navigateur Playwright). **Claude produit chaque spec verbatim, le PO
l'applique** (`DEC-128`). Précédents : patch `search-quick-add`, `US-DEMOCK-1a`, tous deux SE-070.
Conséquence à budgéter : ~1 spec par échange de conversation.

### Groupage

**Caduc pour S42** — `US-ADD-EXTRACT`, `US-CARD-CONVERGE-A` et `US-SEASON-FRESH` sont livrées.

**Reste valable :** `US-CARD-CONVERGE-B` et `US-MORELIKETHIS-FIX` touchent `RecCard.vue` /
`AnimeModal.vue`. À enchaîner dans la même session si les deux entrent dans un même sprint.

**Nouveau :** `US-DEMOCK-HELPER` doit précéder la migration de `search-enriched` — cette spec
asserte studio et année, que `MediaSeed` ne sait pas produire aujourd'hui.

### Blocages levés

- ✅ `US-STALE-SIGNAL` **n'est plus bloquée**. `DEC-151` soldé par `DEC-158` (a) : source unique
  = `WithMeta.stale` (`useAniListApi.ts`) ; `staleDataWarning` (`usePersistence.ts:18,192,305`)
  est supprimé, ainsi que `keepStaleData` / `clearStaleData` s'ils n'ont plus d'appelant.
- 🔻 **`AUD-49`** — `DECISIONS.md` porte deux entrées `DEC-158`. Renumérotation du doublon
  (« rechargement complet du navigateur ») en `DEC-176` à appliquer avant toute citation.

---

## §4 — État du chantier de fiabilité des tests

### ✅ Soldé en SE-070

**`DiscoverSeasonPage.vue` — dette de tokens et de gabarit.** Deux micro-patchs.
- `var(--text-color)` → `var(--text)` · `var(--danger-color, #dc3545)` → `var(--danger)` ·
  `var(--primary-color, #007bff)` → `var(--accent)`. Violation de `DEC-97` levée.
- 🔻 **Correction d'une affirmation SE-069 :** le titre « Current Season » n'était **pas** un
  risque d'illisibilité en thème sombre. Une `var()` non résolue sans fallback rend la
  déclaration invalide et `color` retombe sur l'héritage — `body` porte `var(--text)`, redéfini
  en `html.dark`. Le vrai défaut visible était le **bouton « Retry » en bleu Bootstrap**.
- **`AUD-53`** — `padding: 1.5rem` → `1.5rem 1rem 5rem`, aligné sur Coming Up et Explore.
  This Season était la seule page de découverte sans réserve basse : la dernière rangée de cartes
  passait sous la barre de navigation.

### 🔴 Ouvert — `AUD-52`

**17 specs mockent en direct**, contre 12 estimées. Trois familles :

| Famille | Nb | Nature | Destination |
|---|---|---|---|
| **A — AniList direct** | 5 | Le mock fonctionne, il double le helper | `US-DEMOCK-1a/b/c/d` — 1 migrée |
| **B — 🔴 mocks morts** | 11 | Routent `**/seasons/now**`, `**/anime/**`, `**/api.jikan.moe/**` — **des URL Jikan, abandonnées depuis S40.** Le mock ne matche plus rien | `US-DEMOCK-2` et `-3` |
| **C — catch-all** | 1 | `week-empty-day-cta` route `**/*` : bloque tout, fonctionne par accident | `US-DEMOCK-3` |

**Plus grave que le mock mort : le mock absent.** `modal-next-episode.spec.ts` ne figure dans
aucun `page.route`. Rouge en fin de SE-070. À instruire en priorité.

**Hors périmètre :** `boot-loader.spec.ts` route Firestore — ce n'est pas un fournisseur d'anime.

---

S43 : passer à ✅ CLOS — v0.37.0, 10/10. Réponse à la gate = gain ressenti (type 1). Sorties de périmètre : US-ESLINT-CI-1 ↦ S44 · AUD-54 ↦ S44 · US-DEMOCK-3 ↦ S44 flex. US non prévues entrées en cours : US-HEADER-TINT, US-DEMOCK-2c-3, correctif logout-modal-position.

S44 — « L'arrivée vaut le produit ». 🔴 Arbitrage PO à trancher en SE-073 : le Goal écrit porte sur l'onboarding et la bêta, mais le PO a demandé l'audit pixel (AUD-56) et la refonte du chrome mobile en priorité. Les deux ne tiennent pas dans 7 slots avec J10e, US-MORELIKETHIS-FIX et US-STALE-SIGNAL déjà planifiés. Le Goal doit être réécrit ou le contenu arbitré.

Déjà décidé pour S44 :

Refonte du chrome mobile = option A (rangée unique logo + recherche + menu ⋯) puis option B (icônes nues teintées). Validé par le PO en SE-072.
Audit pixel AUD-56 par parcours, pas écran par écran : 375 × 812, un thème à la fois, parcours ouvrir → ajouter → semaine → modale → déconnexion. Méthode proposée, à valider.
-----
## §5 — Note aux testeurs — à rédiger avant tout envoi

Document jamais écrit. Prérequis à la bêta, quelle que soit sa date.

**Doit porter au minimum :**
- **`AUD-37`** — « Combien de titres avais-tu sur MAL, combien en retrouves-tu ? » L'import MAL
  n'a **jamais** été vérifié sur un vrai fichier (décision PO : `observé en bêta`, pas `soldé`).
  Le test est délégué aux testeurs sur ~300 titres au lieu de 10, sans instrumentation.
- **Multi-appareil** — `AUD-42` était observé en vrai ; `US-ONBOARD-PERSIST-B` livre une bande de
  rattrapage. **Comportement réel non revérifié.** Si la bande ne suffit pas, l'annoncer.
- **Où remonter** — un canal unique. `DEC-157` : retour brut → qualification par Claude → slot
  flex ou `ROADMAP.md`. **Jamais de correctif au vol hors US.**

---

## §6 — Parking

### Reporté, avec raison

| Item | Sort |
|---|---|
| Chrome d'en-tête sur 5 bandes (`BM-09` complet) | S48+ — conflit arbitré par `US-TOUCH-A`. Seul le logo sort (S44) |
| Composition de la modale (`BM-10`) | S48 — sans données d'usage, y toucher serait de l'esthétique |
| Contrôles de modale rares (#7/#12/#15/#16 du benchmark Q2) | Nettoyage, sans sprint assigné |
| Vue Mois | ✅ Tranchée en avance (`DEC-163`). `US-MONTH-COMINGSOON` livrée en S41. Refonte complète à planifier — S48 ou sprint dédié. `MonthDayCell.vue` et les classes `month-*` conservés |
| `AUD-13` — `localStorage` piloté depuis `AppHeader.vue` | ✅ **Soldé** par `US-ONBOARD-PERSIST-A` (`b509ca0`). La clôture antérieure était fausse |
| Libellé « Dismiss » trompeur | Candidat US copie. Le mot ne distingue pas « je masque » de « je bannis ». Proposition : « Not this season » / « Not interested ». **Zéro logique, du texte.** ⚠️ Partiellement caduc : `DEC-180` masque « Skip » sur This Season |
| `AUD-51` — boutons du header à 40 px, pastille « Add » à 36 px | Sous la cible tactile de 44 px. Le volet header est **absorbé par `US-HEADER-ICONS`** (S43). La pastille reste un arbitrage PO assumé, à revoir sur constat de testeur |
| `AUD-40` / `AUD-41` | ~40 lignes de CSS mort, doublon `.card-cp-title` (l.933 et l.1939), `studios: ["8-bit","8-bit"]`, titre tronqué sans recours. Lot dette, sans sprint assigné |
| **La carte de saison n'a pas été dessinée, elle a été assemblée** | Ordre de lecture actuel : action → titre → info. L'utilisateur cherche : jaquette → titre → décider. Une refonte réelle mérite des retours de testeurs, pas un avis sur capture |
| **Zones vides de `RecCard` sur This Season** | Badge et signaux de recommandation n'existent que pour For You. Constat PO à faire à l'œil : la carte paraît-elle creuse ? |

### Refusé — ne pas remettre au backlog

| Item | Raison |
|---|---|
| Post-it à 44 px (benchmark Q2 #5) | Casse la grille du mois — trois post-its à 44 px font 132 px pour une cellule à 100 px |
| Compte à rebours à la minute | Q4 — l'utilisateur demande « quel jour », pas « dans combien de temps » |
| Filtres et tri sur Library / On Air | Q4 — filtrer 20 animes importe le coût d'interface d'un problème que le produit n'a pas |
| Synopsis, studio ou genres sur les cartes de liste | Q4 — transforme une liste de décision en catalogue à lire |
| Bandeau persistant en bas des vues calendrier | Q4 — retire ~10 % de surface utile sur 390 px |

### Règles gravées par le benchmark Q4

1. En vue Semaine, la taille de vignette est plafonnée par la contrainte **« 7 jours sans
   défilement »**. Jamais l'inverse.
2. Tant que la liste médiane d'un utilisateur tient en deux écrans, **aucun filtre**.
3. **Une carte = une décision.** Chaque métadonnée ajoutée entame le seul avantage qu'aucun
   concurrent ne peut reprendre.
