# Guide d'implémentation d'un serveur MCP en Node/TypeScript

## Présentation

Ce document rassemble les bonnes pratiques et les exemples propres à Node/TypeScript pour implémenter des serveurs MCP avec le SDK MCP TypeScript. Il couvre la structure du projet, la mise en place du serveur, les motifs d'enregistrement des outils, la validation des entrées avec Zod, la gestion des erreurs et des exemples complets fonctionnels.

---

## Aide-mémoire

### Importations clés
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import express from "express";
import { z } from "zod";
```

### Initialisation du serveur
```typescript
const server = new McpServer({
  name: "service-mcp-server",
  version: "1.0.0"
});
```

### Motif d'enregistrement d'un outil
```typescript
server.registerTool(
  "tool_name",
  {
    title: "Nom affiché de l'outil",
    description: "Ce que fait l'outil",
    inputSchema: { param: z.string() },
    outputSchema: { result: z.string() }
  },
  async ({ param }) => {
    const output = { result: `Traité : ${param}` };
    return {
      content: [{ type: "text", text: JSON.stringify(output) }],
      structuredContent: output // Motif moderne pour les données structurées
    };
  }
);
```

---

## Le SDK MCP TypeScript

Le SDK MCP TypeScript officiel fournit :
- la classe `McpServer` pour initialiser le serveur
- la méthode `registerTool` pour enregistrer les outils
- l'intégration des schémas Zod pour valider les entrées à l'exécution
- des implémentations de gestionnaires d'outils typées

**IMPORTANT — n'utiliser que les API modernes :**
- **À UTILISER** : `server.registerTool()`, `server.registerResource()`, `server.registerPrompt()`
- **À NE PAS UTILISER** : les anciennes API dépréciées comme `server.tool()`, `server.setRequestHandler(ListToolsRequestSchema, ...)` ou l'enregistrement manuel de gestionnaires
- Les méthodes `register*` offrent une meilleure sûreté de typage, une gestion automatique des schémas, et sont l'approche recommandée

Voir la documentation du SDK MCP dans les références pour tous les détails.

## Convention de nommage du serveur

Les serveurs MCP Node/TypeScript doivent suivre ce modèle de nommage :
- **Format** : `{service}-mcp-server` (minuscules avec traits d'union)
- **Exemples** : `github-mcp-server`, `jira-mcp-server`, `stripe-mcp-server`

Le nom doit être :
- général (non lié à des fonctionnalités précises)
- descriptif du service ou de l'API intégrés
- facile à déduire de la description de la tâche
- sans numéro de version ni date

## Structure du projet

Créer la structure suivante pour un serveur MCP Node/TypeScript :

```
{service}-mcp-server/
├── package.json
├── tsconfig.json
├── README.md
├── src/
│   ├── index.ts          # Point d'entrée principal, initialisation de McpServer
│   ├── types.ts          # Définitions de types et interfaces TypeScript
│   ├── tools/            # Implémentations des outils (un fichier par domaine)
│   ├── services/         # Clients d'API et utilitaires partagés
│   ├── schemas/          # Schémas de validation Zod
│   └── constants.ts      # Constantes partagées (API_URL, CHARACTER_LIMIT, etc.)
└── dist/                 # Fichiers JavaScript compilés (point d'entrée : dist/index.js)
```

## Implémentation des outils

### Nommage des outils

Utiliser le snake_case pour les noms d'outils (par exemple « search_users », « create_project », « get_channel_info »), avec des noms clairs et orientés action.

**Éviter les conflits de noms** : inclure le contexte du service pour éviter les chevauchements :
- utiliser « slack_send_message » plutôt que simplement « send_message »
- utiliser « github_create_issue » plutôt que simplement « create_issue »
- utiliser « asana_list_tasks » plutôt que simplement « list_tasks »

### Structure d'un outil

Les outils s'enregistrent avec la méthode `registerTool`, avec les exigences suivantes :
- utiliser des schémas Zod pour la validation des entrées à l'exécution et la sûreté de typage
- le champ `description` doit être fourni explicitement — les commentaires JSDoc ne sont PAS extraits automatiquement
- fournir explicitement `title`, `description`, `inputSchema` et `annotations`
- `inputSchema` doit être un objet de schéma Zod (pas un schéma JSON)
- typer explicitement tous les paramètres et valeurs de retour

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({
  name: "example-mcp",
  version: "1.0.0"
});

// Schéma Zod de validation des entrées
const UserSearchInputSchema = z.object({
  query: z.string()
    .min(2, "La requête doit compter au moins 2 caractères")
    .max(200, "La requête ne doit pas dépasser 200 caractères")
    .describe("Chaîne à rechercher dans les noms et adresses"),
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Nombre maximal de résultats à renvoyer"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Nombre de résultats à sauter pour la pagination"),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Format de sortie : 'markdown' lisible par un humain, ou 'json' lisible par une machine")
}).strict();

// Définition de type tirée du schéma Zod
type UserSearchInput = z.infer<typeof UserSearchInputSchema>;

server.registerTool(
  "example_search_users",
  {
    title: "Rechercher des utilisateurs Example",
    description: `Rechercher des utilisateurs dans le système Example par nom, adresse ou équipe.

