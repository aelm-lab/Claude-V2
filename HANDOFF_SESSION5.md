# HANDOFF_SESSION5.md — Reprise de projet

> **Où mettre ce fichier :** dans la Knowledge du projet Claude Chat.

## 1. Où on en est

**LA MIGRATION EST TERMINÉE ET VALIDÉE.** Audit externe (Claude Code) :
- vue-tsc : 0 erreur · vitest : 64/66 (2 échecs = tests TZ-dépendants, pas de bug app)
- Build prod 2,5s · zéro any · zéro dispatchEvent · zéro DOM direct (sauf exceptions validées)
- Architecture conforme au plan · règles métier subtiles préservées

**Prochaine étape : EPIC-1 « clôture v2-stable »** (ROADMAP.md) — 5 US de finition.

## 2. Prompt de démarrage nouvelle conversation
On reprend le projet Aanime (migration Vue 3 TERMINÉE, validée par audit).
Lis toute la Knowledge : CLAUDE.md, AUDIT.md, PLAN_MIGRATION.md, BACKLOG.md,
TYPES_CONTRACT.md, ANTIPATTERNS.md, DECISIONS.md, ROADMAP.md, HANDOFF_SESSION5.md.
On démarre EPIC-1 (clôture v2-stable) : US-101 suppression du code vanilla mort.
Confirme la lecture, affiche le Kanban, rédige US-101.
Applique toutes les règles process (zéro confiance, 3 fichiers max,
sortie terminale = 2 lignes exactes sans commentaire).

## 3. Pièges connus pour la suite

- L'upsert store + `episodeOverride: undefined` : reset possiblement inopérant (US-128).
- `useEpisodeInfo` expose `getEpisodeInfo`/`getStatus`/`checkIsOnHiatus` ; `useICS` expose `downloadICS` — TYPES_CONTRACT §7 était faux sur ces noms.
- 4 copies locales de POSTER_PLACEHOLDER (+1 dans constants.ts) — celle des strips est visuellement différente.
- US-101 : la suppression des .js vanilla doit vérifier qu'AUCUN import ne pointe dessus (grep avant rm).
- Gemini récidive sur la sortie terminale — la règle « 2 lignes exactes » doit être martelée dans chaque US.
