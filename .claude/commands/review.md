---
description: Compare what was built against specs/projet.md, point by point, and list every gap or bug found.
---

# /review — Comparer le build au spec

Tu es en mode revue. Ton travail est de comparer ce qui a été **réellement construit** (le code du repo, pas le rapport de `/build`) au plan écrit dans `specs/projet.md`, point par point, et de lister chaque manque ou bug trouvé. Tu ne corriges rien dans cette commande — tu constates.

Arguments reçus (précisions éventuelles, ex. un sous-dossier ou une PR à cibler) : $ARGUMENTS

## Étape 0 — Charger le spec

Lis `specs/projet.md`. S'il n'existe pas, arrête-toi et dis à l'utilisateur d'exécuter `/spec` d'abord — il n'y a rien à comparer sans plan écrit.

## Étape 1 — Inventaire des points à vérifier

Extrais du spec la liste complète des points vérifiables, dans l'ordre du document :
- chaque item de « Besoins exacts »
- chaque ligne de « Portée » (dans le périmètre **et** hors périmètre)
- chaque cas limite listé
- chaque contrainte non-fonctionnelle
- chaque critère de « Définition de « terminé » »

Chaque point devient une ligne à vérifier — ne regroupe pas plusieurs exigences en une seule vérification.

## Étape 2 — Vérifier chaque point contre le code réel

Pour chaque point de la liste, va lire le code, exécuter les tests pertinents, ou lancer l'app/la fonctionnalité pour vérifier ce qui existe réellement — ne te fie pas à un résumé antérieur ni au rapport final d'un `/build` précédent. Détermine pour chaque point l'un de ces statuts :

- **Conforme** — le point est implémenté tel que décrit dans le spec.
- **Manquant** — le point n'est pas implémenté du tout.
- **Bug / divergence** — le point est implémenté mais se comporte différemment de ce que décrit le spec (y compris partiellement implémenté, ou implémenté mais cassé).
- **Hors périmètre violé** — quelque chose listé sous « Hors périmètre » a quand même été construit.

Pour tout statut autre que Conforme, note l'emplacement précis (fichier:ligne) et une description concrète du problème : quelle entrée/action déclenche l'écart, et quel est le comportement observé vs. attendu.

## Étape 3 — Rapport

Produis un rapport structuré, point par point, dans cet ordre : d'abord tous les manques et bugs (les points qui demandent une action), puis les violations de périmètre, puis un résumé bref des points conformes (une ligne chacun, pas de détail puisqu'il n'y a rien à corriger).

```markdown
## Manques et bugs
- [ ] <point du spec> — MANQUANT : <ce qui est absent>
- [ ] <point du spec> — BUG : <fichier:ligne> — <comportement observé> au lieu de <comportement attendu>
...

## Hors périmètre violé
- <ce qui a été construit alors que listé hors périmètre> — <fichier:ligne>
...

## Conforme
- <point du spec> ✓
...

## Résumé
<N>/<total> points conformes. <N> manques. <N> bugs. <N> violations de périmètre.
```

Si absolument tous les points sont conformes et qu'aucune violation de périmètre n'existe, dis-le clairement en une phrase plutôt que de forcer une section vide.

Ne corrige rien toi-même dans cette commande, même pour un problème trivial — la revue s'arrête au constat. Si l'utilisateur veut que les manques soient corrigés, propose-lui de lancer `/build` à nouveau ou de traiter les corrections dans une tâche séparée.
