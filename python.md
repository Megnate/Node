### 利用 utf-8 来判断中英文字符

```python
def is_chinese(uchar):
    """判断一个unicode是否是汉字"""
    if uchar >= u'\u4e00' and uchar<=u'\u9fa5':
        return True
    else:
        return False

 

def is_number(uchar):
    """判断一个unicode是否是数字"""
    if uchar >= u'\u0030' and uchar<=u'\u0039':
        return True
    else:
        return False

 
def is_alphabet(uchar):
    """判断一个unicode是否是英文字母"""
    if (uchar >= u'\u0041' and uchar<=u'\u005a') or (uchar >= u'\u0061' and uchar<=u'\u007a'):
        return True
    else:
        return False

 
def is_other(uchar):
    """判断是否非汉字，数字和英文字符"""
    if not (is_chinese(uchar) or is_number(uchar) or is_alphabet(uchar)):
        return True
    else:
        return False
 

def B2Q(uchar):
    """半角转全角"""
    inside_code=ord(uchar)
    if inside_code<0x0020 or inside_code>0x7e:      #不是半角字符就返回原来的字符
        return uchar
    if inside_code==0x0020: #除了空格其他的全角半角的公式为:半角=全角-0xfee0
        inside_code=0x3000
        else:
            inside_code+=0xfee0
     return unichr(inside_code)
 

def Q2B(uchar):
    """全角转半角"""
    inside_code=ord(uchar)
    if inside_code==0x3000:
        inside_code=0x0020
    else:
        inside_code-=0xfee0
    if inside_code<0x0020 or inside_code>0x7e:      #转完之后不是半角字符返回原来的字符
         return uchar
    return unichr(inside_code)

 
def stringQ2B(ustring):
        """把字符串全角转半角"""
        return "".join([Q2B(uchar) for uchar in ustring])


def uniform(ustring):
        """格式化字符串，完成全角转半角，大写转小写的工作"""
        return stringQ2B(ustring).lower()
 

def string2List(ustring):
    """将ustring按照中文，字母，数字分开"""
    retList=[]
    utmp=[]
    for uchar in ustring:
        if is_other(uchar):
            if len(utmp)==0:
                continue
            else:
                retList.append("".join(utmp))
                utmp=[]
       else:
            utmp.append(uchar)
       if len(utmp)!=0:
            retList.append("".join(utmp))
            return retList
 

if __name__=="__main__":
    #test Q2B and B2Q
    for i in range(0x0020,0x007F):
        print Q2B(B2Q(unichr(i))),B2Q(unichr(i))
    #test uniform
    ustring=u'中国 人名ａ高频Ａ'
    ustring=uniform(ustring)
    ret=string2List(ustring)
    print ret
 

以上转自http://hi.baidu.com/fenghua1893/item/d1a71d5ac47ffdcfd3e10cd1
```

### 装饰器

```python
class FlaskBother():
    def __init__(self):
        self.routes = {}
    # route_str就是相关的路由，f就是关联的函数
    def route(self,route_str):
        def decorator(f):
            self.routes[route_str] = f
            return f
        return decorator
    # 通过这个函数去访问路由，再通过路由访问关联的函数，如果没有就返回一个错误
    def server(self, path):
        # 此时获取的字典中的key就是：'/'，其对应的value函数就是hello（）
        view_function = self.routes.get(path)
        if view_function:
            return view_function()
        else:
            raise ValueError('Route "{}" has not been registered'.format(path))


app = FlaskBother()

@app.route("/")
def hello():
    return "Hello World"

print(app.server('/'))
```

装饰器就是在一个函数内部定义另一个函数，然后返回一个新的函数

对于函数内部需要有函数名字签名的可能会发生错误，需要用到一个包：functools

```python
import functools
import time


def metric(fn):
    @functools.wraps(fn)
    def test(*args, **kwargs):
        print('%s executed in %s ms' % (fn.__name__, 10.24))
        return fn(*args, **kwargs)
    return test

@metric
def fast(x, y):
    time.sleep(0.00012)
    return x + y

@metric
def slow(x, y, z):
    time.sleep(0.1234)
    return x * y * z

f = fast(11, 22)
print(f)
s = slow(11, 22, 33)
print(s)
if f != 33:
    print('测试失败!')
elif s != 7986:
    print('测试失败!')
else:
    print('ok')
print(fast.__name__)
```

