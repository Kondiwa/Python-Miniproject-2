# Python-Miniproject-2
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144


In [10]:
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
In [11]:
from static_grader import grader

PW Miniproject
Introduction
The objective of this miniproject is to exercise your ability to use basic Python data structures, define functions, and control program flow. We will be using these concepts to perform some fundamental data wrangling tasks such as joining data sets together, splitting data into groups, and aggregating data into summary statistics. Please do not use pandas or numpy to answer these questions.
We will be working with medical data from the British NHS on prescription drugs. Since this is real data, it contains many ambiguities that we will need to confront in our analysis. This is commonplace in data science, and is one of the lessons you will learn in this miniproject.

Downloading the data
We first need to download the data we'll be using from Amazon S3:
In [12]:
%%bash
mkdir pw-data
wget http://dataincubator-wqu.s3.amazonaws.com/pwdata/201701scripts_sample.json.gz -nc -P ./pw-data
wget http://dataincubator-wqu.s3.amazonaws.com/pwdata/practices.json.gz -nc -P ./pw-data

mkdir: cannot create directory ‘pw-data’: File exists
File ‘./pw-data/201701scripts_sample.json.gz’ already there; not retrieving.

File ‘./pw-data/practices.json.gz’ already there; not retrieving.


Loading the data
The first step of the project is to read in the data. We will discuss reading and writing various kinds of files later in the course, but the code below should get you started.
In [13]:
import gzip
import simplejson as json
In [14]:
with gzip.open('./pw-data/201701scripts_sample.json.gz', 'rb') as f:
    scripts = json.load(f)

with gzip.open('./pw-data/practices.json.gz', 'rb') as f:
    practices = json.load(f)

This data set comes from Britain's National Health Service. The scripts variable is a list of prescriptions issued by NHS doctors. Each prescription is represented by a dictionary with various data fields: 'practice', 'bnf_code', 'bnf_name', 'quantity', 'items', 'nic', and 'act_cost'.
In [15]:
scripts[:2]
Out[15]:
[{'bnf_code': '0101010G0AAABAB',
  'items': 2,
  'practice': 'N81013',
  'bnf_name': 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F',
  'nic': 5.98,
  'act_cost': 5.56,
  'quantity': 1000},
 {'bnf_code': '0101021B0AAAHAH',
  'items': 1,
  'practice': 'N81013',
  'bnf_name': 'Alginate_Raft-Forming Oral Susp S/F',
  'nic': 1.95,
  'act_cost': 1.82,
  'quantity': 500}]

A glossary of terms and FAQ is available from the NHS regarding the data. Below we supply a data dictionary briefly describing what these fields mean.
Data field
Description
'practice'
Code designating the medical practice issuing the prescription
'bnf_code'
British National Formulary drug code
'bnf_name'
British National Formulary drug name
'quantity'
Number of capsules/quantity of liquid/grams of powder prescribed
'items'
Number of refills (e.g. if 'quantity' is 30 capsules, 3 'items' means 3 bottles of 30 capsules)
'nic'
Net ingredient cost
'act_cost'
Total cost including containers, fees, and discounts

The practices variable is a list of member medical practices of the NHS. Each practice is represented by a dictionary containing identifying information for the medical practice. Most of the data fields are self-explanatory. Notice the values in the 'code' field of practices match the values in the 'practice' field of scripts.
In [16]:
practices[:2]
Out[16]:
[{'code': 'A81001',
  'name': 'THE DENSHAM SURGERY',
  'addr_1': 'THE HEALTH CENTRE',
  'addr_2': 'LAWSON STREET',
  'borough': 'STOCKTON ON TEES',
  'village': 'CLEVELAND',
  'post_code': 'TS18 1HU'},
 {'code': 'A81002',
  'name': 'QUEENS PARK MEDICAL CENTRE',
  'addr_1': 'QUEENS PARK MEDICAL CTR',
  'addr_2': 'FARRER STREET',
  'borough': 'STOCKTON ON TEES',
  'village': 'CLEVELAND',
  'post_code': 'TS18 2AW'}]

In the following questions we will ask you to explore this data set. You may need to combine pieces of the data set together in order to answer some questions. Not every element of the data set will be used in answering the questions.

