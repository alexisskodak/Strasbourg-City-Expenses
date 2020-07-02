# Analyse de données

## Projet IPython visant à représenter visuellement les données publiées sur [Data Gouv](https://www.data.gouv.fr/fr/datasets/achats-des-principales-communes-francaises-en-2019/) afin de les rendre lisibles facilement. Cas pratique: achats de la commune de Strasbourg en 2019.

## Organisation en 3 axes principaux:

1. ### Mise en place d'un ****environnement virtuel Python****
   
   - On choisit pour ce propos un environnement ****Anaconda**** comportant les bibliothèques et modules adéquats pour le ****traitement, nettoyage et la représentation visuelle**** des données

2. ### Traitement des données
   
   - Création des objets ****Pandas DataFrame****.
   - Nettoyage des lignes portant des cases majoritairement vides.
   - **Groupement par colonnes** ou _'groupby'_
   - ajout de colonnes et d'autres éléments supplémentaires pour avoir une table plus exploitable.

3. ### ****Représentation visuelle****
   
   - avec des graphiques traditionnels grâce aux outils ****Matplotlib****

## Attentes:

- Pouvoir tirer des information utiles afin de pouvoir interpréter certains aspects des achats de la commune.
- Rendre l'analyse facilement réplicable

---

# 1. Environnement Virtuel

