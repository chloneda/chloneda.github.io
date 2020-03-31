---
title: Elasticsearch客户端源码剖析
tags: Elasticsearch
categories: Elasticsearch
keywords: Elasticsearch
comments: false
---


**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)


# 前言

今天我们来聊聊Elasticsearch客户端的类型。我们知道Elasticsearch是一种分布式的海量数据搜索与分析的技术，可以用于电商网站、门户网站、企业IT系统等各种场景下的搜索引擎，也可以用于对海量的数据进行近实时的数据分析。

但Elasticsearch版本迭代更新太快，这就意味着在Elasticsearch升级过程中容易出现兼容性问题。也引出了今天对Elasticsearch客户端种类及使用的问题讨论！

# ES客户端种类

ES官方客户端有TransportClient、Java Low Level REST Client和Java High Level REST Client三种。官方文档对他们的说明是：

**TransportClient**

> We plan on deprecating the TransportClient in Elasticsearch 7.0 and removing it completely in 8.0.

**Java Low Level REST Client**

> the official low-level client for Elasticsearch. It allows to communicate with an Elasticsearch cluster through http. Leaves requests marshalling and responses un-marshalling to users. It is compatible with all Elasticsearch versions.

**Java High Level REST Client**

> the official high-level client for Elasticsearch. Based on the low-level client, it exposes API specific methods and takes care of requests marshalling and responses un-marshalling.

意思就是说，TransportClient将会在将来版本进行废弃移除，官方建议使用Java High Level REST Client。

为什么会这样呢？这里涉及到两个问题：
- 未来版本为什么会淘汰TransportClient客户端？
- Java Low/High Level REST Client客户端优点在哪里？

先别急，我们来看看这两个问题！



# 客户端的使用

各客户端使用需要引入相关依赖，这里统一引入相关依赖，后面就不多赘述了！

```bash
<!-- elasticsearch core -->
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>${elasticsearch.version}</version>
</dependency>
<!-- low level rest client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>${elasticsearch.version}</version>
</dependency>
<!-- high level rest client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>${elasticsearch.version}</version>
</dependency>
```

**注**：TransportClient将会在8.x版本后完全移除！



## TransportClient

初始化TransportClient客户端代码示例：

```bash
public TransportClient initTransportClient(String esClusterName,String host,String port) throws UnknownHostException {
    Settings settings = Settings.builder()
            .put("cluster.name", esClusterName)
            .put("client.transport.sniff", true)
            .build();
    TransportClient client = new PreBuiltTransportClient(settings)
            .addTransportAddress(new TransportAddress(InetAddress.getByName(host),port);

    return client;
}
```


## Java Low Level REST Client

初始化 RestClient 客户端代码示例：

```bash
public RestClient initRestClient(String host, int port) {
    RestClientBuilder builder = RestClient.builder(new HttpHost(host,
            port, "http"));
    Header[] defaultHeaders = new Header[]{new BasicHeader("header", "value")};
    builder.setDefaultHeaders(defaultHeaders);
    RestClient restClient = builder.build();
    return restClient;
}
```

## Java High Level REST Client

初始化 RestHighLevelClient 客户端代码示例：

```bash
public RestHighLevelClient restHighLevelClient(List<String> hostArray) {
    //创建HttpHost数组，其中存放es主机和端口的配置信息
    HttpHost[] httpHostArray = new HttpHost[hostArray.size()];
    for (int i = 0; i < hostArray.size(); i++) {
        String item = hostArray.get(i);
        httpHostArray[i] = new HttpHost(item.split(":")[0], 
                Integer.parseInt(item.split(":")[1]), 
                "http");
    }
    //创建RestHighLevelClient客户端
    return new RestHighLevelClient(RestClient.builder(httpHostArray));
}
```

以上就是初始化三种不同客户端的示例代码！下面我们深入客户端代码底层，看看他们之间有什么不一样？



# 深入客户端的底层

## TransportClient

TransportClient客户端自从Elasticsearch诞生以来，一直是Elasticsearch的一部分。 它是一个特殊的客户端，因为它使用传输层协议（TCP）与Elasticsearch进行通信，如果该客户端与其所使用的Elasticsearch不在同一版本上，则会导致兼容性问题。基于这个原因，官方会在8.x后完全移除！

因此，在这里就不对 TransportClient 客户端底层进行深究了！



