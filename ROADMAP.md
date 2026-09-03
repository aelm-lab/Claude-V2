# ROADMAP.md — Cap produit multi-sprints

> **Rôle :** le cap **au-delà** du sprint courant, les règles de composition gravées, et le
> parking. Créé par `DEC-156`.
> **Pas ici :** le sprint en cours (→ `STATE.md §Kanban`) · le passé (→ `HISTORIQUE.md`) · le
> contenu des constats (→ `AUDIT.md`) · les règles de process (→ `PILOTAGE.md`).
>
> **Régénéré intégralement à la clôture de chaque sprint** (`DEC-190`), **entier ou pas du tout** :
> une régénération partielle est une amputation (la version SE-069 avait perdu ses §3 à §6).
> **PI Planning complet toutes les 4 clôtures — prochain : clôture S44.**

---

## §1 — Le cap S41 → S50

| Sprint | 🎯 Sprint Goal | État |
|---|---|---|
| **S41** | **Ce que j'ajoute reste** — la persistance cesse de mentir | ✅ CLOS — v0.35.0, 10/10 |
| **S42** | **On peut nous faire confiance** — plus aucun écran vide ni muet | ✅ CLOS — v0.36.0, 10/10 |
| **S43** | **Rien ne casse en douce** — filets automatisés et sweep déterministe | ✅ CLOS — v0.37.0, 10/10 |
| **S44** | **L'arrivée vaut le produit** — onboarding et premiers retours bêta | 🔄 **EN COURS** — bêta ouverte en SE-074 |
| **S45** | Consolidation bêta 1 — le top des irritants remontés | 🟠 Thématique |
| **S46** | Stats enrichies (EPIC 11) — tendances, historique de visionnage | 🟠 Thématique |
| **S47** | Notifications (EPIC 11) — « ton épisode sort ce soir » | 🟠 Thématique |
| **S48** | Composition modale & polish — sur données d'usage, pas sur esthétique | 🟠 Thématique |
| **S49** | Durcissement pré-lancement — bundle, offline, edge cases sync | 🟠 Thématique |
| **S50** | 🚀 Lancement public | 🟠 Thématique |

> **Ce tableau est un cap, pas un contrat.** À partir de S45, la composition appartient aux
> **retours bêta**, pas à cette page. `DEC-155` : la composition se fait à l'ouverture du sprint,
> sur l'état réel du produit.
>
> 🔴 **S44 était sur-souscrit à 16+.** La recomposition sur retours bêta n'est pas une option de
> confort : c'est la seule sortie, et elle donne enfin un critère objectif pour couper.
>
> **`BENCHMARK.md` attend un triage** — utile seulement une fois les surfaces observables avec du
> contenu réel. **Pas avant S45**, en même temps que l'audit pixel `AUD-56`.

---

## §2 — Règles de composition gravées

### 🔴 La ligne de partage `RecCard` / `AnimeCard`

**Ce n'est pas le composant, c'est *découverte vs bibliothèque*.**

| Composant | Écrans | Action de surface |
|---|---|---|
| **`RecCard`** | For You, This Season, **Coming Soon**, Library › Upcoming | Add (+ Skip sur For You seulement) |
| **`AnimeCard`** | Library › Plan to Watch, Library › Completed | Aucune |

> **Un bouton « Add » sur Completed n'a aucun sens.** L'anime y est déjà.
> ⛔ **`AnimeCard.vue` n'est pas destiné à disparaître.** `US-CARD-CONVERGE-B` migre **Coming Soon
> seul** ; ses deux autres consommateurs sont conformes et doivent le rester.
> 🔻 `AUD-50` (« 3 consommateurs restants ») est **requalifié non-constat** : la question était
> déjà tranchée ici. Motif de l'erreur : ce document n'avait pas été ouvert avant rédaction de
> l'US. **Le parking et les règles gravées se lisent avant toute US de convergence.**

### Règle d'ajout — source unique

✅ **`US-ADD-EXTRACT` est livrée** (`a2bf796`). La logique vit dans `useAddAnime.ts`
(`addToLibrary`, `resolveTargetState`). Toute page qui ajoute un anime l'**appelle**.

- ⛔ **Ne jamais réécrire une logique d'ajout simplifiée dans une page** — motif exact d'`AUD-03`. Deux occurrences soldées : `AnimeModal` (SE-068), `SearchInput` (SE-069).
- ⛔ `DEC-172` : toast sur l'état **appliqué**, sync sur l'état **demandé**. Ne pas unifier.
- ⛔ `DEC-178` : `resolveTargetState` reconnaît `Currently Airing` **et** `Continuing`. Doublon résiduel : `utils/onboardingFilter.ts` (`AUD-03`).
- ⛔ `DEC-180` : `RecCard.showSkip` (défaut `true`), masqué hors For You.

