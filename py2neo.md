# py2neo的使用

```python
import pandas as pd
import numpy as np
import itertools
from py2neo import Graph, Node, Relationship, NodeMatcher

# 终端运行 neo4j console之后就可以从py2neo连接neo4j数据库
graph = Graph("http://localhost:7474", username="neo4j", password="123456")
matcher = NodeMatcher(graph)

# graph.delete_all()  # 清空本地数据库的命令，慎用

# ============================================================
# Demo1: Product
# 目的：把能展示的应用都展示出来

prod = pd.read_excel('type.xlsx')
list(prod)

# 设置基础信息
for i in np.unique(prod['分组类型']):
    node_a = Node('Type', name=i)

    df = prod[prod['分组类型'] == i]
    _p = np.unique(df['产品类型'])
    for j in _p:
        node_b = Node('Product', name=j)
        rela = Relationship(node_a, 'includes', node_b)

        graph.create(rela)

# 因为点Node需要定义性质，所以建议 key-value 和 rela-node分开建表
# add property/relationship时需要第一列的产品已存在，否则需要更多信息
# ------------------------------------------------------------
# add property
prop = pd.read_excel('data/add_property.xlsx')

for _prod in np.unique(prop['product']):

    df = prop[prop['product'] == _prod].reset_index(drop=True)

    # 如果库内没有该产品，进行Node建立
    if matcher.match(name=_prod).first() is None:
        node_target = Node('Product', name=_prod)  # 'Product'是固定的
    else:
        node_target = matcher.match(name=_prod).first()  # 'Product'是固定的

    for idx in range(len(df)):
        node_target[df['key'][idx]] = df['value'][idx]
    graph.push(node_target)

# add relationship
rel = pd.read_excel('add_relationship.xlsx')
# 检查是否存在节点 !!!希望后期的制表规则完善后可以省去检查这一步
# 如果不存在，就创建
# head 与 tail 如何区分

# nodes = list(itertools.chain.from_iterable([np.unique(rel['product']), np.unique(rel['node_name'])]))

for ind in range(len(rel)):
    # 如果不想重复遍历，就建chunk df
    if matcher.match(name=rel['node_head'][ind]).first() is None:
        node_head = Node(rel['head_type'][ind], name=rel['node_head'][ind])

    else:
        node_head = matcher.match(name=rel['node_head'][ind]).first()
    node_tail = matcher.match(name=rel['node_tail'][ind]).first()
    r = rel['rela'][ind]
    # 检查关系是否存在，若不存在，建立关系
    cmd = "MATCH a=()-[:%s]->( {name: '%s'}) RETURN a" % (r, rel['node_tail'][ind])
    if len(graph.run(cmd).data()) == 0:
        relatp = Relationship(node_head, r, node_tail)
        graph.create(relatp)

# ==============================================================
# Demo2: Room

room = pd.read_excel('room.xlsx')

for i in np.unique(room['owner']):
    node_a = Node('Owner', name=i)
    df = room[room['owner'] == i]

    _r = np.unique(df['room_name'])
    for j in _r:
        node_b = Node('Room', name=j)
        rela = Relationship(node_a, 'lives', node_b)
        graph.create(rela)

        _df = df[df['room_name'] == j]
        _p = np.unique(_df['product_id'])
        for k in _p:
            node_c = Node('Prod', name=k)
            relatp = Relationship(node_b, 'has', node_c)
            graph.create(relatp)

for i in np.unique(room['product_id']):
    # 这些点已经存在，只需要匹配
    node_a = matcher.match('Prod', name=i).first()
    df = room[room['product_id'] == i]

    _c = np.unique(df['control'])
    for j in _c:
        node_b = Node('Ctrl', name=j)
        # 把node的答案作为属性填充上去，然后，方便用作cypher查询
        rela = Relationship(node_a, 'controls', node_b)

        graph.create(rela)

# ==============================================================
# Demo3: 西游记
xyj = pd.read_csv('./西游记/triples.csv')

# 所有人物创建完，直接搜索建关系
n = [np.unique(xyj['head']), np.unique(xyj['tail'])]
for i in n:
    for p in i:
        node = Node('Person', name=p)
        graph.create(node)

for idx in range(len(xyj)):
    print(idx)
    node_a = matcher.match('Person', name=xyj['tail'][idx]).first()
    node_b = matcher.match('Person', name=xyj['head'][idx]).first()
    relatp = Relationship(node_a, xyj['label'][idx], node_b)
    graph.create(relatp)

# 查询 '孙悟空的师傅是谁'

len(graph.nodes)
len(graph.nodes.match("Person"))

# 'PERSON'根据命名实体匹配
test = matcher.match('Person', name='唐僧').first()

for rel in matcher.match(start_node=test, rel_type="徒弟"):
    print(rel.end_node()["name"])
```

