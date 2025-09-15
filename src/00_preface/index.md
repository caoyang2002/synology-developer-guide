# 前言

# 群晖API开发初探

群晖的DSM系统功能强大，又是私有化部署，是非常好的家庭共享存储或小型公司的小型存储方案。群晖的套件众多，比如PhotoStation、Moments、FileStation、DownloadStation、CloudSync、NoteStation等有很多的教程，但其实DSM系统的API功能也非常强大，这里初探一下使用方法。

事实上，群晖几乎各个套件都有完整的API的支持，但因为文档不够完整，参考资料也非常少，所以使用非常不便。参考官网开发专区 的FileStation API文档，可以了解到API开发的流程：

获取API 信息：

```py
import json
import requests
ret = requests.get('http://myds.com:port/webapi/query.cgi?api=SYNO.API.Info&version=1&method=query&query=all')
ret = json.loads(ret)
if ret['success']:
  print(ret['data'])
```

query参数如果是all，可以获取所有的API信息，也可以指定查询特定的一批API。比如：

```py
#注意NoteStation后的'.'，可以查出NoteStation下所有的API
query_str = 'SYNO.API.Auth,SYNO.NoteStation.'
```

用户登录

```py
import json
import requests
ret = requests.get('http://myds.com:port/webapi/auth.cgi?api=SYNO.API.Auth&version=3&method=login&account=admin&passwd=12345&session=FileStation&format=sid')
ret = json.loads(ret)
if ret['success']:
  print(ret['data'])
```

如果登录成功，可以获得sid，在后续的所有API调用上一定要附带参数_sid。

调用API

```py
import json
import requests
ret = requests.get('http://myds.com:portGET /webapi/entry.cgi?api=SYNO.NoteStation.Note&method=get&version=3&object_id=NOTE_ID&_sid=SID')
ret = json.loads(ret)
if ret['success']:
  print(ret['data'])
```

这里显示的是NoteStation获取某一篇笔记详细信息的API，由于NoteStation没有API文档，只能通过浏览器的请求来分析得到，比如创建笔记的参数：

```bash
POST /webapi/entry.cgi HTTP/1.1

commit_msg: {"device":"desktop","listable":false}
title: "无标题便签"
parent_id: "1001_ABCDEFGHIJKLMNPQRSTUVW"
encrypt: false
api: SYNO.NoteStation.Note
method: create
version: 3
```

用户登出

```py
import json
import requests
ret = requests.get('http://myds.com:port/webapi/auth.cgi?api=SYNO.API.Auth&version=1&method=logout&session=FileStation')
ret = json.loads(ret)
if ret['success']:
  print('Logout OK.')
```
经过一番努力，顺利利用API获取、创建NoteStation的文章，可以方便的扩展功能了。在使用API的过程中，也发现了很多奇怪的问题，比如创建Note时，Title不能为纯数字，会导致API返回失败，群晖加油，希望可以对开发者提供更完整的社区支持！

# 开发者手册

https://help.synology.cn/developer-guide/
