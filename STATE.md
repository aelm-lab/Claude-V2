# STATE.md — Kanban vivant Aanime

> **Où mettre ce fichier :** dans la **Knowledge** du projet Claude Chat.
> **Rôle :** source de vérité UNIQUE de l'état courant (versions, métriques, Kanban).
> Ce fichier est aussi la **source unique des compteurs** (tests/build/E2E) référencés par
> les autres docs Knowledge (`CLAUDE.md`, `ARCHITECTURE_TECHNIQUE.md`) — ne pas dupliquer
> ces chiffres ailleurs, y renvoyer. Exception : `AGENTS.md`/`AGENTS_E2E.md` (lus par Gemini,
> sans accès à ce fichier) portent leurs propres chiffres en dur, à resynchroniser à la main.
>
> **État de référence : fin S33 (régénéré au cleaning S34).**
> ⚠️ Détail S22→S27 non capturé ici (vit dans les handoffs archivés) — voir §Trous connus.

---

## 📦 Versions

| Version | Sprint | Livré |
|---|---|---|
| v0.29.0 | S29→S33 | Version inchangée depuis S29 — **bump non acté** malgré le travail livré S30-S33 (nav refactor, US-127, US-AUTH-LOGOUT). À bumper à la clôture du prochain sprint (candidat : v0.30.0). |
| v0.28.0 | S28 | **Epic Stats** : `useStats` + `StatsPage.vue` (« Mon année anime ») + route `/stats` (guard auth) + onglet Stats (`PrimaryNav.vue`) |
| S23→S27 | —  | ⚠️ Non capturé dans cette régénération (détail dans handoffs archivés S23→S27) |
| v0.21.0 | S21 | US-152 [REC] · US-157 [BOOT] · US-158 [BOOT] + `EPICS.md` consolidé |
| v0.20.0 | S20 | US-144 · US-145a/b · US-159 + taxonomie EPIC + méthodo agile |
| v0.19.0 | S19 | US-PINIA · US-JST · US-153 · CI · US-154 · US-155 · US-156a/b · US-167 |
| <= S16  | —  | Migration 0->7 · EPIC-1/2/3 clos · dual audit S16 |

---

## 🎯 Métriques actuelles (fin S33)

- **156 tests unit** (20 fichiers, +2 vs baseline S32 : T7/T8 `signOut`) · type-check vert · zéro `any`.
- Build **vite 6.4.2 · 178 modules**.
- ⚠️ **E2E : compteur non rafraîchi depuis session 16** (« 26 specs / 30 tests »). Aucune
  confirmation S17→S33 que ce chiffre a bougé. **Ne pas le citer comme actuel** tant qu'un
  `npx playwright test` complet n'a pas été rejoué et rapporté. À faire au prochain grand check E2E.
- ⚠️ **Statut réel de US-E2E-CONFIG (script Playwright local) à confirmer.** Le handoff S33
  suppose que `npx playwright test ... --workers=1` fonctionne en local (commande donnée comme
  prochaine action), mais rien ne confirme formellement que cette US a été livrée et vérifiée
  entre S30 et S33. Zéro-confiance : à clarifier en tout début de session, ne pas supposer.
- Commit `main` de référence S29 : `532ea36`. **Hash S33 non fourni** — à relever au prochain `git log -1`.
- **`npm install` fonctionne en direct** — la parade `--legacy-peer-deps` n'est **plus nécessaire**
  (downgrade `@pinia/testing` mergé). Garder `--legacy-peer-deps` uniquement si un futur
  `package.json` réintroduit le conflit.
- 🌐 **Jikan (API MyAnimeList) en panne** — confirmé S33 (curl → 504 « Jikan failed to connect to
  MyAnimeList »). Aucune évolution récente. Bloque la confirmation visuelle de plusieurs items (voir Kanban).

---

## 📋 Kanban

### ✅ Done — bloc groupé S30→S33 (gate verte machine PO)
- **US-127** — SyncIndicator sur `startBackgroundRelationFetch` : **confirmé livré**. Résout
  le trou connu laissé ouvert fin S29 (« présent en mémoire ~S28, absent du backlog S30 »).
