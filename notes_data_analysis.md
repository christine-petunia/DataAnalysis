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
    - 方法三: 使用`sklearn.preprocessing`的LabelEncoder
- 将类别文本转换为one-hot编码
    - 在机器学习中, 我们使用one-hot编码器对类别进行二进制化操作, 然后将其作为模型训练的特征, 避免原本无序的标签因为运算过程导致错误. 比如, 降谷零编码为0, 赤井秀一为1, 萩原研二为2, 而阿卡伊秀一叽不可能是Furuya酱和Hagiwara酱的平均.分别编码为001, 010, 100 就不会有这样的问题了!
    - 请查询`pd.get_dummies`
- 从纯文本Name特征里提取出Titles的特征(所谓的Titles就是Mr,Miss,Mrs等)
    - `df['Title'] = df.Name.str.extract('([A-Za-z]+)\.', expand=False)`

## 数据重构
### 合并列表
- `pd.concat(list, axix=0)` list是两个dataframe组成的列表
- 使用`df.join`(左右合并)和`df.append`(上下合并)
- 使用`pd.merge`方法(左右合并)
- `df.stack()`即“堆叠”, 作用是将列旋转到行, 可以形成多层索引; `df.unstack()`即`df.stack()`的反操作, 将行旋转到列.
### groupby
- 可以参考<a href='https://blog.csdn.net/FrankieHello/article/details/97272990'>groupby</a>相关博客
- 分组时可以指定多个列名
    - `grouped_muti = df.groupby(['Gender', 'Age'])`
- 通过对`DataFrame`对象调用`groupby()`函数返回的结果是一个`DataFrameGroupBy`对象, 而不是一个`DataFrame`或者`Series`对象, 所以它们中的一些方法或者函数是无法直接调用的.
    - 通过调用`get_group()`函数可以返回一个按照分组得到的`DataFrame`对象, 所以接下来的使用就可以按照`DataFrame`对象来使用
- 在没有调用`get_group()`时, 数据结构还是`DataFrameGroupBy`, 很多`DataFrame`的函数和方法可以调用, 如`max()`、`count()`、`std()`等，返回的结果是一个`DataFrame`对象(虽然有点难理解, 但理解成统计量组成的的df)
    - 一个例子
        ```
        # grouped by gender
        print(grouped.count())
        print(grouped.max()[['Age', 'Score']])
        print(grouped.mean()[['Age', 'Score']])

                Name  Age  Score
        Gender                  
        Female     3    3      3
        Male       5    5      5
                Age  Score
        Gender            
        Female   22     98
        Male     21    100
                Age      Score
        Gender                 
        Female  19.0  95.666667
        Male    19.6  89.000000
        ```
- 如果可以使用的函数无法满足需求, 也可以使用聚合函数`aggregate`, 传递`numpy`或者自定义函数，前提是返回一个聚合值. `aggregate`函数不同于`apply`，前者是对所有的数值进行聚合的操作，而后者则是对每个数值进行单独的操作：
    - 一个例子
        ```
        def getSum(data):
            total = 0
            for d in data:
                total+=d
            return total

        print(grouped.aggregate(np.median))
        print(grouped.aggregate({'Age':np.median, 'Score':np.sum}))
        print(grouped.aggregate({'Age':getSum}))
        ```
- 绘图: `grouped['Age'].plot(kind='kde', legend=True)`将绘制出两条折线, 这是因为`grouped['Age']`是一个`SeriesGroupby`对象, 也就是每一个组都有一个`Series`. 直接使用`plot`函数相当于遍历了每一个组内的Age数据.


## 数据可视化
### 数据可视化的逻辑
- 比较型数据:
    - 柱状图(单一, 重叠, 并列, 堆叠), 条形图
    - 面积图
    - 气泡图:x, y, 气泡大小, 颜色深浅
    - 雷达图(一体多维)和星状图(多体多维)
- 分布型数据:
    - 直方图: 展示离散型分组数据的分布情况
    - 箱线图: 检测数据中的异常值或离群点
    - 概率密度图: 描述连续型随机变量其分布规律
    - 散点图/气泡图
    - 热力图: 用于表示地图中点的密度的热图，也可用于业务分析、网页分析等
- 关系型数据

### 举例
- 分组后的数据绘制堆叠直方图: 可视化展示泰坦尼克号数据集中男女中生存人与死亡人数的比例图(柱状图)
    - `df.groupby(['Sex','Survived'])['Survived'].count().unstack().plot(kind='bar',stacked='True')`
    - kind参数可以是'line'、'bar'、'barh'、'kde'
- 可视化展示泰坦尼克号数据集中不同仓位等级的人年龄分布情况(折线图)
    ```
    text.Age[text.Pclass == 1].plot(kind='kde')
    text.Age[text.Pclass == 2].plot(kind='kde')
    text.Age[text.Pclass == 3].plot(kind='kde')
    plt.xlabel("age");
    plt.legend((1,2,3),loc="best");
    ```

### seaborn
- seaborn是python中的一个可视化库, 是对matplotlib进行二次封装而成的. 它的有点包括: 绘图接口更为集成, 可通过少量参数设置实现大量封装绘图; 多数图表具有统计学含义, 例如分布、关系、统计、回归等; 对Pandas和Numpy数据类型友好; 风格设置更为多样. 故在进行EDA (Exploratory Data Analysi)时, seaborn往往更为高效.
- 学习资料
    - <a href='https://seaborn.pydata.org'>Official Documentation</a>
    <a href='https://zhuanlan.zhihu.com/p/342945532'>Seaborn入门详细教程-知乎</a>
    - <a href='https://blog.csdn.net/qq_46092061/article/details/121294977'>全网最全的Seaborn详细教程-数据分析必备手册</a>
- Why import as sns?
    - Samuel Norman "Sam" Seaborn is a fictional character portrayed by Rob Lowe on the television serial drama The West Wing. So, it's a joked initialism.