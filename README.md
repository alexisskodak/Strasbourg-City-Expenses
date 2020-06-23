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
  - Installation des librairies nécessaires
    - ```
        $ conda install pandas matplotlib 
        ```
        (Selon la distribution conda choisie, ces librairies peuvent etre installées par défaut)

  - Optionnel: installation de **Jupyter Notebooks**.
  On choisit Jupyter Notebooks pour sa facilité d'utilisation et pour l'aspect interactif ou éxécution par blocs de code.
    - ```
        $ conda install -c conda-forge jupyterlab 
        ```

### Une fois notre environnement créé et activé, ouvrir un Notebook Jupyter ou un editeur de texte a choix

---

# 2. Traitement des données

Pour faciliter la démarche, il convient de télécharger le fichier avec les données sous format .csv et de les sauvegarder dans le meme dossier ou on a crée notre fichier de code source.

- importation des librairies
```Python
import pandas as pd
import matplotlib.pyplot as plt
```
- Creation de notre objet DataFrame principal
  
```Python
sxb = pd.read_csv('sxb.csv', sep=';')

# Supprimer les files ou toutes les cases sont vides
sxb.dropna(how='all', inplace=True) 

# Visualiser les 10 premières files de la table
sxb.head(10)
```
Nous observons une table de la forme suivante

![image](sxbhead.png)

Les colonnes correspondent respectivement:
- au numero de la transaction
- au nom du client (dans ce cas la commune de Strasbourg)
- à l'objet de la transaction
- à la date de réception de facture ou de commande (à vérifier)
- au montant de la commande
- au nom du prestataire de services


## 2.1 Opérations sur les données en fonction des mois

On crée tout d'abord une colonne pour le mois, pour pouvoir représenter graphiquement les achats de la commune par mois

```Python
# Prendre le mois dans la colonne 'DATE' en format 'int'
sxb['MOIS'] = sxb['NOTIFICATION_DATE'].str[5:7].astype(int)

# Convertir le type des cases 'MONTANT' en 'float' lorsque possible
sxb['MONTANT'] = pd.to_numeric(sxb['MONTANT'],errors='coerce')
```
Manipulations sur cette nouvelle colonne

```Python
# Groupement de la table par mois
depenses_par_mois = sxb.groupby('MOIS').sum()

# création d'une colonne pour représenter la fraction du montant total
depenses_par_mois['MONTANT RELATIF'] = depenses_par_mois['MONTANT'] * 100 / total2019

# mettre en indice la colonne relative au mois
depenses_par_mois = depenses_par_mois.reset_index()

# calculer les pourcentages des montants
pourcentages = depenses_par_mois['MONTANT RELATIF']

Mois =  ['Janvier', 'Fevrier', 'Mars', 
         'Avril', 'Mai', 'Juin', 
         'Juillet', 'Aout', 'Septembre', 
         'Octobre', 'Novembre', 'Decembre']
```

## 2.2 Opérations sur les données en fonction des entreprises

```Python
# seuil arbitraire en dessous duquel nous ne voulons pas traiter
seuil = 400000.0

# transformer en minuscules 
sxb['TITULAIRES_DENOMINATION'] = sxb['TITULAIRES_DENOMINATION'].str.lower()

# Regroupement par entreprise
depenses_par_prestataire = sxb.groupby('TITULAIRES_DENOMINATION').sum()

# application du seuil pour filtrer les petits achats
top_depenses_par_prestataire = depenses_par_prestataire[depenses_par_prestataire['MONTANT'] > seuil]
top_depenses_par_prestataire = top_depenses_par_prestataire.reset_index()

# creation d'une liste ayant comme elements les entreprises
prestataires = top_depenses_par_prestataire['TITULAIRES_DENOMINATION']

# une autre liste avec les montants correspondants
montant = top_depenses_par_prestataire['MONTANT']
```





    