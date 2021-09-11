# Association Rules Mining Program
Welcome, this is a association miner built with apriori algorithm.  <br />
Different from the Jupyter's prebuilt algorithm, this one is built by myself and was for one of my school projects, now it's rewritten for personal use.

The program can be use with different datasets, if modified, but currently it is only tested for purcahse rules mining from shopping transactions. The program also has features like changing its minimum support, confidence and lift.

Here are the codes and some instructions on how to use it.

*For instructing ease, we use a shopping transactions data for the following instructions.* <br />

**Hey! If you want to ignore all the codes and theory parts, just copy-paste the codes and go straight to '3.'** <br />
**However reading the following is highly recommended.**


## 1. Initialization and imports:
We will need *numpy, pandas and csv* for this program. <br />
The program is currently written to process csv only. <br />
However, it can be modified to process other format.

The csv should **only** have rows of items or self-assigned categories (e.g. 'milk', 'eggs', 'shop_department:12', 'customer_age:34'), that is, there should not be any headers.<br />
However items in each row can be stored at random places.

```
import numpy as np
import csv
from csv import reader
import pandas as pd
```

## 2. Main codes section for the whole mining procedure
This will be the codes for the algorithm.
Sorry the order for each function is a bit weird. However it should work fine.

### i. Apriori algorithm function:
The algorithm will have these input parameters:
- **rawd**: Raw csv data
- **mspt**: Minimum support value.
- **mcf**: Minimum confidence value.
- **mlft**: Minimum lift.
- (Optional) **mrspt**: Minimum relative support, mspt value but with added constraint of items frequency. *Used only when data has frequent commom items.*

EXPLANATION for **support**, **confidence** and **lift**: <br />
These are constraints for the likelihood i.e. the probability of the found rules.
Hence, they should all be set between 0 and 1. <br />
The larger the probability ceilling, the less rules will be found.

Following are the culculation for each parameters:
- sup(X,Y)=P(XY)=num(XY)/num(all_samples)
- conf(X⇐Y)=P(X|Y)=P(XY)/P(Y)
- lift(X⇐Y)=P(X|Y)/P(X)=conf(X⇐Y)/P(X)

Normally, we do not change the minimal lift if there are large amount of infrequent rules.<br />
***Hence**, assign **mlft=0** if you have very infrequent items data sets or not sure about the rules frequencies in the data.*

```
#This is the main funct to run the algorithm.
def apr(rawd, mspt=0.15,mcf=0.80,mlft=0):
    #since we don't have a freq items data yet,
    #we first convert data to a non-mutable freq list
    #then create a mutable list, with same original data
    rawset, set_mutable=formatdata(rawd)
    #get a minimun-supported (minsup) set, and its freq dict.
    sptitm,sptdct=get_spted(set_mutable, rawset, mspt)
    c=2
    sptitmlst=[sptitm]
    while len(sptitmlst[c-2])>0:
        set_mutable,sptitmlst,sptdct,mspt,c=proc_for_each_row(set_mutable,sptitmlst,sptdct,mspt,c)
    out_rules=get_rules(sptitmlst, sptdct, mcf, mlft)
    return out_rules
```

### ii. Create rules sets:
```
def get_rules(sptitmlst, sptdct, mcf, mlft):
    ruleset=[]
    for i in range(1,len(sptitmlst)):
        for frq in sptitmlst[i]:
            rawfrqlst=[frozenset([x]) for x in frq]
            if i>1:
                #if the rule's itself has >1 items, but outcome has only 1:
                #seperately calculate confidence and outcome of each rule:
                frqlst= get_output(frq, rawfrqlst, sptdct, ruleset, mcf, mlft)
                if len(frqlst) > 1:
                    get_result(frq, frqlst, sptdct, ruleset, mcf, mlft)
            else:
                get_output(frq, rawfrqlst, sptdct, ruleset, mcf, mlft)
    return ruleset
```

