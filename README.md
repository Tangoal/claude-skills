# claude-skills

Une sélection de **Skills** et de **Commands** que j'ai écrites pour
[Claude Code](https://claude.com/product/claude-code), l'agent CLI
d'Anthropic. Deux mécanismes différents :

- un **skill** est un dossier avec un `SKILL.md` (frontmatter `name`/`description`)
  que l'agent charge de lui-même quand le contexte matche, en plus de `/nom` ;
- une **command** est un simple fichier `.md`, invoqué uniquement à la demande
  via `/nom`, jamais automatiquement.

Objectif commun : éviter des dérives fréquentes en usage courant (UI générique
par défaut, objectifs flous, doc de projet qui dérive du code réel, plan
jamais challengé) en encodant une discipline précise plutôt que de compter
sur un rappel manuel à chaque session.

## Skills

| Skill | Ce qu'il fait |
|---|---|
| [`antislop`](./antislop) | Empêche de livrer une UI générique ("IA slop") sur une page/écran/composant précis : force à cadrer le vrai job de l'écran, à s'appuyer sur des références réelles et le design system existant, à rejeter les motifs par défaut (dashboard générique, cartes bento, fausses métriques, dégradés décoratifs), et fait passer un "finish gate" avant de considérer le travail terminé. |
| [`define-goal`](./define-goal) | Cadre un objectif concret et vérifiable dans un `GOAL.md` avant de lancer du travail — prêt à être passé à la commande native `/goal` de Claude Code, qui évalue automatiquement une condition d'arrêt après chaque tour. |
| [`project`](./project) | Point d'entrée unique pour la doc de cycle de vie d'un projet (`BRIEF.md`, `ROADMAP.md`, `AGENTS.md`) et pour un wiki global qui les agrège. Le fil conducteur : garder la doc alignée avec le code et les décisions réelles, jamais de la documentation pour elle-même. |

## Commands

| Command | Ce qu'elle fait |
|---|---|
| [`talk`](./commands/talk.md) | Bascule en mode discussion pure pour le reste de l'échange : aucune action qui modifie l'état (pas d'édition, pas de commit, pas de déploiement), juste explorer et discuter. |
| [`brainstorm`](./commands/brainstorm.md) | Interroge sans relâche sur un plan ou une conception jusqu'à une compréhension partagée, en résolvant chaque branche de l'arbre de décision une par une plutôt que de valider en bloc. |

## Pourquoi ça existe

En usage quotidien avec Claude Code, je retombais sur les mêmes frictions :
UI générique par défaut, tâches lancées sans critère d'arrêt clair, doc de
projet qui dérive silencieusement du code. Chaque skill encode une discipline
pour une de ces frictions, de façon à ce que l'agent l'applique
systématiquement plutôt que de compter sur un rappel manuel à chaque session.

## Utilisation

**Skills** : copier le dossier dans `~/.claude/skills/<nom>/` (perso) ou
`.claude/skills/<nom>/` à la racine d'un projet (scopé au repo) — voir la
[doc officielle des Agent Skills](https://docs.claude.com/en/docs/claude-code/skills).

**Commands** : copier le fichier `.md` dans `~/.claude/commands/<nom>.md`
(perso) ou `.claude/commands/<nom>.md` à la racine d'un projet (scopé au
repo) — voir la
[doc officielle des Slash Commands](https://docs.claude.com/en/docs/claude-code/slash-commands).

Dans les deux cas, s'invoque ensuite avec `/<nom>` dans Claude Code.

## Voir aussi

[`site-audit-skill`](https://github.com/Tangoal/site-audit-skill) — un skill
séparé, pas ici, car il dépend d'outils tiers (testssl.sh, Lighthouse) qu'il
installe à la demande plutôt que d'être un simple fichier d'instructions.
Distribué en plugin (`/plugin marketplace add`) plutôt qu'en copier-coller.
