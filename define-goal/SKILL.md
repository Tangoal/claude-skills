---
name: define-goal
description: Aide à cadrer un objectif concret et vérifiable avant de lancer du travail, et le fige dans un fichier GOAL.md prêt à être passé à la commande native /goal. À utiliser quand l'utilisateur demande de définir un objectif, de créer un GOAL.md, de préparer un /goal, ou de transformer une intention floue en résultat mesurable.
---

# /define-goal — Cadrer un objectif dans GOAL.md

## Rôle du skill

`/goal` est une commande **native** de Claude Code (v2.1.139+) : elle prend
une condition en texte libre, démarre un tour immédiatement, et fait
évaluer la condition par un petit modèle après chaque tour jusqu'à ce
qu'elle soit remplie. Elle ne lit aucun fichier — tout ce qu'elle voit,
c'est la conversation.

Ce skill ne remplace pas `/goal` et ne l'invoque jamais lui-même. Il
produit l'étape d'avant : un `GOAL.md` bien cadré, qui sert de référence
persistante (relisable, versionnable, réutilisable) et dont la section
`Condition` peut être copiée telle quelle dans `/goal <condition>`.

N'utilise ce skill que pour du travail substantiel avec un état de fin
vérifiable. Pour une tâche d'implémentation ordinaire, fais le travail
directement — ne force pas la création d'un GOAL.md.

À ne pas confondre avec une session de cadrage sur les docs d'un projet
(skill `project`) : celle-ci cadre une **idée/vision de projet** encore
floue (résultat continu dans `BRIEF.md`/`ROADMAP.md`), alors que
`/define-goal` cadre un objectif ponctuel déjà identifié, avec un critère
d'arrêt binaire.

## Workflow

### 1. Confirmer le besoin

Utilise ce skill quand l'utilisateur demande `/define-goal`, un GOAL.md,
de cadrer un objectif, ou de préparer un `/goal`. Si la demande est en
fait une tâche d'implémentation classique, fais-la directement.

### 2. Vérifier l'existant

Avant de créer quoi que ce soit, détermine où `GOAL.md` doit vivre : si le
répertoire de travail courant a déjà un dossier `docs/` (convention des
projets du workspace — `docs/BRIEF.md`/`docs/ROADMAP.md`), c'est
`docs/GOAL.md` ; sinon (usage générique du skill, hors convention
workspace) c'est la racine du répertoire de travail courant. Cherche un
`GOAL.md` déjà présent à cet emplacement.

- S'il n'existe pas : passe à l'étape 3.
- S'il existe et correspond toujours à l'intention de l'utilisateur :
  propose de le mettre à jour plutôt que d'en écrire un nouveau à côté.
- S'il existe mais décrit un objectif différent ou déjà atteint : propose à
  l'utilisateur de supprimer ou de mettre à jour le `GOAL.md` existant, en
  lui expliquant en quoi il consiste. S'il préfère le garder tel quel, écris
  le nouvel objectif dans un fichier `GOAL_2.md` à côté (`GOAL_3.md`, etc. si
  plusieurs `GOAL_x.md` existent déjà) — ne jamais écraser silencieusement
  un objectif en cours.

### 3. Restater l'objectif en termes concrets

Un objectif exploitable nomme :
- le résultat précis qui sera vrai à la fin
- l'artefact/système/repo/comportement utilisateur concerné
- comment la réussite sera vérifiée
- ce qui est dans le périmètre
- ce qui est explicitement hors périmètre, si l'ambiguïté peut coûter cher
- la condition d'arrêt qui doit faire revenir vers l'utilisateur plutôt que de continuer à tourner

### 4. Quantifier quand le domaine le permet

Préfère des chiffres qui représentent un vrai succès, pas de la précision
décorative :
- **Validateurs pass/fail** : tests, checks, jobs CI, evals, commandes,
  critères d'acceptation exacts
- **Seuils de qualité** : latence, taux d'erreur, coût, précision, rappel,
  couverture, taux de flake, taille de bundle, mémoire, uptime
- **Contraintes d'artefact** : chemins de fichiers, modules concernés,
  commandes autorisées, formats de sortie, environnements cibles,
  échéance, blast radius maximum
- **Comptages de preuve** : nombre de cas reproduits, de reruns réussis,
  d'exemples relus, d'enregistrements migrés, de commentaires traités, de
  cas vérifiés

### 5. Réparer les objectifs faibles avant de les figer

- Réécris un objectif vague en objectif mesurable quand le contexte local
  rend la réécriture sûre.
