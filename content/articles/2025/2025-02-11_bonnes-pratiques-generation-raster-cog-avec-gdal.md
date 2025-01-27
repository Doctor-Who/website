---
title: Optimiser vos rasters et générer des mosaïques au format COG avec GDAL
subtitle: Des images grille en-tête
authors:
  - Nicolas ROCHARD
categories:
  - article
comments: true
date: 2025-02-11
description: Découvrez comment optimiser vos rasters et créer des mosaïques au format COG avec GDAL pour une gestion efficace des données raster géospatiales.
icon: material/grid
image:
license: default
robots: index, follow
tags:
    - COG
    - GDAL
    - raster
---

<!-- markdownlint-disable-file MD046 -->

# Optimiser vos rasters et générer des mosaïques au format COG (Cloud Optimized GeoTIFF) avec GDAL

:calendar: Date de publication initiale : {{ page.meta.date | date_localized }}

## Introduction

Les données Raster sont une composante majeure des référentiels de nos systèmes d'information géographique. Ces fichiers sont bien plus volumineux que des données vectorielles et sont parfois fragmenté en plusieurs dalles rendant son chargement laborieux. Lorsque ces données sont disponibles en flux WMS ou WMTS, alors leur consultation est plus aisée mais présente des limitations (pas de possibilité d'affiner la radiométrie, lenteur d'affichage, problème d'impression, etc.). Il est alors indispensable d'avoir une donnée en locale ou sur le réseau de la structure.