Cet outil cherche dans tous les profils d'utilisateurs de la plateforme Example, avec prise en charge des correspondances partielles et de divers filtres. Il NE crée NI ne modifie d'utilisateurs : il cherche seulement parmi ceux qui existent.

Args:
  - query (string) : chaîne à rechercher dans les noms et adresses
  - limit (number) : nombre maximal de résultats, entre 1 et 100 (défaut : 20)
  - offset (number) : nombre de résultats à sauter pour la pagination (défaut : 0)
  - response_format ('markdown' | 'json') : format de sortie (défaut : 'markdown')

Returns:
  Au format JSON : données structurées selon le schéma :
  {
    "total": number,           // Nombre total de correspondances trouvées
    "count": number,           // Nombre de résultats dans cette réponse
    "offset": number,          // Décalage de pagination courant
    "users": [
      {
        "id": string,          // Identifiant de l'utilisateur (par ex. "U123456789")
        "name": string,        // Nom complet (par ex. "John Doe")
        "email": string,       // Adresse électronique
        "team": string,        // Nom de l'équipe (facultatif)
        "active": boolean      // L'utilisateur est-il actif
      }
    ],
    "has_more": boolean,       // Reste-t-il d'autres résultats
    "next_offset": number      // Décalage de la page suivante (si has_more vaut true)
  }

Exemples :
  - À utiliser pour : « Trouver tous les membres de l'équipe marketing » -> params avec query="team:marketing"
  - À utiliser pour : « Chercher le compte de John » -> params avec query="john"
  - À ne pas utiliser pour : créer un utilisateur (utiliser example_create_user)

