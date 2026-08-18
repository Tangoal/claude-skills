# claude-skills

Une sélection de **Skills** que j'ai écrites pour [Claude Code](https://claude.com/product/claude-code),
l'agent CLI d'Anthropic. Un skill est un fichier `SKILL.md` (instructions +
workflow) que l'agent charge à la demande pour appliquer une méthode précise
plutôt que d'improviser — ici, pour éviter des dérives fréquentes des LLM sur
des tâches concrètes (UI générique par défaut, objectifs flous, doc de projet
qui dérive du code réel).

## Skills

| Skill | Ce qu'il fait |
|---|---|
| [`antislop`](./antislop) | Empêche de livrer une UI générique ("IA slop") sur une page/écran/composant précis : force à cadrer le vrai job de l'écran, à s'appuyer sur des références réelles et le design system existant, à rejeter les motifs par défaut (dashboard générique, cartes bento, fausses métriques, dégradés décoratifs), et fait passer un "finish gate" avant de considérer le travail terminé. |
| [`define-goal`](./define-goal) | Cadre un objectif concret et vérifiable dans un `GOAL.md` avant de lancer du travail — prêt à être passé à la commande native `/goal` de Claude Code, qui évalue automatiquement une condition d'arrêt après chaque tour. |
| [`project`](./project) | Point d'entrée unique pour la doc de cycle de vie d'un projet (`BRIEF.md`, `ROADMAP.md`, `AGENTS.md`) et pour un wiki global qui les agrège. Le fil conducteur : garder la doc alignée avec le code et les décisions réelles, jamais de la documentation pour elle-même. |

## Pourquoi ça existe

En usage quotidien avec Claude Code, je retombais sur les mêmes frictions :
UI générique par défaut, tâches lancées sans critère d'arrêt clair, doc de
projet qui dérive silencieusement du code. Chaque skill encode une discipline
pour une de ces frictions, de façon à ce que l'agent l'applique
systématiquement plutôt que de compter sur un rappel manuel à chaque session.

## Utilisation

Copier le dossier d'un skill dans `~/.claude/skills/<nom>/` (skills perso) ou
`.claude/skills/<nom>/` à la racine d'un projet (skills scopés au repo), puis
l'invoquer avec `/<nom>` dans Claude Code — voir la
[doc officielle des Agent Skills](https://docs.claude.com/en/docs/claude-code/skills).
