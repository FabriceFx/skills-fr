---
name: slack-gif-creator
description: Connaissances et utilitaires pour créer des GIF animés optimisés pour Slack. Fournit les contraintes, des outils de validation et des concepts d'animation. À utiliser quand l'utilisateur demande un GIF animé pour Slack, par exemple « fais-moi un GIF de X en train de faire Y pour Slack ».
license: Conditions complètes dans LICENSE.txt
---

# Créateur de GIF pour Slack

Une boîte à outils qui fournit utilitaires et connaissances pour créer des GIF animés optimisés pour Slack.

## Exigences de Slack

**Dimensions :**
- GIF d'émoji : 128 × 128 (recommandé)
- GIF de message : 480 × 480

**Paramètres :**
- Images par seconde : 10 à 30 (moins = fichier plus léger)
- Couleurs : 48 à 128 (moins = fichier plus léger)
- Durée : rester sous 3 secondes pour les GIF d'émoji

## Démarche de base

```python
from core.gif_builder import GIFBuilder
from PIL import Image, ImageDraw

# 1. Créer le constructeur
builder = GIFBuilder(width=128, height=128, fps=10)

# 2. Générer les images
for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))
    draw = ImageDraw.Draw(frame)

    # Dessiner l'animation avec les primitives de PIL
    # (cercles, polygones, lignes, etc.)

    builder.add_frame(frame)

# 3. Enregistrer avec optimisation
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

## Dessiner les graphismes

### Travailler avec des images envoyées par l'utilisateur
Si l'utilisateur envoie une image, se demander s'il veut :
- **L'utiliser directement** (par exemple « anime ceci », « découpe ceci en images »)
- **S'en inspirer** (par exemple « fais quelque chose dans ce genre »)

Charger et manipuler les images avec PIL :
```python
from PIL import Image

uploaded = Image.open('file.png')
# L'utiliser directement, ou seulement comme référence de couleurs / de style
```

### Dessiner à partir de rien
Pour dessiner des graphismes à partir de rien, utiliser les primitives `ImageDraw` de PIL :

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(frame)

# Cercles / ovales
draw.ellipse([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)

# Étoiles, triangles, tout polygone
points = [(x1, y1), (x2, y2), (x3, y3), ...]
draw.polygon(points, fill=(r, g, b), outline=(r, g, b), width=3)

# Lignes
draw.line([(x1, y1), (x2, y2)], fill=(r, g, b), width=5)

# Rectangles
draw.rectangle([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)
```

**À éviter :** les polices d'émojis (peu fiables d'une plateforme à l'autre), et supposer que cette skill contient des graphismes tout faits.

### Soigner les graphismes

Les graphismes doivent paraître soignés et créatifs, pas rudimentaires. Voici comment :

**Utiliser des traits plus épais** — toujours `width=2` ou plus pour les contours et les lignes. Les traits fins (`width=1`) paraissent hachés et amateurs.

