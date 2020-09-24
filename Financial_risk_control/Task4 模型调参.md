```python
import pandas as pd
import numpy as np
from category_encoders.target_encoder import TargetEncoder
from sklearn.model_selection import KFold
import lightgbm as lgb
import datetime
from sklearn.preprocessing import LabelEncoder
from tqdm import tqdm

from sklearn.model_selection import StratifiedKFold
from sklearn.metrics import auc, roc_curve
seed = 2020
import seaborn as sns
import matplotlib.pyplot as plt

sns.set()
# 有五种seaborn的绘图风格，它们分别是：darkgrid, whitegrid, dark, white, ticks。默认的主题是darkgrid。
sns.set_style("whitegrid")
# 有四个预置的环境，按大小从小到大排列分别为：paper, notebook, talk, poster。其中，notebook是默认的。
sns.set_context('talk')
# 中文字体设置-黑体
plt.rcParams['font.sans-serif'] = ['SimHei']
# 解决保存图像是负号'-'显示为方块的问题
plt.rcParams['axes.unicode_minus'] = False
# 解决Seaborn中文显示问题并调整字体大小
sns.set(font='SimHei')

```


```python
# 导入数据
train = pd.read_csv('e:/data/tianchi_rc/train.csv', index_col='id')
test = pd.read_csv('e:/data/tianchi_rc/testA.csv', index_col='id')
```


```python
df = pd.concat([train, test], axis=0, ignore_index=True)
del train
del test

# label encoding
for col in tqdm([f for f in ['grade','subGrade']]):
    le = LabelEncoder()
    df[col].fillna('-1', inplace=True)
    df[col] = le.fit_transform(df[col])

def employmentLength_to_int(s):
    if pd.isnull(s):
        return s
    else:
        return np.int8(s.split()[0])
df['employmentLength'].replace(to_replace='10+ years', value='10 years', inplace=True)
df['employmentLength'].replace('< 1 year', '0 years', inplace=True)
df['employmentLength'] = df['employmentLength'].apply(employmentLength_to_int)

#转化成时间格式
df['issueDate'] = pd.to_datetime(df['issueDate'],format='%Y-%m-%d')
df['issueDateYear']=df['issueDate'].dt.year
df['issueDateMonth']=df['issueDate'].dt.month
df['issueDateMonths']=(df['issueDate'].dt.year-2007)*12+df['issueDate'].dt.month-6
'''
startdate = datetime.datetime.strptime('2007-06-01', '%Y-%m-%d')
df['issueDateDT'] = df['issueDate'].apply(lambda x: x-startdate).dt.days
'''
df['earliesCreditLineYear'] = df['earliesCreditLine'].apply(lambda s: int(s[-4:]))
df['earliesCreditLineMonth'] = df['earliesCreditLine'].apply(lambda s: s[:3])
df['earliesCreditLineMonth']=df['earliesCreditLineMonth'].map({'Jan':1,'Feb':2,'Mar':3,'Apr':4,'May':5,'Jun':6,'Jul':7,'Aug':8,'Sep':9,'Oct':10,'Nov':11,'Dec':12})
df['earliesCreditLineMonths']=(df['earliesCreditLineYear']-1944)*12+df['earliesCreditLineMonth']


train = df[~df['isDefault'].isnull()].copy()
test = df[df['isDefault'].isnull()].copy()
```

    100%|████████████████████████████████████████████████████████████████████████████████████| 2/2 [00:00<00:00,  5.22it/s]
    


```python
def reduce_mem_usage(df):
    start_mem = df.memory_usage().sum() 
    print('Memory usage of dataframe is {:.2f} MB'.format(start_mem))
    
    for col in df.columns:
        col_type = df[col].dtype
        
        if col_type != object:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == 'int':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)  
            else:
                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
        else:
            df[col] = df[col].astype(np.float64)

    end_mem = df.memory_usage().sum() 
    print('Memory usage after optimization is: {:.2f} MB'.format(end_mem))
    print('Decreased by {:.1f}%'.format(100 * (start_mem - end_mem) / start_mem))
    
    return df
```


```python
del train['issueDate']
del train['earliesCreditLine']
del test['issueDate']
del test['earliesCreditLine']
```


```python
train=reduce_mem_usage(train)
test=reduce_mem_usage(test)
```

    Memory usage of dataframe is 320000000.00 MB
    Memory usage after optimization is: 82400000.00 MB
    Decreased by 74.2%
    Memory usage of dataframe is 80000000.00 MB
    

    <ipython-input-4-57680f5f721f>:21: RuntimeWarning: invalid value encountered in less
      if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
    <ipython-input-4-57680f5f721f>:23: RuntimeWarning: invalid value encountered in less
      elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
    

    Memory usage after optimization is: 21800000.00 MB
    Decreased by 72.8%
    


```python

```