## 查找节点

```python
graph = Graph("http://localhost:7474", username="neo4j", password="123456")
matcher = NodeMatcher(graph)

node_1 = matcher.match(name=).first()
```

## 查找关系

```python
# 检查关系是否存在，若不存在，建立关系
cmd = "MATCH a=()-[:%s]->( {name: '%s'}) RETURN a" % (r, rel['node_tail'][ind])
if len(graph.run(cmd).data()) == 0:
    relatp = Relationship(node_head, r, node_tail)
    graph.create(relatp)
```

# Neo4j的语句

```
1.如何找到一个节点x，x以某种关系同时连接两个不同节点a和b
match (a)-[r:relation]->(x)<-[r:relation]-(b) return x

2.如何找到节点a和b之间的最短路径
(1)match p=shortestpath((a)-[r:relation]-(b)) return nodes(p)
(2)match(n:na{name:’###’}),(m:nb{name:’###’})with n,m match p=shortestpath((n)-[r*…]-(m)) return p;


3.如何找到节点a和b之间以某种关系相连接的最短路径
p=shortestpath((a)-[r:relationname]->(b)) return nodes(p)

4.找到数据库中出现的唯一节点标签
match n return distinct labels(n)

5.找到数据库中出现的唯一关系类型
match n-[r]-() return distinct type(r)

6.找到数据库中的唯一节点标签和唯一关系类型
match n-[r]-() return distinct labels(n),type(r)

7.找到不与任何关系（或某种关系）向连的节点
start n = node() match n-[r:relationname]-() where r is null return n

8.找到某个带有特定属性的节点
start n=node() match n where has (n.someproperty) return n

9.找到与某种关系相连接的全部节点
start n= node() match n-[r:relationshipname]-() return distinct n

10.找到节点和它们的关系数，并以关系数目降序排列显示
start n=node() match n-[r]-() return n,count(r) as rel_count order by rel_count desc

11.返回图中所有节点的个数
start n = node() match n return count(n)

12.
（1）删除图中关系：start n=node(*) match n-[r]-() delete r
（2）删除图中节点：start n =node(*) match n delete n
（3）删除图中所有东西：match (n) detach delete n

13.查询某类节点下某属性为特定值的节点
match (n:person)where n.name=”alice” return n

14.with
Cypher中的With关键字可以将前步查询的结果作为后一步查询的条件，这个在我的工作中可是帮了大忙哈哈。下面是两个栗子。
（1）match(p:node_se)-[re:推理条件]->(q:node_se) where p.name=‘FEV1%pred’and p.value=’<30%’ WITH p,re,q match (q:node_se) <-[re2:推理条件]- (c:node_se)return p, re,q,re2,c
（2）match(p:node_patient)-[re:个人情况]->(q:node_se) where p.name=‘qwe’ WITH p,re,q match (q:node_se) -[re2:推荐方案]-> (c:node_se) where q.name=‘first’ WITH p, re,q,re2,c match (c:node_se)-[re3:方案细节]->(d:drugs) return p, re,q,re2,c,re3,d

15.查询符合条件的某个节点的id
match(p) where p.name = ‘***’ and p.value = ‘***’ return id(p)

16.直接连接关系节点进行多层查询
match(na:bank{id:‘001’})-[re1]->(nb:company)-[re2]->(nc:people) return na,re1,nb,re2,nc

17.可以将查询结果赋给变量，然后返回
match data=(na:bank{id:‘001’})-[re1]->(nb:company)-[re2]->(nc:company) return data

18.变长路径检索
变长路径的表示方式是：[*N…M]，N和M表示路径长度的最小值和最大值。
(a)-[ *2]->(b)：表示路径长度为2，起始节点是a，终止节点是b；
(a)-[ *3…5]->(b)：表示路径长度的最小值是3，最大值是5，起始节点是a，终止节点是b；
(a)-[ *…5]->(b)：表示路径长度的最大值是5，起始节点是a，终止节点是b；
(a)-[ *3…]->(b)：表示路径长度的最小值是3，起始节点是a，终止节点是b；
(a)-[ *]->(b)：表示不限制路径长度，起始节点是a，终止节点是b；

19.Cypher对查询的结果进行去重
栗：match(p:node_se)-[re]->(q)where re.name <> ‘and’ return distinct(re.name)
（注：栗子中的<>为Cypher中的操作符之一，表示‘不等于’）

20．更新节点的 labels
Neo4j中的一个节点可以有多个 label，返回所有节点的label:match (n) return labels(n)
修改节点的 label，可以先新加 label，再删除旧的label
match (n:label_old) set n:label_new remove n:label_old
match(n:label_new) return labels(n)

21.更新节点的属性
match(n:) set n.new_property = n.old_property remove n.old_proerty
```