### 🔴 Un chantier 100 % test ne passe pas par Gemini

`R7` : l'auteur du test ≠ l'auteur du code. Une US qui ne livre **que** des fichiers de test ne
peut donc pas lui être confiée — il écrirait le test validant son propre travail, et il ne peut
pas l'exécuter (pas de navigateur Playwright). **Claude produit chaque spec verbatim, le PO
l'applique** (`DEC-128`). Conséquence à budgéter : ~1 spec par échange de conversation.

### Groupage

- `US-CARD-CONVERGE-B` touche `RecCard.vue` / `AnimeModal.vue` : à enchaîner avec toute autre US sur ces fichiers dans la même session.
- Toute extension du helper de mock (`MediaSeed`) précède la migration des specs qui en dépendent.

---

## §3 — Parking

### Reporté, avec raison

| Item | Sort |
|---|---|
| **Chantiers nommés sans slot (issus de S44)** | **Dette de vérité** « We'll sync it later » → devenu `AUD-59` · **Convergence `onboardingFilter`** (`DEC-178`) → `AUD-03` · **Indicateur de sync** : le rattrapage progressif reste invisible · **Bouton « See my calendar (0) »** : trois sorties concurrentes sur un écran vide d'onboarding · **Titre tronqué sans recours** sur This Season, le titre complet n'est accessible nulle part depuis la grille · **`<style scoped>` sur `AppHeader.vue`**, viole `R-CODE-8` |
| `J10e` — repli des orphelins par titre + année | 🔻 **GELÉE** (SE-074). Deux motifs indépendants : sa spécification n'existe dans aucun document (`DEC-145` annonce 3 slices sans contenu), et la fréquence du problème n'a jamais été mesurée. **La bêta est l'instrument de mesure : `J10e` remonte au premier signalement testeur.** Lecture faite : le rejet des orphelins est à **5 endroits** (`normalizeAniList.ts:42` · `useAniListApi.ts:232, 265, 331, 447`) et `mal_id` **et** `id` valent tous deux `idMal` — ce n'est pas un socle de 3 fichiers, c'est une refonte d'identité |
| Chrome mobile **option A** (rangée unique logo + recherche + menu ⋯) | 🔻 **REPORTÉE** (SE-074). Le menu ⋯ casse **deux** specs : `header-icons` (compte exactement 5 `.header-btn` visibles à 44 px, ordre des `aria-label` figé) et `logout-modal-position` (clique `header button[aria-label="Log out"]`). Faisable, mais ce n'est plus une US de CSS. **Option B livrée ; A rediscutée sur données d'usage bêta** |
| Refonte de la vue Mois | `DEC-163` tranchée en avance — « Coming Soon » assumé, `MonthDayCell.vue` conservé. S48 ou sprint dédié |
| Chrome d'en-tête sur 5 bandes · Composition de la modale · Contrôles de modale rares | S48+ — sans données d'usage, y toucher serait de l'esthétique |
| Libellé « Dismiss » trompeur | Le mot ne distingue pas « je masque » de « je bannis ». Proposition : « Not this season » / « Not interested ». **Zéro logique, du texte.** ⚠️ Partiellement caduc depuis `DEC-180` |
| **La carte de saison n'a pas été dessinée, elle a été assemblée** | Ordre de lecture actuel : action → titre → info. L'utilisateur cherche : jaquette → titre → décider. Une refonte réelle mérite des retours de testeurs, pas un avis sur capture |
| **Zones vides de `RecCard` sur This Season** | Badge et signaux n'existent que pour For You. Constat PO à faire à l'œil : la carte paraît-elle creuse ? |

### Refusé — ne pas remettre au backlog

| Item | Raison |
|---|---|
| Post-it à 44 px en vue Mois | Casse la grille — trois post-its à 44 px font 132 px pour une cellule à 100 px |
| Compte à rebours à la minute | L'utilisateur demande « quel jour », pas « dans combien de temps » |
| Filtres et tri sur Library / On Air | Filtrer 20 animes importe le coût d'interface d'un problème que le produit n'a pas |
| Synopsis, studio ou genres sur les cartes de liste | Transforme une liste de décision en catalogue à lire |
| Bandeau persistant en bas des vues calendrier | Retire ~10 % de surface utile sur 390 px |

### Règles gravées par le benchmark

1. En vue Semaine, la taille de vignette est plafonnée par la contrainte **« 7 jours sans défilement »**. Jamais l'inverse.
2. Tant que la liste médiane d'un utilisateur tient en deux écrans, **aucun filtre**.
3. **Une carte = une décision.** Chaque métadonnée ajoutée entame le seul avantage qu'aucun concurrent ne peut reprendre.