- Pose une seule question de clarification courte quand le détail manquant
  changerait le résultat visé ou la façon de le vérifier.
- Rejette les objectifs de pure activité ("avancer", "continuer à
  investiguer", "améliorer les choses", "travailler sur X") tant qu'ils ne
  sont pas précisés en résultat vérifiable.

### 6. Écrire GOAL.md

Utilise le gabarit ci-dessous. Écris-le à l'emplacement déterminé en étape
2 (`docs/GOAL.md` si un dossier `docs/` existe déjà, sinon racine du
répertoire de travail courant), en français ou dans la langue de
l'utilisateur.

```markdown
# Goal — <titre court>

## Condition
<Une phrase autonome, copiable telle quelle dans `/goal <condition>`.
Doit être démontrable par ce que Claude produit dans la conversation
(sortie de commande, résultat de test, contenu de fichier affiché) —
l'évaluateur de /goal ne lit aucun fichier lui-même.>

## Contexte
<2-4 lignes : pourquoi cet objectif, d'où il vient.>

## Vérification
<Commande(s) exacte(s) ou méthode de preuve. Ex: "npm run test:checkout
sort en 0" ou "gh pr view 123 ne montre plus de thread change-request
non résolu".>

## Périmètre
- Dans le périmètre : ...
- Hors périmètre : ...

## Condition d'arrêt
<Ce qui doit faire revenir vers l'utilisateur plutôt que de continuer à
tourner — ex: "si un test hors du périmètre échoue", "après 20 tours
sans progrès", "si une migration de schéma semble nécessaire".>

## Statut
Non démarré — `<date ISO>`
```

Inclus une clause de tours/temps dans la `Condition` seulement si
l'utilisateur en a demandé une explicitement (ex: "ou stop après 20
tours") — `/goal` la respecte nativement si elle est présente dans la
condition passée.

### 7. Rapporter et proposer la suite

Après écriture, montre uniquement :
- le chemin du fichier créé/mis à jour
- la `Condition` telle qu'elle apparaît dans le fichier, prête à copier
- une ligne suggérant `/goal <condition>` (éventuellement associée à
  l'auto mode si le travail doit tourner sans confirmation à chaque outil)

Ne relance jamais `/goal` toi-même — c'est à l'utilisateur de décider du
moment où démarrer la boucle.

## Barre de qualité avant d'écrire la Condition

- Quel résultat concret sera vrai à la fin ?
- Quelle preuve le démontrera, visible dans la conversation ?
- Quel seuil quantitatif ou binaire définit le succès ?
- Quelles limites de périmètre comptent ?
- Qu'est-ce qui doit interrompre la boucle pour demander à l'utilisateur ?

Bon exemple de `Condition` :

> Le p95 de latence de l'API checkout est sous 250 ms sur le slow path documenté, `npm run test:checkout` passe, et le benchmark de latence local montre p95 < 250 ms sur 3 runs consécutifs.

Faible :

> Rendre le checkout plus rapide.

## Heuristiques de quantification

- **Bug** : reproduction d'abord, fix ensuite, validateur failing→passing
  quand possible.
- **Tests** : nom de la commande exacte et condition de passage requise.
- **Performance** : métrique, seuil cible, méthode de mesure, nombre de
  runs.
- **Qualité** : barre d'acceptation observable (exemples relus,
  lint/typecheck/tests verts, artefact validé par l'utilisateur).
- **Recherche** : décision que la recherche doit permettre de prendre,
  sources/systèmes dans le périmètre, standard de preuve.
- **Ops** : état sain, fenêtre de monitoring, seuil d'échec, trigger de
  rollback/escalade.

## Questions de clarification

Ne pose une question que quand une réécriture raisonnable risquerait de
viser le mauvais résultat. Garde-la courte, orientée vers le validateur ou
la limite de périmètre manquants.

- "Quelle métrique définit le succès ici : latence, coût, précision, ou
  comportement visible utilisateur ?"
- "Contre quel environnement je vérifie : local, staging, ou prod ?"
- "Quelle est la preuve minimale que tu veux avant que je considère cet
  objectif atteint ?"

Si l'utilisateur ne peut pas donner de métrique, propose le validateur
binaire le plus honnête possible et demande confirmation avant d'écrire
`GOAL.md`.

```tandem-comments
// Schema: { "<id>": { anchor:{exact,prefix,suffix,pos?}, status:open|resolved, thread:[{author,ts,text}] } }
// Anchor = quote from the prose. To locate: search for "exact", disambiguate via prefix/suffix.
{}
```