# 图谱

```python
from py2neo import Node, Relationship
a = Node('Person', name='Alince')
b = Node('Person', name='Alinc')  # 表示创建一个节点，其标签是Person，有一个name的属性值
ab = Relationship(a, 'KNOWS', b)  # 这个边得标签是KNOWS
```

当上诉创建成功后，也可以进行`增，改，查`

因为Node，Relationship都继承了PropertyDict类，所以可以存在很多属性，且类似于字典的形式，可以通过以下方式接着上面的代码进行赋值：

```python
a['age'] = 21
b['age'] = 20
ab['time'] = '2021-03-16'
print(a,b,ab)
```

还可以通过`Setdefault`的方式**赋予默认值**：

```python
a.setdefault('location', '北京')
print(a)
>>> (alice:Person {age:20,location:"北京",name:"Alice"})
```

可以通过`udate()`的方式进行**批量更新**：

```python
data = {
    'name': 'Amy',
    'age': 21
}
a.update(data)
```

## 包含方法

### 节点

- `hash(node) `返回node的ID的哈希值
- `node[key]` 返回node的属性值，没有此属性就返回None
- `node[key] = value` 设定node的属性值
- `del node[key]` 删除属性值，如果不存在此属性报KeyError
- `len(node) `返回node属性的数量
- `dict(node) `返回node所有的属性
- `walk(node)`返回一个生成器且只包含一个node
- `labels() `返回node的标签的集合
- `has_label(label)` node是否有这个标签
- `add_label(label)` 给node添加标签
- `remove_label(label)` 删除node的标签
- `clear_labels() `清楚node的所有标签
- `update_labels(labels)` 添加多个标签，注labels为可迭代的
  ————————————————
  版权声明：本文为CSDN博主「悟乙己」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/sinat_26917383/article/details/79901207

### 边

- `hash(relationship)` 返回一个关系的hash值
- `relationship[key] `返回关系的属性值
- `relationship[key] = value `设定关系的属性值
- `del relationship[key]` 删除关系的属性值
- `len(relationship) `返回关系的属性值数目
- `dict(relationship) `以字典的形式返回关系的所有属性
- `walk(relationship) `返回一个生成器包含起始node、关系本身、终止node
- `type() `返回关系type
  ————————————————
  版权声明：本文为CSDN博主「悟乙己」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/sinat_26917383/article/details/79901207

## 子图

