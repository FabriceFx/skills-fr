---
name: doc-coauthoring
description: Guide l'utilisateur dans une démarche structurée de corédaction de documents. À utiliser quand l'utilisateur veut rédiger de la documentation, des propositions, des spécifications techniques, des notes de décision ou tout contenu structuré semblable. Cette démarche aide l'utilisateur à transmettre efficacement son contexte, à affiner le contenu par itérations et à vérifier que le document fonctionne pour ses lecteurs. Se déclenche quand l'utilisateur parle de rédiger une documentation, de créer une proposition, d'ébaucher une spécification ou de tâches de documentation semblables.
---

# Démarche de corédaction de documents

Cette skill fournit une démarche structurée pour guider l'utilisateur dans la création collaborative d'un document. Se comporter en guide actif et accompagner l'utilisateur à travers trois étapes : collecte du contexte, affinage et structure, test par un lecteur.

## Quand proposer cette démarche

**Conditions de déclenchement :**
- L'utilisateur parle de rédiger un document : « rédige un document », « ébauche une proposition », « crée une spécification », « mets par écrit »
- L'utilisateur cite un type de document précis : « PRD » (cahier des exigences produit), « document de conception », « note de décision », « RFC »
- L'utilisateur semble entamer une tâche de rédaction conséquente

**Proposition initiale :**
Proposer à l'utilisateur une démarche structurée de corédaction. Expliquer les trois étapes :

1. **Collecte du contexte** : l'utilisateur fournit tout le contexte utile, pendant que Claude pose des questions de clarification
2. **Affinage et structure** : construire chaque section par itérations, par remue-méninges et retouches
3. **Test par un lecteur** : tester le document avec un Claude neuf (sans contexte) pour repérer les angles morts avant que d'autres ne le lisent

Expliquer que cette approche aide à garantir que le document fonctionne bien quand d'autres le lisent (y compris quand ils le collent dans Claude). Demander si l'utilisateur veut essayer cette démarche ou préfère travailler librement.

S'il refuse, travailler librement. S'il accepte, passer à l'étape 1.

## Étape 1 : collecte du contexte

**Objectif :** combler l'écart entre ce que sait l'utilisateur et ce que sait Claude, pour pouvoir le guider intelligemment ensuite.

### Questions initiales

Commencer par demander à l'utilisateur le méta-contexte du document :

1. De quel type de document s'agit-il ? (par exemple spécification technique, note de décision, proposition)
2. Quel est le public principal ?
3. Quel effet doit-il produire sur celui qui le lit ?
4. Y a-t-il un gabarit ou un format précis à suivre ?
5. D'autres contraintes ou éléments de contexte à connaître ?

Lui indiquer qu'il peut répondre en style télégraphique ou déverser l'information comme cela lui convient le mieux.

**Si l'utilisateur fournit un gabarit ou cite un type de document :**
- Demander s'il a un document gabarit à partager
- S'il donne le lien d'un document partagé, utiliser l'intégration appropriée pour le récupérer
- S'il fournit un fichier, le lire

**Si l'utilisateur parle de modifier un document partagé existant :**
- Utiliser l'intégration appropriée pour en lire l'état actuel
- Rechercher les images sans texte alternatif
- S'il y en a, expliquer que lorsque d'autres utiliseront Claude pour comprendre le document, Claude ne pourra pas les voir. Demander s'il veut qu'un texte alternatif soit généré. Si oui, lui demander de coller chaque image dans la conversation pour en générer une description.

### Déversement d'informations

