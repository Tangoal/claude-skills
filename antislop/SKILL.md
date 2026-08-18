---
name: antislop
description: Empêche de livrer une UI générique ("IA slop") quand on crée ou reprend une page/un élément d'interface précis. Force à définir le vrai job de l'écran, à s'appuyer sur des références réelles et sur le design system existant du projet, à rejeter les motifs par défaut (dashboard générique, cartes/bento par défaut, fausses métriques, dégradés décoratifs, labels vagues, états manquants), et fait passer un finish gate avant de considérer le travail terminé. À utiliser quand l'utilisateur demande `/antislop`, ou de créer/refaire/reprendre une page, un écran, un composant ou un élément d'UI en particulier.
---

# /antislop — Ne pas livrer de l'IA slop

## Rôle du skill

Un LLM livré à lui-même sur une tâche d'UI converge vers les mêmes
défauts : dashboard générique, grille de cartes, métriques inventées,
dégradés décoratifs, libellés vagues ("Overview", "Insights"), états
manquants (loading/empty/error). Ce skill force une étape de cadrage et
de vérification avant et après l'implémentation pour casser cette
convergence.

Ce skill ne remplace pas `frontend-design` ou `ui-ux-pro-max` (choix
esthétiques, palettes, typographies, patterns par stack) ni `dataviz`
(graphiques) — utilise-les pour les décisions de style. `/antislop` est
la couche de discipline au-dessus : il force à regarder le produit réel
avant de choisir un layout, et à vérifier le résultat avant de le
livrer.

N'utilise ce skill que pour une page, un écran, ou un composant d'UI
identifiable — pas pour du code backend, des scripts, ou de la
configuration sans surface visuelle.

## Workflow

### 1. Cadrer le job de l'écran/élément avant tout choix de layout

Avant d'écrire une ligne de JSX/HTML, réponds explicitement (mentalement
ou à voix haute pour l'utilisateur) à :

- Quel est le job réel de cet écran/élément ? (une phrase, pas
  "afficher des infos")
- Qui l'utilise, dans quel contexte (mobile en déplacement ? desktop en
  usage intensif ? admin interne ?) ?
- Quelle est l'action principale que l'utilisateur doit pouvoir faire ?
- Quel contenu doit obligatoirement être présent ?
- Quels états sont importants pour ce cas précis (pas la liste
  générique — celle qui compte vraiment ici) ?

Si l'un de ces points est ambigu et changerait significativement le
résultat, pose une question courte à l'utilisateur plutôt que de
deviner.

### 2. Regarder l'existant avant d'inventer

- Si le projet a un `docs/DESIGN.md`, lis-le en premier — il fige
  l'identité visuelle (tokens, prose au format design.md) et prime sur
  ce qui peut être déduit du code seul.
- Cherche dans le projet le design system déjà en place : composants
  réutilisables, tokens de couleur/espacement, conventions de layout
  déjà établies ailleurs dans le code. Réutilise-les — n'invente pas un
  nouveau vocabulaire visuel pour un seul écran.
- Si le produit a des écrans voisins (autre page du même flux, même
  section), regarde-les pour rester cohérent en densité, hiérarchie,
  ton des libellés.
- Si l'utilisateur reprend un écran existant, lis le fichier actuel en
  entier avant de le réécrire — comprends ce qui marche déjà et ne le
  jette pas sans raison.

### 3. Chercher des références réelles (pas des templates)

Avant de choisir un layout, cherche 2-3 références de produits réels
qui résolvent un problème comparable :

- Si l'utilisateur a donné des liens/captures, pars de ceux-là.
- Sinon, utilise `WebSearch`/`WebFetch` pour trouver des écrans réels
  comparables (produits connus dans la même catégorie), ou demande à
  l'utilisateur 2-3 exemples s'il en a en tête.
- Si aucune recherche n'est possible et que l'utilisateur ne fournit
  rien, ne bloque pas le travail — mais reste plus vigilant en étape 5
  (auto-critique) faute de références externes pour calibrer.

Traite les références comme des preuves de décisions structurelles
(hiérarchie, densité, quels contrôles existent, comment les états sont
gérés) — jamais comme des assets à copier telles quelles. Ne recopie
jamais le branding, les textes propriétaires, ou le layout exact d'un
produit tiers.

### 4. Écrire un contrat de design court avant de coder

Fige en quelques lignes (dans la conversation, pas forcément un
fichier) :

- Job de l'écran (repris de l'étape 1)
- Hiérarchie : quel élément domine visuellement, dans quel ordre le
  regard doit aller
- Composants autorisés (ceux du design system existant, + nouveaux si
  justifiés)
- États requis et comment chacun s'affiche
- Comportement responsive si pertinent
- Motifs génériques à explicitement rejeter pour cet écran (liste en
  étape 6, celle qui s'applique ici)

### 5. Construire

Implémente avec les composants et tokens existants du produit. Écris du
contenu et des libellés spécifiques au produit — pas de texte
placeholder générique ("Lorem", "Sample data", "John Doe") sauf si
explicitement demandé par l'utilisateur pour du mock.

### 6. Rejeter ces défauts

Refuse le résultat (le tien y compris) s'il contient :

- Un shell de dashboard générique choisi avant d'avoir compris le
  produit
- Une grille de cartes ou un bento layout comme réponse par défaut
- De fausses métriques, flux d'activité, témoignages, utilisateurs ou
  données placeholder non demandées
- Des dégradés, glows, glass, blobs décoratifs sans raison produit
- Des libellés vagues ("Overview", "Insights", "Learn more") là où un
  libellé précis est possible
- Des contrôles qui ne font rien ou ne mènent nulle part
- Des états loading/empty/error/succès/permission manquants alors
  qu'ils sont pertinents pour ce contexte précis (défini en étape 1)
- Un layout desktop simplement compressé pour mobile, sans réflexion
  responsive réelle
- Un vocabulaire visuel qui pourrait être réutilisé tel quel pour un
  autre produit sans rien changer

### 7. Finish gate — vérifier avant de dire "terminé"

Ne considère le travail fini que si :

- Le job de l'écran est évident immédiatement
- Une action principale domine clairement la hiérarchie
- Chaque contrôle visible a un effet réel
- Le contenu et les libellés appartiennent spécifiquement à ce produit
- Les états requis (étape 1/4) sont implémentés et atteignables
- Le comportement responsive est intentionnel, pas subi
- Les règles du design system existant sont respectées
- Le résultat rendu (pas juste le code) a été regardé — lance le dev
  server et vérifie dans le navigateur si le contexte le permet, comme
  le veut la règle générale du projet sur les changements UI

### 8. Rapporter

Rapporte de façon concise : ce qui a été construit, les états
vérifiés, et les écarts assumés par rapport au contrat de design
(étape 4) si l'implémentation a dû s'en éloigner. Pas de récapitulatif
promotionnel du type "cette interface offre une expérience moderne et
intuitive" — dis ce qui a été fait et ce qui reste à vérifier par
l'utilisateur.
