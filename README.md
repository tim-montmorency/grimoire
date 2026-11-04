# grimoire — socle commun des cours TIM

Ce répertoire ne contient **aucun contenu de cours**. Il fournit à tous les cours de l'organisation :

| Élément | Rôle |
|---|---|
| `base.yml` | Configuration commune (thème, extensions Markdown, fonctionnalités). Importée par chaque cours avec `INHERIT: _grimoire/base.yml`. |
| `requirements.txt` | Version de Zensical utilisée partout. |
| `.github/workflows/build.yml` | Workflow réutilisable qui construit et publie un cours sur GitHub Pages. |

## Fonctionnement

Lors du build d'un cours, GitHub Actions :

1. récupère le répertoire du cours ;
2. récupère ce répertoire `grimoire` dans le sous-dossier `_grimoire/` ;
3. installe la version de Zensical indiquée dans `requirements.txt` ;
4. lance `zensical build` puis publie le dossier `site/` sur GitHub Pages.

Dans le `mkdocs.yml` d'un cours, tous les chemins vers le grimoire commencent donc par `_grimoire/`.

## Brancher un cours

Dans les réglages du répertoire du cours, sous **Settings → Pages → Source**, choisir **GitHub Actions**.

Ajouter ensuite ces deux fichiers au cours.

`mkdocs.yml` :

```yaml
INHERIT: _grimoire/base.yml
site_name: Nom du cours
site_url: https://tim-montmorency.com/nom-du-depot/
```

`site_url` est à définir dans chaque cours : sans lui, `sitemap.xml` est vide.

`.github/workflows/publier.yml` :

```yaml
name: Publier le cours

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  publier:
    uses: tim-montmorency/grimoire/.github/workflows/build.yml@main
    permissions:
      contents: read
      pages: write
      id-token: write
```

Les permissions sont obligatoires : sans elles, la publication sur GitHub Pages échoue.

## Prévisualiser un cours localement

Zensical demande Python 3.11 ou plus récent. Depuis le répertoire du cours, avec une copie du grimoire dans `_grimoire/` :

```bash
python3 -m venv .venv
.venv/bin/pip install -r _grimoire/requirements.txt
.venv/bin/zensical serve
```

## Règles de fusion de la configuration

Vérifiées avec Zensical 0.0.74 :

- **dictionnaires** (`theme`, `extra`…) : fusionnés, la valeur du cours l'emporte ;
- **listes** (`extra_css`, `markdown_extensions`, `theme.features`…) : **concaténées** — un cours ajoute des éléments, il n'en retire pas.

Exception utile : un cours peut changer les options d'une extension de `base.yml` en la déclarant de nouveau avec ses propres options, par exemple `toc`.

## Modifier le grimoire sans casser les cours

Pendant la preuve de concept, les cours suivent la branche `main`. Pour la production, il est recommandé de créer des étiquettes (`v1`, `v2`…) et de faire pointer chaque cours sur une étiquette :

```yaml
jobs:
  publier:
    uses: tim-montmorency/grimoire/.github/workflows/build.yml@v1
    with:
      grimoire-ref: v1
    permissions:
      contents: read
      pages: write
      id-token: write
```

Une modification du grimoire ne touche alors aucun cours tant que celui-ci n'a pas été basculé sur la nouvelle étiquette.

Les deux valeurs doivent toujours être changées ensemble : `@v1` choisit la version du workflow, et `grimoire-ref` celle de la configuration. Sans `grimoire-ref`, le cours utilise la configuration de `main`.