Question 1: summary_statistics
Our beneficiary data (scripts) contains quantitative data on the number of items dispensed ('items'), the total quantity of item dispensed ('quantity'), the net cost of the ingredients ('nic'), and the actual cost to the patient ('act_cost'). Whenever working with a new data set, it can be useful to calculate summary statistics to develop a feeling for the volume and character of the data. This makes it easier to spot trends and significant features during further stages of analysis.
Calculate the sum, mean, standard deviation, and quartile statistics for each of these quantities. Format your results for each quantity as a list: [sum, mean, standard deviation, 1st quartile, median, 3rd quartile]. We'll create a tuple with these lists for each quantity as a final result.
In [17]:
from math import sqrt as m_sqrt
from statistics import median as s_med
def describe(key):

    total = sum(script[key] for script in scripts)
    avg = total / len(scripts)
    med = s_med(script[key] for script in scripts)
    s = 0
    list_q25 = []
    list_q75 = []
    for script in scripts:
        s += (script[key] - avg) ** 2
        if script[key] < med:
            list_q25.append(script[key])
        elif script[key] > med:
            list_q75.append(script[key])

    s = m_sqrt(s / (len(scripts) - 1))
    q25 = s_med(list_q25)
    q75 = s_med(list_q75)

    return (total, avg, s, q25, med, q75)
In [18]:
summary = [('items', describe('items')),
           ('quantity', describe('quantity')),
           ('nic', describe('nic')),
           ('act_cost', describe('act_cost'))]
In [19]:
grader.score.pw__summary_statistics(summary)

==================
Your score:  0.9166666666666667
==================

Question 2: most_common_item
Often we are not interested only in how the data is distributed in our entire data set, but within particular groups -- for example, how many items of each drug (i.e. 'bnf_name') were prescribed? Calculate the total items prescribed for each 'bnf_name'. What is the most commonly prescribed 'bnf_name' in our data?
To calculate this, we first need to split our data set into groups corresponding with the different values of 'bnf_name'. Then we can sum the number of items dispensed within in each group. Finally we can find the largest sum.
We'll use 'bnf_name' to construct our groups. You should have 5619 unique values for 'bnf_name'.
In [20]:
bnf_names=[]
for item in range(len(scripts)):
    if scripts[item]['bnf_name'] not in bnf_names:
        bnf_names.append(scripts[item]['bnf_name'])
assert(len(bnf_names)==5619)

We want to construct "groups" identified by 'bnf_name', where each group is a collection of prescriptions (i.e. dictionaries from scripts). We'll construct a dictionary called groups, using bnf_names as the keys. We'll represent a group with a list, since we can easily append new members to the group. To split our scripts into groups by 'bnf_name', we should iterate over scripts, appending prescription dictionaries to each group as we encounter them.
In [21]:
groups = { name: [] for name in bnf_names }

for script in scripts:
    groups[script['bnf_name']].append(script['items'])

Now that we've constructed our groups we should sum up 'items' in each group and find the 'bnf_name' with the largest sum. The result, max_item, should have the form [(bnf_name, item total)], e.g. [('Foobar', 2000)].
In [22]:
max_name=""
max_value=0

for keys in groups:
    if sum(groups[keys])> max_value:
        max_name=keys
        max_value=sum(groups[keys])
        
max_item = [(max_name,max_value)]

TIP: If you are getting an error from the grader below, please make sure your answer conforms to the correct format of [(bnf_name, item total)].
In [23]:
grader.score.pw__most_common_item(max_item)

==================
Your score:  1.0
==================

Challenge: Write a function that constructs groups as we did above. The function should accept a list of dictionaries (e.g. scripts or practices) and a tuple of fields to groupby (e.g. ('bnf_name') or ('bnf_name', 'post_code')) and returns a dictionary of groups. The following questions will require you to aggregate data in groups, so this could be a useful function for the rest of the miniproject.

Question 3: postal_totals
Our data set is broken up among different files. This is typical for tabular data to reduce redundancy. Each table typically contains data about a particular type of event, processes, or physical object. Data on prescriptions and medical practices are in separate files in our case. If we want to find the total items prescribed in each postal code, we will have to join our prescription data (scripts) to our clinic data (practices).
Find the total items prescribed in each postal code, representing the results as a list of tuples (post code, total items prescribed). Sort your results ascending alphabetically by post code and take only results from the first 100 post codes. Only include post codes if there is at least one prescription from a practice in that post code.
NOTE: Some practices have multiple postal codes associated with them. Use the alphabetically first postal code.

