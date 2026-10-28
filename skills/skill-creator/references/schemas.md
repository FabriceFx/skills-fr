# Schémas JSON

Ce document définit les schémas JSON utilisés par skill-creator. Les noms de champs et les valeurs énumérées restent en anglais : les scripts et la visionneuse en dépendent ; seules les valeurs d'exemple sont traduites.

---

## evals.json

Définit les évaluations d'une skill. Se trouve dans `evals/evals.json`, dans le répertoire de la skill.

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "Requête d'exemple de l'utilisateur",
      "expected_output": "Description du résultat attendu",
      "files": ["evals/files/sample1.pdf"],
      "expectations": [
        "La sortie contient X",
        "La skill a utilisé le script Y"
      ]
    }
  ]
}
```

**Champs :**
- `skill_name` : nom correspondant à l'en-tête de la skill
- `evals[].id` : identifiant entier unique
- `evals[].prompt` : la tâche à exécuter
- `evals[].expected_output` : description lisible de la réussite
- `evals[].files` : liste facultative de chemins de fichiers d'entrée (relatifs à la racine de la skill)
- `evals[].expectations` : liste d'énoncés vérifiables

---

## history.json

Suit la progression des versions en mode amélioration. Se trouve à la racine de l'espace de travail.

```json
{
  "started_at": "2026-01-15T10:30:00Z",
  "skill_name": "pdf",
  "current_best": "v2",
  "iterations": [
    {
      "version": "v0",
      "parent": null,
      "expectation_pass_rate": 0.65,
      "grading_result": "baseline",
      "is_current_best": false
    },
    {
      "version": "v1",
      "parent": "v0",
      "expectation_pass_rate": 0.75,
      "grading_result": "won",
      "is_current_best": false
    },
    {
      "version": "v2",
      "parent": "v1",
      "expectation_pass_rate": 0.85,
      "grading_result": "won",
      "is_current_best": true
    }
  ]
}
```

**Champs :**
- `started_at` : horodatage ISO du début de l'amélioration
- `skill_name` : nom de la skill améliorée
- `current_best` : identifiant de la version la plus performante
- `iterations[].version` : identifiant de version (v0, v1, ...)
- `iterations[].parent` : version parente dont celle-ci est dérivée
- `iterations[].expectation_pass_rate` : taux de réussite issu de la notation
- `iterations[].grading_result` : "baseline" (référence), "won" (gagné), "lost" (perdu) ou "tie" (égalité)
- `iterations[].is_current_best` : s'agit-il de la meilleure version actuelle

---

## grading.json

Sortie de l'agent correcteur. Se trouve dans `<répertoire-d-exécution>/grading.json`.

```json
{
  "expectations": [
    {
      "text": "La sortie contient le nom « John Smith »",
      "passed": true,
      "evidence": "Trouvé à l'étape 3 de la transcription : « Noms extraits : John Smith, Sarah Johnson »"
    },
    {
      "text": "La feuille de calcul contient une formule SOMME en cellule B10",
      "passed": false,
      "evidence": "Aucune feuille de calcul n'a été créée. La sortie était un fichier texte."
    }
  ],
  "summary": {
    "passed": 2,
    "failed": 1,
    "total": 3,
    "pass_rate": 0.67
  },
  "execution_metrics": {
    "tool_calls": {
      "Read": 5,
      "Write": 2,
      "Bash": 8
    },
    "total_tool_calls": 15,
    "total_steps": 6,
    "errors_encountered": 0,
    "output_chars": 12450,
    "transcript_chars": 3200
  },
  "timing": {
    "executor_duration_seconds": 165.0,
    "grader_duration_seconds": 26.0,
    "total_duration_seconds": 191.0
  },
  "claims": [
    {
      "claim": "Le formulaire comporte 12 champs remplissables",
      "type": "factual",
      "verified": true,
      "evidence": "12 champs comptés dans field_info.json"
    }
  ],
  "user_notes_summary": {
    "uncertainties": ["Données 2023 utilisées, peut-être périmées"],
    "needs_review": [],
    "workarounds": ["Repli sur une surimpression de texte pour les champs non remplissables"]
  },
  "eval_feedback": {
    "suggestions": [
      {
        "assertion": "La sortie contient le nom « John Smith »",
        "reason": "Un document halluciné qui mentionne ce nom réussirait aussi"
      }
    ],
    "overall": "Les assertions vérifient la présence mais pas l'exactitude."
  }
}
```

**Champs :**
- `expectations[]` : attentes notées, avec preuves
- `summary` : décomptes agrégés de réussites et d'échecs
- `execution_metrics` : usage des outils et taille de la sortie (tirés du metrics.json de l'exécutant)
- `timing` : durées réelles (tirées de timing.json)
- `claims` : affirmations extraites de la sortie et vérifiées
- `user_notes_summary` : problèmes signalés par l'exécutant
- `eval_feedback` : (facultatif) suggestions d'amélioration des évaluations, présent seulement quand le correcteur relève des problèmes qui valent d'être signalés

---

## metrics.json

Sortie de l'agent exécutant. Se trouve dans `<répertoire-d-exécution>/outputs/metrics.json`.

```json
{
  "tool_calls": {
    "Read": 5,
    "Write": 2,
    "Bash": 8,
    "Edit": 1,
    "Glob": 2,
    "Grep": 0
  },
  "total_tool_calls": 18,
  "total_steps": 6,
  "files_created": ["filled_form.pdf", "field_values.json"],
  "errors_encountered": 0,
  "output_chars": 12450,
  "transcript_chars": 3200
}
```

**Champs :**
- `tool_calls` : nombre d'appels par type d'outil
- `total_tool_calls` : somme de tous les appels d'outils
- `total_steps` : nombre de grandes étapes d'exécution
- `files_created` : liste des fichiers de sortie créés
- `errors_encountered` : nombre d'erreurs pendant l'exécution
- `output_chars` : nombre total de caractères des fichiers de sortie
- `transcript_chars` : nombre de caractères de la transcription

---

## timing.json

Durées réelles d'une exécution. Se trouve dans `<répertoire-d-exécution>/timing.json`.

**Comment les relever :** à la fin d'une tâche de sous-agent, la notification de tâche contient `total_tokens` et `duration_ms`. Les enregistrer immédiatement — elles ne sont conservées nulle part ailleurs et ne peuvent pas être récupérées après coup.

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3,
  "executor_start": "2026-01-15T10:30:00Z",
  "executor_end": "2026-01-15T10:32:45Z",
  "executor_duration_seconds": 165.0,
  "grader_start": "2026-01-15T10:32:46Z",
  "grader_end": "2026-01-15T10:33:12Z",
  "grader_duration_seconds": 26.0
}
```

