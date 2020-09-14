### 1.PyQuery

```python
1.导入
	from pyquery import PyQuery as pq
    doc = pq(html)
    
2.基于CSS选择器
	doc('#container .list li')  --->返回li节点，PyQuery类型
    
3.查找节点
    1.子节点：children()，可加CSS选择器做筛选  --->返回PyQuery类型
        items = doc('.list')
        lis = items.children('.active') --->获取class属性为‘active’的子节点

    2.子孙节点：find()，可加CSS选择器做筛选  --->返回PyQuery类型
        items = doc('.list')
        lis = items.find('.active') --->获取class属性为‘active’的子孙节点

    3.父节点：parent()  --->返回PyQuery类型
        items = doc('.list')
        par = items.parent()  --->获取items的父节点

    4.祖先节点：partens()，可加CSS选择器做筛选  --->返回PyQuery类型
        items = doc('.list')
        par = items.parents('.wrap')  --->获取class属性为‘wrap’的祖先节点

    5.兄弟节点：siblings()，可加CSS选择器做筛选  --->返回PyQuery类型
        items = doc('.list .tem-0')
        bro = items.siblings('.active') --->返回class属性为‘active’的兄弟节点
            
4.遍历
	4.1.返回单个节点
    	li = doc('.tem-o')
        
    4.2.返回多个节点：调用items()方法，得到一个生成器
    	li_list = doc('li').items()
        for li in li_list:
            print(li,type(li))   --->li也是PyQuery类型
            
5.获取信息
	5.1.获取属性：attr()
    	items = doc('.item-o a')
        content = items.attr('href')  --->调用attr()方法
        content = items.attr.href     --->调用attr属性
        
        需要注意的是，在进行属性获取时，可观察返回的节点是一个还是多个；
        如果是一个，直接通过attr获取；如果是多个，需要遍历才能依次获取每个节点的属性；
        
    5.2.获取文本：text()
    	items = doc('.item-o a')
        content = items.text()
        
    5.3.获取html代码：html()
    	items = doc('.item-o a')
        content = items.html()
        
    需要注意的是，如果得到多个节点，如果想获取每个节点内的html代码，则需要遍历每个节点；
    获取文本的text()则不需要遍历，它将获取所有节点文本，并合并成一个字符串；
    
6.节点操作
	6.1.addClass和removeClass
    	li = doc('.item-o.active')
        li.removeClass('active')   --->将li节点的active这个class删除掉
        li.addClass('active')      --->添加class属性，内容’active‘
        
    6.2.attr()  --->对节点属性进行操作
    	li = doc('.item-o')
        li.attr('name','link')
        若只传入第一个参数，是获取这个属性值；
        如果传入第二个参数，就是修改该属性值；
        
        text()和html()
        一样的，如果不传参数，就是获取节点文本和html代码；
        如果传入参数，则就是进行节点相关赋值；
        
    6.3.remove()  --->删除某个节点
    	items = doc('.wrap')
        items.find('p').remove()  --->删除p节点
```

### 2.Beautiful Soup

```python
1.导入
	from bs4 import BeautifulSoup
    soup = BeautifulSoup(html,'lxml')
    
2.节点选择器
	直接调用节点的名称，再调用string，就可以节点内的文本
    soup.a.string   --->获取第一个a标签的文本，返回文本字符串
    soup.a          --->获取第一个a标签的html代码（Tag类型），可嵌套选择
    soup.a.attrs['class']
    soup.a['class'] --->获取属性；属性唯一，返回字符串；属性不唯一，返回列表；
    
3.关联选择
	3.1.子节点
    	soup.a.contents  --->返回列表
        soup.a.children  --->返回生成器
        
    3.2.子孙节点
    	soup.a.descendands  --->返回生成器
        
    3.3.父节点
    	soup.a.parent  --->返回第一个a节点的父节点
        
    3.4.祖先节点
    	soup.a.parents  --->返回生成器
        
    3.5.兄弟节点
    	soup.a.next_sibling  --->返回下一个兄弟节点
        soup.a.previous_sibling  --->返回上一个兄弟节点
        soup.a.next_siblings  --->返回后面的兄弟节点，生成器类型
        soup.a.previous_siblings  --->返回前面的兄弟节点，生成器类型
        
    以上方法返回结果是单个节点，可直接调用string、attrs等属性获得其文本和属性内容；
    如果返回的是多个节点的生成器，则可转化为列表后，取出某个元素然后再调用string、attrs等属性；
    
4.方法选择器
	4.1.find_all(name,attrs,recusive,text,**kwargs)  --->查询所有符合条件的元素
    	1.name  --->根据节点名查询，返回列表，Tag类型
        	soup.find_all(name='li')  
            
        2.attrs  --->根据属性来查询，返回列表，Tag类型
        	soup.find_all(attrs={'id':'list-1'})
            soup.find_all(id='list-1')
            soup.find_all(class_='element')  --->class为python关键字，后边需加'_'
            
        3.text  --->该参数可用来匹配节点的文本，传入形式可以是字符串，也可以是正则表达式对象
        	soup.find_all(text=re.compile('link'))
            						--->返回所有匹配正则表达式的节点文本组成的列表
                
	4.2.find(name,attrs,recusive,text,**kwargs)   --->查询第一个符合条件的元素
    	用法与find_all()方法相同，返回的是单个Tag类型
        
5.CSS选择器
	调用select()方法，传入相应的CSS选择器即可；
    soup.select('CSS选择器')   --->返回列表，元素为Tag类型
    
    1.支持嵌套选择
    	for ul in soup.select('ul'):
            print(ul.select('li'))
            
    2.获取属性
    	for ul in soup.select('ul'):
            content = ul['id']
            content = ul.attrs['id']
            
    3.获取文本
    	for ul in soup.select('ul'):
            content = ul.string       --->获取直系文本
            content = ul.get_text()   --->获取节点内的所有文本
```

