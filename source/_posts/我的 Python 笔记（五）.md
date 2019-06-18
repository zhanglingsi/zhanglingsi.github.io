---
title: 我的 Python 笔记（五）
categories: python
copyright: true
---
# 我的 Python 笔记（五）

## Python 中的 正则表达式 爬虫  

### 常见匹配模式

| 模式| 描述|
|----|----|
| \w	| 匹配字母数字及下划线 |
| \W	| 匹配非字母数字下划线 |
| \s	| 匹配任意空白字符，等价于 [\t\n\r\f]. |
| \S	| 匹配任意非空字符 |
| \d	| 匹配任意数字，等价于 [0-9] |
| \D	| 匹配任意非数字 |
| \A	| 匹配字符串开始 |
| \Z	| 匹配字符串结束，如果是存在换行，只匹配到换行前的结束字符串 |
| \z	| 匹配字符串结束 |
| \G	| 匹配最后匹配完成的位置 |
| \n | 匹配一个换行符 |
| \t | 匹配一个制表符 |
| ^	| 匹配字符串的开头 |
| $	| 匹配字符串的末尾。|
| .	| 匹配任意字符，除了换行符，当re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符。|
| [...]	| 用来表示一组字符,单独列出：[amk] 匹配 'a'，'m'或'k' |
| [^...]	| 不在[]中的字符：[^abc] 匹配除了a,b,c之外的字符。| 
| *	| 匹配0个或多个的表达式。|
| +	| 匹配1个或多个的表达式。|
| ?	| 匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式| 
| {n}	| 精确匹配n个前面表达式。|
| {n, m} | 匹配 n 到 m 次由前面的正则表达式定义的片段，贪婪方式| 
| a&#124;b | 匹配a或b |
| ( )	| 匹配括号内的表达式，也表示一个组 |

### re.match

re.match 尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none。
re.match(pattern, string, flags=0)

### 最常规的匹配

```python
import re

content = 'Hello 123 4567 World_This is a Regex Demo'
print(len(content))
result = re.match('^Hello\s\d\d\d\s\d{4}\s\w{10}.*Demo$', content)
print(result)
print(result.group())
print(result.span())
```

    41
    <_sre.SRE_Match object; span=(0, 41), match='Hello 123 4567 World_This is a Regex Demo'>
    Hello 123 4567 World_This is a Regex Demo
    (0, 41)

### 泛匹配

```python
import re

content = 'Hello 123 4567 World_This is a Regex Demo'
result = re.match('^Hello.*Demo$', content)
print(result)
print(result.group())
print(result.span())
```

    <_sre.SRE_Match object; span=(0, 41), match='Hello 123 4567 World_This is a Regex Demo'>
    Hello 123 4567 World_This is a Regex Demo
    (0, 41)

<!-- more -->

### 匹配目标

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^Hello\s(\d+)\sWorld.*Demo$', content)
print(result)
print(result.group(1))
print(result.span())
```

    <_sre.SRE_Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
    1234567
    (0, 40)

### 贪婪匹配

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^He.*(\d+).*Demo$', content)
print(result)
print(result.group(1))
```

    <_sre.SRE_Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
    7

### 非贪婪匹配

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^He.*?(\d+).*Demo$', content)
print(result)
print(result.group(1))
```

    <_sre.SRE_Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
    1234567

### 匹配模式

```python
import re

content = '''Hello 1234567 World_This
is a Regex Demo
'''
result = re.match('^He.*?(\d+).*?Demo$', content, re.S)
print(result.group(1))
```

    1234567

### 转义

```python
import re

content = 'price is $5.00'
result = re.match('price is $5.00', content)
print(result)
```

    None

```python
import re

content = 'price is $5.00'
result = re.match('price is \$5\.00', content)
print(result)
```

    <_sre.SRE_Match object; span=(0, 14), match='price is $5.00'>

总结：尽量使用泛匹配、使用括号得到匹配目标、尽量使用非贪婪模式、有换行符就用re.S

## re.search

re.search 扫描整个字符串并返回第一个成功的匹配。

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
result = re.match('Hello.*?(\d+).*?Demo', content)
print(result)
```

    None

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
result = re.search('Hello.*?(\d+).*?Demo', content)
print(result)
print(result.group(1))
```

    <_sre.SRE_Match object; span=(13, 53), match='Hello 1234567 World_This is a Regex Demo'>
    1234567

总结：为匹配方便，能用search就不用match

### 匹配演练

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君"><i class="fa fa-user"></i>但愿人长久</a>
        </li>
    </ul>
</div>'''
result = re.search('<li.*?active.*?singer="(.*?)">(.*?)</a>', html, re.S)
if result:
    print(result.group(1), result.group(2))
```

    齐秦 往事随风

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君">但愿人长久</a>
        </li>
    </ul>
