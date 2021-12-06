# %% [markdown]
# # Markowitz Portfolio Optimization

# %% [markdown]
# - [Importations des packages](attachment:./#Importations-des-packages)
# - [Choix d'entreprises](attachment:./#Choix-d'entreprises)
# - [Calcul de rentabilité des actions par entreprise](attachment:./#Calcul-de-rentabilité-des-actions-par-entreprise)
# - [Matrice de Corrélations et de Covariances](attachment:./#Matrice-de-Corrélations-et-de-Covariances)
# - [Moyenne des rentabilités par entreprise](attachment:./#Moyenne-des-rentabilités-par-entreprise)
# - [Courbes interactives des taux de rentabilités](attachment:./#Courbes-interactives-des-taux-de-rentabilités)
# - [Courbe interactives des cours des actions](attachment:./#Courbe-interactives-des-cours-des-actions)
# - [Genérer les podérations($10.0000\; pondérations\;différentes$)](attachment:./#Genérer-les-podérations($10.0000\;-pondérations\;différentes$))
# - [Rentabilités du portefeuille pour les $10.0000$ pondérations](attachment:./#Rentabilités-du-portefeuille-pour-les-$10.0000$-pondérations)
# - [Calcul des risques du portefeuille pour les $10.0000$ pondérations](attachment:./#Calcul-des-risques-du-portefeuille-pour-les-$10.0000$-pondérations)
# - [Graphe Rentabilité-Risque du portefeuille](attachment:./#Graphe-Rentabilité-Risque-du-portefeuille)

# %% [markdown]
# # Importations des packages

# %%
#importation des package qu'on va utiliser pour la visualisation et le traitement des données 

import Casabourselib as cbl
import pandas as pd 
import plotly.graph_objects as go
import plotly.graph_objects as go
import plotly.express as px
import numpy as np
import random
pd.options.plotting.backend = "plotly"

# %% [markdown]
# # Choix d'entreprises

# %%

#Alias des entreprises
entreprises=cbl.get_tickers()
entreprises[0:5][:]

# %% [markdown]
# **Nous choisirons Lydec, Attijariwafa,M2M Group et CTM**
# 
# car secteurs d'activités différents

# %% [markdown]
# # Calcul de rentabilité des actions par entreprise

# %%
entreprises,tickers,j,rentability=[],['ATW','CTM','LYD','M2M'],0,[0]
for i in tickers:
    entreprises.append(cbl.get_price(i,'01/10/2016','01/10/2021'))
    rentability=[0]
    cours=entreprises[j]['value']
    for k in range(len(cours)-1):
        rentability.append(cours[k+1]/cours[k] -1)
    entreprises[j].insert(1,'rentability',rentability,allow_duplicates=False)
    j+=1
ATW,CTM,LYD,M2M=entreprises[0],entreprises[1],entreprises[2],entreprises[3]
#Sorting by index
ATW.index,CTM.index,LYD.index,M2M.index=pd.to_datetime(ATW.index),pd.to_datetime(CTM.index),pd.to_datetime(LYD.index),pd.to_datetime(M2M.index)
ATW,CTM,LYD,M2M=ATW.sort_index(),CTM.sort_index(),LYD.sort_index(),M2M.sort_index()

# %%
matRent=pd.DataFrame({
'ATW':ATW['rentability'],
'CTM':CTM['rentability'],
'LYD':LYD['rentability'],
'M2M':M2M['rentability']
})
matRent

# %% [markdown]
# # Matrice de Corrélations et de Covariances

# %%
matCorr=matRent.corr()
matCov=matRent.cov()
print('Matrice de corrélations :\n',matCorr)
print('Matrice de covariances :\n',matCov)

# %% [markdown]
# **Les coeffs de corr sont proche de 0 alors la diversification est assez bonne**

# %% [markdown]
# # Moyenne des rentabilités par entreprise

# %%
avgRent=matRent.mean()
avgRent

# %% [markdown]
# # Courbes interactives des taux de rentabilités

# %%
matRent.plot(x=matRent.index,y=matRent.columns,title='Courbe des taux de returns',labels=dict(index='Date',value='Rentabilité',variable='Entreprise'),height=1000,width=1000)

# %% [markdown]
# # Courbe interactives des cours des actions

# %%
matValue=pd.DataFrame({
'ATW':ATW['value'],
'CTM':CTM['value'],
'LYD':LYD['value'],
'M2M':M2M['value']
})
matValue.plot(title='Cours des actions',labels=dict(index='Date',value='Cours'),width=1000,height=1000)

# %% [markdown]
# # Genérer les podérations($10.0000\; pondérations\;différentes$)

# %%
import random
vec1 = []
vec2 = []
vec3 = []
vec4 = []

for _ in range(0, 10001):
    
    a = round(random.uniform(0.2, 0.8),6)
    b = round(random.uniform(0.2, 0.8),6)
    c = round(random.uniform(0.2, 0.8),6)
    d = 0
    while a + b + c > 1:
        a = round(random.uniform(0.2, 1),6)
        b = round(random.uniform(0.2, 1),6)
        c = round(random.uniform(0.2, 1),6)
    d = 1 - (a + b + c)
    vec1.append(a)
    vec2.append(b)
    vec3.append(c)
    vec4.append(d)

vec1 = np.array(vec1)
vec2 = np.array(vec2)
vec3 = np.array(vec3)
vec4 = np.array(vec4)

vec=pd.DataFrame([vec1,vec2,vec3,vec4],index=tickers)
vec=vec.transpose()
vec

# %% [markdown]
# # Rentabilités du portefeuille pour les $10.0000$ pondérations

# %%
RentPort,j=pd.DataFrame(),0
for i in tickers:
    RentPort.insert(j,i,vec[i]*avgRent[i])
    j+=1
RentPort.insert(0,'RentPort',RentPort.sum(axis=1))
RentPort

# %% [markdown]
# # Calcul des risques du portefeuille pour les $10.0000$ pondérations

# %%
VarPort=pd.DataFrame()
j=0
for i in tickers:
    k=pd.DataFrame(matCov[i]*vec)
    k=k.apply(lambda x: np.sum(x),axis=1)
    VarPort.insert(j,i,k)
    j+=1
VarPort=VarPort.apply(lambda x: np.sum(x),axis=1)
VarPort=VarPort.apply(lambda x: np.sqrt(x))
VarPort.columns=['RisquePort']

# %%
var=VarPort.to_numpy()
var

# %% [markdown]
# # Graphe Rentabilité-Risque du portefeuille

# %%
Portefeuille=RentPort
Portefeuille.insert(0,'RisquePort',VarPort)
Portefeuille

# %%

x=[matCov[i][i] for i in tickers]
y=avgRent
y=y.to_numpy()
df=pd.DataFrame()
df.insert(0,'x',x)

# %%

fig=Portefeuille.plot.scatter(Portefeuille['RisquePort'],['RentPort'],labels=dict(index='Risque Portefeuille',value='Rentability'),title='Rent par Risque pris')

fig.show()



# %%

# %%
