# Guide d'évaluation des serveurs MCP

## Présentation

Ce document explique comment créer des évaluations complètes pour des serveurs MCP. Les évaluations testent si des LLM savent utiliser efficacement le serveur MCP pour répondre à des questions réalistes et complexes, avec les seuls outils fournis.

---

## Aide-mémoire

### Exigences de l'évaluation
- Créer 10 questions lisibles par un humain
- Les questions doivent être EN LECTURE SEULE, INDÉPENDANTES, NON DESTRUCTIVES
- Chaque question demande plusieurs appels d'outils (potentiellement des dizaines)
- Les réponses doivent être des valeurs uniques et vérifiables
- Les réponses doivent être STABLES (ne pas changer avec le temps)

### Format de sortie
```xml
<evaluation>
   <qa_pair>
      <question>La question ici</question>
      <answer>Réponse unique et vérifiable</answer>
   </qa_pair>
</evaluation>
```

---

## But des évaluations

La qualité d'un serveur MCP ne se mesure PAS à la qualité ou à l'exhaustivité avec laquelle il implémente ses outils, mais à la façon dont ces implémentations (schémas d'entrée et de sortie, docstrings et descriptions, fonctionnalités) permettent à des LLM sans autre contexte, et n'ayant accès QU'AUX serveurs MCP, de répondre à des questions réalistes et difficiles.

## Vue d'ensemble de l'évaluation

Créer 10 questions lisibles par un humain, auxquelles on répond UNIQUEMENT par des opérations EN LECTURE SEULE, INDÉPENDANTES, NON DESTRUCTIVES et IDEMPOTENTES. Chaque question doit être :
- réaliste
- claire et concise
- sans ambiguïté
- complexe, demandant potentiellement des dizaines d'appels d'outils ou d'étapes
- résoluble par une valeur unique et vérifiable, identifiée à l'avance

## Consignes pour les questions

### Exigences fondamentales

1. **Les questions DOIVENT être indépendantes**
   - Aucune question ne doit dépendre de la réponse à une autre
   - Elles ne doivent pas supposer d'écritures préalables faites en traitant une autre question

2. **Les questions DOIVENT n'exiger QUE des usages d'outils NON DESTRUCTIFS ET IDEMPOTENTS**
   - Elles ne doivent pas demander ni nécessiter de modifier un état pour arriver à la bonne réponse

3. **Les questions doivent être RÉALISTES, CLAIRES, CONCISES et COMPLEXES**
   - Elles doivent obliger un autre LLM à utiliser plusieurs outils ou étapes (potentiellement des dizaines) pour répondre

### Complexité et profondeur

4. **Les questions doivent exiger une exploration approfondie**
   - Envisager des questions en plusieurs sauts, demandant plusieurs sous-questions et des appels d'outils successifs
   - Chaque étape doit s'appuyer sur l'information trouvée aux étapes précédentes

5. **Les questions peuvent exiger une pagination étendue**
   - Il peut falloir parcourir plusieurs pages de résultats
   - Il peut falloir interroger des données anciennes (1 à 2 ans) pour trouver une information pointue
   - Les questions doivent être DIFFICILES

6. **Les questions doivent exiger une compréhension profonde**
   - Plutôt qu'une connaissance superficielle
   - On peut poser des idées complexes sous forme de questions Vrai / Faux exigeant des preuves
   - On peut utiliser un format à choix multiples où le LLM doit explorer différentes hypothèses

7. **Les questions ne doivent pas être résolubles par une simple recherche de mots-clés**
   - Ne pas inclure de mots-clés précis du contenu visé
   - Utiliser des synonymes, des notions voisines ou des paraphrases
   - Exiger plusieurs recherches, l'analyse de plusieurs éléments liés, l'extraction du contexte, puis la déduction de la réponse

### Test des outils

8. **Les questions doivent mettre à l'épreuve les valeurs de retour des outils**
   - Elles peuvent amener les outils à renvoyer de gros objets ou de longues listes JSON, qui submergent le LLM
   - Elles doivent exiger la compréhension de plusieurs modalités de données :
     - identifiants et noms
     - horodatages et dates-heures (mois, jours, années, secondes)
     - identifiants, noms, extensions et types MIME de fichiers
     - URL, GID, etc.
   - Elles doivent sonder la capacité de l'outil à renvoyer toutes les formes de données utiles

9. **Les questions doivent refléter SURTOUT des cas d'usage humains réels**
   - Le genre de recherches d'information qui intéresseraient des HUMAINS assistés d'un LLM

10. **Les questions peuvent exiger des dizaines d'appels d'outils**
    - Cela met à l'épreuve les LLM au contexte limité
    - Cela incite les outils du serveur MCP à réduire l'information renvoyée

11. **Inclure des questions ambiguës**
    - Elles peuvent être ambiguës OU exiger des décisions difficiles sur les outils à appeler
    - Pousser le LLM à se tromper ou à mal interpréter
    - Veiller à ce que, malgré l'AMBIGUÏTÉ, il y ait TOUJOURS UNE RÉPONSE UNIQUE ET VÉRIFIABLE

### Stabilité

12. **Les questions doivent être conçues pour que la réponse NE CHANGE PAS**
    - Ne pas poser de questions qui dépendent d'un « état actuel » dynamique
    - Par exemple, ne pas compter :
      - le nombre de réactions à un message
      - le nombre de réponses dans un fil
      - le nombre de membres d'un canal

13. **NE PAS laisser le serveur MCP RESTREINDRE le genre de questions créées**
    - Créer des questions difficiles et complexes
    - Certaines pourraient ne pas être résolubles avec les outils disponibles du serveur MCP
    - Les questions peuvent exiger des formats de sortie précis (date-heure ou temps Unix, JSON ou MARKDOWN)
    - Les questions peuvent exiger des dizaines d'appels d'outils

## Consignes pour les réponses

### Vérification

1. **Les réponses doivent être VÉRIFIABLES par comparaison directe de chaînes**
   - Si la réponse peut s'écrire de plusieurs façons, préciser clairement le format attendu dans la QUESTION
   - Exemples : « Utilisez AAAA/MM/JJ. », « Répondez Vrai ou Faux. », « Répondez A, B, C ou D et rien d'autre. »
   - La réponse doit être une valeur unique VÉRIFIABLE, par exemple :
     - identifiant d'utilisateur, nom d'utilisateur, nom affiché, prénom, nom
     - identifiant ou nom de canal
     - identifiant de message, chaîne
     - URL, titre
     - quantité numérique
     - horodatage, date-heure
     - booléen (pour les questions Vrai / Faux)
     - adresse électronique, numéro de téléphone
     - identifiant, nom ou extension de fichier
     - réponse à choix multiples
   - Les réponses ne doivent pas exiger de mise en forme particulière ni de sortie structurée complexe
   - La réponse sera vérifiée par COMPARAISON DIRECTE DE CHAÎNES

### Lisibilité

2. **Les réponses doivent en général privilégier des formats LISIBLES PAR UN HUMAIN**
   - Exemples : noms, prénom, nom de famille, date-heure, nom de fichier, texte de message, URL, oui/non, vrai/faux, a/b/c/d
   - Plutôt que des identifiants opaques (même si les identifiants sont acceptables)
   - La GRANDE MAJORITÉ des réponses doivent être lisibles par un humain

### Stabilité

3. **Les réponses doivent être STABLES**
   - Examiner du contenu ancien (par exemple des conversations terminées, des projets lancés, des questions résolues)
   - Créer des QUESTIONS sur des notions « closes » qui renverront toujours la même réponse
   - Les questions peuvent demander de considérer une fenêtre de temps fixe pour se prémunir contre les réponses non stationnaires
   - S'appuyer sur un contexte PEU SUSCEPTIBLE de changer
   - Exemple : pour trouver le titre d'un article, être assez PRÉCIS pour qu'il ne soit pas confondu avec des articles publiés plus tard

4. **Les réponses doivent être CLAIRES et SANS AMBIGUÏTÉ**
   - Les questions doivent être conçues pour n'avoir qu'une réponse claire
   - La réponse doit pouvoir être obtenue avec les outils du serveur MCP

### Diversité

5. **Les réponses doivent être VARIÉES**
   - La réponse doit être une valeur unique VÉRIFIABLE, de modalités et de formats variés
   - Notion d'utilisateur : identifiant, nom d'utilisateur, nom affiché, prénom, nom, adresse électronique, téléphone
   - Notion de canal : identifiant, nom, sujet du canal
   - Notion de message : identifiant, texte, horodatage, mois, jour, année

6. **Les réponses ne doivent PAS être des structures complexes**
   - pas une liste de valeurs
   - pas un objet complexe
   - pas une liste d'identifiants ou de chaînes
   - pas un texte en langage naturel
   - SAUF si la réponse peut être vérifiée simplement par COMPARAISON DIRECTE DE CHAÎNES
   - et peut être reproduite de façon réaliste
   - Il doit être peu probable qu'un LLM renvoie la même liste dans un autre ordre ou un autre format

## Processus d'évaluation

### Étape 1 : examen de la documentation

Lire la documentation de l'API visée pour comprendre :
- les points d'accès et fonctionnalités disponibles
- en cas d'ambiguïté, récupérer des informations complémentaires sur le web
- paralléliser cette étape AUTANT QUE POSSIBLE
- veiller à ce que chaque sous-agent n'examine QUE de la documentation, sur le système de fichiers ou sur le web

### Étape 2 : examen des outils

Lister les outils disponibles dans le serveur MCP :
- examiner directement le serveur MCP
- comprendre les schémas d'entrée et de sortie, les docstrings et les descriptions
- SANS appeler les outils eux-mêmes à ce stade

### Étape 3 : approfondir la compréhension

Répéter les étapes 1 et 2 jusqu'à bien comprendre :
- itérer plusieurs fois
- réfléchir au genre de tâches à créer
- affiner sa compréhension
- à AUCUN moment ne LIRE le code de l'implémentation du serveur MCP lui-même
- s'appuyer sur son intuition et sa compréhension pour créer des tâches raisonnables, réalistes, mais TRÈS difficiles

### Étape 4 : examen du contenu en lecture seule

Une fois l'API et les outils compris, UTILISER les outils du serveur MCP :
- examiner le contenu UNIQUEMENT par des opérations EN LECTURE SEULE et NON DESTRUCTIVES
- but : repérer du contenu précis (utilisateurs, canaux, messages, projets, tâches, etc.) pour créer des questions réalistes
- n'appeler AUCUN outil qui modifie un état
- ne PAS lire le code de l'implémentation du serveur MCP lui-même
- paralléliser cette étape avec des sous-agents menant des explorations indépendantes
- veiller à ce que chaque sous-agent ne fasse que des opérations EN LECTURE SEULE, NON DESTRUCTIVES et IDEMPOTENTES
- ATTENTION : CERTAINS OUTILS peuvent renvoyer BEAUCOUP DE DONNÉES, au point d'épuiser le CONTEXTE
- faire des appels d'outils PROGRESSIFS, PETITS ET CIBLÉS pour explorer
- dans tous les appels d'outils, utiliser le paramètre `limit` pour limiter les résultats (< 10)
- utiliser la pagination

### Étape 5 : génération des tâches

Après avoir examiné le contenu, créer 10 questions lisibles par un humain :
- un LLM doit pouvoir y répondre avec le serveur MCP
- suivre toutes les consignes ci-dessus sur les questions et les réponses

## Format de sortie

Chaque paire QR se compose d'une question et d'une réponse. La sortie doit être un fichier XML de cette structure :

```xml
<evaluation>
   <qa_pair>
      <question>Trouvez le projet créé au 2e trimestre 2024 qui compte le plus de tâches terminées. Quel est le nom du projet ?</question>
      <answer>Refonte du site web</answer>
   </qa_pair>
   <qa_pair>
      <question>Cherchez les tickets étiquetés « bug » fermés en mars 2024. Quel utilisateur en a fermé le plus ? Donnez son nom d'utilisateur.</question>
      <answer>sarah_dev</answer>
   </qa_pair>
   <qa_pair>
      <question>Cherchez les demandes de fusion qui ont modifié des fichiers du répertoire /api et ont été fusionnées entre le 1er et le 31 janvier 2024. Combien de contributeurs différents y ont travaillé ?</question>
      <answer>7</answer>
   </qa_pair>
   <qa_pair>
      <question>Trouvez le dépôt créé avant 2023 qui a le plus d'étoiles. Quel est le nom du dépôt ?</question>
      <answer>data-pipeline</answer>
   </qa_pair>
</evaluation>
```

## Exemples d'évaluation

### Bonnes questions

**Exemple 1 : question en plusieurs sauts exigeant une exploration approfondie (MCP GitHub)**
```xml
<qa_pair>
   <question>Trouvez le dépôt archivé au 3e trimestre 2023 qui avait auparavant été le projet le plus forké de l'organisation. Quel était le principal langage de programmation de ce dépôt ?</question>
   <answer>Python</answer>
</qa_pair>
```

Cette question est bonne parce que :
- elle exige plusieurs recherches pour trouver les dépôts archivés
- il faut identifier celui qui avait le plus de forks avant l'archivage
- il faut examiner les détails du dépôt pour le langage
- la réponse est une valeur simple et vérifiable
- elle repose sur des données historiques (closes) qui ne changeront pas

**Exemple 2 : exige de comprendre le contexte sans correspondance de mots-clés (MCP de gestion de projet)**
```xml
<qa_pair>
   <question>Trouvez l'initiative visant à améliorer l'accueil des clients, achevée fin 2023. Le responsable du projet a rédigé un bilan après son achèvement. Quel était l'intitulé de son poste à l'époque ?</question>
   <answer>Chef de produit</answer>
</qa_pair>
```

Cette question est bonne parce que :
- elle n'utilise pas le nom précis du projet (« initiative visant à améliorer l'accueil des clients »)
- elle exige de trouver les projets achevés dans une période précise
- il faut identifier le responsable du projet et son poste
- il faut comprendre le contexte à partir des documents de bilan
- la réponse est lisible par un humain et stable
- elle repose sur un travail achevé (qui ne changera pas)

