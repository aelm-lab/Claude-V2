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
> **Prochain PI complet : clôture S44**, avec 4 sprints de retours bêta réels.

---

## §1 — Le cap S41 → S50

| Sprint | 🎯 Sprint Goal | Confiance |
|---|---|---|
| **S41** | **Ce que j'ajoute reste** — la persistance cesse de mentir | ✅ **CLOS — v0.35.0, 10/10** |
| **S42** | **On peut nous faire confiance** — plus aucun écran vide ni muet | 🟢 Ferme |
| **S43** | **Rien ne casse en douce** — filets automatisés et sweep déterministe | 🟢 Ferme |
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

### S42 — « On peut nous faire confiance » · 7 planifiées + 3 flex

| US | Risque | Effort | Impact utilisateur |
|---|---|---|---|
| `US-MORELIKETHIS-FIX` (absorbe `AUD-16`) | 🔴 | S | « More like this » revit ; une relation ouvre une modale avec jaquette, score et synopsis au lieu du vide |
| `US-ONBOARD-TOAST` (`AUD-06`) | 🟠 | S | Sur `/welcome` et `/login`, un échec s'affiche au lieu de laisser un écran figé. Solde la spec rouge `onboarding-toast` (`AUD-23`) |
| `US-ONBOARD-EMPTY` | 🟠 | S | Pool de saison vide ou en échec : message + réessai au lieu d'une grille muette |
| `US-HEADER-ICONS` | 🟠 | M | Fin des emojis ☀️📅📥❓🚪 illisibles et changeants selon l'OS ; `aria-label` traduits en anglais |
| `US-STALE-SIGNAL` (`AUD-05`, DEC-158) | 🟠 | M | L'utilisateur voit quand ses données sont périmées au lieu de croire à un calendrier faux |
| `US-PERF-BASELINE` | 🟢 | M | Aucun direct — mesure boot, premier rendu, latence perçue sur mock déterministe. Chiffres → `STATE.md` |
| `US-PERF-GATE` | 🟠 | S | Un ralentissement futur devient une spec rouge, bloqué au merge. **Après la baseline, jamais avant** |
| `US-ADD-EXTRACT` | 🟢 | S | **Aucun — refactor pur.** La logique d'ajout sort d'`AnimeModal.vue` (`onAdd`, l.139-150 : 3 `newState` possibles, 3 toasts) vers un composable partagé. Prérequis strict d'`US-SEASON-1TAP` |
| `US-SEASON-1TAP` | 🟠 | S | This Season : 1 tap au lieu de 2 + modale. Skip session-only (DEC-159). **Après `US-ADD-EXTRACT`, jamais avant** |
| `US-FIRESTORE-LIMITS` (`AUD-32`) | 🔴 | M | Les sauvegardes cessent d'être refusées en silence ; la liste ne s'arrête plus à 100 titres. **Glissée de S41 — bloquant bêta, non soldé** |
| `US-CARD-CONVERGE-A` (`AUD-25`, `AUD-39`) | 🟠 | S | **P1.** This Season passe sur `RecCard` : Skip/Add en surface, 1 tap au lieu de 2 + modale. Absorbe l'asymétrie d'action et le double nommage. **Après `US-ADD-EXTRACT`** |
| `US-CARD-CONVERGE-B` (`AUD-25`) | 🟠 | S | **P1.** Coming Soon passe sur `RecCard`. Même geste que CONVERGE-A |
| `US-ONBOARD-PERSIST-B` (`AUD-42`) | 🔴 | M | Le multi-appareil cesse d'être dégradé ; un appareil neuf ne relance plus une sync complète au premier lancement |
| `US-TOUCH-A` | 🟢 | S | Glissée de S41. 6 contrôles deviennent tapables — zéro pixel visuel changé |
| `US-TOUCH-B` | 🟢 | S | Glissée de S41. 24 à 48 cibles remontées à 44 px sur les écrans de tri |
| `US-SWEEP-S41` | 🟢 | S | **Premier geste de S42.** Rejeu du sweep E2E, non armé depuis SE-063 — 4 sessions, 10 US |

Groupage obligatoire S42 : US-ADD-EXTRACT → US-CARD-CONVERGE-A → US-CARD-CONVERGE-B → US-MORELIKETHIS-FIX. Tous touchent AnimeModal.vue ou RecCard.vue. À enchaîner dans la même session, dans cet ordre.

⛔ Ne jamais réécrire une logique d'ajout simplifiée dans une page — motif exact d'AUD-03.

🔴 La ligne de partage n'est pas le composant, c'est découverte vs bibliothèque. RecCard (avec Skip/Add) : For You, This Season, Coming Soon, Library › Upcoming. AnimeCard (sans) : Library › Plan to Watch, Library › Completed. Un bouton Add sur Completed n'a aucun sens.

Groupage S42 : US-ADD-EXTRACT, US-SEASON-1TAP et US-MORELIKETHIS-FIX touchent
AnimeModal.vue. À enchaîner dans la même session, dans cet ordre.
⛔ Ne jamais réécrire une logique d'ajout simplifiée dans une page — c'est le motif
exact d'AUD-03.

