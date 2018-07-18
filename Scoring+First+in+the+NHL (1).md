---
layout: teaching
title: "Scoring first in the NHL"
date: 2018-07-18
---
# Scoring First in the NHL

The data for this analysis were collected in another section. I separated this to keep things a little shorter. I'm working with SAS university edition, which includes a Jupyter Notebook and access to SAS through Python or using magics %% to flag SAS code. I'll primarily work with SAS code. Unfortunately, I can't do all the steps in SAS University edition, since I cannot install external libraries onto their virtual machine. If the library is pure python, I could probably use it locally, but that is not the case with things like sklearn and matplotlib. I mostly wanted to use SAS for the multilevel modeling capabilities. Otherwise sklearn or statsmodel could be used.


I've stored the data as a csv file to load into SAS, but I'll load it through a pandas data frame initially.


```python
import saspy
import pandas as pd
import numpy as np
from IPython.display import HTML
import math
```


```python
df = pd.read_csv(#rename if you need to
    ,index_col=0) 
df['notwon'] = np.where(df['won']==1,0,1)
df['penalty_cen'] = df['penaltycount']-3.98446
print(df.head())
print(df.describe())
print(df['team'].unique())
#to fix the arizona team
#for i,row in df.iterrows():
#    if row['team']=='ARI':
#        df.set_value(i,'team','PHX')
```

Create a sas session and load the dataframe into SAS for analysis.


```python
sas_session = saspy.SASsession()
sas_session.saslib("nhl",path="/folders/myfolders/nhl/Update20180709")
sas_session.teach_me_SAS(False)
games = sas_session.df2sd(df,table='games',libref='nhl')
print(games.describe())
games.head()
```

    Using SAS Config named: default
    SAS Connection established. Subprocess id is 5436
    
    
    27   
    28   libname nhl    '/folders/myfolders/nhl/Update20180709'  ;
    NOTE: Libref NHL was successfully assigned as follows: 
          Engine:        V9 
          Physical Name: /folders/myfolders/nhl/Update20180709
    29   
    30   
            Variable      N  NMiss        Median          Mean        StdDev  \
    0        gamekey  12356      0  2.016290e+07  5.380185e+07  7.256136e+07   
    1          goals  12356      0  3.000000e+00  2.785529e+00  1.621079e+00   
    2           home  12356      0  5.000000e-01  5.000000e-01  5.000202e-01   
    3       overtime  12356      0  0.000000e+00  1.317579e-01  3.382410e-01   
    4   penaltycount  12356      0  4.000000e+00  3.984461e+00  2.089983e+00   
    5         period  12356      0  1.000000e+00  1.248786e+00  5.256075e-01   
    6    scoredfirst  12356      0  5.000000e-01  5.000000e-01  5.000202e-01   
    7   scoredsecond  12356      0  0.000000e+00  4.876174e-01  4.998669e-01   
    8            won  12356      0  5.000000e-01  5.000000e-01  5.000202e-01   
    9         notwon  12356      0  5.000000e-01  5.000000e-01  5.000202e-01   
    10   penalty_cen  12356      0  1.554000e-02  9.906119e-07  2.089983e+00   
    
                 Min           P25           P50           P75           Max  
    0   201421.00000  2.015226e+07  2.016290e+07  2.018265e+07  2.018213e+08  
    1        0.00000  2.000000e+00  3.000000e+00  4.000000e+00  1.000000e+01  
    2        0.00000  0.000000e+00  5.000000e-01  1.000000e+00  1.000000e+00  
    3        0.00000  0.000000e+00  0.000000e+00  0.000000e+00  1.000000e+00  
    4        0.00000  3.000000e+00  4.000000e+00  5.000000e+00  2.000000e+01  
    5        1.00000  1.000000e+00  1.000000e+00  1.000000e+00  4.000000e+00  
    6        0.00000  0.000000e+00  5.000000e-01  1.000000e+00  1.000000e+00  
    7        0.00000  0.000000e+00  0.000000e+00  1.000000e+00  1.000000e+00  
    8        0.00000  0.000000e+00  5.000000e-01  1.000000e+00  1.000000e+00  
    9        0.00000  0.000000e+00  5.000000e-01  1.000000e+00  1.000000e+00  
    10      -3.98446 -9.844600e-01  1.554000e-02  1.015540e+00  1.601554e+01  





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gamekey</th>
      <th>goals</th>
      <th>home</th>
      <th>overtime</th>
      <th>penaltycount</th>
      <th>period</th>
      <th>scoredfirst</th>
      <th>scoredsecond</th>
      <th>team</th>
      <th>won</th>
      <th>notwon</th>
      <th>penalty_cen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>201421</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>14</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>MTL</td>
      <td>0</td>
      <td>1</td>
      <td>10.01554</td>
    </tr>
    <tr>
      <th>1</th>
      <td>201421</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>12</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>TOR</td>
      <td>1</td>
      <td>0</td>
      <td>8.01554</td>
    </tr>
    <tr>
      <th>2</th>
      <td>201422</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>6</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>CHI</td>
      <td>1</td>
      <td>0</td>
      <td>2.01554</td>
    </tr>
    <tr>
      <th>3</th>
      <td>201422</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>WSH</td>
      <td>0</td>
      <td>1</td>
      <td>0.01554</td>
    </tr>
    <tr>
      <th>4</th>
      <td>201423</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>6</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>EDM</td>
      <td>0</td>
      <td>1</td>
      <td>2.01554</td>
    </tr>
  </tbody>
</table>
</div>



One of the problems with the dataset is that really each observation is not independent. The value of won depends on who one. Know one of these you know who lost the game. I won't go into the details here, but I experimented with a couple of ways to approach this problem. For one, I decided to look at a multilevel approach where games were nested within teams, a sort of cross classification problem. This didn't work out that well because of the small number of samples per gamekey. Often, it had trouble estimated the covariance matrix, although it would converge.

Instead I chose alternative route of sampling a team from each game - stratified random sampling; where the gamekey was the strata.

Regardless of the approach, the estimated parameters were usually close. Here is an example of a random sample and the full model.


```python
def convertToProb(logit):
    odds = math.exp(logit)
    return odds/(1.0+odds)


```


```python
%%SAS sas_session
proc sort data=nhl.games;
 by gamekey;
run;
Proc SurveySelect data=nhl.games out=nhl.gamesrand noprint Method = urs N = 1 outhits
    rep = 1;
    Strata gamekey;
run;
proc freq data=nhl.gamesrand;
    table won;
run;
proc print data=nhl.gamesrand(obs=10);
run;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The FREQ Procedure</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="One-Way Frequencies">
<caption aria-label="One-Way Frequencies"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r b header" scope="col">won</th>
<th class="r b header" scope="col">Frequency</th>
<th class="r b header" scope="col">Percent</th>
<th class="r b header" scope="col">Cumulative<br/>Frequency</th>
<th class="r b header" scope="col">Cumulative<br/>Percent</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3093</td>
<td class="r data">50.06</td>
<td class="r data">3093</td>
<td class="r data">50.06</td>
</tr>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="r data">3085</td>
<td class="r data">49.94</td>
<td class="r data">6178</td>
<td class="r data">100.00</td>
</tr>
</tbody>
</table>
</div>
</div>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<hr class="pagebreak"/>
<div id="IDX1" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Data Set NHL.GAMESRAND">
<caption aria-label="Data Set NHL.GAMESRAND"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r header" scope="col">Obs</th>
<th class="r header" scope="col">gamekey</th>
<th class="r header" scope="col">Replicate</th>
<th class="r header" scope="col">goals</th>
<th class="r header" scope="col">home</th>
<th class="r header" scope="col">overtime</th>
<th class="r header" scope="col">penaltycount</th>
<th class="r header" scope="col">period</th>
<th class="r header" scope="col">scoredfirst</th>
<th class="r header" scope="col">scoredsecond</th>
<th class="header" scope="col">team</th>
<th class="r header" scope="col">won</th>
<th class="r header" scope="col">notwon</th>
<th class="r header" scope="col">penalty_cen</th>
<th class="r header" scope="col">NumberHits</th>
<th class="r header" scope="col">ExpectedHits</th>
<th class="r header" scope="col">SamplingWeight</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="r data">201421</td>
<td class="r data">1</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<td class="r data">201422</td>
<td class="r data">1</td>
<td class="r data">6</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">6</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">CHI</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">2.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">3</th>
<td class="r data">201423</td>
<td class="r data">1</td>
<td class="r data">4</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">6</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">EDM</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">2.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">4</th>
<td class="r data">201424</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">4</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">PHI</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">0.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">5</th>
<td class="r data">201425</td>
<td class="r data">1</td>
<td class="r data">2</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">7</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="data">DET</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">3.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">6</th>
<td class="r data">201426</td>
<td class="r data">1</td>
<td class="r data">6</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">8</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="data">COL</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">4.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">7</th>
<td class="r data">201427</td>
<td class="r data">1</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">7</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">BOS</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">3.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">8</th>
<td class="r data">201428</td>
<td class="r data">1</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="data">PIT</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data" style="white-space: nowrap">-0.9845</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">9</th>
<td class="r data">201429</td>
<td class="r data">1</td>
<td class="r data">4</td>
<td class="r data">0</td>
<td class="r data">0</td>
<td class="r data">5</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="data">CGY</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">1.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">10</th>
<td class="r data">201521</td>
<td class="r data">1</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">2</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">TOR</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-1.9845</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




This means that about 50% of the sample are wins. A 50/50 chance of picking one or the other in each game.

Looking solely at the scoring first variable, we get an estimate of the effect of scoring first has on winning. This does a better job predicting as compared to the intercept only model. The odds ratio is pretty high at 4.6 (versus not scoring first). I find plugging in the estimates to be a useful exercise. And you can see this below. Not surprisingly the probability is close to the proportion of wins where the winner scored first at 68%. This will change when we start to control for other factors. Next I will fit the full model.


```python
%%SAS sas_session
proc logistic data=nhl.gamesrand;
class scoredfirst(ref='0') /param=ref;
model won(event='1')= scoredfirst/;
run; quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The LOGISTIC Procedure</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Model Information">
<caption aria-label="Model Information"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Model Information</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Data Set</th>
<td class="data">NHL.GAMESRAND</td>
</tr>
<tr>
<th class="rowheader" scope="row">Response Variable</th>
<td class="data">won</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Response Levels</th>
<td class="data">2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Model</th>
<td class="data">binary logit</td>
</tr>
<tr>
<th class="rowheader" scope="row">Optimization Technique</th>
<td class="data">Fisher&apos;s scoring</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX1" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Observations Summary">
<caption aria-label="Observations Summary"></caption>
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Number of Observations Read</th>
<td class="r data">6178</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Observations Used</th>
<td class="r data">6178</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX2" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Response Profile">
<caption aria-label="Response Profile"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Response Profile</th>
</tr>
<tr>
<th class="r b header" scope="col">Ordered<br/>Value</th>
<th class="b header" scope="col">won</th>
<th class="r b header" scope="col">Total<br/>Frequency</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="data">0</td>
<td class="r data">3071</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<td class="data">1</td>
<td class="r data">3107</td>
</tr>
</tbody>
</table>
<div class="proc_note_group">
<p class="c proctitle">Probability modeled is won=1.</p>
</div>
</div>
<div id="IDX3" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Class Level Information">
<caption aria-label="Class Level Information"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Class Level Information</th>
</tr>
<tr>
<th class="b header" scope="col">Class</th>
<th class="b header" scope="col">Value</th>
<th class="c b header" scope="col">Design Variables</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="rowheader" scope="row">0</th>
<td class="r data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">&#160;</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX4" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Convergence Status">
<caption aria-label="Convergence Status"></caption>
<colgroup><col/></colgroup>
<thead>
<tr>
<th class="c b header" scope="col">Model Convergence Status</th>
</tr>
</thead>
<tbody>
<tr>
<td class="c data">Convergence criterion (GCONV=1E-8) satisfied.</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX5" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Fit Statistics">
<caption aria-label="Fit Statistics"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Model Fit Statistics</th>
</tr>
<tr>
<th class="b header" scope="col">Criterion</th>
<th class="r b header" scope="col">Intercept Only</th>
<th class="r b header" scope="col">Intercept and Covariates</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">AIC</th>
<td class="r data">8566.317</td>
<td class="r data">7728.093</td>
</tr>
<tr>
<th class="rowheader" scope="row">SC</th>
<td class="r data">8573.046</td>
<td class="r data">7741.551</td>
</tr>
<tr>
<th class="rowheader" scope="row">-2 Log L</th>
<td class="r data">8564.317</td>
<td class="r data">7724.093</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX6" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Global Tests">
<caption aria-label="Global Tests"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Testing Global Null Hypothesis: BETA=0</th>
</tr>
<tr>
<th class="b header" scope="col">Test</th>
<th class="r b header" scope="col">Chi-Square</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Likelihood Ratio</th>
<td class="r data">840.2236</td>
<td class="r data">1</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">Score</th>
<td class="r data">820.9889</td>
<td class="r data">1</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">Wald</th>
<td class="r data">782.1602</td>
<td class="r data">1</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX7" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Type 3 Tests">
<caption aria-label="Type 3 Tests"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Type 3 Analysis of Effects</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Wald<br/>Chi-Square</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<td class="r data">1</td>
<td class="r data">782.1602</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX8" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Parameter Estimates">
<caption aria-label="Parameter Estimates"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="7" scope="colgroup">Analysis of Maximum Likelihood Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Parameter</th>
<th class="b header" scope="col"> &#160;</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="r b header" scope="col">Wald<br/>Chi-Square</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="rowheader" scope="row">&#160;</th>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-0.7489</td>
<td class="r data">0.0385</td>
<td class="r data">378.5074</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
<td class="r data">1.5285</td>
<td class="r data">0.0547</td>
<td class="r data">782.1602</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX9" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Odds Ratios">
<caption aria-label="Odds Ratios"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Odds Ratio Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">Point Estimate</th>
<th class="c b header" colspan="2" scope="colgroup">95% Wald<br/>Confidence Limits</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst 1 vs 0</th>
<td class="r data">4.611</td>
<td class="r data">4.143</td>
<td class="r data">5.133</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX10" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Association Statistics">
<caption aria-label="Association Statistics"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Association of Predicted Probabilities and Observed Responses</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Percent Concordant</th>
<td class="r data">46.5</td>
<th class="rowheader" scope="row">Somers&apos; D</th>
<td class="r data">0.365</td>
</tr>
<tr>
<th class="rowheader" scope="row">Percent Discordant</th>
<td class="r data">10.1</td>
<th class="rowheader" scope="row">Gamma</th>
<td class="r data">0.644</td>
</tr>
<tr>
<th class="rowheader" scope="row">Percent Tied</th>
<td class="r data">43.4</td>
<th class="rowheader" scope="row">Tau-a</th>
<td class="r data">0.182</td>
</tr>
<tr>
<th class="rowheader" scope="row">Pairs</th>
<td class="r data">9541597</td>
<th class="rowheader" scope="row">c</th>
<td class="r data">0.682</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
print("Probability of winning when scoring first: ")
print("{0:.2f}%".format(convertToProb(-.7489+1.5285)*100))
```

    Probability of winning when scoring first: 
    68.56%


The model certainly performs better than the intercept only model, and converged properly.  Overtime is the only non-significant factor, and so one of the other variables explains the same information as the overtime. That could be the period scored. It may be worth looking later which variables are added that overlap with overtime. As expected our scoredfirst variable is important, and significant. Penalties were centered around the mean of approximately 4 penalties (3.9...), and so a 0 would indicate 4. 



```python
%%SAS sas_session
proc logistic data=nhl.gamesrand;
class scoredfirst(ref='0') scoredsecond(ref='0') home(ref='0')overtime(ref='0')/param=ref;
model won(event='1')= scoredfirst scoredsecond home overtime period penalty_cen/;
run; quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The LOGISTIC Procedure</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Model Information">
<caption aria-label="Model Information"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Model Information</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Data Set</th>
<td class="data">NHL.GAMESRAND</td>
</tr>
<tr>
<th class="rowheader" scope="row">Response Variable</th>
<td class="data">won</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Response Levels</th>
<td class="data">2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Model</th>
<td class="data">binary logit</td>
</tr>
<tr>
<th class="rowheader" scope="row">Optimization Technique</th>
<td class="data">Fisher&apos;s scoring</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX1" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Observations Summary">
<caption aria-label="Observations Summary"></caption>
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Number of Observations Read</th>
<td class="r data">6178</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Observations Used</th>
<td class="r data">6178</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX2" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Response Profile">
<caption aria-label="Response Profile"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Response Profile</th>
</tr>
<tr>
<th class="r b header" scope="col">Ordered<br/>Value</th>
<th class="b header" scope="col">won</th>
<th class="r b header" scope="col">Total<br/>Frequency</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="data">0</td>
<td class="r data">3071</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<td class="data">1</td>
<td class="r data">3107</td>
</tr>
</tbody>
</table>
<div class="proc_note_group">
<p class="c proctitle">Probability modeled is won=1.</p>
</div>
</div>
<div id="IDX3" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Class Level Information">
<caption aria-label="Class Level Information"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Class Level Information</th>
</tr>
<tr>
<th class="b header" scope="col">Class</th>
<th class="b header" scope="col">Value</th>
<th class="c b header" scope="col">Design Variables</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="rowheader" scope="row">0</th>
<td class="r data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">&#160;</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredsecond</th>
<th class="rowheader" scope="row">0</th>
<td class="r data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">&#160;</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
</tr>
<tr>
<th class="rowheader" scope="row">home</th>
<th class="rowheader" scope="row">0</th>
<td class="r data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">&#160;</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
</tr>
<tr>
<th class="rowheader" scope="row">overtime</th>
<th class="rowheader" scope="row">0</th>
<td class="r data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">&#160;</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX4" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Convergence Status">
<caption aria-label="Convergence Status"></caption>
<colgroup><col/></colgroup>
<thead>
<tr>
<th class="c b header" scope="col">Model Convergence Status</th>
</tr>
</thead>
<tbody>
<tr>
<td class="c data">Convergence criterion (GCONV=1E-8) satisfied.</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX5" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Fit Statistics">
<caption aria-label="Fit Statistics"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Model Fit Statistics</th>
</tr>
<tr>
<th class="b header" scope="col">Criterion</th>
<th class="r b header" scope="col">Intercept Only</th>
<th class="r b header" scope="col">Intercept and Covariates</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">AIC</th>
<td class="r data">8566.317</td>
<td class="r data">6727.321</td>
</tr>
<tr>
<th class="rowheader" scope="row">SC</th>
<td class="r data">8573.046</td>
<td class="r data">6774.422</td>
</tr>
<tr>
<th class="rowheader" scope="row">-2 Log L</th>
<td class="r data">8564.317</td>
<td class="r data">6713.321</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX6" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Global Tests">
<caption aria-label="Global Tests"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Testing Global Null Hypothesis: BETA=0</th>
</tr>
<tr>
<th class="b header" scope="col">Test</th>
<th class="r b header" scope="col">Chi-Square</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Likelihood Ratio</th>
<td class="r data">1850.9960</td>
<td class="r data">6</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">Score</th>
<td class="r data">1648.0410</td>
<td class="r data">6</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">Wald</th>
<td class="r data">1235.9008</td>
<td class="r data">6</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX7" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Type 3 Tests">
<caption aria-label="Type 3 Tests"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Type 3 Analysis of Effects</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Wald<br/>Chi-Square</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<td class="r data">1</td>
<td class="r data">870.7582</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredsecond</th>
<td class="r data">1</td>
<td class="r data">814.4736</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">home</th>
<td class="r data">1</td>
<td class="r data">29.2964</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">overtime</th>
<td class="r data">1</td>
<td class="r data">1.4692</td>
<td class="r data">0.2255</td>
</tr>
<tr>
<th class="rowheader" scope="row">period</th>
<td class="r data">1</td>
<td class="r data">9.4676</td>
<td class="r data">0.0021</td>
</tr>
<tr>
<th class="rowheader" scope="row">penalty_cen</th>
<td class="r data">1</td>
<td class="r data">10.0272</td>
<td class="r data">0.0015</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX8" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Parameter Estimates">
<caption aria-label="Parameter Estimates"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="7" scope="colgroup">Analysis of Maximum Likelihood Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Parameter</th>
<th class="b header" scope="col"> &#160;</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="r b header" scope="col">Wald<br/>Chi-Square</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="rowheader" scope="row">&#160;</th>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-2.2164</td>
<td class="r data">0.1022</td>
<td class="r data">470.7539</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
<td class="r data">1.9117</td>
<td class="r data">0.0648</td>
<td class="r data">870.7582</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredsecond</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
<td class="r data">1.8564</td>
<td class="r data">0.0650</td>
<td class="r data">814.4736</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">home</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
<td class="r data">0.3241</td>
<td class="r data">0.0599</td>
<td class="r data">29.2964</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">overtime</th>
<th class="rowheader" scope="row">1</th>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-0.1016</td>
<td class="r data">0.0838</td>
<td class="r data">1.4692</td>
<td class="r data">0.2255</td>
</tr>
<tr>
<th class="rowheader" scope="row">period</th>
<th class="rowheader" scope="row">&#160;</th>
<td class="r data">1</td>
<td class="r data">0.1746</td>
<td class="r data">0.0567</td>
<td class="r data">9.4676</td>
<td class="r data">0.0021</td>
</tr>
<tr>
<th class="rowheader" scope="row">penalty_cen</th>
<th class="rowheader" scope="row">&#160;</th>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-0.0463</td>
<td class="r data">0.0146</td>
<td class="r data">10.0272</td>
<td class="r data">0.0015</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX9" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Odds Ratios">
<caption aria-label="Odds Ratios"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Odds Ratio Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">Point Estimate</th>
<th class="c b header" colspan="2" scope="colgroup">95% Wald<br/>Confidence Limits</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst  1 vs 0</th>
<td class="r data">6.764</td>
<td class="r data">5.958</td>
<td class="r data">7.680</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredsecond 1 vs 0</th>
<td class="r data">6.401</td>
<td class="r data">5.635</td>
<td class="r data">7.271</td>
</tr>
<tr>
<th class="rowheader" scope="row">home         1 vs 0</th>
<td class="r data">1.383</td>
<td class="r data">1.230</td>
<td class="r data">1.555</td>
</tr>
<tr>
<th class="rowheader" scope="row">overtime     1 vs 0</th>
<td class="r data">0.903</td>
<td class="r data">0.767</td>
<td class="r data">1.065</td>
</tr>
<tr>
<th class="rowheader" scope="row">period</th>
<td class="r data">1.191</td>
<td class="r data">1.065</td>
<td class="r data">1.331</td>
</tr>
<tr>
<th class="rowheader" scope="row">penalty_cen</th>
<td class="r data">0.955</td>
<td class="r data">0.928</td>
<td class="r data">0.983</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX10" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Association Statistics">
<caption aria-label="Association Statistics"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Association of Predicted Probabilities and Observed Responses</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Percent Concordant</th>
<td class="r data">78.6</td>
<th class="rowheader" scope="row">Somers&apos; D</th>
<td class="r data">0.580</td>
</tr>
<tr>
<th class="rowheader" scope="row">Percent Discordant</th>
<td class="r data">20.7</td>
<th class="rowheader" scope="row">Gamma</th>
<td class="r data">0.584</td>
</tr>
<tr>
<th class="rowheader" scope="row">Percent Tied</th>
<td class="r data">0.7</td>
<th class="rowheader" scope="row">Tau-a</th>
<td class="r data">0.290</td>
</tr>
<tr>
<th class="rowheader" scope="row">Pairs</th>
<td class="r data">9541597</td>
<th class="rowheader" scope="row">c</th>
<td class="r data">0.790</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