### iii. Data pre-processing:
We create an immutable data set in this function.
```
def formatdata(d):
    dset=[]
    for row in d:
        for itm in row:
            if not [itm] in dset:
                dset.append([itm])
    dset.sort()
    return list(map(frozenset, dset)), list(map(set, d))
```

### iv. Single Row processing:
```
def proc_for_each_row(set_mute,set_freeze,frqdct,mspt,c):
        #remove repetition and
        #get all min-supported item list again after data cleaning/processing
        cleansptitm,cleansptdct=get_spted(set_mute, get_non_rep(set_freeze[c-2], c), mspt)
        c+=1
        frqdct.update(cleansptdct)
        set_freeze.append(cleansptitm)
        return set_mute,set_freeze,frqdct,mspt,c
```

### v. Reserve the rules that match the min-support constraint:
```
def get_spted(rawd, dset, mspt):  
    frqdct={}
    sptitm=[]
    sptdct={}
    #first make a freq dict of setX
    for i in rawd:
        for x in dset:
            if x.issubset(i):
                if x not in frqdct:
                    frqdct[x]=1
                else:
                    frqdct[x]+=1
    #then calculate sup = {setX in a setY} / number_of_sets
    #and reserve sets that its sup > minsup
    total=float(len(rawd))
    for i in frqdct:
        spt=frqdct[i]/total
        if spt>=mspt:
            sptitm.insert(0,i)
        sptdct[i]=spt
    #return sets that match minsup, and their freq dict
    return sptitm,sptdct
```

### vi. Check and remove repetitions:
```
def get_non_rep(rawlst, k):
    cleanlst=[]
    size=len(rawlst)
    for i in range(size):
        for x in range(i+1,size):
            #do so by checking last k-2 sets
            #if they are the same, combine them with bitwise or.
            l1=list(rawlst[i])[:(k-2)]
            l2=list(rawlst[x])[:(k-2)]
            l1.sort()
            l2.sort()
            if l1==l2:
                cleanlst.append(rawlst[i]|rawlst[x])
    return cleanlst
```

### vii. Get a single rule with single item in apriori:
This will output a single rule as a **python list**, the output contains:
- **Apriori**, python list, e.g. ['flour'], ['party_lights', 'beer']
- **Posterior**, python list, e.g. ['milk', 'eggs'], ['chips']
- Its **lift** value.
- Its **confidence** value.
- Its **support** value.

__Yeah but what does it mean?__<br />
For example, in a shopping scenario/dataset, if you get something like [['flour'], ['milk', 'eggs'], 0, 0.8, 0.15]. <br/>
It means: "For the rule '*buying flour is associated with buying milk and eggs*' has a lift=0, confidence=0.8 and support=0.15".
```
def get_output(frqset,rawfrqlst,sptd,ruleset,mcf,mlft):
    cleanfrqset=[]
    for outcome in rawfrqlst:
        #calculate conf and lift
        cf=sptd[frqset]/sptd[frqset-outcome]
        lft=sptd[frqset]/(sptd[frqset-outcome]*sptd[outcome])
        #filter with minsup and minlift
        if cf>=mcf and lft>=mlft:
            ruleset.append([frqset-outcome,outcome,lft,cf,sptd[frqset]])
            cleanfrqset.append(outcome)
    return cleanfrqset
```

### viii. Get a single rule with more than 2 items in the apriori:
Same as the previous one, but the previous one can only get rules with single item in apriori. <br />
This one allows to process and store rules with multi items in the apriori.
```
def get_result(frqset, rawfrqlst, sptd, rulelst, mcf, mlft):
    i=len(rawfrqlst[0])
    if len(frqset)>(i+1):
        #check repetition
        frqlst=get_non_rep(rawfrqlst, i+1)
        frqlst=get_output(frqset,frqlst,sptd,rulelst,mcf,mlft)
        if len(frqlst)>1:
            get_result(frqset,frqlst,sptd,rulelst,mcf,mlft)
```