Gestion des erreurs :
  - Renvoie "Erreur : limite de débit dépassée" en cas de requêtes trop nombreuses (statut 429)
  - Renvoie "Aucun utilisateur ne correspond à '<requête>'" si la recherche ne donne rien`,
    inputSchema: UserSearchInputSchema,
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async (params: UserSearchInput) => {
    try {
      // La validation des entrées est assurée par le schéma Zod
      // Faire la requête d'API avec les paramètres validés
      const data = await makeApiRequest<any>(
        "users/search",
        "GET",
        undefined,
        {
          q: params.query,
          limit: params.limit,
          offset: params.offset
        }
      );

      const users = data.users || [];
      const total = data.total || 0;

      if (!users.length) {
        return {
          content: [{
            type: "text",
            text: `Aucun utilisateur ne correspond à '${params.query}'`
          }]
        };
      }

      // Préparer la sortie structurée
      const output = {
        total,
        count: users.length,
        offset: params.offset,
        users: users.map((user: any) => ({
          id: user.id,
          name: user.name,
          email: user.email,
          ...(user.team ? { team: user.team } : {}),
          active: user.active ?? true
        })),
        has_more: total > params.offset + users.length,
        ...(total > params.offset + users.length ? {
          next_offset: params.offset + users.length
        } : {})
      };

      // Mettre en forme la représentation texte selon le format demandé
      let textContent: string;
      if (params.response_format === ResponseFormat.MARKDOWN) {
        const lines = [`# Résultats de la recherche d'utilisateurs : '${params.query}'`, "",
          `${total} utilisateurs trouvés (${users.length} affichés)`, ""];
        for (const user of users) {
          lines.push(`## ${user.name} (${user.id})`);
          lines.push(`- **Adresse** : ${user.email}`);
          if (user.team) lines.push(`- **Équipe** : ${user.team}`);
          lines.push("");
        }
        textContent = lines.join("\n");
      } else {
        textContent = JSON.stringify(output, null, 2);
      }

      return {
        content: [{ type: "text", text: textContent }],
        structuredContent: output // Motif moderne pour les données structurées
      };
    } catch (error) {
      return {
        content: [{
          type: "text",
          text: handleApiError(error)
        }]
      };
    }
  }
);
```

## Schémas Zod pour la validation des entrées

Zod assure la validation des types à l'exécution :

```typescript
import { z } from "zod";

// Schéma de base avec validation
const CreateUserSchema = z.object({
  name: z.string()
    .min(1, "Le nom est obligatoire")
    .max(100, "Le nom ne doit pas dépasser 100 caractères"),
  email: z.string()
    .email("Format d'adresse invalide"),
  age: z.number()
    .int("L'âge doit être un nombre entier")
    .min(0, "L'âge ne peut pas être négatif")
    .max(150, "L'âge ne peut pas dépasser 150")
}).strict();  // Utiliser .strict() pour interdire les champs supplémentaires

// Énumérations
enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

const SearchSchema = z.object({
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Format de sortie")
});

// Champs facultatifs avec valeurs par défaut
const PaginationSchema = z.object({
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Nombre maximal de résultats à renvoyer"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Nombre de résultats à sauter")
});
```

## Options de format de réponse

Prendre en charge plusieurs formats de sortie, pour la souplesse :

```typescript
enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

const inputSchema = z.object({
  query: z.string(),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Format de sortie : 'markdown' lisible par un humain, ou 'json' lisible par une machine")
});
```

**Format Markdown** :
- utiliser titres, listes et mise en forme pour la clarté
- convertir les horodatages en format lisible
- afficher les noms avec les identifiants entre parenthèses
- omettre les métadonnées verbeuses
- regrouper logiquement les informations liées

**Format JSON** :
- renvoyer des données complètes et structurées, adaptées au traitement par programme
- inclure tous les champs et métadonnées disponibles
- utiliser des noms et types de champs cohérents

## Implémentation de la pagination

Pour les outils qui listent des ressources :

```typescript
const ListSchema = z.object({
  limit: z.number().int().min(1).max(100).default(20),
  offset: z.number().int().min(0).default(0)
});

async function listItems(params: z.infer<typeof ListSchema>) {
  const data = await apiRequest(params.limit, params.offset);

  const response = {
    total: data.total,
    count: data.items.length,
    offset: params.offset,
    items: data.items,
    has_more: data.total > params.offset + data.items.length,
    next_offset: data.total > params.offset + data.items.length
      ? params.offset + data.items.length
      : undefined
  };

  return JSON.stringify(response, null, 2);
}
```

## Limites de caractères et troncature

Ajouter une constante `CHARACTER_LIMIT` pour éviter des réponses trop volumineuses :

```typescript
// Au niveau du module, dans constants.ts
export const CHARACTER_LIMIT = 25000;  // Taille maximale d'une réponse, en caractères

