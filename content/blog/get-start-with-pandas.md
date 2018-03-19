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

## read json
pd.DataFrame.from_records()

## basic

column selection: df['colname'] or df[['col1', 'col2']]
pd.unique(pdSeries)

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

封面图片来自 [Pandas for Pornhub](https://dribbble.com/shots/3367311-Pandas-for-Pornhub) <a href="https://dribbble.com/cjiabka"><i class="fa fa-dribbble" aria-hidden="true"></i> Yaroslav Kuryanovich</a>