What is the probability of wining a game given these factors?

It can be broken up in many different ways.
Probability of winning all else being zero: 
9.83%
Probability of winning when scoring first in the first period, not at home, no overtime, and ~4 penalties: 
46.75%
Probability of winning when scoring first at home, in the first period, no overtime, and ~4 penalties: 
54.83%
Probability of winning when scoring first  in the second period,not at home, no overtime, and ~4 penalties: 
51.11%
Probability of winning when scoring first in the first period, scoring second, not at home,  no overtime, and ~4 penalties: 
84.89%
Probability of winning when scoring first in the second period, scoring second, at home,  no overtime, and ~4 penalties: 
87.00%



```python
print("Probability of winning all else being zero: ")
print("{0:.2f}%".format(convertToProb(-2.2164)*100))

print("Probability of winning when scoring first not at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.2164+1.9117+0.1746)*100))

print("Probability of winning when scoring first at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.2164+1.9117+.3241+0.1746)*100))

print("Probability of winning when scoring first not at home, in the second period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.2164+1.9117+2*0.1746)*100))

print("Probability of winning when scoring first and second not at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.2164+1.9117+1.8564+0.1746)*100))

print("Probability of winning when scoring first and second at home, in the second period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.2164+1.9117+1.8564+2*0.1746)*100))
```

So when you control for many of the other factors scoring first is no garauntee of winning. Having a home game and scoring first gives you a slightly better chance over. It really isn't till you are the team that scores second does it become more than likely that you will. Again, that makes sense because it becomes all the more harder to beat 2 goals than just one. 

Ultimately, it is the team that scores the first two goals that tend to go on to win. So while scoring first is important, you absolutely need to step on the gas and get that second goal.

But this is just one sample from the population of games from 2013-2014 to 2017-2018. Could this just have been a fluke of the sample that was drawn. Actually, because I didn't set the seed, if you are rerunning this code, you probably got slightly different estimates. 

Let's bootstrap the process and look at the sampling distribution of the estimates to get a better picture of the possible win probabilities. In SAS this can be done more efficiently using the survey select process, and then the by statement in the logistic process.



```python
%%SAS sas_session
proc sort data=nhl.games;
 by gamekey;
run;
proc surveyselect data=nhl.games NOPRINT
     out=nhl.gamessamp
     noprint Method = urs N = 1 outhits               
     reps=100;  /* generate this many bootstrap resamples */
     Strata gamekey;
run;

proc print data=nhl.gamessamp(obs=10);
run;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Data Set NHL.GAMESSAMP">
<caption aria-label="Data Set NHL.GAMESSAMP"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r header" scope="col">Obs</th>
<th class="r header" scope="col">gamekey</th>
<th class="r header" scope="col">Replicate</th>
<th class="r header" scope="col">goals</th>
<th class="r header" scope="col">home</th>
<th class="r header" scope="col">overtime</th>
<th class="r header" scope="col">penaltycount</th>
<th class="r header" scope="col">period</th>
<th class="r header" scope="col">scoredfirst</th>
<th class="r header" scope="col">scoredsecond</th>
<th class="header" scope="col">team</th>
<th class="r header" scope="col">won</th>
<th class="r header" scope="col">notwon</th>
<th class="r header" scope="col">penalty_cen</th>
<th class="r header" scope="col">NumberHits</th>
<th class="r header" scope="col">ExpectedHits</th>
<th class="r header" scope="col">SamplingWeight</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="r data">201421</td>
<td class="r data">1</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<td class="r data">201421</td>
<td class="r data">2</td>
<td class="r data">4</td>
<td class="r data">0</td>
<td class="r data">0</td>
<td class="r data">12</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">TOR</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">8.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">3</th>
<td class="r data">201421</td>
<td class="r data">3</td>
<td class="r data">4</td>
<td class="r data">0</td>
<td class="r data">0</td>
<td class="r data">12</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">TOR</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">8.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">4</th>
<td class="r data">201421</td>
<td class="r data">4</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">5</th>
<td class="r data">201421</td>
<td class="r data">5</td>
<td class="r data">4</td>
<td class="r data">0</td>
<td class="r data">0</td>
<td class="r data">12</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">TOR</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">8.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">6</th>
<td class="r data">201421</td>
<td class="r data">6</td>
<td class="r data">4</td>
<td class="r data">0</td>
<td class="r data">0</td>
<td class="r data">12</td>
<td class="r data">1</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="data">TOR</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">8.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">7</th>
<td class="r data">201421</td>
<td class="r data">7</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">8</th>
<td class="r data">201421</td>
<td class="r data">8</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">9</th>
<td class="r data">201421</td>
<td class="r data">9</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
<tr>
<th class="r rowheader" scope="row">10</th>
<td class="r data">201421</td>
<td class="r data">10</td>
<td class="r data">3</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">14</td>
<td class="r data">1</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="data">MTL</td>
<td class="r data">0</td>
<td class="r data">1</td>
<td class="r data">10.0155</td>
<td class="r data">1</td>
<td class="r data">0.5</td>
<td class="r data">2</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
%%SAS sas_session
proc sort data=nhl.gamessamp;
 by replicate;
run;
ods trace on;
ods output ParameterEstimates=nhl.estimates;
proc logistic data=nhl.gamessamp;
by replicate;
class scoredfirst(ref='0') scoredsecond(ref='0') home(ref='0')overtime(ref='0')/param=ref;
model won(event='1')= scoredfirst scoredsecond home overtime period penalty_cen/;
run; quit;
ods trace off;

```

I cleared the outputs so you didn't have to see each iteration of the logistic step. Instead we can summarize the data by looking at it visually or by the mean of the sample distribution.