### ix. (Optional) Sort the result:
Sort all the found rules descending in following order: number of apriori, number of posterior, lift, confidence then support. 
```
def tidy_sort(rules):
    for r in rules:
        r[0]=list(r[0])
        r[1]=list(r[1])
    rules.sort(key = lambda i:(len(i[0]),len(i[1]),i[2],i[3],i[4]),reverse=True)
```

### x. (Optional) Print all the rules (CAUSTION: may print a lot!):
Prints all the rules found. <br />
Usefull for presenting if the rules found are in small number. <br />
Suggest checking the total length of the found rules list first.
```
def print_rules(rls):
    for r in rls:
        print('{',str(r[0])[1:-1].replace("'",'').replace(' ',''),'}',sep='',end='')
        print('-->',end='')
        print('{',str(r[1])[1:-1].replace("'",'').replace(' ',''),'}',sep='',end=',  ')
        print(f'Lift={round(r[2],2)}',end=',  ')
        print(f'Conf={round(r[3],2)}',end=',  ')
        print(f'Sup={round(r[4],2)}')
```


## 3. How to use (demo):
For demo ease, I've generate some random rules:
```
#Store transactions as list:
d1=[['A','B','C','D','F'],['A','B','C','D'],['A','B','C','D'],['A','B'],['B','C','E']]
```

We then run with the previous written algorithm: <br />
Use the function **apr(your_data, mspt, mcf, mlft)** or apr(your_data) if using default parameters. <br />
The output will be a nested list containing all the rules.
Each list item (or row) will be a single rule, containing:
- list[0]: Apriori, i.e. an item set
- list[1]: Posterior, i.e. the output item set that found to be associated with the apriori item set.
- list[2]: Lift value.
- list[3]: Confidence value.
- list[4]: Support value.

If you are using CSV, goto '4.' or convert it to a nested list if you know how.

In here, we use mspt=0.30 for shorter output.
```
#run apriori:
rules=apr(d1, mspt=0.30,mcf=0.80,mlft=0)
#i.e the same as rules = apr(d1), with default parameters.
tidy_sort(rules)
print_rules(rules)
print(rules)

print()
print(f'We have {len(rules)} sets of rules found.')
```
{A,C,B}-->{D},  Lift=1.67,  Conf=1.0,  Sup=0.6
{D,A,B}-->{C},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,C,B}-->{A},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,C,A}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.6
{A,C}-->{D,B},  Lift=1.67,  Conf=1.0,  Sup=0.6
{D,B}-->{A,C},  Lift=1.67,  Conf=1.0,  Sup=0.6
{D,A}-->{C,B},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,C}-->{A,B},  Lift=1.25,  Conf=1.0,  Sup=0.6
{A,C}-->{D},  Lift=1.67,  Conf=1.0,  Sup=0.6
{D,B}-->{C},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,A}-->{C},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,C}-->{A},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,B}-->{A},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D,C}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.6
{D,A}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.6
{A,C}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.6
{D}-->{A,C,B},  Lift=1.67,  Conf=1.0,  Sup=0.6
{D}-->{A,C},  Lift=1.67,  Conf=1.0,  Sup=0.6
{D}-->{C,B},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D}-->{A,B},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D}-->{A},  Lift=1.25,  Conf=1.0,  Sup=0.6
{D}-->{C},  Lift=1.25,  Conf=1.0,  Sup=0.6
{A}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.8
{C}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.8
{D}-->{B},  Lift=1.0,  Conf=1.0,  Sup=0.6
{B}-->{A},  Lift=1.0,  Conf=0.8,  Sup=0.8
{B}-->{C},  Lift=1.0,  Conf=0.8,  Sup=0.8

We have 27 sets of rules found.

**Here are all the rules found with the demo.**


## 4. CSV Input:
For CSV input, first convert the data into a nested list.
```
with open('YOUR_FILE_NAME.csv', 'r') as rawfile:
    readfile=reader(rawfile)
    data=list(map(list,readfile))
```
Then do the same as in the demo, with self-assigned parameters or go all default.
```
allrules1 = apr(data, mspt=0.15,mcf=0.80,mlft=0)
#OR
allrules2 = apr(data) #defualt parameters
```
