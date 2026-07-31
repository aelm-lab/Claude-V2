# PROCESS_TIERS.md — Process étagé par classe de risque

> **Rôle :** garder la rigueur là où elle a attrapé de vrais bugs, l'alléger là où elle ne fait que taxer le débit.
> **Principe :** le gabarit « boot/persistance » (où vivent les bugs P0 silencieux) **ne doit pas** s'appliquer tel quel à une US CSS de 44px. La gate suit le risque, pas un réflexe uniforme.
> Référence des règles : `AGENTS.md` (R1→R6, R-CODE, R-SCOPE). Ce doc ne les ré-écrit pas, il dit **lesquelles s'appliquent à quoi**.
> **État de référence : intégré au cleaning pré-S35.**

---

## 1. La matrice

| Classe de risque | Tags concernés (voir §1-bis) | Gate exigée | Justification |
|---|---|---|---|
| **🔴 CRITIQUE** | EPIC 8 (Boot/Persist/Sync), EPIC 10 (Rec Engine) + toute US touchant l'orchestration / le store / le câblage de composables | **Gate complète** : R1 (triple preuve) + R2 (test runtime) + R4 (E2E DOM si écran) + R6 (audit live avant clôture epic) + zéro-confiance code brut | C'est ici que sont nés les 4 bugs s6 et le P0 s16. La rigueur est *méritée*. |
| **🟠 STANDARD** | `[FEATURE]` `[UX]` sur un composant existant, `[REC]` hors engine | R1 (triple preuve) + R4 (E2E ciblé) | Touche l'écran → un clic réel doit valider. |
| **🟢 LÉGÈRE** | `[CSS]` `[A11Y]` pur, `[UX-copy]`, libellés, styles dans `style.css`, suppression de code mort prouvée | **R1-allégée : type-check + test:run + build, les 3 verts (1 run groupé).** Pas d'E2E imposé — **sauf si un `v-if` interactif est ajouté, alors R4-bis obligatoire.** | Une couleur ou un padding ne casse pas l'orchestration runtime. Le coût E2E n'a pas de ROI ici. *(Correction cleaning : `test:run` reste obligatoire même en 🟢 — seul l'E2E rouge/vert saute. Sauter les tests unitaires existants, déjà rapides, n'a aucune justification et contredirait le zéro-confiance du projet.)* |

## 1-bis. Mapping tags → EPICs (aligné sur `EPICS.md` / `METHODOLOGY.md`)

Les zones "à risque" ne sont pas une taxonomie séparée — elles pointent vers les EPICs déjà
existants, utilisées comme `[SECTION]` dans le tag `US-XXX [EPIC][SECTION][TYPE]` :
- `[BOOT]` `[PERSIST]` `[SYNC]` → sous-zones d'**EPIC 8 — Boot & Démarrage**.
- `[REC][ENGINE]` → **EPIC 10 — Moteur de Recommandation**, spécifiquement `recEngine`/`useRecommendations` (pas le reste d'EPIC 10, ex. UI des RecCards, qui reste 🟠).

---

## 2. Règle de classement (sans ambiguïté)

1. **Une US hérite de la classe la plus haute qu'elle touche.** Une US « CSS » qui modifie aussi `usePersistence` est 🔴, pas 🟢.
2. **En cas de doute → classe supérieure.** Le doute coûte moins cher que le bug silencieux.
3. **Le tag `[DETTE]` ne baisse jamais la classe.** Une dette sur le boot reste 🔴.

---

## 3. Ce qui ne change JAMAIS (quelle que soit la classe)

Ces invariants sont la valeur dure du système — ils restent universels :
- **Zéro `any`** (R-CODE-1).
- **Périmètre = fichiers listés** (R-SCOPE-1), démarrage session = état des fichiers modifiés.
- **Sortie de commande = terminal brut** (jamais de paraphrase).
- **Contenu intégral des fichiers livrés.**
- **Impact utilisateur + reco** sur chaque US (PO non-technique).
- **`npm run test:run` reste vert, quelle que soit la classe.** Seule la partie E2E se module.

La gate s'allège ; l'honnêteté des preuves et la discipline de périmètre, non.

---

## 4. Effet attendu

| Avant | Après |
|---|---|
| Chaque US paie la taxe complète (triple preuve + rouge/vert E2E + audit live) | Seules les US 🔴/🟠 la paient ; les 🟢 passent en gate légère |
| US CSS bloquée derrière une suite E2E par lots `--workers=1` | US CSS livrée et mergée le jour même |
| Débit produit étranglé par le coût de cérémonie | Plus de sprints disponibles pour le gain visible |

> **Garde-fou :** si une US 🟢 régresse un comportement runtime (ça arrivera une fois), elle remonte définitivement en 🟠 par antipattern logué — et on apprend la frontière par l'usage, pas par la peur a priori.

---

## 5. Intégration au format US (ajout requis dans le template)

Le template `Format obligatoire d'une User Story` (custom instructions) doit désormais
porter la classe de risque, sinon ce doc reste théorique. Ligne à ajouter juste après
« Taille » :

```
**Classe de risque (PROCESS_TIERS.md) :** 🔴 CRITIQUE / 🟠 STANDARD / 🟢 LÉGÈRE
→ détermine la gate exigée (voir §1)
```

---

## 6. Exemples appliqués au backlog (⚠️ table périodiquement périmée — vérifier contre `STATE.md`)

| US | Classe | Gate | Statut |
|---|---|---|---|
| US-140a/b/c Onboarding | 🟠 STANDARD (écran, pas d'orchestration boot) | R1 + R4 ciblé | Backlog (EPIC 9) |
| US-153 try/catch save | 🔴 CRITIQUE | gate complète | ✅ **Déjà livré (S19/S21)** — cité ici comme exemple historique de bon réflexe, pas comme US ouverte. |
| US-166-CSS dette CSS groupée (inclut `.rc-mark-done`) | 🟢 LÉGÈRE | type-check + test:run + build, pas d'E2E | Backlog (EPIC 1) |
| US-127 SyncIndicator | 🟠 STANDARD | R1 + R4 ciblé | ✅ **Déjà livré (confirmé S30-S33)** — exemple historique. |
| F8 dark mode lisibilité | 🟢 LÉGÈRE | idem CSS | Backlog (EPIC 6) |
| US-MODAL-CENTER-AUDIT | 🟠 STANDARD (touche l'écran, pas l'orchestration) | R1 + R4 (le test boundingBox déjà écrit) | Backlog P1, prêt à démarrer |
