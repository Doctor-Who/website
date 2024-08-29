---
title: ESRI et FME
subtitle: "Série : De la tolérance en SIG - chapitre 7"
authors:
    - Loïc Bartoletti
categories:
    - article
comments: true
date: 2024-08-29
description: "Septième et avant-dernière partie du tour d'horizon des SIG sur les dessous des calculs géométriques : un petit tour chez ESRI et FME"
icon: material/vector-polygon-variant
image: https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/geometrie_tolerance_sig/splash_serie_geometrie_07_esri_fme.png
license: beerware
robots: index, follow
tags:
    - analyse
    - ArcGIS
    - ESRI
    - FME
    - géométrie
---

# Et chez la concurrence, ça se passe comment ?

On ne va pas tous les faire, mais seulement deux un peu connus et installés parfois à côté de QGIS :wink:

Le premier sera FME, une sorte de boîte à outils de QGIS et l'autre ArcGis Pro, le concurrent payant de GRASS/QGIS.

![Série d'été 2024 de Loïc Bartoletti - Les Géométries et les SIG : ESRI & FME - Crédits : Sylvain Beorchia](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/geometrie_tolerance_sig/splash_serie_geometrie_07_esri_fme.png){: .img-center loading=lazy }

## FME

![logo FME](https://cdn.geotribu.fr/img/logos-icones/logiciels_librairies/FME.png){: .img-thumbnail-left }

Pour FME, pas de blabla. J'insère les WKB, je fais une test d'intersection et je regarde si les points intersectent `line` et `base`.

Vous pouvez trouver le fichier [fmw sur mon github](https://github.com/lbartoletti/lbartoletti.github.io/blob/master/assets/2024_intersection_intersects/data/fme_test_intersects.fmw)

Et le résultat :

![FME test intersects](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/geometrie_tolerance_sig/fme_test_intersects.png){: .img-center loading=lazy }

KO !

FME utilise, et contribue, aux outils open source. Néanmoins, même si le résultat est le même qu'avec GEOS, ce n'est pas cette bibliothèque qui est utilisée, mais une de leur conception. Encore une fois, le problème n'est donc pas GEOS.

## ESRI ArcGis Pro

![logo ArcGis Pro](https://cdn.geotribu.fr/img/logos-icones/logiciels_librairies/arcgis_pro.png){: .img-thumbnail-left }

Comme pour QGIS, nous allons tester notre problème de deux façons : par les traitements via une couche SIG et directement avec le WKB.

### Utilisation du ShapeFile

Sauf erreur de ma part, ArcGis ne sait pas ouvrir les fichiers GeoPackage. Qu'importe, nous utiliserons le bon vieux ShapeFile qui sera importé dans une GeoDatabase.

Afin de réaliser le calcul de l'intersection, nous utilisons l'outil [Pairwise Intersect](https://pro.arcgis.com/en/pro-app/latest/tool-reference/analysis/pairwise-intersect.htm).

Contrairement à ce que j'ai pu faire pour QGIS, je ne montre pas les formulaires graphiques, mais le code qu'exécute ArcGis.

En entrée `in_features`, on donne nos deux couches `line` et `base`. On sait que l'on va avoir des points, donc on déclare le type de sortie comme `POINT`.

```python
arcpy.analysis.PairwiseIntersect(
    in_features="line;base",
    out_feature_class=r"C:\Users\xxx\AppData\Local\Temp\ArcGISProTemp37912\Sans titre\Default.gdb\line_PairwiseIntersect",
    join_attributes="ALL",
    cluster_tolerance=None,
    output_type="POINT"
)
```

Je passe les étapes pour l'extraction du WKB et WKT, dont voici les résultats :

- `0104000000020000000101000000e034efc8c83c3e4120166a8166d55341010100000040a4df9e8f3c3e416054525379d55341`
- `MultiPoint ((1981640.78490000218153 5199258.02210000157356262),(1981583.62060000002384186 5199333.30189999938011169))`

ArcGis nous sort un résultat légèrement différent. Testons avec l'autre outil pour les intersections : [Intersect](https://pro.arcgis.com/en/pro-app/latest/tool-reference/analysis/intersect.htm)

```python
arcpy.analysis.Intersect(
    in_features="line #;base #",
    out_feature_class=r"C:\Users\xxx\AppData\Local\Temp\ArcGISProTemp37912\Sans titre\Default.gdb\line_Intersect1",
    join_attributes="ALL",
    cluster_tolerance=None,
    output_type="POINT"
)
```

- `01040000000200000001010000008016d99e8f3c3e416054525379d553410101000000e034efc8c83c3e4120166a8166d55341`
- `MultiPoint ((1981583.62049999833106995 5199333.30189999938011169),(1981640.78490000218153 5199258.02210000157356262))`

Un résultat également légèrement différent, mais proche de celui que nous obtenons avec les autres SIG.
Que se passe-t-il ici ?

ArcGis utilise, pour tous les calculs, une notion que l'on retrouve parfois dans les SIG OpenSource, celui de résolution et tolérance.
On peut la modifier en passant en paramètres des valeurs XY. Voici les requêtes et les résultats avec une valeur de 0.00001 mm.

```python
with arcpy.EnvManager(XYResolution="0.00001 Millimeters", XYTolerance="0.00001 Millimeters"):
    arcpy.analysis.PairwiseIntersect(
        in_features="line;base",
        out_feature_class=r"C:\Users\xxx\AppData\Local\Temp\ArcGISProTemp37912\Sans titre\Default.gdb\line_PairwiseIntersect1",
        join_attributes="ALL",
        cluster_tolerance=None,
        output_type="POINT"
    )
```

- `MultiPoint ((1981640.78490600734949112 5199258.02208840474486351),(1981583.6205737441778183 5199333.30187807604670525))`
- `0104000000020000000101000000a099efc8c83c3e417ce5698166d553410101000000c0ebdd9e8f3c3e416cf8515379d55341`

Ah, on retrouve nos petits ! Du moins, l'écart a été réduit.

L'équivalent de notre sélection par localisation se fait comme suit :

```python
arcpy.management.SelectLayerByLocation(
    in_layer="line_PairwiseIntersect1;pariwiseIntersect_tolerance;lineIntersect",
    overlap_type="INTERSECT",
    select_features="line",
    search_distance=None,
    selection_type="NEW_SELECTION",
    invert_spatial_relationship="NOT_INVERT"
)
```

Avec une distance de recherche (tolérance)

```python
arcpy.management.SelectLayerByLocation(
    in_layer="line_PairwiseIntersect1;pariwiseIntersect_tolerance;lineIntersect",
    overlap_type="INTERSECT",
    select_features="line",
    search_distance="0.000000001 Millimeters",
    selection_type="NEW_SELECTION",
    invert_spatial_relationship="NOT_INVERT"
)
```

Dans les deux cas, ArcGis sélectionne les points d'intersection. C'est donc un bon point pour eux.

### Via le WKB et ArcPy

Dans la partie suivante, nous allons regarder comment cela se passe, en utilisant directement les fonctions de base.

```python
import binascii
# on importe le WKB de base
base = arcpy.FromWKB(binascii.unhexlify('0102000000050000007997c6b68d3c3e4139eb62c260d55341ac9ea7316a3c3e41cbeb40e073d55341403e0bfbc33c3e41b3fc06f380d55341387a2a800c3d3e41f256b8176dd553417997c6b68d3c3e4139eb62c260d55341'))
# on importe le WKB de line
line = arcpy.FromWKB(binascii.unhexlify('010200000002000000ea9c6d2b873c3e41a03d941b7cd5534133db7796ce3c3e413fba569864d55341'))
# calcul du point d'intersection en WKT
base.intersect(line, 1).WKT
# 'MULTIPOINT ((1981583.6207275391 5199333.3018798828), (1981640.7850952148 5199258.0220947266))'
# en WKB
base.intersect(line, 1).WKB
# bytearray(b'\x01\x04\x00\x00\x00\x02\x00\x00\x00\x01\x01\x00\x00\x00\x00\x00\xe8\x9e\x8f<>A\x00\x00RSy\xd5SA\x01\x01\x00\x00\x00\x00\x00\xfc\xc8\xc8<>A\x00\x00j\x81f\xd5SA')
# avec la conversion pour l'afficher en hexa
binascii.hexlify(base.intersect(line, 1).WKB)
# b'01040000000200000001010000000000e89e8f3c3e410000525379d5534101010000000000fcc8c83c3e4100006a8166d55341'
```

Maintenant, regardons les relations spatiales entre notre résultat `result` et les [géométries](https://pro.arcgis.com/en/pro-app/latest/arcpy/classes/geometry.htm) `base` et `line`.
On utilisera : disjoint, contains, crosses, equals, overlaps, touches et whitin ; ce dernier étant notre intersects.

Base :

```python
result.disjoint(base)
# False
```

```python
result.contains(base), result.crosses(base), result.equals(base), result.overlaps(base), result.touches(base), result.within(base)
# (False, False, False, False, False, True)
```

Line :

```python
result.disjoint(line)
# False
```

```python
result.contains(line), result.crosses(line), result.equals(line), result.overlaps(line), result.touches(line), result.within(line)
# (False, False, False, False, False, True)
```

On obtient bien le résultat souhaité. En fait, vous l'aurez peut-être compris en filigrane, ArcGis ne propose pas un calcul "strict" comme les autres, mais bien quelque chose de particulier. Il est "tolérant".

<!-- geotribu:authors-block -->

{% include "licenses/beerware.md" %}

<!-- Notes de bas de page -->