```python
%%SAS sas_session
proc sql;
    select Variable, mean(Estimate) as MeanEst
      from nhl.estimates
      group by variable;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="r b header" scope="col">MeanEst</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">Intercept</td>
<td class="r data" style="white-space: nowrap">-2.14334</td>
</tr>
<tr>
<td class="data">home</td>
<td class="r data">0.342078</td>
</tr>
<tr>
<td class="data">overtime</td>
<td class="r data" style="white-space: nowrap">-0.01201</td>
</tr>
<tr>
<td class="data">penalty_cen</td>
<td class="r data" style="white-space: nowrap">-0.02247</td>
</tr>
<tr>
<td class="data">period</td>
<td class="r data">0.10468</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="r data">1.90498</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="r data">1.836549</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
%%SAS sas_session
%MACRO histograms(curvar=);
    %let titlevar = &curvar;
    proc template;
        define statgraph Histogram;
            begingraph;
                entrytitle "Histogram of &titlevar ";
                layout lattice/columns=1;
                    layout overlay;
                        histogram Estimate;
                    endlayout;
                    layout overlay;
                        boxplot y=Estimate / orient=horizontal;
                    endlayout;
                endlayout;
            endgraph;
        end;
    run;

    proc sgrender data=nhl.estimates(where=(variable=&curvar)) template = Histogram;
    run;
%MEND histograms;
%histograms(curvar='scoredfirst')
%histograms(curvar='scoredsecond')
%histograms(curvar='home')

```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div class="c">
<img style="height: 480px; width: 640px" alt="The SGRender Procedure" src="data:image/png;base64,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cHR0NHkjVptPzXzgCzTEtNFdhBCRa0+/9GTfDe+kkL5Ay3U/0auy2gAGLJOawS+9fn7N/IdIX6A1I4CB5jZtdJdh/b06eXFdK9CqWeHfAQOWj/QFQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAWw2dlqblA8iZFcBwNo87N/2/bUjZFcB8yOAzalnZ3fZJQCwNk4O2k5efLWfFSKAAQCQgAAGAEACAhgAAAkIYAAAJCCAAQCQgAAGAEACAhgAAAkIYAAAJCCAAQCQgAAGAEACrewCgCaRd/Pm/P+bKruKn1Up9jbVg+f/35uyC6nD7du3NbJrAFohAhjW6bPPDskuoU5jZBdQN0dHR9klAK0OAQzrFBwcLLsEAKgPnwEDACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACABAQwAgAQEMAAAEhDAAABIQAADACCBVnYB98vVxSXu7XdlVwEAwM9cXXX3XKbFB/D0iN/KLqHFOHz0WI9uXbv4dpZdCMS/L39fUHTr0aFDZBcCUazXHzpydNqUMNmFQAgh9nxwYNL4MS66e6eXFeAUNAAAEhDAAABIQAADACABAdyKuLq42Nvbya4CQgjh4OCgc3aWXQWEEMLGxtbD3U12FfgvD3c3Wxtb2VU0E01SUlJAQIDsMgAAaEWSk5M5AgYAQAICGAAACQhgAAAkIIABAJCAAG7x9Hp9QkKCn5/f/v3773x137593bt31+l0M2bMKC8vVwcTExP9/f0dHR0nTpxYXFxczyAay1zTodVqNf8TFsaXNJnChLmocxV2jftnrrmwsv2CAG7xQkJCoqOjbWzqmMqMjIynnnrqrbfeysjIuHr16rp164QQFRUVERERUVFR2dnZDg4OS5cuvdsgTGCW6RBCeHh4KP/zz3/+s1l7sBaNnYs6V2HXMAuzzIWwvv0iKSlJQcs3evToffv21Rrctm1beHi4+vjkyZM+Pj6Konz55ZedO3dWBy9cuNCuXbu7DcJk9zkdiqJ4eno2V7FWruFzUecq7BpmdJ9zoVjXfpGUlMQRsDUzGAzGx3379s3JySkvL09LS+vbt6862KtXr7y8PL1eX+eghIqtWsOnQwhhZ2fXrl07T0/P2bNnFxUVyanYetU5F3Uuya7R1Bo+F8Lq9gsC2JqNHDkyISHh+PHjt27dioqKEkIUFhaWl5e7urqqCzg5OWm12rKysjoHpdVtpRo+HUKI3NzcvLy8lJSUwsLCefPmyazbGtU5F3Uuya7R1Bo+F8Lq9gsC2Jr5+/vHxsbOnj27S5cuHTt2tLGxcXd31+l0JSUl6gIGg8FgMOh0ujoH5RVunRo+HcZVvL29N2zYcPDgQUVRJFVtneqcizqXZNdoag2fCyOr2S8IYCs3Y8aMrKysoqKiSZMmde/e3cXFpUePHikpKeqrqampXl5erq6udQ7Kq9pqNXA6aq5SWVlpY2Oj0Whk1GvN7pyLOhdj12gGDZyLmqxjvyCArVlxcfHUqVMzMzMzMzOfe+65Z555RggRGBhoMBhiY2MLCgpWrlw5efLkuw3CvBo+HVu3bl23bl1OTk5OTs6iRYvCw8Nl125t6pyLOrFrNLWGz4X17RcEsHV6+OGHL1y44OLi0q9fv4EDBw4YMCA4OHjBggVCCDs7u/j4+JiYGB8fn9LS0rVr195tEObS2OkICwtLTU0dNGhQv379OnXqtHnzZtkdWI965qJO7BpNp7FzYX37BXdDAgCguXE3JAAA5CCAAQCQgAAGAEACAhgAAAkIYAAAJCCAAQCQgAAGAEACAhgAAAkIYAAAJCCAAQCQgAAGAEACAhgAAAkIYKBlc3Fx0fxScHDw3RY2GAwajeb69etCiJs3b/bp06eiosK09625KQAm0MouAMD9Onv27MCBAxu7Vvv27S9evNgU9QBoCI6AAav18ssv+/j4ODk5TZo0KSMjQwih5rS3t/fu3buzs7M1Go0QIjMzU6vVbtu2zdfXt0OHDgcPHty4caOnp2fnzp0TEhLUTX3yySePPPKIk5OTr6/vjh07am1KCHHx4sXg4GA3N7cBAwacOXNGVstAC0IAA9bp5MmTcXFxx48fz83NDQ0N/eijj4QQ586dE0Lk5ubOnDmz5sJVVVVpaWnffPPNwoULp02bdvny5YyMjKeffvr5559XF9iyZUt0dHRhYeH27dvnzZuXm5tbc1OlpaWhoaEhISE5OTmLFy8eP358UVFRs3cMtDRJSUkKgBZLp9PV2qlPnTqlKEpKSkqbNm127dqVn59vXLiyslIIkZubqyjK1atXhRCKovzwww+2trbqAqmpqba2tgaDQVGU77//3tnZ+c539PX1PXbsWM1NxcfH9+zZ07hAaGjo9u3bm7BnoOVLSkriCBho8c6ePVtzxw4MDBRC9OnT58MPPzx8+HDv3r1DQ0O/++67hmzKzs5OCGFrayuEsLe3r6qqUsd3794dEhLSuXNnnU6XlZWlpq9RdnZ2amqq8SqwI0eOpKenm7lJwOpwERZgtYYPHz58+PCqqqpNmzY98cQTKSkppm3n448/XrJkyZYtWwIDAx944IH+/fvXWsDX13fYsGGJiYn3XTLQinAEDFin8+fPz5w589KlS5WVlU5OTuqhrVar1el06gVZDZeVldWzZ081y9esWXPlyhWDwVBzU2PGjMnOzn7ttdeKi4uvXr26fv361atXN0lXgBUhgIEWb9CgQTX/DtjPz08I0bt3765du06YMKFNmzZ79uzZuXOnunBUVNTIkSNXrFjR8O3PnDnTzs6uU6dOo0eP9vLyGjFixKVLl2puSqfTHTly5IsvvujateuQIUNSUlJqXeQF4E6apKSkgIAA2WUAANCKJCcncwQMAIAEBDAAABIQwAAASEAAAwAgAQEMAIAEBDAAABIQwAAASEAAAwAgAQEMAIAEBDAAABIQwAAASEAAAwAgAQEMAIAEBDAAABIQwAAASEAAAwAggVYIkZycLLsMAABal/8P3nc24wF386EAAAAASUVORK5CYII="/>
</div>
</div>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<hr class="pagebreak"/>
<div id="IDX1" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div class="c">
<img style="height: 480px; width: 640px" alt="The SGRender Procedure" src="data:image/png;base64,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eqWkpLRVRkjT1M1jzpw5QogRI0b8+uuvbRgTcjRp83jggQc+//xzRVGcnZ2LioraNChkaEa55ObmTpgw4ZFHHmnM+g0GQ/s8Ak5NTV24cOHBgwdLS0vnzZs3bty46upq9UdlZWUxMTELFy4MDw+XGxKyNLB5bNy40Wg0BgcHP/7443JDQpY6N4+PPvrIzc1t/PjxstNBsgb+9RBCBAQErF69eu/evYqiNGZt7bOAv/3228jIyOHDh+v1+pdeeuncuXN5eXlCCJPJNG7cuOnTp8+ePVt2RkhT3+YhhNBoNEFBQc8888wPP/wgNyRkqXPz+Pvf/7579271Gtfq6urbbrtt7969spNCggb+9VBVVVU5OTmph8i/q30W8LBhw/bv35+SklJWVrZu3bouXboEBQUVFBRER0c/9dRTTzzxhOyAkKnOzWPFihUbN268cuVKbm7uqlWrRo8eLTsm5Khz89i9e7flnUP1LegHHnhAdlJIUOfmkZiYuHLlypycnJycnAULFkyZMqWRa2tvBXzPPfecPHly+PDhK1asiI2N9fPz++yzz/bs2aPRaDZs2HD8+PFHH31U/W+sVttuvwkK9Wlg84iNjT18+HBYWFhYWJivr29iYqLssGhrDWwesqNBvgY2j5iYmIyMjMGDBw8YMKBr167r1q1r5Dr5NiQAANoa34YEAIAcFDAAABJQwAAASEABAwAgAQUMAIAEFDAAABJQwAAASEABAwAgAQUMAIAEFDAAABJQwAAASEABAwAgAQUMOLAOHTpo/t2oUaPqG2w2mzUazeXLl4UQV69e7devX2VlZfMe13pVAJqHr+QDHFtycvKgQYOa+rc6d+58+vTp1sgDoJE4Agbap9dffz0oKMjd3f3BBx/MysoSQqg9HRAQsH37dqPRqH7N7cWLF7Va7dtvvx0cHNylS5e9e/euWbPG19e3W7du+/fvV1f1+eef33vvve7u7sHBwe+9916tVQkhTp8+PWrUKC8vr/Dw8OPHj8uaMuBYKGCgHTpy5MjmzZu///77vLy8qKiozz77TAiRkpIihMjLy5sxY4b14Orq6vPnz//444/z58+PjY09e/ZsVlbW008//cILL6gDNm3alJCQUFhY+M4778yZMycvL896VaWlpVFRUZGRkTk5OS+99NKECROKiorafMaAAzIYDAoAx6TX62vt0UePHlUUJT093cfHZ9u2bfn5+ZbBVVVVQoi8vDxFUbKzs4UQiqJcuHDB2dlZHZCRkeHs7Gw2mxVF+fnnnz08PG5+xODg4O+++856Vbt27erZs6dlQFRU1DvvvNOKcwbaBYPBwBEw4NiSk5Ot9+qIiAghRL9+/T755JOvvvqqT58+UVFRp06dasyqdDqdEMLZ2VkI4eLiUl1drS7fvn17ZGRkt27d9Hr9pUuX1Pa1MBqNGRkZlqvADhw4kJmZaeNJAu0RF2EB7dPIkSNHjhxZXV29du3aRx55JD09vXnr2bNnz8KFCzdt2hQREXHbbbcNHDiw1oDg4ODhw4cfOnSoxZGBWwtHwEA7lJqaOmPGjDNnzlRVVbm7u6uHtlqtVq/XqxdkNd6lS5d69uypdvny5cvPnTtnNputVzV27Fij0bh+/XqTyZSdnb1q1aply5a1yqyA9oUCBhzb4MGDrX8POCQkRAjRp0+f0NDQiRMn+vj47NixY8uWLergefPmjRkzZunSpY1f/4wZM3Q6XdeuXaOjo/39/UePHn3mzBnrVen1+gMHDnzzzTehoaFDhw5NT0+vdZEXgDppDAZDWFiY7BgAANxC0tLSOAIGAEACChgAAAkoYAAAJKCAAQCQgAIGAEACChgAAAkoYAAAJKCAAQCQgAIGAEACChgAAAkoYAAAJKCAAQCQgAIGAEACChgAAAkoYAAAJKCAAQCQQCuESEtLkx0DAIBby/8DP9sxfRChjtcAAAAASUVORK5CYII="/>
</div>
</div>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<hr class="pagebreak"/>
<div id="IDX2" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div class="c">
<img style="height: 480px; width: 640px" alt="The SGRender Procedure" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAHgCAIAAAC6s0uzAAAACXBIWXMAAA7DAAAOwwHHb6hkAAAgAElEQVR4nO3de1yUZf7/8WtgBhhGDooKeIK0PGe6aFIeQoXU2gxXKdc11g77K9fK3C2zdckOa7a5WmYey2xLd3Vzt/pq66mTikdQG8SvhoqnQUg5S6AwML8/5ruzhGDMMPCZw+v52EePm4uba95z7Uzv7vseuDVGo1EBAICWpVVK9evXTzoGAABeJCMjw0c6AwAA3ogCBgBAAAUMAIAAChgAAAEUMAAAAihgAAAEUMAAAAiggAEAEEABAwAggAIGAEAABQwAgAAKGAAAARQwYLeAgACNRnPq1CnbyObNmzUazcCBA5VSNTU1gwcP7tq1a0FBgVzG5lVaWpqUlBQSEqLRaBYvXmwbX79+vUaj6dSpk/XL0NBQjUaTl5fXYsHqBABcGQUMOF91dbVSymKx3GCfuLg4jUbzxRdftFQoZ1q4cOHGjRtLS0uVUv3797eN+/n5KaV0Op1UMPEAQONRwICT+fj4pKenZ2dnt23bVjpLczlx4oRSau7cuRaL5a677rKNd+3aVSkVHR0tFUw8ANB4FDDgfLVPvR48eDAhISEsLCw0NHTYsGHr1q2rrq4eOHDgzp07lVIJCQnx8fFKqcrKyrlz53bt2tXPz69Dhw7Tp08vLi62zpafnz9p0qSgoKDIyMjFixdHRERoNBqTyWR7oN27d3fr1u3WW29VSu3cuTMpKaldu3ZBQUFDhgw5cOBA7UjvvvvubbfdFhwcPGLEiMOHDycnJ4eHh0dGRq5ater6Z9FQpNDQ0H/84x9KqZdffjk0NLT2j/To0UOj0Vhb0Gbr1q0DBw5s1arVwIEDDx06dOPJGx+1oqJi5syZkZGRAQEBQ4cOPXz4cEMBABdlNBotAOzh7+9f77spJibGukNISIhSKjc39+rVq3UqSil18uTJmJgY25ejRo2yWCwTJ06ss9uAAQMqKystFsuYMWOuf6wLFy7YHig2NlYpNWXKlOrqauuITfv27UtLS217NkSn0+Xk5NR5mg1Fqj1VSEjIDRbq+gft2rXrjSdvfNT77ruv9nhoaGhBQYFz/48Gmo/RaOQIGGhGhYWFxcXFwcHBq1evvnjx4ubNm+Pi4rp165aenm49c7tjx44vvvjCaDRu3LjRx8fnH//4R1lZ2VdffRUcHHzkyJGNGzcePXp069atPj4+n3zySWFh4euvv67RaOo8yvnz57/77ruPPvrIx8dnwoQJy5YtKywsvHjxYvfu3S9dumQ0Gm17PvTQQ3l5ec8995xSKjAwcP369fn5+V26dKmqqsrMzKw95w0iFRcXT5gwQSn10Ucf2Q5bb2Dy5Mnnz5/fv3+/j49PdnZ2Xl7eDSZvZNRvv/1206ZNbdu2TUtLKy8vf/XVV4uLi2t/HAxwfRQw4KCTJ0/a/mN206ZN9e4TGRn58ssvl5eXP/roo7feemtaWtpnn312fYNaT8wOHz48KSnJYDCMGDFiypQpSqn09PRjx44ppYYNG5aYmNi6devnn3++ffv2dX78z3/+c/fu3a3bc+fOPX78eFxcXK9evbKyspRS+fn5tj3feOON8PBw6/H3iBEjHnzwwbCwsN69eyulysvLGxnJ3oVauHBh586dBw8eHBERoZQqKytrzOQ3jmo94Zyfnz9o0KDAwMCUlBRbZsBdUMBA83rxxRePHz/++9//vqam5uWXX+7Vq9fJkyfr3dNS61PTNTU11o2qqiqllK+v7w0ewnoVWSn1/fffx8TELFmyJCMjo6SkpKH968x2g8nrjeSwOh9ObszkDUX18ann313W6+KAu6CAgWaUk5MzevToCxcuvPjii9nZ2aNGjbp48aL1Y0RarVYpdeTIEaWU9ReId+/evWHDhvLy8i+//HLt2rXW8Z49eyqldu3atWXLluLi4tdff/3SpUsNPdz27dvz8/PHjBlz+PDhhQsXBgQEOJz8BpEcntOJk1sve7dp02bLli0VFRVnz55dtGhRYmJi07MBLUYrHQDwZBkZGdu3b9++fXvtQet51N69e3/55ZezZs1avHixyWSaOHHixo0bJ02aZNttwIABEydO1Ol0w4YN27179z333POTD2f9AxRbt27dunWrbbCystKB5P369WsokgOzOX3ynj17PvbYY++9997YsWNtg7V/IQpwfRwBA81oxIgRy5cvj42NDQ0NbdWqVZ8+ff7yl788/PDDSqk//vGPY8aMCQwMzMnJKSkpWbduXUpKSnR0tFarjYyMnDZt2ldffWU9Z7t+/fpx48bp9fqbbrrprbfesn5IuN7zxiNGjHjmmWdCQkKioqLmzJljvbBqvRLsgBtEarqmT75y5cqFCxf26tXL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zZTZLt5ewBaLZffu3T169PD39x87dmxhYaF1cMCAAYcPHz59+vSDDz7Yvn37sLCwRx55pLi42GKx1DvY0Dz1DrpIZrueiItktlgsx44du/322wMCAgYPHpyZmen6mc1m85/+9KewsLA2bdr84Q9/qKmpcanMtdn+3VTnw02+vr6unzknJ2fq1KmRkZHWPUtLS10/s8ViWb16dbdu3QIDA4cPH56RkeFSmRtaUqvaBewumS0/Xvxmytx4RqORuyEBANDSuBsSAAAyKGAAAARQwAAACKCAAQAQQAEDACCAAgYAQAAFDACAAAoYAAABFDAAAAIoYAAABFDAAAAIoIABABBAAQNuqVWrVpofi4uLa2hns9ms0Wjy8vKUUpcuXerTp09lZaVjj1t7KgBNoZUOAMBBaWlpAwcOtPen2rdvf+zYsebIA8AuHAEDnubVV1/t2LGjXq+///77s7OzlVLWno6MjFy7dq3JZNJoNEqps2fParXaFStWREVFRUREbN68eeHChWFhYV26dNm2bZt1qs8///zOO+/U6/VRUVHvvfdenamUUseOHYuLiwsODo6JiTlw4IDUUwbcEQUMeJS9e/euXLly586dubm5CQkJn332mVIqPT1dKZWbmztlypTaO1dXV586derQoUMzZ86cNGnSiRMnsrOzn3jiid/97nfWHZYuXZqSklJYWLhq1arp06fn5ubWnqqsrCwhISE+Pj4nJ+e555679957i4qKWvwZA27LaDRaALgbg8FQ5728b98+i8WSmZnZpk2bDz74ID8/37ZzVVWVUio3N9disVy4cEEpZbFYzpw54+vra90hKyvL19fXbDZbLJbvvvsuMDDw+keMior65ptvak+1YcOG7t2723ZISEhYtWpVMz5nwIMYjUaOgAF3lZaWVvv9HBsbq5Tq06fPJ598sn379l69eiUkJBw9erQxU+l0OqWUr6+vUsrPz6+6uto6vnbt2vj4+C5duhgMhnPnzlnb18ZkMmVlZdk+BbZjx47Tp087+UkCnosPYQGeZvjw4cOHD6+url60aNEvf/nLzMxMx+bZtGnTrFmzli5dGhsb27p16/79+9fZISoqasiQIampqU2ODHgjjoABj3L48OEpU6YcP368qqpKr9dbD221Wq3BYLB+IKvxzp071717d2uXz5s37+TJk2azufZUY8aMMZlMb731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"/>
</div>
</div>
</div>
</body>
</html>




As expected the sampling distributions look normal. The means are not far off from the original estimates. The probabilities can be recalculated again.


|Variable | Value|
|---------|------|
|Intercept|	-2.14334|
|home | 0.342078|
|overtime|-0.01201|
|penalty_cen|	-0.02247|
|period|0.10468|
|scoredfirst|1.90498|
|scoredsecond|1.836549|




Probability of winning at home before the first period
13.14%

Probability of winning not at home before the first period
9.70%

Probability of winning when scoring first not at home, in the first period, no overtime, and ~4 penalties: 
46.66%

Probability of winning when scoring first at home, in the first period, no overtime, and ~4 penalties: 
55.19%

Probability of winning when scoring first not at home, in the second period, no overtime, and ~4 penalties: 
49.28%

