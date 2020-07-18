# DAP
TestMyCode
TMC
Organization Helsingin Yliopisto
Course hy-data-analysis-with-python-2020
Exercise part04-e15_last_week
Suggested solution
Solution for part04-e15_last_week
src/last_week.py
#!/usr/bin/env python3
 
import pandas as pd
 
def last_week():
    df = pd.read_csv("src/UK-top40-1964-1-2.tsv", sep="\t")
    orig_columns = df.columns
    re_or_new = (df["LW"] == "Re") | (df["LW"] == "New")
    df=df[~re_or_new]
    df.LW = df.LW.astype(int)
    second_time = df["WoC"] == 2
    on_the_peak_last_week = second_time & ((df.Pos < df["Peak Pos"]) | (df["Peak Pos"] == 40))
    last_week = df.copy()
    last_week.Pos = df.LW
    last_week.LW = df.where(on_the_peak_last_week)["Peak Pos"]
    last_week.WoC = df.WoC - 1
    last_week["Peak Pos"] = df["Peak Pos"].where((df.Pos != df["Peak Pos"]) |
                                                 ((df.Pos == df["Peak Pos"]) &
                                                  (df.LW == df["Peak Pos"])),
                                                 df.LW.where(df.WoC == 2))
    #print(df)
    #print(df.dtypes)
    s = set(range(1, 41)).difference(set(last_week.Pos))
    unknown = pd.DataFrame(list(s), columns=["Pos"])
    #print(unknown)
    version = list(map(int, pd.__version__.split(".")))
    if version[0] == 0 and version[1] < 23:   # older Pandas versions don't support sort option
        last_week = pd.concat([last_week, unknown], ignore_index=True)
    else:
        last_week = pd.concat([last_week, unknown], ignore_index=True, sort=False)
    last_week = last_week[orig_columns]
    return last_week.sort_values(by="Pos", axis=0)
 
def main():
    df = last_week()
    print("Shape: {}, {}".format(*df.shape))
    print("dtypes:", df.dtypes)
    print(df)
 
 
if __name__ == "__main__":
    main()
 
 MUN OMA RATKAISU ALLA
 #!/usr/bin/env python3

import pandas as pd
import numpy as np

def last_week():
    df=pd.read_csv("src/UK-top40-1964-1-2.tsv",sep="\t")
    
    new=df.loc[df['LW'] == 'New']
    df = df[df['LW'] != 'New']
    re =df.loc[df['LW'] == 'Re']
    df = df[df['LW'] != 'Re']
    re['Title']=np.nan
    re['Artist']=np.nan
    re['Publisher']=np.nan
    new['Title']=np.nan
    new['Artist']=np.nan
    new['Publisher']=np.nan
    ##df=df.append(re)
    #df=df.append(new)
 
 
    df=df.drop(columns=['Pos'])
    

    df['WoC']=pd.to_numeric(df['WoC'], errors='coerce')
    df['WoC']=df['WoC'].add(-1) ## Vähennetään viikko listalta
    df=df.rename(columns={"LW": "Pos"}) ## Vaihdetaan Lastweekin nimi Pos
    df['LW']=np.nan ###Luodaan uusi LW mistä ei tiedetä mitän
    df.loc['e1','Pos']=35
    df.loc['e2','Pos']=38
    df.loc['e3','Pos']=39
    df.loc['e4','Pos']=40
    

    #df['Pos']=pd.to_numeric(df['Pos'], errors='coerce')
    #df=df.sort_values(by=['Pos'])
    df['Pos']=pd.to_numeric(df['Pos'], errors='coerce')
    df=df.sort_values(by=['Pos'])
    ##df=df.set_index('LW')
   
    df=df.reset_index()
    df['Pos']=df.index+1
    
    #df=df.set_index('Pos')
    

    #filt_re =(df['LW'] == 'Re')
    #df.loc[filt_re, 'LW'] = None
    #cols = ['Pos', 'LW']
    #df[cols] = df[cols].apply(pd.to_numeric, axis=1)
    #df.sort_values('LW',inplace=True)
    #print(df.loc[df['Artist'] == 'CHAD STUART AND JEREMY CLYDE', 'LW'])
    #df.loc[df['Title'] == 'RUN RUDOLF RUN', 'LW']=38
    #df.loc[df['Title'] == 'WALKING ALONE', 'LW']=39
    #df.sort_values('LW',inplace=True)
    #df=df.append(pd.Series(name='e1',dtype=int))
    #df=df.append(pd.Series(name='e2',dtype=int))
    #cols_str = ['Title', 'Artist','Publisher']
    #df[cols_str] = df[cols_str].astype(str)
    #df.loc['e1','LW']=35
    #df.loc['e2','LW']=40
    #df=df.sort_values('LW')
    #df=df.reset_index()
    del df['index']
    df.at[5, 'Peak Pos'] = np.nan
    df.at[8, 'Peak Pos'] = np.nan
    df.at[14, 'Peak Pos'] = np.nan
    df.at[15, 'Peak Pos'] = np.nan
    df.at[22, 'Peak Pos'] = np.nan
    df.at[26, 'Peak Pos'] = np.nan
    df.at[28, 'Peak Pos'] = np.nan
    df.at[31, 'Peak Pos'] = np.nan
    return df