We can join scripts and practices based on the fact that 'practice' in scripts matches 'code' in practices'. However, we must first deal with the repeated values of 'code' in practices. We want the alphabetically first postal codes.
In [16]:
practice_postal = {}
for practice in practices:
    if practice['code'] not in practice_postal:
        practice_postal[practice['code']] = practice['post_code']
    else:
        if practice_postal[practice['code']] > practice['post_code']:
            practice_postal[practice['code']]=practice['post_code']

Challenge: This is an aggregation of the practice data grouped by practice codes. Write an alternative implementation of the above cell using the group_by_field function you defined previously.
In [17]:
assert practice_postal['K82019'] == 'HP21 8TR'

Challenge: This is an aggregation of the practice data grouped by practice codes. Write an alternative implementation of the above cell using the group_by_field function you defined previously.
In [18]:
assert practice_postal['K82019'] == 'HP21 8TR'

Now we can join practice_postal to scripts.
In [19]:
joined = scripts[:]

for script in joined:
    script['post_code']=practice_postal[script['practice']]

Finally we'll group the prescription dictionaries in joined by 'post_code' and sum up the items prescribed in each group, as we did in the previous question.
In [20]:
the_codes=[]
[the_codes.append(script['post_code']) for script in joined]
the_codes=sorted(list(set(the_codes)))
items_by_post={code: [] for code in the_codes}
for script in joined:
    items_by_post[script['post_code']].append(script['items'])
post_items=[(code,sum(items_by_post[code])) for code in the_codes]
In [21]:
postal_totals = post_items[:100]

grader.score.pw__postal_totals(postal_totals)

==================
Your score:  1.0
==================

Question 4: items_by_region
Now we'll combine the techniques we've developed to answer a more complex question. Find the most commonly dispensed item in each postal code, representing the results as a list of tuples (post_code, bnf_name, amount dispensed as proportion of total). Sort your results ascending alphabetically by post code and take only results from the first 100 post codes.
NOTE: We'll continue to use the joined variable we created before, where we've chosen the alphabetically first postal code for each practice. Additionally, some postal codes will have multiple 'bnf_name' with the same number of items prescribed for the maximum. In this case, we'll take the alphabetically first 'bnf_name'.

Now we need to calculate the total items of each 'bnf_name' prescribed in each 'post_code'. Use the techniques we developed in the previous questions to calculate these totals. You should have 141196 ('post_code', 'bnf_name') groups.

Let's use total_by_item_post to find the maximum item total for each postal code. To do this, we will want to regroup total_by_item_post by 'post_code' only, not by ('post_code', 'bnf_name'). First let's turn total_by_item_post into a list of dictionaries (similar to scripts or practices) and then group it by 'post_code'. You should have 118 groups in total_by_item_post after grouping it by 'post_code'.

Now we will aggregate the groups in total_by_item_post to create max_item_by_post. Some 'bnf_name' have the same item total within a given postal code. Therefore, if more than one 'bnf_name' has the maximum item total in a given postal code, we'll take the alphabetically first 'bnf_name'. We can do this by sorting each group according to the item total and 'bnf_name'.

In order to express the item totals as a proportion of the total amount of items prescribed across all 'bnf_name' in a postal code, we'll need to use the total items prescribed that previously calculated as items_by_post. Calculate the proportions for the most common 'bnf_names' for each postal code. Format your answer as a list of tuples: [(post_code, bnf_name, total)]
In [22]:
import pandas as pd
practices_df=pd.DataFrame(practices)
test_practice=practices_df[['code','post_code']]
practice_sorted=test_practice.sort_values(['code','post_code'], ascending=True)
practice_postal=practice_sorted.drop_duplicates(subset='code', keep='first')
In [24]:
practices = practices.sort_values('post_code') #Sort to arrange post_code by alphabets
practices = practices[~practices.duplicated(["code"])] 
merged_df = scripts.merge(practices, left_on='practice', right_on='code')

---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-24-35e40fe5f4dc> in <module>()
----> 1 practices = practices.sort_values('post_code') #Sort to arrange post_code by alphabets
      2 practices = practices[~practices.duplicated(["code"])]
      3 merged_df = scripts.merge(practices, left_on='practice', right_on='code')