Probability of winning when scoring first and second not at home, in the first period, no overtime, and ~4 penalties: 
84.59%

Probability of winning when scoring first and second at home,  no overtime, and ~4 penalties: 
88.54%


```python
print("Probability of winning at home before the first period")
print("{0:.2f}%".format(convertToProb(-2.14334+	0.342078+(3.9*-0.02247))*100))

print("Probability of winning not at home before the first period")
print("{0:.2f}%".format(convertToProb(-2.14334+(3.9*-0.02247))*100))

print("Probability of winning when scoring first not at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.14334+1.90498+0.10468)*100))

print("Probability of winning when scoring first at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.14334+1.90498+0.342078+0.10468)*100))

print("Probability of winning when scoring first not at home, in the second period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.14334+1.90498+2*0.10468)*100))

print("Probability of winning when scoring first and second not at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.14334+1.90498+1.836549+0.10468)*100))

print("Probability of winning when scoring first and second at home,  no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.14334+1.90498+0.342078+1.836549+0.10468)*100))
```

Something else to consider in terms of independence are the games and the teams. Wins might be considered nested within a team. A team playing well may be getting more wins in a row or something along those lines. We could examine this by looking at the interclass correlation. 


```python
%%SAS sas_session
proc nlmixed data=nhl.gamesrand;
    ods output ParameterEstimates=p1; 
    parms gamma00 = -1 tau00=1;
    gamma0j = gamma00 + u0j;
    eta = gamma0j;
    expEta = exp(eta);
    pij = expEta/(1+expEta);
    model scoredfirst ~ binary(pij);
    random u0j ~ normal([0],[tau00]) subject=team OUT=randout;
    estimate 'ICC' tau00/(tau00+3.29);
run;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The NLMIXED Procedure</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Specifications">
<caption aria-label="Specifications"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Specifications</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Data Set</th>
<td class="data">NHL.GAMESRAND</td>
</tr>
<tr>
<th class="rowheader" scope="row">Dependent Variable</th>
<td class="data">scoredfirst</td>
</tr>
<tr>
<th class="rowheader" scope="row">Distribution for Dependent Variable</th>
<td class="data">Binary</td>
</tr>
<tr>
<th class="rowheader" scope="row">Random Effects</th>
<td class="data">u0j</td>
</tr>
<tr>
<th class="rowheader" scope="row">Distribution for Random Effects</th>
<td class="data">Normal</td>
</tr>
<tr>
<th class="rowheader" scope="row">Subject Variable</th>
<td class="data">team</td>
</tr>
<tr>
<th class="rowheader" scope="row">Optimization Technique</th>
<td class="data">Dual Quasi-Newton</td>
</tr>
<tr>
<th class="rowheader" scope="row">Integration Method</th>
<td class="data">Adaptive Gaussian Quadrature</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX1" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Dimensions">
<caption aria-label="Dimensions"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Dimensions</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Observations Used</th>
<td class="r data">6178</td>
</tr>
<tr>
<th class="rowheader" scope="row">Observations Not Used</th>
<td class="r data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">Total Observations</th>
<td class="r data">6178</td>
</tr>
<tr>
<th class="rowheader" scope="row">Subjects</th>
<td class="r data">31</td>
</tr>
<tr>
<th class="rowheader" scope="row">Max Obs per Subject</th>
<td class="r data">238</td>
</tr>
<tr>
<th class="rowheader" scope="row">Parameters</th>
<td class="r data">2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Quadrature Points</th>
<td class="r data">1</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX2" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Initial Parameters">
<caption aria-label="Initial Parameters"></caption>
<colgroup><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Initial Parameters</th>
</tr>
<tr>
<th class="r b header" scope="col">gamma00</th>
<th class="r b header" scope="col">tau00</th>
<th class="r b header" scope="col">Negative<br/>Log<br/>Likelihood</th>
</tr>
</thead>
<tbody>
<tr>
<td class="r data" style="white-space: nowrap">-1</td>
<td class="r data">1</td>
<td class="r data">4329.25972</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX3" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Iteration History">
<caption aria-label="Iteration History"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="6" scope="colgroup">Iteration History</th>
</tr>
<tr>
<th class="r b header" scope="col">Iteration</th>
<th class="r b header" scope="col">Calls</th>
<th class="r b header" scope="col">Negative<br/>Log<br/>Likelihood</th>
<th class="r b header" scope="col">Difference</th>
<th class="r b header" scope="col">Maximum<br/>Gradient</th>
<th class="r b header" scope="col">Slope</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="r data">6</td>
<td class="r data">4314.1825</td>
<td class="r data">15.07724</td>
<td class="r data">14.4613</td>
<td class="r data" style="white-space: nowrap">-151.871</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<td class="r data">16</td>
<td class="r data">4302.8614</td>
<td class="r data">11.32104</td>
<td class="r data">41.4893</td>
<td class="r data" style="white-space: nowrap">-34.2401</td>
</tr>
<tr>
<th class="r rowheader" scope="row">3</th>
<td class="r data">22</td>
<td class="r data">4302.2958</td>
<td class="r data">0.565654</td>
<td class="r data">53.4288</td>
<td class="r data" style="white-space: nowrap">-60.7572</td>
</tr>
<tr>
<th class="r rowheader" scope="row">4</th>
<td class="r data">29</td>
<td class="r data">4298.0747</td>
<td class="r data">4.22112</td>
<td class="r data">184.088</td>
<td class="r data" style="white-space: nowrap">-20.6365</td>
</tr>
<tr>
<th class="r rowheader" scope="row">5</th>
<td class="r data">36</td>
<td class="r data">4280.8413</td>
<td class="r data">17.23342</td>
<td class="r data">415.583</td>
<td class="r data" style="white-space: nowrap">-32.7086</td>
</tr>
<tr>
<th class="r rowheader" scope="row">6</th>
<td class="r data">40</td>
<td class="r data">4278.4399</td>
<td class="r data">2.401311</td>
<td class="r data">36.7201</td>
<td class="r data" style="white-space: nowrap">-156.999</td>
</tr>
<tr>
<th class="r rowheader" scope="row">7</th>
<td class="r data">43</td>
<td class="r data">4278.3651</td>
<td class="r data">0.074847</td>
<td class="r data">34.0781</td>
<td class="r data" style="white-space: nowrap">-0.19067</td>
</tr>
<tr>
<th class="r rowheader" scope="row">8</th>
<td class="r data">45</td>
<td class="r data">4278.2916</td>
<td class="r data">0.073493</td>
<td class="r data">8.94353</td>
<td class="r data" style="white-space: nowrap">-0.09359</td>
</tr>
<tr>
<th class="r rowheader" scope="row">9</th>
<td class="r data">48</td>
<td class="r data">4278.2854</td>
<td class="r data">0.006165</td>
<td class="r data">2.81772</td>
<td class="r data" style="white-space: nowrap">-0.01572</td>
</tr>
<tr>
<th class="r rowheader" scope="row">10</th>
<td class="r data">51</td>
<td class="r data">4278.2849</td>
<td class="r data">0.0005</td>
<td class="r data">0.078424</td>
<td class="r data" style="white-space: nowrap">-0.00096</td>
</tr>
<tr>
<th class="r rowheader" scope="row">11</th>
<td class="r data">54</td>
<td class="r data">4278.2849</td>
<td class="r data" style="white-space: nowrap">9.217E-7</td>
<td class="r data">0.002819</td>
<td class="r data" style="white-space: nowrap">-1.92E-6</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX4" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Convergence Status">
<caption aria-label="Convergence Status"></caption>
<colgroup><col/></colgroup>
<tbody>
<tr>
<td class="c data">NOTE: GCONV convergence criterion satisfied.</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX5" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Fit Statistics">
<caption aria-label="Fit Statistics"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Fit Statistics</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">-2 Log Likelihood</th>
<td class="r data">8556.6</td>
</tr>
<tr>
<th class="rowheader" scope="row">AIC (smaller is better)</th>
<td class="r data">8560.6</td>
</tr>
<tr>
<th class="rowheader" scope="row">AICC (smaller is better)</th>
<td class="r data">8560.6</td>
</tr>
<tr>
<th class="rowheader" scope="row">BIC (smaller is better)</th>
<td class="r data">8563.4</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX6" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Parameter Estimates">
<caption aria-label="Parameter Estimates"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="9" scope="colgroup">Parameter Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Parameter</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">t&#160;Value</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;|t|</th>
<th class="c b header" colspan="2" scope="colgroup">95% Confidence Limits</th>
<th class="r b header" scope="col">Gradient</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">gamma00</th>
<td class="r data" style="white-space: nowrap">-0.00313</td>
<td class="r data">0.03522</td>
<td class="r data">30</td>
<td class="r data" style="white-space: nowrap">-0.09</td>
<td class="r data">0.9298</td>
<td class="r data" style="white-space: nowrap">-0.07505</td>
<td class="r data">0.06879</td>
<td class="r data" style="white-space: nowrap">-0.00232</td>
</tr>
<tr>
<th class="rowheader" scope="row">tau00</th>
<td class="r data">0.01792</td>
<td class="r data">0.009702</td>
<td class="r data">30</td>
<td class="r data">1.85</td>
<td class="r data">0.0747</td>
<td class="r data" style="white-space: nowrap">-0.00190</td>
<td class="r data">0.03773</td>
<td class="r data">0.002819</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX7" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Additional Estimates">
<caption aria-label="Additional Estimates"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="9" scope="colgroup">Additional Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Label</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">t&#160;Value</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;|t|</th>
<th class="r b header" scope="col">Alpha</th>
<th class="r b header" scope="col">Lower</th>
<th class="r b header" scope="col">Upper</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">ICC</th>
<td class="r data">0.005416</td>
<td class="r data">0.002917</td>
<td class="r data">30</td>
<td class="r data">1.86</td>
<td class="r data">0.0732</td>
<td class="r data">0.05</td>
<td class="r data" style="white-space: nowrap">-0.00054</td>
<td class="r data">0.01137</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




The average estimates for the coefficients in this model are very similar to what we were seeing with the bootstrapped model. The ICC is .005 which is extremely low and indicates that the wins and loses with this sample are not affected by the grouping of teams. We could still use a multilevel model to examine the random effect associated with each team, and that is shown below. 

Were any teams better at scoring first? To answer that I'll use a different approach.


```python
%%SAS sas_session
proc glimmix data=nhl.gamesrand method=quad;
    class team;
    ODS OUTPUT SOLUTIONR=R;
    model won(event='1')= scoredfirst scoredsecond home overtime period penaltycount/dist=binary solution oddsratio ddfm=bw;
    nloptions maxiter=10000;
    random intercept scoredfirst /subject=team type=chol g solution;
run;