## Java Low Level REST Client

2016年，Elasticsearch官方发布了一个低级REST客户端，该客户端基于众所周知的Apache HTTP客户端，它允许使用 HTTP 与任何版本的Elasticsearch集群进行通信。 

我们来看看RestClient客户端的代码：

```bash
package org.elasticsearch.client;

public class RestClient implements Closeable {

    //已省略其他非必要属性代码。。。
    
    // RestClient 类构造器的第一个参数是 CloseableHttpAsyncClient，是Apache HTTP client 中的类，相关请求也是通过该参数

    RestClient(CloseableHttpAsyncClient client, long maxRetryTimeoutMillis, Header[] defaultHeaders, HttpHost[] hosts, String pathPrefix, RestClient.FailureListener failureListener) {
        this.client = client;
        this.maxRetryTimeoutMillis = maxRetryTimeoutMillis;
        this.defaultHeaders = Collections.unmodifiableList(Arrays.asList(defaultHeaders));
        this.failureListener = failureListener;
        this.pathPrefix = pathPrefix;
        this.setHosts(hosts);
    }

    //已省略其他非必要代码。。。

    public void performRequestAsync(String method, String endpoint, Map<String, String> params, HttpEntity entity, HttpAsyncResponseConsumerFactory httpAsyncResponseConsumerFactory, ResponseListener responseListener, Header... headers) {
        try {
            Objects.requireNonNull(params, "params must not be null");
            Map<String, String> requestParams = new HashMap(params);
            String ignoreString = (String)requestParams.remove("ignore");
            Object ignoreErrorCodes;
            if (ignoreString == null) {
                if ("HEAD".equals(method)) {
                    ignoreErrorCodes = Collections.singleton(404);
                } else {
                    ignoreErrorCodes = Collections.emptySet();
                }
            } else {
                String[] ignoresArray = ignoreString.split(",");
                ignoreErrorCodes = new HashSet();
                if ("HEAD".equals(method)) {
                    ((Set)ignoreErrorCodes).add(404);
                }

                String[] var12 = ignoresArray;
                int var13 = ignoresArray.length;

                for(int var14 = 0; var14 < var13; ++var14) {
                    String ignoreCode = var12[var14];

                    try {
                        ((Set)ignoreErrorCodes).add(Integer.valueOf(ignoreCode));
                    } catch (NumberFormatException var17) {
                        throw new IllegalArgumentException("ignore value should be a number, found [" + ignoreString + "] instead", var17);
                    }
                }
            }

            URI uri = buildUri(this.pathPrefix, endpoint, requestParams);
            HttpRequestBase request = createHttpRequest(method, uri, entity);
            this.setHeaders(request, headers);
            RestClient.FailureTrackingResponseListener failureTrackingResponseListener = new RestClient.FailureTrackingResponseListener(responseListener);
            long startTime = System.nanoTime();
            this.performRequestAsync(startTime, this.nextHost(), request, (Set)ignoreErrorCodes, httpAsyncResponseConsumerFactory, failureTrackingResponseListener);
        } catch (Exception var18) {
            responseListener.onFailure(var18);
        }

    }

    //已省略其他非必要代码。。。

    private static HttpRequestBase createHttpRequest(String method, URI uri, HttpEntity entity) {
        String var3 = method.toUpperCase(Locale.ROOT);
        byte var4 = -1;
        switch(var3.hashCode()) {
        case -531492226:
            if (var3.equals("OPTIONS")) {
                var4 = 3;
            }
            break;
        case 70454:
            if (var3.equals("GET")) {
                var4 = 1;
            }
            break;
        case 79599:
            if (var3.equals("PUT")) {
                var4 = 6;
            }
            break;
        case 2213344:
            if (var3.equals("HEAD")) {
                var4 = 2;
            }
            break;
        case 2461856:
            if (var3.equals("POST")) {
                var4 = 5;
            }
            break;
        case 75900968:
            if (var3.equals("PATCH")) {
                var4 = 4;
            }
            break;
        case 80083237:
            if (var3.equals("TRACE")) {
                var4 = 7;
            }
            break;
        case 2012838315:
            if (var3.equals("DELETE")) {
                var4 = 0;
            }
        }

        switch(var4) {
        case 0:
            return addRequestBody(new HttpDeleteWithEntity(uri), entity);
        case 1:
            return addRequestBody(new HttpGetWithEntity(uri), entity);
        case 2:
            return addRequestBody(new HttpHead(uri), entity);
        case 3:
            return addRequestBody(new HttpOptions(uri), entity);
        case 4:
            return addRequestBody(new HttpPatch(uri), entity);
        case 5:
            HttpPost httpPost = new HttpPost(uri);
            addRequestBody(httpPost, entity);
            return httpPost;
        case 6:
            return addRequestBody(new HttpPut(uri), entity);
        case 7:
            return addRequestBody(new HttpTrace(uri), entity);
        default:
            throw new UnsupportedOperationException("http method not supported: " + method);
        }
    }

}
```