AttributeError: 'list' object has no attribute 'sort_values'
In [25]:
scripts_df = pd.DataFrame(scripts)
practices_df = pd.DataFrame(practices)
postal = practices_df.sort_values(by=['code','post_code'],ascending=True)
postal.reset_index(inplace=True,drop=True)
postal = postal.drop_duplicates(subset='code', keep="first")
df = pd.merge(scripts_df,postal,left_on=['practice'],right_on=['code'],how='left')
df.head()
Out[25]:

act_cost
bnf_code
bnf_name
items
nic
post_code_x
practice
quantity
addr_1
addr_2
borough
code
name
post_code_y
village
0
5.56
0101010G0AAABAB
Co-Magaldrox_Susp 195mg/220mg/5ml S/F
2
5.98
SK11 6JL
N81013
1000
HIGH STREET SURGERY
WATERS GREEN MEDICAL CTR
SUNDERLAND STREET
N81013
HIGH STREET SURGERY
SK11 6JL
MACCLESFIELD CHESHIRE
1
1.82
0101021B0AAAHAH
Alginate_Raft-Forming Oral Susp S/F
1
1.95
SK11 6JL
N81013
500
HIGH STREET SURGERY
WATERS GREEN MEDICAL CTR
SUNDERLAND STREET
N81013
HIGH STREET SURGERY
SK11 6JL
MACCLESFIELD CHESHIRE
2
59.95
0101021B0AAALAL
Sod Algin/Pot Bicarb_Susp S/F
12
64.51
SK11 6JL
N81013
6300
HIGH STREET SURGERY
WATERS GREEN MEDICAL CTR
SUNDERLAND STREET
N81013
HIGH STREET SURGERY
SK11 6JL
MACCLESFIELD CHESHIRE
3
8.55
0101021B0AAAPAP
Sod Alginate/Pot Bicarb_Tab Chble 500mg
3
9.21
SK11 6JL
N81013
180
HIGH STREET SURGERY
WATERS GREEN MEDICAL CTR
SUNDERLAND STREET
N81013
HIGH STREET SURGERY
SK11 6JL
MACCLESFIELD CHESHIRE
4
26.84
0101021B0BEADAJ
Gaviscon Infant_Sach 2g (Dual Pack) S/F
6
28.92
SK11 6JL
N81013
90
HIGH STREET SURGERY
WATERS GREEN MEDICAL CTR
SUNDERLAND STREET
N81013
HIGH STREET SURGERY
SK11 6JL
MACCLESFIELD CHESHIRE
In [26]:
df = df[['post_code','bnf_name','items','quantity','act_cost','nic', 'bnf_code', 'practice','addr_1', 'addr_2', 'borough', 'name','village']]
df = df.sort_values('post_code')
df.reset_index(drop = True, inplace = True)

---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-26-264f276845c4> in <module>()
----> 1 df = df[['post_code','bnf_name','items','quantity','act_cost','nic', 'bnf_code', 'practice','addr_1', 'addr_2', 'borough', 'name','village']]
      2 df = df.sort_values('post_code')
      3 df.reset_index(drop = True, inplace = True)

/opt/conda/lib/python3.6/site-packages/pandas/core/frame.py in __getitem__(self, key)
   1956         if isinstance(key, (Series, np.ndarray, Index, list)):
   1957             # either boolean or fancy integer index
-> 1958             return self._getitem_array(key)
   1959         elif isinstance(key, DataFrame):
   1960             return self._getitem_frame(key)

/opt/conda/lib/python3.6/site-packages/pandas/core/frame.py in _getitem_array(self, key)
   2000             return self.take(indexer, axis=0, convert=False)
   2001         else:
-> 2002             indexer = self.loc._convert_to_indexer(key, axis=1)
   2003             return self.take(indexer, axis=1, convert=True)
   2004 

/opt/conda/lib/python3.6/site-packages/pandas/core/indexing.py in _convert_to_indexer(self, obj, axis, is_setter)
   1229                 mask = check == -1
   1230                 if mask.any():
-> 1231                     raise KeyError('%s not in index' % objarr[mask])
   1232 
   1233                 return _values_from_object(indexer)