</div>'''
result = re.search('<li.*?singer="(.*?)">(.*?)</a>', html, re.S)
if result:
    print(result.group(1), result.group(2))
```

    任贤齐 沧海一声笑

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君">但愿人长久</a>
        </li>
    </ul>
</div>'''
result = re.search('<li.*?singer="(.*?)">(.*?)</a>', html)
if result:
    print(result.group(1), result.group(2))
```

    beyond 光辉岁月

## re.findall

搜索字符串，以列表形式返回全部能匹配的子串。

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君">但愿人长久</a>
        </li>
    </ul>
</div>'''
results = re.findall('<li.*?href="(.*?)".*?singer="(.*?)">(.*?)</a>', html, re.S)
print(results)
print(type(results))
for result in results:
    print(result)
    print(result[0], result[1], result[2])
```

    [('/2.mp3', '任贤齐', '沧海一声笑'), ('/3.mp3', '齐秦', '往事随风'), ('/4.mp3', 'beyond', '光辉岁月'), ('/5.mp3', '陈慧琳', '记事本'), ('/6.mp3', '邓丽君', '但愿人长久')]
    <class 'list'>
    ('/2.mp3', '任贤齐', '沧海一声笑')
    /2.mp3 任贤齐 沧海一声笑
    ('/3.mp3', '齐秦', '往事随风')
    /3.mp3 齐秦 往事随风
    ('/4.mp3', 'beyond', '光辉岁月')
    /4.mp3 beyond 光辉岁月
    ('/5.mp3', '陈慧琳', '记事本')
    /5.mp3 陈慧琳 记事本
    ('/6.mp3', '邓丽君', '但愿人长久')
    /6.mp3 邓丽君 但愿人长久

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君">但愿人长久</a>
        </li>
    </ul>
</div>'''
results = re.findall('<li.*?>\s*?(<a.*?>)?(\w+)(</a>)?\s*?</li>', html, re.S)
print(results)
for result in results:
    print(result[1])
```

    [('', '一路上有你', ''), ('<a href="/2.mp3" singer="任贤齐">', '沧海一声笑', '</a>'), ('<a href="/3.mp3" singer="齐秦">', '往事随风', '</a>'), ('<a href="/4.mp3" singer="beyond">', '光辉岁月', '</a>'), ('<a href="/5.mp3" singer="陈慧琳">', '记事本', '</a>'), ('<a href="/6.mp3" singer="邓丽君">', '但愿人长久', '</a>')]
    一路上有你
    沧海一声笑
    往事随风
    光辉岁月
    记事本
    但愿人长久

### re.sub

替换字符串中每一个匹配的子串后返回替换后的字符串。

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
content = re.sub('\d+', '', content)
print(content)
```

    Extra stings Hello  World_This is a Regex Demo Extra stings

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
content = re.sub('\d+', 'Replacement', content)
print(content)
```

    Extra stings Hello Replacement World_This is a Regex Demo Extra stings

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
content = re.sub('(\d+)', r'\1 8910', content)
print(content)
```

    Extra stings Hello 1234567 8910 World_This is a Regex Demo Extra stings

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君">但愿人长久</a>
        </li>
    </ul>
</div>'''

```

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君">但愿人长久</a>
        </li>
    </ul>
</div>'''
html = re.sub('<a.*?>|</a>', '', html)
print(html)
results = re.findall('<li.*?>(.*?)</li>', html, re.S)
print(results)
for result in results:
    print(result.strip())
```

    <div id="songs-list">
        <h2 class="title">经典老歌</h2>
        <p class="introduction">
            经典老歌列表
        </p>
        <ul id="list" class="list-group">
            <li data-view="2">一路上有你</li>
            <li data-view="7">
                沧海一声笑
            </li>
            <li data-view="4" class="active">
                往事随风
            </li>
            <li data-view="6">光辉岁月</li>
            <li data-view="5">记事本</li>
            <li data-view="5">
                但愿人长久
            </li>
        </ul>
    </div>
    ['一路上有你', '\n            沧海一声笑\n        ', '\n            往事随风\n        ', '光辉岁月', '记事本', '\n            但愿人长久\n        ']
    一路上有你
    沧海一声笑
    往事随风
    光辉岁月
    记事本
    但愿人长久

