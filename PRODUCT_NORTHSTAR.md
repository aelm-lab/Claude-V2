# PRODUCT_NORTHSTAR.md — Ancrage produit Aanime

> **Rôle :** rééquilibrer le système vers le PRODUIT. À placer dans la Knowledge, à lire au Sprint Planning.
> **Pourquoi ce doc existe :** jusqu'ici la seule métrique au mur était le vert CI (tests/build). Le vert prouve la mécanique, pas la valeur. Ce doc donne au PO un cadran produit en face du cadran qualité.
> **État de référence : intégré au cleaning pré-S35.**

---

## 1. North Star

> **« Un nouvel utilisateur suit son premier anime sur son calendrier en moins de 2 minutes, et revient ≥ 2 jours la première semaine. »**

Tout sprint se demande : *est-ce que ce qu'on livre rapproche ou éloigne de cette phrase ?*
Si la réponse est « ni l'un ni l'autre » (dette pure), le sprint doit le déclarer explicitement.

---

## 2. Les 3 métriques produit (le cadran d'en face)

À mesurer même de façon artisanale (vous êtes l'utilisateur test ; à terme, instrumentation légère ou observation).

| Métrique | Question à laquelle elle répond | Cible directionnelle |
|---|---|---|
| **TTFA** — Time To First Anime | Combien de temps/clics pour suivre son 1er anime depuis une session vierge ? | ↓ (viser < 2 min, < 5 clics) |
| **Adds/semaine** | L'app crée-t-elle l'habitude d'ajouter des séries ? | ↑ |
| **Jours-retour S1** | Sur la 1ère semaine, combien de jours distincts l'utilisateur revient ? | ↑ (viser ≥ 2/7) |

> ⚠️ **État initial : aucune donnée collectée à ce jour.** Ces 3 métriques ne sont pas
> encore instrumentées ni observées — baseline à zéro. Elles n'ont de sens qu'à partir du
> moment où un sprint produit (typiquement EPIC 9 Onboarding) les rend mesurables.
>
> Règle : ces 3 chiffres apparaissent dans `STATE.md` §Métriques **à côté** du compteur de tests, jamais à la place.

---

## 3. Sprint Outcome Gate (nouvelle cérémonie de clôture)

S'ajoute à la porte technique (R1→R6), ne la remplace pas. **S'ajoute aussi aux règles de
clôture de session déjà actées dans `METHODOLOGY.md`** (mise à jour `STATE.md` + double
checklist Gemini/State) — troisième volet, pas un remplacement.

**Un sprint ne se clôt pas sans répondre, en une ligne, à :**

> *« Qu'est-ce que l'utilisateur peut faire / voir / ressentir aujourd'hui qu'il ne pouvait pas avant ce sprint ? »*

Trois réponses possibles :
1. **Gain ressenti** (ex. « il peut s'onboarder en 90 s ») → ✅ sprint produit, nominal.
2. **Gain de fiabilité visible** (ex. « il ne perd plus ses données en silence ») → ✅ acceptable.
3. **Aucun gain visible — dette/audit** → ⚠️ **autorisé en exception, pas en routine.** Doit être justifié (« filet avant correctif », « risque silencieux ») ET suivi d'un sprint produit.

**Garde-fou anti-dérive :** pas plus d'**1 sprint « aucun gain visible » consécutif**. Deux d'affilée = signal d'alarme, on bascule sur un levier produit.

---

## 4. Budget dette (plafond)

Pour éviter que la dette/l'audit ne recolonise le débit :

> **Ratio cible par sprint : ≤ 1 US de dette pure pour 1 US à gain visible.**

La dette invisible est réelle et doit être traitée — mais sous plafond, jamais en flux libre. Un audit qui génère plusieurs US de dette se **étale** sur plusieurs sprints, il ne se déverse pas en un seul.

> ⚠️ **Tension immédiate à trancher avant S35** : le backlog prêt à démarrer contient
> `US-MODAL-CENTER-AUDIT`, `US-E2E-BATCH-AUDIT` et `US-JIKAN-HEALTHCHECK` — trois items
> sans gain utilisateur visible — contre `US-SEARCH-3` comme seul item à gain visible prêt.
> Appliqué strictement, ce budget imposerait soit d'étaler la dette sur 2-3 sprints, soit de
> faire remonter un item produit supplémentaire . **Décision PO requise
> à la prochaine Sprint Planning, pas tranchée dans ce document.**

---

## 5. Re-priorisation — ⚠️ À RÉÉVALUER CONTRE LE BACKLOG RÉEL (voir note ci-dessus)

> Cette section a été rédigée avant le cleaning de doc et référençait un sprint figé (« S22 »)
> — retiré. Le principe reste valable , mais
> la table ci-dessous ne reflète plus le backlog actuel (`STATE.md`) et ne doit pas être
> appliquée telle quelle sans repasser par une Sprint Planning explicite.


| Proposition d'origine | Après (si retenu) | Raison |
|---|---|---|
| Dette/polish (CSS, audits) | Glissés en remplissage *après* le gain produit, dans la limite du budget dette (§4) | Polish ≠ valeur tant que personne n'arrive jusque-là |

US-140 confirmée livrée (S30-S34, découverte tardive au cleaning) — gap résiduel : toast de
bienvenue (US-140d, backlog immédiat). N'est plus le levier "jamais shippé" ; EPIC 9 passe
en polish, pas en démarrage.

Chaque lot est démontrable au PO (audit live R6), donc chacun fait avancer la North Star.

**Question ouverte pour la Sprint Planning S35** : on ouvre S35 Audit US 140  puis `US-MODAL-CENTER-AUDIT`/`US-SEARCH-3` (déjà
raffinés, prêts à specer) 
