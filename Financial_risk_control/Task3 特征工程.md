# Task3 特征工程
## 3.1 学习目标

- 学习特征预处理、缺失值、异常值处理、数据分桶等特征处理方法
- 学习特征交互、编码、选择的相应方法
- 完成相应学习打卡任务，两个选做的作业不做强制性要求，供学有余力同学自己探索

## 3.2 内容介绍

- 数据预处理
  - 缺失值的填充
  - 时间格式处理
  - 对象类型特征转换到数值
- 异常值处理
  - 基于3segama原则
  - 基于箱型图
- 数据分箱
  - 固定宽度分箱
  - 分位数分箱
    - 离散数值型数据分箱
    - 连续数值型数据分箱
  - 卡方分箱（选做作业）
- 特征交互
  - 特征和特征之间组合
  - 特征和特征之间衍生
  - 其他特征衍生的尝试（选做作业）
- 特征编码
  - one-hot编码
  - label-encode编码
- 特征选择
    - 1 Filter
    - 2 Wrapper （RFE）
    - 3 Embedded

### 3.3.1 导入包并读取数据


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import datetime
from tqdm import tqdm
from sklearn.preprocessing import LabelEncoder
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2
from sklearn.preprocessing import MinMaxScaler
import xgboost as xgb
import lightgbm as lgb
from catboost import CatBoostRegressor
import warnings
from sklearn.model_selection import StratifiedKFold, KFold
from sklearn.metrics import accuracy_score, f1_score, roc_auc_score, log_loss
warnings.filterwarnings('ignore')
```


```python
data_train =pd.read_csv('e:/data/tianchi_rc/train.csv', index_col='id')
data_test_a = pd.read_csv('e:/data/tianchi_rc/testA.csv', index_col='id')
```

### 3.3.2特征预处理

- 数据EDA部分我们已经对数据的大概和某些特征分布有了了解，数据预处理部分一般我们要处理一些EDA阶段分析出来的问题，这里介绍了数据缺失值的填充，时间格式特征的转化处理，某些对象类别特征的处理。

首先我们查找出数据中的对象特征和数值特征


```python
numerical_fea = list(data_train.select_dtypes(exclude=['object']).columns)
category_fea = list(filter(lambda x: x not in numerical_fea,list(data_train.columns)))
label = 'isDefault'
numerical_fea.remove(label)
```

在比赛中数据预处理是必不可少的一部分，对于缺失值的填充往往会影响比赛的结果，在比赛中不妨尝试多种填充然后比较结果选择结果最优的一种；
比赛数据相比真实场景的数据相对要“干净”一些，但是还是会有一定的“脏”数据存在，清洗一些异常值往往会获得意想不到的效果。

#### 缺失值填充

- 把所有缺失值替换为指定的值0

    data_train = data_train.fillna(0)


- 向用缺失值上面的值替换缺失值

    data_train = data_train.fillna(axis=0,method='ffill')


- 纵向用缺失值下面的值替换缺失值,且设置最多只填充两个连续的缺失值

    data_train = data_train.fillna(axis=0,method='bfill',limit=2)


```python
#查看缺失值情况
data_train.isnull().sum()
```




    loanAmnt                  0
    term                      0
    interestRate              0
    installment               0
    grade                     0
    subGrade                  0
    employmentTitle           1
    employmentLength      46799
    homeOwnership             0
    annualIncome              0
    verificationStatus        0
    issueDate                 0
    isDefault                 0
    purpose                   0
    postCode                  1
    regionCode                0
    dti                     239
    delinquency_2years        0
    ficoRangeLow              0
    ficoRangeHigh             0
    openAcc                   0
    pubRec                    0
    pubRecBankruptcies      405
    revolBal                  0
    revolUtil               531
    totalAcc                  0
    initialListStatus         0
    applicationType           0
    earliesCreditLine         0
    title                     1
    policyCode                0
    n0                    40270
    n1                    40270
    n2                    40270
    n3                    40270
    n4                    33239
    n5                    40270
    n6                    40270
    n7                    40270
    n8                    40271
    n9                    40270
    n10                   33239
    n11                   69752
    n12                   40270
    n13                   40270
    n14                   40270
    dtype: int64




```python
#按照平均数填充数值型特征
data_train[numerical_fea] = data_train[numerical_fea].fillna(data_train[numerical_fea].median())
data_test_a[numerical_fea] = data_test_a[numerical_fea].fillna(data_train[numerical_fea].median())
#按照众数填充类别型特征
data_train[category_fea] = data_train[category_fea].fillna(data_train[category_fea].mode())
data_test_a[category_fea] = data_test_a[category_fea].fillna(data_train[category_fea].mode())
```


```python
data_train.isnull().sum()
```




    loanAmnt                  0
    term                      0
    interestRate              0
    installment               0
    grade                     0
    subGrade                  0
    employmentTitle           0
    employmentLength      46799
    homeOwnership             0
    annualIncome              0
    verificationStatus        0
    issueDate                 0
    isDefault                 0
    purpose                   0
    postCode                  0
    regionCode                0
    dti                       0
    delinquency_2years        0
    ficoRangeLow              0
    ficoRangeHigh             0
    openAcc                   0
    pubRec                    0
    pubRecBankruptcies        0
    revolBal                  0
    revolUtil                 0
    totalAcc                  0
    initialListStatus         0
    applicationType           0
    earliesCreditLine         0
    title                     0
    policyCode                0
    n0                        0
    n1                        0
    n2                        0
    n3                        0
    n4                        0
    n5                        0
    n6                        0
    n7                        0
    n8                        0
    n9                        0
    n10                       0
    n11                       0
    n12                       0
    n13                       0
    n14                       0
    dtype: int64




```python
#查看类别特征
category_fea
```




    ['grade', 'subGrade', 'employmentLength', 'issueDate', 'earliesCreditLine']



- category_fea：对象型类别特征需要进行预处理，其中['issueDate']为时间格式特征。

#### 时间格式处理


```python
#转化成时间格式
for data in [data_train, data_test_a]:
    data['issueDate'] = pd.to_datetime(data['issueDate'],format='%Y-%m-%d')
    startdate = datetime.datetime.strptime('2007-06-01', '%Y-%m-%d')
    #构造时间特征
    data['issueDateDT'] = data['issueDate'].apply(lambda x: x-startdate).dt.days
```


```python
data_train['employmentLength'].value_counts(dropna=False).sort_index()
```




    1 year        52489
    10+ years    262753
    2 years       72358
    3 years       64152
    4 years       47985
    5 years       50102
    6 years       37254
    7 years       35407
    8 years       36192
    9 years       30272
    < 1 year      64237
    NaN           46799
    Name: employmentLength, dtype: int64




```python
#### 对象类型特征转换到数值
def employmentLength_to_int(s):
    if pd.isnull(s):
        return s
    else:
        return np.int8(s.split()[0])
for data in [data_train, data_test_a]:
    data['employmentLength'].replace(to_replace='10+ years', value='10 years', inplace=True)
    data['employmentLength'].replace('< 1 year', '0 years', inplace=True)
    data['employmentLength'] = data['employmentLength'].apply(employmentLength_to_int)
```


```python
data['employmentLength'].value_counts(dropna=False).sort_index()
```




    0.0     15989
    1.0     13182
    2.0     18207
    3.0     16011
    4.0     11833
    5.0     12543
    6.0      9328
    7.0      8823
    8.0      8976
    9.0      7594
    10.0    65772
    NaN     11742
    Name: employmentLength, dtype: int64



#### 待补......


```python

```


```python

```


```python

```
