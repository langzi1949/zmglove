---
title: Python分析豆瓣热映电影影评
date: 2017-11-30 12:00:01
tags:
    - python
---
!["豆瓣影评词云"](/images/douban_4.jpg)
<!--more-->
* [前提条件](#m1)
* [抓取网页](#m2)
* [清洗数据](#m3)
* [词云展示](#m4)

<span id="m1"><strong>前提条件</strong></span>

---
1. python版本:**3.6**
2. 使用的各种包 **jieba numpy pandas matplotlib beautifulsoup4  wordcloud scipy**

<span id="m2"><strong>抓取网页</strong></span>

---
1.**python的urllib库进行网页抓取**

!["豆瓣热映影片"](/images/douban_1.jpg)
```python
    from urllib import request
    resp = request.urlopen('https://movie.douban.com/cinema/nowplaying/shanghai/')        
    html_data = resp.read().decode('utf-8')
```
可以访问<a href="https://movie.douban.com/cinema/nowplaying/shanghai/">https://movie.douban.com/cinema/nowplaying/shanghai/</a>，就是豆瓣热映的电影首页，其中html_data存放的数据就是网页的html代码。

2.**使用beautifulsoup对html代码进行解析**
```python
    from bs4 import BeautifulSoup as bs
    soup = bs(html_data, 'html.parser')    
    nowplaying_movie = soup.find_all('div', id='nowplaying')        
    nowplaying_movie_list = nowplaying_movie[0].find_all('li', class_='list-item')
```
!["豆瓣热映影片div"](/images/douban_2.jpg)
通过我们对页面的分析(在浏览器上进行页面检查)，发现热映的影片都在`div id="nowplaying"`标签下，find_all方法返回的列表，然后在这个`div`中继续检查，发现所有的影片在一个`ul`的`li`下面，都有一个统一的名字`list-item`，
这样就能够拿到所有热映的影片了。

3.**继续使用beautifulsoup分析影评**
上一步中得到的`data-subject`就是影片的id，通过访问<a href="https://movie.douban.com/subject/20495023/comments">https://movie.douban.com/subject/20495023/comments</a>，就能看到相关的影评，其中<font color="red">20495023</font>就是上一步得到的`data-subject`。
!["豆瓣热映影评div"](/images/douban_3.jpg)
```python
    #只取一页影评中的前20条
    requrl = 'https://movie.douban.com/subject/' + movieId + '/comments' +'?' +'start=' + str(start) + '&limit=20' 
    resp = request.urlopen(requrl) 
    html_data = resp.read().decode('utf-8') 
    soup = bs(html_data, 'html.parser') 
    comment_div_lits = soup.find_all('div', class_='comment')
     for item in comment_div_lits: 
        if item.find_all('p')[0].string is not None:     
            eachCommentList.append(item.find_all('p')[0].string)
```
通过`print(eachCommentList)`就可以得到影评的内容，至此我们已经爬取了豆瓣最近播放电影的评论数据，接下来就要对数据进行清洗和词云显示了。

<span id="m3"><strong>清洗数据</strong></span>

---
通过上面的方法得到影评的内容，但是有很多的标点符号还有一些常用的虚词(的,看,太.....)，这些都不是在统计分析的范围以内，所以要进行清洗数据。

1. **为了方便清洗数据,将列表中影评变成一个长字符串**
```python
    for k in range(len(commentList)):
        comments = comments + (str(commentList[k])).strip()
```
2. **使用正则表达式去除标点符号**
```python
    import re
    pattern = re.compile(r'[\u4e00-\u9fa5]+')
    filterdata = re.findall(pattern, comments)
    # 去除标签符号以后，变成了一个纯字符串
    cleaned_comments = ''.join(filterdata)
```
3. **进行分词**
```python
    import jieba    #分词包
    import pandas as pd  

    segment = jieba.lcut(cleaned_comments)
    words_df=pd.DataFrame({'segment':segment})
```
4. **使用停用词对数据进行清洗**
百度搜索一个`stopwords.txt`,下载下来使用。去停用词代码如下代码如下:
```python
    stopwords=pd.read_csv(file_path+"stopwords.txt",index_col=False,quoting=3,sep="\t",names=['stopword'], encoding='utf-8')#quoting=3全不引用
    words_df=words_df[~words_df.segment.isin(stopwords.stopword)] # 其中 ~是取反
```
5. **词频统计**
```python
    import numpy    #numpy计算包
    words_stat=words_df.groupby(by=['segment'])['segment'].agg({"计数":numpy.size})
    words_stat=words_stat.reset_index().sort_values(by=["计数"],ascending=False)
```
清洗完数据并且词频统计的结果为:
!["清洗数据"](/images/douban_6.jpg)

<span id="m4"><strong>词云展示</strong></span>

---
少废话，直接上代码:
```python
    import matplotlib.pyplot as plt
    import matplotlib
    matplotlib.rcParams['figure.figsize'] = (10.0, 5.0)
    from wordcloud import WordCloud#词云包

    wordcloud=WordCloud(font_path="simhei.ttf",background_color="white",max_font_size=80) #指定字体类型、字体大小和字体颜色
    word_frequence = {x[0]:x[1] for x in words_stat.head(1000).values}

    wordcloud=wordcloud.fit_words(word_frequence_list) #参数为dict类型
    
    #显示词云
    plt.imshow(wordcloud)
    plt.axis('off')
    plt.show()
```
其中`simhei.ttf`字体在mac系统可能没有，需要从百度下载，放在`/Library/Fonts/`下就行，最终图像展示为:
!["豆瓣影评词云"](/images/douban_4.jpg)
如果想换一个图片方式展示可以使用以下代码:
```python
    import matplotlib.pyplot as plt
    import matplotlib
    matplotlib.rcParams['figure.figsize'] = (10.0, 5.0)
    from wordcloud import WordCloud,ImageColorGenerator#词云包
    from scipy.misc import imread

    #读取图片信息
    bg_pic = imread('3.jpg')
    wordcloud=WordCloud(font_path="simhei.ttf",background_color="white",max_font_size=80) #指定字体类型、字体大小和字体颜色
    word_frequence = {x[0]:x[1] for x in words_stat.head(1000).values}

    wordcloud=wordcloud.fit_words(word_frequence_list) #参数为dict类型
    image_colors = ImageColorGenerator(bg_pic)

    #显示词云
    plt.imshow(wordcloud)
    plt.axis('off')
    plt.show()
```
最终呈现的词云为:
!["豆瓣影评词云"](/images/douban_5.jpg)

至此，python分析豆瓣热映电影影评的工作全部完成了，[点击获取源码](https://github.com/langzi1949/funing/blob/master/crawler/douban.py)，其中肯定有很多的瑕疵和不足，请不吝赐教。