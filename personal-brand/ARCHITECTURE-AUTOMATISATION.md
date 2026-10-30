# Personal Brand — Socle d'exécution multi-agents (v0.1)

Statut : architecture préparée, **non déployée**. Moteur IA initial : OpenAI API. Orchestration cible : n8n Cloud. Répertoire d'agents : l'intégralité du dépôt courant (283 profils au dernier inventaire), sans tri ni exclusion.

## Principe
Ne pas modifier les 283 profils sources. Sélectionner à chaque mission les profils pertinents et charger leurs instructions depuis GitHub. Les contraintes métier se définissent par projet, progressivement. La configuration de projet n'écrase ni la sécurité ni les autorisations explicites nécessaires. Les actions sensibles exigent des contrôles indépendants du prompt.

## Composants mutualisés
1. **Entrée mission** : Webhook n8n ou déclencheur planifié / événement d'application. Chaque mission a project_id, mission_id, objectif, contexte et niveau d'autorisation.
2. **Registre GitHub** : liste les fichiers d'agents des départements; lit le contenu Markdown à la demande et en extrait le frontmatter; conserve un hash/commit pour audit.
3. **Sélection dynamique** : un modèle OpenAI choisit les compétences appropriées sur la base des descriptions, puis un validateur vérifie que tous les chemins existent. Aucune exclusion permanente; possibilité d'ajouter des profils en cours de mission.
4. **Moteur d'exécution** : AI Agent n8n connecté à OpenAI Chat Model, avec instructions combinant mission + profil + règles du projet. En phase 1, les compétences peuvent être invoquées séquentiellement sans prétendre à de véritables agents autonomes indépendants.
5. **Outils autorisés** : catalogue par projet (Google Workspace, GitHub, n8n, CRM, etc.). Les secrets restent dans les Credentials n8n, jamais dans le dépôt ou un prompt.
6. **Mémoire** : journal d'exécutions et décisions partagé par projet; contextes séparés entre clients. Simple Memory n'est pas à considérer comme base de données métier durable. Stockage initial éventuel : Google Sheets/Drive, puis base adaptée selon l'échelle.
7. **Qualité** : vérification d'un livrable sur critères explicites; contrôle des faits quand nécessaire; reprises limitées, remontée des blocages; audit des appels.
8. **Suivi des coûts** : calcul du nombre d'appels, tokens, coût estimé et erreurs par mission; budget maximal configurable.
9. **Remplacement du modèle** : interface fournisseur abstraite; conseil GPT/Claude/Grok plus tard après A/B test, sans réécrire les missions.

## Workflows n8n proposés
- PB-00 Registre : rafraîchit le catalogue depuis GitHub (paginé, sans filtre d'exclusion).
- PB-01 Mission : réception, validation des données, chargement des règles du projet, sélection dynamique, journal initial.
- PB-02 Exécuteur : invoque OpenAI avec instructions de l'agent et outils configurés, produit un résultat structuré.
- PB-03 Contrôle : valide résultats, collecte preuves, gère les échecs et états.
- PB-04 Mémoire : enregistre décisions, résultats, événements, coût et identifiants.
- PB-05 Supervision : alertes incidents, approbations et récapitulatif à la demande.

## Schéma minimal mission
{
  "mission_id": "uuid",
  "project_id": "personal-brand",
  "objective": "Texte",
  "context_refs": [],
  "requested_at": "ISO-8601",
  "agent_profiles": [],
  "status": "new|running|awaiting_approval|completed|failed",
  "result_refs": [],
  "cost_estimate": null
}

## Ordre de montage
A. Vérifier l'accès n8n Cloud et les crédits/API OpenAI ; ne jamais publier une clé.
B. Installer PB-00 et tester la lecture d'un profil GitHub depuis n8n.
C. Installer PB-01/PB-02 et exécuter une mission sans effet de bord.
D. Brancher PB-03/PB-04 avec mémoire et supervision.
E. Ajouter les outils de chaque projet au fur et à mesure ; activation d'actions externes seulement après tests et permissions.
F. Mesurer performances et coûts avant d'introduire un conseil multi-modèles.

## Critères d'acceptation du premier MVP
- Retrouver dynamiquement un agent existant sans copie locale;
- Traçabilité du chemin et commit du profil;
- Invocation OpenAI réussie via n8n avec un cas test;
- Résultat structuré vérifié et enregistré;
- Aucun accès secret écrit dans GitHub;
- Exécution mise en échec proprement si credential absent ou budget dépassé.

## Références
- https://github.com/nuhfear1/agency-agents/blob/main/personal-brand/CONFIGURATION.md
- https://docs.n8n.io/
- https://platform.openai.com/docs/
