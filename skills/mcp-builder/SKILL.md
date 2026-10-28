---
name: mcp-builder
description: Guide pour créer des serveurs MCP (Model Context Protocol) de qualité, qui permettent aux LLM d'interagir avec des services externes au moyen d'outils bien conçus. À utiliser pour construire un serveur MCP qui intègre des API ou des services externes, en Python (FastMCP) comme en Node/TypeScript (SDK MCP).
license: Conditions complètes dans LICENSE.txt
---

# Guide de développement de serveurs MCP

## Présentation

Créer des serveurs MCP (Model Context Protocol) qui permettent aux LLM d'interagir avec des services externes au moyen d'outils bien conçus. La qualité d'un serveur MCP se mesure à la façon dont il permet aux LLM d'accomplir des tâches réelles.

---

# Démarche

## 🚀 Vue d'ensemble

Créer un serveur MCP de qualité passe par quatre grandes phases :

### Phase 1 : recherche approfondie et planification

#### 1.1 Comprendre la conception MCP moderne

**Couverture de l'API ou outils de démarche :**
Trouver l'équilibre entre une couverture complète des points d'accès de l'API et des outils spécialisés par démarche. Les outils de démarche peuvent être plus pratiques pour des tâches précises, tandis qu'une couverture complète laisse aux agents la souplesse de composer les opérations. Les performances varient selon le client : certains clients tirent profit de l'exécution de code qui combine des outils de base, d'autres fonctionnent mieux avec des démarches de plus haut niveau. En cas de doute, privilégier une couverture complète de l'API.

**Nommage et découvrabilité des outils :**
Des noms d'outils clairs et parlants aident les agents à trouver vite le bon outil. Utiliser des préfixes cohérents (par exemple `github_create_issue`, `github_list_repos`) et des noms orientés action.

**Gestion du contexte :**
Les agents tirent profit de descriptions d'outils concises et de la possibilité de filtrer et de paginer les résultats. Concevoir des outils qui renvoient des données ciblées et pertinentes. Certains clients prennent en charge l'exécution de code, ce qui aide les agents à filtrer et traiter les données efficacement.

**Messages d'erreur exploitables :**
Les messages d'erreur doivent orienter les agents vers une solution, avec des suggestions précises et les étapes suivantes.

#### 1.2 Étudier la documentation du protocole MCP

**Parcourir la spécification MCP :**

Commencer par le plan du site pour trouver les pages utiles : `https://modelcontextprotocol.io/sitemap.xml`

Puis récupérer les pages voulues avec le suffixe `.md` pour les obtenir en Markdown (par exemple `https://modelcontextprotocol.io/specification/draft.md`).

Pages clés à consulter :
- présentation et architecture de la spécification
- mécanismes de transport (HTTP en flux, stdio)
- définitions des outils, des ressources et des invites (*prompts*)

#### 1.3 Étudier la documentation du framework

**Pile recommandée :**
- **Langage** : TypeScript (SDK de qualité et bonne compatibilité avec de nombreux environnements d'exécution, par exemple MCPB. En outre, les modèles d'IA génèrent bien le TypeScript, grâce à son large usage, à son typage statique et à ses bons outils d'analyse)
- **Transport** : HTTP en flux (*streamable HTTP*) pour les serveurs distants, en JSON sans état (plus simple à faire évoluer et à maintenir que des sessions avec état et des réponses en flux). stdio pour les serveurs locaux.

**Charger la documentation du framework :**

- **Bonnes pratiques MCP** : [📋 Voir les bonnes pratiques](./reference/mcp_best_practices.md) — consignes fondamentales

**Pour TypeScript (recommandé) :**
- **SDK TypeScript** : utiliser WebFetch pour charger `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ Guide TypeScript](./reference/node_mcp_server.md) — motifs et exemples TypeScript

**Pour Python :**
- **SDK Python** : utiliser WebFetch pour charger `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Guide Python](./reference/python_mcp_server.md) — motifs et exemples Python

#### 1.4 Planifier l'implémentation

**Comprendre l'API :**
Étudier la documentation de l'API du service pour repérer les points d'accès clés, les exigences d'authentification et les modèles de données. Utiliser la recherche web et WebFetch au besoin.

**Choix des outils :**
Privilégier une couverture complète de l'API. Lister les points d'accès à implémenter, en commençant par les opérations les plus courantes.

---

### Phase 2 : implémentation

#### 2.1 Mettre en place la structure du projet

Voir les guides propres à chaque langage pour la mise en place :
- [⚡ Guide TypeScript](./reference/node_mcp_server.md) — structure du projet, package.json, tsconfig.json
- [🐍 Guide Python](./reference/python_mcp_server.md) — organisation des modules, dépendances

#### 2.2 Implémenter l'infrastructure de base

Créer des utilitaires partagés :
- client d'API avec authentification
- aides pour la gestion des erreurs
- mise en forme des réponses (JSON / Markdown)
- prise en charge de la pagination

#### 2.3 Implémenter les outils

Pour chaque outil :

**Schéma d'entrée :**
- utiliser Zod (TypeScript) ou Pydantic (Python)
- inclure des contraintes et des descriptions claires
- ajouter des exemples dans la description des champs