```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The GLIMMIX Procedure</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Model Information">
<caption aria-label="Model Information"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Model Information</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Data Set</th>
<td class="data">NHL.GAMESRAND</td>
</tr>
<tr>
<th class="rowheader" scope="row">Response Variable</th>
<td class="data">won</td>
</tr>
<tr>
<th class="rowheader" scope="row">Response Distribution</th>
<td class="data">Binary</td>
</tr>
<tr>
<th class="rowheader" scope="row">Link Function</th>
<td class="data">Logit</td>
</tr>
<tr>
<th class="rowheader" scope="row">Variance Function</th>
<td class="data">Default</td>
</tr>
<tr>
<th class="rowheader" scope="row">Variance Matrix Blocked By</th>
<td class="data">team</td>
</tr>
<tr>
<th class="rowheader" scope="row">Estimation Technique</th>
<td class="data">Maximum Likelihood</td>
</tr>
<tr>
<th class="rowheader" scope="row">Likelihood Approximation</th>
<td class="data">Gauss-Hermite Quadrature</td>
</tr>
<tr>
<th class="rowheader" scope="row">Degrees of Freedom Method</th>
<td class="data">Between-Within</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX1" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Class Level Information">
<caption aria-label="Class Level Information"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Class Level Information</th>
</tr>
<tr>
<th class="b header" scope="col">Class</th>
<th class="r b header" scope="col">Levels</th>
<th class="b header" scope="col">Values</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">team</th>
<td class="r data">31</td>
<td class="data">ANA BOS BUF CAR CBJ CGY CHI COL DAL DET EDM FLA LA MIN MTL NJ NSH NYI NYR OTT PHI PHX PIT SJ STL TB TOR VAN VGK WPG WSH</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX2" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Number of Observations">
<caption aria-label="Number of Observations"></caption>
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Number of Observations Read</th>
<td class="r data">6178</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Observations Used</th>
<td class="r data">6178</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX3" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Response Profiles">
<caption aria-label="Response Profiles"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="3" scope="colgroup">Response Profile</th>
</tr>
<tr>
<th class="r b header" scope="col">Ordered<br/>Value</th>
<th class="b header" scope="col">won</th>
<th class="r b header" scope="col">Total<br/>Frequency</th>
</tr>
</thead>
<tfoot>
<tr>
<th class="c b footer" colspan="3">The GLIMMIX procedure is modeling the probability that won=&apos;1&apos;.</th>
</tr>
</tfoot>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<th class="data">0</th>
<td class="r data">3071</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<th class="data">1</th>
<td class="r data">3107</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX4" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Dimensions">
<caption aria-label="Dimensions"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Dimensions</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">G-side Cov. Parameters</th>
<td class="r data">3</td>
</tr>
<tr>
<th class="rowheader" scope="row">Columns in X</th>
<td class="r data">7</td>
</tr>
<tr>
<th class="rowheader" scope="row">Columns in Z per Subject</th>
<td class="r data">2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Subjects (Blocks in V)</th>
<td class="r data">31</td>
</tr>
<tr>
<th class="rowheader" scope="row">Max Obs per Subject</th>
<td class="r data">238</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX5" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Optimization Information">
<caption aria-label="Optimization Information"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Optimization Information</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Optimization Technique</th>
<td class="data">Dual Quasi-Newton</td>
</tr>
<tr>
<th class="rowheader" scope="row">Parameters in Optimization</th>
<td class="data">10</td>
</tr>
<tr>
<th class="rowheader" scope="row">Lower Boundaries</th>
<td class="data">2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Upper Boundaries</th>
<td class="data">0</td>
</tr>
<tr>
<th class="rowheader" scope="row">Fixed Effects</th>
<td class="data">Not Profiled</td>
</tr>
<tr>
<th class="rowheader" scope="row">Starting From</th>
<td class="data">GLM estimates</td>
</tr>
<tr>
<th class="rowheader" scope="row">Quadrature Points</th>
<td class="data">1</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX6" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Iteration History">
<caption aria-label="Iteration History"></caption>
<colgroup><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="6" scope="colgroup">Iteration History</th>
</tr>
<tr>
<th class="r b header" scope="col">Iteration</th>
<th class="r b header" scope="col">Restarts</th>
<th class="r b header" scope="col">Evaluations</th>
<th class="r b header" scope="col">Objective<br/>Function</th>
<th class="r b header" scope="col">Change</th>
<th class="r b header" scope="col">Max<br/>Gradient</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">0</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">4</td>
<td class="r data">6711.4553725</td>
<td class="r data">.</td>
<td class="r data">60.84809</td>
</tr>
<tr>
<th class="r rowheader" scope="row">1</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6710.7871203</td>
<td class="r data">0.66825213</td>
<td class="r data">263.1968</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6706.5499723</td>
<td class="r data">4.23714802</td>
<td class="r data">148.9014</td>
</tr>
<tr>
<th class="r rowheader" scope="row">3</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">2</td>
<td class="r data">6703.3524844</td>
<td class="r data">3.19748792</td>
<td class="r data">159.9946</td>
</tr>
<tr>
<th class="r rowheader" scope="row">4</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6703.2107923</td>
<td class="r data">0.14169209</td>
<td class="r data">147.1521</td>
</tr>
<tr>
<th class="r rowheader" scope="row">5</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">2</td>
<td class="r data">6702.940897</td>
<td class="r data">0.26989531</td>
<td class="r data">130.1817</td>
</tr>
<tr>
<th class="r rowheader" scope="row">6</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6702.7805741</td>
<td class="r data">0.16032292</td>
<td class="r data">116.9171</td>
</tr>
<tr>
<th class="r rowheader" scope="row">7</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">2</td>
<td class="r data">6702.4437877</td>
<td class="r data">0.33678642</td>
<td class="r data">48.60881</td>
</tr>
<tr>
<th class="r rowheader" scope="row">8</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6702.1046842</td>
<td class="r data">0.33910344</td>
<td class="r data">5.312724</td>
</tr>
<tr>
<th class="r rowheader" scope="row">9</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6702.043703</td>
<td class="r data">0.06098120</td>
<td class="r data">12.1805</td>
</tr>
<tr>
<th class="r rowheader" scope="row">10</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6702.0167753</td>
<td class="r data">0.02692775</td>
<td class="r data">13.42428</td>
</tr>
<tr>
<th class="r rowheader" scope="row">11</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6702.0127666</td>
<td class="r data">0.00400870</td>
<td class="r data">12.09659</td>
</tr>
<tr>
<th class="r rowheader" scope="row">12</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">4</td>
<td class="r data">6701.999187</td>
<td class="r data">0.01357963</td>
<td class="r data">0.689931</td>
</tr>
<tr>
<th class="r rowheader" scope="row">13</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6701.9990994</td>
<td class="r data">0.00008755</td>
<td class="r data">0.015468</td>
</tr>
<tr>
<th class="r rowheader" scope="row">14</th>
<th class="r rowheader" scope="row">0</th>
<td class="r data">3</td>
<td class="r data">6701.9990992</td>
<td class="r data">0.00000016</td>
<td class="r data">0.000821</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX7" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Convergence Status">
<caption aria-label="Convergence Status"></caption>
<colgroup><col/></colgroup>
<tbody>
<tr>
<td class="c data">Convergence criterion (GCONV=1E-8) satisfied.</td>
</tr>
</tbody>
</table>
</div>
<div class="proc_note_group">
<p class="c proctitle">Estimated G matrix is not positive definite.</p>
</div>
<div id="IDX8" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Fit Statistics">
<caption aria-label="Fit Statistics"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Fit Statistics</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">-2 Log Likelihood</th>
<td class="r data">6702.00</td>
</tr>
<tr>
<th class="rowheader" scope="row">AIC  (smaller is better)</th>
<td class="r data">6720.00</td>
</tr>
<tr>
<th class="rowheader" scope="row">AICC (smaller is better)</th>
<td class="r data">6720.03</td>
</tr>
<tr>
<th class="rowheader" scope="row">BIC  (smaller is better)</th>
<td class="r data">6732.90</td>
</tr>
<tr>
<th class="rowheader" scope="row">CAIC (smaller is better)</th>
<td class="r data">6741.90</td>
</tr>
<tr>
<th class="rowheader" scope="row">HQIC (smaller is better)</th>
<td class="r data">6724.21</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX9" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Fit Statistics for Conditional Distribution">
<caption aria-label="Fit Statistics for Conditional Distribution"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Fit Statistics for Conditional Distribution</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">-2 log L(won | r. effects)</th>
<td class="r data">6663.54</td>
</tr>
<tr>
<th class="rowheader" scope="row">Pearson Chi-Square</th>
<td class="r data">6138.44</td>
</tr>
<tr>
<th class="rowheader" scope="row">Pearson Chi-Square / DF</th>
<td class="r data">0.99</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX10" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Estimated G Matrix">
<caption aria-label="Estimated G Matrix"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Estimated G Matrix</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">Row</th>
<th class="r b header" scope="col">Col1</th>
<th class="r b header" scope="col">Col2</th>
</tr>
</thead>
<tbody>
<tr>
<th class="data">Intercept</th>
<th class="r rowheader" scope="row">1</th>
<td class="r data">0.02931</td>
<td class="r data">0.000438</td>
</tr>
<tr>
<th class="data">scoredfirst</th>
<th class="r rowheader" scope="row">2</th>
<td class="r data">0.000438</td>
<td class="r data" style="white-space: nowrap">6.533E-6</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX11" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Covariance Parameter Estimates">
<caption aria-label="Covariance Parameter Estimates"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Covariance Parameter Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Cov Parm</th>
<th class="b header" scope="col">Subject</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">CHOL(1,1)</th>
<th class="data">team</th>
<td class="r data">0.1712</td>
<td class="r data">0.05534</td>
</tr>
<tr>
<th class="rowheader" scope="row">CHOL(2,1)</th>
<th class="data">team</th>
<td class="r data">0.002556</td>
<td class="r data">0.07133</td>
</tr>
<tr>
<th class="rowheader" scope="row">CHOL(2,2)</th>
<th class="data">team</th>
<td class="r data">0</td>
<td class="r data">.</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX12" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Solutions for Fixed Effects">
<caption aria-label="Solutions for Fixed Effects"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="6" scope="colgroup">Solutions for Fixed Effects</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">t&#160;Value</th>
<th class="r b header" scope="col">Pr &gt; |t|</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<td class="r data" style="white-space: nowrap">-2.0111</td>
<td class="r data">0.1244</td>
<td class="r data">30</td>
<td class="r data" style="white-space: nowrap">-16.17</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<td class="r data">1.9085</td>
<td class="r data">0.06506</td>
<td class="r data">6141</td>
<td class="r data">29.34</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredsecond</th>
<td class="r data">1.8539</td>
<td class="r data">0.06533</td>
<td class="r data">6141</td>
<td class="r data">28.38</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">home</th>
<td class="r data">0.3270</td>
<td class="r data">0.06014</td>
<td class="r data">6141</td>
<td class="r data">5.44</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">overtime</th>
<td class="r data" style="white-space: nowrap">-0.09928</td>
<td class="r data">0.08424</td>
<td class="r data">6141</td>
<td class="r data" style="white-space: nowrap">-1.18</td>
<td class="r data">0.2386</td>
</tr>
<tr>
<th class="rowheader" scope="row">period</th>
<td class="r data">0.1754</td>
<td class="r data">0.05700</td>
<td class="r data">6141</td>
<td class="r data">3.08</td>
<td class="r data">0.0021</td>
</tr>
<tr>
<th class="rowheader" scope="row">penaltycount</th>
<td class="r data" style="white-space: nowrap">-0.05052</td>
<td class="r data">0.01484</td>
<td class="r data">6141</td>
<td class="r data" style="white-space: nowrap">-3.40</td>
<td class="r data">0.0007</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX13" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Odds Ratio Estimates">
<caption aria-label="Odds Ratio Estimates"></caption>
<colgroup><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/></colgroup><colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="16" scope="colgroup">Odds Ratio Estimates</th>
</tr>
<tr>
<th class="r b header" scope="col">scoredfirst</th>
<th class="r b header" scope="col">scoredsecond</th>
<th class="r b header" scope="col">home</th>
<th class="r b header" scope="col">overtime</th>
<th class="r b header" scope="col">period</th>
<th class="r b header" scope="col">penaltycount</th>
<th class="r b header" scope="col">_scoredfirst</th>
<th class="r b header" scope="col">_scoredsecond</th>
<th class="r b header" scope="col">_home</th>
<th class="r b header" scope="col">_overtime</th>
<th class="r b header" scope="col">_period</th>
<th class="r b header" scope="col">_penaltycount</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">DF</th>
<th class="c b header" colspan="2" scope="colgroup">95% Confidence Limits</th>
</tr>
</thead>
<tfoot>
<tr>
<th class="b footer" colspan="16">Effects of continuous variables are assessed as one unit offsets from the mean. The AT suboption modifies the reference value and the UNIT suboption modifies the offsets.</th>
</tr>
</tfoot>
<tbody>
<tr>
<th class="r data">1.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<td class="r data">6.743</td>
<td class="r data">6141</td>
<td class="r data">5.936</td>
<td class="r data">7.661</td>
</tr>
<tr>
<th class="r data">0.4989</th>
<th class="r data">1.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<td class="r data">6.385</td>
<td class="r data">6141</td>
<td class="r data">5.617</td>
<td class="r data">7.257</td>
</tr>
<tr>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">1.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<td class="r data">1.387</td>
<td class="r data">6141</td>
<td class="r data">1.233</td>
<td class="r data">1.560</td>
</tr>
<tr>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">1.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<td class="r data">0.905</td>
<td class="r data">6141</td>
<td class="r data">0.768</td>
<td class="r data">1.068</td>
</tr>
<tr>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">2.2488</th>
<th class="r data">3.9908</th>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<td class="r data">1.192</td>
<td class="r data">6141</td>
<td class="r data">1.066</td>
<td class="r data">1.333</td>
</tr>
<tr>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">4.9908</th>
<th class="r data">0.4989</th>
<th class="r data">0.4909</th>
<th class="r data">0.5034</th>
<th class="r data">0.1318</th>
<th class="r data">1.2488</th>
<th class="r data">3.9908</th>
<td class="r data">0.951</td>
<td class="r data">6141</td>
<td class="r data">0.923</td>
<td class="r data">0.979</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX14" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Type III Tests of Fixed Effects">
<caption aria-label="Type III Tests of Fixed Effects"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="5" scope="colgroup">Type III Tests of Fixed Effects</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="r b header" scope="col">Num DF</th>
<th class="r b header" scope="col">Den DF</th>
<th class="r b header" scope="col">F Value</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;F</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<td class="r data">1</td>
<td class="r data">6141</td>
<td class="r data">860.64</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredsecond</th>
<td class="r data">1</td>
<td class="r data">6141</td>
<td class="r data">805.20</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">home</th>
<td class="r data">1</td>
<td class="r data">6141</td>
<td class="r data">29.57</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">overtime</th>
<td class="r data">1</td>
<td class="r data">6141</td>
<td class="r data">1.39</td>
<td class="r data">0.2386</td>
</tr>
<tr>
<th class="rowheader" scope="row">period</th>
<td class="r data">1</td>
<td class="r data">6141</td>
<td class="r data">9.47</td>
<td class="r data">0.0021</td>
</tr>
<tr>
<th class="rowheader" scope="row">penaltycount</th>
<td class="r data">1</td>
<td class="r data">6141</td>
<td class="r data">11.58</td>
<td class="r data">0.0007</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX15" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Solution for Random Effects">
<caption aria-label="Solution for Random Effects"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="7" scope="colgroup">Solution for Random Effects</th>
</tr>
<tr>
<th class="b header" scope="col">Effect</th>
<th class="b header" scope="col">Subject</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Std Err Pred</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">t&#160;Value</th>
<th class="r b header" scope="col">Pr &gt; |t|</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team ANA</th>
<td class="r data">0.1035</td>
<td class="r data">0.1275</td>
<td class="r data">6171</td>
<td class="r data">0.81</td>
<td class="r data">0.4169</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team ANA</th>
<td class="r data">0.001546</td>
<td class="r data">0.04294</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9713</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team BOS</th>
<td class="r data">0.1345</td>
<td class="r data">0.1305</td>
<td class="r data">6171</td>
<td class="r data">1.03</td>
<td class="r data">0.3027</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team BOS</th>
<td class="r data">0.002008</td>
<td class="r data">0.05592</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team BUF</th>
<td class="r data" style="white-space: nowrap">-0.2700</td>
<td class="r data">0.1522</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-1.77</td>
<td class="r data">0.0761</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team BUF</th>
<td class="r data" style="white-space: nowrap">-0.00403</td>
<td class="r data">0.1123</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team CAR</th>
<td class="r data" style="white-space: nowrap">-0.2826</td>
<td class="r data">0.1553</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-1.82</td>
<td class="r data">0.0690</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team CAR</th>
<td class="r data" style="white-space: nowrap">-0.00422</td>
<td class="r data">0.1174</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9713</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team CBJ</th>
<td class="r data">0.008767</td>
<td class="r data">0.1201</td>
<td class="r data">6171</td>
<td class="r data">0.07</td>
<td class="r data">0.9418</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team CBJ</th>
<td class="r data">0.000131</td>
<td class="r data">0.003898</td>
<td class="r data">6171</td>
<td class="r data">0.03</td>
<td class="r data">0.9732</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team CGY</th>
<td class="r data">0.08337</td>
<td class="r data">0.1249</td>
<td class="r data">6171</td>
<td class="r data">0.67</td>
<td class="r data">0.5046</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team CGY</th>
<td class="r data">0.001245</td>
<td class="r data">0.03460</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9713</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team CHI</th>
<td class="r data">0.1148</td>
<td class="r data">0.1248</td>
<td class="r data">6171</td>
<td class="r data">0.92</td>
<td class="r data">0.3576</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team CHI</th>
<td class="r data">0.001714</td>
<td class="r data">0.04811</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team COL</th>
<td class="r data" style="white-space: nowrap">-0.1140</td>
<td class="r data">0.1241</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.92</td>
<td class="r data">0.3584</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team COL</th>
<td class="r data" style="white-space: nowrap">-0.00170</td>
<td class="r data">0.04749</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team DAL</th>
<td class="r data" style="white-space: nowrap">-0.04652</td>
<td class="r data">0.1179</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.39</td>
<td class="r data">0.6932</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team DAL</th>
<td class="r data" style="white-space: nowrap">-0.00069</td>
<td class="r data">0.01952</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team DET</th>
<td class="r data" style="white-space: nowrap">-0.09946</td>
<td class="r data">0.1226</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.81</td>
<td class="r data">0.4173</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team DET</th>
<td class="r data" style="white-space: nowrap">-0.00149</td>
<td class="r data">0.04182</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9717</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team EDM</th>
<td class="r data" style="white-space: nowrap">-0.06339</td>
<td class="r data">0.1213</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.52</td>
<td class="r data">0.6013</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team EDM</th>
<td class="r data" style="white-space: nowrap">-0.00095</td>
<td class="r data">0.02658</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team FLA</th>
<td class="r data" style="white-space: nowrap">-0.03694</td>
<td class="r data">0.1195</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.31</td>
<td class="r data">0.7573</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team FLA</th>
<td class="r data" style="white-space: nowrap">-0.00055</td>
<td class="r data">0.01543</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9715</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team LA</th>
<td class="r data" style="white-space: nowrap">-0.05814</td>
<td class="r data">0.1219</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.48</td>
<td class="r data">0.6333</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team LA</th>
<td class="r data" style="white-space: nowrap">-0.00087</td>
<td class="r data">0.02468</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9719</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team MIN</th>
<td class="r data">0.1063</td>
<td class="r data">0.1267</td>
<td class="r data">6171</td>
<td class="r data">0.84</td>
<td class="r data">0.4014</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team MIN</th>
<td class="r data">0.001587</td>
<td class="r data">0.04433</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team MTL</th>
<td class="r data">0.01175</td>
<td class="r data">0.1219</td>
<td class="r data">6171</td>
<td class="r data">0.10</td>
<td class="r data">0.9232</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team MTL</th>
<td class="r data">0.000175</td>
<td class="r data">0.005245</td>
<td class="r data">6171</td>
<td class="r data">0.03</td>
<td class="r data">0.9733</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team NJ</th>
<td class="r data" style="white-space: nowrap">-0.1392</td>
<td class="r data">0.1221</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-1.14</td>
<td class="r data">0.2540</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team NJ</th>
<td class="r data" style="white-space: nowrap">-0.00208</td>
<td class="r data">0.05832</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team NSH</th>
<td class="r data">0.1093</td>
<td class="r data">0.1279</td>
<td class="r data">6171</td>
<td class="r data">0.85</td>
<td class="r data">0.3929</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team NSH</th>
<td class="r data">0.001632</td>
<td class="r data">0.04540</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9713</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team NYI</th>
<td class="r data" style="white-space: nowrap">-0.03941</td>
<td class="r data">0.1202</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.33</td>
<td class="r data">0.7430</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team NYI</th>
<td class="r data" style="white-space: nowrap">-0.00059</td>
<td class="r data">0.01672</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9719</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team NYR</th>
<td class="r data">0.1213</td>
<td class="r data">0.1217</td>
<td class="r data">6171</td>
<td class="r data">1.00</td>
<td class="r data">0.3193</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team NYR</th>
<td class="r data">0.001810</td>
<td class="r data">0.05085</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team OTT</th>
<td class="r data">0.05774</td>
<td class="r data">0.1216</td>
<td class="r data">6171</td>
<td class="r data">0.47</td>
<td class="r data">0.6349</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team OTT</th>
<td class="r data">0.000862</td>
<td class="r data">0.02421</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team PHI</th>
<td class="r data" style="white-space: nowrap">-0.06709</td>
<td class="r data">0.1233</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.54</td>
<td class="r data">0.5863</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team PHI</th>
<td class="r data" style="white-space: nowrap">-0.00100</td>
<td class="r data">0.02831</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9718</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team PHX</th>
<td class="r data" style="white-space: nowrap">-0.2198</td>
<td class="r data">0.1461</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-1.50</td>
<td class="r data">0.1324</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team PHX</th>
<td class="r data" style="white-space: nowrap">-0.00328</td>
<td class="r data">0.09139</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team PIT</th>
<td class="r data">0.1195</td>
<td class="r data">0.1313</td>
<td class="r data">6171</td>
<td class="r data">0.91</td>
<td class="r data">0.3629</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team PIT</th>
<td class="r data">0.001784</td>
<td class="r data">0.04981</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team SJ</th>
<td class="r data">0.03619</td>
<td class="r data">0.1214</td>
<td class="r data">6171</td>
<td class="r data">0.30</td>
<td class="r data">0.7656</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team SJ</th>
<td class="r data">0.000540</td>
<td class="r data">0.01527</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9718</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team STL</th>
<td class="r data">0.1790</td>
<td class="r data">0.1342</td>
<td class="r data">6171</td>
<td class="r data">1.33</td>
<td class="r data">0.1825</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team STL</th>
<td class="r data">0.002672</td>
<td class="r data">0.07458</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team TB</th>
<td class="r data">0.1397</td>
<td class="r data">0.1292</td>
<td class="r data">6171</td>
<td class="r data">1.08</td>
<td class="r data">0.2797</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team TB</th>
<td class="r data">0.002086</td>
<td class="r data">0.05817</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9714</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team TOR</th>
<td class="r data" style="white-space: nowrap">-0.06816</td>
<td class="r data">0.1281</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.53</td>
<td class="r data">0.5947</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team TOR</th>
<td class="r data" style="white-space: nowrap">-0.00102</td>
<td class="r data">0.02818</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9712</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team VAN</th>
<td class="r data" style="white-space: nowrap">-0.08880</td>
<td class="r data">0.1202</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.74</td>
<td class="r data">0.4600</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team VAN</th>
<td class="r data" style="white-space: nowrap">-0.00133</td>
<td class="r data">0.03721</td>
<td class="r data">6171</td>
<td class="r data" style="white-space: nowrap">-0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team VGK</th>
<td class="r data">0.01550</td>
<td class="r data">0.1552</td>
<td class="r data">6171</td>
<td class="r data">0.10</td>
<td class="r data">0.9205</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team VGK</th>
<td class="r data">0.000231</td>
<td class="r data">0.007014</td>
<td class="r data">6171</td>
<td class="r data">0.03</td>
<td class="r data">0.9737</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team WPG</th>
<td class="r data">0.08358</td>
<td class="r data">0.1203</td>
<td class="r data">6171</td>
<td class="r data">0.69</td>
<td class="r data">0.4873</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team WPG</th>
<td class="r data">0.001248</td>
<td class="r data">0.03502</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9716</td>
</tr>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="data">team WSH</th>
<td class="r data">0.1675</td>
<td class="r data">0.1264</td>
<td class="r data">6171</td>
<td class="r data">1.33</td>
<td class="r data">0.1850</td>
</tr>
<tr>
<th class="rowheader" scope="row">scoredfirst</th>
<th class="data">team WSH</th>
<td class="r data">0.002501</td>
<td class="r data">0.07022</td>
<td class="r data">6171</td>
<td class="r data">0.04</td>
<td class="r data">0.9716</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
%%SAS sas_session
proc sql;
    select Subject, (Estimate+1.9085) as ScoredFirst
      from r
      where Effect="scoredfirst"
        order by ScoredFirst;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Subject</th>
<th class="r b header" scope="col">ScoredFirst</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">team CAR</td>
<td class="r data">1.904281</td>
</tr>
<tr>
<td class="data">team BUF</td>
<td class="r data">1.904468</td>
</tr>
<tr>
<td class="data">team PHX</td>
<td class="r data">1.905218</td>
</tr>
<tr>
<td class="data">team NJ</td>
<td class="r data">1.906421</td>
</tr>
<tr>
<td class="data">team COL</td>
<td class="r data">1.906798</td>
</tr>
<tr>
<td class="data">team DET</td>
<td class="r data">1.907015</td>
</tr>
<tr>
<td class="data">team VAN</td>
<td class="r data">1.907174</td>
</tr>
<tr>
<td class="data">team TOR</td>
<td class="r data">1.907482</td>
</tr>
<tr>
<td class="data">team PHI</td>
<td class="r data">1.907498</td>
</tr>
<tr>
<td class="data">team EDM</td>
<td class="r data">1.907554</td>
</tr>
<tr>
<td class="data">team LA</td>
<td class="r data">1.907632</td>
</tr>
<tr>
<td class="data">team DAL</td>
<td class="r data">1.907805</td>
</tr>
<tr>
<td class="data">team NYI</td>
<td class="r data">1.907912</td>
</tr>
<tr>
<td class="data">team FLA</td>
<td class="r data">1.907948</td>
</tr>
<tr>
<td class="data">team CBJ</td>
<td class="r data">1.908631</td>
</tr>
<tr>
<td class="data">team MTL</td>
<td class="r data">1.908675</td>
</tr>
<tr>
<td class="data">team VGK</td>
<td class="r data">1.908731</td>
</tr>
<tr>
<td class="data">team SJ</td>
<td class="r data">1.90904</td>
</tr>
<tr>
<td class="data">team OTT</td>
<td class="r data">1.909362</td>
</tr>
<tr>
<td class="data">team CGY</td>
<td class="r data">1.909745</td>
</tr>
<tr>
<td class="data">team WPG</td>
<td class="r data">1.909748</td>
</tr>
<tr>
<td class="data">team ANA</td>
<td class="r data">1.910046</td>
</tr>
<tr>
<td class="data">team MIN</td>
<td class="r data">1.910087</td>
</tr>
<tr>
<td class="data">team NSH</td>
<td class="r data">1.910132</td>
</tr>
<tr>
<td class="data">team CHI</td>
<td class="r data">1.910214</td>
</tr>
<tr>
<td class="data">team PIT</td>
<td class="r data">1.910284</td>
</tr>
<tr>
<td class="data">team NYR</td>
<td class="r data">1.91031</td>
</tr>
<tr>
<td class="data">team BOS</td>
<td class="r data">1.910508</td>
</tr>
<tr>
<td class="data">team TB</td>
<td class="r data">1.910586</td>
</tr>
<tr>
<td class="data">team WSH</td>
<td class="r data">1.911001</td>
</tr>
<tr>
<td class="data">team STL</td>
<td class="r data">1.911172</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




To break up the effect for each team, I will run the logistic regression by team.



```python
%%SAS sas_session
proc sort data=nhl.games;
 by team;