async function searchTool(params: SearchInput) {
  let result = generateResponse(data);

  // Vérifier la limite de caractères et tronquer si nécessaire
  if (result.length > CHARACTER_LIMIT) {
    const truncatedData = data.slice(0, Math.max(1, data.length / 2));
    response.data = truncatedData;
    response.truncated = true;
    response.truncation_message =
      `Réponse tronquée de ${data.length} à ${truncatedData.length} éléments. ` +
      `Utilisez le paramètre 'offset' ou ajoutez des filtres pour voir plus de résultats.`;
    result = JSON.stringify(response, null, 2);
  }

  return result;
}
```

## Gestion des erreurs

Fournir des messages d'erreur clairs et exploitables :

```typescript
import axios, { AxiosError } from "axios";

function handleApiError(error: unknown): string {
  if (error instanceof AxiosError) {
    if (error.response) {
      switch (error.response.status) {
        case 404:
          return "Erreur : ressource introuvable. Vérifiez que l'identifiant est correct.";
        case 403:
          return "Erreur : accès refusé. Vous n'avez pas accès à cette ressource.";
        case 429:
          return "Erreur : limite de débit dépassée. Patientez avant de refaire des requêtes.";
        default:
          return `Erreur : la requête d'API a échoué avec le statut ${error.response.status}`;
      }
    } else if (error.code === "ECONNABORTED") {
      return "Erreur : la requête a expiré. Réessayez.";
    }
  }
  return `Erreur : une erreur inattendue s'est produite : ${error instanceof Error ? error.message : String(error)}`;
}
```

## Utilitaires partagés

Extraire les fonctionnalités communes dans des fonctions réutilisables :

```typescript
// Fonction partagée de requête d'API
async function makeApiRequest<T>(
  endpoint: string,
  method: "GET" | "POST" | "PUT" | "DELETE" = "GET",
  data?: any,
  params?: any
): Promise<T> {
  try {
    const response = await axios({
      method,
      url: `${API_BASE_URL}/${endpoint}`,
      data,
      params,
      timeout: 30000,
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    });
    return response.data;
  } catch (error) {
    throw error;
  }
}
```

## Bonnes pratiques async/await

Toujours utiliser async/await pour les requêtes réseau et les entrées-sorties :

```typescript
// Bien : requête réseau asynchrone
async function fetchData(resourceId: string): Promise<ResourceData> {
  const response = await axios.get(`${API_URL}/resource/${resourceId}`);
  return response.data;
}

// Mal : chaînes de promesses
function fetchData(resourceId: string): Promise<ResourceData> {
  return axios.get(`${API_URL}/resource/${resourceId}`)
    .then(response => response.data);  // Plus difficile à lire et à maintenir
}
```

## Bonnes pratiques TypeScript

1. **TypeScript strict** : activer le mode strict dans tsconfig.json
2. **Définir des interfaces** : créer des interfaces claires pour toutes les structures de données
3. **Éviter `any`** : utiliser des types appropriés ou `unknown` plutôt que `any`
4. **Zod pour la validation à l'exécution** : valider les données externes avec des schémas Zod
5. **Gardes de type** : créer des fonctions de garde de type pour les vérifications complexes
6. **Gestion des erreurs** : toujours utiliser try-catch avec une vérification correcte du type d'erreur
7. **Sûreté face à null** : utiliser le chaînage optionnel (`?.`) et la coalescence des nuls (`??`)

```typescript
// Bien : typé, avec Zod et des interfaces
interface UserResponse {
  id: string;
  name: string;
  email: string;
  team?: string;
  active: boolean;
}

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  team: z.string().optional(),
  active: z.boolean()
});

type User = z.infer<typeof UserSchema>;

async function getUser(id: string): Promise<User> {
  const data = await apiCall(`/users/${id}`);
  return UserSchema.parse(data);  // Validation à l'exécution
}

