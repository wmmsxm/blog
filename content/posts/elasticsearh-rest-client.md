---
title: "elasticsearh-rest-client"
date: 2019-11-22T10:51:38+08:00
tags: ["elasticsearch"]
categories: ["elasticsearch"]
draft: false
---
## <center>elasticsearch rest client(一)</center>
### low level rest client使用和问题
#### low client使用
low level rest client使用http与elasticsearch进行通信，不会对请求进行编码和响应编码。它与所有elasticsearch版本兼容。  
如果在命令或者语法上有问题，建议查看官方的文档，需要查看java doc的同学可以点击[这里](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.5/java-rest-low.html "文档")

1. maven配置
```
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-client</artifactId>
        <version>7.3.2</version>
    </dependency>
```

2. elasticsearch配置
restClient可以设置请求头和请求回调。此处不多加叙述，网上介绍有很多，不懂的可以去看看。
```
@Component
@Configuration
public class EsConfig {

	private static final String HTTP_SCHEME = "http";

	@Value("${elasticsearch.config.address}")
	private String ip;

	@Value("${elasticsearch.config.port}")
	private Integer port;

	@Bean
	public RestClient client() {
		HttpHost[] hosts = new HttpHost[]{new HttpHost(ip, port, HTTP_SCHEME)};
		RestClientBuilder builder = RestClient.builder(hosts);
		// 添加header
		Header[] defaultHeader = new Header[]{
				// 内容格式
				new BasicHeader("content-type", "application/json"),
				// 长连接
				new BasicHeader("Connection", "keepalive"),
		};
		builder.setDefaultHeaders(defaultHeader);
		// 默认连接池    线程1
		builder.setHttpClientConfigCallback(httpAsyncClientBuilder ->
				httpAsyncClientBuilder.setDefaultIOReactorConfig(IOReactorConfig.custom().setIoThreadCount(1).build())
		);

		builder.setRequestConfigCallback(builder1 -> builder1
				// 连接超时
				.setConnectTimeout(30000)
				// 数据请求超时
				.setSocketTimeout(40000)
				// 连接池超时设置
				.setConnectionRequestTimeout(0)

		);
		builder.setFailureListener(new RestClient.FailureListener(){
			@Override
			public void onFailure(Node node) {
				super.onFailure(node);
			}
		});

		return  builder.build();
	}

}
```

3. 封装基层语法
低级客户端的命令都需要进行拼接，所以本人就封装了一下底层，并添加了一个简单的方法封装工具。源代码在[gitee](https://gitee.com/wmmsxm/yuzhi_tools.git "源代码")上。
[!工具结构](/images/elasticsearch/es-2.png)

4. 案例
可以参考这个[案例](https://gitee.com/wmmsxm/btw_learn/tree/master/elasticsearch-seven "案例")来使用的封装的方法。


#### low client问题
封装的low rest client方法中，可能会出现以下问题
- Low-level REST client status error:ava.lang.IllegalStateException: Request cannot be executed; I/O reactor status: STOPPED  

    这个异常的原因主要是httpClient中的IOReactor被关闭了，原因有很多。出现异常后，再次访问时，就可以正常访问。  
    查了百度和谷歌，基本上没有能完全解决这个问题的，有的说是内存原因，有的则是减少client链接，或者使用新线程创建client。  
    本人的解决方式  在restClient.performRequest(request)处捕捉IO异常，再次请求；超过2次(可配)，返回null，异常打印。方式如下图：  
    ![异常处理](/images/elasticsearch/es-1.png)