---

## benchmark.json

Sortie du mode benchmark. Se trouve dans `benchmarks/<horodatage>/benchmark.json`.

```json
{
  "metadata": {
    "skill_name": "pdf",
    "skill_path": "/path/to/pdf",
    "executor_model": "claude-sonnet-4-20250514",
    "analyzer_model": "most-capable-model",
    "timestamp": "2026-01-15T10:30:00Z",
    "evals_run": [1, 2, 3],
    "runs_per_configuration": 3
  },

  "runs": [
    {
      "eval_id": 1,
      "eval_name": "Ocean",
      "configuration": "with_skill",
      "run_number": 1,
      "result": {
        "pass_rate": 0.85,
        "passed": 6,
        "failed": 1,
        "total": 7,
        "time_seconds": 42.5,
        "tokens": 3800,
        "tool_calls": 18,
        "errors": 0
      },
      "expectations": [
        {"text": "...", "passed": true, "evidence": "..."}
      ],
      "notes": [
        "Données 2023 utilisées, peut-être périmées",
        "Repli sur une surimpression de texte pour les champs non remplissables"
      ]
    }
  ],

  "run_summary": {
    "with_skill": {
      "pass_rate": {"mean": 0.85, "stddev": 0.05, "min": 0.80, "max": 0.90},
      "time_seconds": {"mean": 45.0, "stddev": 12.0, "min": 32.0, "max": 58.0},
      "tokens": {"mean": 3800, "stddev": 400, "min": 3200, "max": 4100}
    },
    "without_skill": {
      "pass_rate": {"mean": 0.35, "stddev": 0.08, "min": 0.28, "max": 0.45},
      "time_seconds": {"mean": 32.0, "stddev": 8.0, "min": 24.0, "max": 42.0},
      "tokens": {"mean": 2100, "stddev": 300, "min": 1800, "max": 2500}
    },
    "delta": {
      "pass_rate": "+0.50",
      "time_seconds": "+13.0",
      "tokens": "+1700"
    }
  },

  "notes": [
    "L'assertion « La sortie est un fichier PDF » réussit à 100 % dans les deux configurations — elle ne fait peut-être pas ressortir la valeur de la skill",
    "L'évaluation 3 montre une forte variance (50 % ± 40 %) — peut-être instable ou dépendante du modèle",
    "Les exécutions sans skill échouent systématiquement aux attentes d'extraction de tableaux",
    "La skill ajoute en moyenne 13 s d'exécution mais améliore le taux de réussite de 50 %"
  ]
}
```

**Champs :**
- `metadata` : informations sur l'exécution du benchmark
  - `skill_name` : nom de la skill
  - `timestamp` : date et heure du benchmark
  - `evals_run` : liste des noms ou identifiants d'évaluations
  - `runs_per_configuration` : nombre d'exécutions par configuration (par ex. 3)
