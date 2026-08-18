---
name: project
description: Gestion du cycle de vie et du suivi des tâches des projets (wiki global + audit doc automatique par projet)
---

# /project — Cycle de vie et suivi des projets

Un seul point d'entrée pour tout ce qui touche à la doc d'un projet
(`docs/BRIEF.md`, `docs/ROADMAP.md`, `docs/DESIGN.md` si UI, `AGENTS.md` +
stub `CLAUDE.md` à la racine du projet — voir
`references/project-docs-format.md`), et au wiki global qui les agrège
(le dossier `docs/` à la racine du wiki). `docs/GOAL.md` peut aussi exister dans un projet mais
n'est ni produit ni géré par `/project` — voir plus bas.

Trois familles d'actions :
- **Toujours globale** (`sync`) — parcourt tous les projets, pas de scope possible.
- **Globale par défaut, restreignable** (`status`, `audit`) — parcourent
  tout le périmètre par défaut ; un nom de projet en argument optionnel
  restreint à un seul.
- **Toujours scopée à un projet** (`new`) — prend un nom de projet en
  argument obligatoire, ne touche que ses fichiers doc.

## Emplacements

- Wiki global : le dossier `docs/` à la racine du workspace (là où vit ce
  wiki — généralement le répertoire depuis lequel le skill est invoqué, ou
  un dossier `docs/` remonté depuis le cwd).
- Projets : tout dossier contenant un `docs/BRIEF.md` ou `docs/ROADMAP.md`,
  détecté par scan récursif depuis la racine du workspace — pas de
  convention de sous-dossiers imposée. Un projet se déclare simplement en
  existant avec cette structure.
- Un dossier sans aucun `docs/BRIEF.md`/`docs/ROADMAP.md` n'est jamais
  traité comme un projet, même s'il contient du code — évite de scanner du
  tooling interne (scripts, infra selfhosted) qui n'a pas vocation à avoir
  cette doc.

## Objectif implicite

Quel que soit l'action invoquée, le fil conducteur du skill est de garder
chaque projet dans un état où sa doc (`BRIEF.md`/`ROADMAP.md`/`AGENTS.md`)
reflète fidèlement le code et les décisions réelles — pas de produire de la
documentation pour elle-même. Un `audit`, ou une mise à jour de
`ROADMAP.md` doivent réduire l'écart entre ce que dit la doc et ce qui
existe vraiment, même quand ils portent sur un point précis plutôt que sur
tout le projet.

## Actions disponibles

### `/project` ou `/project status [nom]`

Sans argument : affiche l'état de la documentation de tous les projets détectés (voir `## Emplacements`) — quels fichiers (`BRIEF.md`, `ROADMAP.md`, `AGENTS.md`, `DESIGN.md` si applicable) sont présents ou manquants, résumé des tâches en cours/bloquées de chaque roadmap. Un `DESIGN.md` manquant n'est signalé comme manquant que pour un projet identifié comme UI-facing, pas systématiquement. `GOAL.md` s'affiche s'il est présent (avec son statut) mais **n'est jamais listé comme manquant** — il est ponctuel/optionnel par nature, produit par le skill `define-goal` (pas par `/project`), pas un fichier attendu par défaut comme les trois autres.

Avec un nom de projet : même résumé mais restreint à ce seul projet (fichiers présents/manquants, état de sa roadmap).

### `/project sync`

Synchronise la doc globale (action strictement globale, pas de scope projet).
Pour éviter de relire tous les `ROADMAP.md`/`BRIEF.md` de chaque projet à
chaque passage (surtout via le timer hebdomadaire), s'appuie sur les dates
de modification filesystem plutôt que sur une relecture systématique :

1. **Repérage rapide (pas de lecture LLM)** : `stat -c '%Y %n'` (ou
   équivalent) sur tous les `docs/ROADMAP.md` et `docs/BRIEF.md` des
   projets, comparé à l'état enregistré dans
   `docs/.sync-state.json` (à la racine du wiki) (fichier interne, pas un doc
   utilisateur — `{ "<chemin projet>": { "roadmap_mtime": N, "brief_mtime": N } }`).
   Un projet absent de ce fichier (nouveau, ou état pas encore initialisé)
   est traité comme modifié.
2. **Lecture ciblée** : ne lire (`Read`) que les `ROADMAP.md`/`BRIEF.md`
   dont le mtime dépasse la valeur enregistrée — pas les autres. Si aucun
   fichier n'a changé depuis le dernier sync, le dire et s'arrêter là (pas
   de réécriture des docs globaux).
3. Met à jour `docs/ROADMAP.md` avec l'état actuel des projets modifiés,
   `docs/BRIEF.md` si des projets ont été ajoutés ou ont changé de statut.
4. Réécrit `.sync-state.json` avec les mtimes actuels de tous les projets
   (y compris ceux non modifiés, déjà à jour) une fois le sync terminé.

Un projet disparu du filesystem (dossier supprimé) mais encore présent
dans `.sync-state.json` est signalé et retiré du fichier d'état — la
détection de "projet fantôme" dans le wiki global reste portée par
`/project audit`, pas par `sync`.

### `/project new <nom>`

Crée la structure docs pour un nouveau projet :
- Demande : nom, emplacement (chemin du dossier), vision, stack, et si le projet a une UI (pour décider de créer `DESIGN.md`)
- Crée `docs/BRIEF.md`, `docs/ROADMAP.md`, `AGENTS.md` + stub `CLAUDE.md` (racine du projet), et `docs/DESIGN.md` si UI-facing — gabarits dans `references/project-docs-format.md`, variante early-stage si le concept n'est pas encore cadré
- Met à jour `docs/BRIEF.md` et `docs/ROADMAP.md` globaux

