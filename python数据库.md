# SQLite3数据库

使用**SQLite3**来创建数据库

```python
# 导入sqlite3模块（python内置的）
import sqlite3


# 1. 硬盘上创建连接
con = sqlite3.connect('e:/sqlitedb/first.db')

# 2. 获取cursor对象
cur = con.cursor

# 3. 执行sql创建表
sql = '''create table t_person(
		   # pno就是主要的参数，所以是 primary key
            pno INTEGER primary key autoincrement,
            pname VARCHAR not null,
            age INTEGER 
)'''

try:
    cur.execute(sql)
    print('创建表成功')
except Exception as e:
    print(e)
    print('表创建失败')
finally:
    # 关闭游标
    cur.close()
    #  关闭连接
    con.close()
```

