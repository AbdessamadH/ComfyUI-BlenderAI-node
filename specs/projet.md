# Skill « Esquisse scannée → DXF (AutoCAD) »

## Objectif
Deuxième skill de la série d'outillage du workflow de concours d'architecture d'Abdessamad. Il convertit une esquisse manuscrite (plan, coupe, ou façade), dessinée à la main à une échelle conventionnelle (1/1000 à 1/100) sur papier A3/A4/A5 puis scannée, en un fichier DXF exploitable directement dans AutoCAD. Il est appelé de façon répétée au fil des itérations de conception (l'architecte redessine, rescanne, reconvertit) jusqu'à ce que la conception réponde aux exigences du règlement de concours — cette satisfaction est jugée par l'architecte lui-même, pas par le skill. Le DXF produit sert aussi d'entrée au skill #4 (élaboration 3D via Blender), prévu plus tard.

## Portée

### Dans le périmètre
- Conversion d'**une esquisse à la fois** (un fichier/une feuille source), de type plan, coupe, ou façade.
- Si cette feuille contient **plusieurs niveaux du même bâtiment** (RDC, R+1, R+2...) — pratique courante de l'utilisateur, qui ne les sépare jamais par calque — ils sont **tous traités ensemble dans la même invocation et le même DXF de sortie**, chacun identifié uniquement par son titre texte (pas de calque par niveau), positionnés côte à côte **dans la même disposition relative que sur l'esquisse d'origine**.
- Prise en charge des 3 types de dessins (plan, coupe, façade), chacun avec ses conventions propres.
- Lecture de cotes, nomenclature des espaces, et notations d'angle explicites portées à la main sur l'esquisse.
- Nettoyage/redressement automatique de la géométrie (murs alignés, angles à 90° par défaut).
- Sortie en fichier DXF structuré par calques selon les conventions de trait de l'utilisateur (calques par type d'élément — murs/menuiserie/aménagement/caché — jamais par niveau).
- Auto-vérification de complétude et rendu visuel de contrôle avant de livrer le DXF (voir Contraintes non-fonctionnelles).