- **Nav refactor** (header scroll asymétrique style iOS) — mergé, gate verte.
  - Root cause du flicker secondary-nav diagnostiquée : `v-show` (`display:none`) provoque des
    sauts de hauteur en cours de scroll.
  - **Piste A retenue** : sticky CSS pur, sans logique JS de scroll-hide. Confirmé : élimine le flicker.
  - Régression détectée en cours de route sur Discover (chips de filtre passées sous la nav
    devenue opaque) → corrigée en retirant le `sticky` des chips (flux normal sous la nav).
  - **N9** résolu : taille/graisse de police de `.current-period` réduite.
  - Les deux correctifs ont passé la porte locale (134 tests / 175 modules à ce stade du sprint —
    le compteur a continué de monter jusqu'à 156/178 en fin S33).
- **US-AUTH-LOGOUT** — **MERGE.** `useFirebaseAuth.signOut()` ajouté (try/catch conforme) +
  `LogoutConfirmModal.vue` créée + `AppHeader.vue` câblé (bouton 🚪, purge `aanime_*` +
  `animeStore.clearAll()` + redirect `/login`) + `useFirebaseAuth.spec.ts` T7/T8 (succès + erreur).
  Porte locale confirmée (type-check/test:run/build). *Chemin réel confirmé au passage :
  `AppHeader.vue` vit dans `src/components/ui/`, pas `src/components/` comme supposé en
  début d'investigation — à corriger partout où ce chemin était supposé.*
  **Le volet centrage de la modale est traité séparément** (voir `US-MODAL-CENTER-AUDIT`
  ci-dessous) — ne bloque plus ce merge, cf. décision PO.

### 🔄 In Progress
- *(vide)*

### 📝 To Do — priorité immédiate
1. **US-MODAL-CENTER-AUDIT** (P1, NOUVEAU) — audit du centrage sur **tous** les popups
   (`AnimeModal`, `LogoutConfirmModal`, `RecEngineModal` partagent tous `.modal-backdrop`)
   **et** le reste du site, pas seulement le logout. Point de départ déjà identifié :
   `findstr /s /n "modal-backdrop" src\*.css` (jamais exécuté — la classe n'apparaît que dans
   les 3 templates `.vue`, aucune définition de style dedans ; le CSS réel du centrage est
   ailleurs, non localisé). Le test E2E `logout-modal-position.spec.ts` (boundingBox vs centre
   viewport, tolérance 24px) fourni par Claude n'a **jamais été exécuté** (ni rouge, ni vert) —
   à faire en tout premier avant tout correctif CSS.
2. **US-SEARCH-3** — séparation « IN YOUR LIBRARY » / « ADD TO YOUR LIST ». Décision ouverte :
   cacher les en-têtes de section vides (reco Claude) ou toujours les afficher. Seul
   `SearchInput.vue` à modifier — `isAdded` existe déjà sur `enrichedSuggestions`.
3. **Micro-fix visuel Month** — signalé par le PO, capture pas encore fournie. Déféré.
4. **US-JIKAN-HEALTHCHECK** (P1) — refinement acté cette session : usage **dev-only** (pas de
   signal visible utilisateur), avec **détail par test** au-delà d'un simple verdict global
   OK/KO. Prêt à specer après le cleaning doc.

### 🗂️ Backlog (epics — détail dans `EPICS.md`)
- **US-ANILIST-SEARCH** · **US-PWA** · scroll horizontal · 4 E2E search jamais modifiées, juste
  à faire tourner (`search-enriched`, `search-quick-add`, `search-dedup`, `search-hides-nav`).
- **US-NAV-A-FIX2** (chips Discover) — correctif déjà appliqué techniquement (retrait sticky),
  mais **confirmation visuelle bloquée** tant que Jikan est KO (impossible de charger de vraies
  données pour vérifier visuellement).
- Login redesign (+ logo Google officiel) · Cluster B/C découverte · dual-titre rollout
  (modale / RecCards / carte semaine) · **US-124** (mapping MAL `Dropped` → non importé,
  déféré ; repasse P0 si campagne import MAL lancée).
- **Dette mineure** : `reject: any` dans `QueueTask` · fusion `.search-loading`/`.search-message`
  · piste `signInWithRedirect` (Safari mobile privé) · `.search-suggestion-added` `#10b981`
  en dur → `var(--airing)`.

### ⏸️ Standby
- Correctifs Jikan — panne externe confirmée S33, aucune évolution.

---

## ❓ Trous connus / à confirmer (R3 — ne pas inventer)
- **S22→S27** : aucun détail per-sprint dans cette régénération.
- **Compteur E2E** : périmé depuis session 16, jamais rafraîchi. Ne pas le citer comme actuel.
- **US-E2E-CONFIG** : statut d'exécution locale non formellement confirmé S30-S33 (voir §Métriques).
- **Hash commit `main` S33** : non fourni, à relever.
- Si des entrées DEC/US ont été ajoutées hors de cette vue, vérifier le dernier numéro réel
  avant de coller un bloc d'append (dernier connu : **DEC-99**, cf. `DECISIONS.md`).
Désynchronisation confirmée entre AGENTS.md/AGENTS_E2E.md racine (lus par Gemini) et leurs copies Knowledge — corrigé au cleaning, à surveiller à l'avenir.
US-E2E-BATCH-AUDIT (P1, nouveau) — le §8 de AGENTS_E2E.md (racine) liste les specs E2E enregistrées dans les batchs sweep ; au moins 3 specs (search-enriched, search-quick-add, search-hides-nav) semblent absentes de cette liste → probablement jamais exécutées. Audit complet de tests/e2e/ nécessaire avant de faire confiance à un futur "grand check vert".
---

## 🗄️ Archivage (nouveau — S34)
Un dossier `OLD/` a été créé dans le repo Knowledge (`aelm-lab/Claude-V2`) pour les documents
n'ayant plus de rôle opérationnel actif :
- `OLD/PLAN_MIGRATION.md` — migration Vue 3 100 % terminée, plan devenu pur historique.
- `OLD/AUDIT_UX_SESSION7.md` — audit UX live session 7, lecture seule. Ses items encore ouverts
  ont été rapatriés dans `EPICS.md` avant archivage (voir EPIC 6 — dette dark mode F8).
Ces deux fichiers ne sont plus référencés dans la table des docs actifs de `CLAUDE.md` §10.

---

## 🔁 Démarrage de session (rappel)
1. Lire la Knowledge → confirmer (1 phrase) + signaler si `STATE.md` n'est plus à jour.
2. Afficher ce Kanban.
3. Questions de clarification s'il y a ambiguïté.
4. Traiter en priorité **US-MODAL-CENTER-AUDIT** (grep CSS + E2E rouge/vert, jamais fait).
5. Attendre le `go` PO avant toute spec.