### 3.Xpath

```python
1.导入
	from lxml import etree
    html = etree.HTML(response)
    result = html.xpath('//li')  --->result为列表，列表元素
    
    # xpath常用规则
        /  --->从当前节点选取直接子节点
        // --->从当前节点选取子孙节点
        .  --->选取当前节点
        .. --->选取当前节点的父节点
        @  --->选取属性
    
2.获取属性内容：
	/@属性名 --->返回列表
	result = html.xpath('//li/@class')
    
3.获取文本：
	/text() 直系文本；//text() 全部文本  --->返回列表
	result = html.xpath('//div[@class="item-o"]//text()')
    
4.属性多值匹配：
	比如class = 'A B'，含有多个值，则需要使用contains()函数
	result = html.xpath('//div[contains(@class,"A")]/text()')
    
5.按序选择
	result = html.xpath('//li[1]/text()')  --->选择第一个li节点，这里下标从1开始
    result = html.xpath('//li[last()]/text()')  --->选择最后一个li节点
    result = html.xpath('//li[position()<3]/text()')  --->选择位置小于3的li节点
    result = html.xpath('//li[last()-2]/text()')  --->选择倒数第3个li节点
```

##### 表一：选取节点

| 表达式   | 描述                               | 实例              |                           |
| -------- | ---------------------------------- | ----------------- | ------------------------- |
| nodename | 选取nodename节点的所有子节点       | xpath('//div')    | 选取了div节点的所有子节点 |
| /        | 从根节点选取                       | xpath('/div')     | 从根节点选取div节点       |
| //       | 选取所有的当前节点，不考虑节点位置 | xpath('//div')    | 选取所有的div节点         |
| .        | 选取当前节点                       | xpath('./div')    | 选取当前节点下的div节点   |
| ..       | 选取当前节点的父节点               | xpath('..')       | 回到上一个节点            |
| @        | 选取属性                           | xpath('//@class') | 选取所有的class属性       |

##### 表二：谓语

谓语被嵌在方括号内，用来查找某个特定的节点或含某个特定值的节点

| 表达式                            | 结果                                 |
| --------------------------------- | ------------------------------------ |
| xpath('/body/div[1]')             | 选取body下的第一个div节点            |
| xpath('/body/div[last()]')        | 选取body下最后一个div节点            |
| xpath('/body/div[last()-1]')      | 选取body下倒数第二个个div节点        |
| xpath('/body/div[position()<3]')  | 选取body下前两个div节点              |
| xpath('/body/div[@class]')        | 选取body下带有class属性的div节点     |
| xpath('/body/div[@class="main"]') | 选取body下class属性为main的div节点   |
| xpath('/body/div[price>35.00]')   | 选取body下price元素值大于35的div节点 |

##### 表三：通配符

通过通配符来选取未知的xml元素

| 表达式            | 结果                    |
| ----------------- | ----------------------- |
| xpath('/div/*')   | 选取div下的所有子节点   |
| xpath('/div[@*]') | 选取多有带属性的div节点 |

##### 表四：取多个路径

使用‘|’运算符可以选取多个路径

| 表达式                  | 结果                     |
| ----------------------- | ------------------------ |
| xpath('//div\|//table') | 选取多有的div和table节点 |

##### 表五：xpath轴

轴可以定义相对于当前节点的节点集