// Mal : utilisation de any
async function getUser(id: string): Promise<any> {
  return await apiCall(`/users/${id}`);  // Aucune sûreté de typage
}
```

## Configuration du paquet

### package.json

```json
{
  "name": "{service}-mcp-server",
  "version": "1.0.0",
  "description": "Serveur MCP pour l'intégration de l'API {Service}",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "clean": "rm -rf dist"
  },
  "engines": {
    "node": ">=18"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.6.1",
    "axios": "^1.7.9",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@types/node": "^22.10.0",
    "tsx": "^4.19.2",
    "typescript": "^5.7.2"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Exemple complet

```typescript
#!/usr/bin/env node
/**
 * Serveur MCP pour le service Example.
 *
 * Ce serveur fournit des outils pour interagir avec l'API Example : recherche
 * d'utilisateurs, gestion de projets et export de données.
 */

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import axios, { AxiosError } from "axios";

// Constantes
const API_BASE_URL = "https://api.example.com/v1";
const CHARACTER_LIMIT = 25000;

// Énumérations
enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

// Schémas Zod
const UserSearchInputSchema = z.object({
  query: z.string()
    .min(2, "La requête doit compter au moins 2 caractères")
    .max(200, "La requête ne doit pas dépasser 200 caractères")
    .describe("Chaîne à rechercher dans les noms et adresses"),
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Nombre maximal de résultats à renvoyer"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Nombre de résultats à sauter pour la pagination"),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Format de sortie : 'markdown' lisible par un humain, ou 'json' lisible par une machine")
}).strict();

type UserSearchInput = z.infer<typeof UserSearchInputSchema>;

// Fonctions utilitaires partagées
async function makeApiRequest<T>(
  endpoint: string,
  method: "GET" | "POST" | "PUT" | "DELETE" = "GET",
  data?: any,
  params?: any
): Promise<T> {
  try {
    const response = await axios({
      method,
      url: `${API_BASE_URL}/${endpoint}`,
      data,
      params,
      timeout: 30000,
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    });
    return response.data;
  } catch (error) {
    throw error;
  }
}

function handleApiError(error: unknown): string {
  if (error instanceof AxiosError) {
    if (error.response) {
      switch (error.response.status) {
        case 404:
          return "Erreur : ressource introuvable. Vérifiez que l'identifiant est correct.";
        case 403:
          return "Erreur : accès refusé. Vous n'avez pas accès à cette ressource.";
        case 429:
          return "Erreur : limite de débit dépassée. Patientez avant de refaire des requêtes.";
        default:
          return `Erreur : la requête d'API a échoué avec le statut ${error.response.status}`;
      }
    } else if (error.code === "ECONNABORTED") {
      return "Erreur : la requête a expiré. Réessayez.";
    }
  }
  return `Erreur : une erreur inattendue s'est produite : ${error instanceof Error ? error.message : String(error)}`;
}

// Créer l'instance du serveur MCP
const server = new McpServer({
  name: "example-mcp",
  version: "1.0.0"
});

