---
name: skill-creator
description: Créer de nouvelles skills, modifier et améliorer des skills existantes, et mesurer leurs performances. À utiliser quand l'utilisateur veut créer une skill à partir de rien, modifier ou optimiser une skill existante, lancer des évaluations pour tester une skill, mesurer ses performances avec une analyse de variance, ou optimiser sa description pour qu'elle se déclenche plus justement.
---

# Créateur de skills

Une skill pour créer de nouvelles skills et les améliorer par itérations.

Dans les grandes lignes, la création d'une skill se déroule ainsi :

- Décider ce que la skill doit faire et, à peu près, comment
- Rédiger un brouillon de la skill
- Créer quelques requêtes de test et les soumettre à un Claude ayant accès à la skill
- Aider l'utilisateur à évaluer les résultats, qualitativement et quantitativement
  - Pendant que les exécutions tournent en arrière-plan, rédiger quelques évaluations quantitatives s'il n'y en a pas (s'il y en a, on peut les reprendre telles quelles ou les modifier si quelque chose doit changer). Puis les expliquer à l'utilisateur (ou, si elles existaient déjà, expliquer celles qui existent)
  - Utiliser le script `eval-viewer/generate_review.py` pour montrer les résultats à l'utilisateur, et lui faire consulter aussi les indicateurs quantitatifs
- Réécrire la skill d'après le retour de l'utilisateur sur les résultats (et d'après les défauts flagrants que révèlent les mesures quantitatives)
- Recommencer jusqu'à satisfaction
- Élargir le jeu de tests et réessayer à plus grande échelle

Quand on utilise cette skill, il s'agit de déterminer où en est l'utilisateur dans ce processus, puis d'intervenir pour l'aider à franchir ces étapes. Par exemple, il dit peut-être « je veux faire une skill pour X ». On peut alors l'aider à préciser ce qu'il veut dire, rédiger un brouillon, écrire les cas de test, déterminer comment il veut évaluer, lancer toutes les requêtes, et recommencer.

À l'inverse, il a peut-être déjà un brouillon de skill. On peut alors passer directement à la partie évaluation / itération de la boucle.

Bien sûr, rester souple : si l'utilisateur dit « je n'ai pas besoin de lancer tout un tas d'évaluations, on avance au feeling », on peut faire ainsi.

Puis, une fois la skill terminée (mais là encore l'ordre est souple), on peut aussi lancer l'optimiseur de description, qui dispose de son propre script, pour améliorer le déclenchement de la skill.

C'est bon ? C'est bon.

## Communiquer avec l'utilisateur

Le créateur de skills peut être utilisé par des gens très inégalement familiers du jargon informatique. Au cas où on ne le saurait pas (et comment le saurait-on, c'est très récent), une tendance se dessine : la puissance de Claude pousse des plombiers à ouvrir leur terminal, des parents et des grands-parents à chercher « comment installer npm ». Cela dit, la majorité des utilisateurs sont sans doute assez à l'aise avec l'informatique.

Être donc attentif aux indices du contexte pour savoir comment formuler ! Par défaut, pour donner une idée :

- « évaluation » et « benchmark » sont limites, mais ça passe
- pour « JSON » et « assertion », il faut des indices sérieux que l'utilisateur sait de quoi il s'agit avant de les employer sans les expliquer

On peut expliquer brièvement les termes en cas de doute, et ne pas hésiter à donner une courte définition si l'on n'est pas sûr que l'utilisateur comprendra.

---

## Créer une skill

### Saisir l'intention

Commencer par comprendre l'intention de l'utilisateur. La conversation en cours contient peut-être déjà une démarche qu'il veut capturer (par exemple il dit « transforme ça en skill »). Dans ce cas, tirer d'abord les réponses de l'historique de la conversation — les outils utilisés, l'enchaînement des étapes, les corrections apportées par l'utilisateur, les formats d'entrée et de sortie observés. L'utilisateur devra peut-être combler les manques, et doit confirmer avant de passer à l'étape suivante.

1. Que doit permettre à Claude de faire cette skill ?
2. Quand doit-elle se déclencher ? (quelles formulations, quels contextes de l'utilisateur)
3. Quel est le format de sortie attendu ?
4. Faut-il mettre en place des cas de test pour vérifier que la skill fonctionne ? Les skills dont les sorties sont objectivement vérifiables (transformation de fichiers, extraction de données, génération de code, étapes de démarche fixes) gagnent à avoir des cas de test. Celles dont les sorties sont subjectives (style d'écriture, art) n'en ont souvent pas besoin. Suggérer le choix par défaut adapté au type de skill, mais laisser l'utilisateur décider.

### Entretien et recherche

Poser de soi-même des questions sur les cas limites, les formats d'entrée et de sortie, les fichiers d'exemple, les critères de réussite et les dépendances. Attendre d'avoir clarifié ces points avant d'écrire les requêtes de test.

Vérifier les MCP disponibles — s'ils sont utiles pour la recherche (consulter la documentation, trouver des skills semblables, chercher les bonnes pratiques), faire la recherche en parallèle par des sous-agents si possible, sinon directement. Arriver avec du contexte pour alléger la charge de l'utilisateur.

### Rédiger le SKILL.md

D'après l'entretien avec l'utilisateur, remplir ces éléments :

- **name** : identifiant de la skill
- **description** : quand la déclencher, ce qu'elle fait. C'est le principal mécanisme de déclenchement — y inclure à la fois ce que fait la skill ET les contextes précis où l'utiliser. Toutes les informations « quand l'utiliser » vont ici, pas dans le corps. Remarque : Claude a actuellement tendance à « sous-déclencher » les skills — à ne pas les utiliser alors qu'elles seraient utiles. Pour compenser, rendre les descriptions un peu « insistantes ». Par exemple, au lieu de « Comment construire un tableau de bord simple et rapide pour afficher des données internes d'Anthropic. », on peut écrire « Comment construire un tableau de bord simple et rapide pour afficher des données internes d'Anthropic. Utiliser impérativement cette skill dès que l'utilisateur parle de tableaux de bord, de visualisation de données, d'indicateurs internes, ou veut afficher n'importe quel type de données d'entreprise, même s'il ne demande pas explicitement un "tableau de bord". »
- **compatibility** : outils et dépendances requis (facultatif, rarement nécessaire)
- **le reste de la skill :)**

### Guide de rédaction des skills

#### Anatomie d'une skill

```
nom-de-la-skill/
├── SKILL.md (obligatoire)
│   ├── En-tête YAML (name et description obligatoires)
│   └── Instructions en Markdown
└── Ressources fournies (facultatif)
    ├── scripts/    - Code exécutable pour les tâches déterministes ou répétitives
    ├── references/ - Documentation chargée dans le contexte au besoin
    └── assets/     - Fichiers utilisés dans la sortie (gabarits, icônes, polices)
```

#### Divulgation progressive

Les skills utilisent un système de chargement à trois niveaux :
1. **Métadonnées** (name + description) — toujours dans le contexte (~100 mots)
2. **Corps du SKILL.md** — dans le contexte dès que la skill se déclenche (moins de 500 lignes idéalement)
3. **Ressources fournies** — au besoin (illimitées ; les scripts peuvent s'exécuter sans être chargés)

Ces volumes sont approximatifs ; on peut aller plus loin si nécessaire.

**Motifs clés :**
- Garder le SKILL.md sous 500 lignes ; à l'approche de cette limite, ajouter un niveau de hiérarchie, avec des renvois clairs indiquant au modèle qui utilise la skill où aller ensuite.
- Référencer clairement les fichiers depuis le SKILL.md, en indiquant quand les lire
- Pour les gros fichiers de référence (plus de 300 lignes), inclure une table des matières

**Organisation par domaine** : quand une skill couvre plusieurs domaines ou frameworks, organiser par variante :
```
cloud-deploy/
├── SKILL.md (démarche + choix)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```
Claude ne lit que le fichier de référence pertinent.

#### Principe d'absence de surprise

Cela va sans dire, mais une skill ne doit contenir ni logiciel malveillant, ni code d'exploitation, ni aucun contenu susceptible de compromettre la sécurité d'un système. Le contenu d'une skill ne doit pas surprendre l'utilisateur dans son intention s'il était décrit. Ne pas accepter de créer des skills trompeuses, ni des skills conçues pour faciliter un accès non autorisé, une exfiltration de données ou d'autres activités malveillantes. Des choses comme « joue le rôle d'un XYZ » sont en revanche acceptables.

#### Motifs d'écriture

Privilégier l'impératif (ou l'infinitif) dans les instructions.

**Définir les formats de sortie** — on peut procéder ainsi :
```markdown
## Structure du rapport
TOUJOURS utiliser exactement ce gabarit :
# [Titre]
## Synthèse
## Principaux constats
## Recommandations
```

**Motif d'exemples** — il est utile d'inclure des exemples. On peut les présenter ainsi (mais si « Entrée » et « Sortie » figurent dans les exemples, on peut s'écarter un peu de ce modèle) :
```markdown
## Format des messages de commit
**Exemple 1 :**
Entrée : Ajout de l'authentification des utilisateurs par jetons JWT
Sortie : feat(auth): authentification par JWT
```

### Style d'écriture

Chercher à expliquer au modèle pourquoi les choses sont importantes plutôt que d'aligner de lourds « IL FAUT » poussiéreux. Faire appel à la théorie de l'esprit, et chercher à rendre la skill générale plutôt que collée à des exemples précis. Commencer par un brouillon, puis le relire avec un regard neuf et l'améliorer.

### Cas de test

Après avoir rédigé le brouillon de la skill, imaginer 2 ou 3 requêtes de test réalistes — le genre de chose qu'un vrai utilisateur dirait. Les partager avec l'utilisateur : [pas besoin de reprendre ces mots exacts] « Voici quelques cas de test que j'aimerais essayer. Vous conviennent-ils, ou voulez-vous en ajouter ? » Puis les lancer.

Enregistrer les cas de test dans `evals/evals.json`. Ne pas encore écrire d'assertions — seulement les requêtes. On rédigera les assertions à l'étape suivante, pendant que les exécutions tournent.

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "Requête de l'utilisateur",
      "expected_output": "Description du résultat attendu",
      "files": []
    }
  ]
}
```

Voir `references/schemas.md` pour le schéma complet (dont le champ `assertions`, qu'on ajoutera plus tard).

## Lancer et évaluer les cas de test

Cette section est une séquence continue — ne pas s'arrêter en route. NE PAS utiliser `/skill-test` ni aucune autre skill de test.

Placer les résultats dans `<nom-de-la-skill>-workspace/`, à côté du répertoire de la skill. Dans cet espace de travail, organiser les résultats par itération (`iteration-1/`, `iteration-2/`, etc.), et dans chacune, un répertoire par cas de test (`eval-0/`, `eval-1/`, etc.). Ne pas tout créer d'avance — créer les répertoires au fur et à mesure.

### Étape 1 : lancer toutes les exécutions (avec la skill ET de référence) dans le même tour

Pour chaque cas de test, lancer deux sous-agents dans le même tour — l'un avec la skill, l'autre sans. C'est important : ne pas lancer d'abord les exécutions avec la skill pour revenir plus tard aux exécutions de référence. Tout lancer d'un coup, pour que tout se termine à peu près en même temps.

**Exécution avec la skill :**

```
Exécute cette tâche :
- Chemin de la skill : <chemin-de-la-skill>
- Tâche : <requête de l'évaluation>
- Fichiers d'entrée : <fichiers de l'évaluation s'il y en a, sinon « aucun »>
- Enregistre les sorties dans : <workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- Sorties à enregistrer : <ce qui intéresse l'utilisateur — par ex. « le fichier .docx », « le CSV final »>
```

**Exécution de référence** (même requête, mais la référence dépend du contexte) :
- **Création d'une nouvelle skill** : pas de skill du tout. Même requête, sans chemin de skill, enregistrement dans `without_skill/outputs/`.
- **Amélioration d'une skill existante** : l'ancienne version. Avant de la modifier, en faire un instantané (`cp -r <chemin-de-la-skill> <workspace>/skill-snapshot/`), puis faire pointer le sous-agent de référence vers cet instantané. Enregistrement dans `old_skill/outputs/`.

Écrire un `eval_metadata.json` pour chaque cas de test (les assertions peuvent rester vides pour l'instant). Donner à chaque évaluation un nom parlant, fondé sur ce qu'elle teste — pas simplement « eval-0 ». Utiliser aussi ce nom pour le répertoire. Si cette itération utilise des requêtes d'évaluation nouvelles ou modifiées, créer ces fichiers pour chaque nouveau répertoire d'évaluation — ne pas supposer qu'ils sont repris des itérations précédentes.

```json
{
  "eval_id": 0,
  "eval_name": "nom-parlant-ici",
  "prompt": "La requête de l'utilisateur",
  "assertions": []
}
```

### Étape 2 : pendant les exécutions, rédiger les assertions

Ne pas se contenter d'attendre la fin des exécutions — ce temps peut être utilisé utilement. Rédiger des assertions quantitatives pour chaque cas de test et les expliquer à l'utilisateur. Si des assertions existent déjà dans `evals/evals.json`, les passer en revue et expliquer ce qu'elles vérifient.

De bonnes assertions sont objectivement vérifiables et portent des noms parlants — elles doivent se lire clairement dans la visionneuse de benchmark, pour qu'un coup d'œil aux résultats suffise à comprendre ce que chacune vérifie. Les skills subjectives (style d'écriture, qualité du design) s'évaluent mieux qualitativement — ne pas forcer des assertions sur ce qui relève du jugement humain.

Mettre à jour les fichiers `eval_metadata.json` et `evals/evals.json` avec les assertions une fois rédigées. Expliquer aussi à l'utilisateur ce qu'il verra dans la visionneuse — à la fois les sorties qualitatives et le benchmark quantitatif.

### Étape 3 : à la fin de chaque exécution, relever les durées

À la fin de chaque tâche de sous-agent, on reçoit une notification contenant `total_tokens` et `duration_ms`. Enregistrer immédiatement ces données dans `timing.json`, dans le répertoire de l'exécution :

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

C'est la seule occasion de relever ces données — elles arrivent par la notification de tâche et ne sont conservées nulle part ailleurs. Traiter chaque notification à son arrivée plutôt que de chercher à les regrouper.

### Étape 4 : noter, agréger et lancer la visionneuse

Une fois toutes les exécutions terminées :

1. **Noter chaque exécution** — lancer un sous-agent correcteur (ou noter directement) qui lit `agents/grader.md` et évalue chaque assertion au regard des sorties. Enregistrer les résultats dans `grading.json`, dans chaque répertoire d'exécution. Le tableau `expectations` de grading.json doit utiliser les champs `text`, `passed` et `evidence` (pas `name` / `met` / `details` ni d'autres variantes) — la visionneuse dépend de ces noms de champs exacts. Pour les assertions vérifiables par programme, écrire et lancer un script plutôt que de vérifier à l'œil — les scripts sont plus rapides, plus fiables et réutilisables d'une itération à l'autre.

2. **Agréger en benchmark** — lancer le script d'agrégation depuis le répertoire de skill-creator :
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <nom>
   ```
   On obtient `benchmark.json` et `benchmark.md`, avec taux de réussite, durée et jetons pour chaque configuration, avec moyenne ± écart-type et l'écart entre configurations. Pour générer benchmark.json à la main, voir `references/schemas.md` pour le schéma exact qu'attend la visionneuse.
Placer chaque version with_skill avant son homologue de référence.

3. **Faire une passe d'analyse** — lire les données du benchmark et faire ressortir les tendances que les statistiques agrégées pourraient masquer. Voir `agents/analyzer.md` (la section « Analyser les résultats du benchmark ») pour savoir quoi chercher — par exemple des assertions qui passent toujours, avec ou sans skill (non discriminantes), des évaluations à forte variance (peut-être instables), et les compromis durée / jetons.

4. **Lancer la visionneuse** avec à la fois les sorties qualitatives et les données quantitatives :
   ```bash
   nohup python <chemin-de-skill-creator>/eval-viewer/generate_review.py \
     <workspace>/iteration-N \
     --skill-name "my-skill" \
     --benchmark <workspace>/iteration-N/benchmark.json \
     > /dev/null 2>&1 &
   VIEWER_PID=$!
   ```
   À partir de l'itération 2, passer aussi `--previous-workspace <workspace>/iteration-<N-1>`.

   **Cowork / environnements sans affichage :** si `webbrowser.open()` n'est pas disponible ou que l'environnement n'a pas d'écran, utiliser `--static <chemin_de_sortie>` pour écrire un fichier HTML autonome au lieu de démarrer un serveur. Les retours seront téléchargés sous forme de fichier `feedback.json` quand l'utilisateur cliquera sur « Submit All Reviews ». Après le téléchargement, copier `feedback.json` dans le répertoire de l'espace de travail pour que l'itération suivante le reprenne.

Remarque : utiliser generate_review.py pour créer la visionneuse ; inutile d'écrire du HTML sur mesure.

5. **Le dire à l'utilisateur**, par exemple : « J'ai ouvert les résultats dans votre navigateur. Il y a deux onglets — "Outputs" permet de parcourir chaque cas de test et de laisser un commentaire, "Benchmark" montre la comparaison quantitative. Quand vous avez terminé, revenez ici et dites-le-moi. »

### Ce que l'utilisateur voit dans la visionneuse

(L'interface de la visionneuse est en anglais ; les libellés sont donnés tels qu'ils apparaissent.)

L'onglet « Outputs » montre un cas de test à la fois :
- **Prompt** : la tâche donnée
- **Output** : les fichiers produits par la skill, affichés en ligne quand c'est possible
- **Previous Output** (à partir de l'itération 2) : section repliée montrant la sortie de l'itération précédente
- **Formal Grades** (si la notation a été faite) : section repliée montrant les assertions réussies ou échouées
- **Feedback** : une zone de texte qui s'enregistre automatiquement pendant la saisie
- **Previous Feedback** (à partir de l'itération 2) : ses commentaires de la fois précédente, affichés sous la zone de texte

L'onglet « Benchmark » montre la synthèse statistique : taux de réussite, durées et consommation de jetons pour chaque configuration, avec le détail par évaluation et les observations de l'analyste.

La navigation se fait par les boutons précédent / suivant ou les flèches du clavier. À la fin, l'utilisateur clique sur « Submit All Reviews », ce qui enregistre tous les retours dans `feedback.json`.

### Étape 5 : lire les retours

Quand l'utilisateur indique avoir terminé, lire `feedback.json` :

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "il manque les étiquettes des axes sur le graphique", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_skill", "feedback": "parfait, j'adore", "timestamp": "..."}
  ],
  "status": "complete"
}
```

Un retour vide signifie que l'utilisateur a trouvé le résultat correct. Concentrer les améliorations sur les cas de test où il a formulé des reproches précis.

Arrêter le serveur de la visionneuse quand on n'en a plus besoin :

```bash
kill $VIEWER_PID 2>/dev/null
```

---

## Améliorer la skill

C'est le cœur de la boucle. On a lancé les cas de test, l'utilisateur a passé les résultats en revue, et il faut maintenant améliorer la skill d'après ses retours.

### Comment penser les améliorations

1. **Généraliser à partir des retours.** L'enjeu de fond, c'est de créer des skills qui pourront servir un million de fois (peut-être littéralement, peut-être plus, qui sait) sur une multitude de requêtes différentes. Ici, on itère avec l'utilisateur sur quelques exemples seulement, encore et encore, parce que cela permet d'avancer plus vite. L'utilisateur connaît ces exemples sur le bout des doigts et peut évaluer rapidement de nouvelles sorties. Mais si la skill qu'on développe ensemble ne fonctionne que pour ces exemples, elle ne sert à rien. Plutôt que d'introduire des retouches pointilleuses qui collent trop aux exemples, ou des « IL FAUT » oppressants, face à un problème tenace, on peut essayer d'élargir, d'employer d'autres métaphores, ou de recommander d'autres façons de travailler. C'est relativement peu coûteux à essayer, et on tombera peut-être sur quelque chose d'excellent.

2. **Garder la consigne légère.** Supprimer ce qui ne pèse pas son poids. Lire les transcriptions, pas seulement les sorties finales — si la skill semble faire perdre beaucoup de temps au modèle en activités improductives, on peut essayer de retirer les passages qui l'y poussent et voir ce qui se passe.

3. **Expliquer le pourquoi.** S'efforcer d'expliquer le **pourquoi** de tout ce qu'on demande au modèle. Les LLM d'aujourd'hui sont *intelligents*. Ils ont une bonne théorie de l'esprit et, dans un bon cadre, peuvent dépasser les consignes appliquées mécaniquement et vraiment faire avancer les choses. Même si le retour de l'utilisateur est laconique ou agacé, chercher à comprendre vraiment la tâche, pourquoi l'utilisateur a écrit ce qu'il a écrit, et ce qu'il a réellement écrit, puis transmettre cette compréhension dans les instructions. Si l'on se surprend à écrire TOUJOURS ou JAMAIS en capitales, ou à employer des structures très rigides, c'est un signal d'alerte — si possible, reformuler et expliquer le raisonnement pour que le modèle comprenne pourquoi ce qu'on demande est important. C'est une approche plus humaine, plus puissante et plus efficace.

4. **Repérer le travail répété d'un cas de test à l'autre.** Lire les transcriptions des exécutions de test et remarquer si les sous-agents ont tous écrit, chacun de leur côté, des scripts d'aide semblables ou suivi la même démarche en plusieurs étapes. Si les 3 cas de test ont amené le sous-agent à écrire un `create_docx.py` ou un `build_chart.py`, c'est un signal fort que la skill devrait fournir ce script. L'écrire une fois, le placer dans `scripts/`, et indiquer à la skill de l'utiliser. Cela évite à chaque invocation future de réinventer la roue.

Cette tâche est assez importante (on cherche ici à créer des milliards de valeur économique par an !) et le temps de réflexion n'est pas le facteur limitant ; prendre son temps et vraiment mûrir les choses. Je suggère de rédiger une révision, puis de la relire d'un œil neuf et de l'améliorer. Faire vraiment de son mieux pour se mettre à la place de l'utilisateur et comprendre ce qu'il veut et ce dont il a besoin.

### La boucle d'itération

Après avoir amélioré la skill :

1. Appliquer les améliorations à la skill
2. Relancer tous les cas de test dans un nouveau répertoire `iteration-<N+1>/`, exécutions de référence comprises. Pour une nouvelle skill, la référence est toujours `without_skill` (pas de skill) — elle reste la même d'une itération à l'autre. Pour l'amélioration d'une skill existante, juger de la référence la plus pertinente : la version d'origine apportée par l'utilisateur, ou l'itération précédente.
3. Lancer la visionneuse avec `--previous-workspace` pointant vers l'itération précédente
4. Attendre que l'utilisateur ait passé les résultats en revue et dise avoir terminé
5. Lire les nouveaux retours, améliorer encore, recommencer

Continuer jusqu'à ce que :
- l'utilisateur se dise satisfait
- tous les retours soient vides (tout semble bon)
- on ne progresse plus de façon significative

---

## Avancé : comparaison à l'aveugle

Pour les situations où l'on veut une comparaison plus rigoureuse entre deux versions d'une skill (par exemple l'utilisateur demande « la nouvelle version est-elle vraiment meilleure ? »), il existe un système de comparaison à l'aveugle. Lire `agents/comparator.md` et `agents/analyzer.md` pour les détails. L'idée de base : donner deux sorties à un agent indépendant sans lui dire laquelle est laquelle, et le laisser juger de la qualité. Puis analyser pourquoi la gagnante a gagné.

C'est facultatif, cela demande des sous-agents, et la plupart des utilisateurs n'en auront pas besoin. La boucle de relecture humaine suffit généralement.

---

## Optimisation de la description

Le champ `description` de l'en-tête du SKILL.md est le principal mécanisme qui détermine si Claude invoque une skill. Après avoir créé ou amélioré une skill, proposer d'optimiser la description pour un déclenchement plus juste.

### Étape 1 : générer les requêtes d'évaluation du déclenchement

Créer 20 requêtes d'évaluation — un mélange de requêtes qui doivent déclencher la skill et de requêtes qui ne le doivent pas. Les enregistrer en JSON :

```json
[
  {"query": "la requête de l'utilisateur", "should_trigger": true},
  {"query": "une autre requête", "should_trigger": false}
]
```

Les requêtes doivent être réalistes, le genre de chose qu'un utilisateur de Claude Code ou de Claude.ai taperait vraiment. Pas des demandes abstraites, mais des demandes concrètes, précises et suffisamment détaillées. Par exemple des chemins de fichiers, du contexte personnel sur le métier ou la situation de l'utilisateur, des noms et des valeurs de colonnes, des noms d'entreprises, des URL. Un peu d'histoire. Certaines peuvent être en minuscules, contenir des abréviations, des fautes de frappe ou un langage familier. Varier les longueurs, et se concentrer sur les cas limites plutôt que sur des cas tranchés (l'utilisateur aura l'occasion de les valider).

Mauvais : `"Mets ces données en forme"`, `"Extrais le texte du PDF"`, `"Crée un graphique"`

Bon : `"bon alors mon chef vient de m'envoyer ce xlsx (il est dans mes téléchargements, un truc comme 'ventes T4 final FINAL v2.xlsx') et elle veut que j'ajoute une colonne avec la marge en pourcentage. Le CA est en colonne C et les coûts en colonne D je crois"`

Pour les requêtes **qui doivent déclencher** (8 à 10), penser à la couverture. Il faut différentes formulations de la même intention — certaines soutenues, d'autres familières. Inclure des cas où l'utilisateur ne nomme pas explicitement la skill ou le type de fichier mais en a clairement besoin. Ajouter quelques cas d'usage peu courants et des cas où cette skill est en concurrence avec une autre mais doit l'emporter.

Pour les requêtes **qui ne doivent pas déclencher** (8 à 10), les plus précieuses sont les quasi-correspondances — des requêtes qui partagent des mots-clés ou des notions avec la skill mais demandent en réalité autre chose. Penser aux domaines voisins, aux formulations ambiguës où une correspondance naïve de mots-clés déclencherait à tort, et aux cas où la requête touche à quelque chose que fait la skill, mais dans un contexte où un autre outil est plus approprié.

L'essentiel à éviter : des requêtes négatives manifestement hors sujet. « Écris une fonction de Fibonacci » comme test négatif pour une skill PDF est trop facile — cela ne teste rien. Les cas négatifs doivent être réellement piégeux.

### Étape 2 : relecture avec l'utilisateur

Présenter le jeu d'évaluation à l'utilisateur pour relecture, avec le gabarit HTML :

1. Lire le gabarit `assets/eval_review.html`
2. Remplacer les espaces réservés :
   - `__EVAL_DATA_PLACEHOLDER__` → le tableau JSON des requêtes (sans guillemets autour — c'est une affectation de variable JS)
   - `__SKILL_NAME_PLACEHOLDER__` → le nom de la skill
   - `__SKILL_DESCRIPTION_PLACEHOLDER__` → la description actuelle de la skill
3. Écrire dans un fichier temporaire (par exemple `/tmp/eval_review_<nom-de-la-skill>.html`) et l'ouvrir : `open /tmp/eval_review_<nom-de-la-skill>.html`
4. L'utilisateur peut modifier les requêtes, basculer « doit déclencher », ajouter ou supprimer des entrées, puis cliquer sur « Export Eval Set »
5. Le fichier est téléchargé dans `~/Downloads/eval_set.json` — vérifier le dossier Téléchargements pour prendre la version la plus récente s'il y en a plusieurs (par exemple `eval_set (1).json`)

Cette étape compte — de mauvaises requêtes d'évaluation donnent de mauvaises descriptions.

### Étape 3 : lancer la boucle d'optimisation

Dire à l'utilisateur : « Cela va prendre un peu de temps — je lance la boucle d'optimisation en arrière-plan et je la surveillerai régulièrement. »

Enregistrer le jeu d'évaluation dans l'espace de travail, puis lancer en arrière-plan :

```bash
python -m scripts.run_loop \
  --eval-set <chemin-du-jeu-d-evaluation.json> \
  --skill-path <chemin-de-la-skill> \
  --model <identifiant-du-modele-de-cette-session> \
  --max-iterations 5 \
  --verbose
```

Utiliser l'identifiant de modèle indiqué dans le prompt système (celui qui fait tourner la session en cours), pour que le test de déclenchement corresponde à ce que vit réellement l'utilisateur.

Pendant l'exécution, consulter régulièrement la fin de la sortie pour tenir l'utilisateur informé de l'itération en cours et des scores.

Le script gère automatiquement toute la boucle d'optimisation. Il partage le jeu d'évaluation en 60 % d'entraînement et 40 % de test réservé, évalue la description actuelle (chaque requête est lancée 3 fois pour obtenir un taux de déclenchement fiable), puis appelle Claude pour proposer des améliorations d'après les échecs. Il réévalue chaque nouvelle description sur l'entraînement et sur le test, jusqu'à 5 itérations. À la fin, il ouvre dans le navigateur un rapport HTML des résultats par itération et renvoie un JSON avec `best_description` — choisie selon le score de test plutôt que celui d'entraînement, pour éviter le surapprentissage.

### Comment fonctionne le déclenchement des skills

Comprendre le mécanisme de déclenchement aide à concevoir de meilleures requêtes d'évaluation. Les skills figurent dans la liste `available_skills` de Claude avec leur nom et leur description, et Claude décide de consulter une skill sur la base de cette description. Ce qu'il faut savoir : Claude ne consulte les skills que pour les tâches qu'il ne peut pas facilement traiter seul — des requêtes simples, en une étape, comme « lis ce PDF », peuvent ne pas déclencher de skill même si la description correspond parfaitement, parce que Claude peut les traiter directement avec des outils de base. Les requêtes complexes, en plusieurs étapes ou spécialisées déclenchent les skills de façon fiable quand la description correspond.

Les requêtes d'évaluation doivent donc être assez substantielles pour que Claude ait réellement intérêt à consulter une skill. Des requêtes simples comme « lis le fichier X » sont de mauvais cas de test — elles ne déclencheront pas de skill, quelle que soit la qualité de la description.

### Étape 4 : appliquer le résultat

Prendre `best_description` dans la sortie JSON et mettre à jour l'en-tête du SKILL.md de la skill. Montrer à l'utilisateur l'avant / après et indiquer les scores.

---

### Empaqueter et présenter (seulement si l'outil `present_files` est disponible)

Vérifier si l'on a accès à l'outil `present_files`. Sinon, sauter cette étape. Si oui, empaqueter la skill et présenter le fichier .skill à l'utilisateur :

```bash
python -m scripts.package_skill <chemin/du/dossier-de-la-skill>
```

Après l'empaquetage, indiquer à l'utilisateur le chemin du fichier `.skill` obtenu pour qu'il puisse l'installer.

---

## Instructions propres à Claude.ai

Dans Claude.ai, la démarche de base est la même (brouillon → test → relecture → amélioration → on recommence), mais comme Claude.ai n'a pas de sous-agents, certains mécanismes changent. Voici ce qu'il faut adapter :

**Lancer les cas de test** : sans sous-agents, pas d'exécution parallèle. Pour chaque cas de test, lire le SKILL.md de la skill, puis suivre ses instructions pour accomplir soi-même la requête de test. Les traiter un par un. C'est moins rigoureux que des sous-agents indépendants (on a écrit la skill et on l'exécute aussi, donc on a tout le contexte), mais c'est un contrôle de cohérence utile — et l'étape de relecture humaine compense. Sauter les exécutions de référence — utiliser simplement la skill pour accomplir la tâche demandée.

**Relire les résultats** : si l'on ne peut pas ouvrir de navigateur (par exemple la VM de Claude.ai n'a pas d'écran, ou l'on est sur un serveur distant), sauter entièrement la visionneuse. Présenter plutôt les résultats directement dans la conversation. Pour chaque cas de test, montrer la requête et la sortie. Si la sortie est un fichier que l'utilisateur doit voir (comme un .docx ou un .xlsx), l'enregistrer sur le système de fichiers et lui dire où il se trouve pour qu'il puisse le télécharger et l'examiner. Demander un retour directement : « Qu'en pensez-vous ? Quelque chose à changer ? »

**Benchmark** : sauter le benchmark quantitatif — il repose sur des comparaisons avec une référence, qui n'ont pas de sens sans sous-agents. Se concentrer sur le retour qualitatif de l'utilisateur.

**La boucle d'itération** : comme avant — améliorer la skill, relancer les cas de test, demander un retour — simplement sans la visionneuse au milieu. On peut quand même organiser les résultats en répertoires d'itération si l'on dispose d'un système de fichiers.

**Optimisation de la description** : cette section demande l'outil en ligne de commande `claude` (plus précisément `claude -p`), disponible seulement dans Claude Code. La sauter si l'on est sur Claude.ai.

**Comparaison à l'aveugle** : demande des sous-agents. La sauter.

**Empaquetage** : le script `package_skill.py` fonctionne partout où il y a Python et un système de fichiers. Sur Claude.ai, on peut le lancer, et l'utilisateur peut télécharger le fichier `.skill` obtenu.

**Mettre à jour une skill existante** : l'utilisateur demande peut-être de mettre à jour une skill existante, et non d'en créer une nouvelle. Dans ce cas :
- **Conserver le nom d'origine.** Noter le nom du répertoire de la skill et le champ `name` de l'en-tête — les utiliser sans les changer. Par exemple, si la skill installée est `research-helper`, produire `research-helper.skill` (et non `research-helper-v2`).
- **Copier dans un emplacement accessible en écriture avant de modifier.** Le chemin de la skill installée peut être en lecture seule. Copier dans `/tmp/nom-de-la-skill/`, modifier là, et empaqueter à partir de la copie.
- **En cas d'empaquetage manuel, préparer d'abord dans `/tmp/`**, puis copier vers le répertoire de sortie — une écriture directe peut échouer pour des raisons de droits.

---

## Instructions propres à Cowork

Dans Cowork, l'essentiel à savoir :

- On dispose de sous-agents, donc la démarche principale (lancer les cas de test en parallèle, les exécutions de référence, la notation, etc.) fonctionne. (Toutefois, en cas de graves problèmes de délais d'expiration, on peut lancer les requêtes de test en série plutôt qu'en parallèle.)
- On n'a ni navigateur ni écran : pour générer la visionneuse d'évaluation, utiliser `--static <chemin_de_sortie>` afin d'écrire un fichier HTML autonome au lieu de démarrer un serveur. Puis proposer un lien sur lequel l'utilisateur peut cliquer pour ouvrir le HTML dans son navigateur.
- Pour une raison ou une autre, l'environnement Cowork semble dissuader Claude de générer la visionneuse après les tests ; donc, pour insister : que l'on soit dans Cowork ou dans Claude Code, après avoir lancé les tests, toujours générer la visionneuse d'évaluation pour que l'humain regarde les exemples avant de réviser soi-même la skill et de tenter des corrections, avec `generate_review.py` (sans écrire son propre HTML artisanal). Désolé d'avance, mais je passe en capitales : GÉNÉRER LA VISIONNEUSE *AVANT* d'évaluer soi-même. Il faut mettre les résultats sous les yeux de l'humain AU PLUS VITE !
- Les retours fonctionnent autrement : comme il n'y a pas de serveur en marche, le bouton « Submit All Reviews » de la visionneuse télécharge `feedback.json` sous forme de fichier. On peut alors le lire à partir de là (il faudra peut-être d'abord en demander l'accès).
- L'empaquetage fonctionne — `package_skill.py` n'a besoin que de Python et d'un système de fichiers.
- L'optimisation de la description (`run_loop.py` / `run_eval.py`) devrait bien fonctionner dans Cowork puisqu'elle utilise `claude -p` en sous-processus, pas un navigateur ; mais la garder pour la fin, quand la skill est complètement terminée et que l'utilisateur la juge en bon état.
- **Mettre à jour une skill existante** : l'utilisateur demande peut-être de mettre à jour une skill existante, et non d'en créer une nouvelle. Suivre les consignes de mise à jour de la section Claude.ai ci-dessus.

---

## Fichiers de référence

Le répertoire agents/ contient les instructions des sous-agents spécialisés. Les lire quand on doit lancer le sous-agent correspondant.

- `agents/grader.md` — comment évaluer des assertions au regard des sorties
- `agents/comparator.md` — comment faire une comparaison A/B à l'aveugle entre deux sorties
- `agents/analyzer.md` — comment analyser pourquoi une version l'a emporté sur une autre

Le répertoire references/ contient de la documentation complémentaire :
- `references/schemas.md` — structures JSON de evals.json, grading.json, etc.

---

Je répète une dernière fois la boucle centrale, pour insister :

- Déterminer de quoi traite la skill
- Rédiger ou modifier la skill
- Soumettre les requêtes de test à un Claude ayant accès à la skill
- Avec l'utilisateur, évaluer les sorties :
  - Créer benchmark.json et lancer `eval-viewer/generate_review.py` pour aider l'utilisateur à les passer en revue
  - Lancer les évaluations quantitatives
- Recommencer jusqu'à ce que l'utilisateur et soi-même soient satisfaits
- Empaqueter la skill finale et la remettre à l'utilisateur.

Ajouter ces étapes à sa liste de tâches, si l'on en a une, pour ne rien oublier. Dans Cowork, mettre précisément « Créer le JSON des évaluations et lancer `eval-viewer/generate_review.py` pour que l'humain passe en revue les cas de test » dans sa liste de tâches, pour être sûr que cela se fasse.

Bonne chance !
