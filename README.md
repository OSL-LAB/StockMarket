# StockMarket

import numpy as np
import pandas as pd
df_bseindia = pd.read_csv("India.csv")
df_sp500 = pd.read_csv("s&p500.csv")
df_vix = pd.read_csv("VIX.csv")
df_bseindia = pd.read_csv("India.csv")
df_crudeoil = pd.read_csv("crudeoil.csv", header=None)
df_gold = pd.read_csv("Gold2.csv", header=None)

col_a = list(df_crudeoil[0])
col_b = list(df_gold[0])
col_c = list(df_vix["Date"])
col_d = list(df_bseindia["Date"])
col_e = list(df_sp500["Date"])

def intersect(col_c, col_d):
    return list(set(col_c) & set(col_d))

answer = intersect(col_c, col_d)
answer = intersect(col_a, answer)
answer = intersect(col_e, answer)
answer = intersect(col_b, answer)

cleanedbse = df_bseindia[df_bseindia["Date"].isin(answer)]
cleanedvix = df_vix[df_vix["Date"].isin(answer)]
cleanedsp500 = df_sp500[df_sp500["Date"].isin(answer)]
cleanedoil = df_crudeoil[df_crudeoil[0].isin(answer)]
cleanedgold = df_gold[df_gold[0].isin(answer)]

cleanedoil['Date']=cleanedoil[0]
del cleanedoil[0]
cleanedoil['Adj Close']=cleanedoil[1]
del cleanedoil[1]
cleanedgold['Date']=cleanedgold[0]
del cleanedgold[0]
cleanedgold['Adj Close']=cleanedgold[1]
del cleanedgold[1]

temp=pd.merge(cleanedvix,cleanedbse,on="Date")
combined=pd.merge(temp,cleanedsp500,on="Date")
len(combined)

del cleanedbse["Date"]
del combined["Date"]
cleanedbse[:5]

original = cleanedbse["Adj Close"].as_matrix()
x=np.diff(original)
x

labels=[]
for i in list(x):
    if i>0:
        labels.append(1)
    else: 
        labels.append(0)
labels.append(1)
print(len(labels))

from sklearn.cross_validation import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.linear_model import LogisticRegression

def find_classification(X,Y):
    x_train, x_test, y_train, y_test = train_test_split(X, Y, test_size = 0.2)
    clf = DecisionTreeClassifier(random_state=0)
    c=clf.fit(x_train,y_train)
    clf2 = LogisticRegression()
    c2=clf2.fit(x_train,y_train)
    return c.score(x_test,y_test),c2.score(x_test,y_test)

find_classification(cleanedbse,labels)
find_classification(combined,labels)