### S43 — « Rien ne casse en douce » · 7 planifiées + 3 flex

| US | Risque | Effort | Impact utilisateur |
|---|---|---|---|
| `US-DEMOCK-1/2/3` (`AUD-24`, 3 slices de 4 specs) | 🟢 | M ×3 | Aucun — le sweep devient déterministe. Absorbe le résidu `onboarding-seed.spec.ts:56` et la fragilité des 2 clés de seed (`AUD-26`) |
| `US-ESLINT-CI-1` (config + première exécution) | 🟢 | S | Aucun — « zéro `any` » devient vérifié au lieu de déclaré |
| `US-ESLINT-CI-2` (purge des violations) | 🟢 | M | Aucun — filet permanent contre les régressions de typage |
| `US-REMOVE-DANGER` | 🟢 | XS | « Remove from list » cesse de ressembler à un lien anodin (il fait 2× la cible du « +1 ») |
| `US-DARK-HEADER` (`B-04`) | 🟢 | XS | Fin du bandeau incohérent en mode sombre (logo à 1,47:1) |

### S44 — « L'arrivée vaut le produit » · 7 planifiées + 3 flex

| US | Risque | Effort | Impact utilisateur |
|---|---|---|---|
| `J10e-a/b/c` (repli orphelins MAL titre+année, DEC-145) | 🟠 | M ×3 | Les animes sans `idMal` cessent de disparaître silencieusement |
| `US-STATUS-UNKNOWN` (`AUD-15`) | 🟢 | XS | Un anime au statut inconnu ne s'affiche plus « Finished » |
| `US-ADD-DIRECT` | 🟠 | XS | Fin des ~10 s où « Added to On Air » ment sur For You |
| `US-SHOW-FALSE` (`AUD-10`, 1 cas sur 4) | 🟠 | S | Un anime présent avec un `day` renseigné cesse de disparaître de la semaine |
| `US-LOGO-INTERNAL` (`BM-09` partiel) | 🟢 | S | Une ligne de chrome rendue : ~3 cartes visibles à l'ouverture de Library au lieu de 2,5 |
| `US-SYNOPSIS-VERSIONTOP` | 🟢 | S | Le synopsis apparaît en recherche — on sait ce qu'on ajoute |
| `US-MODAL-NEXTEP-HIERARCHY` | 🟢 | S | La modale hiérarchise prochain épisode / compteur / +1 |

> **PI Planning complet à la clôture de S44.** S45→S50 sont recomposés à cette occasion.

---

## §3 — Parking

### Reporté, avec raison

| Item | Sort |
|---|---|
| Chrome d'en-tête sur 5 bandes (`BM-09` complet) | S48+ — conflit arbitré par `US-TOUCH-A`. Seul le logo sort (S44) |
| Composition de la modale (`BM-10`) | S48 — sans données d'usage, y toucher serait de l'esthétique |
| Contrôles de modale rares (`#7/#12/#15/#16` du benchmark Q2) | Nettoyage, sans sprint assigné |
| `AUD-13` — `localStorage` piloté depuis `AppHeader.vue` | P3 dette, 1 occurrence. Sera probablement absorbé par `US-PERSIST-P0` |
| **Vue Mois** | ✅ **Tranchée en avance (DEC-163).** Elle existe, mais pas encore : `US-MONTH-COMINGSOON` livrée en S41. Refonte complète à planifier — S48 polish ou sprint dédié. `MonthDayCell.vue` et les classes `month-*` conservés |
| `AUD-13` — `localStorage` piloté depuis `AppHeader.vue` | ✅ **Soldé** par `US-ONBOARD-PERSIST-A` (`b509ca0`). La clôture antérieure était fausse |
| **Libellé « Dismiss » trompeur** | Candidat US copie S42. Le mot ne distingue pas « je masque » de « je bannis », même après correction d'`AUD-38`. Proposition : « Not this season » sur This Season, « Not interested » sur For You. Zéro logique, du texte |
### Refusé — ne pas remettre au backlog

| Item | Raison |
|---|---|
| Post-it à 44 px (benchmark Q2 #5) | Casse la grille du mois — trois post-its à 44 px font 132 px pour une cellule à 100 px |
| Compte à rebours à la minute | Q4 — l'utilisateur demande « quel jour », pas « dans combien de temps » |
| Filtres et tri sur Library / On Air | Q4 — filtrer 20 animes importe le coût d'interface d'un problème que le produit n'a pas |
| Synopsis, studio ou genres sur les cartes de liste | Q4 — transforme une liste de décision en catalogue à lire |
| Bandeau persistant en bas des vues calendrier | Q4 — retire ~10 % de surface utile sur 390 px |

### Règles gravées par le benchmark Q4

- En vue Semaine, la taille de vignette est **plafonnée par la contrainte « 7 jours sans défilement »**. Jamais l'inverse.
- Tant que la liste médiane d'un utilisateur tient en deux écrans, **aucun filtre**.
- Une carte = une décision. Chaque métadonnée ajoutée entame le seul avantage qu'aucun concurrent ne peut reprendre.