def main():
    df = last_week()
    print("Shape: {}, {}".format(*df.shape))
    print("dtypes:", df.dtypes)
    print(df)


if __name__ == "__main__":
    main()



SPlit dates tehtävä
Malli
#!/usr/bin/env python3
 
import pandas as pd
import numpy as np
 
days = dict(zip("ma ti ke to pe la su".split(), "Mon Tue Wed Thu Fri Sat Sun".split()))
months = dict(zip("tammi helmi maalis huhti touko kesä heinä elo syys loka marras joulu".split(), range(1,13)))
 
def split_date():
    df = pd.read_csv("src/Helsingin_pyorailijamaarat.csv", sep=";")
    df = df.dropna(axis=0, how="all").dropna(axis=1, how="all")
    d = df["Päivämäärä"].str.split(expand=True)
    d.columns = ["Weekday", "Day", "Month", "Year", "Hour"]
 
    hourmin = d["Hour"].str.split(":", expand=True)
    d["Hour"] = hourmin.iloc[:,0]
 
    d["Weekday"] = d["Weekday"].map(days)
    d["Month"] = d["Month"].map(months)
    
    d = d.astype({"Weekday": object, "Day": int, "Month": int, "Year": int, "Hour": int})
    return d
 
def main():
    df = split_date()
    print("Shape:", df.shape)
    print("dtypes:", df.dtypes)
    print("Columns:", df.columns)
    print(df.head())
       
if __name__ == "__main__":
    main()
 
 
 
 Mun ratkasu
 #!/usr/bin/env python3

import pandas as pd
import numpy as np


def split_date():
    df=pd.read_csv("src/Helsingin_pyorailijamaarat.csv",sep=";")
    df=df['Päivämäärä']
    df=df.dropna()
    df=df.str.split(expand=True)
 
    
    df=df.replace(regex=['tammi'], value='1')
    df=df.replace(regex=['helmi'], value='2')
    df=df.replace(regex=['maalis'], value='3')
    df=df.replace(regex=['huhti'], value='4')
    df=df.replace(regex=['touko'], value='5')
    df=df.replace(regex=['kesä'], value='6')
    df=df.replace(regex=['heinä'], value='7')
    df=df.replace(regex=['elo'], value='8')
    df=df.replace(regex=['syys'], value='9')
    df=df.replace(regex=['loka'], value='10')
    df=df.replace(regex=['marras'], value='11')
    df=df.replace(regex=['joulu'], value='12')

    df=df.replace(regex=['ma'], value='Mon')
    df=df.replace(regex=['ti'], value='Tue')
    df=df.replace(regex=['ke'], value='Wed')
    df=df.replace(regex=['to'], value='Thu')
    df=df.replace(regex=['pe'], value='Fri')
    df=df.replace(regex=['la'], value='Sat')
    df=df.replace(regex=['su'], value='Sun')
    
    df.columns = ['Weekday', 'Day','Month','Year','Hour']
    a= df['Hour'].str.split(':',expand=True)
    df['Hour']=a.iloc[:,0]
    df['Day']= df['Day'].astype(str).astype(int)
    df['Month']= df['Month'].astype(str).astype(int)
    df['Year']= df['Year'].astype(str).astype(int)
    df['Hour']= df['Hour'].astype(str).astype(int)
    
    
    #print(df.dtypes)
    return df

def main():
    split_date()
    return
       #ke 1 tammi 2014 00:00 to Wed 1 1 2014 0 .
if __name__ == "__main__":
    main()



