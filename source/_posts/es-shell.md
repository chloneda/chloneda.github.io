title: Elasticsearch启动、停止脚本
tags: Elasticsearch
categories: Elasticsearch
keywords: elasticsearch
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

[Elasticsearch官网](https://www.elastic.co/cn/)

构建Elasticsearch启动脚本 start_es.sh。
```
#!/bin/bash

export ES_HOME=xxx

su elastic -c "sh ${ES_HOME}/bin/elasticsearch -d -p ${ES_HOME}/pid"
```

参数说明：
- su：登录用户。
- elastic：部署Elasticsearch用户，避免root用户而无法启动。
- c：c参数后跟具体命令。
- d：Elasticsearch作为守护线程后台启动。
- p：指定线程ID文件，需要新建。


构建Elasticsearch停止脚本 stop_es.sh。
```
#!/bin/bash

export ES_HOME=xxx

kill `cat ${ES_HOME}/pid` 
``` `cat ${ES_HOME}/pid` 
```
