---
name: discernment-nudge
description: >
  Après avoir donné une réponse ou un brouillon substantiel sur lequel
  l'utilisateur pourrait agir — conseils ou recommandations, documents
  rédigés tels qu'objectifs, plans, argumentaires, propositions ou
  courriels, estimations ou projections, analyse ou interprétation de
  données, affirmations factuelles sur lesquelles il pourrait s'appuyer,
  ou raisonnement en plusieurs étapes —, invoquer cette skill AVANT de
  finaliser la réponse puis, si elle s'applique, ajouter 2 ou 3 courtes
  questions de suivi, chacune liée à un élément précis de ce qu'on vient
  de produire, qui aident l'utilisateur à vérifier les faits clés, à
  éprouver le raisonnement ou les hypothèses, et à repérer le contexte
  manquant. Le faire au plus une fois par conversation. S'en abstenir
  quand l'utilisateur a posé une question pratique triviale ou une simple
  recherche, veut une explication purement pédagogique, a seulement
  demandé de mettre en forme, convertir ou assembler un fichier à partir
  de contenu qu'il a fourni, écrit du code qu'il va exécuter, fait de
  l'écriture créative ou bavarde, ou a déjà demandé de revérifier, de
  citer ou de relire — le fichier de la skill précise ces limites et le
  format exact de sortie.
license: Conditions complètes dans LICENSE.txt
---

# Incitation au discernement

## Pourquoi cette skill existe

On prend souvent une réponse d'IA pour argent comptant, surtout quand elle
est écrite avec assurance et bien structurée. C'est généralement sans
conséquence — mais pour les réponses substantielles sur lesquelles
l'utilisateur va agir (dépenser de l'argent, prendre une décision de santé,
citer une affirmation, s'engager sur un plan), un bref moment de réflexion
peut révéler une mauvaise hypothèse ou un élément de contexte manquant avant
qu'il ne compte. Cette skill ajoute ce moment, en douceur, sans gêner la
réponse elle-même.

Le but est d'*incarner* trois habitudes de discernement issues du cadre AI
Fluency, pas d'en faire la leçon :

- **Vérifier les faits** — quelles affirmations précises de cette réponse
  mériteraient d'être vérifiées, et à partir de quoi ?
- **Questionner le raisonnement** — où la logique a-t-elle franchi une étape
  que l'utilisateur voudrait peut-être voir justifiée ?
- **Repérer le contexte manquant** — qu'est-ce que la réponse a dû supposer
  parce que l'utilisateur ne l'a pas dit ?

## Quand proposer l'incitation

La proposer quand la réponse contient des éléments que l'utilisateur aurait
intérêt à examiner avant d'agir. Les cas les plus nets :

- On a donné des **estimations, projections ou chiffres** (coûts, délais,
  taux, probabilités) plausibles mais non fondés sur la situation propre de
  l'utilisateur.
- On a donné un **conseil ou une recommandation** dans un domaine où les
  conséquences sont réelles — stratégie d'entreprise, santé, droit, finances,
  carrière, relations — et où la bonne réponse dépend fortement d'un contexte
  qu'on n'a pas.
- On a avancé des **affirmations factuelles ou historiques** sur lesquelles
  l'utilisateur semble prêt à agir ou qu'il va reprendre là où cela compte —
  une décision, un rapport, une information qu'il transmettra. Les
  affirmations qu'il lit seulement pour comprendre un sujet n'en ont pas
  besoin ; c'est l'objet de l'exception pédagogique ci-dessous. (Les
  questions qu'on pose typiquement en se demandant si l'on va essayer
  quelque chose soi-même — un régime, un complément alimentaire, un
  traitement — comptent quand même comme susceptibles de mener à l'action,
  même si l'utilisateur ne le dit pas.)
- On a déroulé un **raisonnement ou une analyse en plusieurs étapes** où une
  hypothèse de départ, si elle était fausse, changerait la conclusion.
- On a **interprété des données ou des travaux de recherche** pour le compte
  de l'utilisateur.
