# Skill « Esquisse scannée → DXF (AutoCAD) »

## Objectif
Deuxième skill de la série d'outillage du workflow de concours d'architecture d'Abdessamad. Il convertit une esquisse manuscrite (plan, coupe, ou façade), dessinée à la main à une échelle conventionnelle (1/1000 à 1/100) sur papier A3/A4/A5 puis scannée, en un fichier DXF exploitable directement dans AutoCAD. Il est appelé de façon répétée au fil des itérations de conception (l'architecte redessine, rescanne, reconvertit) jusqu'à ce que la conception réponde aux exigences du règlement de concours — cette satisfaction est jugée par l'architecte lui-même, pas par le skill. Le DXF produit sert aussi d'entrée au skill #4 (élaboration 3D via Blender), prévu plus tard.

## Portée

### Dans le périmètre
- Conversion d'**une esquisse à la fois** (un fichier/une feuille source), de type plan, coupe, ou façade.
- Si cette feuille contient **plusieurs niveaux du même bâtiment** (RDC, R+1, R+2...) — pratique courante de l'utilisateur, qui ne les sépare jamais par calque — ils sont **tous traités ensemble dans la même invocation et le même DXF de sortie**, chacun identifié uniquement par son titre texte (pas de calque par niveau), positionnés côte à côte **dans la même disposition relative que sur l'esquisse d'origine**.
- Prise en charge des 3 types de dessins (plan, coupe, façade), chacun avec ses conventions propres.
- Lecture de cotes, nomenclature des espaces, et notations d'angle explicites portées à la main sur l'esquisse.
- Nettoyage/redressement automatique de la géométrie (murs alignés, angles à 90° par défaut ; biais et courbes francs conservés — voir Nettoyage géométrique).
- Entraînement sur des **photos ou captures de plans existants** (plans tiers, éventuellement rendus en couleur, sans cotes) fournis par l'utilisateur, en plus de ses propres esquisses.
- Sortie en fichier DXF structuré par calques selon les conventions de trait de l'utilisateur (calques par type d'élément — murs/menuiserie/aménagement/caché — jamais par niveau).
- Auto-vérification de complétude et rendu visuel de contrôle avant de livrer le DXF (voir Contraintes non-fonctionnelles).

