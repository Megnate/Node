# 知识图谱

挖个坑先。最近对知识图谱有个入门级的了解，希望以后还会有项目继续探究。年初实践了下怎么搭建一个最基本的知识图谱，顺便尝试了下使用Neo4j 搭建图数据库，在此稍作记录。入门选手可以看这篇科普帖：[知识图谱的技术与应用](https://zhuanlan.zhihu.com/p/38056557)

### 介绍

知识图谱到底是个什么东西？说到底就是拥有一定知识，可以进行相关知识问答的系统，这个系统可以不停地输入知识，也可以输出知识。其实，每个人都是一个行走的知识图谱。我们从小就被问“几岁啦？”，“叫什么名字呀？”，当我们进行回答时就是知识输出的过程。

本质上知识图谱的建立有如下几个过程：知识存储、知识融合、知识验证、知识计算。

- **知识存储：** 信息获取及存储的过程，比如3岁的时候学会背唐诗，8岁的时候学会加减乘除，15岁开始学物理，可以源源不断地获取不同的知识，语文、数学、英文等等。
- **知识融合：** 很多知识都是相互关联的，比如方程求解时会用到四则运算，求微分积分也会用到四则运算，很多知识都需要不断整合。
- **知识验证：** 对知识的正确性、实时性、整个体系的逻辑一致性的检验。比如2018年对于“小朋友你今年几岁啦？”的回答是3岁，那么到2019年这个回答的知识要更新为4岁了。
- **知识计算：** 一个知识体系是很庞大的，有很多很多的关系。比如“小朋友今年几岁了？”，需要对语义进行计算，明白问的问题是年龄相关，而且是当前年龄，需要计算当前时间与出生时间的差值。计算问题要的是什么样的答案，而不是问年龄要数字时返回一个“天空的颜色”的答案。

先分享下两个有趣的可视化图谱：

1. 漫威英雄的知识图谱可视化 [[demo link](https://graphics.straitstimes.com/STI/STIMEDIA/Interactives/2018/04/marvel-cinematic-universe-whos-who-interactive/index.html)]
2. 星际战争知识图谱可视化 [[demo link](http://codewithzhangyi.com/starwar_visualization/)]

### 搭建图数据库

虽然这两demo很大程度是前端写的好，对于我这种前端白痴，市面上已经有了相似的很好上手的开源工具（图数据库Neo4j），多用于金融的反欺诈，社交关系图谱等等。接下来我也尝试搭建一个基础的知识图谱。

需要思考几个基本问题：

1. 哪里来的数据？
2. 选择哪种数据库？数据存储格式
3. 如何将数据导入数据库？
4. 如何做到知识融合？进行数据库的增删改查？
5. 如何做知识计算？

#### 数据来源

介绍一个基于中文的知识图谱开源网站：[开放的中文知识图谱 openkg.cn](http://openkg.cn/)

这个网站里有图谱数据，图谱工具，资源涵盖：常识、金融、农业、社交、物联网、气象、生活、出行、科教、医疗等等。总之是比较全面的中文知识图谱开源库了。比如我用过的是四大名著里的西游记人物关系数据：[中国四大名著人物关系知识图谱和OWL本体](http://openkg.cn/dataset/ch4masterpieces) ，在数据与资源那边可自行下载。然后解压即可，就有csv格式的人物关系表格。

顺便介绍个openkg里的一个比较成型的知识图谱工具：[思知（OwnThink）](https://www.ownthink.com/) ，体验入口：[思知机器人](https://robot.ownthink.com/)

#### 数据库选择

虽然我一开始已经选定了**图数据库** ，这里还是将各种 **数据结构 和 数据库** 做个简单的对比：

数据类型可能是：

- **多元 结构化表格：** 就像excel表格一样整整齐齐
- **JSON 键值 ：** key-value 组
- **RDF 三元组：** 三元组就是两个实体，中间有一个关系。比如“中国的首都是北京”，两个实体分别是“中国”和“北京”，它们之间的关系是“的首都”，这两个实体加一个关系就是三元组。

不同的数据库：

- **MySQL：** 适合结构化数据，像excel表格
- **MongoDB：** 适合json格式数据
- **Neo4j：** 图数据库，可视化，适合三元组，并且实体和关系可以包含属性。

因此，选择什么样的数据库，是基于你的数据打算以什么样的格式存储。选择什么样的数据库以及怎么设计 schema。选**关系数据库**还是**NoSQL 数据库**？要不要用**内存数据库**？要不要用**图数据库**？这些都需要根据数据场景慎重选择。西游记的人物关系数据是三元组合格，这里我们选择尝试用Neo4j 图数据库。

#### 图数据库：Neo4j

Neo4j 是一个图数据库，主要包括节点和关系。节点和关系都可以包含属性。介绍文档 [[doc](http://neo4j.com.cn/public/docs/index.html)], 社区[[community](http://neo4j.com.cn/)]。

##### 安装

社区版下载链接：[neo4j-community-3.5.2](https://neo4j.com/download-thanks/?edition=community&release=3.5.2&flavour=winzip&_ga=2.190945954.961878394.1548140621-1486946031.1548140621)

解压后在bin目录下运行命令：`neo4j console`

```
D:\>cd D:\neo4j-community-3.5.2\bin

D:\neo4j-community-3.5.2\bin>neo4j console
2019-04-23 12:23:39.692+0000 INFO  ======== Neo4j 3.5.2 ========
2019-04-23 12:23:39.708+0000 INFO  Starting...
2019-04-23 12:23:42.950+0000 INFO  Bolt enabled on 127.0.0.1:7687.
2019-04-23 12:23:44.544+0000 INFO  Started.
2019-04-23 12:23:45.696+0000 INFO  Remote interface available at http://localhost:7474/
```

浏览器Chrome访问：localhost:7474，第一次访问会提示修改密码。

第一次玩可以参考这个入门小教程：[Neo4j 简单入门](https://blog.csdn.net/hxg117/article/details/79929579)
更详细的安装以及其他信息：[neo4j 安装](https://github.com/leondgarse/Atom_notebook/blob/master/public/2018/07-09_neo4j.md#安装)

#### Cypher 语法总结

Cypher入门：[tutorial](https://www.jianshu.com/p/53e2a67e9f40)

cypher就像MySQL的sql语言，demo示例：[neo4j_movie_graph.cypher](https://github.com/leondgarse/Atom_notebook/blob/master/public/2018/neo4j_movie_graph.cypher) 。cypher可对数据库做增删改查。更多的操作参考：[Cypher Basic](https://github.com/leondgarse/Atom_notebook/blob/master/public/2018/07-09_neo4j.md#cypher-basic) ，可以自己跑几行命令找找感觉。

#### Py2neo [[doc](https://py2neo.org/v4/index.html)]

毕竟写sql或者写cypher不是我擅长的事，所幸可以通过Python API **py2neo** 来访问neo4j。

[py2neo使用教程-1](https://github.com/leondgarse/Atom_notebook/blob/master/public/2018/07-09_neo4j.md#py2neo)
[py2neo使用教程-2](https://blog.csdn.net/sinat_26917383/article/details/79901207)
[py2neo使用教程-3](https://www.jianshu.com/p/da84712ef62b)

贴一些使用过的代码，要配合合适的数据集一起使用：

```python
import pandas as pd
import numpy as np
import itertools
from py2neo import Graph,Node,Relationship,NodeMatcher

# 终端运行 neo4j console之后就可以从py2neo连接neo4j数据库
graph = Graph("http://localhost:7474",username="neo4j",password="neo4j")
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
        node_b = Node('Product',name=j)
        rela = Relationship(node_a,'includes',node_b)
        
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
        node_target = Node('Product', name = _prod) # 'Product'是固定的
    else:
        node_target = matcher.match(name=_prod).first() # 'Product'是固定的

    for idx in range(len(df)):
        node_target[df['key'][idx]] = df['value'][idx]
    graph.push(node_target)
        
        
# add relationship
rel = pd.read_excel('add_relationship.xlsx')
# 检查是否存在节点 !!!希望后期的制表规则完善后可以省去检查这一步
# 如果不存在，就创建
# head 与 tail 如何区分

#nodes = list(itertools.chain.from_iterable([np.unique(rel['product']), np.unique(rel['node_name'])]))

for ind in range(len(rel)):
    # 如果不想重复遍历，就建chunk df
    if matcher.match(name=rel['node_head'][ind]).first() is None:
        node_head = Node(rel['head_type'][ind], name = rel['node_head'][ind])
        
    else:
        node_head = matcher.match(name=rel['node_head'][ind]).first()
    node_tail = matcher.match(name=rel['node_tail'][ind]).first()
    r = rel['rela'][ind]
    # 检查关系是否存在，若不存在，建立关系
    cmd = "MATCH a=()-[:%s]->( {name: '%s'}) RETURN a" %(r,rel['node_tail'][ind])
    if len(graph.run(cmd).data()) == 0:
        relatp = Relationship(node_head,r,node_tail)
        graph.create(relatp)
    

# ==============================================================
# Demo2: Room

room = pd.read_excel('room.xlsx')

for i in np.unique(room['owner']):
    node_a = Node('Owner',name=i)
    df = room[room['owner'] == i]
    
    _r = np.unique(df['room_name'])
    for j in _r:
        node_b = Node('Room',name=j)
        rela = Relationship(node_a,'lives',node_b)  
        graph.create(rela)
      
      
        _df = df[df['room_name']==j]
        _p = np.unique(_df['product_id'])
        for k in _p:
            node_c = Node('Prod',name=k)
            relatp = Relationship(node_b,'has',node_c)
            graph.create(relatp)
            
for i in np.unique(room['product_id']):
    # 这些点已经存在，只需要匹配
    node_a = matcher.match('Prod',name=i).first()
    df = room[room['product_id'] == i]    
    
    _c = np.unique(df['control'])
    for j in _c:
        node_b = Node('Ctrl',name=j)
        # 把node的答案作为属性填充上去，然后，方便用作cypher查询        
        rela = Relationship(node_a,'controls',node_b)
        
        graph.create(rela)

# ==============================================================
# Demo3: 西游记
xyj = pd.read_csv('./西游记/triples.csv')

#所有人物创建完，直接搜索建关系
n = [np.unique(xyj['head']),np.unique(xyj['tail'])]
for i in n:
    for p in i:
        node = Node('Person', name=p)
        graph.create(node)

for idx in range(len(xyj)):
    print(idx)
    node_a = matcher.match('Person',name=xyj['tail'][idx]).first()
    node_b = matcher.match('Person',name=xyj['head'][idx]).first()
    relatp = Relationship(node_a,xyj['label'][idx],node_b)
    graph.create(relatp)
    
    
# 查询 '孙悟空的师傅是谁'

len(graph.nodes)
len(graph.nodes.match("Person"))

# 'PERSON'根据命名实体匹配
test = matcher.match('Person',name='唐僧').first()

for rel in matcher.match(start_node=test, rel_type="徒弟"):
    print(rel.end_node()["name"])
```

### 拓展

#### 大批量数据导入参考：

[csv文件导入Neo4j(包括结点和关系的导入)](https://blog.csdn.net/quiet_girl/article/details/71155442)
[neo4j批量导入neo4j-import](https://blog.csdn.net/sinat_26917383/article/details/82424508)
[如何将大规模数据导入Neo4j](http://paradoxlife.me/how-to-insert-bulk-data-into-neo4j)

#### 删空属性

一般节点和关系可以通过py2neo删空，但是属性会存留：
[Neo4j - How to delete unused property keys from browser?](https://stackoverflow.com/questions/33982639/neo4j-how-to-delete-unused-property-keys-from-browser)

#### 别人开发的有趣的图谱成果 [[refer](https://blog.csdn.net/sinat_26917383/article/details/66473253)]

- 一家做NLP的公司：[wisers AI lab](https://www.wisers.ai/zh-cn/browse/relation-extraction/demo/)

- [zhishi.me](http://zhishi.me/)

- [Acemap](http://acemap.sjtu.edu.cn/)  交大

  - 贼有意思的网站以及随便点开的链接：[acemap.info](https://acemap.info/)
  - [main page 2018](https://www.acemap.info/ConferenceStatistics/MainPage?name=NIPS&year=2018)
  - [NeurIPS 2018](https://www.acemap.info/conference?confID=43319DD4)
  - https://www.acemap.info/topic?topicID=0304C748
  - https://www.acemap.info/paper-map?topicID=0304C748
  - 有趣的算法介绍：https://www.acemap.info/acenap/algorithms

- [CN-DBpedia](http://kw.fudan.edu.cn/cndbpedia/) 复旦大学，我现在点开属于网站维护中，所以这块以后再补充
  样例数据文件是txt格式，每行一条数据，每条数据是一个(实体名称，属性名称，属性值)的三元组，中间用tab分隔，具体如下所示。
  【复旦大学 简称 复旦】
  包含900万+的百科实体以及6700万+的三元组关系。其中mention2entity信息110万+，摘要信息400万+，标签信息1980万+，infobox信息4100万+。复旦大学还有个knowledge works http://kw.fudan.edu.cn/

- 可以下载数据集的网站：[grouplens](https://grouplens.org/datasets/movielens/)

- 一个中草药的知识服务系统：

  http://zcy.ckcest.cn/tcm/

  - 逆天http://zcy.ckcest.cn/tcm/qaos/profilenet

- NLPIR：http://ictclas.nlpir.org/nlpir/

==转载网址：==http://codewithzhangyi.com/2019/04/23/knowledge-graph-intro/

挺好的网站：[[知识图谱+Recorder︱中文知识图谱API与工具、科研机构与算法框架](https://www.cnblogs.com/jpfss/p/11423007.html)]

# 搭建博客

本篇的个人网站搭建教程基于 GitHub + Hexo。
适宜人群：想要有自己说话的地方，没人干涉。
系统环境：win10

## 搭建正文：

------

## 1. 准备软件的安装

- [Node.js](https://nodejs.org/en/)
- [Git](https://git-scm.com/)

## 2. 注册github

- 点击

  👉https://github.com

  右上角

  sign up

  ![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_01.PNG?raw=true)

  ![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_02.PNG?raw=true)

  个人网站的网址是固定格式： username.github.io

  这个username就是你的github用户名。当然也可以自己买域名啦。

  > 我的GitHub账号：YZHANG1270
  > 个人网站：yzhang1270.github.io
  > 但是我绑定域名啦~有个更酷炫的网址：[codewithzhangyi.com](http://www.codewithzhangyi.com/)，具体如何绑定将在后续篇介绍。

## 3. 创建Repository

登陆GitHub，点击右上角的 **+**号，选择**New repository** 创建一个与你的博客相关的Repository项目进行管理，之后所有你博客的动态都会在这Repository更新。
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_05.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_05.png?raw=true)
Repository的名字是username.github.io，比如我的yzhang1270.github.io已经创建。其余可以先不填，点击**Create repository**

## 4. 配置和使用Github

开始—所有应用—找到git bash
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_06.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_06.png?raw=true)

## 5. 配置SSH Keys

SSH Keys用来使本地git项目与GitHub联系，这样能在GitHub上的博客项目是最新更新的。

- 检查SSH Keys的设置

  首先检查自己电脑上现有的SSH Key：

  ```
  $ cd ~/.ssh
  ```

[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_07.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_07.png?raw=true)
如果显示 **No such file or directory**，说明这是你第一次用git

- 生成新的SSH Key:

  ```
  $ ssh-keygen -t rsa -C "邮件地址@youremail.com"
  Generating public/private rsa key pair.
  Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>
  ```

这里的邮箱地址，输入注册 Github 的邮箱地址
然后系统会要你输入密码：

```
Enter passphrase (empty for no passphrase):<设置密码>
Enter same passphrase again:<再次输入密码>
```



再回车，这里会提示你输入一个密码，作为你提交项目时使用。
这个密码的作用就是在个人网站里所有的改动只能经过你的手，也可以不设置密码，直接为空。
注意：输入密码的时候没有输入痕迹的，不要以为什么也没有输入。
最后看到这样的界面，就成功设置ssh key了：
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_08.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_08.png?raw=true)

- 添加SSH Key到GitHub上
  在本地文件夹找到id_rsa.pub文件，看上面的图片第四行的位置告诉你存在哪里了
  没找到的勾选一下文件扩展名 隐藏的项目
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_09.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_09.png?raw=true)
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_10.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_10.png?raw=true)
  .ssh文件夹里记事本打开这个文件复制全部内容到github相应位置
  回到你的GitHub主页，右上角点击头像选中**Setting**
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_11.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_11.png?raw=true)
  继续选中左边菜单栏的**SSH and GPG keys**
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_12.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_12.png?raw=true)
  Title最好写，随便写。网上有说不写title也有可能后期出现乱七八糟的错误
  Key部分就是放刚才复制的内容了，点击**Add SSH key**

## 6. 测试

回到git bash 框里
输入以下代码，不要改任何一个字。

```
$ ssh -T git@github.com
```



回车，看到如下：

```
The authenticity of host 'GitHub.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)
```



输入yes回车

```
Enter passphrase for key '/c/Users/Yi/.ssh/id_rsa':
```



输入刚才设置的密码回车，看到“You’ve successfully authenticated…”
成功！下一步！

## 7. 设置用户信息

现在已经可以通过 SSH 链接到 GitHub 啦!当然还需要完善一些个人信息:

```
$ git config --global user.name "yzhang1270" //输入注册时的username
$ git config --global user.email  "yzhang1270@gmail.com" //填写注册邮箱
```



GitHub 也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的。
到此，SSH Key配置成功啦！😀
本机本机已成功连接到 github。

如有问题，请重新设置。常见错误请参考：
[Connecting to GitHub with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/)
[Error: Permission denied](https://help.github.com/articles/error-permission-denied-publickey/)

## 9. 搭建Hexo博客

利用npm命令安装hexo

```
$ cd
$ npm install -g hexo
```



- 创建独立博客项目文件夹
  安装完成后，关掉之前的**Git Bash**窗口。在本地创建一个与 Repository中博客项目同名的文件夹username.github.io(如D:/yzhang1270.github.io)在文件夹上点击鼠标右键，选择 Git bash here(搞的我现在每次要写文章的时候脑子里冒出的第一句话永远是Bash Here!)

【提示】在进行博客搭建工作时，每次使用命令都要在D:/yzhang1270.github.io目录下。

执行下面的指令，Hexo 就会自动在 D:/yzhang1270.github.io 文件夹建立独立博客所需要的所有文件啦！

```
$ hexo init
```



- 安装依赖包

  ```
  $ npm install
  ```

- 确保git部署

  ```
  $ npm install hexo-deployer-git --save
  ```

- 本地查看
  恭喜你！👏现在已经搭建好本地的 Hexo 博客了，执行完下面的命令就可以到浏览器输入 localhost:4000 查看到啦！

  ```
  $ hexo g
  $ hexo s
  ```

hexo g 每次进行相应改动都要hexo g 生成一下
hexo s 启动服务预览

- 用Hexo克隆主题
  执行完 hexo init 命令后会给一个默认的主题：landscape
  里面还有一篇写好的示例文章：Hello World
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_13.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_13.png?raw=true)

你也可以到官网你喜欢的主题进行下载:[hexo themes](https://hexo.io/themes/)
[知乎：有哪些好看的 Hexo 主题？](https://www.zhihu.com/question/24422335/answer/46357100)

找到之后通过git命令下载
界面右侧，在主题的repository点击clone 复制一下那个地址
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_15.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_15.png?raw=true)

```
$ git clone +复制的地址+themes/typing
```



后面就是clone之后放到你本地的博客文件夹themes文件夹下
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_16.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_16.png?raw=true)
后面还可以将自己博客个性化装饰~

- 修改整站配置文件
  自己把blog.io中文件都点开看一遍，主要配置文件是 _config.yml，推荐使用 nodepad++ 打开。

修订清单如下，文档内有详细注释，可按注释逐个修订
(1)博客名字及作者信息：_config.yml
(2)个人介绍页面：about.md

```
这里贴一份网上看到的  可以复制替换原来的  但是替换之前最好备份 可能会出错
那要么你就对照着看一下改就好


# Hexo Configuration
## Docs: http://zespia.tw/hexo/docs/configure.html
## Source: https://github.com/tommy351/hexo/

# Site 这里的配置，哪项配置反映在哪里，可以参考我的博客
title: My Blog #博客名
subtitle: to be continued... #副标题
description: My blog #给搜索引擎看的，对网站的描述，可以自定义
author: Yourname #作者，在博客底部可以看到
email: yourname@yourmail.com #你的联系邮箱
language: zh-CN #中文。如果不填则默认英文

# URL #这项暂不配置，绑定域名后，欲创建sitemap.xml需要配置该项
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
tag_dir: tags
archive_dir: archives
category_dir: categories

# Writing 文章布局、写作格式的定义，不修改
new_post_name: :title.md # File name of new posts
default_layout: post
auto_spacing: false # Add spaces between asian characters and western characters
titlecase: false # Transform title into titlecase
max_open_file: 100
filename_case: 0
highlight:
  enable: true
  backtick_code_block: true
  line_number: true
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Archives 默认值为2，这里都修改为1，相应页面就只会列出标题，而非全文
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 1
category: 1
tag: 1

# Server 不修改
## Hexo uses Connect as a server
## You can customize the logger format as defined in
## http://www.senchalabs.org/connect/logger.html
port: 4000
logger: false
logger_format:

# Date / Time format 日期格式，可以修改成自己喜欢的格式
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-M-D
time_format: H:mm:ss

# Pagination 每页显示文章数，可以自定义，贴主设置的是10
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Disqus Disqus插件，我们会替换成“多说”，不修改
disqus_shortname:

# Extensions 这里配置站点所用主题和插件，暂时默认
## Plugins: https://github.com/tommy351/hexo/wiki/Plugins
## Themes: https://github.com/tommy351/hexo/wiki/Themes
theme: landscape
exclude_generator:
plugins:
- hexo-generator-feed
- hexo-generator-sitemap

# Deployment 站点部署到github要配置
## Docs: http://zespia.tw/hexo/docs/deploy.html
deploy:
  type: git
  repository: 
  branch: master
```

- 启用新下载的主题
  在刚打开的的_config.yml 文件中，找到“# Extensions”，把默认主题 landscape 修改为刚刚下载下来的主题名：

【提示】username.github.io 里有两个 config.yml 文件，一个在根目录，一个在 theme 下，现在修改的是在根目录下的。

- 更新主题
  git bash 里执行

  ```
  $ cd themes/主题名
  $ git pull
  ```

- 本地查看调试
  每次修改都要hexo g 生成一下

  ```
  $ hexo g #生成
  $ hexo s #启动本地服务，进行文章预览调试，退出服务用Ctrl+c
  ```

浏览器输入 localhost：4000 预览效果

## 10. 将博客部署到username.github.io

- 复制SSH码
  进入 Github 个人主页中的 Repository，复制新建的独立博客项目username.github.io的 SSH码
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_17.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_17.png?raw=true)

- 编辑整站配置文件
  打开 D:/username.github.io/_config.yml,把刚刚复制的 SSH码粘贴到**repository：**后面，别忘了冒号后要空一格。

  ```
  deploy:
    type: git
    repository: git@github.com:username/username.github.io.git
    branch: master
  ```

- 执行下列指令即可完成部署
  【提示】每次修改本地文件后，需要 hexo g 才能保存。每次使用命令时，都要在你的博客文件夹目录下：
  在D:/username.github.io/ 右键打开 **Git Bash Here**

  ```
  # 黄金三命令
  $ hexo g  //(g = generate 修改生产)
  $ hexo s  //(s = server   修改预览)
  $ hexo d  //(d = deploy   修改部署)
  ```

【提示】如果在配置 SSH key 时设置了密码，执行 hexo d 命令上传文件时需要输入密码进行确认，会出现一个小框框。
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_18.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_18.png?raw=true)
输入密码之后在浏览器输入：username.github.io

> Surprise🎉！恭喜你~
> 你已经拥有一个属于你自己的个人网站啦~嘿嘿

## 11. 写博客啦！

内涵才是重点！
在D:\username.github.io\source_posts的空白处右键Git Bash Here

```
hexo new 'article'
```



此时已经在D:\username.github.io\source_posts目录下有一个 article.md的Markdown文件
Hexo的博客都是用Markdown写的。我就随便写了点试试我的新博客啦~~
[![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_14.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_14.png?raw=true)

------

写博文参考：[如何写一篇hexo博客](https://www.jianshu.com/p/3c7ddd48bfa9)

搭建参考：

- [超详细Hexo+Github博客搭建小白教程](https://zhuanlan.zhihu.com/p/35668237)
- [如何搭建一个独立博客——简明Github Pages与Hexo教程](https://www.jianshu.com/p/05289a4bc8b2#)
- [技术小白搭建个人博客 github+hexo](https://zhuanlan.zhihu.com/p/32957389)

其他参考：

- [为什么你应该写博客](http://mindhacks.cn/2009/02/15/why-you-should-start-blogging-now/)
- [为什么要自建博客](https://www.zhihu.com/question/19916345)

本篇涉及Hexo个人网站的域名绑定、添加评论功能、访问次数统计设置等教程。
适宜人群：有追求的博主们（主题Theme未使用Next的博主们也请进）

## 域名绑定

根据上篇教程，目前默认的域名还是username.github.io，但印象中个人网站好像都是www.name.com格式的？怎么样想换吗？
首先✋，你得有___？ 当然是域名啦！

- 买域名
  你得买一个域名。xx云都能买，我在[腾讯云](https://dnspod.cloud.tencent.com/act/yearendsales?from=domainlist)买的。（请给我广告费！@腾讯云）
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_21.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_21.png?raw=true)
  我实在太喜欢Karlie Kloss，她的一个网站[kodewithklossy.com](https://www.kodewithklossy.com/)
  那么我的肯定就是codewithzhangyi.com😜
- 实名认证审核
  买好域名之后，[打开域名服务](https://console.cloud.tencent.com/domain/mydomain)，可以看见你刚买的域名记录。此时你需要做实名认证，很重要，超过有效期（大概3-5天）未认证域名将被锁定。
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_19.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_19.png?raw=true)
- 备案
  如果服务器是内地的就需要备案。Hexo网站的服务器是海外的因此可以跳过这步。
- 添加解析记录
  认证成功后的域名才能被做解析。认证需要3-5个工作日。我的大概半天就OK了。
  当解析状态变成【正常解析】时，点击域名记录最右侧操作的【解析】添加解析：
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_20.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_20.png?raw=true)
  解析记录设置两个：www和@，线路默认就ok
  (1) **@**:记录类型选A，A记录的就是ip地址，github(官方文档)提供了两个IP地址，192.30.252.153和192.30.252.154，这两个IP地址为github的服务器地址，两个都要填上
  (2) **www**:记录类型选CNAME，CNAME记录值填你的github博客网址，如我的是yzhang1270.github.io
- 本地设置修改
  这些全部设置完成后，此时你并不能根据申请的域名访问你的博客。接着你需要做的是在hexo根目录的source文件夹里创建CNAME文件，不带任何后缀，里面添加你的域名信息，如：codewithzhangyi.com。实践证明如果此时你填写的是www.codewithzhangyi.com那么以后你只能用www.codewithzhangyi.com访问，而如果你填写的是codewithzhangyi.com，那么用www.codewithzhangyi.com和codewithzhangyi.com访问都是可以的。重新hexo g,并发布即可用新的域名访问。
- 访问出现404的原因可能是：
  (1)绑定了个人域名，但是域名解析错误。
  (2)域名解析正确但你的域名是通过国内注册商注册的，你的域名因没有实名制而无法访问。
  (3)配置没问题的情况下，换个浏览器试试。
  (4)下载的hexo有问题，重新下载。

## 添加评论模块

> 在添加评论这个设置上费了点时间，因为整顿，评论服务挂了一大片，请各位寻找教程的时候重点看时间，2017年及之前的文章就没有多大的借鉴意义，包括这篇教程也是有时限性的，谁能跟的上变化呢。

(1) **多说** - 最多用户使用的评论，但遗憾2017年6月将暂定服务；不建议新用户使用，但为旧用户保留，也感谢多说一路的陪伴；
(2) **网易云跟帖** - 网易提供的评论组件，功能比较简单，性能优秀；管理后台在查询上还不算特别智能，但足够普通用户使用；
(3) **畅言** - 搜狐提供的评论组件，功能丰富，体验优异；但必须进行域名备案。只要域名备过案就可以通过审核。
(4) **Disqus** - 国外使用较多的评论组件。万里长城永不倒，一枝红杏出墙来，你懂的。
以上评论模块应该大家都知道，多说和网易云跟帖没有了，畅言要备案，对于对于挂靠在GitHub的博客非常的不友好，放弃！Disqus，不希望自己的博客，可以不分国界！也放弃！

踩坑总结：使用[Gitment评论服务](https://github.com/imsun/gitment)
Gitment 是作者imsun实现的一款基于 GitHub Issues 的评论系统。支持在前端直接引入，不需要任何后端代码。可以在页面进行登录、查看、评论、点赞等操作，同时有完整的 Markdown / GFM 和代码高亮支持。尤为适合各种基于 GitHub Pages 的静态博客或项目页面。

- 注册 OAuth Application
  注册一个新的[OAuth Application](https://github.com/settings/applications/new)，其他内容可以随意填写，但要确保填入正确的 callback URL（如 https:// yzhang1270.github.io）这个真的很重要！！！
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_22.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_22.png?raw=true)
  创建成功后，你会得到一个 client ID 和一个 client secret，这个将被用于之后的用户登录。
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_23.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_23.png?raw=true)

- 引入 Gitment
  将下面的代码添加到你的D:\username.github.io\themes\typing\layout_partial\after-footer.ejs：
  【提示】：我这里的主题是typing，在typing里是自带gitment的。

  ```
  <div id="container"></div>
  <link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
  <script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
  <script>
  var gitment = new Gitment({
    id: '页面 ID', // 可选。默认为 location.href  这里有机关，后面会再讲到！😎
    owner: '你的 GitHub Name',              //比如我的叫YZHANG1270
    repo: '存储评论的 repo',                 //比如我的叫YZHANG1270.github.io
    oauth: {
      client_id: '你的 client ID',          //比如我的328***********
      client_secret: '你的 client secret',  //比如我的49ce***********************
    },
  })
  gitment.render('container')
  </script>
  ```

- 可选：在主题的_config.yml中配置好全局参数：
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_24.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_24.png?raw=true)
  同时也要在脚本修改指定地址。

- 初始化评论
  页面发布后，你发现评论有个error:
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_25.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_25.png?raw=true)
  此时，你点击**Login**用自己账户登陆，再刷新页面就有初始化按键，点击初始化即可：
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_26.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_26.png?raw=true)
  正常情况下，只要用GitHub账户登陆即可发布评论啦：
  [![img](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_27.png?raw=true)](https://github.com/YZHANG1270/Markdown_pic/blob/master/hexo_web/web_27.png?raw=true)

> 在评论功能设置这里踩了几个坑，进行总结一下😝

## Gitment评论功能踩坑总结

- owner: ‘Your GitHub ID’

  ```
  owner: '你的 GitHub ID'
  ```

可以是你的GitHub用户名，也可以是GitHub id，建议直接用GitHub用户名就可以。
获取GitHub id的方法： https:// api.github.com/users/你的账户名

- Error: Not Found
  owner或者repo配置错误了，注意名字和仓库名字的大小写。

- Error: Comments Not Initialized
  (1)在注册OAuth Application这个步骤中，给Authorization callback URL指定的地址错了
  (2)还没有在该页面的Gitment评论区登陆GitHub账号
  (3)https://github.com/imsun/gitment/issues/95

- Error：validation failed
  这个真的折腾我一下午！！(咬牙切齿.jpg)
  issue的标签label有长度限制！labels的最大长度限制是50个字符。

  ```
  id: '页面 ID', // 可选。默认为 location.href
  ```

这个是之前配置的时候提到的机关。id的作用，就是针对一个文章有唯一的标识来判断这篇本章。

在issues里面，可以发现是根据网页标题来新建issues的，然后每个issues有两个labels（标签），一个是gitment，另一个就是id。
所以明白了原理后，就是因为id太长，导致初始化失败，现在就是要让id保证在50个字符内。
对应配置的id为：

```
id: '<%= page.title %>'
```



> 彩蛋🎊：

如果用网页标题也不能保证在50个字符！
最后，我用文章的时间，这样长度是保证在50个字符内，完美解决！（避免了文章每次更新标题或路径时，会重新创建一个issue评论的问题。）

```
id: '<%= page.date %>'
```



如果你原来没有设置id这一行，记得在这行后面加逗号，我就栽了傻了。

- Gitment的汉化
  只需到模板里将原来定义CSS和JS的那两行改成：

  ```
  <link rel="stylesheet" href="https://billts.site/extra_css/gitment.css">
  <script src="https://billts.site/js/gitment.js"></script>
  ```

- 所有文章一键初始化评论
  到本文编写时，还没有一个完善的解决方法，就是用脚本来执行自动化，有需要的可以详细了解：https://github.com/imsun/gitment/issues/5

参考文章：[Gitment评论功能接入踩坑教程](https://www.jianshu.com/p/57afa4844aaa)

## 文章/网站访问次数统计

访问次数统计也有很多方法，这里只简单介绍[不蒜子](http://ibruce.info/2015/04/04/busuanzi/)计数方法。很简单，就三步！

- 安装脚本
  打开D:\username.github.io\themes\typing\layout_partial\header.ejs
  在最后加入下面代码：

  ```
  <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js">
  </script>
  ```

- 显示站点总访问量
  要显示站点总访问量，复制以下代码添加到你需要显示的位置。有两种算法可选：
  (1)算法a：pv的方式，单个用户连续点击n篇文章，记录n次访问量。

  ```
  <span id="busuanzi_container_site_pv">
      本站总访问量<span id="busuanzi_value_site_pv"></span>次
  </span>
  ```

(2)算法b：uv的方式，单个用户连续点击n篇文章，只记录1次访客数。

```
<span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
```



打开themes/你的主题/layout/_partial/
你可以选择显示在网页的头部header.ejs文件里，文章的article.ejs文件里，或者网页的尾部after-footer.ejs文件里，等等。

- 显示单页面访问量

  要显示每篇文章的访问量，复制以下代码添加到你需要显示的位置。与上面同理。

  算法：pv的方式，单个用户点击1篇文章，本篇文章记录1次阅读量。

  ```
  <span id="busuanzi_container_page_pv">
    本文总阅读量<span id="busuanzi_value_page_pv"></span>次
  </span>
  ```

代码中文字是可以修改的，只要保留id正确即可。

【提示】：修改标题的文章、隔天再修改内容的文章，git会根据日期做版本控制。每篇文章的访问地址会因此更改。所以为了访问数建议一次性写完不要做改动了。

如果你看到这里，恭喜你，教程已经到此结束啦~快去试试吧！

------

> 另外，我是前端零基础小白，我的专业是人工智能机器学习类的，这网页也只是我的随意尝试，这些东西有时候还是会出错，毕竟分享内容才是我的初衷，其他还有很多可以改进的地方。
> 也欢迎大家的意见和指导在下面或者在微博给我留言。我发现一个学习的小窍门就是，在你喜欢的网页右键选择【查看网页源代码】，就能偷学人家的代码啦~嘿嘿😀
>
> 有一个我很喜欢的旅游博主，微博：北京小风子，她提到过心理学上有个词叫 positive reinforcement（正加强），这也是我不断写分享的初衷。
> 因为喜欢，才能坚持。希望写的东西对你有帮助，也期待你们的鼓励期待你们的打赏~
>
> 愿大家都玩的开心❤

转载：http://codewithzhangyi.com/2018/04/19/%E5%A6%82%E4%BD%95%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E4%B8%AA%E4%BA%BA%E7%BD%91%E7%AB%99%EF%BC%88%E4%B8%8A%EF%BC%89/