子图的节点和关系不可变：

```python
from py2neo import Node, Relationship
 
a = Node('Person', name='Alice')
b = Node('Person', name='Bob')
r = Relationship(a, 'KNOWS', b)
s = a | b | r
print(s)
>>> ({(alice:Person {name:"Alice"}), (bob:Person {name:"Bob"})}, {(alice)-[:KNOWS]->(bob)})

# 获取所有的Node和Relationship
print(s.nodes())
print(s.relationships())
```

获取两个子图的交集：

```python
s1 = a | b | r
s2 = a | b
print(s1 & s2)
>>> ({(alice:Person {name:"Alice"}), (bob:Person {name:"Bob"})}, {})
```

### 方法

```python
from py2neo import Node, Relationship, size, order
s = a | b | r
print(s.keys())
print(s.labels())
print(s.nodes())
print(s.relationships())
print(s.types())
print(order(s))
print(size(s))

>>> frozenset({'name'})
>>> frozenset({'Person'})
>>> frozenset({(alice:Person {name:"Alice"}), (bob:Person >>> >>> >>> {name:"Bob"})})
>>> frozenset({(alice)-[:KNOWS]->(bob)})
>>> frozenset({'KNOWS'})
>>> 2
>>> 1
```

- `subgraph | other | … `子图的并
- `subgraph & other & …` 子图的交
- `subgraph - other - … `子图的差
- `subgraph ^ other ^ … `子图对称差
- `subgraph.keys()` 返回子图节点和关系所有属性的集合
- `subgraph.labels()` 返回节点label的集合
- `subgraph.nodes() `返回所有节点的集合
- `subgraph.relationships() `返回所有关系的集合
- `subgraph.types() `返回所有关系的type的集合
- `order(subgraph) `返回子图节点的数目
- `size(subgraph)` 返回子图关系的数目
  ————————————————
  版权声明：本文为CSDN博主「悟乙己」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/sinat_26917383/article/details/79901207

## Walkable Type

是一个拥有遍历功能的子图，最简单的构造就是将子图合并

```python
from py2neo import Node, Relationship
 
a = Node('Person', name='Alice')
b = Node('Person', name='Bob')
c = Node('Person', name='Mike')
ab = Relationship(a, "KNOWS", b)
ac = Relationship(a, "KNOWS", c)
w = ab + Relationship(b, "LIKES", c) + ac
print(w)
>>> (alice)-[:KNOWS]->(bob)-[:LIKES]->(mike)<-[:KNOWS]-(alice)
```

调用`walk()`方法进行遍历：

```python
from py2neo import walk
 
for item in walk(w):
    print(item)

>>> 
(alice:Person {name:"Alice"})
(alice)-[:KNOWS]->(bob)
(bob:Person {name:"Bob"})
(bob)-[:LIKES]->(mike)
(mike:Person {name:"Mike"})
(alice)-[:KNOWS]->(mike)
(alice:Person {name:"Alice"})
```

### 方法

```python
print(w.start_node())
print(w.end_node())
print(w.nodes())
print(w.relationships())

>>> (alice:Person {name:"Alice"})
>>> (alice:Person {name:"Alice"})
>>> ((alice:Person {name:"Alice"}), (bob:Person {name:"Bob"}), (mike:Person {name:"Mike"}), (alice:Person {name:"Alice"}))
>>> ((alice)-[:KNOWS]->(bob), (bob)-[:LIKES]->(mike), (alice)-[:KNOWS]->(mike))
```

- `walk(walkable)` 转为一个生成器包含节点和关系
- `start_node()` 返回walk()的起始节点
- `end_node()` 返回walk()的最后节点
- `nodes()` 返回walk()所有节点的元组
- `relationships()` 返回walk()所有关系的元组

## 查询

### 通过`node`的**ID**进行查询：

```python
graph = Graph()
# 其中的数字对应的是节点，ID
# 这个ID不按顺序来的，要注意
graph.nodes[1234]
graph.nodes.get(1234)
```