// Enregistrer les outils
server.registerTool(
  "example_search_users",
  {
    title: "Rechercher des utilisateurs Example",
    description: `[Description complète comme ci-dessus]`,
    inputSchema: UserSearchInputSchema,
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async (params: UserSearchInput) => {
    // Implémentation comme ci-dessus
  }
);

// Fonction principale
// Pour stdio (local) :
async function runStdio() {
  if (!process.env.EXAMPLE_API_KEY) {
    console.error("ERREUR : la variable d'environnement EXAMPLE_API_KEY est obligatoire");
    process.exit(1);
  }

  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Serveur MCP lancé en stdio");
}

// Pour le HTTP en flux (distant) :
async function runHTTP() {
  if (!process.env.EXAMPLE_API_KEY) {
    console.error("ERREUR : la variable d'environnement EXAMPLE_API_KEY est obligatoire");
    process.exit(1);
  }

  const app = express();
  app.use(express.json());

  app.post('/mcp', async (req, res) => {
    const transport = new StreamableHTTPServerTransport({
      sessionIdGenerator: undefined,
      enableJsonResponse: true
    });
    res.on('close', () => transport.close());
    await server.connect(transport);
    await transport.handleRequest(req, res, req.body);
  });

  const port = parseInt(process.env.PORT || '3000');
  app.listen(port, () => {
    console.error(`Serveur MCP lancé sur http://localhost:${port}/mcp`);
  });
}

// Choisir le transport selon l'environnement
const transport = process.env.TRANSPORT || 'stdio';
if (transport === 'http') {
  runHTTP().catch(error => {
    console.error("Erreur du serveur :", error);
    process.exit(1);
  });
} else {
  runStdio().catch(error => {
    console.error("Erreur du serveur :", error);
    process.exit(1);
  });
}
```

---

## Fonctionnalités MCP avancées

### Enregistrement de ressources

Exposer des données comme ressources, pour un accès efficace par URI :

```typescript
import { ResourceTemplate } from "@modelcontextprotocol/sdk/types.js";

// Enregistrer une ressource avec un gabarit d'URI
server.registerResource(
  {
    uri: "file://documents/{name}",
    name: "Ressource de document",
    description: "Accéder aux documents par leur nom",
    mimeType: "text/plain"
  },
  async (uri: string) => {
    // Extraire le paramètre de l'URI
    const match = uri.match(/^file:\/\/documents\/(.+)$/);
    if (!match) {
      throw new Error("Format d'URI invalide");
    }

    const documentName = match[1];
    const content = await loadDocument(documentName);

    return {
      contents: [{
        uri,
        mimeType: "text/plain",
        text: content
      }]
    };
  }
);

// Lister dynamiquement les ressources disponibles
server.registerResourceList(async () => {
  const documents = await getAvailableDocuments();
  return {
    resources: documents.map(doc => ({
      uri: `file://documents/${doc.name}`,
      name: doc.name,
      mimeType: "text/plain",
      description: doc.description
    }))
  };
});
```

**Ressources ou outils :**
- **Ressources** : pour l'accès à des données avec des paramètres simples, par URI
- **Outils** : pour les opérations complexes qui demandent validation et logique métier
- **Ressources** : quand les données sont relativement statiques ou fondées sur des gabarits
- **Outils** : quand les opérations ont des effets de bord ou des enchaînements complexes

### Options de transport

Le SDK TypeScript prend en charge deux grands mécanismes de transport :

#### HTTP en flux (recommandé pour les serveurs distants)

```typescript
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";

const app = express();
app.use(express.json());

app.post('/mcp', async (req, res) => {
  // Créer un nouveau transport pour chaque requête (sans état, évite les collisions d'identifiants de requête)
  const transport = new StreamableHTTPServerTransport({
    sessionIdGenerator: undefined,
    enableJsonResponse: true
  });

  res.on('close', () => transport.close());

  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});

app.listen(3000);
```

#### stdio (pour les intégrations locales)

```typescript
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const transport = new StdioServerTransport();
await server.connect(transport);
```

**Choix du transport :**
- **HTTP en flux** : services web, accès distant, plusieurs clients
- **stdio** : outils en ligne de commande, développement local, intégration en sous-processus

### Prise en charge des notifications

Prévenir les clients quand l'état du serveur change :

```typescript
// Prévenir quand la liste des outils change
server.notification({
  method: "notifications/tools/list_changed"
});

// Prévenir quand les ressources changent
server.notification({
  method: "notifications/resources/list_changed"
});
```

Utiliser les notifications avec parcimonie — seulement quand les capacités du serveur changent réellement.

---

## Bonnes pratiques de code

### Composabilité et réutilisation du code

L'implémentation DOIT privilégier la composabilité et la réutilisation du code :

1. **Extraire les fonctionnalités communes** :
   - créer des fonctions d'aide réutilisables pour les opérations utilisées par plusieurs outils
   - construire des clients d'API partagés pour les requêtes HTTP au lieu de dupliquer le code
   - centraliser la logique de gestion des erreurs dans des fonctions utilitaires
   - extraire la logique métier dans des fonctions dédiées, composables
   - extraire la sélection de champs et la mise en forme Markdown ou JSON partagées

2. **Éviter la duplication** :
   - ne JAMAIS copier-coller du code semblable d'un outil à l'autre
   - si l'on écrit deux fois une logique semblable, l'extraire dans une fonction
   - les opérations courantes comme la pagination, le filtrage, la sélection de champs et la mise en forme doivent être partagées
   - la logique d'authentification et d'autorisation doit être centralisée

## Construire et lancer

Toujours compiler le code TypeScript avant de le lancer :

```bash
# Compiler le projet
npm run build

# Lancer le serveur
npm start

# Développement avec rechargement automatique
npm run dev
```

Toujours s'assurer que `npm run build` se termine sans erreur avant de considérer l'implémentation comme terminée.

## Liste de contrôle qualité

Avant de finaliser l'implémentation d'un serveur MCP Node/TypeScript, vérifier :

### Conception stratégique
- [ ] Les outils permettent des démarches complètes, pas seulement des enveloppes de points d'accès
- [ ] Les noms d'outils reflètent un découpage naturel des tâches
- [ ] Les formats de réponse optimisent l'usage du contexte de l'agent
- [ ] Des identifiants lisibles sont utilisés quand c'est approprié
- [ ] Les messages d'erreur orientent les agents vers un usage correct

### Qualité de l'implémentation
- [ ] IMPLÉMENTATION CIBLÉE : les outils les plus importants et les plus utiles sont implémentés
- [ ] Tous les outils sont enregistrés avec `registerTool` et une configuration complète
- [ ] Tous les outils incluent `title`, `description`, `inputSchema` et `annotations`
- [ ] Les annotations sont correctement définies (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)
- [ ] Tous les outils utilisent des schémas Zod pour valider les entrées à l'exécution, avec `.strict()`
- [ ] Tous les schémas Zod ont des contraintes appropriées et des messages d'erreur parlants
- [ ] Tous les outils ont des descriptions complètes avec des types d'entrée et de sortie explicites
- [ ] Les descriptions incluent des exemples de valeur de retour et la documentation complète du schéma
- [ ] Les messages d'erreur sont clairs, exploitables et pédagogiques

### Qualité TypeScript
- [ ] Des interfaces TypeScript sont définies pour toutes les structures de données
- [ ] Le mode strict est activé dans tsconfig.json
- [ ] Aucun usage du type `any` — utiliser `unknown` ou des types appropriés
- [ ] Toutes les fonctions asynchrones ont un type de retour `Promise<T>` explicite
- [ ] La gestion des erreurs utilise des gardes de type appropriées (par exemple `axios.isAxiosError`, `z.ZodError`)

### Fonctionnalités avancées (le cas échéant)
- [ ] Ressources enregistrées pour les points d'accès aux données appropriés
- [ ] Transport approprié configuré (stdio ou HTTP en flux)
- [ ] Notifications implémentées pour les capacités dynamiques du serveur
- [ ] Typage sûr avec les interfaces du SDK

### Configuration du projet
- [ ] package.json inclut toutes les dépendances nécessaires
- [ ] Le script de compilation produit du JavaScript fonctionnel dans le répertoire dist/
- [ ] Le point d'entrée principal est correctement configuré sur dist/index.js
- [ ] Le nom du serveur suit le format `{service}-mcp-server`
- [ ] tsconfig.json est correctement configuré en mode strict

### Qualité du code
- [ ] La pagination est correctement implémentée là où elle s'applique
- [ ] Les réponses volumineuses vérifient la constante `CHARACTER_LIMIT` et sont tronquées avec un message clair
- [ ] Des options de filtrage sont proposées pour les résultats potentiellement volumineux
- [ ] Toutes les opérations réseau gèrent proprement les délais d'expiration et les erreurs de connexion
- [ ] Les fonctionnalités communes sont extraites dans des fonctions réutilisables
- [ ] Les types de retour sont cohérents entre opérations semblables

### Tests et compilation
- [ ] `npm run build` se termine sans erreur
- [ ] dist/index.js est créé et exécutable
- [ ] Le serveur se lance : `node dist/index.js --help`
- [ ] Toutes les importations se résolvent
- [ ] Des appels d'outils d'exemple fonctionnent comme prévu
