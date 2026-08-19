# BENCHMARK.md — Benchmark concurrentiel, trié

> **Rôle :** conserver le tri du benchmark pour que la discussion reprenne sans re-uploader le rapport. **Aucune US n'est ouverte depuis ce document sans arbitrage PO.**
> **Pas ici :** l'état courant et le Kanban (→ `STATE.md`), les décisions (→ `DECISIONS.md`).

**🔻 Satellite hors ordre de lecture.** Source : mesures du 10 août 2026, viewport 390 × 844, production `aanime-v2.ai.studio`, contre anichart.net · livechart.me · animeschedule.net. **Gel PO levé — triage à mener.**

---

## 1. Portée réelle du rapport

Le benchmark a été mené pendant une panne de la source de données de l'époque. On a longtemps cru la pile de constats inexploitables largement plus large qu'elle ne l'est : le rapport a surtout mesuré du **CSS, du DOM et du timing de boot** — trois choses qui n'appellent aucune API. La panne a détruit la partie *contenu*, pas la *coquille*.

**Résultat : 10 écarts sur 12 sont exploitables en l'état.**

---

## 2. Nos forces confirmées par l'extérieur

Le produit n'est pas en retard, il est **mal encaissé**.

- **La vue semaine est unique sur ce marché.** 7 jours dans 844 px sans défiler. Aucun des trois concurrents ne propose de vue semaine. C'est la seule différence structurelle qui justifie l'existence d'Aanime, et elle est réussie.
- **La conversion de fuseau est exacte à la minute** (22:00 JST → 15:00 CEST, vérifié). Cœur de métier, correct — mais jamais affiché, donc jamais crédité.
- **La recherche sait échouer proprement** (« Search unavailable right now »). Ce savoir-faire existe en production, il n'a simplement pas été répliqué ailleurs.
- **La microcopie de l'onboarding est meilleure que celle des trois concurrents** : le libellé du bouton est un compteur vivant (« Pick 3 genres » → « 2 more » → « See my calendar (1) »).

---

## 3. Corrections de faits

Ces trois lignes évitent du travail inutile. Elles sont acquises, indépendamment de la suite.

| # | Croyance interne | Fait mesuré |
|---|---|---|
| **F1** | « Page blanche de 6 secondes » | **2,5 s en production.** Les 8 s venaient du serveur Vite de dev hébergé à Singapour, servant 59 fichiers un par un. Bundle prêt à **152 ms** → **aucun chantier de réduction de bundle** (DEC-133) |
| **F2** | « La sous-nav Week/Month a un problème de contraste » | **Faux.** Elle est à **14,82:1**, l'un des meilleurs de l'app. Le vrai fautif est `.app-header`, qui garde `background: rgba(255,255,255,0.65)` en mode sombre → logo à **1,47:1** |
| **F3** | *(jamais vu)* | Le CTA « Explore this season » échoue en mode **clair** : **3,33:1**. Pas en sombre |

---

## 4. Pile A — 10 écarts exploitables

Classés par ratio impact / effort décroissant.

| ID | Écart | Impact utilisateur concret | Effort |
|---|---|---|---|
| **B-04** | `.app-header` jamais rethémé | En mode sombre, une barre gris clair reste collée en haut avec un logo illisible. **L'app a l'air cassée** | petit |
| **B-12** | Stats en français dans une app anglaise | Il lit « épisodes regardés », « 1 séries », « Terminées (all-time) ». 2 `aria-label` en français. Viole le vocabulaire figé (`CLAUDE.md §4`) | très petit |
| **B-05** | Fuseau jamais nommé | La carte dit « 15:00 ». Rien ne dit que c'est **son** heure. Calcul juste, confiance perdue pour rien | petit |
| **B-10** | Modale sans `role=dialog`, focus jamais pris, scroll de fond libre | Au clavier, il doit tabuler à travers toute la page pour atteindre « +1 ». 3 boutons sur 4 ont déjà leur libellé | petit |
| **B-11** | Aucun manifeste PWA | Pas d'icône sur l'écran d'accueil. **Le service worker est déjà actif** : la complexité est payée, le bénéfice pas encaissé | petit |
| **B-06** | Pas de compte à rebours | Il voit « 15:00 » et calcule lui-même. C'est le seul écran qui change tout seul — le moteur du retour. **Quasi gratuit désormais** : AniList sert `nextAiringEpisode.timeUntilAiring` | petit |
| **B-03** | 2,5 s d'écran vide, dont **910 ms mortes** | Rectangle crème, aucun signe de vie. Bundle prêt à 152 ms, chunk de route lancé à 1549 ms | petit (squelette) + moyen (parallélisation auth) |
| **B-07** | **0 / 13 contrôles ≥ 44 × 44 px** | Il rate ses appuis. Sous-onglets à 26 px, flèches à 28 × 26. Contradiction frontale avec la promesse mobile-first | petit |
| **B-09** | 0 titre, 0 rôle ARIA, **0 lien `<a>`** | Muet au lecteur d'écran. Et le zéro `<a>` a un coût produit : **impossible d'ouvrir une série dans un onglet, de la partager ou de la mettre en favori** | moyen, étalable |
| **B-08** | Vue mois : titre invisible, scroll horizontal | 7 colonnes dans 390 px = 55 px chacune. L'épisode se réduit à une pastille « 15:00 » sans titre. **Décision produit d'abord** | moyen |
| *(B-02)* | Pas d'état d'erreur calendrier / Library / Discover | Écran vide indistinguable de « tu ne suis rien ». **Corrobore `AUD-05`** | moyen |
| *(B-01)* | Mur d'inscription — `/` → `/login` | Aucun contenu avant compte. Les 3 concurrents laissent suivre une série en 2 taps sans inscription | **gros** — voir §6 |