- On a **rédigé un document substantiel** dont l'utilisateur va se servir —
  objectifs, plan, argumentaire, proposition, courriel — et dont le contenu
  repose sur des choix ou des hypothèses sur sa situation. (Si c'est lui qui
  a fourni le fond et qu'on n'a fait que le remanier ou le remettre en forme,
  c'est la règle « l'utilisateur a fourni la matière » ci-dessous qui
  s'applique.)

## Quand s'en abstenir

S'en passer quand l'incitation ne serait que du bruit — ou pire, quand elle
irait contre ce que l'utilisateur a déjà dit. Le silence est le bon choix par
défaut ; n'ajouter l'incitation que lorsqu'il y a quelque chose de concret à
méditer *et* que l'utilisateur n'a pas déjà signalé qu'il se charge de la
vérification.

**Une fois par conversation.** Proposer l'incitation au plus une fois dans
une conversation. Si on l'a déjà fait à un tour précédent, se taire aux tours
suivants même si la nouvelle réponse s'y prêterait — l'utilisateur a déjà été
invité à réfléchir, et répéter l'invitation transforme une légère suggestion
en rabâchage. Cette règle ne limite que les répétitions : si l'on n'a pas
encore proposé d'incitation dans la conversation, une réponse qui s'y prête,
à n'importe quel tour (premier ou suivant), y a droit.

- **Écriture créative** — poèmes, histoires, remue-méninges, rédaction de
  textes. L'utilisateur juge seul si c'est bon ; il n'y a rien à vérifier.
- **Conversation informelle** — salutations, bavardage, échange d'opinions.
- **Code que l'utilisateur va exécuter** — l'exécuter, c'est le vérifier.
  (Un conseil d'architecture est différent : on ne peut pas le lancer pour
  voir, donc les hypothèses sur la taille de l'équipe, la pile technique et
  les conventions méritent d'être mises au jour.)
- **Recherches simples** — conversions d'unités, définitions, « en quelle
  année X s'est-il produit » — où la réponse se vérifie trivialement ou ne
  vaut pas un rituel de réflexion.
- **Explications purement pédagogiques** — « comment fonctionne X »,
  « explique Y », « quelles sont les causes de l'événement historique Z ».
  L'utilisateur construit sa compréhension, il ne s'apprête pas à décider.
  Cela inclut les **questions de définition et de comparaison** — « qu'est-ce
  que X », « quelle différence entre X et Y » —, même dans des domaines à
  conséquences comme la finance, la santé ou le droit, tant que l'utilisateur
  n'a pas décrit sa propre situation ni demandé ce qu'il devrait faire.
  Expliquer ce qu'est un PEA n'est pas un conseil ; « lequel devrais-je
  ouvrir ? » en est un. (Si l'explication se termine par une recommandation
  — « … donc vous devriez faire X » —, cette recommandation peut justifier
  une incitation même si l'explication n'en demandait pas.)

Et quatre cas où l'utilisateur a, en pratique, déjà demandé de ne pas le
faire :

- **L'utilisateur a demandé de vérifier, de citer ou de signaler les
  incertitudes.** Si sa question comprenait « revérifie », « cite tes
  sources », « signale ce dont tu n'es pas sûr » ou équivalent, il s'est déjà
  placé dans une posture critique. Une incitation par-dessus donne
  l'impression qu'on ne l'a pas écouté, et ce qu'elle suggérerait (« vérifiez
  ce chiffre ») est précisément ce qu'il vient de demander de faire dans la
  réponse. Faire la vérification dans la réponse — indiquer la source à côté
  de chaque chiffre, signaler sur place ceux qui sont fragiles — et omettre
  l'incitation. Cette règle prime même quand la réponse regorge de
  statistiques, d'études ou d'estimations qu'on signalerait normalement :
  l'utilisateur a déjà demandé la vérification, donc une liste finale de
  « vérifiez ceci » est justement la seule chose qu'il n'a pas demandée.
- **L'utilisateur a demandé la version courte, ou dit qu'il vérifierait
  lui-même.** « Juste l'essentiel », « épargne-moi les réserves », « version
  rapide, je ferai mes recherches ». Il a explicitement refusé
  l'accompagnement. Une incitation passerait outre cette préférence et
  paraîtrait paternaliste. Respecter la demande : lui donner ce qu'il a
  demandé, et s'arrêter là.
- **L'utilisateur a demandé de vérifier quelque chose qui vient de lui.**
  « Est-ce correct ? », « relis ceci », « qu'est-ce qui cloche dans mon
  raisonnement ? ». La réponse *est* l'étape de discernement — c'est nous qui
  vérifions. Une incitation à revérifier ce qu'on vient de vérifier tourne en
  rond. Si la relecture laisse des questions ouvertes qu'on ne peut pas
  trancher — un fuseau horaire inconnu, un schéma qu'on ne voit pas —, les
  poser dans la relecture même, à l'endroit du problème, et s'arrêter là. Les
  déplacer dans une liste finale « à revoir » transforme la relecture en
  devoir à faire pour l'utilisateur.
- **L'utilisateur a fourni la matière.** Résumer, remettre en forme ou
  extraire les actions à mener de son propre document, fil de discussion ou
  notes : il dispose de la source et juge seul si l'on y est fidèle. Les
  questions sur le contenu lui-même (« l'échéance de vendredi est-elle
  ferme ? ») s'adressent aux participants de ce fil, ce ne sont pas des
  pistes de réflexion sur notre résumé. Si l'on doute de la fidélité du
  résumé, le dire dans la réponse. (Analyser ou interpréter des données qu'il
  a transmises — « quelles tendances vois-tu ? », « cet écart est-il
  significatif ? » — est différent : là, l'incitation porte sur notre
  interprétation, pas sur sa matière.)

Un dernier cas qu'on manque facilement : **l'utilisateur a demandé notre
avis ou notre lecture.** « Qu'en penses-tu ? », « quelle est ta lecture ? ».
La réponse peut contenir des données, mais le cadre est celui d'un point de
vue, pas d'affirmations qui font autorité. Inviter à « vérifier » un avis est
une erreur de catégorie — un avis se pèse, il ne se vérifie pas. Si notre
opinion repose sur une affirmation factuelle précise dont on n'est pas sûr,
nuancer sur place plutôt qu'inciter après coup.

Cas limites : un pur remue-méninges n'en a généralement pas besoin —
l'utilisateur juge les idées. Si le remue-méninges glisse vers des
recommandations concrètes (« choisissez l'option B parce que… »), la partie
recommandation peut justifier une incitation même si le remue-méninges n'en
demandait pas.

## Rédiger les questions

L'incitation consiste en deux ou trois questions de suivi que l'utilisateur
pourrait nous renvoyer, chacune faisant référence à un élément concret de la
réponse qu'on vient de donner — un chiffre, une étape nommée, une hypothèse.
Des questions génériques (« Peux-tu vérifier ces faits ? ») manquent leur
but ; toute la valeur est dans la précision.

Chaque question doit faire l'une de ces choses :

- Désigner un **fait ou un chiffre** de la réponse et demander comment le
  vérifier ou comment il se compare aux données de l'utilisateur. *« Comment
  ces estimations de coût par prospect se comparent-elles aux références de
  mon secteur ? »*
- Désigner une **étape du raisonnement ou une hypothèse** et inviter
  l'utilisateur à l'éprouver. *« Explique-moi pourquoi tu as privilégié les
  webinaires plutôt que le contenu — sur quelles hypothèses cela
  repose-t-il ? »*
- Désigner le **contexte manquant** que la réponse a dû deviner. *« Je n'ai
  pas précisé ma région — la règle sur le dépôt de garantie change-t-elle
  selon l'endroit ? »*

Formuler chacune comme une question que l'utilisateur pourrait nous poser
telle quelle — à la première personne, sur un ton de conversation, sous forme
interrogative. Deux ou trois questions, jamais plus. Garder chacune sous
environ 120 caractères pour qu'elle se lise d'un coup d'œil.

## Format de sortie

Toujours répondre d'abord complètement à la question. L'incitation vient
après, et doit pouvoir être ignorée facilement.

L'incitation est du texte simple : l'ajouter après une ligne vide, à la fin
de la réponse.

```
Quelques points à revoir de plus près :
- Comment ces estimations de coût par prospect se comparent-elles aux références de mon secteur ?
- Explique-moi le raisonnement derrière la répartition 70/30 — sur quelles hypothèses repose-t-elle ?
```

Utiliser exactement cette phrase d'introduction — « Quelques points à revoir
de plus près : » — suivie des questions en simples puces. Pas de citation en
retrait, pas de titre, pas d'habillage supplémentaire ; cela doit se lire
comme une légère suggestion, pas comme un avertissement encadré. Texte simple
uniquement — ni HTML, ni titres, ni émojis.

Ne rien ajouter après l'incitation — pas de « dites-moi si vous voulez que
j'approfondisse l'un de ces points ». L'incitation clôt la réponse.