### 通过`match`的方式：

```python
# .run/.data查询
test_graph.data("MATCH (a:Person {name:'You'}) RETURN a")
>>> [{'a': (c7d1cb9:Person {name:"You"})}]
list(test_graph.run("MATCH (a:Person {name:'You'}) RETURN a"))
>>>[('a': (c7d1cb9:Person {name:"You"}))]
test_graph.run("MATCH (a:Person {name:'You'}) RETURN a").data()
>>>[{'a': (c7d1cb9:Person {name:"You"})}]
# 查询关系
test_graph.run("MATCH (a:Person {name:'You'})-[b:FRIEND]->(c:Person {name:'Johan'}  )   RETURN a,b,c")
```

其查询结果也可以是`dataform`的形式：

```python
pd.DataFrame(test_graph.data("MATCH (a:Person {name:'Anna'}) RETURN a"))
                  a
0  {'name': 'Anna'}
1  {'name': 'Anna'}
2  {'name': 'Anna'}
3  {'name': 'Anna'}
```

查询出来的结果是list/dict形式的，不是graph类型的，所以后续的查询时不能够进行的，但是可以标准化一些格式：

```python
# graph查询
graph.run("MATCH (n:leafCategory) RETURN n LIMIT 25").data()  # list型
graph.run("MATCH (n:leafCategory) RETURN n LIMIT 25").to_data_frame()  # dataframe型
graph.run("MATCH (n:leafCategory) RETURN n LIMIT 25").to_table()  # table
```

### 通过`find/find_one`的形式：

`find`查找全部，需要传入不定参数`label,property_value,property_key,limit`返回符合条件的生成器

`find_one`只是查找单节点，需要传入不定参数`label,property_value,property_key`，返回符合条件的一个节点

```python
# 查找全部
graph=test_graph.find(label='Person')
for node in graph:
    print(node)
>>>(b54ad74:Person {age:18,name:"Johan"})
(b1d7b9d:Person {name:"Rajesh"})
(cf7fe65:Person {name:"Anna"})
(d780197:Person {name:"Julia"})
# 查找单节点
test_graph.find_one(label='Person',property_key='name',property_value='You')
>>> (c7d1cb9:Person {name:"You"})
```

返回的都是graph的图类型的数据，所以可以继续进行查询

**判断节点是否存在：**`# 该节点是否存在 test_graph.exists(graph.nodes[1234])`

### NodeMatcher

在4中没有这个函数了，变成：`class.py2neo.matching.NodeMatcher(graph)`

```python
selector = NodeMatcher(test_graph)
#selector = NodeSelector(test_graph)
list(selector.match("Person", name="Anna"))
list(selector.match("Person").where("_.name =~ 'J.*'", "1960 <= _.born < 1970"))
```

筛选`age == 21的Person  Node`

```python
from py2neo import Graph, NodeMatcher
 
graph = Graph(password='123456')
selector = NodeMatcher(graph)
#selector = NodeSelector(graph)
persons = selector.match('Person', age=21)
print(list(persons))
```

用`where()`进行更复杂的查询

```python
from py2neo import Graph, NodeSelector
 
graph = Graph(password='123456')
selector = NodeMatcher(graph)
persons = selector.match('Person').where('_.name =~ "A.*"')
print(list(persons))
```

`order_by`进行排序：

```python
from py2neo import Graph, NodeSelector
 
graph = Graph(password='123456')
selector = NodeMatcher(graph)
persons = selector.match('Person').order_by('_.age')
print(list(persons))
```

- `first()`返回单个节点
- `limit(amount)`返回底部节点的限值条数
- `skip(amount)`返回顶部节点的限值条数
- `order_by(*fields)`排序
- `where(*conditions, **properties)`筛选条件

### match/match_one()查询关系

match 匹配关系

match_one 匹配并返回所有满足条件的唯一条关系