### `/project audit [nom]`

Sans argument : vérifie la cohérence du wiki sur tous les projets — fichiers manquants dans les projets existants, tâches Done dans les roadmaps projets mais pas reflétées dans la roadmap globale, projets mentionnés dans le global mais sans dossier sur le filesystem, **et fichiers `ROADMAP.md` qui ne suivent pas le schéma canonique** (voir `references/project-docs-format.md`) — bon vocabulaire de statuts, colonnes `Description`/`Dépendances` manquantes.

Pour chaque fichier non conforme détecté, propose la normalisation (traduire les statuts, ajouter les colonnes manquantes vides, sans jamais supprimer ou réinventer le contenu existant des colonnes déjà là) et applique-la seulement après validation.

Si un projet est identifié comme UI-facing mais que son `DESIGN.md` est absent, le signaler explicitement dans le rapport — même logique de détection que pour `/project status`.

**Ne normalise pas aveuglément un fichier qui n'a manifestement pas cette forme de table** (ex: un `ROADMAP.md` en sections de titres au lieu d'un tableau plutôt que le format canonique) — signale-le comme hors-schéma plutôt que de forcer une conversion qui perdrait de l'information ou n'aurait pas de sens.

**Cohérence interne (par projet, en plus de la conformité de schéma) :** le schéma peut être respecté à la lettre tout en racontant deux histoires différentes — c'est ça qu'il faut traquer, pas juste des colonnes manquantes.
- `BRIEF.md` — la `Vision` et les autres sections (`Concept à définir`, `Périmètre`, `MVP`) ne se contredisent pas sur un fait vérifiable (nombre d'éléments/entités clés, plateforme cible, mode de monétisation...). Une contradiction ici veut dire que personne n'a relu le fichier après l'avoir fait évoluer par petits ajouts successifs.
- `ROADMAP.md` — une `Dépendances` ne pointe pas vers une tâche dont le statut la contredit (ex: dépend d'une tâche `Cancelled`), pas deux lignes quasi identiques avec des statuts différents (doublon de fait).
- `AGENTS.md` — la stack déclarée en tête de fichier ne contredit pas une décision listée dans une sous-section `### Décisions du {date}` plus bas (ex: stack déclarée "Postgres" mais une décision plus récente dit "on part sur SQLite" sans que la section stack ait été mise à jour).
- Entre fichiers du même projet — un fait cité dans plusieurs docs (nom de la stack, portée du MVP) est identique partout ; sinon c'est un signal qu'un fichier a été mis à jour et pas l'autre.

Signale chaque contradiction trouvée avec les deux passages exacts en conflit et propose une reformulation, mais ne tranche jamais lequel des deux est correct à la place de l'utilisateur — c'est un désaccord de fond, pas une case de schéma à corriger automatiquement.

**Optimisation mtime (mêmes principes que `/project sync`, mais pas la même sémantique) :** les checks structurels (fichiers manquants, `DESIGN.md` absent, projets fantômes dans le global) restent un simple `ls`/`stat` à refaire systématiquement — pas de cache utile là-dessus. Les checks de fond ci-dessus (conformité de schéma, cohérence interne, `GOAL.md`) sont ce qui coûte cher à relire, et s'appuient sur `docs/.audit-state.json` (à la racine du wiki) (`{ "<chemin projet>": { "roadmap_mtime": N, "brief_mtime": N, "agents_mtime": N, "design_mtime": N } }`, présent uniquement pour un projet dont le dernier audit n'a signalé **aucun** problème de fond) :
- Un projet absent de `.audit-state.json`, ou dont un mtime dépasse la valeur enregistrée, est ré-audité en profondeur (relecture + checks de fond).
- Un projet présent avec tous ses mtimes à jour est sauté pour les checks de fond (mais reste soumis aux checks structurels).
- Après l'audit, n'enregistrer/mettre à jour l'entrée d'un projet dans `.audit-state.json` que si les checks de fond n'ont trouvé **aucun** problème dessus. Si un problème est trouvé et que l'utilisateur ne le corrige pas dans la foulée, ne pas écrire d'entrée (ou retirer l'entrée existante) — sinon le problème non résolu disparaîtrait silencieusement du rapport suivant tant que le fichier ne change pas. Si le problème est corrigé sur le moment (fichier réécrit), le mtime aura de toute façon bougé et l'entrée peut être écrite normalement.
- Fichier interne, pas un doc utilisateur, distinct de `.sync-state.json` (cycles de vie différents : `sync` écrit toujours après un passage, `audit` n'écrit que du "propre").

**`GOAL.md`/`GOAL.done.md`, si présent :** lire sa `Condition` et sa section `Statut` (format défini par le skill `define-goal`, pas par `/project` — `GOAL.md` reste hors de la famille BRIEF/ROADMAP/AGENTS/DESIGN, c'est un objectif ponctuel, pas un doc vivant). Si le contenu du projet (BRIEF/ROADMAP/code) indique clairement que la condition est atteinte, supprimer le fichier (`GOAL.md` ou `GOAL.done.md`) et le signaler dans le rapport.

Avec un nom de projet : même audit restreint à ce projet (n'a pas de sens de vérifier les "projets mentionnés dans le global mais absents du filesystem" pour un seul nom — dans ce cas ne vérifie que la cohérence interne de ses fichiers, `GOAL.md` inclus s'il existe).