run;
ods trace on;
ods output ParameterEstimates=nhl.teamestimates;
proc logistic data=nhl.games;
by team;
class scoredfirst(ref='0') scoredsecond(ref='0') home(ref='0')overtime(ref='0')/param=ref;
model won(event='1')= scoredfirst scoredsecond home overtime period penalty_cen/;
run; quit;
ods trace off;

```


```python
%%SAS sas_session
proc print data=nhl.teamestimates(obs=10);
run;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Data Set NHL.TEAMESTIMATES">
<caption aria-label="Data Set NHL.TEAMESTIMATES"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r header" scope="col">Obs</th>
<th class="header" scope="col">team</th>
<th class="header" scope="col">Variable</th>
<th class="header" scope="col">ClassVal0</th>
<th class="r header" scope="col">DF</th>
<th class="r header" scope="col">Estimate</th>
<th class="r header" scope="col">StdErr</th>
<th class="r header" scope="col">WaldChiSq</th>
<th class="r header" scope="col">ProbChiSq</th>
<th class="header" scope="col">_ESTTYPE_</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="data">ANA</td>
<td class="data">Intercept</td>
<td class="data"> &#160;</td>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-1.9796</td>
<td class="r data">0.4050</td>
<td class="r data">23.8903</td>
<td class="r data">&lt;.0001</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">2</th>
<td class="data">ANA</td>
<td class="data">scoredfirst</td>
<td class="data">1</td>
<td class="r data">1</td>
<td class="r data">1.5697</td>
<td class="r data">0.2389</td>
<td class="r data">43.1713</td>
<td class="r data">&lt;.0001</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">3</th>
<td class="data">ANA</td>
<td class="data">scoredsecond</td>
<td class="data">1</td>
<td class="r data">1</td>
<td class="r data">1.5974</td>
<td class="r data">0.2415</td>
<td class="r data">43.7494</td>
<td class="r data">&lt;.0001</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">4</th>
<td class="data">ANA</td>
<td class="data">home</td>
<td class="data">1</td>
<td class="r data">1</td>
<td class="r data">0.6988</td>
<td class="r data">0.2314</td>
<td class="r data">9.1181</td>
<td class="r data">0.0025</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">5</th>
<td class="data">ANA</td>
<td class="data">overtime</td>
<td class="data">1</td>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-0.5108</td>
<td class="r data">0.3259</td>
<td class="r data">2.4570</td>
<td class="r data">0.1170</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">6</th>
<td class="data">ANA</td>
<td class="data">period</td>
<td class="data"> &#160;</td>
<td class="r data">1</td>
<td class="r data">0.3861</td>
<td class="r data">0.2362</td>
<td class="r data">2.6721</td>
<td class="r data">0.1021</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">7</th>
<td class="data">ANA</td>
<td class="data">penalty_cen</td>
<td class="data"> &#160;</td>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-0.00960</td>
<td class="r data">0.0528</td>
<td class="r data">0.0331</td>
<td class="r data">0.8556</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">8</th>
<td class="data">BOS</td>
<td class="data">Intercept</td>
<td class="data"> &#160;</td>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-2.4694</td>
<td class="r data">0.4521</td>
<td class="r data">29.8340</td>
<td class="r data">&lt;.0001</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">9</th>
<td class="data">BOS</td>
<td class="data">scoredfirst</td>
<td class="data">1</td>
<td class="r data">1</td>
<td class="r data">2.1861</td>
<td class="r data">0.2719</td>
<td class="r data">64.6617</td>
<td class="r data">&lt;.0001</td>
<td class="data">MLE</td>
</tr>
<tr>
<th class="r rowheader" scope="row">10</th>
<td class="data">BOS</td>
<td class="data">scoredsecond</td>
<td class="data">1</td>
<td class="r data">1</td>
<td class="r data">1.8727</td>
<td class="r data">0.2724</td>
<td class="r data">47.2734</td>
<td class="r data">&lt;.0001</td>
<td class="data">MLE</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
%%SAS sas_session
proc sql;
    select Variable, Team, Estimate, ProbChiSQ 
      from nhl.teamestimates
      where Variable="scoredfirst"
        order by Estimate;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Pr &gt; Chi-Square</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">scoredfirst</td>
<td class="data">NYI</td>
<td class="r data">1.1333</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">VAN</td>
<td class="r data">1.3658</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">PIT</td>
<td class="r data">1.5193</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">ANA</td>
<td class="r data">1.5697</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">LA</td>
<td class="r data">1.6351</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">DET</td>
<td class="r data">1.6427</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">PHX</td>
<td class="r data">1.6749</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">CBJ</td>
<td class="r data">1.7355</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">EDM</td>
<td class="r data">1.7418</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">NSH</td>
<td class="r data">1.7765</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">DAL</td>
<td class="r data">1.8462</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">OTT</td>
<td class="r data">1.8750</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">PHI</td>
<td class="r data">1.8914</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">CGY</td>
<td class="r data">1.8929</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">BUF</td>
<td class="r data">1.9120</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">FLA</td>
<td class="r data">1.9808</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">MIN</td>
<td class="r data">2.0916</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">TOR</td>
<td class="r data">2.1204</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">CHI</td>
<td class="r data">2.1262</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">WPG</td>
<td class="r data">2.1340</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">STL</td>
<td class="r data">2.1541</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">SJ</td>
<td class="r data">2.1664</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">NJ</td>
<td class="r data">2.1830</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">BOS</td>
<td class="r data">2.1861</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">TB</td>
<td class="r data">2.1991</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">COL</td>
<td class="r data">2.2399</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">CAR</td>
<td class="r data">2.2597</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">NYR</td>
<td class="r data">2.2732</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">MTL</td>
<td class="r data">2.3390</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">VGK</td>
<td class="r data">2.3544</td>
<td class="r data">0.0002</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">WSH</td>
<td class="r data">2.5618</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




Surprisingly, the Vegas Golden Knights ranked really high in this. The Knights were really good at winning after scoring first (72% probability of winning). But, maybe it isn't so surprising. There is only one season to go off of, and they made it to the stanley cup, meaning they won lots of games in that season.

Another consideration, if you look at a lot of the top teams that are likely to win after scoring first they have some of the best goalies in the league: Holtby, Fleury, Bishop/Vasilevskiy (some of TB seasons), Price, and Lundqvist. If you have a good goalie, it likely makes it even harder on the opponent to score the equalizer and a second goal. That first goal also takes off some pressure on the goalie. It also probably indicates more offensive zone posession, meaning the team is keeping the puck away from their own goal. VGK had Fleury who played phenomenally throughout the regular season and playoffs.

What about COL and CAR? That seems to make less sense. But going back to the section about downloading the data, I reviewed the players that scored first or assisted on the first goal. Colorado had RANTANEN and Carolina STAAL pop up for different seasons. So there could just be a factor that the low-winning teams just happened to win by scoring first.


```python
%%SAS sas_session
proc sql;
    select Variable, Team, Estimate, ProbChiSQ 
      from nhl.teamestimates
      where team="VGK"
        order by Estimate;
run;
quit;

```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Pr &gt; Chi-Square</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">Intercept</td>
<td class="data">VGK</td>
<td class="r data" style="white-space: nowrap">-1.8942</td>
<td class="r data">0.0563</td>
</tr>
<tr>
<td class="data">penalty_cen</td>
<td class="data">VGK</td>
<td class="r data" style="white-space: nowrap">-0.1949</td>
<td class="r data">0.3537</td>
</tr>
<tr>
<td class="data">period</td>
<td class="data">VGK</td>
<td class="r data">0.0616</td>
<td class="r data">0.9138</td>
</tr>
<tr>
<td class="data">overtime</td>
<td class="data">VGK</td>
<td class="r data">0.3167</td>
<td class="r data">0.6663</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">VGK</td>
<td class="r data">0.6651</td>
<td class="r data">0.2225</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">VGK</td>
<td class="r data">1.3322</td>
<td class="r data">0.0316</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">VGK</td>
<td class="r data">2.3544</td>
<td class="r data">0.0002</td>
</tr>
</tbody>
</table>
</div>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<hr class="pagebreak"/>
<div id="IDX1" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Pr &gt; Chi-Square</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">Intercept</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-2.0994</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">overtime</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-0.0818</td>
<td class="r data">0.8050</td>
</tr>
<tr>
<td class="data">penalty_cen</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-0.0151</td>
<td class="r data">0.7778</td>
</tr>
<tr>
<td class="data">period</td>
<td class="data">PIT</td>
<td class="r data">0.2954</td>
<td class="r data">0.1847</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PIT</td>
<td class="r data">0.9300</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">PIT</td>
<td class="r data">1.5193</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">PIT</td>
<td class="r data">1.7099</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
print("VGK Probability of winning after scoring first in the first period at home with about 4 penalties")
print("{0:.2f}%".format(convertToProb(-1.8942+2.3544+.6651+.0616-.1949)*100))
print("VGK Probability of winning after scoring second in the first period at home with about 4 penalties")
print("{0:.2f}%".format(convertToProb(-1.8942+1.3322+.6651+.0616-.1949)*100))
```

    VGK Probability of winning after scoring first in the first period at home with about 4 penalties
    72.95%
    VGK Probability of winning after scoring second in the first period at home with about 4 penalties
    49.25%



```python
%%SAS sas_session
proc sql;
    select Variable, Team, Estimate, ProbChiSQ 
      from nhl.teamestimates
      where Variable="scoredsecond"
        order by Estimate;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Pr &gt; Chi-Square</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">scoredsecond</td>
<td class="data">VGK</td>
<td class="r data">1.3322</td>
<td class="r data">0.0316</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">NYI</td>
<td class="r data">1.4828</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">MIN</td>
<td class="r data">1.4955</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">NSH</td>
<td class="r data">1.5128</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">PHX</td>
<td class="r data">1.5876</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">EDM</td>
<td class="r data">1.5884</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">ANA</td>
<td class="r data">1.5974</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">SJ</td>
<td class="r data">1.6608</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">PIT</td>
<td class="r data">1.7099</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">CBJ</td>
<td class="r data">1.7114</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">OTT</td>
<td class="r data">1.7161</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">FLA</td>
<td class="r data">1.7174</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">VAN</td>
<td class="r data">1.7430</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">DAL</td>
<td class="r data">1.8158</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">BOS</td>
<td class="r data">1.8727</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">DET</td>
<td class="r data">1.8870</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">BUF</td>
<td class="r data">1.8951</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">CHI</td>
<td class="r data">1.9125</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">NYR</td>
<td class="r data">1.9689</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">CAR</td>
<td class="r data">1.9849</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">WSH</td>
<td class="r data">2.0039</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">PHI</td>
<td class="r data">2.0239</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">STL</td>
<td class="r data">2.0241</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">COL</td>
<td class="r data">2.0283</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">CGY</td>
<td class="r data">2.1101</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">TB</td>
<td class="r data">2.1191</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">LA</td>
<td class="r data">2.1662</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">TOR</td>
<td class="r data">2.1773</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">WPG</td>
<td class="r data">2.1887</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">MTL</td>
<td class="r data">2.2306</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">NJ</td>
<td class="r data">2.2486</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




Interestingly the value for scoring second is flipped a little bit. The VKG are now at the bottom, again this is difficult because it is based only on their one great season. NJ was good at sealing the deal when they got the second goal, even without the first goal. Pittsburg seems to be the biggest outlier. They are low in winning probability even if they had the first or second goals. But still over 50% after scoring first. Another big factor for them seems to be home ice advantage, and holding the onto the win even if they scored in the first period.

Some other things that I'm not controlling for that could be influencing the result. The team's even strength goals, penalty killing percentages, etc...