## re.compile

将正则字符串编译成正则表达式对象

```python
将一个正则表达式串编译成正则对象，以便于复用该匹配模式
```

```python
import re

content = '''Hello 1234567 World_This
is a Regex Demo'''
pattern = re.compile('Hello.*Demo', re.S)
result = re.match(pattern, content)
#result = re.match('Hello.*Demo', content, re.S)
print(result)
```

    <_sre.SRE_Match object; span=(0, 40), match='Hello 1234567 World_This\nis a Regex Demo'>

## 实战练习

```python
import requests
import re
content = requests.get('https://book.douban.com/').text
pattern = re.compile('<li.*?cover.*?href="(.*?)".*?title="(.*?)".*?more-meta.*?author">(.*?)</span>.*?year">(.*?)</span>.*?</li>', re.S)
results = re.findall(pattern, content)
for result in results:
    url, name, author, date = result
    author = re.sub('\s', '', author)
    date = re.sub('\s', '', date)
    print(url, name, author, date)
```

    https://book.douban.com/subject/26925834/?icn=index-editionrecommend 别走出这一步 [英]S.J.沃森 2017-1
    https://book.douban.com/subject/26953532/?icn=index-editionrecommend 白先勇细说红楼梦 白先勇 2017-2-1
    https://book.douban.com/subject/26959159/?icn=index-editionrecommend 岁月凶猛 冯仑 2017-2
    https://book.douban.com/subject/26949210/?icn=index-editionrecommend 如果没有今天，明天会不会有昨天？ [瑞士]伊夫·博萨尔特（YvesBossart） 2017-1
    https://book.douban.com/subject/27001447/?icn=index-editionrecommend 人类这100年 阿夏 2017-2
    https://book.douban.com/subject/26864566/?icn=index-latestbook-subject 眼泪的化学 [澳]彼得·凯里 2017-2
    https://book.douban.com/subject/26991064/?icn=index-latestbook-subject 青年斯大林 [英]西蒙·蒙蒂菲奥里 2017-3
    https://book.douban.com/subject/26938056/?icn=index-latestbook-subject 带艾伯特回家 [美]霍默·希卡姆 2017-3
    https://book.douban.com/subject/26954757/?icn=index-latestbook-subject 乳房 [美]弗洛伦斯·威廉姆斯 2017-2
    https://book.douban.com/subject/26956479/?icn=index-latestbook-subject 草原动物园 马伯庸 2017-3
    https://book.douban.com/subject/26956018/?icn=index-latestbook-subject 贩卖音乐 [美]大卫·伊斯曼 2017-3-1
    https://book.douban.com/subject/26703649/?icn=index-latestbook-subject 被占的宅子 [阿根廷]胡利奥·科塔萨尔 2017-3
    https://book.douban.com/subject/26578402/?icn=index-latestbook-subject 信仰与观看 [法]罗兰·雷希特(RolandRecht) 2017-2-17
    https://book.douban.com/subject/26939171/?icn=index-latestbook-subject 妹妹的坟墓 [美]罗伯特·杜格尼(RobertDugoni) 2017-3-1
    https://book.douban.com/subject/26972465/?icn=index-latestbook-subject 全栈市场人 Lydia 2017-2-1
    https://book.douban.com/subject/26986928/?icn=index-latestbook-subject 终极X战警2 [英]马克·米勒&nbsp;/&nbsp;[美]亚当·库伯特 2017-3-15
    https://book.douban.com/subject/26948144/?icn=index-latestbook-subject 格调（修订第3版） [美]保罗·福塞尔（PaulFussell） 2017-2
    https://book.douban.com/subject/26945792/?icn=index-latestbook-subject 原谅石 [美]洛里·斯皮尔曼 2017-2
    https://book.douban.com/subject/26974207/?icn=index-latestbook-subject 庇护二世闻见录 [意]皮科洛米尼 2017-2
    https://book.douban.com/subject/26983143/?icn=index-latestbook-subject 遇见野兔的那一年 [芬]阿托·帕西林纳 2017-3-1
    https://book.douban.com/subject/26976429/?icn=index-latestbook-subject 鲍勃·迪伦：诗人之歌 [法]让-多米尼克·布里埃 2017-4
    https://book.douban.com/subject/26962860/?icn=index-latestbook-subject 牙医谋杀案 [英]阿加莎·克里斯蒂 2017-3
    https://book.douban.com/subject/26923022/?icn=index-latestbook-subject 石挥谈艺录：把生命交给舞台 石挥 2017-2
    https://book.douban.com/subject/26897190/?icn=index-latestbook-subject 理想 [美]安·兰德 2017-2
    https://book.douban.com/subject/26985981/?icn=index-latestbook-subject 青苔不会消失 袁凌 2017-4
    https://book.douban.com/subject/26984949/?icn=index-latestbook-subject 地下铁道 [美]科尔森·怀特黑德（ColsonWhitehead） 2017-3
    https://book.douban.com/subject/26944012/?icn=index-latestbook-subject 极简进步史 [英]罗纳德·赖特 2017-4-1
    https://book.douban.com/subject/26969002/?icn=index-latestbook-subject 驻马店伤心故事集 郑在欢 2017-2
    https://book.douban.com/subject/26854223/?icn=index-latestbook-subject 致薇拉 [美]弗拉基米尔·纳博科夫 2017-3
    https://book.douban.com/subject/26841616/?icn=index-latestbook-subject 北方档案 [法]玛格丽特·尤瑟纳尔 2017-2
    https://book.douban.com/subject/26980391/?icn=index-latestbook-subject 食帖15：便当灵感集 林江 2017-2
    https://book.douban.com/subject/26958882/?icn=index-latestbook-subject 生火 [法]克里斯多夫·夏布特（ChristopheChabouté）编绘 2017-3
    https://book.douban.com/subject/26989163/?icn=index-latestbook-subject 文明之光（第四册） 吴军 2017-3-1
    https://book.douban.com/subject/26878906/?icn=index-latestbook-subject 公牛山 [美]布赖恩·帕诺威奇 2017-2
    https://book.douban.com/subject/26989534/?icn=index-latestbook-subject 几乎消失的偷闲艺术 [加拿大]达尼·拉费里埃 2017-4
    https://book.douban.com/subject/26939973/?icn=index-latestbook-subject 散步去 [日]谷口治郎 2017-3
    https://book.douban.com/subject/26865333/?icn=index-latestbook-subject 中国1945 [美]理查德·伯恩斯坦(RichardBernstein) 2017-3-1
    https://book.douban.com/subject/26989242/?icn=index-latestbook-subject 有匪2：离恨楼 Priest 2017-3
    https://book.douban.com/subject/26985790/?icn=index-latestbook-subject 女人、火与危险事物 [美]乔治·莱考夫 2017-3
    https://book.douban.com/subject/26972277/?icn=index-latestbook-subject 寻找时间的人 [爱尔兰]凯特·汤普森 2017-3
    https://www.douban.com/note/610758170/ 白先勇细说红楼梦【全二册】 白先勇 2017-2-1
    https://read.douban.com/ebook/31540864/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 奇爱博士 [英]彼得·乔治 2016-8-1
    https://read.douban.com/ebook/31433872/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 在时光中盛开的女子 李筱懿 2017-3
    https://read.douban.com/ebook/31178635/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 如何高效记忆（原书第2版） [美]肯尼思•希格比（KennethL.Higbee） 2017-3-5
    https://read.douban.com/ebook/31358183/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 愿无岁月可回头 回忆专用小马甲 2016-9
    https://read.douban.com/ebook/31341636/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 走神的艺术与科学 [新西兰]迈克尔·C.科尔巴里斯 2017-3-1
    https://read.douban.com/ebook/27621094/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 神秘的量子生命 [英]吉姆•艾尔－哈利利/约翰乔•麦克法登 2016-8
    https://read.douban.com/ebook/31221966/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 寻找时间的人 [爱尔兰]凯特·汤普森 2017-3
    https://read.douban.com/ebook/31481323/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 山之四季 [日]高村光太郎 2017-1
    https://read.douban.com/ebook/31154855/?dcs=book-hot&amp;dcm=douban&amp;dct=read-subject 东北游记 [美]迈克尔·麦尔 2017-1
    
