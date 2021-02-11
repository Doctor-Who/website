---
title: "Gérer facilement l'aide de son plugin QGIS"
authors: ["Julien MOURA"]
categories: ["article", "tutoriel"]
date: "2021-03-02 10:20"
description: "XXXXXXXXXXXXXXXXXX"
image: "https://cdn.geotribu.fr/img/tuto/qgis_plugin_show_help/documentation_le-chat.jpg"
tags: "geotribu,QGIS,plugin,documentation,help"
---

# Utiliser les icônes intégrées de QGIS pour égayer ses plugins

:calendar: Date de publication initiale : 02 mars 2021

**Mots-clés :** QGIS | PyQGIS | développement | plugin | documentation

Pré-requis :

- des notions en PyQGIS (Python + Qt + API QGIS)

## Introduction

![icône PyQT](https://cdn.geotribu.fr/img/logos-icones/programmation/python_and_qt.svg "Python + Qt = PyQt"){: .img-rdp-news-thumb }

Plutôt que de stocker toute la documentation au format HTML et les fichiers associés (CSS, JS, images...) dans le plugin QGIS qui est téléchargé par vos millions d'utilisateur/ices, je vous propose une méthode plus simple que j'ai utilisée dans différents plugins comme [GMLAS Toolbox], [Menu Layers from Project] ou [Land Survey Codes Import].

Si vous n'avez pas d'aide en ligne, de documentation, de wiki sur le dépôt GitHub ou GitLab ou même de README, alors je ne peux rien pour vous et il est temps d'aller consulter un autre article du site :wink:.

![Documentation Le Chat](https://cdn.geotribu.fr/img/tuto/qgis_plugin_show_help/documentation_le-chat.jpg "Documentation Le Chat"){: loading=lazy }
{: align=middle }

Pour la suite de ce tutoriel, considérons que l'aide en ligne de notre plugin est la fiche métier "M1808 Information Géographique" du registre ROME[^1] :

```txt
https://candidat.pole-emploi.fr/marche-du-travail/fichemetierrome?codeRome=M1808
```

## Créer la redirection 301

![icône HTML5](https://cdn.geotribu.fr/img/logos-icones/programmation/html5.png "icône HTML5"){: .img-rdp-news-thumb }

A l'intérieur de l'arborescence de notre plugin, on crée un fichier `docs/index.html` dans lequel on écrit simplement une [redirection HTTP](https://developer.mozilla.org/fr/docs/Web/HTTP/Status/301) :

```htmlmixed
<!doctype html>
<html lang="fr">

<head>
    <meta charset="utf-8">
    <title>Redirecting...</title>
    <script>var anchor = window.location.hash.substr(1); location.href = "https://candidat.pole-emploi.fr/marche-du-travail/fichemetierrome?codeRome=M1808/"</script>
</head>

<body>

    <p>Redirection to the online documentation...</p>

</body>

</html>
```

----

## Appeler la page depuis le plugin

![icône Edit Help Content - QGIS](https://raw.githubusercontent.com/qgis/QGIS/master/images/themes/default/mActionEditHelpContent.svg "icône Edit Help Content - QGIS"){: .img-rdp-news-thumb }

Une fois que tout cela est en place, il reste à brancher notre menu d'aide sur cette redirection. Pour cela, on utilise [la fonction showPluginHelp](https://github.com/qgis/QGIS/blob/ltr-3_16/python/utils.py#L502-L530).

Quelque part dans les imports :

```python
from qgis.utils import showPluginHelp
```

Quelque part dans la définition du [menu défini dans un précédent tuto](/articles/2021/2021-01-19_pyqgis_utiliser_icones_integrees/#recycler-les-icones-de-linterface-de-qgis) :


```python hl_lines="17"
# imports
from qgis.core import QgsApplication
from qgis.PyQt.QtGui import QIcon

[...]

class PluginGeotribu:

    [...]

    # quelque part dans la fonction de définition du menu de notre plugin
    def initGui(self):

        [...]

        # ici on définit notre bouton de menu
        self.action_menu_help = QAction(
            QIcon(QgsApplication.iconPath("mActionHelpContents.svg")),
            self.tr("Help") + "...",
            self.iface.mainWindow(),
        )

        # on ajoute le bouton au menu de notre plugin
        self.iface.addPluginToMenu("Le nom du plugin", self.action_menu_help)

        # on connecte notre bouton à la fonction d'ouverture de l'aide
        self.action_menu_help.triggered.connect(
                lambda: showPluginHelp(filename="doc/index")
            )

```


----

## Conclusion

Ce que j'aime bien dans cette façon de faire c'est que ça tient en un seul fichier HTML et 2 lignes de Python.  
C'est pas cher payé pour orienter facilement vos utilisateur/ices vers la documentation que vous avez mis tant de soin à rédiger (et qu'ils/elles prendront bien soin de survoler) !

----

## Auteur

--8<-- "content/team/jmou.md"

<!-- Footnotes reference -->
[^1]: [**R**épertoire **O**pérationnel des **M**étiers et des **E**mplois](https://fr.wikipedia.org/wiki/R%C3%A9pertoire_op%C3%A9rationnel_des_m%C3%A9tiers_et_des_emplois). En savoir plus sur l'historique de création de cette fiche pour le domaine de la géomatique, voir [la démarche métier sur GeoRezo](https://georezo.net/wiki/main:formetiers:dem_metiers/).

<!-- hyperlinks reference -->
[GMLAS Toolbox]: https://plugins.qgis.org/plugins/gml_application_schema_toolbox/
[Menu Layers from Project]: https://plugins.qgis.org/plugins/menu_from_project/
[Land Survey Codes Import]: https://plugins.qgis.org/plugins/LandSurveyCodesImport/