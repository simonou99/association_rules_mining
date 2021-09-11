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

'''
import numpy as np
import csv
from csv import reader
import pandas as pd
'''

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

'''
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
'''

### ii. Create rules sets:
'''
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
'''