### Hors périmètre
- Jugement de la qualité architecturale ou vérification de conformité réglementaire (rôle de l'architecte, et en amont du skill #1).
- Traitement de **plusieurs esquisses distinctes** (fichiers séparés, ou projets différents) en une seule invocation — ça reste un fichier à la fois ; seule la présence de plusieurs niveaux d'un même bâtiment *dans une même feuille* est prise en charge nativement (voir ci-dessus).
- Les autres skills de la série (benchmark, 3D, métré, plaquette A3).
- Support de conventions de dessin autres que celles décrites ici (si l'utilisateur change de convention plus tard, ce spec devra être révisé).

## Besoins exacts

**Entrée** : une image scannée (PDF ou image) d'un seul dessin architectural à main levée, dessiné à une échelle conventionnelle (1/1000, 1/500, 1/200, ou 1/100) sur papier A3, A4 ou A5 — ou, pour l'entraînement, la photo/capture d'un plan existant. L'utilisateur précise à l'invocation : le type de dessin (plan / coupe / façade), l'échelle (si connue), le dossier du concours où enregistrer le résultat, et le nom du fichier de sortie.

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

**Méthode de lecture** (leçon du test « plan majlis ») : l'esquisse est lue **zone par zone, zoomée avec une grille de coordonnées**, jamais d'un seul coup d'œil global. Les murs (hachurés/pochés) et les fenêtres (traits ou couleurs) sont d'abord **extraits automatiquement par analyse d'image** pour obtenir leurs positions ; la lecture visuelle sert à interpréter (portes, sens d'ouverture, usage des pièces) et à écarter les faux positifs (textes colorés, mobilier, filigranes). Un **inventaire** (nombre de pièces, portes, fenêtres relevés sur l'esquisse ; sièges, sanitaires et mobilier quand il y en a) est établi avant de générer quoi que ce soit. Leçons du test « auditorium » : les traits très clairs (portes, projections en gris pâle) sont recherchés par **plusieurs passes de seuil** ; les morceaux colinéaires interrompus sont **recousus** (trait continu si couverture ≥ 82 %, sinon pointillé → calque caché) ; les éléments répétitifs (sièges en rangées, WC, lavabos, chaises) sont **comptés par rangée** puis redessinés en blocs, au lieu d'être recopiés trait par trait.

**Calage dimensionnel** : quand des cotes sont écrites sur l'esquisse, l'échelle est calée **par recoupement de plusieurs cotes** (et non une seule), puis la géométrie est **reconstruite à partir des cotes** (les pixels ne servent qu'à la topologie et au contrôle). Sinon, échelle indiquée par l'utilisateur + format papier (A3/A4/A5). **Sans cote ni échelle** (plans tiers) : calage sur des éléments normalisés recoupés entre eux (pas des rangées de sièges 1,00 m, portes 0,90 m, cabines WC ≈ 1,00 m), l'incertitude (de l'ordre de ±5 %) étant affichée dans le DXF et le compte rendu. Les cotes explicites priment sur le calcul par échelle quand les deux divergent (voir cas limites).

**Épaisseur des murs** (règle proposée, à valider par l'utilisateur) : épaisseur cotée > épaisseur **déduite de la fermeture des cotes** (rangées de pièces qui se referment exactement) > épaisseur mesurée sur le dessin à l'échelle calée > convention par défaut (35/25/10 cm). Toute épaisseur non cotée est signalée.

**Nettoyage géométrique** : le skill redresse et nettoie la géométrie automatiquement (aligne les murs, force les angles à 90° par défaut) — sauf si l'esquisse porte une cotation d'angle explicite en degrés indiquant un angle volontairement différent de 90°, auquel cas cet angle est respecté tel quel.
Affinage **proposé, à valider** (test « auditorium » : salle en éventail, anneau courbe — forcer 90° aurait détruit le projet) : les directions des murs sont redressées sur les **directions dominantes mesurées** sur le dessin, avec une tolérance de ±4° ; un écart inférieur est une imprécision de main (redressé à 0/90°), un écart franc et répété (ou symétrique) est conservé tel que mesuré, arrondi, et **signalé** pour confirmation. Les murs courbes sont restitués en **vrais arcs**. Une **symétrie** détectée est imposée seulement dans la tolérance (quelques centimètres) ; au-delà, chaque côté est reproduit tel que dessiné et l'écart est signalé.

**Sortie** : un fichier DXF dans le sous-dossier `dxf/` du dossier du concours indiqué, nommé par l'utilisateur (ex. `plan-rdc.dxf`, `coupe-aa.dxf`, `facade-nord.dxf`). Le DXF utilise des calques correspondant aux catégories de trait ci-dessus (murs, menuiserie, aménagement, éléments cachés ; pour les façades, calques par plan de profondeur), en respectant les épaisseurs de mur et de trait données, plus des calques annexes : sièges, abords extérieurs, signalétique (flèches d'évacuation), axes de coupe, textes, cotes, et `ALERTE` (points à vérifier numérotés, à masquer pour le tirage). Unités du DXF : mètres. Les nomenclatures d'espaces (nom, et surface si indiquée) sont reportées en tant qu'entités TEXTE dans le DXF.

**Règles de qualité de dessin** (exigées par l'architecte après le test « auditorium » ; chacune est vérifiée automatiquement avant livraison et bloque la livraison si elle échoue) :
- **Langue** : tout texte du DXF est en français ; les textes d'une esquisse dans une autre langue sont traduits (nomenclature d'usage française : SANITAIRES HOMMES/FEMMES, LOGE, RÉGIE, SORTIE DE SECOURS…).
- **Murs** : les deux faces de chaque mur sont parfaitement parallèles (murs droits) ou concentriques (murs courbes) — chaque mur est reconstruit par son axe et une épaisseur constante, jamais par le contour irrégulier du tracé.
- **Cloisons** : toutes les cloisons dessinées sont reproduites, y compris les cloisons fines des sanitaires (cabines) et des loges.
- **Portes** : chaque porte est placée dans la baie exacte entre les deux tableaux du mur ; vantail et arc d'ouverture au bon tableau et du bon côté ; un vantail ne traverse jamais un mur ; portes doubles et va-et-vient dessinées selon leur symbole.
- **Sanitaires et mobilier fixe** : chaque appareil (WC, lavabo, vasque) est adossé à la face du mur, perpendiculaire à ce mur (radial le long d'un mur courbe), centré dans sa cabine ; aucun appareil ne dépasse dans un mur.
- **Escaliers** : marches parfaitement parallèles, à espacement parfaitement régulier, giron compris entre 0,25 et 0,27 m (nombre de marches recalculé sur la longueur de volée) ; l'écart avec le giron dessiné est signalé.
- **Circulation** : avant livraison, le skill vérifie que **chaque pièce est accessible** depuis l'extérieur en passant par des baies (calcul des espaces fermés par les murs, vitrages fixes et fenêtres) et que **chaque baie entre deux murs est pourvue d'une porte**, sauf les passages ouverts déclarés (espaces décloisonnés). Une pièce inaccessible ou sans porte bloque la livraison : la porte manquante est ajoutée sur le mur le plus plausible (côté circulation, sans heurter le mobilier ni un mur), et listée comme « porte ajoutée, non dessinée sur l'esquisse ».
- Les traits non reconnus sont redressés (droites, vrais arcs) ; un tracé irrégulier n'est jamais reproduit en zigzag.

## Cas limites
- Esquisse multi-niveaux (RDC/R+1/R+2...) où un niveau n'a pas de titre clair, ou où les niveaux se chevauchent visuellement au point de ne pas pouvoir les séparer → signaler plutôt que deviner à quel niveau appartient un élément.
- Élément de l'esquisse ambigu ou illisible (catégorie de trait pas claire, cote illisible, symbole de porte/fenêtre pas clair) → signaler explicitement dans le résultat plutôt que deviner et produire un DXF silencieusement faux.
- Cote écrite sur l'esquisse en désaccord significatif avec la valeur déduite de l'échelle/format papier → la cote explicite prime, mais l'écart est signalé pour que l'architecte le vérifie, plutôt que d'être résolu silencieusement.
- Angle qui semble volontairement différent de 90° sur le dessin mais sans annotation en degrés → traité comme 90° par défaut (règle de redressement), avec un signalement que l'intention pourrait être différente, pour que l'architecte ajoute la cotation d'angle si besoin.
- Épaisseur de mur non cotée → appliquer la règle de priorité ci-dessus (Besoins exacts) et signaler l'hypothèse faite.
- Cote qui ne se recoupe pas avec les autres → vérifier d'abord si c'est une cote **en entraxe** (axe de mur à axe de mur) plutôt qu'une cote libre ; dans les deux cas, signaler (mélange de conventions, ou vraie incohérence à faire trancher par l'architecte).
- Texte ou marque hors du bâtiment sans rapport avec le plan (filigrane, signature, logo) → non reproduit dans le DXF, mentionné dans le compte rendu.
- Élément dessiné dont la nature est incertaine (trait de couleur inhabituelle, lignes extérieures, escalier sans sens de montée) → reproduit **tel que dessiné**, avec une note d'interprétation, plutôt qu'omis ou réinterprété.
- Total écrit sur le dessin en désaccord avec les éléments dessinés (ex. « AFORO 505 » pour 389 fauteuils dessinés) → reproduire le dessin, jamais l'étiquette, et signaler l'écart.
- Plan sans aucune cote ni échelle → échelle estimée sur éléments normalisés (voir Calage dimensionnel), signalée comme estimation avec son incertitude.
- Élément attendu mais absent du dessin (ex. local sans porte dessinée) ou trop pâle pour être lu → non inventé, signalé.

## Contraintes non-fonctionnelles
- Le DXF doit être ouvrable et exploitable directement dans AutoCAD, sans retouche de structure de fichier nécessaire avant ouverture.
- Les murs sont toujours représentés **pochés** (remplissage plein entre les deux lignes), quelle que soit leur épaisseur — une double ligne non remplie n'est pas assez lisible à l'échelle d'un plan complet.
- Le skill part d'un **gabarit DXF réutilisable** (calques et style de cote pré-configurés avec la bonne échelle d'affichage) plutôt que de redéfinir ces conventions à chaque génération.
- **Avant de livrer un DXF**, le skill vérifie que chaque catégorie attendue (murs extérieurs, murs intérieurs, portes, fenêtres, aménagement, cotes) contient bien au moins une entité générée, et signale explicitement toute catégorie vide plutôt que de livrer un résultat silencieusement incomplet.
- **Avant de livrer un DXF**, le skill **superpose le DXF sur l'esquisse source à la même échelle** et vérifie que murs, ouvertures et éléments tombent sur les traits d'origine — un rendu du DXF seul ne suffit pas (il peut paraître plausible tout en étant faux, cf. erreurs d'escalier/hall du test « majlis »). Il compare aussi l'**inventaire généré à l'inventaire relevé** (pièces, portes, fenêtres).
- Murs assemblés par **union géométrique** : jonctions en T et en L propres, sans traits parasites.
- Textes multilignes en MTEXT ; **aucun chevauchement** de textes ou de cotes (vérifié sur le rendu) ; textes en arabe sur un **calque séparé** (`TEXTE_AR`), leur affichage de droite à gauche étant à vérifier dans AutoCAD.
- Le DXF reprend les **cotes de l'esquisse** (calque `COTES`) et des cotes hors-tout ; les noms de pièces non portés sur l'esquisse mais déduits de l'usage sont marqués d'un astérisque.
- Les éléments répétitifs (fauteuils, chaises, WC, lavabos, plantes...) sont des **blocs** nommés, comptables et modifiables en une fois dans AutoCAD ; un texte posé sur des éléments dessinés reçoit un masque de fond.
- Le DXF passe l'audit ezdxf sans erreur, et il est **régénérable depuis l'image par un script** (pas de retouche manuelle non tracée).
- **Passerelle vers le skill de métré (`avant-metre-cao`)** : à chaque plan, le skill livre en plus du DXF de travail un **DXF « charte »** conforme à `bibliotheque-elements.json` : calques `A-MUR-AXE` (axes en polylignes), `A-PIECE` (pièces fermées nommées), `A-DALLE`, `A-MEN-PORTE`, `A-MEN-FENETRE`, `A-SANITAIRE`, `A-MOB`, `P-VEG-ARBRE`, `X-` pour tout ce qui est exclu du métré ; blocs `CATEGORIE_TYPE_LARGEURxHAUTEUR` aux codes canoniques ; attributs `TYPE`, `LARGEUR` et `HAUTEUR` **en cm**, `MATERIAU`, `LOT`, `REPERE` sur menuiseries et sanitaires. Critère de réussite : les scripts du skill de métré (reconnaissance, nomenclature, extraction, interférences) tournent sur ce DXF sans retouche, avec 100 % des calques reconnus et au plus les blocs d'habillage non identifiés. Un pré-métré autonome (JSON + Excel) reste fourni comme pré-contrôle.
- Vit dans le repo `architecte-workflow-skills`, comme un nouveau skill à côté de `lecture-reglement-concours`.

## Définition de « terminé »
- [ ] Le skill est livré dans `architecte-workflow-skills` (structure Claude Code skill standard).
- [ ] Testé par l'utilisateur avec une vraie esquisse scannée qu'il fournira (plan, coupe, ou façade).
- [ ] Le DXF produit respecte les calques et épaisseurs de trait convenus, s'ouvre correctement, et la géométrie est redressée/nettoyée (angles à 90° sauf cotation explicite contraire ; biais francs et courbes conservés si l'affinage proposé est validé).
- [ ] Les ambiguïtés/illisibilités et écarts de cotation sont signalés plutôt que devinés silencieusement.
- [ ] Les contrôles automatiques des règles de qualité de dessin (langue, murs, portes, circulation, sanitaires, escaliers) passent tous.
- [ ] Le fichier est enregistré dans `dxf/` du dossier du concours indiqué, avec le nom donné par l'utilisateur.
