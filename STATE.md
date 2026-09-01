STATE.md — État courant du projet Aanime

Rôle : le présent uniquement — session en cours, métriques, Kanban, trous ouverts. Pas ici : le passé (HISTORIQUE.md) · le cap multi-sprints (ROADMAP.md) · les règles (PILOTAGE.md).

Régénéré intégralement à chaque session (DEC-146), jamais patché. Plafond : 200 lignes.

📍 Session courante
	
Session	SE-073 — ouverture de S44
Sprint	S44 EN COURS — « L'arrivée vaut le produit » — 2/10 slots
Version	v0.37.0 — pas de bump (le sprint n'est pas clos)
Commit main	726a34e
Prochaine session	SE-074 — suite de S44
📊 Métriques
Métrique	Valeur	Fraîcheur
Tests unitaires	331 / 38 fichiers	SE-073 — rejoué, vert
Build	180 modules, index.js 372,59 kB (gzip 109,07) · index.css 65,20 kB (gzip 12,38)	SE-073 — relevé
Sweep E2E	🔻 non rejoué en SE-073 — dernier complet 55/55 en SE-072	SE-072
Specs E2E	45 sur disque / 45 enregistrées, mapping 1:1	SE-073 — modal-navigate-enriched (batch5, 9 specs) et mlt-real-recommendations (batch3, 9 specs) créées et enregistrées
Tests E2E attendus au sweep	58 (55 + 2 + 1)	SE-073 — à confirmer au prochain sweep
Specs sur helper unique	17	SE-073
Specs à mock mort restant	0 + 1 catch-all (week-empty-day-cta)	SE-072
ESLint	❌ jamais exécuté	—

vue-tsc --noEmit est embarqué dans npm run build, lui-même embarqué dans le webServer Playwright : un batch E2E qui démarre vaut type-check + build verts. ⚠️ Aucune sortie de Gemini n'a valeur de preuve (AUD-43). Seule la machine du PO mesure. 🔻 Le PO commite et pousse avant chaque test local. Un git status propre après un patch signifie commité, jamais perdu.

📚 Métriques documentaires
Métrique	Valeur
Corpus	15 documents (10 en ordre de lecture + 5 satellites)
Documents au-dessus du plafond H7	0
Patchs documentaires en dette	0 — lot SE-072 appliqué en ouverture de SE-073
Dernier AUD-xx	AUD-58
Dernier DEC-xxx	DEC-188
📋 Kanban — S44 EN COURS · SE-073
✅ Done — 2/10 slots
US	Impact utilisateur livré
US-MORELIKETHIS-FIX 🔴 (AUD-16)	Cliquer une saison 2 ou une relation ouvre une vraie fiche — jaquette, note, studio, genres. Avant : un titre sur fond vide. Un anime déjà en bibliothèque s'ouvre instantanément avec son état correct
US-MLT-REAL 🟠 (AUD-58)	« MORE LIKE THIS » propose enfin des titres que l'utilisateur ne possède pas. Avant : sa propre bibliothèque, triée par genre

Gemini : 2 US en SE-073. Streak → 26. 2 micro-patchs DEC-128 (helper de mock, package.json).

🔄 In Progress

Aucune.

📝 To Do — S44, 5 slots planifiés restants
#	US	Risque	Impact bêta
3	J10e-a (repli orphelins MAL, socle)	🟠	🔴
4	J10e-b	🟠	🔴
5	J10e-c	🟠	🔴
6	Refonte chrome mobile option A	🟠	🟠
7	US-STALE-SIGNAL (AUD-05) — ⚠️ DEC d'arbitrage requis avant rédaction	🟠	🟠

Flex (3) : 1 slot réservé aux retours bêta · chrome mobile option B · dette de vérité « We'll sync it later ».

Hors slot, côté PO : note aux testeurs (rédigée, prête à envoyer) · revérification AUD-42 · audit pixel AUD-56 (PO + Claude, en parallèle, ne consomme aucun slot Gemini).

Sorti de S44 : AUD-54 (remplacé par US-MLT-REAL, arbitrage PO SE-073) · US-ESLINT-CI-1 · US-DEMOCK-3 · renommage modal-status-gating · lot polish ↦ S45.

🗂️ Backlog

Cap S44 → S50 : ROADMAP.md.

🌐 Faits externes
Fait	État	Dernière mesure
AniList graphql.anilist.co	✅ Opérationnel — 200 sur la requête exacte du code (Summer 2026)	SE-073
Service worker de production	❌ Passe-plat, zéro cache	SE-071
PWA hors ligne	❌ Inexistante — cap S49	SE-071
Multi-appareil (AUD-42)	🔻 Non revérifié — prérequis bêta	SE-06x
🕳️ Trous ouverts
Bêta non ouverte — note aux testeurs prête, AUD-42 reste à revérifier. 27+ US livrées depuis S41, zéro observation utilisateur. Risque projet n°1.
AUD-56 — audit pixel complet (toutes pages, modales, clair et sombre). Aucune mesure faite. Méthode arrêtée : par parcours, 375 × 812, un thème à la fois.
Sweep non rejoué en SE-073 — 45 specs, 3 tests nouveaux jamais passés en sweep complet.
AUD-54 — vue Semaine qui reconstruit ses cartes en synchro (~7 s). Sorti de S44, cause non établie, composant non lu.
AGENTS.md §6 — règle du seed mono-jour non vérifiée (hérité SE-071).
Famille C d'AUD-52 — week-empty-day-cta route **/*, fonctionne par accident.
Titres mensongers dans modal-status-gating — deux tests nommés « ROUGE » / « VERT » testent la même chose.
Aucun token de couleur hors --accent — les 5 teintes de l'en-tête sont en dur.
modal-open.spec.ts seed la clé animeCalendar, rattrapée par la migration usePersistence.ts:139. Le jour où la migration tombe, la spec devient rouge sans qu'aucun code de modale ait bougé.
modal-navigate-enriched et mlt-real-recommendations : preuve rouge obtenue par git checkout d'un état antérieur, Gemini ayant livré avant le lancement. Procédure valide mais à anticiper : lancer la spec avant d'envoyer l'US.
