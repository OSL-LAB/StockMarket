# StockMarket

mport numpy as np
import pandas as pd
import statsmodels.api as sm
import statsmodels.formula.api as smf
%matplotlib inline
import matplotlib.pyplot as plt
df_sp500 = pd.read_csv("/Users/nikimanwani/Desktop/OSL/Python/Project/Data/s&p500.csv", usecols=[0,6])
df_vix = pd.read_csv("/Users/nikimanwani/Desktop/OSL/Python/Project/Data/VIX.csv", usecols=[0,6])
df_bseindia = pd.read_csv("/Users/nikimanwani/Desktop/OSL/Python/Project/Data/India.csv", usecols=[0,6])
df_crudeoil = pd.read_csv("/Users/nikimanwani/Desktop/OSL/Python/Project/Data/crudeoil.csv", header=None)
df_gold = pd.read_csv("/Users/nikimanwani/Desktop/OSL/Python/Project/Data/Gold2.csv", header=None)
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
##cleanedgold = cleanedgold.dropna() 
##np.isinf(cleanedgold) 
cleanedgold.replace([np.inf, -np.inf], np.nan)
cleanedgold.dropna() 
def findqregression(df1, df2, qu):
    print("qu is" + str(qu))
    merged = pd.merge(df1, df2, on='Date')
    print('here')
    ##print(merged.shape)
    del merged["Date"]
    merged = merged.rename(columns={'Adj Close_x': 'adjx', 'Adj Close_y': 'adjy'})
    mod = smf.quantreg('adjx ~ adjy', merged)
    print("qu is" + str(qu))
    print(df1[:10],df2[:10])
    res = mod.fit(q=qu)
    res=res.summary().as_csv()
    res=res.split("\n")
    print('here')
    res=[i.split(",") for i in res]
    print('here')
    return res
    print('here')
cleaned={
    "BSE":cleanedbse,
    "VIX":cleanedvix,
    "SP500":cleanedsp500,
    "CrudeOil":cleanedoil,
    "Gold":cleanedgold
}
qlist=[]
data={}
#comparison = [ ["BSE","VIX"], ["BSE","SP500"], ["BSE","CrudeOil"], ["BSE","Gold"]]
comparison = [ ["BSE","CrudeOil"], ["BSE","Gold"]]
for c in comparison:
    print("########################################################")
    print("Comaprision between "+c[0]+" and "+c[1])
    data[c[0]+c[1]]={}
    q=0.1
    qlist=[]
    while q<=1:
        r=findqregression(cleaned[c[0]],cleaned[c[1]],q)
        #print(r)
        data[c[0]+c[1]][q]={}
        print("**     Q value : "+str(q))
        print("          "+r[1][2]+" --> "+r[1][3])
        print("           "+r[7][3]+"("+r[8][0]+")"+" --> "+r[8][3])
        print("           "+r[7][3]+"("+r[9][0]+")"+" --> "+r[9][3])
        print("              "+r[7][4]+"("+r[8][0]+")"+" --> "+r[8][4])
        print("              "+r[7][4]+"("+r[9][0]+")"+" --> "+r[9][4])
        print("\n")
        data[c[0]+c[1]][q]['r']=r[1][3]
        data[c[0]+c[1]][q]['ti']=r[8][3]
        data[c[0]+c[1]][q]['ta']=r[9][3]
        data[c[0]+c[1]][q]['pti']=r[8][4]
        data[c[0]+c[1]][q]['pta']=r[9][4]
        qlist.append(q)
        q+=0.25
    for c in comparison:
    print("###### Graph of R2 Values: Comparision between "+c[0]+" and "+c[1]+" ####### ")
    plt.plot(qlist,[data[c[0]+c[1]][i]['r'] for i in qlist])
    plt.xlabel('Quantiles')
    plt.ylabel('R^2 value')
    plt.show()
for c in comparison:
    print("###### Graph of t(adjaceny) Values: Comparision between "+c[0]+" and "+c[1]+ " ######")
    plt.plot(qlist,[data[c[0]+c[1]][i]['ta'] for i in qlist])
    plt.xlabel('Quantiles')
    plt.ylabel('Adjacency')
    plt.show()
s=k.split("\n")
s2 = [i.split(",") for i in s]

#Pseudo R-squared:  ->1,(2,3)
# t value ->7,(3,4)
#            8(0) , 8(3,4)
#            9(0), 9(3,4)
