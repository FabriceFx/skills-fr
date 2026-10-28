# Guide d'implémentation d'un serveur MCP en Python

## Présentation

Ce document rassemble les bonnes pratiques et les exemples propres à Python pour implémenter des serveurs MCP avec le SDK MCP Python. Il couvre la mise en place du serveur, les motifs d'enregistrement des outils, la validation des entrées avec Pydantic, la gestion des erreurs et des exemples complets fonctionnels.

---

## Aide-mémoire

### Importations clés
```python
from mcp.server.fastmcp import FastMCP
from pydantic import BaseModel, Field, field_validator, ConfigDict
from typing import Optional, List, Dict, Any
from enum import Enum
import httpx
```

### Initialisation du serveur
```python
mcp = FastMCP("service_mcp")
```

### Motif d'enregistrement d'un outil
```python
@mcp.tool(name="tool_name", annotations={...})
async def tool_function(params: InputModel) -> str:
    # Implémentation
    pass
```

---

## Le SDK MCP Python et FastMCP

Le SDK MCP Python officiel fournit FastMCP, un framework de haut niveau pour construire des serveurs MCP. Il offre :
- la génération automatique de la description et de l'`inputSchema` à partir des signatures de fonction et des docstrings
- l'intégration des modèles Pydantic pour la validation des entrées
- l'enregistrement des outils par décorateur avec `@mcp.tool`

**Pour la documentation complète du SDK, charger avec WebFetch :**
`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`

## Convention de nommage du serveur

Les serveurs MCP Python doivent suivre ce modèle de nommage :
- **Format** : `{service}_mcp` (minuscules avec tirets bas)
- **Exemples** : `github_mcp`, `jira_mcp`, `stripe_mcp`

Le nom doit être :
- général (non lié à des fonctionnalités précises)
- descriptif du service ou de l'API intégrés
- facile à déduire de la description de la tâche
- sans numéro de version ni date

## Implémentation des outils

### Nommage des outils

Utiliser le snake_case pour les noms d'outils (par exemple « search_users », « create_project », « get_channel_info »), avec des noms clairs et orientés action.

**Éviter les conflits de noms** : inclure le contexte du service pour éviter les chevauchements :
- utiliser « slack_send_message » plutôt que simplement « send_message »
- utiliser « github_create_issue » plutôt que simplement « create_issue »
- utiliser « asana_list_tasks » plutôt que simplement « list_tasks »

### Structure d'un outil avec FastMCP

Les outils se définissent avec le décorateur `@mcp.tool` et des modèles Pydantic pour valider les entrées :

```python
from pydantic import BaseModel, Field, ConfigDict
from mcp.server.fastmcp import FastMCP

# Initialiser le serveur MCP
mcp = FastMCP("example_mcp")

# Définir le modèle Pydantic de validation des entrées
class ServiceToolInput(BaseModel):
    '''Modèle d'entrée de l'opération de l'outil.'''
    model_config = ConfigDict(
        str_strip_whitespace=True,  # Retirer automatiquement les espaces des chaînes
        validate_assignment=True,    # Valider à l'affectation
        extra='forbid'              # Interdire les champs supplémentaires
    )

    param1: str = Field(..., description="Description du premier paramètre (par ex. 'user123', 'project-abc')", min_length=1, max_length=100)
    param2: Optional[int] = Field(default=None, description="Paramètre entier facultatif avec contraintes", ge=0, le=1000)
    tags: Optional[List[str]] = Field(default_factory=list, description="Liste des étiquettes à appliquer", max_items=10)

@mcp.tool(
    name="service_tool_name",
    annotations={
        "title": "Titre lisible de l'outil",
        "readOnlyHint": True,     # L'outil ne modifie pas l'environnement
        "destructiveHint": False,  # L'outil ne fait pas d'opérations destructives
        "idempotentHint": True,    # Des appels répétés n'ont pas d'effet supplémentaire
        "openWorldHint": False     # L'outil n'interagit pas avec des entités externes
    }
)
async def service_tool_name(params: ServiceToolInput) -> str:
    '''La docstring de l'outil devient automatiquement le champ 'description'.

    Cet outil effectue une opération précise sur le service. Il valide toutes
    les entrées avec le modèle Pydantic ServiceToolInput avant le traitement.

    Args:
        params (ServiceToolInput): paramètres d'entrée validés, contenant :
            - param1 (str) : description du premier paramètre
            - param2 (Optional[int]) : paramètre facultatif avec valeur par défaut
            - tags (Optional[List[str]]) : liste d'étiquettes

    Returns:
        str: réponse au format JSON contenant les résultats de l'opération
    '''
    # Implémentation ici
    pass
```