- `runs[]` : résultats de chaque exécution
  - `eval_id` : identifiant numérique de l'évaluation
  - `eval_name` : nom lisible de l'évaluation (utilisé comme titre de section dans la visionneuse)
  - `configuration` : doit valoir `"with_skill"` ou `"without_skill"` (la visionneuse utilise cette chaîne exacte pour regrouper et colorer)
  - `run_number` : numéro entier de l'exécution (1, 2, 3...)
  - `result` : objet imbriqué avec `pass_rate`, `passed`, `total`, `time_seconds`, `tokens`, `errors`
- `run_summary` : agrégats statistiques par configuration
  - `with_skill` / `without_skill` : chacun contient des objets `pass_rate`, `time_seconds`, `tokens` avec des champs `mean` (moyenne) et `stddev` (écart-type)
  - `delta` : chaînes d'écart comme `"+0.50"`, `"+13.0"`, `"+1700"`
- `notes` : observations libres de l'analyste

**Important :** la visionneuse lit ces noms de champs exactement. Écrire `config` au lieu de `configuration`, ou placer `pass_rate` au premier niveau d'une exécution au lieu de l'imbriquer sous `result`, fera afficher des valeurs vides ou nulles. Toujours se référer à ce schéma pour générer benchmark.json à la main.

---

## comparison.json

Sortie du comparateur à l'aveugle. Se trouve dans `<répertoire-de-notation>/comparison-N.json`.

```json
{
  "winner": "A",
  "reasoning": "La sortie A apporte une solution complète, correctement mise en forme, avec tous les champs requis. Il manque à la sortie B le champ de date, et sa mise en forme présente des incohérences.",
  "rubric": {
    "A": {
      "content": {
        "correctness": 5,
        "completeness": 5,
        "accuracy": 4
      },
      "structure": {
        "organization": 4,
        "formatting": 5,
        "usability": 4
      },
      "content_score": 4.7,
      "structure_score": 4.3,
      "overall_score": 9.0
    },
    "B": {
      "content": {
        "correctness": 3,
        "completeness": 2,
        "accuracy": 3
      },
      "structure": {
        "organization": 3,
        "formatting": 2,
        "usability": 3
      },
      "content_score": 2.7,
      "structure_score": 2.7,
      "overall_score": 5.4
    }
  },
  "output_quality": {
    "A": {
      "score": 9,
      "strengths": ["Solution complète", "Bien mise en forme", "Tous les champs présents"],
      "weaknesses": ["Légère incohérence de style dans l'en-tête"]
    },
    "B": {
      "score": 5,
      "strengths": ["Sortie lisible", "Structure de base correcte"],
      "weaknesses": ["Champ de date manquant", "Incohérences de mise en forme", "Extraction des données partielle"]
    }
  },
  "expectation_results": {
    "A": {
      "passed": 4,
      "total": 5,
      "pass_rate": 0.80,
      "details": [
        {"text": "La sortie contient le nom", "passed": true}
      ]
    },
    "B": {
      "passed": 3,
      "total": 5,
      "pass_rate": 0.60,
      "details": [
        {"text": "La sortie contient le nom", "passed": true}
      ]
    }
  }
}
```

---

## analysis.json

Sortie de l'agent d'analyse a posteriori. Se trouve dans `<répertoire-de-notation>/analysis.json`.

```json
{
  "comparison_summary": {
    "winner": "A",
    "winner_skill": "chemin/de/la/skill/gagnante",
    "loser_skill": "chemin/de/la/skill/perdante",
    "comparator_reasoning": "Bref résumé des raisons du choix du comparateur"
  },
  "winner_strengths": [
    "Instructions claires, étape par étape, pour traiter les documents de plusieurs pages",
    "Script de validation inclus, qui a détecté des erreurs de mise en forme"
  ],
  "loser_weaknesses": [
    "L'instruction vague « traiter le document de façon appropriée » a mené à un comportement incohérent",
    "Pas de script de validation : l'agent a dû improviser"
  ],
  "instruction_following": {
    "winner": {
      "score": 9,
      "issues": ["Mineur : étape facultative de journalisation sautée"]
    },
    "loser": {
      "score": 6,
      "issues": [
        "N'a pas utilisé le gabarit de mise en forme de la skill",
        "A inventé sa propre approche au lieu de suivre l'étape 3"
      ]
    }
  },
  "improvement_suggestions": [
    {
      "priority": "high",
      "category": "instructions",
      "suggestion": "Remplacer « traiter le document de façon appropriée » par des étapes explicites",
      "expected_impact": "Supprimerait l'ambiguïté à l'origine du comportement incohérent"
    }
  ],
  "transcript_insights": {
    "winner_execution_pattern": "Lecture de la skill -> Démarche en 5 étapes suivie -> Script de validation utilisé",
    "loser_execution_pattern": "Lecture de la skill -> Approche floue -> 3 méthodes différentes essayées"
  }
}
```