```python
%%SAS sas_session
proc sql;
    select Variable, Team, Estimate, ProbChiSQ 
      from nhl.teamestimates
      where team="PIT"
        order by Estimate;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Pr &gt; Chi-Square</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">Intercept</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-2.0994</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">overtime</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-0.0818</td>
<td class="r data">0.8050</td>
</tr>
<tr>
<td class="data">penalty_cen</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-0.0151</td>
<td class="r data">0.7778</td>
</tr>
<tr>
<td class="data">period</td>
<td class="data">PIT</td>
<td class="r data">0.2954</td>
<td class="r data">0.1847</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PIT</td>
<td class="r data">0.9300</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredfirst</td>
<td class="data">PIT</td>
<td class="r data">1.5193</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<td class="data">scoredsecond</td>
<td class="data">PIT</td>
<td class="r data">1.7099</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>





```python
print("Probability of winning after scoring first in the first period at home with about 4 penalties")
print("{0:.2f}%".format(convertToProb(-2.0994+1.5193+.9300+.2954-.0151)*100))
print("Probability of winning after scoring second in the first period at home with about 4 penalties")
print("{0:.2f}%".format(convertToProb(-2.0994+1.7099+.9300+.2954-.0151)*100))
```

    Probability of winning after scoring first in the first period at home with about 4 penalties
    65.25%
    Probability of winning after scoring second in the first period at home with about 4 penalties
    69.44%


Looking at home ice advantage when controling for overtime and who scores first is interesting. Some teams actually has a negative or even non-significant effect, while others, like PIT, it has a high effect. Some of the others are not so surprising, Toronto and Philadelphia (if I was a ref I'd be intimidated in Philadelphia too). Tampa Bay is surprising to me, given the other teams in the top spots. Certainly, they have a great fan base (biased), but one of the problems is with out of state fans coming in. It isn't surprising to see more red sweaters than blue sweaters when, say, the blackhawks come through. I wonder, though, if ice has something to do with this. It can be hot and humid in Tampa, and the quality of the ice may be a factor, although it generally sits at the right temperature. Negating that argument is that PHX is hot too, but doesn't rank high in the home ice effect. Ignoring significance, PHX has more of an effect than BOS, which I would think the opposite would be true.

Perhaps more importantly, the top teams are also the teams that just won more games over the seasons of analysis, and would by that have more home wins.


```python
%%SAS sas_session
proc sql;
    select Variable, Team, Estimate, ProbChiSQ 
      from nhl.teamestimates
      where Variable="home"
        order by Estimate;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Pr &gt; Chi-Square</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">home</td>
<td class="data">MTL</td>
<td class="r data" style="white-space: nowrap">-0.0638</td>
<td class="r data">0.7957</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NYI</td>
<td class="r data" style="white-space: nowrap">-0.0379</td>
<td class="r data">0.8621</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">DET</td>
<td class="r data">0.0381</td>
<td class="r data">0.8691</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">SJ</td>
<td class="r data">0.0876</td>
<td class="r data">0.7082</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">VAN</td>
<td class="r data">0.0998</td>
<td class="r data">0.6632</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">COL</td>
<td class="r data">0.1336</td>
<td class="r data">0.5819</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CGY</td>
<td class="r data">0.1383</td>
<td class="r data">0.5713</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">MIN</td>
<td class="r data">0.1708</td>
<td class="r data">0.4756</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CHI</td>
<td class="r data">0.1838</td>
<td class="r data">0.4493</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">BOS</td>
<td class="r data">0.1872</td>
<td class="r data">0.4215</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">OTT</td>
<td class="r data">0.1959</td>
<td class="r data">0.3941</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CBJ</td>
<td class="r data">0.2357</td>
<td class="r data">0.3013</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">BUF</td>
<td class="r data">0.2436</td>
<td class="r data">0.3206</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NYR</td>
<td class="r data">0.2618</td>
<td class="r data">0.2845</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">WPG</td>
<td class="r data">0.2762</td>
<td class="r data">0.2346</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">STL</td>
<td class="r data">0.3251</td>
<td class="r data">0.1768</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">FLA</td>
<td class="r data">0.3905</td>
<td class="r data">0.0989</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PHX</td>
<td class="r data">0.3960</td>
<td class="r data">0.0867</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">EDM</td>
<td class="r data">0.4256</td>
<td class="r data">0.0643</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CAR</td>
<td class="r data">0.4366</td>
<td class="r data">0.0673</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NJ</td>
<td class="r data">0.4426</td>
<td class="r data">0.0706</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">LA</td>
<td class="r data">0.4487</td>
<td class="r data">0.0591</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NSH</td>
<td class="r data">0.4628</td>
<td class="r data">0.0445</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">DAL</td>
<td class="r data">0.4668</td>
<td class="r data">0.0447</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">WSH</td>
<td class="r data">0.5049</td>
<td class="r data">0.0427</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">VGK</td>
<td class="r data">0.6651</td>
<td class="r data">0.2225</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">ANA</td>
<td class="r data">0.6988</td>
<td class="r data">0.0025</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PHI</td>
<td class="r data">0.7184</td>
<td class="r data">0.0026</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">TB</td>
<td class="r data">0.8152</td>
<td class="r data">0.0010</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">TOR</td>
<td class="r data">0.8890</td>
<td class="r data">0.0006</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PIT</td>
<td class="r data">0.9300</td>
<td class="r data">&lt;.0001</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




One final look at a different component of home ice advantage is suggested in Soccernomics. That the home influence has nothing to do with time change, jet lag, or the fans and players; but it has something to do with fans and referees. To look at that, I will look at the relationship between penalties and home. This will just be a simple model, and as you will see in the results, it doesn't explain the variation in penalty count that well. Still it can tell us the overall difference in penalties between the home team and the away team


```python
%%SAS sas_session
proc reg data=nhl.gamesrand;

model penaltycount= home won /;
run; quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The REG Procedure</p>
<p class="c proctitle">Model: MODEL1</p>
<p class="c proctitle">Dependent Variable: penaltycount</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Number of Observations">
<caption aria-label="Number of Observations"></caption>
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Number of Observations Read</th>
<td class="r data">6178</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Observations Used</th>
<td class="r data">6178</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX1" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Analysis of Variance">
<caption aria-label="Analysis of Variance"></caption>
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="6" scope="colgroup">Analysis of Variance</th>
</tr>
<tr>
<th class="b header" scope="col">Source</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Sum of<br/>Squares</th>
<th class="r b header" scope="col">Mean<br/>Square</th>
<th class="r b header" scope="col">F Value</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;F</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Model</th>
<td class="r data">2</td>
<td class="r data">326.99322</td>
<td class="r data">163.49661</td>
<td class="r data">37.78</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">Error</th>
<td class="r data">6175</td>
<td class="r data">26721</td>
<td class="r data">4.32737</td>
<td class="r data">&#160;</td>
<td class="r data">&#160;</td>
</tr>
<tr>
<th class="rowheader" scope="row">Corrected Total</th>
<td class="r data">6177</td>
<td class="r data">27048</td>
<td class="r data">&#160;</td>
<td class="r data">&#160;</td>
<td class="r data">&#160;</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX2" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Fit Statistics">
<caption aria-label="Fit Statistics"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Root MSE</th>
<td class="r data">2.08023</td>
<th class="rowheader" scope="row">R-Square</th>
<td class="r data">0.0121</td>
</tr>
<tr>
<th class="rowheader" scope="row">Dependent Mean</th>
<td class="r data">3.99077</td>
<th class="rowheader" scope="row">Adj R-Sq</th>
<td class="r data">0.0118</td>
</tr>
<tr>
<th class="rowheader" scope="row">Coeff Var</th>
<td class="r data">52.12603</td>
<th class="rowheader" scope="row">&#160;</th>
<td class="r data">&#160;</td>
</tr>
</tbody>
</table>
</div>
<div id="IDX3" style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Parameter Estimates">
<caption aria-label="Parameter Estimates"></caption>
<colgroup><col/><col/></colgroup><colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="6" scope="colgroup">Parameter Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Parameter<br/>Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="r b header" scope="col">t&#160;Value</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;|t|</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<th class="r data">1</th>
<td class="r data">4.26670</td>
<td class="r data">0.04470</td>
<td class="r data">95.45</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">home</th>
<th class="r data">1</th>
<td class="r data" style="white-space: nowrap">-0.43584</td>
<td class="r data">0.05316</td>
<td class="r data" style="white-space: nowrap">-8.20</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">won</th>
<th class="r data">1</th>
<td class="r data" style="white-space: nowrap">-0.11240</td>
<td class="r data">0.05316</td>
<td class="r data" style="white-space: nowrap">-2.11</td>
<td class="r data">0.0345</td>
</tr>
</tbody>
</table>
</div>
</div>
</div>
<hr class="pagebreak"/>
<div id="IDX4" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The REG Procedure</p>
<p class="c proctitle">Model: MODEL1</p>
<p class="c proctitle">Dependent Variable: penaltycount</p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<div style="padding-bottom: 8px; padding-top: 1px">
<div class="c">
<img style="height: 360px; width: 640px" alt="Panel of heat maps of residuals by regressors for penaltycount." src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAFoCAIAAABIUN0GAAAACXBIWXMAAA7DAAAOwwHHb6hkAAAgAElEQVR4nO3de3wU1d348TOzl9wICYYASbjfKQhSoKhcpIhc9AHjT1B+ViNVtFKkFqW01AcstfWhIliqDaVVedTiBXi0Kk8RabUC2oKCJkWLAREkECSBkAC57GXm+WPrZEjIZrPM7tmd/bxfeeW1Z3dnztnZOfudOefMGaWoqEgAAIDocgohBg8eLLsYAAAkkOLiYlV2GQAASEQEYAAAJCAAAwAgAQEYAAAJCMAAAEhAAAYAQAICMAAAEhCAAQCQgAAMAIAEBGAAACQgAAMAIAEBGAAACQjAuIDk5GTla06ns0ePHnfeeefRo0fDW5umaSNHjuzZs+fJkyebvtqpUydFUUpLS8NbeWZmpqIox48fD/H5EFm7BeJOdXX1jBkzMjIyFEVZtWqV7OK04CK/a0AWAjBa4Pf7Dx069Mwzz1xxxRVnzpwJeyVCCF3XLS1alFiyBeLLihUrNm7cWF1dLYS47LLLZBenFcaNG6coyl/+8hfZBfm3WCsPYgoBGM3av3+/rusej2fHjh1ZWVlHjhx55ZVXwliPqqoffvjhwYMH27dvb3khI8qqLRB39u3bJ4R46KGHdF2/6qqrZBcHsCcCMFrgcrlGjRo1btw4IURZWVngydra2vnz5+fk5CQnJ48ePXrPnj2B53ft2nXNNddkZWVlZmaOGTNm3bp1gXNfcyNhRUXF9OnTU1NTO3fuvGbNGnNeTqdTUZSzZ88Gku3btzeWevfdd2fMmJGdnZ2enj5q1KidO3eGUvgdO3ZcdtllgUL+85//FEJMnTpVUZRFixYF3nDy5Em32+1yucrLyy3ZAhUVFTNnzkxPT8/JyVm1apW5gT2wEbZv396rV69LL700jM3Y3PMej+ehhx7q2bOn2+3Ozc2dO3fu6dOnA6tqmmlzKzFkZmauX79eCLF06dLMzMzWrr/RqhRF2bhxY6NvIchnDyzy3//938OHD2/Tps3w4cN3794deKnFfWD48OHvvvuuEOKaa66ZMGFCkO9a07SVK1cOGjQoOTm5S5cut9122/79+4N/0iA7Z3NlblSe5nYwJK6ioiIdOF9SUpL4+vzP5/Pt3r07OztbCLFhw4bAG6ZOnWreizIzM0+ePFlXVxf4vTYLrCQjI0MIUVZWpuv6tddea36DqqpCiCNHjui67nA4hBBnzpwJ5JKVlRVYyu/3B9Zg6NChQ3V1daM1mwWed7lcxiJ5eXnnzp176aWXhBA9e/YMvG316tVCiKlTp1qyBXRdnzx5ctNaFvh0gSJdfvnlQohbb721tZsxyOadPn16o+eHDh3q8XiaZhpkJY02XUBGRkar1h/itxBkAzb6osXXX1Yo+8CwYcOMV6+++uog3/WsWbMa5TJ9+vTgn7S5nTNImRuVJ4SahwRSVFREAMYFBMJPIwMHDqyvr9d1/aOPPhJCtG/f/oMPPqipqXn44YeFEEuWLDl27JgQom3btk8//fSxY8c2bdo0btw4TdN000/k3r17hRCqqv7pT3+qrKwsLCwM/K4FD8C6rt9xxx2FhYWnTp06duxY3759hRDbt2/XWwrAc+bMOX78uBE+//CHP9TU1LRt21YIsWvXLl3XR48eLYTYuHGjJVuguLg48OleffXVU6dOLVu2TFEUcX4Azs3N/eyzz8LYjM09//HHHwcyXb9+/dmzZ99+++3AB3zhhReaZhrkOzK78cYbhRDPP/+8ruutWn+I30Jzn91YZMaMGcePH//HP/4ROD4LfR8INJhv3bpV1/XmvutPPvkk8IU++uijp0+fPnz48H333bdkyZLgn7TFAHzBMpvLA5gRgHFhTcPPlClTKioqAq8+/fTTTYPTddddp+v60qVLnU6nECIrK+uhhx6qqqoKLGL8RAZOSq666iojr44dO4YSgA8fPjxv3rzBgwcbZxuvvvqq3lIANp6fP3++EGL+/Pn612c/DzzwwOHDhxVFadeuXV1dnSVb4MUXXwzy6QJFCkS18DbjBZ8PrGfcuHFGpt///veFEPfff3/TTIOs3MwcgFu7/lC+hSCfPbDIsWPHAovk5uaKr8/RQ9kHGgW8C37XzzzzjBDiyiuvNBf13LlzwT9piwH4gmUmAKM5RUVF9AGjWYFfkLVr1wohdu7caQwADhzgNxLo5lyyZMm//vWvBx54QNO0pUuXDhgwINC1ZqivrxdCBH7LmuPz+Ro989VXXw0bNuyJJ54oLi6uqqoK47MEujlTUlKEELfeeqsQYv369evWrdN1/eabb77g+a5o/Rbwer2ipU9n9AWGsRmDbF7dNMJc07TmMg2+kiBCX38QxrcQ5LMHBFoOhKn5Orx9IMh3rTczJj/4J226cwYpMxAcARgtmDVr1t13333q1Knp06cHwmegw++SSy7ZvHlzbW3toUOHVq5cmZ+ff/To0UmTJh05cmTJkiUHDx68+uqrjx079vvf/968tv79+wshtm3btnnz5tOnT69cudI8+iktLU0I8frrr1dUVPznf/6ncd3wW2+9VVFRMXny5D179qxYsSI5OTnEwj/22GPl5eV79uxZt26dEGLQoEFCiG9/+9t5eXlHjhz51a9+JYS4/fbbrdoCjT7dsmXLTpw40dxqW7sZm3t++PDhQojt27e//PLLNTU1f/3rX//4xz8KIQLPNxLKd9RIq9Z/QU2/heY+e5CVhLgPBE7uA03copnv+oorrhBC/P3vf3/00UerqqrKysqWLl26YMGC4J+0uZ0zuEblAc5DEzSaMg9B0nW9vr7+W9/6lhDizjvvDDwze/bsRjvSVVdd9ec//7npDvbMM8/o5zcSBrriDOY+YPMIJpfLFTiTKCsre/vtt5uu+eWXX9ZbaoI2n4z27NkzMJpG1/UFCxYEnuzbt6+FW0DX9TFjxjQtqrkJ2lzUVm3GIJu3xUFSRqZBVmJmboJu1fpD/xaa24CNVtitWzchxP79+0PcB+bNmxd4KS8vL8h33TT30aNHB/+kze2cQcp8wfIAAfQB48IahR9d17/88ktjBI2u636/f8WKFQMGDHC73Xl5eTfeeOPHH39cW1u7evXqyy+/PDMzs02bNgMHDnzssccCi5t/oY4ePTp16tTA5R9r164195IePHhw/PjxKSkp/fr1e+ONN8zdbD/84Q8zMjK6dev24IMPBtoVH374Yb2lAPzss8/269fP7XaPGTPm008/NV41zkh+8YtfWLgFAp9u2rRpKSkpPXr0+PWvf23uHWxa1FZtxiCbt76+fvHixd27d3c6nTk5OXPmzKmsrGy65YOvxKxRAA59/aF/C81twCDBLJR94Kuvvpo8eXJqaqoQ4vTp0819136///HHHx84cKDb7e7UqVN+fv6ePXuCf9IgO2eQMjctDxBQVFSkFBUVDR48uOmhJWBXXq93w4YNt956a3Jy8sGDBzt16mThyufNm9e/f//8/PzASOP58+enpaWdPn060BSZaDIzMwNtvNZu5NBF9LsGLkZxcXEi/iggkQ0YMCAwzZMQYsmSJdb+Iuu6/tJLL1VUVNx7773Gk/Pm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"/>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</body>
</html>




R-Squared is quite small, but then again I've had models so bad R-squared was negative (really it is impossible, so that was a really bad model using Twitter data if that explains anything). But the difference is significant. On average you would expect a team to get 4.21545 penalties per game. This is close to the mean calculated without controlling for home ice. Somewhat obviously, there is a negative effect for the team that wins. You have less penalties, you are more likely to win; which was also shown above. 

When home ice is accounted for, there is a -0.43584 drop in penalties. So that shows a slight favortism towards the home team. Let's bootstrap it again for a better estimate. 


```python
%%SAS sas_session
proc sort data=nhl.gamessamp;
 by replicate;
