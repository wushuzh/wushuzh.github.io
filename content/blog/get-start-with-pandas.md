+++
date = "2017-12-03T09:04:10+08:00"
title = "Pandas 入门"
showonlyimage = false
image = "/img/blog/get-start-with-pandas/bambfood.png"
topImage = "/img/blog/get-start-with-pandas/bambfood.gif"
draft = false
weight = 1000
+++

Python 中 Excel 级别的数据操作库
<!--more-->

## Data type
np.array np.ndarray
pd.Series pd.DataFrame

## read csv
df = pd.read_csv('file.csv', index_col='colname', usecols=[collist])
df.to_pickle('file.picle')

user_cols = ['user_id', 'age', 'gender', 'occupation', 'zip_code']
pd.read_table('http://bit.ly/moviesusers', sep='|', header=None, names=user_cols)

## read json
pd.DataFrame.from_records()

## basic

ufo = read.csv('http://bit.ly/uforeports')

ufo['Location'] = ufo.City + ', ' + ufo.State

column selection/creation: 
  bracket notation: df['colname'] or df[['col1', 'col2']]
  dot notation: 仅仅列名不和内置属性重名(如shape)或含特殊字符(如空格) df.colname 为DataFrame 创建新列时不能使用。

pd.unique(pdSeries)

movies = pd.read_csv('http://bit.ly/imdbratings')

movies.shape 和 movies.dtypes
movies.describe()

列改名

ufo.columns
ufo.rename(columns={'Colors Reported': 'Colors_Reported', 'Shape Reported': 'Shape_Reported'}, inplace=True)

## filtering

s = df['artist'] == 'Bacon, Francis'
s.value_counts()

artist_counts = df['artist'].value_counts()
artist_counts['Bacon, Francis']

## indexing

df.loc with label: df[Row-indexer, col-indexer]
df.iloc with position

### found out why multiplication is not working
df['width'].sort_values().head()
df['width'].sort_values().tail()
#### replace 
df.loc[:, 'colname'] = df.to_numeric('colname', errors=['coerce'])

area = df['height'] * df['width']
df = df.assign(area=area)

df['area'].max()
df['area'].idxmax()
df.loc[df['area'].idxmax(), :]

## Operations on Groups

- Aggregation (聚合分析) for name, group in adf.groupby('colname')
- Transformation (补值) series.fillna(series.value_counts().index[0]) pd.concat(serveral_dfs)
- filtering (去除某一类group)

参考文档

> - Ted Petrou (2017-11-15) [How to Learn Pandas](https://medium.com/dunder-data/how-to-learn-pandas-108905ab4955)
> - Quentin Gaudron @ PyData Seattle 2017 [Introduction to Data Analytics With Pandas](https://youtu.be/5XGycFIe8qE) 
> - data school (2016-05-10) [Easier data analysis in Python with pandas(video series)](http://www.dataschool.io/easier-data-analysis-with-pandas/)

封面图片来自 [Pandas for Pornhub](https://dribbble.com/shots/3367311-Pandas-for-Pornhub) <a href="https://dribbble.com/cjiabka"><i class="fa fa-dribbble" aria-hidden="true"></i> Yaroslav Kuryanovich</a>