![logo COG](https://cdn.geotribu.fr/img/logos-icones/logiciels_librairies/COG.png "logo COG"){: .img-thumbnail-left }

C'est là que le format COG (_Cloud Optimized GeoTIFF_) intervient pour simplifier la vie des géomaticiens. Ces fichiers, optimisés pour le cloud, facilitent le traitement et la visualisation des données spatiales à grande échelle grâce à leur accessibilité rapide et leur structure efficiente. Conçus spécifiquement pour le cloud, les COG offrent de nombreux avantages sur d'autres environnements :

- **Performance améliorée** : affichage quasi-immédiat même sur des volumes importants évitant ainsi les frustrations liées aux lenteurs, que ce soit au bureau ou en télétravail.
- **Données peu ou pas alterées**_(en fonction des options de compression choisies)_ : vous pouvez modifier la radiométrie, l'ordre des bandes, l'utiliser dans des processus de geotraitements, etc.
- **Simplicité d'organisation** : une seule image à charger, éliminant le besoin de VRT peu performant, la génération de pyramides et réduisant la gestion de nombreux fichiers

![vador_command_line](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2025/raster_cog_gdal/command_line.jpg){: .img-center loading=lazy }

Dans cet article, nous aborderons les meilleures pratiques pour générer des COG avec GDAL, un outil essentiel des SIG. Certaines options ne soit pas encore disponible dans QGIS, nous utiliserons donc la ligne de commande. Pas d'inquiétude : il s'agit toujours de la même base avec quelques variations, et même sans être un expert, vous vous en sortirez très bien (j'ai pu tester sur mes collègues et ils ont survécut).

[Commenter cet article :fontawesome-solid-comments:](#__comments "Aller aux commentaires"){: .md-button }
{: align=middle }

----

## Pré-requis

Avant de commencer la génération de COG, assurez-vous de disposer des éléments suivants :

- **GDAL Version 3.1 ou supérieure** : Vérifiez que votre installation de GDAL est à jour pour bénéficier des dernières améliorations et fonctionnalités spécifiques aux COG.
- **Types de Raster appropriés** : Pour les données raster à une bande (comme les Modèles Numériques de Terrain - MNT ou d’Élévation - MNE), utilisez des fichiers au format TIF, ASC ou tout autre format compatible avec GDAL. Pour les rasters à trois bandes, les orthophotos sont particulièrement adaptées.
- **Environnement Linux ou Windows** : Les commandes abordées ici ont été testées sur ces deux systèmes d’exploitation.

## Construction du VRT pour un raster à 1 bande

Pour combiner plusieurs fichiers raster ASC en un VRT (Virtual Raster Tile), une étape nécessaire avant de générer le COG final, utilisez la commande suivante :

=== ":penguin: Linux"

    ```bash
    gdalbuildvrt my_dsm.vrt -addalpha -a_srs EPSG:2154 /dsm_directory/*.asc
    ```

=== "🪟 Windows"

    ```cmd
    gdalbuildvrt.exe C:\dsm\my_dsm.vrt C:\dsm_directory\*.asc -addalpha -a_srs EPSG:2154
    ```
Détail des options :

- `-addalpha` : ajoute un canal alpha.
- `-a_srs EPSG:2154` : définit le système de référence spatiale à utiliser.

## Conversion en COG pour un raster à 1 bande

Une fois le VRT construit, transformez-le en COG avec cette commande :

=== ":penguin: Linux"

    ```bash
    gdal_translate input_dsm.vrt my_dsm_output_cog.tif -of COG \
    -co RESAMPLING=NEAREST \
    -co OVERVIEW_RESAMPLING=NEAREST \
    -co COMPRESS=DEFLATE \
    -co PREDICTOR=2 \
    -co NUM_THREADS=20 \
    -co BIGTIFF=IF_NEEDED
    ```

=== "🪟 Windows"

    ```cmd
    gdal_translate.exe C:\dsm\input_dsm.vrt C:\dsm\my_dsm_output_cog.tif -of COG ^
    -co BLOCKSIZE=512 ^
    -co OVERVIEW_RESAMPLING=NEAREST ^
    -co COMPRESS=DEFLATE ^
    -co PREDICTOR=2 ^
    -co NUM_THREADS=20 ^
    -co BIGTIFF=IF_NEEDED
    ```

Aperçu du RGE ALTI® 1m à l'échelle des Hauts-de-France
![rgealti_2017_hdf](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2025/raster_cog_gdal/bdalti_1m.png){: .img-center loading=lazy }

### Points clés

- **Resampling** : Utilisez `RESAMPLING=NEAREST` pour préserver l'intégrité des données.
- **Optimisation des performances** : Ajustez `NUM_THREADS` en fonction de la capacité de votre machine.

### Volumétrie

Mon COG sera plus léger que les données téléchargées, sauf si celles-ci sont déjà compressées. L'objectif est de préserver la donnée source tout en optimisant les performances d'affichage et de traitement. Voici quelques exemples de données assemblées et converties en COG pour la Région Hauts-de-France :

| Données | Format source | Poids brut | Poids en COG |
| --------|--------------|-----|-----|
| [MNS correlés](https://geoservices.ign.fr/modeles-numeriques-de-surfaces-correles) | TIF (compression LZW) | 241.4 Go | 235.5 Go |
| [RGE ALTI®](https://geoservices.ign.fr/rgealti) | ASC | 206.3 Go | 48.2 Go |

## Processus pour un raster à 3 bandes

### Conversion de JP2 en TIF

Commencez par convertir chaque fichier JP2 en TIF en utilisant une boucle bash. Assurez-vous d'avoir créé un répertoire dédié pour les fichiers TIF :

=== ":penguin: Linux"

    ```bash
    for f in *.jp2; do
    gdal_translate -of GTiff \
        -co TILED=YES \
        -co BIGTIFF=YES \
        -co BLOCKXSIZE=512 \
        -co BLOCKYSIZE=512 \
        -co NUM_THREADS=20 \
        -co COMPRESS=ZSTD \
        -co PREDICTOR=2 \
        ${f} ../0_TIF/${f%.*}.tif
    done
    ```

=== "🪟 Windows"

    ```cmd
    FOR %%F IN (C:\ortho\jpg2\*.jp2) DO ^
    gdal_translate.exe -of GTiff ^
    -co TILED=YES ^
    -co BIGTIFF=YES ^
    -co BLOCKXSIZE=512 ^
    -co BLOCKYSIZE=512 ^
    -co NUM_THREADS=20 ^
    -co COMPRESS=ZSTD ^
    -co PREDICTOR=2 ^
    %%F C:\ortho\0_TIF\%%~nxF.tif
    ```

- **Taille des blocs** : `BLOCKXSIZE` et `BLOCKYSIZE` impactent les performances de lecture.

### Construction du VRT pour un raster à 3 bandes

Créez un VRT pour votre ensemble de données TIFF avec la commande suivante :

=== ":penguin: Linux"

    ```bash
    gdalbuildvrt my_orthophotography.vrt 0_TIF/*.tif -addalpha -hidenodata -a_srs EPSG:2154
    ```

=== "🪟 Windows"

    ```cmd
    gdalbuildvrt.exe C:\ortho\my_orthophotography.vrt C:\ortho\0_TIF\*.tif -addalpha -hidenodata -a_srs EPSG:2154
    ```

- `-hidenodata` : Masque les cellules nodata, rendant les zones correspondantes transparentes.

### Conversion du VRT en COG

Générez le COG à partir du VRT :

=== ":penguin: Linux"

    ```bash
    gdal_translate my_orthophotography.vrt my_orthophotography_output_cog.tif -of COG \
    -co BLOCKSIZE=512 \
    -co OVERVIEW_RESAMPLING=BILINEAR \
    -co COMPRESS=JPEG \
    -co QUALITY=85 \
    -co NUM_THREADS=12 \
    -co BIGTIFF=YES
    ```

=== "🪟 Windows"

    ```cmd
    gdal_translate.exe C:\ortho\my_orthophotography.vrt C:\ortho\my_orthophotography_output_cog.tif -of COG ^
    -co BLOCKSIZE=512 ^
    -co OVERVIEW_RESAMPLING=BILINEAR ^
    -co COMPRESS=JPEG ^
    -co QUALITY=85 ^
    -co NUM_THREADS=12 ^
    -co BIGTIFF=YES
    ```

- **Compression JPEG** : Un bon compromis entre taille de fichier et qualité avec une `QUALITY` de 85.
- **Rééchantillonnage** : `BILINEAR` pour un rendu visuel optimal dans les visualisations géospatiales.

Aperçu de la BD ORTHO® 2021 à l'échelle des Hauts-de-France
![bdortho_2021_hdf](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2025/raster_cog_gdal/bdortho_20cm.png){: .img-center loading=lazy }

### Volumétrie

Le format JP2 offre une compression efficace mais requiert un codec propriétaire, limitant son utilisation. Une licence est nécessaire pour des applications serveur. La conversion en COG améliore les performances et la polyvalence d'utilisation (SIG desktop, flux, traitements d'images, etc.), bien qu'au détriment de la taille.
La compression JPEG, bien que destructrice, convient parfaitement aux orthophotos. La qualité visuelle reste suffisante, même pour l'impression ou le traitement d'image.

| Données | Format source | Poids brut | Poids en COG |
| --------|--------------|--------|--------|
| [BD ORTHO®](https://geoservices.ign.fr/bdortho) | JP2 (format compressé) | 30.7 Go | 60.5 Go |
| [PCRS Raster](https://www.geo2france.fr/datahub/dataset/94a69703-572f-463a-9cfc-6bca075384b8) | TIF (ZSTD) | 20.1 To | 3.1 Go |

## Cas particuliers et bonnes pratiques

### Découpe selon un contour

Pour éliminer les pixels indésirables en bordure (non définis comme nodata), utilisez un shapefile d'emprise :

=== ":penguin: Linux"

    ```bash
    gdalwarp -of GTiff \
    -co TILED=YES \
    -co BIGTIFF=YES \
    -co BLOCKXSIZE=512 \
    -co BLOCKYSIZE=512 \
    -co NUM_THREADS=12 \
    -co COMPRESS=ZSTD \
    -co PREDICTOR=2 \
    -s_srs EPSG:2154 \
    -t_srs EPSG:2154 \
    -dstalpha \
    -cutline area_of_interest.shp \
    input_image.jp2 \
    image_output.tif
    ```

=== "🪟 Windows"

    ```cmd
    gdalwarp.exe -of GTiff ^
    -co TILED=YES ^
    -co BIGTIFF=YES ^
    -co BLOCKXSIZE=512 ^
    -co BLOCKYSIZE=512 ^
    -co COMPRESS=ZSTD ^
    -co PREDICTOR=2 ^
    -s_srs EPSG:2154 ^
    -t_srs EPSG:2154 ^
    -dstalpha ^
    -cutline C:\data\area_of_interest.shp ^
    C:\ortho\input_image.jp2 ^
    C:\ortho\image_output.tif
    ```

### Conversion de JP2 en TIFF RVBA

Pour convertir un JP2 en TIFF RVBA tout en préservant l’unité colorimétrique :

=== ":penguin: Linux"

    ```bash
    gdal_translate -of GTiff \
    -co BIGTIFF=YES \
    -co TILED=YES \
    -co BLOCKXSIZE=512 \
    -co BLOCKYSIZE=512 \
    -co NUM_THREADS=12 \
    -co COMPRESS=ZSTD \
    -co PREDICTOR=2 \
    -b 1 -b 2 -b 3 -b mask \
    -colorinterp red,green,blue,alpha \
    -a_srs EPSG:2154 \
    input_image.jp2 \
    output_image.tif
    ```

=== "🪟 Windows"

    ```cmd
    gdal_translate.exe -of GTiff ^
    -co BIGTIFF=YES ^
    -co TILED=YES ^
    -co BLOCKXSIZE=512 ^
    -co BLOCKYSIZE=512 ^
    -co NUM_THREADS=12 ^
    -co COMPRESS=ZSTD ^
    -co PREDICTOR=2 ^
    -b 1 -b 2 -b 3 -b mask ^
    -colorinterp red,green,blue,alpha ^
    -a_srs EPSG:2154 ^
    C:\ortho\input_image.jp2 ^
    C:\ortho\output_image.tif
    ```

## Considérations finales

- **Compression** :
    - Utilisez `JPEG` pour les fichiers RVB (3 bandes).
    - Préférez `DEFLATE` ou `ZSTD` pour les fichiers avec plus de 3 bandes ou en 16 bits.
- **Méthode de Rééchantillonnage** :
    - `BILINEAR` est idéal pour le rendu visuel.
    - `NEAREST` est recommandé pour les traitements analytiques afin de préserver l'intégrité des données.

En suivant ces bonnes pratiques, vous assurerez une génération efficace de COG, améliorant ainsi la manipulation et la visualisation de vos données spatiales quelque soit votre environnement.

Si vous souhaitez apporter votre expertise aux bonnes pratiques et astuces de GDAL et du COG, n'hésitez pas à contribuer à ce dépôt <https://github.com/geo2france/cog-tips>. Merci à [Benjamin Chartier](../../team/benjamin-chartier.md "Profil de Benjamin Chartier") pour avoir proposé les commandes Windows.

## Affichage dans QGIS

1. Si votre fichier est disponible sur le réseau, il se consulte de la même manière qu'une n'importe quel raster.
2. S'il est disponible en flux WMS ou WMTS, c'est transparent pour vous, vous ne saurez pas que vous chargez un COG. NDLR : nous avons remarqué, sur Geo2France, une meilleure performance dans les temps de réponse de flux en utilisant une mosaique COG. Cela évite de devoir générer du cache tuilé consommateur d'espace disque (utile pour la sobriété numérique).
3. Un COG peut être publié via un simple serveur web HTTP(S). Il est disponible via une simple URL. Pour le charger dans QGIS, rendez-vous dans le menu _Raster_ , choisissez _"Protocole HTTP(S), cloud, etc."_ et coller l'URL dans URI.

![menu raster de QGIS](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2025/raster_cog_gdal/raster_qgis.png){: .img-center loading=lazy }

Envie de tester ? Utilisez l'url suivante pour charger l'orthophoto IGN 2021 sur la région Hauts-de-France [IGN_BDOrtho_2021_RVB_COG_Geo2France.tif](http://geo2france.fr/public/cog/ortho/2021_R32_Ortho_0m20_RVB_COG.tif) (~242.6 Go en quelques secondes ^^)

----

<!-- geotribu:authors-block -->

{% include "licenses/default.md" %}
