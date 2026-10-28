# Bonnes pratiques des serveurs MCP

## Aide-mémoire

### Nommage du serveur
- **Python** : `{service}_mcp` (par exemple `slack_mcp`)
- **Node/TypeScript** : `{service}-mcp-server` (par exemple `slack-mcp-server`)

### Nommage des outils
- Utiliser le snake_case avec un préfixe de service
- Format : `{service}_{action}_{ressource}`
- Exemples : `slack_send_message`, `github_create_issue`

### Formats de réponse
- Prendre en charge à la fois JSON et Markdown
- JSON pour le traitement par programme
- Markdown pour la lecture humaine

### Pagination
- Toujours respecter le paramètre `limit`
- Renvoyer `has_more`, `next_offset`, `total_count`
- Par défaut, 20 à 50 éléments

### Transport
- **HTTP en flux** : pour les serveurs distants et les scénarios à plusieurs clients
- **stdio** : pour les intégrations locales et les outils en ligne de commande
- Éviter SSE (déprécié au profit du HTTP en flux)

---

## Conventions de nommage des serveurs

Suivre ces modèles de nommage normalisés :

**Python** : format `{service}_mcp` (minuscules avec tirets bas)
- Exemples : `slack_mcp`, `github_mcp`, `jira_mcp`

**Node/TypeScript** : format `{service}-mcp-server` (minuscules avec traits d'union)
- Exemples : `slack-mcp-server`, `github-mcp-server`, `jira-mcp-server`

Le nom doit être général, décrire le service intégré, se déduire facilement de la description de la tâche, et ne pas contenir de numéro de version.

---

## Nommage et conception des outils

### Nommage des outils

1. **Utiliser le snake_case** : `search_users`, `create_project`, `get_channel_info`
2. **Inclure un préfixe de service** : prévoir que le serveur MCP sera peut-être utilisé avec d'autres serveurs MCP
   - utiliser `slack_send_message` plutôt que simplement `send_message`
   - utiliser `github_create_issue` plutôt que simplement `create_issue`
3. **Être orienté action** : commencer par un verbe (get, list, search, create, etc.)
4. **Être précis** : éviter les noms génériques qui pourraient entrer en conflit avec d'autres serveurs

### Conception des outils

- La description d'un outil doit décrire sa fonction de façon étroite et sans ambiguïté
- Elle doit correspondre exactement à ce que fait l'outil
- Fournir les annotations d'outil (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)
- Garder des opérations ciblées et atomiques

---

## Formats de réponse

Tous les outils qui renvoient des données doivent prendre en charge plusieurs formats :

### Format JSON (`response_format="json"`)
- Données structurées lisibles par une machine
- Inclure tous les champs et métadonnées disponibles
- Noms et types de champs cohérents
- À utiliser pour le traitement par programme

### Format Markdown (`response_format="markdown"`, en général par défaut)
- Texte mis en forme, lisible par un humain
- Utiliser titres, listes et mise en forme pour la clarté
- Convertir les horodatages en format lisible
- Afficher les noms avec les identifiants entre parenthèses
- Omettre les métadonnées verbeuses

---

## Pagination

Pour les outils qui listent des ressources :

- **Toujours respecter le paramètre `limit`**
- **Implémenter la pagination** : par décalage (`offset`) ou par curseur
- **Renvoyer les métadonnées de pagination** : inclure `has_more`, `next_offset` / `next_cursor`, `total_count`
- **Ne jamais charger tous les résultats en mémoire** : particulièrement important pour les gros volumes
- **Choisir des limites par défaut raisonnables** : 20 à 50 éléments en général

Exemple de réponse paginée :
```json
{
  "total": 150,
  "count": 20,
  "offset": 0,
  "items": [...],
  "has_more": true,
  "next_offset": 20
}
```

---

## Options de transport

### HTTP en flux (*streamable HTTP*)

**Idéal pour** : serveurs distants, services web, scénarios à plusieurs clients

**Caractéristiques** :
- communication bidirectionnelle sur HTTP
- prend en charge plusieurs clients simultanés
- peut être déployé comme service web
- permet les notifications du serveur vers le client

**À utiliser pour** :
- servir plusieurs clients simultanément
- un déploiement comme service cloud
- l'intégration avec des applications web

### stdio

**Idéal pour** : intégrations locales, outils en ligne de commande

**Caractéristiques** :
- communication par les flux d'entrée et de sortie standard
- mise en place simple, aucune configuration réseau
- s'exécute comme sous-processus du client

**À utiliser pour** :
- construire des outils pour des environnements de développement locaux
- l'intégration avec des applications de bureau
- les scénarios à un seul utilisateur et une seule session

**Remarque** : un serveur stdio NE DOIT PAS écrire ses journaux sur stdout (utiliser stderr pour les journaux)

### Choix du transport

| Critère | stdio | HTTP en flux |
|-----------|-------|-----------------|
| **Déploiement** | Local | Distant |
| **Clients** | Un seul | Plusieurs |
| **Complexité** | Faible | Moyenne |
| **Temps réel** | Non | Oui |

---

## Bonnes pratiques de sécurité

### Authentification et autorisation

**OAuth 2.1** :
- utiliser OAuth 2.1 sécurisé avec des certificats d'autorités reconnues
- valider les jetons d'accès avant de traiter les requêtes
- n'accepter que les jetons destinés spécifiquement au serveur

**Clés d'API** :
- stocker les clés d'API dans des variables d'environnement, jamais dans le code
- valider les clés au démarrage du serveur
- fournir des messages d'erreur clairs en cas d'échec d'authentification

### Validation des entrées

- assainir les chemins de fichiers pour empêcher la traversée de répertoires
- valider les URL et les identifiants externes
- vérifier la taille et les plages des paramètres
- empêcher l'injection de commandes dans les appels système
- utiliser la validation par schéma (Pydantic / Zod) pour toutes les entrées

### Gestion des erreurs

- ne pas exposer les erreurs internes aux clients
- journaliser côté serveur les erreurs liées à la sécurité
- fournir des messages d'erreur utiles mais qui ne révèlent rien
- libérer les ressources après une erreur

### Protection contre le DNS rebinding

Pour les serveurs HTTP en flux qui tournent en local :
- activer la protection contre le DNS rebinding
- valider l'en-tête `Origin` de toutes les connexions entrantes
- écouter sur `127.0.0.1` plutôt que sur `0.0.0.0`

---

## Annotations d'outil

Fournir des annotations pour aider les clients à comprendre le comportement des outils :

| Annotation | Type | Défaut | Description |
|-----------|------|---------|-------------|
| `readOnlyHint` | booléen | false | L'outil ne modifie pas son environnement |
| `destructiveHint` | booléen | true | L'outil peut faire des modifications destructives |
| `idempotentHint` | booléen | false | Des appels répétés avec les mêmes arguments n'ont pas d'effet supplémentaire |
| `openWorldHint` | booléen | true | L'outil interagit avec des entités externes |

**Important** : les annotations sont des indications, pas des garanties de sécurité. Un client ne doit pas prendre de décision critique pour la sécurité sur la seule base des annotations.

---

## Gestion des erreurs

- utiliser les codes d'erreur JSON-RPC standard
- signaler les erreurs d'outil dans l'objet résultat (pas comme des erreurs de niveau protocole)
- fournir des messages d'erreur utiles et précis, avec les étapes suivantes suggérées
- ne pas exposer les détails d'implémentation internes
- libérer correctement les ressources en cas d'erreur

Exemple de gestion d'erreur :
```typescript
try {
  const result = performOperation();
  return { content: [{ type: "text", text: result }] };
} catch (error) {
  return {
    isError: true,
    content: [{
      type: "text",
      text: `Erreur : ${error.message}. Essayez filter='active_only' pour réduire les résultats.`
    }]
  };
}
```

---

## Exigences de test

Des tests complets doivent couvrir :

- **les tests fonctionnels** : vérifier l'exécution correcte avec des entrées valides et invalides
- **les tests d'intégration** : tester l'interaction avec les systèmes externes
- **les tests de sécurité** : valider l'authentification, l'assainissement des entrées, la limitation de débit
- **les tests de performance** : vérifier le comportement sous charge, les délais d'expiration
- **la gestion des erreurs** : s'assurer que les erreurs sont correctement signalées et les ressources libérées

---

## Exigences de documentation

- documenter clairement tous les outils et toutes les capacités
- inclure des exemples fonctionnels (au moins 3 par fonctionnalité majeure)
- documenter les considérations de sécurité
- préciser les autorisations et niveaux d'accès requis
- documenter les limites de débit et les caractéristiques de performance