- Installation [Anaconda](https://www.anaconda.com/products/individual). Guide d'installation figure dans le site ci contre

- Une fois Anaconda installé:
  
  - Dans un interpreteur de commandes bash ou similaire, saisir:
    
    - ```
       conda create --name nom_environnement
       conda activate nom_environnement
      ```
  
  - Installation des bibliothèques nécessaires
    
    - ```
       conda install pandas matplotlib
      ```
      
       (Selon la distribution conda choisie, ces bibliothèques peuvent être installées par défaut)
  
  - Optionnel: installation de **Jupyter Notebooks**.
    On choisit Jupyter Notebooks pour sa facilité d'utilisation et pour l'aspect interactif ou exécution par blocs de code.
    
    - ```
       conda install -c conda-forge jupyterlab
      ```

### Une fois notre environnement créé et activé, ouvrir un Notebook Jupyter ou un éditeur de texte a choix

---

# 2. Traitement des données

Pour faciliter la démarche, il convient de télécharger le fichier avec les données sous format .csv et de les sauvegarder dans le même dossier ou on a créé notre fichier _.ipynb_.

- Importation des bibliothèques
  
  ```Python
  import pandas as pd
  import matplotlib.pyplot as plt
  ```
- Création de notre objet DataFrame principal
  
  ```Python
  sxb = pd.read_csv('sxb.csv', sep=';')
  ```

# Supprimer les files ou toutes les cases sont vides

sxb.dropna(how='all', inplace=True)

# Visualiser les 10 premières files de la table

sxb.head(10)

```
Nous observons une table de la forme suivante

![image](sxbhead.png)

## 2.1 Opérations sur les données en fonction du mois

### 2.1.1 On crée tout d'abord une colonne pour le mois

```Python
# Prendre le mois dans la colonne 'DATE' en format 'int'
sxb['MOIS'] = sxb['NOTIFICATION_DATE'].str[5:7].astype(int)

# Convertir le type des cases 'MONTANT' en 'float' lorsque possible
sxb['MONTANT'] = pd.to_numeric(sxb['MONTANT'],errors='coerce')
```

### 2.1.2 Manipulations sur cette nouvelle colonne

```Python
# Groupement de la table par mois
depenses_par_mois = sxb.groupby('MOIS').sum()

# création d'une colonne pour représenter la fraction du montant total
depenses_par_mois['MONTANT RELATIF'] = depenses_par_mois['MONTANT'] * 100 / total2019
depenses_par_mois = depenses_par_mois.reset_index()

# calculer les pourcentages des montants
pourcentages = depenses_par_mois['MONTANT RELATIF']
```

## 2.2 Opérations sur les données en fonction des entreprises

```Python
# seuil arbitraire en dessous duquel nous ne traiterons pas
seuil = 400000.0
```

### Transformer en minuscules

```Python
sxb['TITULAIRES_DENOMINATION'] = sxb['TITULAIRES_DENOMINATION'].str.lower()
```

### Regroupement par entreprise

```Python
depenses_par_prestataire = sxb.groupby('TITULAIRES_DENOMINATION').sum()
```

### Application du seuil pour filtrer les petits achats

```Python
top_depenses_par_prestataire = depenses_par_prestataire[depenses_par_prestataire['MONTANT'] > seuil]
top_depenses_par_prestataire = top_depenses_par_prestataire.reset_index()
```

## 2.3 Opérations sur les données en fonction des achats individuels

### Convertir en minuscules les cases portant l'objet de l'achat

```Python
sxb['MARCHE_OBJET'] = sxb['MARCHE_OBJET'].str.lower()
```

### Grouper la table par objet d'achat et en faire de ces derniers l'indice

```Python
depenses_par_projet = sxb.groupby('MARCHE_OBJET').sum()
depenses_par_projet = depenses_par_projet.reset_index()
```

### Filtrer les achats les plus petits

```Python
top_depenses_par_projet = depenses_par_projet[depenses_par_projet['MONTANT'] > seuil]
```

---

# 3. Représentation Graphique

## Achats de la commune par mois

![image](depenses_mois.png) ![image](depenses_mois_relatif.png)

### On constate qu'un tiers des achats de la commune ont été effectués au mois d'Octobre

Interpretation possible:

- La commune prépare la ville pour le fêtes de fin d'année, notamment dans le cadre 'Strasbourg Capitale De Noel'

Pour vérifier cette hypothèse, nous allons rassembler tous les achats qui ont été effectués par la commune pendant le mois d'Octobre. Ensuite, nous allons identifier des tendances pour essayer de déduire le motif de chaque achat. Etant donné que nous n'avons pas de colonne qui nous précise cela, on va se baser entièrement sur la colonne '[MARCHE_OBJET]

```python
achats_octobre = sxb.loc[(sxb['MOIS'] == 10)]
projets_octobre = [projet for projet in sxb['MARCHE_OBJET']]

# Convertir une liste de mots en une seule chaîne de caractères
# en précisant un séparateur
def joindre_mots(liste, separateur):
   string_final = separateur.join(liste)
   return string_final

projets_str = joindre_mots(projets_octobre, ' ')
```

Grâce à des expressions régulières on pourra se débarrasser des caracteres polluants

```python
import re

sans_symboles = re.sub('[0-9]|\'|\.|\-|\/|\,|\:|\(|\)', '', projets_str)
split_line = sans_symboles.split()
```

Nous allons également supprimer des mots qui nous donnent pas d'informations spécialement utiles (par exemple les articles, prépositions....)

```python
articles = ['LE', 'A', 'LA', 'LES', 
            'DE', 'D', 'C', 'DU', 
            'DUN', 'DUNE', 'POUR', 
            'AU', 'AUX', 'QUE', 
            'SUR', 'ET', 'PAR', 
            'OU', 'DES', 'EN', 
            'RUE', 'DANS', 'SITUE']

sans_articles = [word for word in split_line if word not in articles]

# On crée une nouvelle liste avec les mots en minuscules
en_minuscules = []
for mot in new_words:
    mot = mot.lower()
    en_minuscules += [mot]
```

on va ensuite representar la fréquence de chaque mot grace a la bibliotheque WordCloud

```python
from wordcloud import WordCloud

# On transforme à nouveau en une seule chaîne de caractères
# car c'est le type du parametre admis par la méthode ‘generate’ de WordCloud
mots_du_nuage = joindre mots(minuscules, ' ')
nuage_mot = WordCloud(width=800, height=800, background_color ='white', min_font_size = 10).generate(mots_du_nuage)

# Quelques réglages matplotlib pour finaliser
plt.figure(figsize = (10, 10), facecolor = None) 
plt.imshow(nuage_mot) 
plt.axis("off") 
plt.tight_layout(pad = 0) 
  
plt.show() 
```

Voilà ce que l'on obtient

![image](nuages.png)


Les resultats sont tres interessants cependant plutot decevants par rapport a notre interpretation. En effet on voit que la plupart des achats de la commune pendant le mois d'octobre, concernent le gros oeuvre et BTP, eclairage publique, renovation d'ecoles... On voit a peine et en tres petit les mots 'sapin', 'fetes', 'festivites', 'noel'.

Possibles hypotheses:

- C'est la CUS et/ou la Region Grand Est et non pas la commune qui gere ces achats

- Les depenses publiques pour l'amenagement de la ville pour les festivites son beaucoup plus faibles de ce que l'on estimait

Quand à la question du pic d'achats au mois d'Octobre, nous n'avons pas les moyens d'en tirer une interpretation valable depuis les donnees exploitees.

## Achats de la commune par entreprise

![image](depenses_prestataires.png)

## Projets les plus importants

![image](depenses_projets.png)

---

# Conclusion:

### Les données trouvées sur Open Data sont facilement traitables et bien classifiées, le nettoyage a faire est minime et les visualisations semblent satisfaisantes car répondent parfaitement aux attentes

Limites de cette étude:

- On a choisit de représenter les données qui nous sont fournies sans forcément les interpréter, en effet, il faudrait un investissement plus important sur le temps accordé à la partie statistique.
- La manière dont ces données ont été saisies, nous limite en terme de possibilités de traitement. Parmi des améliorations possibles on peut citer, une colonne qui catégorise par domaine d'activité les achats de la commune.
- De même, un fichier identique pour plusieurs années pourrait vraiment donner de l'ampleur à une analyse comme celle effectuée ci dessus, sans forcément déployer de moyens plus sophistiqués.