---

## 5. Pile B — à remesurer

Constats jamais observés avec du contenu réel. L'annexe du rapport l'admet.

| Constat | Sort |
|---|---|
| Hiérarchie typo / densité de Library, Discover, calendrier chargé | **Remesurer** |
| Densité d'information vs concurrents (mesurée avec 1 série à l'écran) | **Remesurer** |
| Validation de la conversion horaire | **Revalider** sur le nouveau chemin `airingAt` (timestamp UTC, mécanisme différent de l'ancien) |
| « Discover : écran vide » | **Jeter** — symptôme de la panne, pas un défaut de conception |
| « Episode 7 / **?** » | **Jeter** — le total d'épisodes est désormais fourni |

---

## 6. Le désaccord de fond — le mode invité

Le rapport recommande le mode invité **en premier**, effort « gros ». **Claude s'y oppose**, et c'est le seul vrai désaccord du tri.

1. **`AUD-02`** : `saveSchedule` rattrape son propre throw. L'app affiche « Saved » pendant une panne Firestore **et la donnée est perdue**. Construire une migration invité → Firestore par-dessus une couche de persistance qui ment déjà, c'est fabriquer une machine à perdre des onboardings entiers.
2. **`AUD-07`** : le flag d'onboarding vit en `localStorage` seul et saute au logout. Le mode invité repose sur ce même stockage et amplifie le défaut.
3. **Aucune baseline.** Faire le chantier le plus gros et le plus risqué du backlog pour améliorer une métrique non instrumentée, c'est répéter au prix fort l'erreur des quatre derniers sprints.

**Position retenue :** le mode invité est un **Sprint Goal à lui seul**, pas une US. Prérequis bloquants : `US-TTFA-INSTRUMENT` livrée **et** `AUD-02` résolue.

---

## 7. Ce que le rapport dit de NE PAS copier — validé

Ces garde-fous protègent la North Star et doivent survivre à toute reprise de discussion.

- **Pas de catalogue de saison à défilement infini.** LiveChart empile 109 cartes sur 33 338 px. Aanime est un calendrier personnel : la bonne unité est le jour. « This Season » et « Coming Soon » restent des **sources d'ajout**, pas des destinations.
- **Pas la profondeur de réglages d'AnimeSchedule.** Chaque réglage est un écran de plus entre l'arrivée et la première série suivie — l'inverse exact du TTFA. **Exception retenue :** le début de semaine lundi/dimanche, une ligne, une incompréhension culturelle réelle évitée.
- **Pas de pile publicitaire.** Le consentement TCF d'AniChart (668 partenaires) coûte 6 340 ms à la première visite.
- **Pas de compte à rebours à la seconde.** 20 `setInterval` sur une vue semaine = batterie et saccades. Granularité jours/heures (minutes sous 24 h) : 95 % de la valeur perçue.
- **Pas de sélecteur de densité.** Choisir une bonne densité et l'assumer.

---

## 8. Remesures à faire — ~20 minutes

**Ne pas rejouer un benchmark complet.** Quatre observations suffisent, calendrier chargé :

1. Library avec ≥ 10 séries — hiérarchie typographique et densité
2. Discover avec du contenu — le « pourquoi » cliquable
3. Calendrier semaine avec ≥ 5 séries — densité réelle vs les 2 séries visibles des concurrents
4. Conversion horaire sur le chemin `airingAt` — revalider 22:00 JST → heure locale

---

## 9. Décisions ouvertes

| # | Question | Statut |
|---|---|---|
| **D1** | Vue mois (B-08) : liste verticale par jour, ou retrait de la barre mobile ? Le rapport a raison sur un point : **ne pas la laisser dans cet état intermédiaire** | Ouverte |
| **D2** | Accessibilité (B-09) : règle continue dans `AGENTS.md` (« toute US touchant un composant y pose ses titres et ses `alt` ») plutôt qu'une US dédiée ? | Ouverte — **reco Claude : oui** |
| **D3** | Étalement des candidats pile A sur les sprints à venir | Ouverte |
