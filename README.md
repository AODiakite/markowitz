---
title: "README"
author: "ABDOUL OUDOUSS DIAKITE"
date: "12/7/2021"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


# Markowitz
- [Importations des packages](#Importations-des-packages)
- [Choix d'entreprises](#Choix-d'entreprises)
- [Calcul de rentabilité des actions par entreprise](#Calcul-de-rentabilité-des-actions-par-entreprise)
- [Matrice de Corrélations et de Covariances](#Matrice-de-Corrélations-et-de-Covariances)
- [Moyenne des rentabilités par entreprise](#Moyenne-des-rentabilités-par-entreprise)
- [Courbes interactives des taux de rentabilités](#Courbes-interactives-des-taux-de-rentabilités)
- [Courbe interactives des cours des actions](#Courbe-interactives-des-cours-des-actions)
- [Genérer les podérations($10.0000\; pondérations\;différentes$)](#Genérer-les-podérations($10.0000\;-pondérations\;différentes$))
- [Rentabilités du portefeuille pour les $10.0000$ pondérations](#Rentabilités-du-portefeuille-pour-les-$10.0000$-pondérations)
- [Calcul des risques du portefeuille pour les $10.0000$ pondérations](#Calcul-des-risques-du-portefeuille-pour-les-$10.0000$-pondérations)
- [Graphe Rentabilité-Risque du portefeuille](#Graphe-Rentabilité-Risque-du-portefeuille)

# Importations des packages

``` python
#importation des package qu'on va utiliser pour la visualisation et le traitement des données 
import Casabourselib as cbl
import pandas as pd 
import plotly.graph_objects as go
import plotly.graph_objects as go
import plotly.express as px
import numpy as np
import random
pd.options.plotting.backend = "plotly"
```

# Choix d'entreprises

```python
#Alias des entreprises
entreprises=cbl.get_tickers()
entreprises[0:5][:]
```

**Nous choisirons Lydec, Attijariwafa,M2M Group et CTM**
car secteurs d'activités différents

# Calcul de rentabilité des actions par entreprise

```python
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
```

```python
matRent=pd.DataFrame({
'ATW':ATW['rentability'],
'CTM':CTM['rentability'],
'LYD':LYD['rentability'],
'M2M':M2M['rentability']
})
matRent
```

# Matrice de Corrélations et de Covariances

```python
matCorr=matRent.corr()
matCov=matRent.cov()
print('Matrice de corrélations :\n',matCorr)
print('Matrice de covariances :\n',matCov)
```

**Les coeffs de corr sont proche de 0 alors la diversification est assez bonne**

# Moyenne des rentabilités par entreprise

```python
avgRent=matRent.mean()
avgRent
```

# Courbes interactives des taux de rentabilités

```python
plotRent=matRent.plot(x=matRent.index,y=matRent.columns,title='Courbe des taux de returns',labels=dict(index='Date',value='Rentabilité',variable='Entreprise'))
plotRent.show()
```

# Courbe interactives des cours des actions

```python
matValue=pd.DataFrame({
'ATW':ATW['value'],
'CTM':CTM['value'],
'LYD':LYD['value'],
'M2M':M2M['value']
})
fig0=matValue.plot(title='Cours des actions',labels=dict(index='Date',value='Cours'))
fig0.show()
```

# Genérer les pondérations(10.0000 pondérations différentes$)

```python
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
```
# Rentabilités du portefeuille pour les 10.0000 pondérations

```python
RentPort,j=pd.DataFrame(),0
for i in tickers:
    RentPort.insert(j,i,vec[i]*avgRent[i])
    j+=1
RentPort.insert(0,'RentPort',RentPort.sum(axis=1))
RentPort
```
# Calcul des risques du portefeuille pour les 10.0000 pondérations

```python
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
```
```python
var=VarPort.to_numpy()
var
```
# Graphe Rentabilité-Risque du portefeuille
```python
Portefeuille=RentPort
Portefeuille.insert(0,'RisquePort',VarPort)
Portefeuille
```
```python

x=[matCov[i][i] for i in tickers]
y=avgRent
y=y.to_numpy()
df=pd.DataFrame()
df.insert(0,'x',x)
```
```python
fig=Portefeuille.plot.scatter(Portefeuille['RisquePort'],['RentPort'],labels=dict(index='Risque Portefeuille',value='Rentability'),title='Rent par Risque pris')

fig.show()
```