**Ajouter de la profondeur** :
- Utiliser des dégradés pour les fonds (`create_gradient_background`)
- Superposer plusieurs formes pour plus de richesse (par exemple une étoile avec une plus petite étoile à l'intérieur)

**Rendre les formes plus intéressantes** :
- Ne pas se contenter d'un cercle uni : ajouter des reflets, des anneaux ou des motifs
- Les étoiles peuvent avoir un halo (dessiner derrière des versions plus grandes et semi-transparentes)
- Combiner plusieurs formes (étoiles + étincelles, cercles + anneaux)

**Soigner les couleurs** :
- Utiliser des couleurs vives et complémentaires
- Ajouter du contraste (contours sombres sur formes claires, contours clairs sur formes sombres)
- Penser à la composition d'ensemble

**Pour les formes complexes** (cœurs, flocons, etc.) :
- Combiner polygones et ellipses
- Calculer soigneusement les points pour la symétrie
- Ajouter des détails (un cœur peut avoir une courbe de reflet, les flocons ont des branches ouvragées)

Faire preuve de créativité et de minutie ! Un bon GIF Slack doit paraître abouti, pas ressembler à un graphisme provisoire.

## Utilitaires disponibles

### GIFBuilder (`core.gif_builder`)
Assemble les images et optimise pour Slack :
```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)  # Ajouter une image PIL
builder.add_frames(frames)  # Ajouter une liste d'images
builder.save('out.gif', num_colors=48, optimize_for_emoji=True, remove_duplicates=True)
```

### Validateurs (`core.validators`)
Vérifient qu'un GIF respecte les exigences de Slack :
```python
from core.validators import validate_gif, is_slack_ready

# Validation détaillée
passes, info = validate_gif('my.gif', is_emoji=True, verbose=True)

# Vérification rapide
if is_slack_ready('my.gif'):
    print("Prêt !")
```

### Fonctions d'accélération (`core.easing`)
Un mouvement fluide plutôt que linéaire :
```python
from core.easing import interpolate

# Progression de 0.0 à 1.0
t = i / (num_frames - 1)

# Appliquer l'accélération
y = interpolate(start=0, end=400, t=t, easing='ease_out')

# Disponibles : linear, ease_in, ease_out, ease_in_out,
#               bounce_out, elastic_out, back_out
```

### Aides pour les images (`core.frame_composer`)
Fonctions pratiques pour les besoins courants :
```python
from core.frame_composer import (
    create_blank_frame,         # Fond de couleur unie
    create_gradient_background,  # Dégradé vertical
    draw_circle,                # Aide pour les cercles
    draw_text,                  # Rendu de texte simple
    draw_star                   # Étoile à 5 branches
)
```

## Concepts d'animation

### Secousse / vibration
Décaler la position de l'objet par oscillation :
- Utiliser `math.sin()` ou `math.cos()` avec l'indice de l'image
- Ajouter de petites variations aléatoires pour un rendu naturel
- Appliquer à la position x et/ou y

### Pulsation / battement de cœur
Faire varier la taille de l'objet en rythme :
- Utiliser `math.sin(t * frequency * 2 * math.pi)` pour une pulsation fluide
- Pour un battement de cœur : deux pulsations rapides puis une pause (ajuster la sinusoïde)
- Faire varier entre 0,8 et 1,2 fois la taille de base

### Rebond
L'objet tombe et rebondit :
- Utiliser `interpolate()` avec `easing='bounce_out'` pour l'atterrissage
- Utiliser `easing='ease_in'` pour la chute (accélération)
- Simuler la gravité en augmentant la vitesse verticale à chaque image

### Rotation
Faire tourner l'objet autour de son centre :
- PIL : `image.rotate(angle, resample=Image.BICUBIC)`
- Pour un dandinement : utiliser une sinusoïde pour l'angle plutôt qu'une progression linéaire

### Fondu d'entrée / de sortie
Apparaître ou disparaître progressivement :
- Créer une image RGBA et ajuster le canal alpha
- Ou utiliser `Image.blend(image1, image2, alpha)`
- Fondu d'entrée : alpha de 0 à 1
- Fondu de sortie : alpha de 1 à 0

### Glissement
Amener l'objet de l'extérieur du cadre à sa position :
- Position de départ : hors des limites de l'image
- Position d'arrivée : emplacement visé
- Utiliser `interpolate()` avec `easing='ease_out'` pour un arrêt en douceur
- Pour un léger dépassement : utiliser `easing='back_out'`

### Zoom
Mettre à l'échelle et positionner pour un effet de zoom :
- Zoom avant : échelle de 0,1 à 2,0, recadrage au centre
- Zoom arrière : échelle de 2,0 à 1,0
- On peut ajouter un flou de mouvement pour l'effet dramatique (filtre PIL)

### Explosion / gerbe de particules
Créer des particules qui rayonnent vers l'extérieur :
- Générer des particules avec des angles et des vitesses aléatoires
- Mettre à jour chaque particule : `x += vx`, `y += vy`
- Ajouter la gravité : `vy += gravity_constant`
- Estomper les particules avec le temps (réduire l'alpha)

## Stratégies d'optimisation

Seulement quand on demande de réduire la taille du fichier, appliquer quelques-unes des méthodes suivantes :

1. **Moins d'images** — moins d'images par seconde (10 au lieu de 20) ou durée plus courte
2. **Moins de couleurs** — `num_colors=48` au lieu de 128
3. **Dimensions plus petites** — 128 × 128 au lieu de 480 × 480
4. **Supprimer les doublons** — `remove_duplicates=True` dans `save()`
5. **Mode émoji** — `optimize_for_emoji=True` optimise automatiquement

```python
# Optimisation maximale pour un émoji
builder.save(
    'emoji.gif',
    num_colors=48,
    optimize_for_emoji=True,
    remove_duplicates=True
)
```

## Philosophie

Cette skill fournit :
- **Des connaissances** : les exigences de Slack et des concepts d'animation
- **Des utilitaires** : GIFBuilder, validateurs, fonctions d'accélération
- **De la souplesse** : on construit la logique d'animation avec les primitives de PIL

Elle NE fournit PAS :
- de gabarits d'animation rigides ni de fonctions toutes faites
- de rendu de polices d'émojis (peu fiable d'une plateforme à l'autre)
- de bibliothèque de graphismes intégrée à la skill

**À propos des images envoyées** : cette skill ne contient pas de graphismes tout faits, mais si l'utilisateur envoie une image, la charger et la manipuler avec PIL — déterminer d'après sa demande s'il veut l'utiliser directement ou seulement s'en inspirer.

Faire preuve de créativité ! Combiner les concepts (rebond + rotation, pulsation + glissement, etc.) et exploiter toutes les possibilités de PIL.

## Dépendances

```bash
pip install pillow imageio numpy
```
