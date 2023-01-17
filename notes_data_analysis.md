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