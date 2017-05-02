---
title: json.loads&json.dumps的使用
date: 2017-03-10 00:34:49
tags: [Python基础]
---


每次遇到json loads/dumps始终搞不清方向，写段代码试下：


```python

import json  

dict_ = {1:2, 3:4, "55":"66"}  

# test json.dumps  

print type(dict_), dict_  
json_str = json.dumps(dict_)  
print "json.dumps(dict) return:"  
print type(json_str), json_str  

# test json.loads  
print "\njson.loads(str) return"  
dict_2 = json.loads(json_str)  
print type(dict_2), dict_2

# 程序结果：

<type 'dict'> {'55': '66', 1: 2, 3: 4}
json.dump(dict) return:
<type 'str'> {"55": "66", "1": 2, "3": 4}


json.loads(str) return
<type 'dict'> {u'55': u'66', u'1': 2, u'3': 4}
```
总结：

json.dumps : dict转成str

json.loads:str转成dict

如此简单。