## Points clés de Pydantic v2

- Utiliser `model_config` au lieu d'une classe `Config` imbriquée
- Utiliser `field_validator` au lieu de `validator`, déprécié
- Utiliser `model_dump()` au lieu de `dict()`, déprécié
- Les validateurs exigent le décorateur `@classmethod`
- Les annotations de type sont obligatoires sur les méthodes de validation

```python
from pydantic import BaseModel, Field, field_validator, ConfigDict

class CreateUserInput(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True
    )

    name: str = Field(..., description="Nom complet de l'utilisateur", min_length=1, max_length=100)
    email: str = Field(..., description="Adresse électronique de l'utilisateur", pattern=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    age: int = Field(..., description="Âge de l'utilisateur", ge=0, le=150)

    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("L'adresse électronique ne peut pas être vide")
        return v.lower()
```

## Options de format de réponse

Prendre en charge plusieurs formats de sortie, pour la souplesse :

```python
from enum import Enum

class ResponseFormat(str, Enum):
    '''Format de sortie des réponses des outils.'''
    MARKDOWN = "markdown"
    JSON = "json"

class UserSearchInput(BaseModel):
    query: str = Field(..., description="Requête de recherche")
    response_format: ResponseFormat = Field(
        default=ResponseFormat.MARKDOWN,
        description="Format de sortie : 'markdown' lisible par un humain, ou 'json' lisible par une machine"
    )
```