### Hors périmètre
- Jugement de la qualité architecturale ou vérification de conformité réglementaire (rôle de l'architecte, et en amont du skill #1).
- Traitement de **plusieurs esquisses distinctes** (fichiers séparés, ou projets différents) en une seule invocation — ça reste un fichier à la fois ; seule la présence de plusieurs niveaux d'un même bâtiment *dans une même feuille* est prise en charge nativement (voir ci-dessus).
- Les autres skills de la série (benchmark, 3D, métré, plaquette A3).
- Support de conventions de dessin autres que celles décrites ici (si l'utilisateur change de convention plus tard, ce spec devra être révisé).

## Besoins exacts

**Entrée** : une image scannée (PDF ou image) d'un seul dessin architectural à main levée, dessiné à une échelle conventionnelle (1/1000, 1/500, 1/200, ou 1/100) sur papier A3, A4 ou A5. L'utilisateur précise à l'invocation : le type de dessin (plan / coupe / façade), l'échelle, le dossier du concours où enregistrer le résultat, et le nom du fichier de sortie.

**Conventions de trait à reconnaître** (constantes pour tous les dessins) :
- Trait fort → murs (structure).
- Trait moyen → menuiserie (portes, fenêtres).
- Trait fin → aménagement vu (meubles, murs bas, carrelage...).
- Trait interrompu (tirets) → éléments cachés (au-dessus du plan de coupe).
- Pour les façades : épaisseur de trait dégressive du plus proche (épais) au plus loin (fin), pour indiquer la profondeur.

**Épaisseurs de murs par défaut** (murs toujours dessinés en double ligne, à l'échelle, sur l'esquisse) :
- Cloisons intérieures : 10 cm.
- Murs extérieurs : 35 cm.
- Murs mitoyens : 25 cm.
- Une cotation explicite sur l'esquisse prime sur ces valeurs par défaut.

**Portes et fenêtres** :
- Portes : dessinées avec l'ouvrant et le sens d'ouverture ; hauteur par défaut 2,20 m (sauf cotation contraire indiquée par l'utilisateur).
- Fenêtres : deux traits rapprochés de 1 mm sur l'esquisse → représentés à 0,7 mm dans le DXF ; pour les fenêtres coulissantes, des flèches indiquent le sens d'ouverture ; allège/linteau notés selon la convention 1/1.2 (étages) ou 1.2/1 (RDC), sauf indication contraire sur l'esquisse.
- Portes-fenêtres : notées « PF » sur l'esquisse, à reconnaître comme telles.

**Calage dimensionnel** : déduit de l'échelle indiquée par l'utilisateur et du format papier (A3/A4/A5). Les cotes explicitement écrites sur l'esquisse priment sur ce calcul par échelle quand les deux sont présentes et divergent (voir cas limites pour la gestion d'un écart significatif).

**Nettoyage géométrique** : le skill redresse et nettoie la géométrie automatiquement (aligne les murs, force les angles à 90° par défaut) — sauf si l'esquisse porte une cotation d'angle explicite en degrés indiquant un angle volontairement différent de 90°, auquel cas cet angle est respecté tel quel.

**Sortie** : un fichier DXF dans le sous-dossier `dxf/` du dossier du concours indiqué, nommé par l'utilisateur (ex. `plan-rdc.dxf`, `coupe-aa.dxf`, `facade-nord.dxf`). Le DXF utilise des calques correspondant aux catégories de trait ci-dessus (murs, menuiserie, aménagement, éléments cachés ; pour les façades, calques par plan de profondeur), en respectant les épaisseurs de mur et de trait données. Les nomenclatures d'espaces (nom, et surface si indiquée) sont reportées en tant qu'entités TEXTE dans le DXF.

## Cas limites
- Esquisse multi-niveaux (RDC/R+1/R+2...) où un niveau n'a pas de titre clair, ou où les niveaux se chevauchent visuellement au point de ne pas pouvoir les séparer → signaler plutôt que deviner à quel niveau appartient un élément.
- Élément de l'esquisse ambigu ou illisible (catégorie de trait pas claire, cote illisible, symbole de porte/fenêtre pas clair) → signaler explicitement dans le résultat plutôt que deviner et produire un DXF silencieusement faux.
- Cote écrite sur l'esquisse en désaccord significatif avec la valeur déduite de l'échelle/format papier → la cote explicite prime, mais l'écart est signalé pour que l'architecte le vérifie, plutôt que d'être résolu silencieusement.
- Angle qui semble volontairement différent de 90° sur le dessin mais sans annotation en degrés → traité comme 90° par défaut (règle de redressement), avec un signalement que l'intention pourrait être différente, pour que l'architecte ajoute la cotation d'angle si besoin.
- Épaisseur de mur non standard sans cotation explicite → utiliser les valeurs par défaut (10/35/25 cm) et signaler l'hypothèse faite.

## Contraintes non-fonctionnelles
- Le DXF doit être ouvrable et exploitable directement dans AutoCAD, sans retouche de structure de fichier nécessaire avant ouverture.
- Les murs sont toujours représentés **pochés** (remplissage plein entre les deux lignes), quelle que soit leur épaisseur — une double ligne non remplie n'est pas assez lisible à l'échelle d'un plan complet.
- Le skill part d'un **gabarit DXF réutilisable** (calques et style de cote pré-configurés avec la bonne échelle d'affichage) plutôt que de redéfinir ces conventions à chaque génération.
- **Avant de livrer un DXF**, le skill vérifie que chaque catégorie attendue (murs extérieurs, murs intérieurs, portes, fenêtres, aménagement, cotes) contient bien au moins une entité générée, et signale explicitement toute catégorie vide plutôt que de livrer un résultat silencieusement incomplet.
- **Avant de livrer un DXF**, le skill en génère un rendu visuel (image) et le compare visuellement à l'esquisse source, plutôt que de considérer la génération terminée dès que le fichier s'écrit sans erreur.
- Vit dans le repo `architecte-workflow-skills`, comme un nouveau skill à côté de `lecture-reglement-concours`.

## Définition de « terminé »
- [ ] Le skill est livré dans `architecte-workflow-skills` (structure Claude Code skill standard).
- [ ] Testé par l'utilisateur avec une vraie esquisse scannée qu'il fournira (plan, coupe, ou façade).
- [ ] Le DXF produit respecte les calques et épaisseurs de trait convenus, s'ouvre correctement, et la géométrie est redressée/nettoyée (angles à 90° sauf cotation explicite contraire).
- [ ] Les ambiguïtés/illisibilités et écarts de cotation sont signalés plutôt que devinés silencieusement.
- [ ] Le fichier est enregistré dans `dxf/` du dossier du concours indiqué, avec le nom donné par l'utilisateur.