**Schéma de sortie :**
- définir un `outputSchema` quand c'est possible pour les données structurées
- utiliser `structuredContent` dans les réponses des outils (fonctionnalité du SDK TypeScript)
- aide les clients à comprendre et traiter les sorties des outils

**Description de l'outil :**
- résumé concis de sa fonction
- description des paramètres
- schéma du type de retour

**Implémentation :**
- async/await pour les entrées-sorties
- gestion correcte des erreurs, avec des messages exploitables
- prise en charge de la pagination quand elle s'applique
- renvoyer à la fois du contenu texte et des données structurées avec les SDK modernes

**Annotations :**
- `readOnlyHint` : true/false (lecture seule)
- `destructiveHint` : true/false (destructif)
- `idempotentHint` : true/false (idempotent)
- `openWorldHint` : true/false (interagit avec un monde ouvert, extérieur)

---

### Phase 3 : relecture et tests

#### 3.1 Qualité du code

Vérifier :
- l'absence de code dupliqué (principe DRY)
- la cohérence de la gestion des erreurs
- une couverture de types complète
- la clarté des descriptions d'outils

#### 3.2 Construire et tester

**TypeScript :**
- lancer `npm run build` pour vérifier la compilation
- tester avec MCP Inspector : `npx @modelcontextprotocol/inspector`

**Python :**
- vérifier la syntaxe : `python -m py_compile your_server.py`
- tester avec MCP Inspector

Voir les guides propres à chaque langage pour les méthodes de test détaillées et les listes de contrôle qualité.

---

### Phase 4 : créer des évaluations

Une fois le serveur MCP implémenté, créer des évaluations complètes pour tester son efficacité.

**Charger le [✅ guide d'évaluation](./reference/evaluation.md) pour les consignes complètes.**

#### 4.1 Comprendre le but de l'évaluation

Les évaluations servent à tester si des LLM savent utiliser efficacement le serveur MCP pour répondre à des questions réalistes et complexes.

#### 4.2 Créer 10 questions d'évaluation

Pour créer des évaluations efficaces, suivre le processus décrit dans le guide d'évaluation :

1. **Inspection des outils** : lister les outils disponibles et comprendre leurs capacités
2. **Exploration du contenu** : utiliser des opérations EN LECTURE SEULE pour explorer les données disponibles
3. **Génération des questions** : créer 10 questions complexes et réalistes
4. **Vérification des réponses** : résoudre soi-même chaque question pour vérifier les réponses

#### 4.3 Exigences de l'évaluation

Chaque question doit être :
- **indépendante** : ne pas dépendre des autres questions
- **en lecture seule** : n'exiger que des opérations non destructives
- **complexe** : demander plusieurs appels d'outils et une exploration approfondie
- **réaliste** : fondée sur des cas d'usage réels qui intéressent des humains
- **vérifiable** : une réponse unique et claire, vérifiable par comparaison de chaînes
- **stable** : la réponse ne changera pas avec le temps

#### 4.4 Format de sortie

Créer un fichier XML de cette structure :

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- Autres qa_pair... -->
</evaluation>
```

(La question d'exemple se traduit ainsi : « Trouver les discussions sur des lancements de modèles d'IA aux noms de code animaliers. L'un des modèles nécessitait une désignation de sécurité au format ASL-X. Quel chiffre X était en cours de détermination pour le modèle nommé d'après un félin sauvage tacheté ? » Les questions s'écrivent dans la langue des données explorées.)

---

# Fichiers de référence

## 📚 Bibliothèque de documentation

Charger ces ressources au besoin pendant le développement :

### Documentation MCP de base (à charger en premier)
- **Protocole MCP** : commencer par le plan du site `https://modelcontextprotocol.io/sitemap.xml`, puis récupérer les pages voulues avec le suffixe `.md`
- [📋 Bonnes pratiques MCP](./reference/mcp_best_practices.md) — consignes MCP universelles, dont :
  - conventions de nommage des serveurs et des outils
  - consignes de format de réponse (JSON ou Markdown)
  - bonnes pratiques de pagination
  - choix du transport (HTTP en flux ou stdio)
  - normes de sécurité et de gestion des erreurs

### Documentation des SDK (à charger pendant les phases 1 et 2)
- **SDK Python** : récupérer `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- **SDK TypeScript** : récupérer `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`

### Guides d'implémentation par langage (à charger pendant la phase 2)
- [🐍 Guide d'implémentation Python](./reference/python_mcp_server.md) — guide complet Python / FastMCP avec :
  - motifs d'initialisation du serveur
  - exemples de modèles Pydantic
  - enregistrement des outils avec `@mcp.tool`
  - exemples complets fonctionnels
  - liste de contrôle qualité

- [⚡ Guide d'implémentation TypeScript](./reference/node_mcp_server.md) — guide complet TypeScript avec :
  - structure du projet
  - motifs de schémas Zod
  - enregistrement des outils avec `server.registerTool`
  - exemples complets fonctionnels
  - liste de contrôle qualité

### Guide d'évaluation (à charger pendant la phase 4)
- [✅ Guide d'évaluation](./reference/evaluation.md) — guide complet de création d'évaluations avec :
  - consignes de rédaction des questions
  - stratégies de vérification des réponses
  - spécification du format XML
  - exemples de questions et de réponses
  - lancement d'une évaluation avec les scripts fournis
