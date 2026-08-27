---
description: Wrap /build and /review in a loop that keeps fixing gaps until a stop condition (default tests verts) is met, or a safety cap is hit.
---

# /cycle — Boucler build → review → fix jusqu'à la condition d'arrêt

Tu enveloppes la séquence `/build` → tests → `/review` dans une boucle qui corrige les écarts trouvés et recommence, jusqu'à ce que la condition d'arrêt soit remplie. Tu ne construis toujours que ce qui est écrit dans `specs/projet.md` — cette boucle sert à converger vers le spec, pas à en sortir.

Arguments reçus : $ARGUMENTS (condition d'arrêt personnalisée et/ou nombre max d'itérations, ex. `tests verts` ou `max=8 tests verts`). Si vide, utilise la condition d'arrêt par défaut ci-dessous.

## Condition d'arrêt

Par défaut : **les tests du repo passent intégralement (exit code 0) ET `/review` ne remonte aucun manque, bug, ni violation de périmètre.**

Si $ARGUMENTS précise une autre condition (ex. « tests verts » seul, sans exiger un `/review` propre), utilise celle-ci à la place — mais annonce explicitement au début de la boucle quelle condition tu vérifies, pour que ce soit sans ambiguïté.

Cap de sécurité : 5 itérations par défaut (ou la valeur `max=N` si fournie dans $ARGUMENTS). Ce n'est pas une boucle infinie.

## Étape 0 — Prérequis

Lis `specs/projet.md`. S'il n'existe pas, arrête-toi et dis à l'utilisateur d'exécuter `/spec` d'abord. Repère aussi comment lancer la suite de tests du repo (script npm, `pytest`, etc.) — si aucune suite de tests n'existe et que la condition d'arrêt par défaut s'applique, dis-le à l'utilisateur et demande une condition d'arrêt alternative avant de boucler dans le vide.

## Étape 1 — Boucle (itération 1 à N)

À chaque itération :

1. **Build.** À l'itération 1, lance `/build`. Aux itérations suivantes, ne relance pas un `/build` complet : corrige uniquement les manques/bugs listés par le `/review` de l'itération précédente, en restant strictement dans le périmètre de `specs/projet.md` — pas de nouvelle fonctionnalité, pas de nettoyage non demandé.
2. **Tests.** Exécute la suite de tests du repo et note le résultat (vert/rouge, et le détail des échecs le cas échéant).
3. **Review.** Lance `/review` pour comparer le code réel au spec point par point.
4. **Évaluer la condition d'arrêt.** Compare le résultat des tests et du `/review` à la condition définie à l'étape 0.
   - **Remplie** → passe à l'étape 2 (rapport final) et arrête la boucle.
   - **Non remplie et itérations restantes** → note précisément ce qui bloque (tests en échec + liste des manques/bugs du `/review`), et relance l'itération suivante en ciblant ces points exactement.
   - **Non remplie et cap atteint** → arrête la boucle sans continuer indéfiniment, passe à l'étape 2 avec le statut « non convergé ».

Annonce brièvement le résultat de chaque itération (numéro, tests verts/rouges, nombre de manques/bugs restants) avant de continuer — pas un pavé, une ligne suffit.

## Étape 2 — Rapport final

```markdown
## Cycle terminé après <N> itération(s)
Condition d'arrêt : <condition vérifiée>
Statut : <ATTEINTE | NON CONVERGÉ (cap de N itérations atteint)>

### Tests
<résumé du dernier run : vert/rouge, échecs restants s'il y en a>

### Review (dernier passage)
<résumé du dernier /review : conforme / manques / bugs restants>
```

Si la condition d'arrêt est atteinte, dis-le clairement en une phrase et arrête — pas de nouvelle itération, pas d'ajout hors spec pour « améliorer » davantage.

Si le cap est atteint sans convergence, liste explicitement ce qui reste à corriger et demande à l'utilisateur comment il veut continuer (augmenter le cap, intervenir manuellement, revoir le spec si le blocage vient d'une ambiguïté du plan) plutôt que de boucler silencieusement au-delà de la limite.

## Utilisation en tâche de fond avec /loop

Cette commande peut aussi être invoquée périodiquement via le skill `/loop` (ex. `/loop 10m /cycle`) si tu préfères espacer les itérations plutôt que les enchaîner dans le même tour — utile si la suite de tests est lente ou dépend d'une CI externe. Dans ce cas, chaque invocation de `/cycle` ne fait qu'**une** itération de la boucle ci-dessus (pas les 5 d'affilée) : dès que la condition d'arrêt est remplie, indique-le explicitement dans le rapport et, si cette invocation tourne dans une boucle `/loop` à pacing dynamique, appelle `ScheduleWakeup` avec `stop: true` pour mettre fin aux invocations suivantes au lieu d'en reprogrammer une.