| 轴名称            | 表达式                          | 描述                                           |
| ----------------- | ------------------------------- | ---------------------------------------------- |
| ancestor          | xpath('./ancestor::*')          | 选取当前节点的所有先辈节点（父，祖父）         |
| ancestor-or-self  | xpath('./ancestor-or-self::*')  | 选取当前节点的所有先辈节点以及节点本身         |
| attribute         | xpath('./attribute::*')         | 选取当前节点的所有属性                         |
| child             | xpath('./child::*')             | 返回当前节点的所有子节点                       |
| descendant        | xpath('./descendant::*')        | 返回当前节点的所有后代节点（子节点，子孙节点） |
| following         | xpath('./following::*')         | 选取文档中当前节点结束标签后的所有节点         |
| following-sibling | xpath('./following-sibling::*') | 选取当前节点之后的兄弟节点                     |
| parent            | xpath('./parent::*')            | 选取当前节点的父节点                           |
| preceding         | xpath('./preceding::*')         | 选取文档中当前节点开始标签前的所有节点         |
| preceding-sibling | xpath('./preceding-sibling::*') | 选取当前节点之前的兄弟节点                     |
| self              | xpath('./self::*')              | 选取当前节点                                   |

##### 表六：功能函数

使用功能函数更够更好的进行模糊搜索

| 函数        | 用法                                                      | 解释                        |
| ----------- | --------------------------------------------------------- | --------------------------- |
| starts-with | xpath('//div[starts-with(@id,"ma")]')                     | 选取id值以ma开头的div节点   |
| contains    | xpath('//div[contains(@id,"ma")]')                        | 选取id值包含ma的div节点     |
| and         | xpath('//div[contains(@id,"ma") and contains(@id,"in")]') | 选取id值包含ma和in的div节点 |
| text()      | xpath('//div[contains(text(),"ma")]')                     | 选取节点文本包含ma的div节点 |



```python
pip install parsel

from parsel import Selector
sel = Selector(html)
# xpath解析
sel.xpath().extract_first() # 第一个
sel.xpath().extract()       # 全部
# css解析
sel.css()
```



### 4.re

```python
# 元字符
 .   匹配除 换行符（'\n'） 以外的任意字符
 \d  匹配所有数字
 \w  匹配字母/数字/下划线
 \s  匹配任意空白（空格 换行符 制表符 换页符）
     \t  匹配制表符
     \n  匹配换行符
 \D  匹配所有的非数字
 \W  匹配除数字/字母/下划线之外的所有字符
 \S  匹配非空白
 []  字符组 ：只要在中括号内的所有字符都是符合规则的字符
     [+*.()]，这些元字符放到中括号之中，就会恢复本来的意义，只代表自己本身
 [^ ]非字符组 ：只要在中括号内的所有字符都是不符合规则的字符
 ^   匹配字符串的开始
 $   匹配字符串的结尾
 |   表示或，注意，如果两个规则有重叠部分，总是长的在前面，短的在后面
 ()  表示分组，给一部分正则规定为一组，|这个符号的作用域就可以缩小了

# 量词
 {n}  匹配只能出现n次
 {n,} 匹配至少出现n次
 {n,m}匹配至少出现n次，至多出现m次

  ？  匹配0次或1次   表示可有可无 但是有只能有一个 比如小数点
  +   匹配1次或多次
  *   匹配0次或多次  表示可有可无 但是有可以有多个 比如小数点后n位 
```

##### re.findall ('正则规则','匹配字符串')

找到所有符合规则的项，并返回列表。

```python
import re

ret = re.findall('\d+','alex83')
print(ret)

['83']

findall 会匹配字符串中所有符合规则的项，并返回一个列表，如果未匹配到返回空列表
```

##### re.search ('正则规则','匹配字符串')

找到第一个符合规则的项，并返回一个对象。

```python
import re

ret = re.search('\d+','alex834')
print(ret)  # 如果能匹配上返回一个对象，如果不能匹配上返回None
if ret:
    print(ret.group())
    
<_sre.SRE_Match object; span=(4, 7), match='834'>
834
    
# 会从头到尾从带匹配匹配字符串中取出第一个符合条件的项
# 如果匹配到了，返回一个对象，用group取值
# 如果没匹配到，返回None，不能用group
```

##### re.match ('正则规则','匹配字符串')

从头开始，找到第一个符合规则的项，并返回一个对象。

```python
import re

ret = re.match('\d+','2345alex83')
print(ret)
if ret:
     print(ret.group())
     
<_sre.SRE_Match object; span=(0, 4), match='2345'>
2345    
     
# 会从头匹配字符串中取出从第一个字符开始是否符合规则
# 如果符合，就返回对象，用group取值
# 如果不符合，就返回None
# match = search + ^正则
```

##### re.finditer ('正则规则','匹配字符串')

找到所有符合规则的项，并返回一个迭代器。