**Exemple 3 : agrégation complexe en plusieurs étapes (MCP de suivi de tickets)**
```xml
<qa_pair>
   <question>Parmi tous les bogues signalés en janvier 2024 et marqués de priorité critique, quel assigné a résolu le plus fort pourcentage de ses bogues en moins de 48 heures ? Donnez son nom d'utilisateur.</question>
   <answer>alex_eng</answer>
</qa_pair>
```

Cette question est bonne parce que :
- elle exige de filtrer les bogues par date, priorité et statut
- il faut regrouper par assigné et calculer des taux de résolution
- il faut comprendre les horodatages pour déterminer les fenêtres de 48 heures
- elle teste la pagination (potentiellement beaucoup de bogues à traiter)
- la réponse est un seul nom d'utilisateur
- elle repose sur des données historiques d'une période précise

**Exemple 4 : exige une synthèse entre plusieurs types de données (MCP de CRM)**
```xml
<qa_pair>
   <question>Trouvez le compte passé de l'offre Starter à l'offre Enterprise au 4e trimestre 2023 avec la plus forte valeur de contrat annuelle. Dans quel secteur ce compte exerce-t-il ?</question>
   <answer>Santé</answer>
</qa_pair>
```

Cette question est bonne parce que :
- elle exige de comprendre les changements de niveau d'abonnement
- il faut repérer les montées en gamme dans une période précise
- il faut comparer des valeurs de contrat
- il faut accéder au secteur du compte
- la réponse est simple et vérifiable
- elle repose sur des transactions historiques terminées

