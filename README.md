# Analyse de données

## Projet IPython visant à représenter visuellement les données publiées sur [Data Gouv](https://www.data.gouv.fr/fr/datasets/achats-des-principales-communes-francaises-en-2019/) afin de les rendre lisibles facilement. Cas pratique: achats de la commune de Strasbourg en 2019.

## Organisation en 3 axes principaux:

1. ### Mise en place d'un ****environnement virtuel Python**** 
   - On choisit pour ce propos un environmement ****Anaconda**** comportant les libraires et modules adéquats pour le ****traitement, nettoyage et la représentation visuelle**** des données

2. ### Traitement des données
   - Création des objets ****Pandas DataFrame****.
   - Nettoyage des files portant des cases majoritairement vides.
   - **Groupement par colonnes** ou _'groupby'_
   - ajout de colonnes et d'autres éléments supplémentaires pour avoir une table plus exploitable.

3. ### ****Représentation visuelle****
   - avec des graphiques traditionnels grace aux outils ****Matplotlib****

## Attentes: 
   - Pouvoir tirer des information utiles afin de pouvoir interpréter certains aspects des achats de la commune.
   - Rendre l'analyse facilement réplicable

---

# 1. Environnement Virtuel

- Installation [Anaconda](https://www.anaconda.com/products/individual). Guide d'installation figure dans le site ci contre

- Une fois Anaconda installé:
  - Sous Linux / BSD / MacOS, ouvrir un terminal et saisir:
    - ```
        $ conda create --name nom_environnement
        $ conda activate nom_environnement
        ```
  - Installation des paquets nécessaires
    - ```
        $ conda install pandas matplotlib 
        ```
        (Selon la distribution conda choisie, ces paquets peuvent etre installées par défaut)

  - Optionnel: installation de **Jupyter Notebooks**.
  On choisit Jupyter Notebooks pour sa facilité d'utilisation et pour l'aspect interactif ou éxécution par blocs de code.
    - ```
        $ conda install -c conda-forge jupyterlab 
        ```

### Une fois notre environnement créé et activé, ouvrir un Notebook Jupyter ou un editeur de texte a choix

---

# 2. Traitement des données

Pour faciliter la démarche, il convient de télécharger le fichier avec les données sous format .csv et de les sauvegarder dans le meme dossier ou on a crée notre fichier de code source.


    