Une fois les questions initiales traitées, encourager l'utilisateur à déverser tout le contexte dont il dispose. Demander par exemple :
- l'historique du projet ou du problème
- les discussions d'équipe ou documents partagés liés
- pourquoi les solutions alternatives ne sont pas retenues
- le contexte organisationnel (dynamique d'équipe, incidents passés, enjeux politiques)
- les contraintes ou pressions de calendrier
- l'architecture technique ou les dépendances
- les préoccupations des parties prenantes

Lui conseiller de ne pas se soucier de l'organisation — juste tout sortir. Proposer plusieurs façons de fournir le contexte :
- un déversement au fil de la pensée
- indiquer des canaux ou des fils d'équipe à lire
- donner des liens vers des documents partagés

**Si des intégrations sont disponibles** (par exemple Slack, Teams, Google Drive, SharePoint ou d'autres serveurs MCP), signaler qu'elles peuvent servir à récupérer directement du contexte.

**Si aucune intégration n'est détectée et qu'on est dans Claude.ai ou l'application Claude :** suggérer d'activer des connecteurs dans les réglages de Claude pour pouvoir récupérer directement du contexte depuis les messageries et les espaces de stockage de documents.

L'informer que des questions de clarification lui seront posées une fois son premier déversement terminé.

**Pendant la collecte du contexte :**

- Si l'utilisateur mentionne des canaux d'équipe ou des documents partagés :
  - si des intégrations sont disponibles : l'informer que le contenu va être lu tout de suite, puis utiliser l'intégration appropriée
  - sinon : expliquer qu'on n'y a pas accès. Suggérer d'activer des connecteurs dans les réglages de Claude, ou de coller directement le contenu utile.

- Si l'utilisateur mentionne des entités ou des projets inconnus :
  - demander s'il faut chercher dans les outils connectés pour en savoir plus
  - attendre sa confirmation avant de chercher

- À mesure que l'utilisateur fournit du contexte, suivre ce qu'on apprend et ce qui reste flou

**Poser des questions de clarification :**

Quand l'utilisateur signale avoir terminé son premier déversement (ou après un apport de contexte substantiel), poser des questions de clarification pour s'assurer d'avoir compris :

Générer 5 à 10 questions numérotées à partir des lacunes du contexte.

L'informer qu'il peut répondre en style télégraphique (par exemple « 1 : oui, 2 : voir #canal, 3 : non, à cause de la rétrocompatibilité »), donner des liens vers d'autres documents, indiquer des canaux à lire, ou simplement continuer à déverser. Ce qui est le plus efficace pour lui.

**Condition de sortie :**
Le contexte est suffisant quand les questions montrent qu'on a compris — quand on peut interroger sur les cas limites et les compromis sans avoir besoin qu'on explique les bases.

**Transition :**
Demander s'il veut apporter encore du contexte à ce stade, ou s'il est temps de passer à la rédaction.

S'il veut en ajouter, le laisser faire. Quand il est prêt, passer à l'étape 2.

## Étape 2 : affinage et structure

**Objectif :** construire le document section par section, par remue-méninges, sélection et affinage itératif.

**Consignes à l'utilisateur :**
Expliquer que le document sera construit section par section. Pour chaque section :
1. des questions de clarification seront posées sur ce qu'il faut inclure
2. 5 à 20 options seront proposées en remue-méninges
3. l'utilisateur indiquera quoi garder, retirer ou combiner
4. la section sera rédigée
5. elle sera affinée par des retouches ciblées

Commencer par la section qui comporte le plus d'inconnues (en général la décision ou la proposition centrale), puis traiter les autres.

**Ordre des sections :**

Si la structure du document est claire :
Demander par quelle section il souhaite commencer.

Suggérer de commencer par celle qui comporte le plus d'inconnues. Pour une note de décision, c'est en général la proposition centrale. Pour une spécification, c'est d'ordinaire l'approche technique. Mieux vaut garder les sections de synthèse pour la fin.

Si l'utilisateur ne sait pas de quelles sections il a besoin :
À partir du type de document et du gabarit, suggérer 3 à 5 sections adaptées.

Demander si cette structure lui convient ou s'il veut l'ajuster.

**Une fois la structure validée :**

Créer la structure initiale du document avec un texte provisoire pour chaque section.

**Si les artefacts sont disponibles :**
Utiliser `create_file` pour créer un artefact. Cela donne à Claude comme à l'utilisateur une ossature de travail.

L'informer que la structure initiale, avec un texte provisoire pour chaque section, va être créée.

Créer l'artefact avec tous les titres de section et un bref texte provisoire comme « [À rédiger] » ou « [Contenu ici] ».

Donner le lien de l'ossature et indiquer qu'il est temps de remplir chaque section.

**Si les artefacts ne sont pas disponibles :**
Créer un fichier Markdown dans le répertoire de travail. Lui donner un nom adapté (par exemple `note-de-decision.md`, `specification-technique.md`).

L'informer que la structure initiale, avec un texte provisoire pour chaque section, va être créée.

Créer le fichier avec tous les titres de section et un texte provisoire.

Confirmer le nom du fichier créé et indiquer qu'il est temps de remplir chaque section.

**Pour chaque section :**

### Étape 1 : questions de clarification

Annoncer qu'on commence la section [NOM DE LA SECTION]. Poser 5 à 10 questions de clarification sur ce qu'elle doit contenir :

Générer 5 à 10 questions précises à partir du contexte et de l'objet de la section.

L'informer qu'il peut répondre en style télégraphique ou simplement indiquer ce qu'il est important de couvrir.

### Étape 2 : remue-méninges

Pour la section [NOM DE LA SECTION], proposer [5 à 20] éléments susceptibles d'y figurer, selon sa complexité. Chercher :
- du contexte partagé qui aurait pu être oublié
- des angles ou des considérations pas encore évoqués

Générer 5 à 20 options numérotées selon la complexité de la section. À la fin, proposer d'en générer davantage s'il le souhaite.

### Étape 3 : sélection

Demander quels points garder, retirer ou combiner. Demander de brèves justifications, pour apprendre ses priorités en vue des sections suivantes.

Donner des exemples :
- « Garder 1, 4, 7, 9 »
- « Retirer 3 (doublon de 1) »
- « Retirer 6 (le public le sait déjà) »
- « Combiner 11 et 12 »

**Si l'utilisateur donne un retour libre** (par exemple « ça me va » ou « j'aime l'essentiel, mais… ») au lieu de numéros, en extraire ses préférences et continuer. Déterminer ce qu'il veut garder, retirer ou changer, et l'appliquer.

### Étape 4 : recherche des manques

D'après sa sélection, demander s'il manque quelque chose d'important pour la section [NOM DE LA SECTION].

### Étape 5 : rédaction

Utiliser `str_replace` pour remplacer le texte provisoire de cette section par le contenu rédigé.

Annoncer que la section [NOM DE LA SECTION] va être rédigée à partir de sa sélection.

**Avec un artefact :**
Après la rédaction, donner le lien de l'artefact.

Lui demander de la relire et d'indiquer ce qu'il faut changer. Préciser qu'être précis aide à apprendre pour les sections suivantes.

**Avec un fichier (sans artefact) :**
Après la rédaction, confirmer qu'elle est terminée.

L'informer que la section [NOM DE LA SECTION] a été rédigée dans [nom du fichier]. Lui demander de la relire et d'indiquer ce qu'il faut changer. Préciser qu'être précis aide à apprendre pour les sections suivantes.

**Consigne clé pour l'utilisateur (à inclure lors de la rédaction de la première section) :**
Ajouter une remarque : plutôt que de modifier lui-même le document, lui demander d'indiquer ce qu'il faut changer. Cela permet d'apprendre son style pour les sections suivantes. Par exemple : « Retire la puce X, déjà couverte par Y » ou « Rends le troisième paragraphe plus concis ».

### Étape 6 : affinage itératif

À mesure que l'utilisateur fait des retours :
- utiliser `str_replace` pour les retouches (ne jamais réimprimer tout le document)
- **avec un artefact :** donner le lien de l'artefact après chaque retouche
- **avec un fichier :** simplement confirmer que les retouches sont faites
- si l'utilisateur modifie lui-même le document et demande de le relire : noter mentalement ses changements et les garder à l'esprit pour les sections suivantes (ils révèlent ses préférences)

**Continuer à itérer** jusqu'à ce que l'utilisateur soit satisfait de la section.

### Contrôle de qualité

Après 3 itérations consécutives sans changement substantiel, demander si quelque chose peut être retiré sans perdre d'information importante.

Quand la section est terminée, confirmer que [NOM DE LA SECTION] est achevée. Demander s'il est prêt à passer à la section suivante.

**Répéter pour toutes les sections.**

### À l'approche de la fin

À l'approche de la fin (80 % des sections ou plus), annoncer qu'on va relire tout le document pour vérifier :
- la fluidité et la cohérence entre les sections
- les redites ou les contradictions
- tout ce qui ressemble à du remplissage générique, à de l'« AI slop »
- que chaque phrase a du poids

Lire tout le document et faire un retour.

**Quand toutes les sections sont rédigées et affinées :**
Annoncer que toutes les sections sont rédigées. Indiquer qu'on va relire une dernière fois le document complet.

Vérifier la cohérence d'ensemble, la fluidité, la complétude.

Faire les dernières suggestions éventuelles.

Demander s'il est prêt à passer au test par un lecteur, ou s'il veut affiner autre chose.

## Étape 3 : test par un lecteur

**Objectif :** tester le document avec un Claude neuf (sans fuite de contexte) pour vérifier qu'il fonctionne pour ses lecteurs.

**Consignes à l'utilisateur :**
Expliquer qu'on va maintenant tester si le document fonctionne réellement pour ses lecteurs. Cela révèle les angles morts — ce qui paraît clair aux auteurs mais pourrait dérouter les autres.

### Méthode de test

**Si des sous-agents sont disponibles (par exemple dans Claude Code) :**

Faire le test directement, sans faire intervenir l'utilisateur.

### Étape 1 : prévoir les questions des lecteurs

Annoncer qu'on va prévoir les questions que des lecteurs pourraient poser en cherchant ce document.

Générer 5 à 10 questions que des lecteurs poseraient réellement.

### Étape 2 : tester avec un sous-agent

Annoncer que ces questions vont être testées avec une instance neuve de Claude (sans aucun contexte de cette conversation).

Pour chaque question, invoquer un sous-agent avec uniquement le contenu du document et la question.

Résumer, pour chaque question, ce que le Claude lecteur a compris juste ou de travers.

### Étape 3 : vérifications supplémentaires

Annoncer que des vérifications supplémentaires vont être faites.

Invoquer un sous-agent pour rechercher les ambiguïtés, les hypothèses erronées, les contradictions.

Résumer les problèmes trouvés.

### Étape 4 : rendre compte et corriger

Si des problèmes sont trouvés :
Signaler que le Claude lecteur a buté sur des points précis.

Lister ces points.

Indiquer qu'on va combler ces lacunes.

Revenir à l'affinage pour les sections problématiques.

---

**Sans accès aux sous-agents (par exemple dans l'interface web de claude.ai) :**

L'utilisateur devra faire le test manuellement.

### Étape 1 : prévoir les questions des lecteurs

Demander quelles questions on pourrait poser en cherchant ce document. Que taperait-on dans Claude.ai ?

Générer 5 à 10 questions que des lecteurs poseraient réellement.

### Étape 2 : préparer le test

Donner les instructions de test :
1. Ouvrir une nouvelle conversation Claude : https://claude.ai
2. Coller ou partager le contenu du document (si l'on utilise une plateforme de documents partagés avec des connecteurs activés, donner le lien)
3. Poser au Claude lecteur les questions générées

Pour chaque question, demander au Claude lecteur de fournir :
- la réponse
- ce qui lui a paru ambigu ou flou
- les connaissances ou le contexte que le document suppose acquis

Vérifier si le Claude lecteur répond juste ou interprète mal quelque chose.

### Étape 3 : vérifications supplémentaires

Demander aussi au Claude lecteur :
- « Qu'est-ce qui, dans ce document, pourrait paraître ambigu ou flou aux lecteurs ? »
- « Quelles connaissances ou quel contexte ce document suppose-t-il chez ses lecteurs ? »
- « Y a-t-il des contradictions ou des incohérences internes ? »

### Étape 4 : itérer selon les résultats

Demander sur quoi le Claude lecteur s'est trompé ou a buté. Indiquer qu'on va combler ces lacunes.

Revenir à l'affinage pour les sections problématiques.

---

### Condition de sortie (dans les deux cas)

Quand le Claude lecteur répond juste de façon constante et ne fait plus apparaître de nouvelles lacunes ou ambiguïtés, le document est prêt.

## Relecture finale

Quand le test par un lecteur est concluant :
Annoncer que le document a passé le test du Claude lecteur. Avant de conclure :

1. Recommander à l'utilisateur une dernière relecture personnelle — le document lui appartient et il est responsable de sa qualité
2. Suggérer de revérifier les faits, les liens et les détails techniques
3. Lui demander de vérifier que le document produit l'effet voulu

Demander s'il veut une relecture de plus, ou si le travail est terminé.

**S'il veut une relecture finale, la faire. Sinon :**
Annoncer que le document est terminé. Donner quelques derniers conseils :
- envisager de mettre en annexe un lien vers cette conversation, pour que les lecteurs voient comment le document a été élaboré
- utiliser des annexes pour approfondir sans alourdir le corps du document
- mettre le document à jour à mesure que de vrais lecteurs font des retours

## Conseils pour bien guider

**Ton :**
- être direct et méthodique
- expliquer brièvement les raisons quand elles influent sur ce que fait l'utilisateur
- ne pas chercher à « vendre » la démarche — simplement l'appliquer

**Gérer les écarts :**
- si l'utilisateur veut sauter une étape : lui demander s'il veut la passer et rédiger librement
- s'il semble agacé : reconnaître que cela prend plus de temps que prévu, et suggérer des moyens d'aller plus vite
- toujours lui laisser la main pour ajuster le processus

**Gestion du contexte :**
- tout au long du travail, si le contexte manque sur un point évoqué, demander de soi-même
- ne pas laisser les lacunes s'accumuler — les traiter au fur et à mesure

**Gestion de l'artefact :**
- utiliser `create_file` pour rédiger des sections entières
- utiliser `str_replace` pour toutes les retouches
- donner le lien de l'artefact après chaque changement
- ne jamais utiliser d'artefact pour les listes de remue-méninges — cela reste dans la conversation

**La qualité avant la vitesse :**
- ne pas se précipiter d'une étape à l'autre
- chaque itération doit apporter une amélioration réelle
- le but est un document qui fonctionne vraiment pour ses lecteurs