### Mauvaises questions

**Exemple 1 : la réponse change avec le temps**
```xml
<qa_pair>
   <question>Combien de tickets ouverts sont actuellement assignés à l'équipe d'ingénierie ?</question>
   <answer>47</answer>
</qa_pair>
```

Cette question est mauvaise parce que :
- la réponse changera à mesure que des tickets seront créés, fermés ou réassignés
- elle ne repose pas sur des données stables
- elle dépend d'un « état actuel » dynamique

**Exemple 2 : trop facile par recherche de mots-clés**
```xml
<qa_pair>
   <question>Trouvez la demande de fusion intitulée « Add authentication feature » et dites-moi qui l'a créée.</question>
   <answer>developer123</answer>
</qa_pair>
```

Cette question est mauvaise parce que :
- elle se résout par une simple recherche du titre exact
- elle n'exige ni exploration ni compréhension approfondies
- aucune synthèse ni analyse n'est nécessaire

**Exemple 3 : format de réponse ambigu**
```xml
<qa_pair>
   <question>Listez tous les dépôts dont Python est le langage principal.</question>
   <answer>repo1, repo2, repo3, data-pipeline, ml-tools</answer>
</qa_pair>
```

Cette question est mauvaise parce que :
- la réponse est une liste qui peut être renvoyée dans n'importe quel ordre
- elle est difficile à vérifier par comparaison directe de chaînes
- le LLM pourrait la mettre en forme autrement (tableau JSON, séparée par des virgules, par des retours à la ligne)
- mieux vaut demander un agrégat précis (un nombre) ou un superlatif (le plus d'étoiles)

## Processus de vérification

Après avoir créé les évaluations :

1. **Examiner le fichier XML** pour en comprendre le schéma
2. **Charger chaque consigne de tâche** et, en parallèle, avec le serveur MCP et ses outils, trouver la bonne réponse en essayant de résoudre la tâche SOI-MÊME
3. **Signaler toute opération** qui exige une ÉCRITURE ou une opération DESTRUCTIVE
4. **Rassembler toutes les réponses CORRECTES** et remplacer les réponses fausses dans le document
5. **Supprimer toute `<qa_pair>`** qui exige une ÉCRITURE ou une opération DESTRUCTIVE

Penser à paralléliser la résolution des tâches pour ne pas épuiser le contexte, puis rassembler toutes les réponses et modifier le fichier à la fin.

## Conseils pour des évaluations de qualité

1. **Bien réfléchir et planifier** avant de générer les tâches
2. **Paralléliser dès que possible** pour accélérer le processus et gérer le contexte
3. **Se concentrer sur des cas d'usage réalistes** que des humains voudraient vraiment accomplir
4. **Créer des questions difficiles** qui testent les limites des capacités du serveur MCP
5. **Garantir la stabilité** en utilisant des données historiques et des notions closes
6. **Vérifier les réponses** en résolvant soi-même les questions avec les outils du serveur MCP
7. **Itérer et affiner** selon ce qu'on apprend en chemin

---

# Lancer les évaluations

Une fois le fichier d'évaluation créé, on peut utiliser le banc d'évaluation fourni pour tester le serveur MCP.

## Installation

1. **Installer les dépendances**

   ```bash
   pip install -r scripts/requirements.txt
   ```

   Ou manuellement :
   ```bash
   pip install anthropic mcp
   ```

2. **Définir la clé d'API**

   ```bash
   export ANTHROPIC_API_KEY=votre_cle_api
   ```

## Format du fichier d'évaluation

Les fichiers d'évaluation sont au format XML, avec des éléments `<qa_pair>` :

```xml
<evaluation>
   <qa_pair>
      <question>Trouvez le projet créé au 2e trimestre 2024 qui compte le plus de tâches terminées. Quel est le nom du projet ?</question>
      <answer>Refonte du site web</answer>
   </qa_pair>
   <qa_pair>
      <question>Cherchez les tickets étiquetés « bug » fermés en mars 2024. Quel utilisateur en a fermé le plus ? Donnez son nom d'utilisateur.</question>
      <answer>sarah_dev</answer>
   </qa_pair>
</evaluation>
```

## Lancer les évaluations

Le script d'évaluation (`scripts/evaluation.py`) prend en charge trois types de transport :

**Important :**
- **transport stdio** : le script d'évaluation lance et gère automatiquement le processus du serveur MCP. Ne pas lancer le serveur à la main.
- **transports sse / http** : il faut démarrer le serveur MCP séparément avant de lancer l'évaluation. Le script se connecte au serveur déjà lancé, à l'URL indiquée.

### 1. Serveur STDIO local

Pour les serveurs MCP exécutés en local (le script lance le serveur automatiquement) :

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  evaluation.xml
```

Avec des variables d'environnement :
```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  -e API_KEY=abc123 \
  -e DEBUG=true \
  evaluation.xml
```

### 2. Server-Sent Events (SSE)

Pour les serveurs MCP en SSE (il faut d'abord démarrer le serveur) :

```bash
python scripts/evaluation.py \
  -t sse \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  -H "X-Custom-Header: value" \
  evaluation.xml
```

### 3. HTTP (HTTP en flux)

Pour les serveurs MCP en HTTP (il faut d'abord démarrer le serveur) :

```bash
python scripts/evaluation.py \
  -t http \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  evaluation.xml
```

## Options de la ligne de commande

```
usage: evaluation.py [-h] [-t {stdio,sse,http}] [-m MODEL] [-c COMMAND]
                     [-a ARGS [ARGS ...]] [-e ENV [ENV ...]] [-u URL]
                     [-H HEADERS [HEADERS ...]] [-o OUTPUT]
                     eval_file

arguments positionnels :
  eval_file             Chemin du fichier XML d'évaluation

arguments facultatifs :
  -h, --help            Afficher l'aide
  -t, --transport       Type de transport : stdio, sse ou http (défaut : stdio)
  -m, --model           Modèle Claude à utiliser (défaut : claude-3-7-sonnet-20250219)
  -o, --output          Fichier de sortie du rapport (défaut : affichage sur stdout)

options stdio :
  -c, --command         Commande qui lance le serveur MCP (par ex. python, node)
  -a, --args            Arguments de la commande (par ex. server.py)
  -e, --env             Variables d'environnement au format CLE=VALEUR

options sse / http :
  -u, --url             URL du serveur MCP
  -H, --header          En-têtes HTTP au format 'Cle: Valeur'
```

## Sortie

Le script d'évaluation produit un rapport détaillé comprenant :

- **Statistiques de synthèse** :
  - exactitude (correctes / total)
  - durée moyenne par tâche
  - nombre moyen d'appels d'outils par tâche
  - nombre total d'appels d'outils

- **Résultats par tâche** :
  - question et réponse attendue
  - réponse effective de l'agent
  - réponse correcte ou non (✅/❌)
  - durée et détail des appels d'outils
  - résumé par l'agent de sa démarche
  - retour de l'agent sur les outils

### Enregistrer le rapport dans un fichier

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_server.py \
  -o evaluation_report.md \
  evaluation.xml
```

## Exemple complet de bout en bout

Voici un exemple complet de création et de lancement d'une évaluation :

1. **Créer le fichier d'évaluation** (`my_evaluation.xml`) :

```xml
<evaluation>
   <qa_pair>
      <question>Trouvez l'utilisateur qui a créé le plus de tickets en janvier 2024. Quel est son nom d'utilisateur ?</question>
      <answer>alice_developer</answer>
   </qa_pair>
   <qa_pair>
      <question>Parmi toutes les demandes de fusion fusionnées au 1er trimestre 2024, quel dépôt en compte le plus ? Donnez le nom du dépôt.</question>
      <answer>backend-api</answer>
   </qa_pair>
   <qa_pair>
      <question>Trouvez le projet achevé en décembre 2023 qui a duré le plus longtemps du début à la fin. Combien de jours a-t-il duré ?</question>
      <answer>127</answer>
   </qa_pair>
</evaluation>
```

2. **Installer les dépendances** :

```bash
pip install -r scripts/requirements.txt
export ANTHROPIC_API_KEY=votre_cle_api
```

3. **Lancer l'évaluation** :

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a github_mcp_server.py \
  -e GITHUB_TOKEN=ghp_xxx \
  -o github_eval_report.md \
  my_evaluation.xml
```

4. **Étudier le rapport** dans `github_eval_report.md` pour :
   - voir quelles questions ont réussi ou échoué
   - lire le retour de l'agent sur les outils
   - repérer les axes d'amélioration
   - faire évoluer la conception du serveur MCP

## Dépannage

### Erreurs de connexion

En cas d'erreurs de connexion :
- **STDIO** : vérifier que la commande et les arguments sont corrects
- **SSE / HTTP** : vérifier que l'URL est accessible et que les en-têtes sont corrects
- s'assurer que les clés d'API nécessaires sont définies dans les variables d'environnement ou les en-têtes

### Exactitude faible

Si beaucoup d'évaluations échouent :
- étudier le retour de l'agent pour chaque tâche
- vérifier que les descriptions d'outils sont claires et complètes
- vérifier que les paramètres d'entrée sont bien documentés
- se demander si les outils renvoient trop ou trop peu de données
- s'assurer que les messages d'erreur sont exploitables

### Délais dépassés

Si des tâches dépassent le délai :
- utiliser un modèle plus performant (par exemple `claude-3-7-sonnet-20250219`)
- vérifier si les outils renvoient trop de données
- vérifier que la pagination fonctionne correctement
- envisager de simplifier les questions complexes