**Format Markdown** :
- utiliser titres, listes et mise en forme pour la clarté
- convertir les horodatages en format lisible (par exemple « 2024-01-15 10:30:00 UTC » plutôt qu'un temps Unix)
- afficher les noms avec les identifiants entre parenthèses (par exemple « @john.doe (U123456) »)
- omettre les métadonnées verbeuses (par exemple n'afficher qu'une URL d'image de profil, pas toutes les tailles)
- regrouper logiquement les informations liées

**Format JSON** :
- renvoyer des données complètes et structurées, adaptées au traitement par programme
- inclure tous les champs et métadonnées disponibles
- utiliser des noms et types de champs cohérents

## Implémentation de la pagination

Pour les outils qui listent des ressources :

```python
class ListInput(BaseModel):
    limit: Optional[int] = Field(default=20, description="Nombre maximal de résultats à renvoyer", ge=1, le=100)
    offset: Optional[int] = Field(default=0, description="Nombre de résultats à sauter pour la pagination", ge=0)

async def list_items(params: ListInput) -> str:
    # Faire la requête d'API avec pagination
    data = await api_request(limit=params.limit, offset=params.offset)

    # Renvoyer les informations de pagination
    response = {
        "total": data["total"],
        "count": len(data["items"]),
        "offset": params.offset,
        "items": data["items"],
        "has_more": data["total"] > params.offset + len(data["items"]),
        "next_offset": params.offset + len(data["items"]) if data["total"] > params.offset + len(data["items"]) else None
    }
    return json.dumps(response, indent=2)
```

## Gestion des erreurs

Fournir des messages d'erreur clairs et exploitables :

```python
def _handle_api_error(e: Exception) -> str:
    '''Mise en forme cohérente des erreurs pour tous les outils.'''
    if isinstance(e, httpx.HTTPStatusError):
        if e.response.status_code == 404:
            return "Erreur : ressource introuvable. Vérifiez que l'identifiant est correct."
        elif e.response.status_code == 403:
            return "Erreur : accès refusé. Vous n'avez pas accès à cette ressource."
        elif e.response.status_code == 429:
            return "Erreur : limite de débit dépassée. Patientez avant de refaire des requêtes."
        return f"Erreur : la requête d'API a échoué avec le statut {e.response.status_code}"
    elif isinstance(e, httpx.TimeoutException):
        return "Erreur : la requête a expiré. Réessayez."
    return f"Erreur : une erreur inattendue s'est produite : {type(e).__name__}"
```

## Utilitaires partagés

Extraire les fonctionnalités communes dans des fonctions réutilisables :

```python
# Fonction partagée de requête d'API
async def _make_api_request(endpoint: str, method: str = "GET", **kwargs) -> dict:
    '''Fonction réutilisable pour tous les appels d'API.'''
    async with httpx.AsyncClient() as client:
        response = await client.request(
            method,
            f"{API_BASE_URL}/{endpoint}",
            timeout=30.0,
            **kwargs
        )
        response.raise_for_status()
        return response.json()
```

## Bonnes pratiques async/await

Toujours utiliser async/await pour les requêtes réseau et les entrées-sorties :

```python
# Bien : requête réseau asynchrone
async def fetch_data(resource_id: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"{API_URL}/resource/{resource_id}")
        response.raise_for_status()
        return response.json()

# Mal : requête synchrone
def fetch_data(resource_id: str) -> dict:
    response = requests.get(f"{API_URL}/resource/{resource_id}")  # Bloquant
    return response.json()
```

## Annotations de type

Utiliser des annotations de type partout :

```python
from typing import Optional, List, Dict, Any

async def get_user(user_id: str) -> Dict[str, Any]:
    data = await fetch_user(user_id)
    return {"id": data["id"], "name": data["name"]}
```

## Docstrings des outils

Chaque outil doit avoir une docstring complète, avec des informations de type explicites :

```python
async def search_users(params: UserSearchInput) -> str:
    '''
    Rechercher des utilisateurs dans le système Example par nom, adresse ou équipe.

    Cet outil cherche dans tous les profils d'utilisateurs de la plateforme Example,
    avec prise en charge des correspondances partielles et de divers filtres. Il NE
    crée NI ne modifie d'utilisateurs : il cherche seulement parmi ceux qui existent.

    Args:
        params (UserSearchInput): paramètres d'entrée validés, contenant :
            - query (str) : chaîne à rechercher dans les noms et adresses (par ex. "john", "@example.com", "team:marketing")
            - limit (Optional[int]) : nombre maximal de résultats, entre 1 et 100 (défaut : 20)
            - offset (Optional[int]) : nombre de résultats à sauter pour la pagination (défaut : 0)

    Returns:
        str: chaîne au format JSON contenant les résultats, selon le schéma suivant :

        Réponse en cas de succès :
        {
            "total": int,           # Nombre total de correspondances trouvées
            "count": int,           # Nombre de résultats dans cette réponse
            "offset": int,          # Décalage de pagination courant
            "users": [
                {
                    "id": str,      # Identifiant de l'utilisateur (par ex. "U123456789")
                    "name": str,    # Nom complet (par ex. "John Doe")
                    "email": str,   # Adresse électronique (par ex. "john@example.com")
                    "team": str     # Nom de l'équipe (par ex. "Marketing") — facultatif
                }
            ]
        }

        Réponse en cas d'erreur :
        "Erreur : <message d'erreur>" ou "Aucun utilisateur ne correspond à '<requête>'"

    Exemples :
        - À utiliser pour : « Trouver tous les membres de l'équipe marketing » -> params avec query="team:marketing"
        - À utiliser pour : « Chercher le compte de John » -> params avec query="john"
        - À ne pas utiliser pour : créer un utilisateur (utiliser example_create_user)
        - À ne pas utiliser pour : obtenir tous les détails d'un utilisateur dont on a l'identifiant (utiliser example_get_user)

    Gestion des erreurs :
        - Les erreurs de validation des entrées sont gérées par le modèle Pydantic
        - Renvoie "Erreur : limite de débit dépassée" en cas de requêtes trop nombreuses (statut 429)
        - Renvoie "Erreur : authentification d'API invalide" si la clé d'API est invalide (statut 401)
        - Renvoie la liste mise en forme des résultats, ou "Aucun utilisateur ne correspond à 'requête'"
    '''
```

## Exemple complet

Voici un exemple complet de serveur MCP en Python :

```python
#!/usr/bin/env python3
'''
Serveur MCP pour le service Example.

Ce serveur fournit des outils pour interagir avec l'API Example : recherche
d'utilisateurs, gestion de projets et export de données.
'''

from typing import Optional, List, Dict, Any
from enum import Enum
import httpx
from pydantic import BaseModel, Field, field_validator, ConfigDict
from mcp.server.fastmcp import FastMCP

# Initialiser le serveur MCP
mcp = FastMCP("example_mcp")

# Constantes
API_BASE_URL = "https://api.example.com/v1"

# Énumérations
class ResponseFormat(str, Enum):
    '''Format de sortie des réponses des outils.'''
    MARKDOWN = "markdown"
    JSON = "json"

# Modèles Pydantic de validation des entrées
class UserSearchInput(BaseModel):
    '''Modèle d'entrée des opérations de recherche d'utilisateurs.'''
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True
    )

    query: str = Field(..., description="Chaîne à rechercher dans les noms et adresses", min_length=2, max_length=200)
    limit: Optional[int] = Field(default=20, description="Nombre maximal de résultats à renvoyer", ge=1, le=100)
    offset: Optional[int] = Field(default=0, description="Nombre de résultats à sauter pour la pagination", ge=0)
    response_format: ResponseFormat = Field(default=ResponseFormat.MARKDOWN, description="Format de sortie")

    @field_validator('query')
    @classmethod
    def validate_query(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("La requête ne peut pas être vide ni faite seulement d'espaces")
        return v.strip()

# Fonctions utilitaires partagées
async def _make_api_request(endpoint: str, method: str = "GET", **kwargs) -> dict:
    '''Fonction réutilisable pour tous les appels d'API.'''
    async with httpx.AsyncClient() as client:
        response = await client.request(
            method,
            f"{API_BASE_URL}/{endpoint}",
            timeout=30.0,
            **kwargs
        )
        response.raise_for_status()
        return response.json()

def _handle_api_error(e: Exception) -> str:
    '''Mise en forme cohérente des erreurs pour tous les outils.'''
    if isinstance(e, httpx.HTTPStatusError):
        if e.response.status_code == 404:
            return "Erreur : ressource introuvable. Vérifiez que l'identifiant est correct."
        elif e.response.status_code == 403:
            return "Erreur : accès refusé. Vous n'avez pas accès à cette ressource."
        elif e.response.status_code == 429:
            return "Erreur : limite de débit dépassée. Patientez avant de refaire des requêtes."
        return f"Erreur : la requête d'API a échoué avec le statut {e.response.status_code}"
    elif isinstance(e, httpx.TimeoutException):
        return "Erreur : la requête a expiré. Réessayez."
    return f"Erreur : une erreur inattendue s'est produite : {type(e).__name__}"

# Définition des outils
@mcp.tool(
    name="example_search_users",
    annotations={
        "title": "Rechercher des utilisateurs Example",
        "readOnlyHint": True,
        "destructiveHint": False,
        "idempotentHint": True,
        "openWorldHint": True
    }
)
async def example_search_users(params: UserSearchInput) -> str:
    '''Rechercher des utilisateurs dans le système Example par nom, adresse ou équipe.

    [Docstring complète comme ci-dessus]
    '''
    try:
        # Faire la requête d'API avec les paramètres validés
        data = await _make_api_request(
            "users/search",
            params={
                "q": params.query,
                "limit": params.limit,
                "offset": params.offset
            }
        )

        users = data.get("users", [])
        total = data.get("total", 0)

        if not users:
            return f"Aucun utilisateur ne correspond à '{params.query}'"

        # Mettre en forme la réponse selon le format demandé
        if params.response_format == ResponseFormat.MARKDOWN:
            lines = [f"# Résultats de la recherche d'utilisateurs : '{params.query}'", ""]
            lines.append(f"{total} utilisateurs trouvés ({len(users)} affichés)")
            lines.append("")

            for user in users:
                lines.append(f"## {user['name']} ({user['id']})")
                lines.append(f"- **Adresse** : {user['email']}")
                if user.get('team'):
                    lines.append(f"- **Équipe** : {user['team']}")
                lines.append("")

            return "\n".join(lines)

        else:
            # Format JSON lisible par une machine
            import json
            response = {
                "total": total,
                "count": len(users),
                "offset": params.offset,
                "users": users
            }
            return json.dumps(response, indent=2)

    except Exception as e:
        return _handle_api_error(e)

if __name__ == "__main__":
    mcp.run()
```

---

## Fonctionnalités avancées de FastMCP

### Injection du paramètre Context

FastMCP peut injecter automatiquement un paramètre `Context` dans les outils, pour des capacités avancées comme la journalisation, le suivi de progression, la lecture de ressources et l'interaction avec l'utilisateur :

```python
from mcp.server.fastmcp import FastMCP, Context

mcp = FastMCP("example_mcp")

@mcp.tool()
async def advanced_search(query: str, ctx: Context) -> str:
    '''Outil avancé avec accès au contexte pour la journalisation et la progression.'''

    # Signaler la progression des opérations longues
    await ctx.report_progress(0.25, "Début de la recherche...")

    # Journaliser des informations pour le débogage
    await ctx.log_info("Traitement de la requête", {"query": query, "timestamp": datetime.now()})

    # Effectuer la recherche
    results = await search_api(query)
    await ctx.report_progress(0.75, "Mise en forme des résultats...")

    # Accéder à la configuration du serveur
    server_name = ctx.fastmcp.name

    return format_results(results)

@mcp.tool()
async def interactive_tool(resource_id: str, ctx: Context) -> str:
    '''Outil qui peut demander des informations supplémentaires à l'utilisateur.'''

    # Demander une information sensible au besoin
    api_key = await ctx.elicit(
        prompt="Veuillez fournir votre clé d'API :",
        input_type="password"
    )

    # Utiliser la clé fournie
    return await api_call(resource_id, api_key)
```

**Capacités du contexte :**
- `ctx.report_progress(progress, message)` — signaler la progression des opérations longues
- `ctx.log_info(message, data)` / `ctx.log_error()` / `ctx.log_debug()` — journalisation
- `ctx.elicit(prompt, input_type)` — demander une saisie à l'utilisateur
- `ctx.fastmcp.name` — accéder à la configuration du serveur
- `ctx.read_resource(uri)` — lire des ressources MCP

### Enregistrement de ressources

Exposer des données comme ressources, pour un accès efficace par gabarit d'URI :

```python
@mcp.resource("file://documents/{name}")
async def get_document(name: str) -> str:
    '''Exposer des documents comme ressources MCP.

    Les ressources conviennent aux données statiques ou semi-statiques qui ne
    demandent pas de paramètres complexes. Elles utilisent des gabarits d'URI
    pour un accès souple.
    '''
    document_path = f"./docs/{name}"
    with open(document_path, "r") as f:
        return f.read()

@mcp.resource("config://settings/{key}")
async def get_setting(key: str, ctx: Context) -> str:
    '''Exposer la configuration comme ressources, avec contexte.'''
    settings = await load_settings()
    return json.dumps(settings.get(key, {}))
```

**Ressources ou outils :**
- **Ressources** : pour l'accès à des données avec des paramètres simples (gabarits d'URI)
- **Outils** : pour les opérations complexes avec validation et logique métier

### Types de sortie structurés

FastMCP prend en charge d'autres types de retour que les chaînes :

```python
from typing import TypedDict
from dataclasses import dataclass
from pydantic import BaseModel

# TypedDict pour des retours structurés
class UserData(TypedDict):
    id: str
    name: str
    email: str

@mcp.tool()
async def get_user_typed(user_id: str) -> UserData:
    '''Renvoie des données structurées — FastMCP gère la sérialisation.'''
    return {"id": user_id, "name": "John Doe", "email": "john@example.com"}

# Modèles Pydantic pour une validation complexe
class DetailedUser(BaseModel):
    id: str
    name: str
    email: str
    created_at: datetime
    metadata: Dict[str, Any]

@mcp.tool()
async def get_user_detailed(user_id: str) -> DetailedUser:
    '''Renvoie un modèle Pydantic — le schéma est généré automatiquement.'''
    user = await fetch_user(user_id)
    return DetailedUser(**user)
```

### Gestion du cycle de vie

Initialiser des ressources qui persistent d'une requête à l'autre :

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def app_lifespan():
    '''Gérer les ressources qui vivent aussi longtemps que le serveur.'''
    # Initialiser les connexions, charger la configuration, etc.
    db = await connect_to_database()
    config = load_configuration()

    # Les rendre disponibles à tous les outils
    yield {"db": db, "config": config}

    # Nettoyage à l'arrêt
    await db.close()

mcp = FastMCP("example_mcp", lifespan=app_lifespan)

@mcp.tool()
async def query_data(query: str, ctx: Context) -> str:
    '''Accéder aux ressources du cycle de vie par le contexte.'''
    db = ctx.request_context.lifespan_state["db"]
    results = await db.query(query)
    return format_results(results)
```

### Options de transport

FastMCP prend en charge deux grands mécanismes de transport :

```python
# Transport stdio (outils locaux) — par défaut
if __name__ == "__main__":
    mcp.run()

# Transport HTTP en flux (serveurs distants)
if __name__ == "__main__":
    mcp.run(transport="streamable_http", port=8000)
```

**Choix du transport :**
- **stdio** : outils en ligne de commande, intégrations locales, exécution en sous-processus
- **HTTP en flux** : services web, accès distant, plusieurs clients

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

### Bonnes pratiques propres à Python

1. **Annotations de type** : toujours annoter les paramètres et les valeurs de retour des fonctions
2. **Modèles Pydantic** : définir des modèles Pydantic clairs pour toute validation d'entrée
3. **Pas de validation manuelle** : laisser Pydantic valider les entrées par des contraintes
4. **Importations correctes** : regrouper les importations (bibliothèque standard, tierces, locales)
5. **Gestion des erreurs** : utiliser des types d'exception précis (`httpx.HTTPStatusError`, pas `Exception` générique)
6. **Gestionnaires de contexte asynchrones** : utiliser `async with` pour les ressources à libérer
7. **Constantes** : définir les constantes de module en MAJUSCULES

## Liste de contrôle qualité

Avant de finaliser l'implémentation d'un serveur MCP Python, vérifier :

### Conception stratégique
- [ ] Les outils permettent des démarches complètes, pas seulement des enveloppes de points d'accès
- [ ] Les noms d'outils reflètent un découpage naturel des tâches
- [ ] Les formats de réponse optimisent l'usage du contexte de l'agent
- [ ] Des identifiants lisibles sont utilisés quand c'est approprié
- [ ] Les messages d'erreur orientent les agents vers un usage correct

### Qualité de l'implémentation
- [ ] IMPLÉMENTATION CIBLÉE : les outils les plus importants et les plus utiles sont implémentés
- [ ] Tous les outils ont des noms parlants et une documentation
- [ ] Les types de retour sont cohérents entre opérations semblables
- [ ] La gestion des erreurs est implémentée pour tous les appels externes
- [ ] Le nom du serveur suit le format `{service}_mcp`
- [ ] Toutes les opérations réseau utilisent async/await
- [ ] Les fonctionnalités communes sont extraites dans des fonctions réutilisables
- [ ] Les messages d'erreur sont clairs, exploitables et pédagogiques
- [ ] Les sorties sont correctement validées et mises en forme

### Configuration des outils
- [ ] Tous les outils déclarent 'name' et 'annotations' dans le décorateur
- [ ] Les annotations sont correctement définies (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)
- [ ] Tous les outils utilisent un `BaseModel` Pydantic pour valider les entrées, avec des définitions `Field()`
- [ ] Tous les `Field` Pydantic ont des types explicites et des descriptions avec contraintes
- [ ] Tous les outils ont des docstrings complètes avec des types d'entrée et de sortie explicites
- [ ] Les docstrings incluent la structure complète du schéma pour les retours dict / JSON
- [ ] Les modèles Pydantic gèrent la validation des entrées (aucune validation manuelle nécessaire)

### Fonctionnalités avancées (le cas échéant)
- [ ] Injection du contexte utilisée pour la journalisation, la progression ou la demande de saisie
- [ ] Ressources enregistrées pour les points d'accès aux données appropriés
- [ ] Gestion du cycle de vie implémentée pour les connexions persistantes
- [ ] Types de sortie structurés utilisés (TypedDict, modèles Pydantic)
- [ ] Transport approprié configuré (stdio ou HTTP en flux)

### Qualité du code
- [ ] Le fichier comporte les importations nécessaires, dont celles de Pydantic
- [ ] La pagination est correctement implémentée là où elle s'applique
- [ ] Des options de filtrage sont proposées pour les résultats potentiellement volumineux
- [ ] Toutes les fonctions asynchrones sont correctement définies avec `async def`
- [ ] L'usage du client HTTP suit les motifs asynchrones avec des gestionnaires de contexte appropriés
- [ ] Les annotations de type sont utilisées dans tout le code
- [ ] Les constantes sont définies au niveau du module, en MAJUSCULES

### Tests
- [ ] Le serveur se lance correctement : `python your_server.py --help`
- [ ] Toutes les importations se résolvent
- [ ] Des appels d'outils d'exemple fonctionnent comme prévu
- [ ] Les scénarios d'erreur sont gérés proprement
