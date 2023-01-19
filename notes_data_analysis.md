# Data Analysis 数据分析
## 数据加载
- 通过相对路径载入数据出错时, 可以利用`os.getcwd`命令查看当前目录.
- read_csv和read_table都是是加载带分隔符的数据，每一个分隔符作为一个数据的标志. read_table是以制表符 \t 作为数据的标志，也就是以行为单位进行存储。
- TSV是用制表符（tab,’\t’）作为字段值的分隔符; CSV是用半角逗号（’,’）作为字段值的分隔符.
- 查看数据基本信息`df.info()`, 判断数据是否为空`df.isnull()`
### 使用chunksize分块处理大型csv文件
- `pandas.read_csv`参数`chunksize`通过指定一个分块大小（每次读取多少行）来读取大数据文件, 可避免一次性读取内存不足, 返回的是一个可迭代对象 `TextFileReader`
- 指定`iterator=True`也可以返回一个可迭代对象`TextFileReader`. `iterator=True`和`chunksize`可以同时指定使用
- `get_chunk(size)`返回一个N行的数据块. 每次执行获取N行数据, 再次执行, 获取下一个数据块
## pandas基础
- series是一个一维数据结构, 它由index和value组成. dataframe是一个二维结构, 除了拥有index和value之外, 还拥有column. dataframe由多个series组成, 无论是行还是列, 单独拆分出来都是一个series
- 删除多余的列: `df.drop(['col_name'],axis=1,inplace=True)`
- 筛选: `df[conditions]`. 筛选过后使用`df.reset_index(drop=True)`是为了丢掉筛选后不连续的index, 重新赋予连续的index
### loc VS iloc
- `df.iloc[:,[0,1,2]].values`切片时前闭后开
- `df.loc[-2:3,'代码']`, 也可以按照条件取`df.loc[df['简称'].str.len()>4,['代码','简称']]`
- loc使用范围比iloc更广更实用, loc可以使用切片、名称(index,columns), 也可以切片和名称混合使用; 但是loc不能使用不存在的索引来充当切片取值, 像-1. iloc只能用整数来取数.
## 探索性数据分析
- 观察数据基本信息`df.describe()`
### 排序
- 让列索引降序排序`df.sort_index(axis=1, ascending=False)`
- 让任选两列数据同时降序排序`df.sort_values(by=['col_name1', 'col_name2'], ascending=False)`

<font color="#8968CD">
彩色部分表示想到什么写什么.
<a href='https://tool.oschina.net/commons?type=3'>颜色和代码对照表.</a>
为什么我用[ ]( )做不了超链接...?震惊!

做了一个关于是否存活与票价、年龄的二元回归OLS:

```
import statsmodels.formula.api as smf
reg = text[['是否幸存', '票价', '年龄']]
reg.columns = ['y', 'a', 'b']
mod = smf.ols(formula='y~a+b',data=reg)
res = mod.fit()
print(res.summary())
```

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                      y   R-squared:                       0.083
Model:                            OLS   Adj. R-squared:                  0.080
Method:                 Least Squares   F-statistic:                     32.02
Date:                Tue, 17 Jan 2023   Prob (F-statistic):           4.84e-14
Time:                        15:40:13   Log-Likelihood:                -474.62
No. Observations:                 714   AIC:                             955.2
Df Residuals:                     711   BIC:                             969.0
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept      0.4210      0.041     10.270      0.000       0.340       0.501
a              0.0026      0.000      7.708      0.000       0.002       0.003
b             -0.0035      0.001     -2.880      0.004      -0.006      -0.001
==============================================================================
Omnibus:                     4201.056   Durbin-Watson:                   1.911
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               95.470
Skew:                           0.396   Prob(JB):                     1.86e-21
Kurtosis:                       1.393   Cond. No.                         154.
==============================================================================
```
从结果来看, 这是两个显著的变量. 票价高、年龄小的人容易存活. 用一些非线性模型效果或许更好, 比如KNN, MLP, 随机森林...
</font>

## 数据清洗及特征处理
### 缺失值观察与处理
- 查看每个特征的缺失值个数
    - `df.info()`
    - `df.isnull().sum()`
- 处理缺失值
    - `df[condition] = 0`, condition可以这样写
        - `df['col_name']==None`
        - `df['col_name'].isnull()`
        - `df['col_name'] == np.nan`
    - 丢弃至少缺少一个元素的行`df.dropna()`
    - `df.fillna()`主要是使用指定方法填充 NA/NaN 值, 填充值可以是scalar, dict, Series, or DataFrame.
    - 请参考<a href='https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.dropna.html'>dropna</a>和<a href='https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.fillna.html'>fillna</a>官方文档.
- 【思考】检索空缺值用`np.nan`,`None`以及`.isnull()`哪个更好，这是为什么？如果其中某个方式无法找到缺失值，原因又是为什么?
    - Pandas用`np.nan`表示缺失数据. 计算时默认不包含空值.数值列空缺值的数据类型为float64, 所以用None索引不到.
### 重复值观察与处理
- 查看`df[df.duplicated()]`
- 删除重复值所在的行`df = df.drop_duplicates()`
### 特征观察与处理
- 分箱处理`df['new_col_name'] = pd.cut(df['col_name'], 5,labels = [1,2,3,4,5])`表示按照'col_name'列(数值类型)平均分成5份, 并分别用类别变量12345表示.
- `df['new_col_name'] = pd.cut(df['col_name'],[0,5,15,30,50,80],labels = [1,2,3,4,5])`表示将连续变量'col_name'划分为(0,5] (5,15] (15,30] (30,50] (50,80]五份，并分别用类别变量12345表示. 用列表的demarcate代表的是区间, `qcut`也一样!
- `df['new_col_name'] = pd.qcut(df['col_name'],[0,0.1,0.3,0.5,0.7,0.9],labels = [1,2,3,4,5])`表示将连续变量'col_name'划分为0% 30% 50 70% 90%五份，并分别用类别变量12345表示.
- 参考<a href='https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.qcut.html'>qcut</a>官方文档
- 查看文本
    - `df['col_name'].value_counts()` 查看类别文本变量名及种类
    - `df['col_name'].unique()`
    - `df['col_name'].nuique()`
- 转换文本
    - `df['Sex_num'] = df['Sex'].replace(['male','female'],[1,2])`
    - `df['Sex_num'] = df['Sex'].map({'male': 1, 'female': 2})`
    - 方法三: 使用sklearn.preprocessing的LabelEncoder
- 将类别文本转换为one-hot编码
    - 在机器学习中, 我们使用one-hot编码器对类别进行二进制化操作, 然后将其作为模型训练的特征, 避免原本无序的标签因为运算过程导致错误. 比如, 降谷零编码为0, 赤井秀一为1, 萩原研二为2, 而阿卡伊秀一叽不可能是Furuya酱和Hagiwara酱的平均.分别编码为001, 010, 100 就不会有这样的问题了!
    - 请查询`pd.get_dummies`
- 从纯文本Name特征里提取出Titles的特征(所谓的Titles就是Mr,Miss,Mrs等)
    - `df['Title'] = df.Name.str.extract('([A-Za-z]+)\.', expand=False)`