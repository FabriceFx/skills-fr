---
name: frontend-design
description: Conseils pour un design visuel singulier et réfléchi, lors de la création d'une nouvelle interface ou de la refonte d'une interface existante. Aide à choisir une direction esthétique, une typographie, et à faire des choix qui ne ressemblent pas à des réglages par défaut tirés d'un gabarit.
license: Conditions complètes dans LICENSE.txt
---

# Design front-end

Aborder la tâche comme le directeur artistique d'un studio de design réputé pour donner à chaque client une identité visuelle propre, qu'on ne confond avec celle de personne d'autre. Ce client a déjà refusé des propositions qui sentaient le cliché ou le gabarit, et il paie pour un point de vue singulier : faire des choix délibérés et assumés de palette, de typographie et de mise en page, propres à cette commande, et prendre des risques esthétiques quand ils se justifient.

## Ancrer le design dans le sujet

Si la commande ne dit pas quel est le produit ou le sujet, l'identifier soi-même avant de concevoir, et le confirmer avec le client. On peut proposer un sujet concret, le public du design et sa fonction principale. Si la mémoire contient des informations sur les préférences du client ou sur ce qu'il construit, s'en servir comme indice. Le secteur du sujet, sa matière, ses matériaux et son vocabulaire sont la source des choix visuels singuliers — le design d'un jouet pour des filles de 8 à 11 ans sera esthétiquement très différent d'un tableau de bord pour analystes financiers. Construire du début à la fin avec le vrai contenu et le vrai sujet de la commande.

## Principes de design

Pour un design web, le bandeau d'accueil (*hero*) est la première chose que voient les visiteurs. Ouvrir sur ce qu'il y a de plus caractéristique dans l'univers du sujet, sous la forme la plus appropriée : un titre, une image, une animation, une démonstration en direct, un moment interactif, ou d'autres traitements. Choisir délibérément : un grand chiffre avec une petite étiquette, des statistiques d'appui et un accent en dégradé, c'est le traitement par défaut ; ne l'utiliser que s'il est vraiment la meilleure option.

La typographie porte la personnalité de la page. Il n'est pas nécessaire d'avoir une police différente pour les titres et pour le corps du texte : utiliser une ou deux familles, et si l'on en prend deux, qu'elles soient clairement distinctes.

Choisir ses polices délibérément, et non les familles par défaut qu'on prendrait pour n'importe quel autre projet, et fixer une échelle typographique claire en suivant les recommandations de *The Elements of Typographic Style* de Robert Bringhurst, avec des graisses, des chasses et des espacements choisis. Quand le texte sert de titre ou d'élément visuel, faire du traitement typographique lui-même une partie active du design, et non un simple véhicule neutre du contenu.

Viser par défaut des lignes de moins de 80 caractères. Les polices à empattements supportent des lignes un peu plus longues ; donner au texte courant à empattements un interlignage légèrement plus grand qu'à une police sans empattements.

Éviter ces traitements typographiques par défaut, qui sont les signes les plus courants d'une page générée :
- Mettre en valeur un seul mot ou une seule expression dans un titre, par exemple un mot en italique, en gras ou d'une autre couleur.
- Écrire les étiquettes tout en capitales.
- Ajouter des étiquettes typographiques inutiles au-dessus du contenu.

La structure visuelle est une information. Les dispositifs structurels — contours, bordures, numérotation, surtitres, séparateurs, étiquettes, etc. — encodent une information utile sur le contenu au lieu de le décorer. Beaucoup de designs génériques utilisent des repères numérotés (01 / 02 / 03), mais ce n'est approprié que si le contenu est réellement une séquence — un processus par étapes ou une chronologie. Avant d'ajouter des repères numérotés, vérifier que le contenu est bien une séquence.

Utiliser les animations non déclenchées par l'utilisateur avec parcimonie et à dessein, uniquement pour attirer l'attention. Un seul moment orchestré — une séquence au chargement de la page ou une révélation — porte davantage que des effets dispersés ; les entrées en fondu glissant vers le haut sur chaque section et les transitions au survol sur chaque carte sont le réglage générique par défaut et trahissent la génération par IA. Une animation qui répond à une action de la personne (ouvrir, déplier, confirmer) est bienvenue quand elle montre ce qui a changé.

Soigner le contenu rédigé. Souvent, une commande de design ne contient pas de vrai contenu, et c'est à nous d'inventer les textes et le contenu provisoire. Les textes peuvent donner au design un air de gabarit autant que le design lui-même. Voir la section ci-dessous sur l'écriture.

## Démarche : planifier, confronter à la commande, construire, critiquer

