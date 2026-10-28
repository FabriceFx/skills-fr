---
name: webapp-testing
description: Boîte à outils pour piloter et tester des applications web locales avec Playwright. Permet de vérifier le fonctionnement du front-end, de déboguer le comportement de l'interface, de prendre des captures d'écran du navigateur et de consulter ses journaux.
license: Conditions complètes dans LICENSE.txt
---

# Tests d'applications web

Pour tester des applications web locales, écrire des scripts Playwright natifs en Python.

**Scripts d'aide disponibles** :
- `scripts/with_server.py` — gère le cycle de vie des serveurs (plusieurs serveurs possibles)

**Toujours lancer d'abord les scripts avec `--help`** pour voir leur mode d'emploi. NE PAS lire le code source tant qu'on n'a pas essayé de lancer le script et constaté qu'une solution sur mesure est absolument nécessaire. Ces scripts peuvent être très longs et encombrer la fenêtre de contexte. Ils sont faits pour être appelés directement comme des boîtes noires, pas pour être chargés dans le contexte.

## Arbre de décision : choisir son approche

```
Tâche de l'utilisateur → Est-ce du HTML statique ?
    ├─ Oui → Lire directement le fichier HTML pour repérer les sélecteurs
    │         ├─ Réussite → Écrire le script Playwright avec ces sélecteurs
    │         └─ Échec / incomplet → Traiter comme dynamique (ci-dessous)
    │
    └─ Non (application dynamique) → Le serveur tourne-t-il déjà ?
        ├─ Non → Lancer : python scripts/with_server.py --help
        │        Puis utiliser l'aide + écrire un script Playwright simplifié
        │
        └─ Oui → Reconnaissance puis action :
            1. Naviguer et attendre networkidle
            2. Prendre une capture d'écran ou inspecter le DOM
            3. Repérer les sélecteurs à partir du rendu
            4. Exécuter les actions avec les sélecteurs trouvés
```

## Exemple : utiliser with_server.py

Pour démarrer un serveur, lancer d'abord `--help`, puis utiliser l'aide :

**Un seul serveur :**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**Plusieurs serveurs (par exemple back-end + front-end) :**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

Le script d'automatisation ne contient que la logique Playwright (les serveurs sont gérés automatiquement) :
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True) # Toujours lancer chromium en mode headless
    page = browser.new_page()
    page.goto('http://localhost:5173') # Serveur déjà lancé et prêt
    page.wait_for_load_state('networkidle') # ESSENTIEL : attendre l'exécution du JS
    # ... logique d'automatisation
    browser.close()
```

## Motif « reconnaissance puis action »

1. **Inspecter le DOM rendu** :
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. **Repérer les sélecteurs** dans les résultats de l'inspection

3. **Exécuter les actions** avec les sélecteurs trouvés

## Piège courant

❌ **Ne pas** inspecter le DOM d'une application dynamique avant d'avoir attendu `networkidle`
✅ **Attendre** `page.wait_for_load_state('networkidle')` avant toute inspection

## Bonnes pratiques

- **Utiliser les scripts fournis comme des boîtes noires** — pour une tâche donnée, vérifier si l'un des scripts de `scripts/` peut aider. Ils gèrent de façon fiable des enchaînements courants et complexes sans encombrer la fenêtre de contexte. Consulter `--help`, puis les appeler directement.
- Utiliser `sync_playwright()` pour les scripts synchrones
- Toujours fermer le navigateur à la fin
- Utiliser des sélecteurs parlants : `text=`, `role=`, sélecteurs CSS ou identifiants
- Ajouter les attentes appropriées : `page.wait_for_selector()` ou `page.wait_for_timeout()`

## Fichiers de référence

- **examples/** — exemples des motifs courants :
  - `element_discovery.py` — découvrir les boutons, liens et champs d'une page
  - `static_html_automation.py` — utiliser des URL `file://` pour du HTML local
  - `console_logging.py` — capturer les journaux de la console pendant l'automatisation
