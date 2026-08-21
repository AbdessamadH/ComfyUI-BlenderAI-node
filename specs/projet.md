# Skill « Lecture critique du règlement de concours »

## Objectif
Abdessamad est architecte freelance au Maroc. Il participe à des concours d'architecture (majoritairement des opérations immobilières d'habitat), publiés et téléchargés via marchespublics.gov.ma. Chaque dossier de concours contient un règlement et des pièces annexes (programme, CCAP, etc.), souvent en Word ou en PDF scanné, parfois volumineux et mal structurés.

Ce premier skill (le 1er d'une série prévue pour outiller son workflow de concours) lit ces pièces et produit une synthèse structurée en français qui lui permet de démarrer sa conception (qu'il réalise lui-même) sans avoir à dépouiller manuellement chaque document. C'est la version la plus simple qui a déjà de la valeur : elle ne couvre que la lecture/synthèse du règlement, pas les autres étapes du workflow (esquisse, DXF, 3D, métré, plaquette — prévues comme skills séparés, un par un, dans de futurs cycles `/spec` → `/build`).

## Portée

### Dans le périmètre
- Un skill Claude Code (`SKILL.md`) qui lit **un fichier à la fois** (Word ou PDF, éventuellement scanné) fourni par l'utilisateur.
- Extraction et synthèse en français des 9 rubriques listées ci-dessous.
- Génération/mise à jour d'un document Word (.docx) de synthèse, avec paragraphes et tableaux selon la nature du contenu.
- Comportement cumulatif : chaque nouvelle pièce complète ou corrige un document de synthèse existant pour le même concours (pas un document par pièce).
- Le skill est livré dans un nouveau repo GitHub privé dédié : **`architecte-workflow-skills`**.

### Hors périmètre (pour ce spec)
- Les 5 autres skills mentionnés (sketch → DXF, benchmark, 3D via Blender MCP, métré, mise en page plaquette A3 sur Canva) — chacun fera l'objet d'un spec et d'un build séparés, plus tard.
- Les idées de skills additionnels évoquées (veille concours, notice architecturale, rétroplanning/checklist, vérification réglementaire, archivage structuré) — non spécifiées ni construites ici.
- Traitement de documents en arabe uniquement (n'arrive jamais dans le workflow de l'utilisateur).
- Traitement automatique de tout un dossier de concours en une fois (l'utilisateur fournit les pièces une à une, volontairement, car elles ne sont pas toutes disponibles en même temps et varient d'un marché à l'autre).
- Création automatique de l'arborescence de dossiers par concours — l'utilisateur crée et nomme lui-même le dossier de chaque concours ; le skill lit/écrit dans le chemin qu'on lui donne.

## Besoins exacts

**Entrée** : un seul fichier par exécution, Word (.docx/.doc) ou PDF (texte ou scanné), plus le chemin du dossier du concours concerné (créé au préalable par l'utilisateur). Les documents sont toujours en français ; certaines pièces existent en double français/arabe avec le même contenu — le skill se base uniquement sur la version française et ignore la version arabe.

**Traitement** :
- Si le PDF est scanné (pas de texte sélectionnable), passer par de l'OCR pour extraire le texte.
- Identifier et extraire les informations relatives aux 9 rubriques suivantes :
  1. Nature et objet du concours
  2. Programme physique (typologie des logements, surfaces, nombre d'unités, répartition) — en tableau
  3. Format et contenu exact du rendu attendu (nombre de planches, format papier, échelles, pièces graphiques et écrites demandées, format numérique de dépôt)
  4. Critères de jugement du jury / grille d'évaluation si mentionnée
  5. Calendrier (dates limites de remise, visite de site, questions-réponses) — en tableau
  6. Pièces administratives à fournir (hors production architecturale)
  7. Budget/enveloppe financière si mentionnée
  8. Contraintes du terrain issues du règlement (servitudes, COS/CUS, prospects, si précisés dans le texte)
  9. Points d'ambiguïté ou d'attention identifiés par le skill (passages flous, contradictoires, ou illisibles)
- Utiliser des tableaux pour les rubriques à données structurées (programme physique, calendrier ; tableau libre pour toute autre rubrique s'y prêtant), des paragraphes pour les rubriques narratives.

**Sortie** : un fichier `.docx` de synthèse dans le dossier du concours fourni par l'utilisateur.
- S'il n'existe pas encore pour ce concours, le skill le crée.
- S'il existe déjà, le skill le met à jour rubrique par rubrique avec les informations de la nouvelle pièce, sans écraser les rubriques non concernées par cette pièce.
- En cas de contradiction entre une information déjà présente et une information de la nouvelle pièce, **la nouvelle pièce prime** et remplace l'ancienne valeur (comportement voulu : les pièces ultérieures rectifient/complètent le marché initial).

## Cas limites
- Passage illisible dans un PDF scanné (mauvaise qualité d'OCR) → signaler explicitement dans la rubrique concernée (« passage illisible à vérifier manuellement », avec localisation si possible) plutôt que deviner ou ignorer silencieusement.
- Information absente de la pièce fournie pour une rubrique donnée → laisser la rubrique en l'état (« non spécifié à ce stade ») jusqu'à ce qu'une autre pièce la renseigne ; ne pas inventer de valeur.
- Nouvelle pièce contredisant une info déjà présente → la nouvelle pièce remplace l'ancienne (voir Besoins exacts).
- Fichier fourni non pertinent, vide, ou pas un document de concours reconnaissable → signaler clairement à l'utilisateur plutôt que de produire une synthèse vide ou erronée.
- Premier fichier fourni pour un concours (document de synthèse pas encore créé) → le skill crée le document dans le dossier du concours indiqué.
- Document bilingue français/arabe à contenu dupliqué → traiter uniquement la partie française.

## Contraintes non-fonctionnelles
- Le skill vit dans un nouveau repo GitHub privé dédié : `architecte-workflow-skills` (à créer).
- Une marge d'erreur mineure est tolérée : une information manquante doit être signalée comme telle plutôt que d'être fausse ou inventée. Pas d'exigence de fidélité à 100 % pour valider le skill.
- S'appuie sur les capacités existantes de génération/édition Word (paragraphes + tableaux) plutôt que de réinventer un générateur de documents.

## Définition de « terminé »
- [ ] Le repo GitHub privé `architecte-workflow-skills` existe, avec le skill livré dedans (structure Claude Code skill standard).
- [ ] Le skill traite un fichier Word ou PDF (y compris scanné, avec OCR) fourni un par un et produit/actualise un `.docx` de synthèse dans le dossier du concours indiqué.
- [ ] Les 9 rubriques définies sont présentes dans le document de synthèse, avec tableaux pour le programme physique et le calendrier, paragraphes pour le reste.
- [ ] Testé par l'utilisateur avec les pièces d'un vrai marché réel mais dépassé (fourni par lui) : le document produit couvre les 9 rubriques avec une marge d'erreur mineure acceptée (infos manquantes signalées, pas inventées).
- [ ] Le comportement cumulatif est vérifié : une deuxième pièce complète/corrige le document existant sans écraser les rubriques non concernées, et une contradiction est bien résolue en faveur de la pièce la plus récente.
- [ ] Un passage illisible (mauvais OCR) est signalé dans le document plutôt que deviné ou passé sous silence.