```
在查询的结果超过1个的情况下，能够有效的节省内存，降低空间复杂度，从而也降低了时间复杂度
```

```python
import re

ret = re.finditer('\d','safhl02urhefy023908'*20000000)  # ret是迭代器
for i in ret:    # 迭代出来的每一项都是一个对象
    print(i.group())  # 通过group取值即可
```

##### re.complie ('正则规则')

预编译一个正则规则，节省多次使用同一个正则的编译时间。

```
在同一个正则表达式重复使用多次的时候使用能够减少时间的开销
```

```python
import re

ret = re.compile('\d')
r3 = ret.finditer('taibai40')
for i in r3:
    print(i.group())
    
4
0
```

##### re.split ('正则规则','匹配字符串')

根据正则规则切割，返回列表，默认不保留切掉的内容。

```python
import re

ret = re.split('\d\d','alex83wusir74taibai')  
print(ret)
['alex', 'wusir', 'taibai']

import re

ret = re.split('\d(\d)','alex83wusir74taibai')  # 默认自动保留分组中的内容
print(ret)
['alex', '3', 'wusir', '4', 'taibai']
```

##### re.sub ('正则规则','替换的字符串',匹配字符串',数字)

默认替换所有，可以使用替换深度参数。

替换（数字指的是匹配替换多少次）

```python
import re

ret = re.sub('\d','D','alex83wusir74taibai',1)
print(ret)
alexD3wusir74taibai

ret = re.sub('\d+','D','alex8323wusir745taibai',1)
print(ret)
alexDwusir745taibai

ret = re.sub('\d+','D','alex8323wusir745taibai',2)
print(ret)
alexDwusirDtaibai

ret = re.subn('\d','D','alex83wusir74taibai')
print(ret)
('alexDDwusirDDtaibai', 4)  # 4 指的是匹配替换了4次
```

##### 分组的概念

```python
import re

s1 = '<h1>wahaha</h1>'
ret = re.search('<(\w+)>(.*?)</\w+>',s1)
print(ret)
print(ret.group(0))   # group参数默认为0 表示取整个正则匹配的结果
print(ret.group(1))   # 取第一个分组中的内容
print(ret.group(2))   # 取第二个分组中的内容

<_sre.SRE_Match object; span=(0, 15), match='<h1>wahaha</h1>'>
<h1>wahaha</h1>
h1
wahaha
```

- 分组名命

  ```python
  # (?P<名字>正则表达式)
  ```

  ```python
  import re
  
  s1 = '<h1>wahaha</h1>'
  ret = re.search('<(?P<tag>\w+)>(?P<cont>.*?)</\w+>',s1)
  print(ret)
  print(ret.group('tag'))   # 取tag分组中的内容
  print(ret.group('cont'))   # 取cont分组中的内容
  
  <_sre.SRE_Match object; span=(0, 15), match='<h1>wahaha</h1>'>
  h1
  wahaha
  ```

- 引用分组

  ```python
  # (?P=组名)   这个组中的内容必须完全和之前已经存在的组匹配到的内容一模一样
  ```

  ```python
  import re
  
  s1 = '<h1>wahaha</h1>'
  ret = re.search('<(?P<tag>\w+)>.*?</(?P=tag)>',s1)
  print(ret.group('tag'))
  h1
  ```

- 分组和findall

  ```python
  import re
  
  ret = re.findall('\d(\d)','aa1alex83')
  # findall遇到正则表达式中的分组，会优先显示分组中的内容
  print(ret)
  ['3']
  ```

  ```python
  import re
  
  ret = re.findall('\d+(\.\d+)?','1.234+2') # 本应该匹配小数和整数的，可优先显示分组中的内容
  print(ret)
  ['.234', '']
  
  ret = re.findall('\d+(?:\.\d+)?','1.234+2')  # ?: 会取消优先显示分组内容
  print(ret)
  ['1.234', '2']
  ```

- 例题

  ```python
  # 有的时候我们想匹配的内容包含在不相匹配的内容当中，这个时候只需要把不想匹配的先匹配出来，再通过手段去掉
   
  # 取整数
  import re
  
  ret=re.findall("\d+\.\d+|(\d+)","1-2*(60+(-40.35/5)-(-4*3))")
  print(ret)
  ret.remove('')
  print(ret)
  ['1', '2', '60', '', '5', '4', '3']  # 小数没有显示，是因为优先显示分组中的内容--整数
  ['1', '2', '60', '5', '4', '3']
  ```

  ```python
  # 从一个四则运算（无括号）中找到第一个乘法或除法表达式
  
  import re
  
  val = '\d+(\.\d+)?[/*]-?\d+(\.\d+)?'
  obj = re.search(val,'1-2*3/6+5/6*4')
  print(obj.group())
  2*3
  ```



















































































