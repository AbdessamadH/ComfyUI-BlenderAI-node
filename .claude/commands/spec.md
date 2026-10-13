---
description: Interview the user one question at a time to clarify a project idea, then write a precise spec to specs/projet.md. No building until the spec is validated.
---

# /spec — Clarify before you build

Tu es en mode spécification. Ton unique travail dans cette commande est de comprendre ce que l'utilisateur veut, puis d'écrire un plan précis dans `specs/projet.md`. **Tu ne dois écrire, modifier ou exécuter aucun code produit tant que l'utilisateur n'a pas validé le spec final.** Tu peux lire le code existant pour te renseigner, mais tu ne construis rien.

Arguments reçus (le sujet du projet, s'il est fourni) : $ARGUMENTS

## Étape 1 — Interview, une question à la fois

Pose **une seule question à la fois**, attends la réponse de l'utilisateur avant de poser la suivante. Ne liste jamais plusieurs questions d'un coup. Adapte chaque question aux réponses précédentes plutôt que de suivre une liste rigide.

Si $ARGUMENTS contient déjà une description du projet, commence par une question de clarification sur ce texte plutôt que de redemander l'évidence.

Couvre au minimum ces zones, dans un ordre naturel, en creusant chaque zone tant que la réponse reste vague :

1. **Objectif** — Quel problème ça résout, pour qui, et pourquoi maintenant. Quelle est la version la plus simple qui aurait déjà de la valeur ?
2. **Portée** — Qu'est-ce qui est explicitement dans le périmètre, et qu'est-ce qui est explicitement hors périmètre ?
3. **Besoins exacts** — Comportement attendu, entrées/sorties, contraintes techniques (langage, framework, compatibilité avec l'existant dans ce repo), intégrations.
4. **Utilisateurs / contexte d'usage** — Qui l'utilise, comment, avec quelle fréquence, dans quel environnement.
5. **Cas limites** — Erreurs possibles, entrées invalides, comportements en cas d'échec, concurrence, données vides/énormes, compatibilité arrière.
6. **Contraintes non-fonctionnelles** — Performance, sécurité, compatibilité, dépendances à éviter.
7. **Définition de « terminé »** — Quels critères concrets et vérifiables permettent de dire que c'est fini (tests, comportements observables, cas de démo). Comment l'utilisateur va-t-il vérifier que c'est bon ?

Règles pour l'interview :
- Une question à la fois, courte et concrète. Évite les questions à choix multiples fermées sauf si ça aide vraiment à trancher vite.
- Si une réponse est vague ou ambiguë, repose une question de clarification avant d'avancer.
- Reformule brièvement ce que tu as comprisde temps en temps pour vérifier l'alignement, sans transformer ça en résumé complet à chaque tour.
- N'invente jamais de réponse à la place de l'utilisateur. S'il dit « peu importe » ou « à toi de voir », propose une option par défaut raisonnable et demande confirmation explicite plutôt que de trancher silencieusement.
- Arrête l'interview quand tu as assez d'informations pour écrire un spec sans zones grises importantes — pas nécessairement quand toutes les questions possibles sont épuisées.

## Étape 2 — Rédiger le spec

Une fois que tu as une compréhension claire et confirmée, écris (ou mets à jour) `specs/projet.md` avec cette structure :

```markdown
# <Nom du projet>

## Objectif
<Le problème résolu, pour qui, pourquoi maintenant.>

## Portée
### Dans le périmètre
- ...
### Hors périmètre
- ...

## Besoins exacts
<Comportement attendu, entrées/sorties, contraintes techniques, intégrations.>

## Cas limites
- <cas limite> → <comportement attendu>
- ...

## Contraintes non-fonctionnelles
- ...

## Définition de « terminé »
- [ ] <critère vérifiable>
- [ ] <critère vérifiable>
...
```

Crée le dossier `specs/` s'il n'existe pas.

## Étape 3 — Validation

Montre le contenu du spec écrit et demande explicitement à l'utilisateur s'il valide ou s'il veut des ajustements. Si des ajustements sont demandés, retourne à l'étape 1 pour clarifier ce qui doit changer, puis met à jour le fichier.

Ne commence aucune implémentation (code, fichiers de production, dépendances) dans cette commande, même si l'utilisateur semble pressé — rappelle-lui que la construction commencera dans une tâche séparée une fois le spec validé.
