---
name: web-artifacts-builder
description: Ensemble d'outils pour créer des artefacts HTML claude.ai élaborés, à plusieurs composants, avec des technologies front-end modernes (React, Tailwind CSS, shadcn/ui). À utiliser pour les artefacts complexes qui demandent une gestion d'état, du routage ou des composants shadcn/ui — pas pour les artefacts HTML/JSX simples en un seul fichier.
license: Conditions complètes dans LICENSE.txt
---

# Constructeur d'artefacts web

Pour construire des artefacts front-end claude.ai ambitieux, suivre ces étapes :
1. Initialiser le dépôt front-end avec `scripts/init-artifact.sh`
2. Développer l'artefact en modifiant le code généré
3. Regrouper tout le code dans un seul fichier HTML avec `scripts/bundle-artifact.sh`
4. Afficher l'artefact à l'utilisateur
5. (Facultatif) Tester l'artefact

**Pile technique** : React 18 + TypeScript + Vite + Parcel (regroupement) + Tailwind CSS + shadcn/ui

## Consignes de design et de style

TRÈS IMPORTANT : pour éviter ce qu'on appelle souvent l'« AI slop » (le rendu standardisé et sans âme typique des contenus générés par IA), éviter l'abus de mises en page centrées, les dégradés violets, les coins arrondis uniformes et la police Inter.

## Démarrage rapide

### Étape 1 : initialiser le projet

Lancer le script d'initialisation pour créer un nouveau projet React :
```bash
bash scripts/init-artifact.sh <nom-du-projet>
cd <nom-du-projet>
```

On obtient un projet entièrement configuré avec :
- ✅ React + TypeScript (via Vite)
- ✅ Tailwind CSS 3.4.1 avec le système de thèmes de shadcn/ui
- ✅ Alias de chemins (`@/`) configurés
- ✅ Plus de 40 composants shadcn/ui préinstallés
- ✅ Toutes les dépendances Radix UI incluses
- ✅ Parcel configuré pour le regroupement (via `.parcelrc`)
- ✅ Compatibilité Node 18+ (détection automatique et version de Vite figée en conséquence)

### Étape 2 : développer l'artefact

Pour construire l'artefact, modifier les fichiers générés. Voir les **tâches de développement courantes** ci-dessous.

### Étape 3 : regrouper en un seul fichier HTML

Pour regrouper l'application React en un seul artefact HTML :
```bash
bash scripts/bundle-artifact.sh
```

On obtient `bundle.html`, un artefact autonome où JavaScript, CSS et dépendances sont tous intégrés. Ce fichier peut être partagé directement comme artefact dans une conversation Claude.

**Prérequis** : le projet doit avoir un `index.html` à la racine.

**Ce que fait le script** :
- Installe les dépendances de regroupement (parcel, @parcel/config-default, parcel-resolver-tspaths, html-inline)
- Crée la configuration `.parcelrc` avec prise en charge des alias de chemins
- Construit avec Parcel (sans source maps)
- Intègre toutes les ressources dans un seul HTML avec html-inline

### Étape 4 : partager l'artefact avec l'utilisateur

Enfin, partager le fichier HTML regroupé dans la conversation, pour que l'utilisateur puisse l'afficher comme artefact.

### Étape 5 : tester ou visualiser l'artefact (facultatif)

Remarque : cette étape est entièrement facultative. Ne la faire que si c'est nécessaire ou demandé.

Pour tester ou visualiser l'artefact, utiliser les outils disponibles (y compris d'autres skills ou des outils intégrés comme Playwright ou Puppeteer). De manière générale, éviter de tester l'artefact d'emblée : cela allonge le délai entre la demande et le moment où l'artefact terminé est visible. Tester plus tard, après l'avoir présenté, si c'est demandé ou si des problèmes apparaissent.

## Référence

- **Composants shadcn/ui** : https://ui.shadcn.com/docs/components