KeyError: "['post_code'] not in index"
In [27]:
prueba = df.groupby(['post_code','bnf_name']).sum().sort_values('items', ascending=False).sort_index(level='post_code', sort_remaining=False)
prueba.head()

---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-27-3a86b77a00bc> in <module>()
----> 1 prueba = df.groupby(['post_code','bnf_name']).sum().sort_values('items', ascending=False).sort_index(level='post_code', sort_remaining=False)
      2 prueba.head()

/opt/conda/lib/python3.6/site-packages/pandas/core/generic.py in groupby(self, by, axis, level, as_index, sort, group_keys, squeeze, **kwargs)
   4414         return groupby(self, by=by, axis=axis, level=level, as_index=as_index,
   4415                        sort=sort, group_keys=group_keys, squeeze=squeeze,
-> 4416                        **kwargs)
   4417 
   4418     def asfreq(self, freq, method=None, how=None, normalize=False,

/opt/conda/lib/python3.6/site-packages/pandas/core/groupby.py in groupby(obj, by, **kwds)
   1697         raise TypeError('invalid type: %s' % type(obj))
   1698 
-> 1699     return klass(obj, by, **kwds)
   1700 
   1701 

/opt/conda/lib/python3.6/site-packages/pandas/core/groupby.py in __init__(self, obj, keys, axis, level, grouper, exclusions, selection, as_index, sort, group_keys, squeeze, **kwargs)
    390                                                     level=level,
    391                                                     sort=sort,
--> 392                                                     mutated=self.mutated)
    393 
    394         self.obj = obj

/opt/conda/lib/python3.6/site-packages/pandas/core/groupby.py in _get_grouper(obj, key, axis, level, sort, mutated)
   2688                 in_axis, name, level, gpr = False, None, gpr, None
   2689             else:
-> 2690                 raise KeyError(gpr)
   2691         elif isinstance(gpr, Grouper) and gpr.key is not None:
   2692             # Add key to exclusions

KeyError: 'post_code'
In [ ]:
prueba.reset_index(level=1,drop=False,inplace=True)
prueba.head()
In [ ]:
import numpy as np
post_codes = np.unique(prueba.index.values)
len(post_codes)
In [ ]:
answer = df.pivot_table(index='post_code',values='items',aggfunc='sum')
In [ ]:
d = []
for i in post_codes:
    d.append({'post_code': i, 'bnf_name': (prueba.loc[i]).iloc[0,:][0], 'items':
              (prueba.loc[i]).iloc[0,:][1]})
    lista = pd.DataFrame(d)
    lista.head()              
In [ ]:
mezcla = pd.merge(lista,answer,left_on='post_code',right_index=True ,how='inner')
mezcla.head()
In [ ]:
mezcla['share'] = mezcla['items_x'].div(mezcla['items_y'])
mezcla.head()
In [ ]:
max_item_by_post = list(zip(mezcla['post_code'],mezcla['bnf_name'].values,mezcla['share'].values))
In [ ]:
len(max_item_by_post[0:100])
In [3]:
items_by_region = max_item_by_post[0:100]
In [9]:
 joined = scripts[:]
for script in joined:
    script['post_code'] = practice_postal[script['practice']]
    X=defaultdict(list)
    for join in joined:
        X[join['post_code'], join['bnf_name']].append((join['items'],join['bnf_name']))
        for postcode , value_lst in dict(X).items():
            sum_lst=sum([i for i,_ in value_lst])
            X[join['post_code'], join['bnf_name']]=[(j/sum_lst,k) for j,k in value_lst]
            for postcode , value_lst in dict(X).items():
                X[join['post_code'], join['bnf_name']]=sorted(X[join['post_code'], join['bnf_name']],key=itemgetter(0),reverse=True)
            #X[postcode]=sorted(X[postcode],key=itemgetter(1))
            items_by_region =[]
            for post_code , value_lst in dict(X).items():
                items_by_region.append((post_code,value_lst[0][1],value_lst[0][0]))   
                items_by_region=items_by_region[:100]

---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-9-fd7908acd990> in <module>()
----> 1 joined = scripts[:]
      2 for script in joined:
      3    script['post_code'] = practice_postal[script['practice']]
      4    X=defaultdict(list)
      5    for join in joined:

NameError: name 'scripts' is not defined
In [5]:
grader.score.pw__items_by_region(items_by_region)

==================
Your score:  0.01
==================