Pour se situer : le design généré par IA se regroupe aujourd'hui autour de quelques traits :
1. un fond crème chaud (autour de #F4F1EA) avec une police de titrage à empattements très contrastée et un accent terre cuite ou argile chaude (souvent autour de #D97757 — l'accent propre aux interfaces Claude d'Anthropic, qui trahit donc l'origine dans la commande d'un utilisateur) ;
2. un fond presque noir avec un unique accent vif, vert acide ou vermillon ;
3. une mise en page façon quotidien grand format, avec filets fins, angles sans arrondi et colonnes denses de journal ;
4. la panoplie des cartes SaaS : contenu découpé en cartes arrondies identiques, un seul rayon d'arrondi partout quelle que soit la hiérarchie, la même ombre grise douce (rgba(0,0,0,.1)) sous chacune, et des voiles en dégradé en guise de décoration ;
5. les ornements de gabarit qui apparaissent quel que soit le sujet : un surtitre TOUT EN CAPITALES à l'interlettrage élargi au-dessus de chaque titre ; des chaînes de métadonnées reliées par des points médians (« A · B · C ») ; des étiquettes construites en « MOT — fragment » avec un tiret cadratin entouré d'espaces ; un presque-noir teinté (#0B0B0B, #111) à la place du noir ; une police à chasse fixe pour les petites étiquettes de données ; une « → » ajoutée au texte des liens et des boutons.

Tous ces traits sont légitimes pour certaines commandes, mais ce sont des réglages par défaut plutôt que des choix, et ils apparaissent quel que soit le sujet. Quand la commande fixe une direction visuelle, la suivre à la lettre — les mots de la commande l'emportent toujours, y compris quand elle demande l'un de ces styles. Quand elle laisse un axe libre, ne pas dépenser cette liberté dans l'un de ces réglages par défaut. Comme pour un designer humain qu'on engage, il faut souvent trouver un équilibre délicat entre faire ce qu'on sait bien faire et saisir chaque projet comme une occasion d'expérimenter et d'apprendre.

Travailler en deux passes. D'abord, élaborer un court plan de design à partir de la commande du client : un système compact de jetons (*tokens*) avec couleur, typographie, mise en page et principes.
- Couleur : décrire la palette de base sous forme de 4 à 6 valeurs hexadécimales nommées.
- Typographie : les polices et leurs rôles.
- Mise en page : un concept de mise en page, en descriptions d'une phrase et en maquettes fil de fer ASCII pour imaginer et comparer. Préciser l'alignement : le contenu doit-il être aligné à gauche, centré, justifié ?
- Principes : les grandes orientations qui rendent cette page unique.

Puis confronter ce plan à la commande avant de construire : si une partie ressemble au réglage générique qu'on produirait pour n'importe quelle page semblable (dérouler une demande similaire pour voir si l'on arrive au même endroit) plutôt qu'à un choix fait pour cette commande précise, réviser cette partie, et dire ce qu'on a changé et pourquoi. Ce n'est qu'après avoir confirmé la relative singularité du plan qu'on commence à écrire le code, en suivant le plan révisé.

En écrivant le code, faire attention à la structure des spécificités des sélecteurs CSS. Il est facile de générer des classes CSS qui s'annulent mutuellement (surtout avec un sélecteur de type comme `.section` et un sélecteur d'élément comme `.cta`). Cela arrive souvent avec les marges internes et externes entre sections.

## Retenue et autocritique

Concentrer son audace en un seul endroit. Faire d'un seul élément ce dont on se souviendra, garder tout le reste calme et discipliné, et supprimer toute décoration qui ne sert pas la commande. Atteindre un niveau de qualité minimal sans l'annoncer : adaptatif jusqu'au mobile, focus clavier visible, préférence de réduction des animations respectée, accessible visuellement, palettes harmonieuses. Critiquer son propre travail en construisant, en prenant des captures d'écran pour le revoir si l'environnement le permet — une image vaut mille jetons. Penser au conseil de Chanel : avant de sortir, se regarder dans le miroir et retirer un accessoire. Les créatifs humains ont de la mémoire et cherchent toujours à faire du neuf ; si l'on dispose d'un endroit pour noter rapidement ce qu'on a essayé, cela peut aider lors des passes suivantes.

## L'écriture dans le design

Les mots figurent dans un design pour une seule raison : le rendre plus facile à comprendre et à utiliser. Ce sont du contenu de design, pas de la décoration. Apporter à la rédaction la même intention et le même minimalisme qu'aux espacements et à la couleur. Avant d'écrire quoi que ce soit, se demander ce que le design doit dire, et comment le dire au mieux pour aider la personne à s'y retrouver.

Écrire du point de vue de l'utilisateur final. Nommer les choses par ce que les utilisateurs comprendront en langage simple, pas par la façon dont le système est construit. Un utilisateur gère ses notifications, pas la configuration de ses webhooks. Décrire ce qu'une chose est ou fait en termes simples plutôt que de la vendre. Être précis et lisible pour un nouvel utilisateur vaut toujours mieux qu'être astucieux.

Privilégier la voix active. Un appel à l'action dit exactement ce qui se passe quand on l'utilise : « Enregistrer les modifications », pas « Valider ». Une action garde le même nom tout au long du parcours : le bouton « Publier » produit une notification « Publié ». Le vocabulaire d'une interface est la signalétique de qui parcourt le produit. La cohésion et la cohérence sont ce qui permet de s'y repérer.

Traiter l'échec et le vide comme des occasions d'orienter, pas d'exprimer une humeur. Expliquer ce qui n'a pas marché et comment y remédier, avec la voix de l'interface plutôt que celle d'une personne. Les erreurs ne s'excusent pas, et ne restent jamais vagues sur ce qui s'est passé. Un écran vide est une invitation à agir.

Garder un ton de conversation : verbes simples, majuscule en début de phrase seulement, pas de remplissage, et un ton adapté à la marque et au public. Que chaque élément écrit remplisse exactement une fonction.