看到上面的代码，**RestClient** 类构造器的第一个参数是 **CloseableHttpAsyncClient**，是 Apache HTTP client 中的类，也就是说 [RestClient](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.x/_motivations_around_a_new_java_client.html) 是基于 Apache HTTP 实现的，这里是 Apache HTTP 的依赖！

```bash
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>${http.version}</version>
</dependency>

```



## Java High Level REST Client

最重要的是，我们发布了基于低级客户端的高级REST客户端，它负责请求编组和响应解组。

我们来看看 RestHighLevelClient 的底层代码！

```bash
package org.elasticsearch.client;

public class RestHighLevelClient {
    private final RestClient client;
    private final NamedXContentRegistry registry;

    public RestHighLevelClient(RestClient restClient) {
        this(restClient, Collections.emptyList());
    }

    // 此处省略多处代码！

    // 该类大部分方法最终会调用以下 performRequestAndParseEntity 方法，我们主要看该方法的调用关系

    protected <Req extends ActionRequest, Resp> Resp performRequestAndParseEntity(Req request, CheckedFunction<Req, Request, IOException> requestConverter, CheckedFunction<XContentParser, Resp, IOException> entityParser, Set<Integer> ignores, Header... headers) throws IOException {
        return this.performRequest(request, requestConverter, (response) -> {
            return this.parseEntity(response.getEntity(), entityParser);
        }, ignores, headers);
    }

    protected <Req extends ActionRequest, Resp> Resp performRequest(Req request, CheckedFunction<Req, Request, IOException> requestConverter, CheckedFunction<Response, Resp, IOException> responseConverter, Set<Integer> ignores, Header... headers) throws IOException {
        ActionRequestValidationException validationException = request.validate();
        if (validationException != null) {
            throw validationException;
        } else {
            Request req = (Request)requestConverter.apply(request);

            Response response;
            try {
            
                // 这里的 client 就是RestClient，最终还是调用 RestClient 的方法，也就是说 RestHighLevelClient 是基于 RestClient 的
                
                response = this.client.performRequest(req.getMethod(), req.getEndpoint(), req.getParameters(), req.getEntity(), headers);
            } catch (ResponseException var13) {
                ResponseException e = var13;
                if (ignores.contains(var13.getResponse().getStatusLine().getStatusCode())) {
                    try {
                        return responseConverter.apply(e.getResponse());
                    } catch (Exception var11) {
                        throw this.parseResponseException(var13);
                    }
                }

                throw this.parseResponseException(var13);
            }

            try {
                return responseConverter.apply(response);
            } catch (Exception var12) {
                throw new IOException("Unable to parse response body for " + response, var12);
            }
        }
    }

}

```

看上面的代码及注解，我相信你很快就豁然开朗了！

其实上面的问题现在就有答案了！TransportClient废弃的主要原因就是考虑到兼容性的问题，而后续两个客户端在兼容性方面就做的很好！



# 小结

关于Elasticsearch的客户端问题，其实 [ES官网](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.x/_motivations_around_a_new_java_client.html) 已经说得很明确了，这里也通过代码剖析的方式去认识一下底层的代码，加深理解！

由此可见，HighLevelClient 是基于 RestClient,而 RestClient 又是基于 Apache HTTP 客户端， 这样一来, 在客户端方面, Elasticsearch 将 Java, Python, Php, Javascript 等各种语言的底层接口就都统一起来了; 与此同时, 使用 rest api, 还可以屏蔽各版本之前的差异。

这也提醒我们，在代码的升级过渡期, 处理好新 client 和旧 client 的关系，可以减少代码后期维护的工作量！



-------