```python
// 此时start_node为节点
for rel in test_graph.match(start_node=node3, rel_type="FRIEND"):
    print(rel.end_node()["name"])
>>>Johan
Julia
Andrew

# match_one
test_graph.match_one(start_node=node3, rel_type="FRIEND")
>>> (c7d1cb9)-[:FRIEND]->(b54ad74)
```

## 重设，更新

push 和set一样：更新添加，

```python
node = test_graph.find_one(label='Person')
node['age'] = 18
test_graph.push(node)
print(test_graph.find_one(label='Person'))
>>> (b54ad74:Person {age:18,name:"Johan"})
```

setdafault() 方法：设置默认值

```python
a.setdefault('location', '北京')
print(a)
>>> (alice:Person {age:20,location:"北京",name:"Alice"})
```

udate() ：批量更新

```python
data = {
    'name': 'Amy',
    'age': 21
}
a.update(data)
print(a)
```

## 删除

`delete(subgraph) `删除节点、关系或子图
`delete_all() `删除数据库所有的节点和关系

```python
from py2neo import Graph
 
graph = Graph(password='123456')
node = graph.find_one(label='Person')
relationship = graph.match_one(rel_type='KNOWS')
graph.delete(relationship)
graph.delete(node)
```

想删除节点就必须删除对应的关系，并且必须先找到这些节点才可以删除

# OGM

Object Graph Mapping  可以实现一个对象和一个Node的关联

```python
from py2neo.ogm import GraphObject, Property, RelatedTo, RelatedFrom


class Movie(GraphObject):
    __primarykey__ = 'title'

    title = Property()
    released = Property()
    actors = RelatedFrom('Person', 'ACTED_IN')
    directors = RelatedFrom('Person', 'DIRECTED')
    producers = RelatedFrom('Person', 'PRODUCED')

class Person(GraphObject):
    __primarykey__ = 'name'

    name = Property()
    born = Property()
    acted_in = RelatedTo('Movie')
    directed = RelatedTo('Movie')
    produced = RelatedTo('Movie')
```

然后结合graph查询

```python
from py2neo import Graph
from py2neo.ogm import GraphObject, Property

graph = Graph(password='123456')


class Person(GraphObject):
    __primarykey__ = 'name'

    name = Property()
    age = Property()
    location = Property()

person = Person.select(graph).where(age=21).first()
print(person)
print(person.name)
print(person.age)
```

通过动态映射改变node和relationship

```python
#修改某个node 的属性
person = Person.select(graph).where(age=21).first()
print(person.__ogm__.node)
person.age = 22
print(person.__ogm__.node)
graph.push(person)


# 移除某一个关联的node
person = Person.select(graph).where(name='Alice').first()
target = Person.select(graph).where(name='Durant').first()
person.knows.remove(target)
graph.push(person)
graph.delete(target)


# 添加一个关联的node
from py2neo import Graph
from py2neo.ogm import GraphObject, Property, RelatedTo

graph = Graph(password='123456')

class Person(GraphObject):
    __primarykey__ = 'name'

    name = Property()
    age = Property()
    location = Property()
    knows = RelatedTo('Person', 'KNOWS')

person = Person.select(graph).where(age=21).first()
print(list(person.knows))
new_person = Person()
new_person.name = 'Durant'
new_person.age = 28
person.knows.add(new_person)
print(list(person.knows))
```

此时的数据库还没有更新，需要以下的代码：

```python
graph.push(person)
```

## Matcher

```python
matcher = RelationshipMatcher(g)
selector = NodeMatcher(g)

>>> from py2neo import Graph, NodeMatcher
>>> graph = Graph()
>>> matcher = NodeMatcher(graph)
>>> matcher.match("Person", name="Keanu Reeves").first()
(_224:Person {born:1964,name:"Keanu Reeves"})


matcher = RelationshipMatcher(graph) 
result = matcher.match({node1,node2},'know')

print list(result)
# match可以加入多个node：{node1, node2}
```

边关系的查询：`class py2neo.matching.RelationshipMatch(graph, nodes=None, r_type=None, conditions=(), order_by=(), skip=None, limit=None)`