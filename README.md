# Skills d'Anthropic, en français

Traduction française du dépôt [anthropics/skills](https://github.com/anthropics/skills) : les skills d'exemple qu'Anthropic publie pour Claude. Ce dépôt est une **bifurcation (fork)** ; il suit l'original et ne le remplace pas. Pour la norme Agent Skills elle-même, voir [agentskills.io](https://agentskills.io).

> **Avertissement repris de l'original.** Ces skills sont fournies **à des fins de démonstration et d'apprentissage**. Le comportement de Claude peut différer de ce qu'elles montrent. Les éprouver dans son propre environnement avant de s'y fier pour une tâche importante.

## Qu'est-ce qu'une skill ?

Une skill est un dossier d'instructions, de scripts et de ressources que Claude charge **à la demande** pour mieux accomplir une tâche spécialisée : produire un document aux couleurs d'une entreprise, suivre une procédure interne, tester une application web…

Le cœur d'une skill est un fichier `SKILL.md` :

```markdown
---
name: ma-skill
description: Ce que fait la skill et QUAND l'utiliser.
---

# Ma skill

Les instructions que Claude suit quand la skill est active.
```

Trois choses à comprendre pour lire les skills de ce dépôt :

1. **Le chargement est progressif.** Claude ne garde en permanence que le `name` et la `description` de chaque skill. Il ne lit le corps de `SKILL.md` que lorsque la tâche correspond, puis les fichiers annexes (`references/`, `scripts/`, `assets/`) seulement s'il en a besoin. C'est ce qui permet d'installer des dizaines de skills sans encombrer le contexte.
2. **La `description` décide du déclenchement.** C'est elle que Claude compare à la demande. Une description vague déclenche mal ; une bonne dit à la fois ce que fait la skill et dans quels cas s'en servir.
3. **Le `name` est un identifiant.** En minuscules, avec des tirets, et il doit correspondre au nom du dossier. Il n'est **jamais traduit** ici : les commandes, les références croisées et le marketplace en dépendent.

La spécification complète, traduite, est dans [`spec/agent-skills-spec.md`](./spec/agent-skills-spec.md) ; un gabarit de départ dans [`template/SKILL.md`](./template/SKILL.md). Pour aller plus loin : [Qu'est-ce qu'une skill ?](https://support.claude.com/en/articles/12512176-what-are-skills), [Créer des skills personnalisées](https://support.claude.com/en/articles/12512198-creating-custom-skills) (pages en anglais).

## Les skills du dépôt

### Création et design

| Skill | Ce qu'elle fait |
|---|---|
| [`algorithmic-art`](./skills/algorithmic-art) | Art génératif avec p5.js : une « philosophie » algorithmique, puis une œuvre interactive à graine reproductible. |
| [`canvas-design`](./skills/canvas-design) | Affiches et visuels statiques (PNG, PDF) à partir d'une philosophie de design. |
| [`frontend-design`](./skills/frontend-design) | Interfaces web soignées et singulières, qui évitent l'esthétique « générée par IA ». |
| [`theme-factory`](./skills/theme-factory) | Dix thèmes prêts à l'emploi (couleurs et polices) à appliquer à des diapositives, documents ou pages, ou un thème créé sur mesure. |
| [`brand-guidelines`](./skills/brand-guidelines) | Applique la charte graphique d'Anthropic ; sert de modèle pour écrire celle de sa propre organisation. |
| [`slack-gif-creator`](./skills/slack-gif-creator) | GIF animés optimisés pour Slack (contraintes de taille, outils de validation). |

### Développement

| Skill | Ce qu'elle fait |
|---|---|
| [`claude-api`](./skills/claude-api) | Référence de l'API Claude et des SDK : modèles, tarifs, réflexion, cache, outils, agents, migrations. |
| [`mcp-builder`](./skills/mcp-builder) | Concevoir et écrire un serveur MCP de qualité (Python ou TypeScript), puis l'évaluer. |
| [`web-artifacts-builder`](./skills/web-artifacts-builder) | Artefacts HTML élaborés (React, Tailwind, shadcn/ui) regroupés en un seul fichier. |
| [`webapp-testing`](./skills/webapp-testing) | Tester une application web locale avec Playwright : captures, journaux, interactions. |
| [`skill-creator`](./skills/skill-creator) | Créer, améliorer et mesurer une skill : évaluations, banc comparatif, optimisation de la description. |

### Entreprise et communication

| Skill | Ce qu'elle fait |
|---|---|
| [`internal-comms`](./skills/internal-comms) | Communications internes : points d'avancement, lettres d'information, FAQ, rapports d'incident. |
| [`doc-coauthoring`](./skills/doc-coauthoring) | Rédiger à deux un document (spécification, proposition, note de décision) en trois étapes guidées. |
| [`academy-guide`](./skills/academy-guide) | Oriente vers les cours et tutoriels de la Claude Academy quand on demande comment utiliser Claude. |
| [`discernment-nudge`](./skills/discernment-nudge) | Après une réponse de fond, ajoute deux ou trois questions pour vérifier les faits et éprouver le raisonnement. |

### Documents (non traduites)

| Skill | Ce qu'elle fait |
|---|---|
| [`docx`](./skills/docx) | Créer, lire et modifier des documents Word, y compris les modifications suivies et les commentaires. |
| [`pdf`](./skills/pdf) | Extraire texte et tableaux, fusionner, découper, remplir des formulaires, créer des PDF. |
| [`pptx`](./skills/pptx) | Créer et modifier des présentations PowerPoint, à partir de zéro ou d'un modèle. |
| [`xlsx`](./skills/xlsx) | Créer et modifier des classeurs Excel : formules, mise en forme, graphiques, nettoyage de données. |

Ces quatre skills sont celles qui alimentent les fonctions documentaires de Claude. Elles sont **à source visible mais propriétaires** (« © 2025 Anthropic, PBC. All rights reserved ») : leur licence ne permet pas d'en publier une version modifiée, donc elles restent exactement telles qu'Anthropic les publie, en anglais. Les descriptions ci-dessus sont rédigées pour ce README, elles ne traduisent pas leurs fichiers.

## Ce qui est traduit, et ce qui ne l'est pas

**Traduit en français :** les `SKILL.md` de toutes les skills sous licence Apache 2.0, leurs exemples et leurs fichiers de référence rédigés (par exemple `mcp-builder/reference/`, `skill-creator/agents/` et `skill-creator/references/schemas.md`, les dix thèmes de `theme-factory`), la spécification et le gabarit.

**Laissé tel quel, et pourquoi :**

- **Les identifiants.** Champ `name`, noms de fichiers et de dossiers, clés et valeurs d'énumération JSON, noms d'API, de modèles et d'en-têtes bêta, noms des thèmes. Des scripts et des visionneuses les lisent ; les traduire les casserait.
- **Le code des scripts** (`scripts/`). Seuls les commentaires et les chaînes affichées dans les fichiers Markdown ont été traduits ; les programmes eux-mêmes sont intacts.
- **L'interface de la visionneuse de `skill-creator`.** Elle reste en anglais (« Outputs », « Benchmark », « Submit All Reviews ») ; la traduction de `SKILL.md` le signale là où ces libellés sont cités.
- **Les fichiers de référence de `claude-api`** (`shared/`, `python/`, `typescript/`…). Environ 245 000 mots d'une documentation d'API qui change toutes les semaines : une traduction serait périmée avant d'être finie, et une erreur sur un paramètre y coûterait plus cher qu'un texte en anglais. Seul son `SKILL.md`, qui oriente vers ces fichiers, est traduit.
- **Les quatre skills documentaires**, pour la raison de licence donnée plus haut.
- **Les textes juridiques** : chaque `LICENSE.txt` et `THIRD_PARTY_NOTICES.md`. Une licence ne se traduit pas : seule la version d'origine fait foi.

## Installation

### Dans Claude Code

Déclarer ce dépôt comme marketplace de plugins :

```
/plugin marketplace add FabriceFx/skills-fr
```

Puis installer un ensemble de skills :

```
/plugin install example-skills@anthropic-agent-skills-fr
/plugin install claude-api@anthropic-agent-skills-fr
```

Ensembles disponibles : `example-skills`, `claude-api`, `academy-guide`, `discernment-nudge`, et `document-skills` (en anglais).

Le marketplace s'appelle `anthropic-agent-skills-fr` pour ne pas entrer en collision avec l'officiel, `anthropic-agent-skills`. **N'installer qu'une version d'une même skill** : avoir à la fois la version anglaise et la française donne à Claude deux skills de même `name`, et l'on ne sait plus laquelle se déclenche.

Une fois installée, une skill s'utilise en la mentionnant ou simplement en formulant une demande qui correspond à sa description, par exemple : « Crée un serveur MCP pour l'API de mon agenda ».

### Dans claude.ai ou par l'API

Sur claude.ai, une skill se téléverse sous forme d'archive ZIP de son dossier : voir [Utiliser des skills dans Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude). Par l'API, voir le [guide de l'API Skills](https://docs.claude.com/en/api/skills-guide) (pages en anglais).

## Rester à jour avec l'original

Le dépôt d'Anthropic évolue. Pour récupérer ses nouveautés :

```bash
git fetch upstream
git log --oneline main..upstream/main   # ce qui a changé
git merge upstream/main                 # conflits attendus sur les fichiers traduits
```

Un conflit sur un fichier traduit signale un texte d'origine modifié : relire la différence anglaise (`git diff <ancien>..upstream/main -- chemin`) et reporter le changement dans la traduction, plutôt que d'accepter l'une ou l'autre version en bloc.

## Licences

Ce dépôt n'ajoute aucune licence : **celles de l'original s'appliquent**. Les skills sous Apache 2.0 portent leur `LICENSE.txt` ; les traductions en sont des œuvres dérivées sous la même licence. Les skills `docx`, `pdf`, `pptx` et `xlsx` relèvent de la licence propriétaire d'Anthropic décrite dans leur `LICENSE.txt`. Seule `doc-coauthoring` n'a pas de fichier de licence dans l'original, et le dépôt d'Anthropic n'en a pas à sa racine non plus : son statut est celui que lui donne Anthropic, et cette traduction n'en revendique aucun autre. Voir aussi `THIRD_PARTY_NOTICES.md`.

Traduction : Fabrice Faucheux ([faucheux.bzh](https://faucheux.bzh)). Ce dépôt n'est ni produit ni approuvé par Anthropic.
