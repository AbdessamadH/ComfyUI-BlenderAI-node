---
description: Read specs/projet.md and build exactly what it describes, nothing more. Ends with a checklist of covered requirements.
---

# /build — Implémenter exactement le spec

Tu es en mode implémentation stricte. Ton travail est de construire **exactement** ce qui est décrit dans `specs/projet.md`, ni plus, ni moins.

Arguments reçus (précisions éventuelles sur cette session de build) : $ARGUMENTS

## Étape 0 — Charger le spec

Lis `specs/projet.md`. S'il n'existe pas, arrête-toi et dis à l'utilisateur d'exécuter `/spec` d'abord — ne construis rien à partir d'une supposition.

Si le fichier existe mais qu'une section clé est manquante, vide, ou contient une ambiguïté qui changerait ce que tu construis (ex. « Besoins exacts » vague au point de permettre plusieurs implémentations différentes), pose une question de clarification à l'utilisateur avant de commencer plutôt que de deviner. Pour tout le reste, suis le spec tel qu'il est écrit.

## Étape 1 — Planifier le travail

Extrais du spec la liste concrète des besoins à couvrir : chaque item de « Besoins exacts », chaque cas limite listé, et chaque critère de « Définition de « terminé » ». Utilise cette liste comme plan de travail (TaskCreate si l'outil est disponible), avec un item par besoin/cas limite/critère.

Ne construis rien qui ne soit pas rattaché à un item du spec :
- **Pas de scope creep** : n'ajoute pas de fonctionnalités, options de config, abstractions, gestion d'erreurs ou validations non demandées, même si elles semblent « évidemment utiles ». Si tu identifies un manque réel dans le spec pendant le build, signale-le à l'utilisateur à la fin plutôt que de l'implémenter silencieusement.
- **Respecte le périmètre** : tout ce qui est listé sous « Hors périmètre » dans le spec reste hors périmètre, même si c'est tentant ou trivial à ajouter au passage.
- **Pas de sous-construction** : couvre chaque besoin listé, y compris les cas limites explicitement mentionnés — ne les laisse pas de côté pour aller plus vite.

## Étape 2 — Construire

Implémente le code en suivant les besoins exacts, les cas limites et les contraintes non-fonctionnelles du spec. Utilise les conventions déjà en place dans ce repo (style de code, structure des fichiers, dépendances existantes) plutôt que d'en introduire de nouvelles sans raison tirée du spec.

Marque chaque item de ton plan comme terminé au fur et à mesure, pas en fin de tâche groupée.

## Étape 3 — Vérifier

Avant de conclure, relis chaque critère de la section « Définition de « terminé » » du spec et vérifie-le concrètement (tests, exécution, lecture du code) plutôt que de le supposer couvert. Fais tourner les tests/lint/typecheck pertinents s'ils existent dans le repo.

## Étape 4 — Rapport final

Termine par une liste explicite des besoins du spec que tu as couverts, sous cette forme :

```markdown
## Besoins couverts (specs/projet.md)
- [x] <besoin exact 1>
- [x] <besoin exact 2>
- [x] <cas limite 1>
- [x] <critère « terminé » 1>
...
```

Si un item du spec n'a pas pu être couvert (ambiguïté non résolue, dépendance manquante, incompatibilité découverte pendant le build), liste-le séparément sous `## Non couvert` avec la raison précise — ne le fais pas disparaître silencieusement de la liste.

Ne termine pas la commande en ajoutant des fonctionnalités, tests ou fichiers qui ne correspondent à aucun item de cette liste.
