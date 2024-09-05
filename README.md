import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import requests
import random
from io import StringIO


url = "https://www.data.gouv.fr/fr/datasets/r/78348f03-a11c-4a6b-b8db-2acf4fee81b1"
response = requests.get(url)
data = response.text

#choisir 20 lignes aleatoirement
num_lines = sum(1 for line in data.split('\n')) - 1
sample_size = 20
sample_indices = sorted(random.sample(range(1, num_lines + 1), sample_size))
data_io = StringIO(data)
def skip_function(x):
    if x == 0:
        return False
    return x not in sample_indices

df3 = pd.read_csv(data_io, sep="|", usecols=['No disposition', 'Date mutation', 'Nature mutation', 'Valeur fonciere',
                                            'No voie', 'B/T/Q', 'Type de voie', 'Code voie', 
                                            'Voie', 'Code postal','Commune', 'Code departement', 
                                            'Code commune', 'Section', 'No plan','1er lot', '2eme lot', 
                                            'Nombre de lots', 'Code type local','Type local', 'Surface reelle bati',
                                            'Nombre pieces principales','Nature culture', 'Surface terrain'],
                                            skiprows=skip_function)



#suppression dépendance
df3 = df3[df3['Code type local'] != 3]



#transformation attribut
df3['Date mutation'] = pd.to_datetime(df3['Date mutation'], format='%d/%m/%Y')


# %% Fonctions auxiliaires


def replaceStr (string,old,new) :
    """Fonction similaire à la méthode str.replace.
    Prend en entrée un string qui va être modifié, le caractère à remplacer et le nouveau caractère.
    Renvoie un nouveau string avec les caractères remplacés"""
    result = ''
    if type(string) == str :
        for c in string :
            if c == old :
                result += new
            else :
                result += c

        return result
    else :
        print(string, " est de type ",type(string), " pas str.")



def dfStrToFloat(df,columns=[]) :
    """Fonction en place
    Prends en paramètres une dataframe et optionnellement une liste de colonnes dont les valeurs doivent être transformées en float. Si columns n'est pas précisé, la fonction va s'appliquer sur toute la dataframe.
    Utilise la fonction auxiliaire : replaceStr
    Renvoie le dataframe avec les valeurs de la colonnes en float."""

    if columns == []: #Si aucune colonnes n'est rentrée en paramètre on traite toute la dataframe
        columns = df.columns

    
    for c in columns : #On déroule sur toutes les colonnes
        newC = []   #Initialisation de la nouvelle colonne
        for string in df[c] :
            if type(string) == str :
                string = replaceStr (string,",",".") #S'il y a des virgules on les transforme en .
                string = float(string) #On transforme la chaîne en flottant

            newC.append(string) #On ajoute le flottant à la nouvelle ligne

        df[c] = newC #On met à jour la ligne

    return df #On renvoie la dataframe mise à jour
