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