run;
ods trace on;
ods output ParameterEstimates=nhl.penestimates;
proc reg data=nhl.gamessamp;
by replicate;
model penaltycount= home won/;
run; quit;
ods trace off;
```


```python
%%SAS sas_session
proc sql;
    select Variable, mean(Estimate) as MeanEst
      from nhl.penestimates
      group by variable;
run;
quit;
%histograms(curvar='home')
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="r b header" scope="col">MeanEst</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">Intercept</td>
<td class="r data">4.156299</td>
</tr>
<tr>
<td class="data">home</td>
<td class="r data" style="white-space: nowrap">-0.29919</td>
</tr>
<tr>
<td class="data">won</td>
<td class="r data" style="white-space: nowrap">-0.04627</td>
</tr>
</tbody>
</table>
</div>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<hr class="pagebreak"/>
<div id="IDX1" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<div class="c">
<img style="height: 480px; width: 640px" alt="The SGRender Procedure" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAHgCAIAAAC6s0uzAAAACXBIWXMAAA7DAAAOwwHHb6hkAAAgAElEQVR4nO3de1yUZf7/8WtgBhhGDooKeIK0PGe6aFIeQoXU2gxXKdc11g77K9fK3C2zdckOa7a5WmYey2xLd3Vzt/pq66mTikdQG8SvhoqnQUg5S6AwML8/5ruzhGDMMPCZw+v52EePm4uba95z7Uzv7vseuDVGo1EBAICWpVVK9evXTzoGAABeJCMjw0c6AwAA3ogCBgBAAAUMAIAAChgAAAEUMAAAAihgAAAEUMAAAAiggAEAEEABAwAggAIGAEAABQwAgAAKGAAAARQwYLeAgACNRnPq1CnbyObNmzUazcCBA5VSNTU1gwcP7tq1a0FBgVzG5lVaWpqUlBQSEqLRaBYvXmwbX79+vUaj6dSpk/XL0NBQjUaTl5fXYsHqBABcGQUMOF91dbVSymKx3GCfuLg4jUbzxRdftFQoZ1q4cOHGjRtLS0uVUv3797eN+/n5KaV0Op1UMPEAQONRwICT+fj4pKenZ2dnt23bVjpLczlx4oRSau7cuRaL5a677rKNd+3aVSkVHR0tFUw8ANB4FDDgfLVPvR48eDAhISEsLCw0NHTYsGHr1q2rrq4eOHDgzp07lVIJCQnx8fFKqcrKyrlz53bt2tXPz69Dhw7Tp08vLi62zpafnz9p0qSgoKDIyMjFixdHRERoNBqTyWR7oN27d3fr1u3WW29VSu3cuTMpKaldu3ZBQUFDhgw5cOBA7UjvvvvubbfdFhwcPGLEiMOHDycnJ4eHh0dGRq5ater6Z9FQpNDQ0H/84x9KqZdffjk0NLT2j/To0UOj0Vhb0Gbr1q0DBw5s1arVwIEDDx06dOPJGx+1oqJi5syZkZGRAQEBQ4cOPXz4cEMBABdlNBotAOzh7+9f77spJibGukNISIhSKjc39+rVq3UqSil18uTJmJgY25ejRo2yWCwTJ06ss9uAAQMqKystFsuYMWOuf6wLFy7YHig2NlYpNWXKlOrqauuITfv27UtLS217NkSn0+Xk5NR5mg1Fqj1VSEjIDRbq+gft2rXrjSdvfNT77ruv9nhoaGhBQYFz/48Gmo/RaOQIGGhGhYWFxcXFwcHBq1evvnjx4ubNm+Pi4rp165aenm49c7tjx44vvvjCaDRu3LjRx8fnH//4R1lZ2VdffRUcHHzkyJGNGzcePXp069atPj4+n3zySWFh4euvv67RaOo8yvnz57/77ruPPvrIx8dnwoQJy5YtKywsvHjxYvfu3S9dumQ0Gm17PvTQQ3l5ec8995xSKjAwcP369fn5+V26dKmqqsrMzKw95w0iFRcXT5gwQSn10Ucf2Q5bb2Dy5Mnnz5/fv3+/j49PdnZ2Xl7eDSZvZNRvv/1206ZNbdu2TUtLKy8vf/XVV4uLi2t/HAxwfRQw4KCTJ0/a/mN206ZN9e4TGRn58ssvl5eXP/roo7feemtaWtpnn312fYNaT8wOHz48KSnJYDCMGDFiypQpSqn09PRjx44ppYYNG5aYmNi6devnn3++ffv2dX78z3/+c/fu3a3bc+fOPX78eFxcXK9evbKyspRS+fn5tj3feOON8PBw6/H3iBEjHnzwwbCwsN69eyulysvLGxnJ3oVauHBh586dBw8eHBERoZQqKytrzOQ3jmo94Zyfnz9o0KDAwMCUlBRbZsBdUMBA83rxxRePHz/++9//vqam5uWXX+7Vq9fJkyfr3dNS61PTNTU11o2qqiqllK+v7w0ewnoVWSn1/fffx8TELFmyJCMjo6SkpKH968x2g8nrjeSwOh9ObszkDUX18ann313W6+KAu6CAgWaUk5MzevToCxcuvPjii9nZ2aNGjbp48aL1Y0RarVYpdeTIEaWU9ReId+/evWHDhvLy8i+//HLt2rXW8Z49eyqldu3atWXLluLi4tdff/3SpUsNPdz27dvz8/PHjBlz+PDhhQsXBgQEOJz8BpEcntOJk1sve7dp02bLli0VFRVnz55dtGhRYmJi07MBLUYrHQDwZBkZGdu3b9++fXvtQet51N69e3/55ZezZs1avHixyWSaOHHixo0bJ02aZNttwIABEydO1Ol0w4YN27179z333POTD2f9AxRbt27dunWrbbCystKB5P369WsokgOzOX3ynj17PvbYY++9997YsWNtg7V/IQpwfRwBA81oxIgRy5cvj42NDQ0NbdWqVZ8+ff7yl788/PDDSqk//vGPY8aMCQwMzMnJKSkpWbduXUpKSnR0tFarjYyMnDZt2ldffWU9Z7t+/fpx48bp9fqbbrrprbfesn5IuN7zxiNGjHjmmWdCQkKioqLmzJljvbBqvRLsgBtEarqmT75y5cqFCxf26tXL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zZTZLt5ewBaLZffu3T169PD39x87dmxhYaF1cMCAAYcPHz59+vSDDz7Yvn37sLCwRx55pLi42GKx1DvY0Dz1DrpIZrueiItktlgsx44du/322wMCAgYPHpyZmen6mc1m85/+9KewsLA2bdr84Q9/qKmpcanMtdn+3VTnw02+vr6unzknJ2fq1KmRkZHWPUtLS10/s8ViWb16dbdu3QIDA4cPH56RkeFSmRtaUqvaBewumS0/Xvxmytx4RqORuyEBANDSuBsSAAAyKGAAAARQwAAACKCAAQAQQAEDACCAAgYAQAAFDACAAAoYAAABFDAAAAIoYAAABFDAAAAIoIABABBAAQNuqVWrVpofi4uLa2hns9ms0Wjy8vKUUpcuXerTp09lZaVjj1t7KgBNoZUOAMBBaWlpAwcOtPen2rdvf+zYsebIA8AuHAEDnubVV1/t2LGjXq+///77s7OzlVLWno6MjFy7dq3JZNJoNEqps2fParXaFStWREVFRUREbN68eeHChWFhYV26dNm2bZt1qs8///zOO+/U6/VRUVHvvfdenamUUseOHYuLiwsODo6JiTlw4IDUUwbcEQUMeJS9e/euXLly586dubm5CQkJn332mVIqPT1dKZWbmztlypTaO1dXV586derQoUMzZ86cNGnSiRMnsrOzn3jiid/97nfWHZYuXZqSklJYWLhq1arp06fn5ubWnqqsrCwhISE+Pj4nJ+e555679957i4qKWvwZA27LaDRaALgbg8FQ5728b98+i8WSmZnZpk2bDz74ID8/37ZzVVWVUio3N9disVy4cEEpZbFYzpw54+vra90hKyvL19fXbDZbLJbvvvsuMDDw+keMior65ptvak+1YcOG7t2723ZISEhYtWpVMz5nwIMYjUaOgAF3lZaWVvv9HBsbq5Tq06fPJ598sn379l69eiUkJBw9erQxU+l0OqWUr6+vUsrPz6+6uto6vnbt2vj4+C5duhgMhnPnzlnb18ZkMmVlZdk+BbZjx47Tp087+UkCnosPYQGeZvjw4cOHD6+url60aNEvf/nLzMxMx+bZtGnTrFmzli5dGhsb27p16/79+9fZISoqasiQIampqU2ODHgjjoABj3L48OEpU6YcP368qqpKr9dbD221Wq3BYLB+IKvxzp071717d2uXz5s37+TJk2azufZUY8aMMZlMb731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"/>
</div>
</div>
</div>
</body>
</html>




The original sample ended up being higher than the bootstrapped estimate. Still there is a significant negative effect on the number of penalties a team is called for if they are the home team. The next question would be if this varies by team. Do certain teams have more of a home ice advantage than others, in terms of the penalties called?


```python
%%SAS sas_session
proc sort data=nhl.games;
 by team;
run;
ods trace on;
ods output ParameterEstimates=nhl.penteamestimates;
proc reg data=nhl.games;
by team;
model penaltycount= home won/;
run; quit;
ods trace off;
```


```python
%%SAS sas_session
proc sql;
    select Variable, Team, Estimate, Probt 
      from nhl.penteamestimates
      where Variable="home"
        order by Estimate;
run;
quit;
```




<!DOCTYPE html>
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c > table, .body.c > pre, .body.c div > table,
.body.c div > pre, .body.c > table, .body.c > pre,
.body.j > table, .body.j > pre, .body.j div > table,
.body.j div > pre, .body.j > table, .body.j > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
h1.heading1 { text-align: left }
h2.heading2 { text-align: left }
h3.heading3 { text-align: left }
h4.heading4 { text-align: left }
h5.heading5 { text-align: left }
h6.heading6 { text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
a:active { color: #800080 }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.heading1, .heading2, .heading3, .heading4, .heading5, .heading6 { font-family: Arial, Helvetica, sans-serif }
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:link { color: #0000ff }
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
a:visited { color: #800080 }
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<div style="padding-bottom: 8px; padding-top: 1px">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div style="padding-bottom: 8px; padding-top: 1px">
<table class="table" style="border-spacing: 0" aria-label="Query Results">
<caption aria-label="Query Results"></caption>
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="b header" scope="col">Variable</th>
<th class="b header" scope="col">team</th>
<th class="r b header" scope="col">Parameter Estimate</th>
<th class="r b header" scope="col">Pr &gt; |t|</th>
</tr>
</thead>
<tbody>
<tr>
<td class="data">home</td>
<td class="data">CGY</td>
<td class="r data" style="white-space: nowrap">-0.72226</td>
<td class="r data">0.0013</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CHI</td>
<td class="r data" style="white-space: nowrap">-0.62266</td>
<td class="r data">0.0004</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">VAN</td>
<td class="r data" style="white-space: nowrap">-0.55893</td>
<td class="r data">0.0151</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">TOR</td>
<td class="r data" style="white-space: nowrap">-0.48348</td>
<td class="r data">0.0213</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NJ</td>
<td class="r data" style="white-space: nowrap">-0.47302</td>
<td class="r data">0.0069</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">MTL</td>
<td class="r data" style="white-space: nowrap">-0.46980</td>
<td class="r data">0.0228</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PIT</td>
<td class="r data" style="white-space: nowrap">-0.46902</td>
<td class="r data">0.0371</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NYR</td>
<td class="r data" style="white-space: nowrap">-0.42721</td>
<td class="r data">0.0142</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">WSH</td>
<td class="r data" style="white-space: nowrap">-0.41880</td>
<td class="r data">0.0390</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NSH</td>
<td class="r data" style="white-space: nowrap">-0.40351</td>
<td class="r data">0.0507</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">VGK</td>
<td class="r data" style="white-space: nowrap">-0.39992</td>
<td class="r data">0.1965</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">FLA</td>
<td class="r data" style="white-space: nowrap">-0.39745</td>
<td class="r data">0.0721</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">MIN</td>
<td class="r data" style="white-space: nowrap">-0.38571</td>
<td class="r data">0.0462</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CAR</td>
<td class="r data" style="white-space: nowrap">-0.36810</td>
<td class="r data">0.0254</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">OTT</td>
<td class="r data" style="white-space: nowrap">-0.35791</td>
<td class="r data">0.1104</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">ANA</td>
<td class="r data" style="white-space: nowrap">-0.33947</td>
<td class="r data">0.1173</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">BUF</td>
<td class="r data" style="white-space: nowrap">-0.30478</td>
<td class="r data">0.1368</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">CBJ</td>
<td class="r data" style="white-space: nowrap">-0.28451</td>
<td class="r data">0.1720</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">WPG</td>
<td class="r data" style="white-space: nowrap">-0.26757</td>
<td class="r data">0.2471</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">DET</td>
<td class="r data" style="white-space: nowrap">-0.25629</td>
<td class="r data">0.1476</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">BOS</td>
<td class="r data" style="white-space: nowrap">-0.25514</td>
<td class="r data">0.2141</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">COL</td>
<td class="r data" style="white-space: nowrap">-0.20313</td>
<td class="r data">0.3130</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PHI</td>
<td class="r data" style="white-space: nowrap">-0.12630</td>
<td class="r data">0.5781</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">EDM</td>
<td class="r data" style="white-space: nowrap">-0.11030</td>
<td class="r data">0.5855</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">NYI</td>
<td class="r data" style="white-space: nowrap">-0.08499</td>
<td class="r data">0.6757</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">PHX</td>
<td class="r data" style="white-space: nowrap">-0.04440</td>
<td class="r data">0.8409</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">SJ</td>
<td class="r data" style="white-space: nowrap">-0.04309</td>
<td class="r data">0.8336</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">DAL</td>
<td class="r data" style="white-space: nowrap">-0.03506</td>
<td class="r data">0.8597</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">LA</td>
<td class="r data" style="white-space: nowrap">-0.03000</td>
<td class="r data">0.8776</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">TB</td>
<td class="r data">0.01714</td>
<td class="r data">0.9290</td>
</tr>
<tr>
<td class="data">home</td>
<td class="data">STL</td>
<td class="r data">0.19078</td>
<td class="r data">0.4047</td>
</tr>
</tbody>
</table>
</div>
</div>
</body>
</html>




If you are a Pittsburg rival, and Sydney Crosby conspiracist, then we might see PITs top rank of home ice advantage as proof of something nefarious. I'll just leave it at that. Then there's Calgary. Or Philadelphia, which no longer seems to have a home ice advantage. 

One question though, is it that these teams have a home ice advantage, or are they just more likely to take more penalties on the road? Many of the teams that showed up with home ice advantage in terms of wins, are not on the top of the list when compared to penalties. Actually, the penalties don't really see any real difference when at home or away for most of the teams.

As with a lot of analysis this just leads to more questions. Does the nationality of the refs come into play? Some of the top teams are Canadian (then again 3 canadian teams aren't). Some of the teams are original six teams, some aren't.

## Summary

As I stated before, this is mostly for fun. It is a toy dataset that can be used to answer some questions. I would say that the first goal is important, but only because of the second goal. If a team scores both the first and second, the game is pretty much over. Home seems to provide, on average, some advantage, and on average the penalties are slightly lower for home teams than for away teams. This doesn't hold up when looking at every team though.

What does it mean for strategy? Well, coming from someone who has never played, I don't know. I do have some thoughts. One is that a team should try and score first. Is that helpful? Just go and score? Of course if it were easy....I think there may be ways to do that. Upload the minutes of your best scoring lines in the first part of the game, including scoring defensemen. Or if you know which player is the best at scoring the first goal, give them lots of time at the start.

Another strategy though, is it gets harder for the opposing team to win if the first score occurs later in in the game. An average team has a 70% chance of winning if the first goal comes in the 3rd period. So, give your most defensive minded players more minutes in the first and part of the second. Really grind it out with the other team, lots of hits, and tire them out. Make them cover the whole rink. Then switch and give more minutes to the best scorers who will have fresh legs.

But...I'm not a coach and I don't play hockey.


```python
print("Probability of winning when scoring first at home, in the first period, no overtime, and ~4 penalties: ")
print("{0:.2f}%".format(convertToProb(-2.14334+1.90498+3*0.342078+0.10468)*100))
```

    Probability of winning when scoring first at home, in the first period, no overtime, and ~4 penalties: 
    70.94%

