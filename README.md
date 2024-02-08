# Site web de Geotribu

[![🚀 Déploiement](https://github.com/geotribu/website/actions/workflows/deploy.yml/badge.svg)](https://github.com/geotribu/website/actions/workflows/deploy.yml)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/geotribu/website/master.svg)](https://results.pre-commit.ci/latest/github/geotribu/website/master)
[![Pull Request Checker 🛃](https://github.com/geotribu/website/actions/workflows/pr_checker_build.yml/badge.svg)](https://github.com/geotribu/website/actions/workflows/pr_checker_build.yml)
[![🎳 Markdown Linter](https://github.com/geotribu/website/actions/workflows/pr_linter_markdown.yml/badge.svg)](https://github.com/geotribu/website/actions/workflows/pr_linter_markdown.yml)
[![🎳 Vérification des liens](https://github.com/geotribu/website/actions/workflows/links_checker.yml/badge.svg)](https://github.com/geotribu/website/actions/workflows/links_checker.yml)
[![🤖 Réponse automatique à un ticket de proposition de contenu](https://github.com/geotribu/website/actions/workflows/issue_comment.yml/badge.svg)](https://github.com/geotribu/website/actions/workflows/issue_comment.yml)

[![Ouvrir dans l'éditeur en ligne](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/geotribu/website)

[![Built with Material for MkDocs](https://img.shields.io/badge/Material_for_MkDocs-526CFE?style=for-the-badge&logo=MaterialForMkDocs&logoColor=white)](https://squidfunk.github.io/mkdocs-material/)

Sources et contenus du site de Geotribu, accessible via les URLs suivantes :

- <https://geotribu.fr>
- <http://geotribu.fr>
- <http://geotribu.net>

----

## Démarrage rapide

> **Note**
> Les instructions données ici se veulent succinctes et ne sont donc pas complètes. Pour un vrai guide de contribution en local, consulter le guide détaillé : [Installation et configuration de l'environnement de travail pour l'édition locale](https://contribuer.geotribu.fr/).

### Prérequis

- Python >= 3.9

#### Recommandés

- Visual Studio Code

### Installation

Après avoir cloné ou téléchargé le dépôt, installer les prérequis (de préférence dans un environnement virtuel) :

```bash
python -m pip install -U pip
python -m pip install -U setuptools wheel
```

### Version gratuite

```bash
python -m pip install -U -r requirements-free.txt
```

#### Version Insiders

Pour utiliser la [version Insiders du thème Material for Mkdocs](https://squidfunk.github.io/mkdocs-material/insiders/), il faut disposer du *token* lié au compte GitHub de Geotribu :

```bash
export GH_TOKEN_MATERIAL_INSIDERS=************
python -m pip install -U -r requirements-insiders.txt
```

### Générer le site

Version complète :

```bash
mkdocs build
```

Version complète gratuite :

```bash
mkdocs build -f mkdocs-free.yml
```

Version minimale (seulement certains plugins) :

```bash
mkdocs build -f mkdocs-minimal.yml
```

### Servir le site en local

Version complète :

```bash
mkdocs serve --dirtyreload
```

Version complète gratuite :

```bash
mkdocs serve -f mkdocs-free.yml --dirtyreload
```

Version minimale (seulement certains plugins) :

```bash
mkdocs serve --dirtyreload -f mkdocs-minimal.yml
```

Le site est accessible en local à l'adresse suivante : <http://localhost:8000/>.  
Quand un contenu est modifié, le site est automatiquement rechargé.

----

## Contribuer

Pour la procédure détaillée, consulter [le site dédié](https://contribuer.geotribu.fr/).

## Soutenir

Afin de pérenniser le site, nous avons ouvert un compte sur Liberapay : <https://liberapay.com/Geotribu/>.

![Liberapay receiving](https://img.shields.io/liberapay/receives/Geotribu?color=green&label=re%C3%A7oit&style=flat-square)
![Liberapay patrons](https://img.shields.io/liberapay/patrons/Geotribu?color=blue&label=soutiens&style=flat-square)

L'objectif de ce financement est de :

- financer les outils open-source que l'on utilise pour le site :
    - Material for MkDocs (voir la page sponsor <https://github.com/sponsors/squidfunk>)
    - GeoRezo (pour le CDN)
- financer les suffixes du nom de domaine (geotribu.fr/.net/.org)
