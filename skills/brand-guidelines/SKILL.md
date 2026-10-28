---
name: brand-guidelines
description: Applique les couleurs et la typographie officielles de la marque Anthropic à tout artefact qui gagnerait à en adopter l'identité visuelle. À utiliser dès que s'appliquent des couleurs de marque, une charte graphique, une mise en forme visuelle ou des normes de design d'entreprise.
license: Conditions complètes dans LICENSE.txt
---

# Charte graphique Anthropic

## Présentation

Cette skill donne accès aux ressources officielles d'identité visuelle et de style d'Anthropic.

**Mots-clés** : marque, identité d'entreprise, identité visuelle, post-traitement, mise en style, couleurs de marque, typographie, marque Anthropic, mise en forme visuelle, design visuel

## Charte graphique

### Couleurs

**Couleurs principales :**

- Foncé : `#141413` — texte principal et fonds sombres
- Clair : `#faf9f5` — fonds clairs et texte sur fond sombre
- Gris moyen : `#b0aea5` — éléments secondaires
- Gris clair : `#e8e6dc` — fonds discrets

**Couleurs d'accent :**

- Orange : `#d97757` — accent principal
- Bleu : `#6a9bcc` — accent secondaire
- Vert : `#788c5d` — accent tertiaire

### Typographie

- **Titres** : Poppins (repli sur Arial)
- **Texte courant** : Lora (repli sur Georgia)
- **Remarque** : pour un meilleur résultat, les polices doivent être préinstallées dans l'environnement

## Fonctionnalités

### Application intelligente des polices

- Applique Poppins aux titres (24 pt et plus)
- Applique Lora au texte courant
- Se replie automatiquement sur Arial / Georgia si les polices personnalisées sont absentes
- Préserve la lisibilité sur tous les systèmes

### Mise en style du texte

- Titres (24 pt et plus) : Poppins
- Texte courant : Lora
- Choix de couleur adapté au fond
- Préserve la hiérarchie et la mise en forme du texte

### Formes et couleurs d'accent

- Les formes non textuelles utilisent les couleurs d'accent
- Alterne entre l'orange, le bleu et le vert
- Garde de l'intérêt visuel sans sortir de la charte

## Détails techniques

### Gestion des polices

- Utilise Poppins et Lora installées sur le système lorsqu'elles sont disponibles
- Repli automatique sur Arial (titres) et Georgia (texte courant)
- Aucune installation de police requise : fonctionne avec les polices système existantes
- Pour un meilleur résultat, préinstaller Poppins et Lora dans l'environnement

### Application des couleurs

- Utilise des valeurs RVB pour une correspondance exacte avec la marque
- Appliquées via la classe `RGBColor` de python-pptx
- Conserve la fidélité des couleurs d'un système à l'autre
