---
title: "Markowitz"
author: "ABDOUL OUDOUSS DIAKITE"
date: "12/7/2021"
Lien pour visualiser les graphes: https://datalore.jetbrains.com/view/notebook/OrrTQ5WHt2niaxQiReIxHr
---
## R Markdown

# Markowitz

-   [Importations des packages](#Importations-des-packages)
-   [Choix d’entreprises](#Choix-d'entreprises)
-   [Calcul de rentabilité des actions par
    entreprise](#Calcul-de-rentabilité-des-actions-par-entreprise)
-   [Matrice de Corrélations et de
    Covariances](#Matrice-de-Corrélations-et-de-Covariances)
-   [Moyenne des rentabilités par
    entreprise](#Moyenne-des-rentabilités-par-entreprise)
-   [Courbes interactives des taux de
    rentabilités](#Courbes-interactives-des-taux-de-rentabilités)
-   [Courbe interactives des cours des
    actions](#Courbe-interactives-des-cours-des-actions)
-   [Genérer les
    podérations(10.0000 *p**o**n**d**é**r**a**t**i**o**n**s* *d**i**f**f**é**r**e**n**t**e**s*)](#Genérer-les-podérations($10.0000;-pondérations;différentes$))
-   [Rentabilités du portefeuille pour les 10.0000
    pondérations](#Rentabilités-du-portefeuille-pour-les-$10.0000$-pondérations)
-   [Calcul des risques du portefeuille pour les 10.0000
    pondérations](#Calcul-des-risques-du-portefeuille-pour-les-$10.0000$-pondérations)
-   [Graphe Rentabilité-Risque du
    portefeuille](#Graphe-Rentabilité-Risque-du-portefeuille)

# Importations des packages
```python
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
# Choix d’entreprises

```python
    #Alias des entreprises
    entreprises=cbl.get_tickers()
    entreprises[0:5][:]
```
    ##           Titre Ticker
    ## 0        Addoha    ADH
    ## 1          AFMA    AFM
    ## 2  Afric Indus.    AFI
    ## 3  Afriquia Gaz    GAZ
    ## 4          Agma    AGM

**Nous choisirons Lydec, Attijariwafa,M2M Group et CTM** car secteurs
d’activités différents

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
    ##                  ATW       CTM       LYD       M2M
    ## date                                              
    ## 2016-06-12  0.000000  0.000000  0.000000  0.000000
    ## 2016-07-12  0.000000       NaN       NaN       NaN
    ## 2016-08-12 -0.000256  0.000000       NaN -0.032542
    ## 2016-09-12  0.015645  0.000000  0.048544  0.057576
    ## 2016-12-14  0.007323 -0.055556  0.061111 -0.071633
    ## ...              ...       ...       ...       ...
    ## 2021-12-03  0.004294  0.000000  0.012821  0.039823
    ## 2021-12-04  0.011765 -0.017143 -0.018349  0.000000
    ## 2021-12-05  0.001235 -0.004871  0.000000  0.009783
    ## 2021-12-07 -0.001571  0.000000  0.000000  0.038824
    ## 2021-12-08 -0.006303  0.000000  0.000000 -0.034665
    ## 
    ## [1197 rows x 4 columns]

# Matrice de Corrélations et de Covariances

```python 
    matCorr=matRent.corr()
    matCov=matRent.cov()
    print('Matrice de corrélations :\n',matCorr)
```
    ## Matrice de corrélations :
    ##            ATW       CTM       LYD       M2M
    ## ATW  1.000000  0.145724  0.068046  0.085371
    ## CTM  0.145724  1.000000  0.030075  0.019452
    ## LYD  0.068046  0.030075  1.000000  0.032830
    ## M2M  0.085371  0.019452  0.032830  1.000000
```python   
    print('Matrice de covariances :\n',matCov)
```
    ## Matrice de covariances :
    ##            ATW       CTM       LYD       M2M
    ## ATW  0.000144  0.000030  0.000018  0.000027
    ## CTM  0.000030  0.000301  0.000012  0.000009
    ## LYD  0.000018  0.000012  0.000494  0.000019
    ## M2M  0.000027  0.000009  0.000019  0.000709

**Les coeffs de corr sont proche de 0 alors la diversification est assez
bonne**

# Moyenne des rentabilités par entreprise
```python
    avgRent=matRent.mean()
    avgRent
```
    ## ATW    0.000266
    ## CTM    0.000184
    ## LYD   -0.000363
    ## M2M    0.001026
    ## dtype: float64

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

# Genérer les pondérations(10.0000 *p**o**n**d**é**r**a**t**i**o**n**s* *d**i**f**f**é**r**e**n**t**e**s*)
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
    ##             ATW       CTM       LYD       M2M
    ## 0      0.285020  0.428565  0.275178  0.011237
    ## 1      0.482639  0.267646  0.244351  0.005364
    ## 2      0.270913  0.371924  0.303576  0.053587
    ## 3      0.233194  0.462270  0.233341  0.071195
    ## 4      0.350975  0.324268  0.245453  0.079304
    ## ...         ...       ...       ...       ...
    ## 9996   0.238335  0.277420  0.467343  0.016902
    ## 9997   0.337909  0.333802  0.277693  0.050596
    ## 9998   0.465503  0.201243  0.271495  0.061759
    ## 9999   0.299801  0.221754  0.405983  0.072462
    ## 10000  0.226346  0.418627  0.207910  0.147117
    ## 
    ## [10001 rows x 4 columns]


# Rentabilités du portefeuille pour les 10.0000 pondérations
```python
    RentPort,j=pd.DataFrame(),0
    for i in tickers:
        RentPort.insert(j,i,vec[i]*avgRent[i])
        j+=1
    RentPort.insert(0,'RentPort',RentPort.sum(axis=1))
    RentPort
```
    ##        RentPort       ATW       CTM       LYD       M2M
    ## 0      0.000066  0.000076  0.000079 -0.000100  0.000012
    ## 1      0.000095  0.000129  0.000049 -0.000089  0.000006
    ## 2      0.000085  0.000072  0.000069 -0.000110  0.000055
    ## 3      0.000136  0.000062  0.000085 -0.000085  0.000073
    ## 4      0.000145  0.000093  0.000060 -0.000089  0.000081
    ## ...         ...       ...       ...       ...       ...
    ## 9996  -0.000038  0.000063  0.000051 -0.000170  0.000017
    ## 9997   0.000103  0.000090  0.000062 -0.000101  0.000052
    ## 9998   0.000126  0.000124  0.000037 -0.000099  0.000063
    ## 9999   0.000048  0.000080  0.000041 -0.000147  0.000074
    ## 10000  0.000213  0.000060  0.000077 -0.000076  0.000151
    ## 
    ## [10001 rows x 5 columns]

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

    var=VarPort.to_numpy()
    var
```
    ## array([0.01927656, 0.01836279, 0.01990855, ..., 0.01918114, 0.02049064,
    ##        0.0205572 ])

# Graphe Rentabilité-Risque du portefeuille
```python
    Portefeuille=RentPort
    Portefeuille.insert(0,'RisquePort',VarPort)
    Portefeuille
```
    ##        RisquePort  RentPort       ATW       CTM       LYD       M2M
    ## 0        0.019277  0.000066  0.000076  0.000079 -0.000100  0.000012
    ## 1        0.018363  0.000095  0.000129  0.000049 -0.000089  0.000006
    ## 2        0.019909  0.000085  0.000072  0.000069 -0.000110  0.000055
    ## 3        0.019880  0.000136  0.000062  0.000085 -0.000085  0.000073
    ## 4        0.019629  0.000145  0.000093  0.000060 -0.000089  0.000081
    ## ...           ...       ...       ...       ...       ...       ...
    ## 9996     0.020414 -0.000038  0.000063  0.000051 -0.000170  0.000017
    ## 9997     0.019528  0.000103  0.000090  0.000062 -0.000101  0.000052
    ## 9998     0.019181  0.000126  0.000124  0.000037 -0.000099  0.000063
    ## 9999     0.020491  0.000048  0.000080  0.000041 -0.000147  0.000074
    ## 10000    0.020557  0.000213  0.000060  0.000077 -0.000076  0.000151
    ## 
    ## [10001 rows x 6 columns]

```python
    x=[matCov[i][i] for i in tickers]
    y=avgRent
    y=y.to_numpy()
    df=pd.DataFrame()
    df.insert(0,'x',x)


    fig=Portefeuille.plot.scatter(Portefeuille['RisquePort'],['RentPort'],labels=dict(index='Risque Portefeuille',value='Rentability'),title='Rent par Risque pris')

    fig.show()
```
    
